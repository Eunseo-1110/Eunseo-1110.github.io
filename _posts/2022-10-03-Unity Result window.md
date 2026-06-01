---
title: "[유니티] 결과창 만들기"
date: 2022-10-03 00:00:00 +0900
categories: [Unity]
tags: [c#, unity]     # TAG names should always be lowercase
---

<span style="color:grey">*이 포스트는 이전 블로그에서 옮겨왔습니다.*</span>  

[GitHub 링크](https://github.com/Eunseo-1110/ResultWindow.git)

게임에서 흔히 나오는 결과창을 만들어 볼 것이다.  
결과창은 ‘이름 * 개수’ 내용을 줄마다 출력할 생각이다. 개수는 카운팅 효과를 넣어서 만들 생각

## 1. UI 만들기

UI는 결과창 타이틀 텍스트와 내용 텍스트, X버튼으로 이루어져 있다.  

![Result UI object structure](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/Result%20UI%20object%20structure.png)  
UI 오브젝트 구조

![Result UI Scene](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/Result%20UI%20Scene.png)  
씬뷰

## 2. 스크립트 생성

먼저 결과창에 출력할 아이템의 이름과 개수로 이루어진 클래스를 하나 만들 것이다.  

```cs
// 아이템 정보 클래스
public class ItemInfo
{
    public string name; // 이름
    public int count;   // 갯수
 
    public ItemInfo(string name, int count)
    {
        this.name = name; 
        this.count = count;
    }
}
```
그다음엔 결과창 모노비헤이비어 클래스를 작성한다.  
결과창에는 내용 텍스트를 SerializeField를 이용하여 참조하는 변수가 하나 있고, 아이템 정보 배열과, bool 변수 등을 가지고 있다.  
```cs
[SerializeField]
Text contentText;       // 내용 텍스트
 
ItemInfo[] itemInfoArr;   // 아이템 정보 배열
 
bool isResultText;  // 결과창 텍스트 출력 중 확인
bool isCounting;    // 텍스트 카운팅 확인
```
카운팅 함수를 만들어 준다. 카운팅 함수는 https://unitys.tistory.com/7 이 블로그의 코드를 참고하여 만들 것이다.  
```cs
// https://unitys.tistory.com/7
/// <summary>
/// 텍스트 카운팅
/// </summary>
/// <param name="text">텍스트 UI</param>
/// <param name="target">목표 수</param>
/// <param name="current">시작 수</param>
/// <returns></returns>
IEnumerator Count(Text text, float target, float current)
{
    float duration = 0.5f; // 카운팅에 걸리는 시간 설정. 
    float offset = (target - current) / duration;
 
    while (current < target)
    {
        current += offset * Time.deltaTime;
        text.text = ((int)current).ToString();
        yield return null;
    }
 
    current = target;
    text.text = ((int)current).ToString();
}
```
결과창에 정보를 세팅할 수 있는 함수를 만들어준다.
```cs
/// <summary>
/// 결과창 세팅
/// </summary>
/// <param name="itemInfoArr">출력할 아이템 정보</param>
public void SetResult(ItemInfo[] itemInfoArr)
{
    // 텍스트 초기화
    contentText.text = "";
 
    this.itemInfoArr = itemInfoArr; // 아이템 정보 배열
    isResultText = true;        // 결과창 텍스트 출력 중
 
    // 첫 텍스트 입력
    contentText.text = itemInfoArr[0].name + " * ";
 
    // 카운팅
    StartCoroutine(Count(contentText, itemInfoArr[0].count, 0));
}
```
그리고 간단하게 테스트 할 수 있게 Start함수에 테스트용 코드를 넣어줬다.
```cs
void Start()
{
    // 테스트용
    ItemInfo[] iteminfo = {
        new ItemInfo("사과", 5),
        new ItemInfo("복숭아", 4),
        new ItemInfo("수박", 3)
    };
 
    SetResult(iteminfo);
}
```
이제 작성한 스크립트를 UI에 컴포넌트를 추가한다.  
이러면 SerializeField를 이용한 변수가 인스펙터 창에서 보인다. 이 변수에 내용텍 스트를 등록.

![Added compoment in Result UI](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/Added%20compoment%20in%20Result%20UI.png)  
결과창 UI에 컴포넌트 등록한 모습

이제 실행하면 된다!  
그러면 문제가 하나 생기는데 바로 이름이 보이지 않고 개수만 카운팅 된다.

## 3. 문제해결
![Invisible name issue](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/Invisible%20name%20issue.png)  

이유는 카운팅 함수에서 텍스트에 카운팅한 수를 바로 입력하기 때문이다.

```cs
text.text = ((int)current).ToString();
```
그럼 =을 +=로 바꾸면 될까? +=로 바꾸면 출력하는 텍스트가 001111222이런식으로 텍스트가 출력된다.  

어떻게 카운팅이 기존 텍스트를 변경하지 않고 카운팅할 수 있을까  
여기선 string 변수를 만들어 이 변수에 카운팅 수를 문자로 가진다. 이 변수를 사용하여 입력하기 전에 이전에 입력된 문자는 삭제하고 새로 카운팅된 문자를 텍스트에 추가한다.

```cs
// https://unitys.tistory.com/7
/// <summary>
/// 텍스트 카운팅
/// </summary>
/// <param name="text">텍스트 UI</param>
/// <param name="target">목표 수</param>
/// <param name="current">시작 수</param>
/// <returns></returns>
IEnumerator Count(Text text, float target, float current)
{
    isCounting = true;   // 텍스트 카운팅 시작
 
    float duration = 0.5f; // 카운팅에 걸리는 시간 설정. 
    float offset = (target - current) / duration;
    string countStr = "";    // 카운트 텍스트
 
    // 현재 수가 목표 수보다 커질 때까지 반복
    while (current < target)
    {
        // 현재 텍스트에서 입력된 카운트 텍스트 만큼 문자 삭제
        text.text = text.text.Remove(text.text.Length - countStr.Length, countStr.Length);
 
        current += offset * Time.deltaTime;
        countStr = ((int)current).ToString();       // 카운트 텍스트 세팅
 
        text.text += countStr;  // 텍스트에 카운트 텍스트 추가
        yield return null;
    }
 
    // 현재 텍스트에서 입력된 카운트 텍스트 만큼 문자 삭제
    text.text = text.text.Remove(text.text.Length - countStr.Length, countStr.Length);
 
    // 목표 수 입력
    current = target;
    text.text += ((int)current).ToString();
 
    isCounting = false;  // 텍스트 카운팅 종료
}
```
함수를 위와 코드로 바꿔 실행해보면 문제없이 실행된다.  

![Resolved Invisible name issue](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/Resolved%20Invisible%20name%20issue.png)  
사과가 유지된 채로 카운팅된 모습

그럼 이제 이걸 반복으로 출력되게 해보자  

## 4. 반복

업데이트 함수에서 텍스트를 출력 중인지 확인한다. 이전에 만들어 뒀던 bool isResultText 변수를 사용하여 확인할 것이다.

```cs
void Update()
{
    // 텍스트 출력 확인
    if (isResultText)
    {
        // 내용 입력
    }
}
```

저 사이에 카운팅 함수 종료 확인하여 배열의 인덱스를 증가 시키고, 인덱스를 확인하여 반복 종료, 다시 반복하는 함수를 구현할 것이다.  

카운팅 종료 확인과 배열 인덱스 변수가 필요하다.  
```cs
bool isEndCounting; // 카운팅 종료 확인
 
int itemInfoIndex;   // 아이템 정보 인덱스
```
카운팅 함수 마지막에 아래 코드 추가
```cs
isEndCounting = true;   // 카운팅 종료 확인
```
SetResult()함수 마지막에 인덱스를 초기화 해주는 코드 추가
```cs
itemInfoIndex = 0; // 아이템 정보 인덱스
```  


이제 업데이트에서 반복을 구현할 것이다.  

우선 카운팅 중인지 확인해서 텍스트를 입력하고 카운팅하는 부분 추가  
```cs
// 카운팅 중인지 확인
if (!isCounting)
{
    // 텍스트 입력
    contentText.text += itemInfoArr[itemInfoIndex].name + " * ";
 
    // 카운팅
    StartCoroutine(Count(contentText, 0, itemInfoArr[itemInfoIndex].count, countingDuration));
}
```
카운팅 종료 확인해서 인덱스 증가하는 부분 추가  
```cs
// 카운팅 종료 확인
if (isEndCounting)
{
    ++itemInfoIndex;        // 인덱스 증가
    contentText.text += "\n";   // 텍스트 줄넘김
    isEndCounting = false;         // 카운팅 종료 false
}
```
인덱스 확인하고 종료하는 부분 추가
```cs
// 인덱스 확인
if (itemInfoIndex >= itemInfoArr.Length)
{
    // 초기화
    isResultText = false;
    isCounting = false;
    isEndCounting = false;
 
    itemInfoArr = null;
    return;
}
```
다됐다. Update함수의 전체적인 코드는 아래와 같다.
```cs
void Update()
{
    // 텍스트 출력 확인
    if (isResultText)
    {
        // 카운팅 종료 확인
        if (isEndCounting)
        {
            ++itemInfoIndex;        // 인덱스 증가
            contentText.text += "\n";   // 텍스트 줄넘김
            isEndCounting = false;         // 카운팅 종료 false
        }
 
        // 인덱스 확인
        if (itemInfoIndex >= itemInfoArr.Length)
        {
            // 초기화
            isResultText = false;
            isCounting = false;
            isEndCounting = false;
 
            itemInfoArr = null;
            return;
        }
 
        // 카운팅 중인지 확인
        if (!isCounting)
        {
            // 텍스트 입력
            contentText.text += itemInfoArr[itemInfoIndex].name + " * ";
 
            // 카운팅
            StartCoroutine(Count(contentText, 0, itemInfoArr[itemInfoIndex].count, countingDuration));
        }
 
    }
}
```
실행해보면 정상적으로 반복된다.  

![Iterated result array](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/Iterated%20result%20array.png)  
배열이 모두 반복된 모습

업데이트 함수를 코루틴으로도 만들어 보았다. 아래 코루틴 함수는 업데이트와 똑같으며 SetResult()함수의 마지막 부분에 StartCoroutine을 사용하여 입력하면 된다.

```cs
IEnumerator ShowCounting()
{
    int inx = 0;        // 배열 인덱스
    while (true)
    {
        // 텍스트 출력 확인
        if (isResultText)
        {
            // 카운팅 종료 확인
            if (isEndCounting)
            {
                ++inx;        // 인덱스 증가
                contentText.text += "\n";   // 텍스트 줄넘김
                isEndCounting = false;         // 카운팅 종료 false
            }
 
            // 인덱스 확인
            if (inx >= itemInfoArr.Length)
            {
                // 초기화
                isResultText = false;
                isCounting = false;
                isEndCounting = false;
 
                itemInfoArr = null;
                break;
            }
            else
            {
                // 카운팅 중인지 확인
                if (!isCounting)
                {
                    // 텍스트 입력
                    contentText.text += itemInfoArr[inx].name + " * ";
 
                    // 카운팅
                    StartCoroutine(Count(contentText, 0, itemInfoArr[inx].count, 0.3f));
                }
            }
 
        }
        yield return null;
    }
}
```
그리고 카운팅 함수의 카운팅 시간이 함수 안에 있길래 매개변수로 빼고 매개변수 이름을 조금 수정했다. 카운팅 시간은 SerializeField를 이용하여 인스펙터 창에서 설정할 수 있도록 변경하였다.
```cs
[SerializeField]
float countingDuration = 0.3f;
 
IEnumerator Count(Text text, float start, float end, float duration)
{
    isCounting = true;   // 텍스트 카운팅 시작
 
    float offset = (end - start) / duration;
    string countStr = "";    // 카운트 텍스트
 
    // 현재 수가 목표 수보다 커질 때까지 반복
    while (start < end)
    {
        // 현재 텍스트에서 입력된 카운트 텍스트 만큼 문자 삭제
        text.text = text.text.Remove(text.text.Length - countStr.Length, countStr.Length);
 
        start += offset * Time.deltaTime;
        countStr = ((int)start).ToString();       // 카운트 텍스트 세팅
 
        text.text += countStr;  // 텍스트에 카운트 텍스트 추가
        yield return null;
    }
 
    // 현재 텍스트에서 입력된 카운트 텍스트 만큼 문자 삭제
    text.text = text.text.Remove(text.text.Length - countStr.Length, countStr.Length);
 
    // 목표 수 입력
    start = end;
    text.text += ((int)start).ToString();
 
    isCounting = false;  // 텍스트 카운팅 종료
    isEndCounting = true;   // 카운팅 종료 확인
}
```
이제 X버튼을 만들어 볼 것이다. 아주 간단하다!

## 5. X버튼

X버튼은 결과창이 출력되는 동안 비활성화 되었다가 출력이 끝난 시점에서 활성화되게 만들 것이다.  

일단 스크립트에서 제어해야하니 SerializeField를 통해 X버튼을 참조할 수 있는 변수 추가  
```cs
[SerializeField]
Button buttonX;       // X버튼
```

SetResult()함수의 마지막 부분에서 오브젝트 비활성화 코드 추가
```cs
buttonX.gameObject.SetActive(false);    // X버튼 오브젝트 비활성화
```
업데이트 함수 인덱스를 확인하여 종료하는 부분에서 버튼 오브젝트 활성화하는 코드 추가
```cs
buttonX.gameObject.SetActive(true);    // X버튼 오브젝트 활성화
```
그리고 인스펙터 창에서 설정을 해준다.

X버튼의 OnClick()에서 결과창을 등록해주고 SetActive를 false로 설정해준다.

![XButton Inspector](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/XButton%20Inspector.png)  

결과창 스크립트 컴포넌트에 버튼을 등록해주면 끝난다.

![Set XButton in Result Panel component](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/Set%20XButton%20in%20Result%20Panel%20component.png)  

X버튼이 끝났다! 결과창이 출력되는 동안 X버튼이 안보이다가 끝나면 보이고, X버튼을 누르면 결과창이 사라지는 것 까지 확인할 수 있다.

결과창을 천천히 감상할 수도 있지만 빠르게 다음으로 넘어가고 싶은 사람도 있다. 스킵 기능을 만들어보자.

## 6. 스킵

스킵또한 간단하게 끝난다. 일단 카운팅 코루틴을 멈춰야하기 때문에 변수를 만들어 준다.

```cs
Coroutine coutingCoroutine;       // 카운팅 코루틴
```
이제 카운팅 코루틴을 사용하는 부분에 가서 변수에 코루틴을 등록 해줘야 한다. 업데이트 함수의 카운팅을 실행하는 부분에 가서 코드를 바꿔주자.
```cs
// 카운팅
coutingCoroutine = StartCoroutine(Count(contentText, 0, itemInfoArr[itemInfoIndex].count, countingDuration));
```
스킵 함수를 만들기 전에 자주 사용하는 텍스트 출력 종료 시 초기화 하는 부분을 함수로 만들어 주자
```cs
/// <summary>
/// 결과창 텍스트 출력 종료
/// </summary>
void ShowEnd()
{
    // 초기화
    isResultText = false;
    isCounting = false;
    isEndCounting = false;
 
    itemInfoArr = null;
 
    buttonX.gameObject.SetActive(true);    // X버튼 오브젝트 활성화
}
```
위 코드를 사용하는 부분은 함수로 대체해주면 된다.  

스킵함수를 만들어 보자.  
결과창에 출력할 string변수를 만들어서 입력해주고, 코루틴을 종료, 위에서 만든 텍스트 출력 종료함수를 이용해주고 텍스트에 변수를 입력해주면 끝이다.  
```cs
/// <summary>
/// 결과창 텍스트 출력 스킵
/// </summary>
public void _OnSkip()
{
    string reulstStr = "";      // 결과창 텍스트
    for (int i = 0; i < itemInfoArr.Length; i++)
 
    {
        // 텍스트 입력
        reulstStr += itemInfoArr[i].name + " * " + itemInfoArr[i].count + "\n";
    }
 
    StopCoroutine(coutingCoroutine);        // 카운팅 코루틴 종료
    ShowEnd();
 
    contentText.text = reulstStr;   // 결과창 텍스트 세팅
}
```
스크립트는 끝났다. 유니티로 이동해서 몇 가지 설정을 하면 끝난다.  

결과창을 클릭하면 스킵이 되도록 만들 것이다. 결과창에 버튼 컴포넌트를 추가해주고 스킵 함수를 등록한다.  

![Skip button Component](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/Skip%20button%20Component.png)  

타이틀과 내용 텍스트는 클릭이 되지 않도록 RaycastTarget을 꺼주자

![Set RaycastTarget on TitleNContent Text](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/Set%20RaycastTarget%20on%20TitleNContent%20Text.png)

기능을 모두 구현했다!  

블로그에 글을 쓰다보니 업데이트 함수의 반복 부분을 줄일 수 있을 것 같아서 줄여보겠다.  

## 7. 코드최적화

우선 SetResult()함수에 텍스트 입력과 카운팅 부분을 추가한다.

```cs
public void SetResult(ItemInfo[] itemInfoArr)
{
    // 텍스트 초기화
    contentText.text = "";
 
    this.itemInfoArr = itemInfoArr; // 아이템 정보 배열
    isResultText = true;        // 결과창 텍스트 출력 중
 
    itemInfoIndex = 0;      // 아이템 정보 인덱스
 
    // 텍스트 입력
    contentText.text += itemInfoArr[itemInfoIndex].name + " * ";
    // 카운팅
    coutingCoroutine = StartCoroutine(Count(contentText, 0, itemInfoArr[itemInfoIndex].count, countingDuration));
 
 
    buttonX.gameObject.SetActive(false);    // X버튼 오브젝트 비활성화
}
```
그리고 업데이트 함수로 간다.  
인덱스를 확인하는 부분과 카운팅 중인지 확인하고 텍스트를 입력하는 부분을 나눠 놨었는데, 이 부분을 카운팅 종료를 확인하는 부분에서 같이 실행한다.  

```cs
void Update()
{
    // 텍스트 출력 확인
    if (isResultText)
    {
        // 카운팅 종료 확인
        if (isEndCounting)
        {
            ++itemInfoIndex;        // 인덱스 증가
            isEndCounting = false;         // 카운팅 종료 false
 
            // 인덱스 확인(반복출력 종료 확인)
            if (itemInfoIndex >= itemInfoArr.Length)
            {
                ShowEnd();
                return;
            }
            else
            {
                contentText.text += "\n";   // 텍스트 줄넘김
                // 텍스트 입력
                contentText.text += itemInfoArr[itemInfoIndex].name + " * ";
 
                // 카운팅
                coutingCoroutine = StartCoroutine(Count(contentText, 0, itemInfoArr[itemInfoIndex].count, countingDuration));
            }
        }
    }
}
```

카운팅 종료 확인 부분에서 인덱스를 증가하고 바로 인덱스를 확인하여 종료할 수 있다. 종료되지 않는다면 다음 텍스트로 넘어간다.  
이제 isCounting 변수는 필요가 없다. 지워도 된다.  

결과창 만들기가 끝났다!  
이전에는 구현된 글을 올리는 방식으로 했었는데 이번엔 차근히 만드는 과정을 올려보았다. 읽는 입장에선 이쪽이 더 재밌게 읽을 듯하여 이런 방식도 도전했는데 괜찮은 것 같다.  