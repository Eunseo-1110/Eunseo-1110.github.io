---
title: "유니티 패럴랙스/배경움직이기"
date: 2022-05-22 00:00:00 +0900
categories: [Unity]
tags: [c#, unity]     # TAG names should always be lowercase
---

<span style="color:grey">*이 포스트는 이전 블로그에서 옮겨왔습니다.*</span>  

```cs
using System.Collections.Generic;
using UnityEngine;
 
public class Parallaxing : MonoBehaviour
{
    [Header("이동할 배경들")]
    public Transform[] Background;
    [Header("속도")]
    public float Speed;
    [Header("이동 좌표")]
    public bool x;
    public bool y;
 
    [SerializeField]
    Vector2 MaxPos;
    [SerializeField]
    Vector2 MinPos;
 
    void Awake()
    {
        // 정렬
        List<Transform> list = new List<Transform>();
        list.AddRange(Background);
        list.Sort((a, b) => a.position.x > b.position.x ? 1 : -1);
        Background = list.ToArray();
    }
 
    void Update()
    {
        float hor = Input.GetAxisRaw("Horizontal");
        float ver = Input.GetAxisRaw("Vertical");
 
        // 움직임 확인
        if (hor != 0 || ver != 0)
        {
            Vector2 tempVec = new Vector2(hor, ver).normalized;
            Vector3 moveVec = tempVec;
            // 플레이어 이동 벡터 * 속도 * 델타타임
            moveVec *= Speed * Time.deltaTime;
            moveVec.z = 0f;
 
            // 배경들 이동
            foreach (Transform item in Background)
            {
                // 최대,최소 위치 확인
                if (MaxPos.x != 0 && item.position.x + moveVec.x > MaxPos.x)
                    continue;
                if (MinPos.x != 0 && item.position.x + moveVec.x < MinPos.x)
                    continue;
 
                if (MaxPos.y != 0 && item.position.y + moveVec.y > MaxPos.y)
                    continue;
                if (MinPos.y != 0 && item.position.y + moveVec.y < MinPos.y)
                    continue;
 
                // y,z 위치 값 0
                if (!x)
                    moveVec.x = 0f;
                if (!y)
                    moveVec.y = 0f;
 
                // 이동
                item.position += moveVec;
            }
        }
    }
}
```

![Parallax preview](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/Parallax%20preview.gif)  

게임을 하다보면 입체감을 위해 배경이 움직이는 경우가 있다. 런게임같은 배경이 단순한 게임에서는 캐릭터는 이동하지 않고 가만히 두고 배경만 움직이는 방식으로 느낌을 준다.

간단하게 배경움직이기, 패럴랙스를 만들어보았다.

![Parallax inspactor](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/Parallax%20inspactor.png)  
패럴랙스 인스펙터 창

이동할 배경을 배열로 추가할 수 있게 하였고, 속도를 지정한다.
이동 좌표는 x,y축을 bool로 만들어서 이동하고 싶은 좌표축을 선택할 수 있도록 만들었다.  
Max,Min Pos는 최대 최소 위치인데 해당 위치보다 이동할 위치가 크거나 작으면 이동하지않는다.  