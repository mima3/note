# 目的  
PHPのテンプレートフレームワークである[Twig](https://twig.symfony.com/)とCSSのフレームワークである[Bulma](https://bulma.io/)でページネーションをやってみる  
![image.png](/image/6717907d-e77c-f24d-5da3-a3697c6e622b.png)  
  
![image.png](/image/fac445ea-3cc0-b8ae-62d2-1ae2bf53ec26.png)  
  
![image.png](/image/aa4ff01d-0c32-3661-8868-aa5c661539d9.png)  
  
![image.png](/image/5bdffc15-ca1f-4ec5-39ca-ee7cc14a2141.png)  
  
![image.png](/image/1a439f93-928c-657e-f341-aeb82fb6ed23.png)  
  
![image.png](/image/e04b0af9-55ba-8e33-8199-d299c71fba70.png)  
  
![image.png](/image/4a82e298-fb17-af2d-5955-435c382e6c0c.png)  
  
![image.png](/image/08d4c82b-9e53-1d0c-3125-17a0e37f9334.png)  
  
## コード  
PHPでテンプレートを使用する際に下記のデータを指定してください。  
  
 - currentPage: 現在のページ番号  
 - maxPage : 最大ページ番号  
 - pageRange : ここで指定したページ数分、最初のページ, 最終ページ, 現在ページの前後ページへのリンクを省略せずに表示する。  
  
## PHP側  
  
```php
        return $this->view->render(
            $response,
            'commitlog.twig',
            [
                'BASE_PATH' => $this->config['BASE_PATH'],
                'commitlogs' => $commitlogs,
                'pageLimit' => $limit,
                'currentPage' => $page,
                'maxPage' => $maxPage,
                'pageRange' => 2
            ]
        );
```  
  
## ページネーション用のテンプレート  
  
**pagination.php**  
```twig:pagination.php
<nav class="pagination is-centered" role="navigation" aria-label="pagination">
    {% if currentPage != 1 %}
        <a class="pagination-previous" href="?page={{currentPage - 1}}">Previous</a>
    {% endif %}
    {% if currentPage != maxPage %}
        <a class="pagination-next" href="?page={{currentPage + 1}}">Next page</a>
    {% endif %}
    <ul class="pagination-list">
        {% set preItemHasEllipsis = false %}
        {% for i in range(1,maxPage) %}
            {% if i == currentPage %}
                <li><a class="pagination-link is-current" aria-lasbel="Goto page {{i}}">{{i}}</a></li>
                {% set preItemHasEllipsis = false %}
            {% elseif (i <= 1 + pageRange) or 
                      (i >= maxPage - pageRange) or
                      ((currentPage - pageRange <= i) and (i <= currentPage + pageRange))
            %}
                <li><a class="pagination-link" aria-lasbel="Goto page {{i}}" href="?page={{i}}">{{i}}</a></li>
                {% set preItemHasEllipsis = false %}
            {% elseif preItemHasEllipsis == false and ( 
                        (i == 1 + 1 + pageRange) or 
                        (i == maxPage - pageRange - 1) or
                        (i == currentPage - pageRange - 1) or
                        (i == currentPage + pageRange + 1)
                    )
            %}
                <li><span class="pagination-ellipsis">&hellip;</span></li>
                {% set preItemHasEllipsis = true %}
            {% endif %}
        {% endfor %}                
    </ul>
</nav>
```  
  
## 読み出し側のテンプレート  
  
```twig
// 略
        <h1 class="title">コミットログ</h1>
        <div class="content">
            <div class="table-container">
             // 略
            </div>

            {{ include("component/pagination.twig")}}
// 略
```  
  
# メモ  
ループは効率が悪そうなので、速度が重要なら分岐でうまいこと作った方がいいと思う。  
