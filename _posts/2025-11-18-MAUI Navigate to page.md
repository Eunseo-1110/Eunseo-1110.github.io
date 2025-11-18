---
title: "MAUI 페이지 이동"
date: 2025-11-18 00:00:00 +0900
categories: [MAUI]
tags: [MAUI]     # TAG names should always be lowercase
---
<details markdown="1">
<summary>참고 링크</summary>

> [NavigationPage-.Net MAUI](https://learn.microsoft.com/ko-kr/dotnet/maui/user-interface/pages/navigationpage?view=net-maui-10.0)  
> [Shell 탐색-.Net MAUI](https://learn.microsoft.com/ko-kr/dotnet/maui/fundamentals/shell/navigation?view=net-maui-10.0)  
> [Shell 페이지-.Net MAUI](https://learn.microsoft.com/ko-kr/dotnet/maui/fundamentals/shell/pages?view=net-maui-10.0)  
> [Shell vs Navigation Page in .Net MAUI](https://www.c-sharpcorner.com/blogs/shell-vs-navigationpage-in-net-maui)  
> [MAUI: What is the difference between GoToAsync() and PushAsync() to navigate between pages?](https://stackoverflow.com/questions/75022413/maui-what-is-the-difference-between-gotoasync-and-pushasync-to-navigate-bet)
    
</details>

# 새로운 페이지 생성

- 프로젝트 우클릭 → 추가 → 새 항목 → .NET MAUI 파일 생성을 해서 새로운 페이지를 만들 수 있다.

![New file Create MAUI](/assets/Images/New_file_Create_MAUI.png)

# Shell

## Shell.Current.GoToAsync()

- **URI 기반 네비게이션**을 제공하는 메서드. 해당 메서드를 통해서 페이지 간 이동을 간단하고 직관적으로 처리할 수 있다.

## 경로등록

- 라우트를 등록해야 페이지 이동을 할 수 있다.
- xaml: `Route` 속성

```xml
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

	<ShellContent 
        Title="NewPage1"
				Route = "NewPage1" 
        ContentTemplate="{DataTemplate local:NewPage1}"/>

</Shell>
```

- cs: `Routing.RegisterRoute` 함수

```csharp
public partial class AppShell : Shell
{
    public AppShell()
    {
        InitializeComponent();

        Routing.RegisterRoute("NewPage1", typeof(NewPage1));
    }
}
```

## 페이지 이동

```csharp
// 절대경로
Shell.Current.GoToAsync("//NewPage1");

// 상대경로 Routing.RegisterRoute함수를 통해 등록한 페이지 
Shell.Current.GoToAsync("NewPage1");
```

## 데이터 전달

- 전달

```csharp
Shell.Current.GoToAsync($"NewPage1?id=123&str=abc");
```

- 받기

```csharp
[QueryProperty(nameof(ID), "id")]
[QueryProperty(nameof(Str), "str")]
public partial class NewPage1 : ContentPage
{
	private int _id;
	private string _str;

	public int ID
	{
		get => _id;
		set
		{
			_id = value;
			LoadData();
		}
	}

	public string Str
	{
		get => _str;
		set
		{
			_str = value;
			LoadData();
		}
	}

	void LoadData()
	{
		label1.Text = $"ID: {ID}, Str:{Str}";
	}
}
```

- 결과
    ![Pass Data GoToAsync](/assets/Images/Pass_Data_GoToAsync.png)
    

### 메소드 정리

| 패턴 | 코드 | 설명 |
| --- | --- | --- |
| **기본 이동** | `GoToAsync("details")` | 앞으로 이동 |
| **뒤로 가기** | `GoToAsync("..")` | 한 단계 뒤로 |
| **절대 경로** | `GoToAsync("//main")` | 루트부터 시작 |
| **파라미터** | `GoToAsync("details?id=1")` | 쿼리 스트링 전달 |
| **애니메이션** | `GoToAsync("details", false)` | 애니메이션 제어 |

# NavigationPage

## **모덜리스 탐색 수행**

- .NET MAUI는 모덜리스 페이지 탐색을 지원한다. 모덜리스 페이지는 화면에 유지되며 다른 페이지로 이동할 때까지 계속 사용할 수 있다.
- BackButton을 누르면 이전 화면으로 돌아갈 수 있다.

![페이지를 탐색 스택으로 푸시합니다.](https://learn.microsoft.com/ko-kr/dotnet/maui/user-interface/pages/media/navigationpage/pushing.png?view=net-maui-10.0.png)

![탐색 스택에서 페이지를 표시합니다.](https://learn.microsoft.com/ko-kr/dotnet/maui/user-interface/pages/media/navigationpage/popping.png?view=net-maui-10.0.png)

![NavigationPage 구성 요소.](https://learn.microsoft.com/ko-kr/dotnet/maui/user-interface/pages/media/navigationpage/components.png?view=net-maui-10.0.png)

## 페이지로 이동

```csharp
await Navigation.PushAsync(new NewPage1());
```

```csharp
await Navigation.PopModalAsync();
```

## 데이터 전달

- 생성자를 통해서 데이터를 전달할 수 있다.
    - 이동하려는 페이지에 string데이터를 전달하여 라벨에 출력하도록 하는 코드
    
    ```csharp
    Navigation.PushAsync(new NewPage1("abc"));
    ```
    
    ```csharp
    public NewPage1(string str)
    {
        InitializeComponent();

        label1.Text = str;
    }
    ```
    
    ![Pass data during navigation](/assets/Images/Pass_data_during_navigation.png)

# GoToAsync vs PushAsync

| **특징** | **Shell.Current.GoToAsync** | **Navigation.PushAsync** |
| --- | --- | --- |
| **기반** | Shell + URI 라우팅 | NavigationPage 스택 |
| **이동 방식** | **URI 문자열** (예: `"//details"`) | **페이지 객체** (예: `new DetailPage()`) |
| **데이터 전달** | 쿼리 파라미터 (`?id=10`) 및 `QueryProperty` 사용 | 생성자 주입 또는 `BindingContext` 직접 할당 |
| **결합도** | **느슨한 결합** (페이지 타입을 몰라도 됨) | **강한 결합** (페이지 클래스를 직접 참조해야 함) |
| **딥 링크** | 기본 지원 | 직접 구현 필요 |