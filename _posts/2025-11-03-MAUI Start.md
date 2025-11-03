---
title: "MAUI 시작"
date: 2025-11-03 00:00:00 +0900
categories: [MAUI]
tags: [MAUI]     # TAG names should always be lowercase
---

### 시작할 때 참고하면 좋을 링크
[MAUI Doc](https://learn.microsoft.com/en-us/dotnet/maui/?view=net-maui-9.0)  
[MAUI Beginners Guide Video](https://youtube.com/playlist?list=PLdo4fOcmZ0oUBAdL2NwBpDs32zwGqb9DY&si=kOB4Jo0jaleIfTAt)

# 폴더 구조
<details markdown="1">
<summary>참고 링크</summary>

> [Get started with XAML](https://learn.microsoft.com/ko-kr/dotnet/maui/xaml/fundamentals/get-started?view=net-maui-9.0)  
> [런타임 시 XAML 로드](https://learn.microsoft.com/ko-kr/dotnet/maui/xaml/runtime-load?view=net-maui-9.0)  
> [Application Class](https://learn.microsoft.com/en-us/dotnet/api/microsoft.maui.controls.application?view=net-maui-9.0)  
> [.NET MAUI Shell 개요](https://learn.microsoft.com/ko-kr/dotnet/maui/fundamentals/shell/?view=net-maui-9.0)  
> [SemanticScreenReader.Announce(String)](https://learn.microsoft.com/ko-kr/dotnet/api/microsoft.maui.accessibility.semanticscreenreader.announce?view=net-maui-9.0)  
> [플랫폼 코드 호출](https://learn.microsoft.com/ko-kr/dotnet/maui/platform-integration/invoke-platform-code?view=net-maui-9.0)  
> [네이티브 포함](https://learn.microsoft.com/ko-kr/dotnet/maui/platform-integration/native-embedding?view=net-maui-9.0&pivots=devices-android)
> [종속성 주입](https://learn.microsoft.com/ko-kr/dotnet/maui/fundamentals/dependency-injection?view=net-maui-9.0)

</details>

## 기본 설명
![MAUI Project Folder](/assets/Images/MAUIProjectFolder.png)

- xaml파일에는 비하인드 코드 cs파일이 포함되어 있다.

![Behind cs](https://learn.microsoft.com/ko-kr/dotnet/maui/xaml/fundamentals/media/get-started/new-solution.png?view=net-maui-9.0.png)

## App

### App.xaml

- 앱의 레이아웃에 사용하는 리소스에 대한 정의가 되어있다.
- 기본적으로 리소스 파일에 리소스가 존재하는데, 이 코드는 앱의 색과 스타일에 대해 정의하고 있다.

```xml
<?xml version = "1.0" encoding = "UTF-8" ?>
<Application xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:local="clr-namespace:MauiApp1"
             x:Class="MauiApp1.App">
    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <ResourceDictionary Source="Resources/Styles/Colors.xaml" />
                <ResourceDictionary Source="Resources/Styles/Styles.xaml" />
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Application.Resources>
</Application>
```

> **ResourceDictionar 란?**  
> 색, 스타일, 템플릿 등을 한 곳에서 정의하여 리소스를 앱 전체에서 참조할 수 있게 해준다.  
> [Resource dictionaries](https://learn.microsoft.com/en-us/dotnet/maui/fundamentals/resource-dictionaries?view=net-maui-9.0) 
{: .prompt-info }

### App.xaml.cs

- 앱의 진입점 역할을 하는 코드이다.
- MAUI의 기본 애플리케이션 클래스 “Application”를 상속받고 있다.
- InitializeComponent() 메서드를 호출하여 xaml 리소스를 로드한다.
- CreateWindow() 메소드에서 앱이 시작될 때 어떤 화면을 보여줄 지 정의하고 있다. 이 코드에서는 “AppShell”이라는 화면을 보여줄 것이다. 
이 코드를 변경하면 시작할 때 보여줄 화면을 바꿀 수 있다.
- OnStart(), OnSleep(), OnResume()을 상속받아 구현하면 각 이벤트에 맞는 처리를 할 수도 있다.
    - OnStart(): 앱이 시작될 때
    - OnSleep(): 앱이 백그라운드로 전환될 때
    - OnResume(): 앱이 다시 활성화 될 때

```csharp
public partial class App : Application
{
    public App()
    {
        InitializeComponent();
    }

    protected override Window CreateWindow(IActivationState? activationState)
    {
        return new Window(new AppShell());
    }
}
```

## AppShell

### AppShell.xaml

- AppShell은 MAUI앱의 네비게이션 구조와 UI계층을 정의한다. 앱의 전체적인 화면 구성와 이동 방식을 관리한다.
- Title: 탭이나 메뉴에 표시될 이름을 정의한다.
- ContentTemplate: 표시할 페이지를 지정한다. 이 ContentTemplate를 다른 Page로 변경하면 다른 화면을 보여줄 수 있다.
    - 탭, 플라이아웃, 계층적 네비게이션(탭+하위페이지) 등을 구성할 수도 있다.
        
        ![ContentTemplate Pages](https://learn.microsoft.com/ko-kr/dotnet/maui/user-interface/pages/media/contentpage/pages.png?view=net-maui-9.0.png)
        
- Route: URI 기반 네비게이션용 라우트 이름

```csharp
<Shell
    x:Class="MauiApp1.AppShell"
    xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
    xmlns:local="clr-namespace:MauiApp1"
    Title="MauiApp1">

    <ShellContent
        Title="Home"
        ContentTemplate="{DataTemplate local:MainPage}"
        Route="MainPage" />

</Shell>
```

> **URI 기반 탐색?**  
> 설정된 탐색 계층 구조를 따르지 않고도 경로를 사용하여 앱의 모든 페이지로 이동할 수 있는 환경이다.  
> [.NET MAUI Shell 탐색](https://learn.microsoft.com/ko-kr/dotnet/maui/fundamentals/shell/navigation?view=net-maui-9.0) 
{: .prompt-info }

## MainPage

### MainPage.xaml

- 앱을 시작하면 나오는 화면의 UI를 구성한다.
- 이 코드는 ScrollView 안에 VerticalStackLayout를 통해서 요소들을 수직으로 정렬하고 있다.
요소는 이미지, 2개의 텍스트를 보여주는 라벨과 버튼으로 이루어져 있다.
    - CounterBtn버튼이 Clicked 항목을 보면 OnCounterClicked 이벤트로 연결되어 있는데 이는 cs파일에서 확인할 수 있다.

```xml
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="MauiApp1.MainPage">

    <ScrollView>
        <VerticalStackLayout
            Padding="30,0"
            Spacing="25">
            <Image
                Source="dotnet_bot.png"
                HeightRequest="185"
                Aspect="AspectFit"
                SemanticProperties.Description="dot net bot in a hovercraft number nine" />

            <Label
                Text="Hello, World!"
                Style="{StaticResource Headline}"
                SemanticProperties.HeadingLevel="Level1" />

            <Label
                Text="Welcome to &#10;.NET Multi-platform App UI"
                Style="{StaticResource SubHeadline}"
                SemanticProperties.HeadingLevel="Level2"
                SemanticProperties.Description="Welcome to dot net Multi platform App U I" />

            <Button
                x:Name="CounterBtn"
                Text="Click me" 
                SemanticProperties.Hint="Counts the number of times you click"
                Clicked="OnCounterClicked"
                HorizontalOptions="Fill" />
        </VerticalStackLayout>
    </ScrollView>

</ContentPage>
```

### MainPage.xaml.cs

- MainPage.xaml 페이지의 동작 등을 정의한다.
- OnCounterClicked():
    - count 변수를 선언하여 CounterBtn버튼을 클릭하면 count값을 1씩 올려서 버튼의 텍스트를 변경한다.
    - SemanticScreenReader.Announce() 메소드는 운영 체제의 화면 읽기 프로그램을 통해 지정된 텍스트를 알려주는 기능을 가진 메소드다.

```csharp
public partial class MainPage : ContentPage
{
    int count = 0;

    public MainPage()
    {
        InitializeComponent();
    }

    private void OnCounterClicked(object? sender, EventArgs e)
    {
        count++;

        if (count == 1)
            CounterBtn.Text = $"Clicked {count} time";
        else
            CounterBtn.Text = $"Clicked {count} times";

        SemanticScreenReader.Announce(CounterBtn.Text);
    }
}
```

## MainProgram.cs

- 앱이 시작할 때 진입점이다. 종속성 주입을 위한 컨테이너 설정을 담당한다. 프로그램에서 사용할 다양한 서비스, 환경, 종속성 등을 여기서 설정한다.
- CreateBuilder(): 선택적 기본값으로 MauiAppBuilder 클래스의 새 인스턴스를 초기화 한다.
- .UseMauiApp<App>(): App 클래스(위에서 다룬 App.xaml과 App.xaml.cs)를 앱의 루트로 지정한다. App이 실행되기 전에 설정되어야 한다.
- ConfigureFonts(): 커스텀 폰트를 등록하고있다.
- builder.Build(): 앱 인스턴스를 생성한다. 이 코드까지 실지 실행되고 App클래스의 생성자가 실행된다.

```csharp
    public static class MauiProgram
    {
        public static MauiApp CreateMauiApp()
        {
            var builder = MauiApp.CreateBuilder();
            builder
                .UseMauiApp<App>()
                .ConfigureFonts(fonts =>
                {
                    fonts.AddFont("OpenSans-Regular.ttf", "OpenSansRegular");
                    fonts.AddFont("OpenSans-Semibold.ttf", "OpenSansSemibold");
                });

#if DEBUG
    		builder.Logging.AddDebug();
#endif

            return builder.Build();
        }
    }
```

- builder에 아래 코드같은 식으로 서드파티 라이브러리를 초기화 할 수 있다.
    - 이 코드는 UseMauiCommunityToolkit을 사용하여 옵션으로 윈도우에서 스낵바를 사용할 수 있도록 하는 코드이다.

```csharp
builder
    .UseMauiApp<App>()
    .UseMauiCommunityToolkit(options =>
    {
        options.SetShouldEnableSnackbarOnWindows(true);
    })
```

## Resources 폴더

- 폰트, 이미지 등의 리소스를 관리하는 폴더. 이 폴더에 이미지를 등록하고 xaml에서 호출하면 이미지가 보인다.

```xml
<Image
    Source="dotnet_bot.png" <!--이미지 이름-->
    HeightRequest="185"
    Aspect="AspectFit"
    SemanticProperties.Description="dot net bot in a hovercraft number nine" />
```

## Platforms 폴더

- 각 플랫폼 폴더는 MAUI 지원하는 각 플랫폼에서 앱을 시작하는 코드와 추가하는 추가 플랫폼 코드가 포함된다. 빌드 시 특정 플랫폼을 빌드할 때는 각 폴더의 코드만 포함된다.
- 각 플랫폼별 초기화가 끝나면 CreateMauiApp()을 오버라이드 하여 MauiProgram.CreateMauiApp()을 호출하여 앱을 시작한다.

# 실행

- .NET MAUI APP로 프로젝트를 만들고 실행하면 기본적으로 화면이 하나 나온다.
![Maui Start Project Execute Screen](/assets/Images/MauiStartProjectExecuteScreen.png)

- Click me라 적혀있는 버튼을 누르면 몇 번 눌렀는지 카운트가 올라간다.
![Click me Button Clicked](/assets/Images/mauiClickmeButtonClicked.png)

### 디버그 기능

- 핫 리로드
    - xaml을 수정하면 실행 중인 앱에서 변경된 화면을 바로 볼 수 있다.
    - 만약 화면이 업데이트가 잘 되지 않는 다면 핫 다시 리로드(Alt+F10) 눌러주면 된다.
    ![Hot reload](/assets/Images/mauiHotreload.png)
    

# 플랫폼별 배포와 디버그

- 윈도우
[Deploy and debug your .NET MAUI app on Windows](https://learn.microsoft.com/en-us/dotnet/maui/windows/setup?view=net-maui-9.0)
- 안드로이드
[Debug on the Android Emulator](https://learn.microsoft.com/en-us/dotnet/maui/android/emulator/debug-on-emulator?view=net-maui-9.0)
- iOS
[Remote iOS Simulator for Windows](https://learn.microsoft.com/en-us/dotnet/maui/ios/remote-simulator?view=net-maui-9.0)