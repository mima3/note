このドキュメントではASP.NET MVCとXBAPをどうやって連携するか考える。  
  
## XBAPのファイルをどう配置すべきか？  
XBAPは別プロジェクトにする。  
ASP.NET MVCのプロジェクトはXBAPのプロジェクトに依存させる。  
https://qiita-image-store.s3.amazonaws.com/0/47856/37328a2f-44a7-049c-e1bc-41fb48b4437d.png  
  
ASP.NET MVCのxabapフォルダを対象にxbapを発行するようにしとけばいい。  
VisualStudioからXBAPプロジェクトを選んで一々発行するのもいいが、以下のようにASP.NET MVCプロジェクトのビルド前イベントにバッチを組むといい。  
  
1.MSBUILD用のスクリプトを記述する。  
  
```xml
<?xml version="1.0" encoding="utf-8"?>
<Project ToolsVersion="4.0" DefaultTargets="Publish" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
    <PropertyGroup>
        <!-- the folder of the project to build -->
        <ProjLocation>.\WpfBrowserApplication1</ProjLocation>
        <ProjLocationReleaseDir>$(ProjLocation)\bin\Release</ProjLocationReleaseDir>
        <ProjPublishLocation>$(ProjLocationReleaseDir)\app.publish</ProjPublishLocation>
        <!-- This is the web-folder, which provides the artefacts for click-once. After this
         build the project is actually deployed on the server -->
        <DeploymentFolder>MvcApplicationTest\xbap</DeploymentFolder>
    </PropertyGroup>


    <Target Name="Publish" DependsOnTargets="Clean">
        <Message Text="Publish-Build started for build no $(ApplicationRevision)" />
        <MSBuild Projects="$(ProjLocation)/WpfBrowserApplication1.csproj" Properties="Configuration=Release" Targets="Publish"/>   


        <ItemGroup>
            <SchoolPlannerSetupFiles Include="$(ProjPublishLocation)\*.*"/>
            <SchoolPlannerUpdateFiles Include="$(ProjPublishLocation)\Application Files\**\*.*"/>
        </ItemGroup>
        <Copy
            SourceFiles="@(SchoolPlannerSetupFiles)"
            DestinationFolder="$(DeploymentFolder)\"
        />
        <Copy
            SourceFiles="@(SchoolPlannerUpdateFiles)"
            DestinationFolder="$(DeploymentFolder)\Application Files\%(RecursiveDir)"
        />      
    </Target>
    <Target Name="Clean">   
        <Message Text="Clean project:" />
        <MSBuild Projects="$(ProjLocation)/WpfBrowserApplication1.csproj" Properties="Configuration=Release" Targets="Clean"/>
    </Target>       
</Project>
```  
  
2.ASP.NET MVCプロジェクトのビルド前イベントで以下のようなバッチを記述する  
  
```
Mage.exe -cc
"C:\Program Files (x86)\MSBuild\12.0\Bin\MSBuild.exe" "$(SolutionDir)deployxbap.xml" /target:publish
```  
  
Marge.exeはXBAPのキャッシュをクリアしている。  
  
http://stackoverflow.com/questions/1919625/msbuild-doesnt-respect-publishurl-property-for-my-clickonce-app  
  
## ASP.NET MVCからどうXBAPにデータを渡すべきか？  
セッションは無理。  
クエリーストリングを使うか、ブラウザーのクッキーつかうか、どっちか。  
  
### クエリーストリングを使う方法  
1.XBAPプロジェクトのプロパティで「発行」のオプションを押す  
https://qiita-image-store.s3.amazonaws.com/0/47856/9c0e6947-006e-28a1-543b-63a53fa537ec.png  
  
2.マニフェストで「URLパラメータをアプリケーションに渡すことを許可する」  
https://qiita-image-store.s3.amazonaws.com/0/47856/34e4e078-9865-3fca-a8a5-5d919387818a.png  
  
3.XBAPプロジェクトに「System.Deployment」を参照設定する。  
  
4.ASP.NET MVCプロジェクトからXBAPのパスを呼ぶときにパラメータを与える。  
  
```html
    <iframe height="130"
        width="100%"
        src="/xbap/WpfBrowserApplication1.xbap?param=1" />
```  
  
5.XBAP側の実装で以下のようにするとクエリー付きのURLがとれるので解析しとく。  
  
```csharp
Uri launchUri = System.Deployment.Application.ApplicationDeployment.CurrentDeployment.ActivationUri;
```  
  
### セッションを使う方法  
  
1.ASP.NET MVCプロジェクトのビューで以下のような実装をする。ASPXの例だが、razorとかでもできるはず。  
  
```aspx-cs
    <%
        // Construct the cookies.
HttpCookie cookie = new HttpCookie("MyData");
cookie.Value = HttpUtility.HtmlEncode("MyData");

//Add to the response stream
Response.Cookies.Add(cookie);
    %>
```  
  
2.XBAPプロジェクト側は次のような実装で、クッキーの中身をとる  
  
```csharp
            string cookieString = "";
            try
            {
                String cookie = Application.GetCookie(System.Windows.Interop.BrowserInteropHelper.Source);
                cookieString = cookie;
            }
            catch (Exception)
            {
            }
```  
  
http://bytes.com/topic/net/answers/844652-pass-parameters-xbap-webpage  
