---
title: "C# 명명법"
date: 2022-04-20 00:00:00 +0900
categories: [C#]
tags: [c#]     # TAG names should always be lowercase
---

<span style="color:grey">*이 포스트는 이전 블로그에서 옮겨왔습니다.*</span>
 

파스칼: 첫 번째 문자가 대문자 시작하여 단어마다 대문자로 시작. ex) IsMove  
카멜: 첫 번째 문자가 소문자로 시작하여 단어마다 대문자로 시작. ex) isMove 

class, record, struct: 파스칼  
interface: I+파스칼  
public 필드, 속성, 이벤트, 메서드, 로컬 함수: 파스칼  
internal, private필드: 카멜 private, internal인 static필드: s카멜  
private, internal인 thread static 필드: t_카멜  
메소드 매개변수: 카멜  
주석: 별도의 행, 대문자 시작, 마침표 종료, // 텍스트  