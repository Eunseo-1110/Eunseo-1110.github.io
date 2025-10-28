---
title: "MAUI 소개"
date: 2025-10-28 00:00:00 +0900
categories: [MAUI]
tags: [MAUI]     # TAG names should always be lowercase
---
<details markdown="1">
<summary>관련 자료 보기</summary>

> 게임 개발을 하면서 늘어나는 데이터 테이블을 보기가 점점 어려워졌다. 그래서 '데이터를 눈에 보기 쉽게 하는 프로그램을 한 번 만들어보자'라는 생각을 하여, 어떤 프레임워크를 써볼까 고민이 되었다.
> 만드는데 오랜 시간을 들이고 싶지 않아서 C#언어가 지원되는 프레임워크를 사용하고 싶었다.  WPF나 MAUI 둘 중 고민하다가 자마린이라는 프레임워크가 기술 지원을 종료하며 MAUI로 바뀌었다는 소식을 찾고 궁금해져서 MAUI를 사용하게 되었다. 
>
> 출시된지 얼마 안된 프레임워크라 그런지 버그가 꽤 많다. 찾아보니 MAUI에 대한 평가는 별로 좋은 편은 아니였다. 그냥 React Native나 Flutter쓰는걸 추천하는 편.
> 
> 사용해보니까 사내 프로그램 같은거 개발할 때는 괜찮을 것 같다. 특히나 C#사용자가 많으면 빠르게 개발 가능 할 것 같다.
>
> 그리고 다양한 플랫폼을 지원한다는 점이 플러스다. iOS지원이 별로라고는 하지만 안하는 것 보단 좋겠지..
> 기왕 사용해봤으니 사용해본 기능 위주로 포스트를 해볼 생각이다.
>
> 이번에 포스트하려고 조사해보니 BlazorBindings.MAUI를 꽤나 사용하는 것 같다.
> [BlazorBindings MAUI](https://github.com/Dreamescaper/BlazorBindings.Maui)

</details>

[Microsoft .NET MAUI란?](https://learn.microsoft.com/ko-kr/dotnet/maui/what-is-maui?view=net-maui-9.0) 

[.NET MAUI](https://dotnet.microsoft.com/ko-kr/apps/maui)

# 개요
## MAUI란?
- maui는 .Net **M**ulti-Platform **A**pp **UI**를 줄인 말로 네이티브 모바일 앱과 데스크톱 앱을 개발하기 위한 프레임워크다.
- 사용언어: C#, XAML
- 지원하는 플랫폼: iOS, Android, macOS, Windows

![MAUI Platform](https://learn.microsoft.com/ko-kr/dotnet/maui/media/what-is-maui/maui-overview.png?view=net-maui-9.0.png)


> iOS 및 macOS용 앱을 빌드하려면 Mac이 필요합니다.
{: .prompt-info }

### .NET MAUI 아키텍처 다이어그램
![MAUI Architecture Diagram](https://learn.microsoft.com/ko-kr/dotnet/maui/media/what-is-maui/architecture-diagram.png?view=net-maui-9.0.png)

# 다운로드
1. Visual Studio 2022 설치
2. 데스크톱 및 모바일을 보면 ‘.NET Multi-Platform App UI 개발’이 있다 선택하여 설치하면 다운로드는 끝난다.
![Install MAUI](/assets/Images/InstallMAUI.png)

### 프로젝트
1. 프로젝트 타입에서 MAUI를 설정하면 MAUI 프로젝트들이 나온다.
![Project Type MAUI](/assets/Images/ProjectTypeMAUI.png)