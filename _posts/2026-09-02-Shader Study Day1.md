---
title: "Hello World!~Shaping fuctions"
date: 2026-09-02 00:00:00 +0900
categories: [Shader, The book of shaders]
tags: [shader, study]     # TAG names should always be lowercase
---

우선은 The book of shaders를 살펴보기로 하였다. 웹페이지로 되어있어서 접근성이 용이하고 예제에서 바로바로 코드를 고쳐서 결과를 눈으로 볼 수 있다.

여기서 나오는 예제들은 GLSL로 쓰여있다. 만약 HLSL을 쓴다면 다른 문법이 있다. 참고해서 보자.  

| GLSL                     | HLSL                                    |
| ------------------------ | --------------------------------------- |
| `vec2` / `vec3` / `vec4` | `float2` / `float3` / `float4`          |
| `mix(a, b, t)`           | `lerp(a, b, t)`                         |
| `fract(x)`               | `frac(x)`                               |
| `mod(x, y)`              | `fmod(x, y)` — **음수 처리가 다릅니다** |
| `clamp(x, 0.0, 1.0)`     | `saturate(x)`                           |
| `atan(y, x)`             | `atan2(y, x)`                           |
| `texture2D(tex, uv)`     | `tex.Sample(sampler, uv)`               |
| `mat2` / `mat3`          | `float2x2` / `float3x3`                 |
  
---
## Hello world!

이론적인 부분을 설명하고 난 뒤 HellowWorld를 보면 여기부터 쉐이더 코드가 있다. 웹페이지에서 바로바로 코드를 바꿔볼 수 있다.

![Hello world exam magenta](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/Hello world magenta.png)

컬러는 rgba로 되어있으니까 민트색 만들어보기

![Hello world exam mint](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/Hello world mint.png)

- 전역변수 `gl_FragColor` 에 최종 픽셀 컬러를 할당할 수 있다.
- 위 변수의 타입은 vec4로 되어있는데 이건 4차원 벡터다.
- vec4는 red, green, blue, alpha채널로 되어있고, 이 값들은 정규화 되어 있다. 0.0~1.0이라는 소리.
- 위 코드에는 매크로가 있는데, 글로벌 변수를 사용하고 기본적인 연산을 (#ifdef, #endif)할 수 있다고 한다.
    - 위 코드에서는 `GL_ES` 가 #ifdef로 선언되어 있는데 주로 모바일이나 브라우저에서 컴파일 된다. (GL_ES: OpenGL경령화 버전)
    - `precision mediump float;` 는 float의 정밀도를 정하는 코드다. `lowp` , `highp` 로 낮거나 높게도 설정가능
- float를 사용할 때 부동소수점을 안쓰면 에러가 날 수도 있다. 그래픽 카드에서 자동형변환은 필수가 아니며 스펙에 포함되지 않을 수도 있기 때문.

- 함수 써보기
    - c언어랑 비슷해서 함수 사용 전에 함수가 선언되어 있어야됨.

![Hello world function](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/Hello world function.png)

## Uniforms

- cpu에서 보내는 인풋을 uniform으로 선언한다. 이 데이터는 읽기전용으로 선언된다.
    - 지원되는 타입: `float`, `vec2`, `vec3`, `vec4`, `mat2`, `mat3`, `mat4`, `sampler2D` and `samplerCube`
- gpu에서 지원하는 몇가지 함수들: `sin()`, `cos()`, `tan()`, `asin()`, `acos()`, `atan()`, `pow()`, `exp()`, `log()`, `sqrt()`, `abs()`, `sign()`, `floor()`, `ceil()`, `fract()`, `mod()`, `min()`, `max()` and `clamp()`.

- 코드의 u_time은 cpu에서 보내주는 시간이다. 예제에서는 로드 이후 경과 시간으로 주석이 달려있다.
- 아래 코드는 시간에 따라 빨강색의 색상이 변하게 한 것. u_time에 *0.1을하여 천천히 변하게 해보았다.
- abs()를 사용하는 이유는 sin함수는 -1.0~1.0의 값을 반환하는데 컬러에서는 0.0~1.0으로 정규화된 값을 사용한다. 이러면 음수는 0으로 들어가게 되어 그 부분만큼의 시간은 0이 되는 것.
    
    ![Graph sinNabs](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/Graph sinNabs.png)
    

```glsl
#ifdef GL_ES
precision mediump float;
#endif

uniform float u_time;

void main() {
    gl_FragColor = vec4(abs(sin(u_time *  0.1)),0.0,0.0,1.0);
}
```

- GLSL에서 기본적으로 주는 인풋이 있다. `vec4 gl_FragCoord` 이건 현재 작업 중인 픽셀의 좌표다. 이런건 우리가 uniform으로 선언하지 않는다.
    
    ![Input Coord](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/Input Coord.png)
    
- 마우스 주변에만 색이 있게 해본 코드
    - 시간에 따라서 색이 변하게 해보았다.
    - glsl에는 distance와 step 내장함수가 있어서 직접 구현 안해도 된다. 내장함수 쓰는게 성능면에서 더 좋다.
    - 테스트용으로 만져본거라 코드가 조금 더럽다.
    - 결과는 이러함
        
        ![Circle Where Mouse](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/CircleWMouse.png)
        

```glsl
#ifdef GL_ES
precision mediump float;
#endif

uniform vec2 u_resolution;
uniform vec2 u_mouse;
uniform float u_time;

float getDistance(vec2 v1, vec2 v2)
{
    float res = 0.0;
    float xdis = v2.x - v1.x;
    float xx = xdis * xdis;
    
    float ydis = v2.y - v1.y;
    float yy = ydis * ydis;
    
    res = sqrt(xx + yy);
    return res;
}

void main() {
        vec2 st = gl_FragCoord.xy/u_resolution;
    // 정규화
    vec2 mousePoint = u_mouse/u_resolution;
    
    float dist = getDistance(st, mousePoint);
    float reverseDist = clamp(0.0, 1.0, 1.0 - dist);
    
    float timeR = 0.0;
    float timeG = 0.0;
    float timeB = 0.0;
    
    // true: 원 안에서 색이 변함 다른 곳은 검은색
    // false: 그라데이션
    if (true)
    {
        // step()로 쓸 수 있음
            if (reverseDist > 0.7)
        {
            timeR = abs(sin(u_time * 0.1));
                timeG = abs(sin(u_time * 0.5));
                timeB = abs(sin(u_time * 2.0));
            // 거리 안 곱하면 단일 색
            // 있으면 원 안에 그라데이션
            timeR *= reverseDist;
            timeG *= reverseDist;
            timeB *= reverseDist;
        }
    }
    else
    {
        timeR = abs(sin(u_time * 0.1));
            timeG = abs(sin(u_time * 0.5));
            timeB = abs(sin(u_time * 2.0));
        timeR *= reverseDist;
        timeG *= reverseDist;
        timeB *= reverseDist;
    }
    
    gl_FragColor = vec4(
        timeR, 
        timeG, 
        timeB,
        1.0);
}
```

## Running your shader

- 이 파트는 쉐이더를 실행하는 환경을 만들고 공유하는 방법에 대해서 설명하고 있다.
- 저자가 만든 공유 사이트와 온라인 편집기에 대해서 알려주는 중

## Shaping fuctions

- `step()` : step은 인자로 (float edge, float x)를 받는다.
    - `x < edge` : 0 아니면 1
- `smoothstep()` : smoothstep은 인자로 (float edge0, float edge1, float x)를 받는다
    - `x ≤ edge0` : 0반환
    - `x ≥ edge1` : 1 반환
    - `edge0 < x < edge1`이면 0과 1 사이의 부드러운 보간값 반환
- `pow()` : 거듭제곱`sqrt()` : 제곱근`exp()` : 지수함수`log()` : 로그
    - 직선을 얼마나, 어느 방향으로 휘게 만들지
    - 애니메이션 속도 조절(Easing), 빛/색 감쇠 등에 사용한다
- `fract()` : 소수점 뒷부분만 남기는 함수
    - 같은 패턴을 반복할 때 사용
- `ceil()` : 값을 올림
- `floor()` : 값을 내림
- `mod()` : `%` 와 같음

![Shaping fuctions Exam](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/Shaping fuctions Exam.png)

- 위 코드를 살펴보자
    - `smoothstep(0.02, 0.0, abs(st.y - st.x));` :
        - 일단 `st.y - st.x` 는 y와 x좌표의 차이를 구한다. y와 x좌표가 차이없다면 0이 된다(대각선 위). 이를 abs로 절대값을 구한다.
        - `smoothstep()` 함수로 대각선과의 거리가 가까우면(0.02이내) 1, 이상이면 0을 반환한다.
    - 이 부분은 뒤에 x축을 기준으로 0~1의 컬러값(검은색→흰색 그라데이션)을 그리는 코드
        
        `vec2 st = gl_FragCoord.xy/u_resolution;
        float y = st.x;
        vec3 color = vec3(y);`
        
    - `float pct = plot(st);` : 이건 만들어둔 plot함수로 대각선 위인지 아닌지를 판정하는 코드. 선 위가 아니면 1.0이 유지되고 선 위면 0.0이 된다.
    - `color = (1.0-pct)**color+pct**vec3(0.0,1.0,0.0);` : 이 코드는 대각선 위에 초록색 선인지 컬러를 판정하는 코드이다.
        - `(1.0-pct)*color` : 원래 배경색을 선 위인지에 따라서 0(검은색으로) 만드는지 원래 배경색을 두는지(그라데이션)
        - `*pct**vec3(0.0,1.0,0.0)` : 대각선 위를 초록색 선으로 만드는 코드
- 위 코드 아래에는 x에 5제곱하여 곡선을 만들게 바꾼 코드가 있다.

```glsl
float plot(vec2 st, float pct){
    return  smoothstep( pct-0.02, pct, st.y) -
            smoothstep( pct, pct+0.02, st.y);
}
```

- 그 중 plot함수가 바뀌었다. 살펴보면 원래는 좌표의 차이를 구해서 abs를 씌워 선 근처면 1을 반환했었는데, 이건 위쪽 경계선 하나 + 아래쪽 경계선 하나를 따로 만들어서 빼는 방식이 되었다.
- 원래 abs를 사용하던 방식으로 바꿔보면: `smoothstep( 0.02, 0.0, abs(pct - st.y));`