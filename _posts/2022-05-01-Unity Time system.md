---
title: "유니티 시간 구현하기"
date: 2022-05-01 00:00:00 +0900
categories: [Unity]
tags: [c#, unity]     # TAG names should always be lowercase
---

<span style="color:grey">*이 포스트는 이전 블로그에서 옮겨왔습니다.*</span>  

전체 코드

TimeManager.cs
```cs
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
 
public class TimeManager : SingletonMono<TimeManager>
{
    public float Multiple = 60f;       // 배속
    public bool IsPm;       // AM,PM확인
    public int Days = 1;        // 날짜
 
    // 확인용 유아이
    [SerializeField]
    Image image;    
    [SerializeField]
    Text text;
 
    DelayTimer Timer;       // 타이머
    float hr_12;       // 12시간
    float hr_24;         // 24시간
 
    bool isInit;
 
    protected override void SingletonInit()
    {
        base.SingletonInit();
        Init();
    }
 
    void Awake()
    {
        Init();
    }
 
    void Init()
    {
        if (isInit)
            return;
 
        isInit = true;
        Timer = new DelayTimer();
        // 12시간 초 계산
        hr_12 = 3600 * 12;
        hr_24 = hr_12 * 2;
        Timer.Delay = hr_24; 
        Timer.Multiple = Multiple; //배율
    }
 
    void Update()
    {
        IsPm = Timer.Timer > hr_12 ? true : false;
        if (Timer.UpdateTimer())
        {
            ++Days;
        }
 
        // 유아이 세팅
        SetTimerUI();
    }
 
    void SetTimerUI()
    {
        image.fillAmount = 1.0f - (Timer.Timer / (hr_24));
 
        string am;
        //AM,PM텍스트
        if (!IsPm)
            am = "AM";
        else
            am = "PM";
 
        int min = (int)(Timer.Timer / 60f); // 분
        int hours = min / 60;   // 시
        int sec = (int)(Timer.Timer % 60);  // 초
        min %= 60;
 
        text.text = Days + "일" + am + " " + hours + ":" + min + ":" + sec;
 
    }
 
    public void AddDay(int day)
    {
        Days += day;
    }
 
    // 시
    public void AddHour(int hr)
    {
        float add = hr * 3600;
        AddTime(add);
    }
 
    // 분
    public void AddMinute(int min)
    {
        float add = min * 60;
        AddTime(add);
    }
    // 초
    public void AddSecond(int sec)
    {
        float add = sec;
        AddTime(add);
    }
 
    // 시간 추가
    void AddTime(float add)
    {
        int days = Timer.SetTimer(add);    //타이머에 추가
        IsPm = Timer.Timer  > hr_12 ? true : false;
        if(days > 0)
        {
            Days += days;
        }
 
        //타이머 UI
        SetTimerUI();
    }
}
```
DelayTimer.cs
```cs
public class DelayTimer
{
    public float Delay;     
    public float Multiple = 1;
    public bool IsStop;
    public float Timer { get ; private set; }
 
    public bool UpdateTimer()
    {
        Timer += (Time.deltaTime * Multiple);
 
 
        if (!IsStop && Timer > Delay)
        {
            Timer = 0;
            return true;
        }
        return false;
    }
 
    public int SetTimer(float sec)
    {
        Timer += sec;
        int rest = (int)(Timer / Delay);
        Timer -= (Delay * rest);
        return rest;
    }
 
    public bool IsDelayTime()
    {
        if (Timer > Delay)
        {
            return true;
        }
        return false;
    }
}
```
SingletonMono.cs
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
 
 
public class SingletonMono<T> : MonoBehaviour where T: MonoBehaviour
{
    protected static T m_instance = null;
 
    protected virtual void SingletonInit() { }
 
 
    public static T Instance
    {
        get
        {
            if (m_instance == null)
            {
                m_instance = FindObjectOfType(typeof(T)) as T;
                SingletonMono<T> tempinst = m_instance as SingletonMono<T>;
                tempinst.SingletonInit();
                if (m_instance == null)
                {
                    Debug.LogError(typeof(T) + "is none.");
                }
            }
 
            return m_instance;
 
        }
    }
}
```
전에 만든 딜레이 타이머 클래스를 이용해서 유니티로 시간을 구현해봤다.  
시간이라면 이곳저곳 불러올 곳이 많을 듯 하여 싱글톤으로 구현했다.  

업데이트에서 24시간이 지나면 타이머는 초기화되고 다음 날짜로 넘어간다.  
인스펙터 창에서 시간 배율을 받아 타이머를 가속할 수 있다.

![a](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/TimeManager_Img1.png)  

시간 추가 함수를 시,분,초 단위로 각각 만들어서 함수 안에서 초를 계산하여 시간이 더해지도록 만들었다.  
UI는 간단하게 확인을 하기 위해서 만들었다.  

![b](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/TimeManager_Img2.png)  
이런 식으로 보인다.
  
유니티의 Time.deltaTime은 프레임 단위로 계산이 되는 거라 정확한 시간으로 사용할 순 없을 듯 하다.