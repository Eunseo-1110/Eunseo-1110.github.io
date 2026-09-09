---
title: "Random"
date: 2026-09-08 00:00:00 +0900
categories: [Shader, The book of shaders]
tags: [shader, study]     # TAG names should always be lowercase
---

## Random

- 쉐이더 코드에는 기본 랜덤함수가 없다. 그래서 우리는 의사난수 값을 얻어서 랜덤과 비슷한 효과를 내야한다. `sin()` 은 -1.0~1.0사이에서 변동하는 값을 반환한다. 그리고 이 값을 fract()를 통해 양수의 값만 남기고, 이 값을 아주 작게 분할하면 의사 난수값을 얻을 수 있다.
    
    `y = fract(sin(x)*100000.0`
    

### Controlling chaos

- 랜덤을 사용하는 것은 어려울 수도 있다. 랜덤은 너무 혼란할수도 그리고 랜덤이 충분하지 않을 수도 있다.
- sin()의 -1.5707, 1.5707부분에서는 사인파의 crest(진폭이 가장 높은 부분)를 볼 수 있다. 이 부분은 사인파의 최대값과 최소값이 발생하는 부분이다.
    
    ![Draw Two Ractangle](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/Controlling chaos_sin crest.png)
    
- `sqrt` , `pow` 함수를 통해서 분포도를 조절할 수 있다.
    - `y = pow(rand(x), 5.0)`
        
        ![Draw Two Ractangle](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/Controlling chaos_sin pow.png)
        
- 한가지 더 알아둬야하는게, 위에서 사용하는 함수는 결정론적 랜덤이다. 이 말은 `rand(1.0)` 은 언제나 같은 결과라는 뜻이다. ActionScript `Math.random()` 함수처럼 매번 호출할 때마다 다른값을 내뱉는게 아니다.

### 2d Random

- 2차원 벡터를 하나의 차원인 float 값으로 변환해야하는데 이럴 때 `dot()` 함수를 사용하면 된다. 이 함수는 두 벡터에 따라 0~1사이의 float 값을 반환한다.
    
    ```glsl
    // Author @patriciogv - 2015
    // http://patriciogonzalezvivo.com
    
    float random (vec2 st) {
        return fract(sin(dot(st.xy,
                                vec2(12.9898,78.233)))*
            43758.5453123);
    }
    ```
    
- 마우스 위치에 따라 노이즈가 변하게 해본 코드
    
    ```glsl
    void main() {
        vec2 st = gl_FragCoord.xy/u_resolution.xy;
        vec2 stM = u_mouse / u_resolution.xy;
        float rnd = random( st * stM );
    
        gl_FragColor = vec4(vec3(rnd),1.0);
    }
    ```
    

### Using the chaos

- TV같은 노이즈는 원재료 마테리얼로 쓰기 어려운데 다른 방식을 배워보자.
- `floor()` 을 사용하면 정수 셀 테이블을 만들 수 있다.
- 아래 코드를 살펴보자면
    - `st *= 10.0;` 을 통해서 좌표계를 스케일링 해준다.
    - floor을 통해서 정수를 분리한다.
    - 분리된 정수값을 random의 인자로 사용하면 난수값은 같은 값에 대해서 같은 결과를 내놓는다. 이렇게 하나의 셀처럼 보이게 만들 수 있다.
    - `fract(st)` 로 이전에 배웠던 타일링을 할 수 있다. 랜덤과 타일링을 섞어서 다양한 결과를 만들어 낼 수 있다.
    
    ```glsl
    // Author @patriciogv - 2015
    // Title: Mosaic
    
    #ifdef GL_ES
    precision mediump float;
    #endif
    
    uniform vec2 u_resolution;
    uniform vec2 u_mouse;
    uniform float u_time;
    
    float random (vec2 st) {
        return fract(sin(dot(st.xy,
                                vec2(12.9898,78.233)))*
            43758.5453123);
    }
    
    void main() {
        vec2 st = gl_FragCoord.xy/u_resolution.xy;
    
        st *= 10.0; // Scale the coordinate system by 10
        vec2 ipos = floor(st);  // get the integer coords
        vec2 fpos = fract(st);  // get the fractional coords
    
        // Assign a random value based on the integer coord
        vec3 color = vec3(random( ipos ));
    
        // Uncomment to see the subdivided grid
        // color = vec3(fpos,0.0);
    
        gl_FragColor = vec4(color,1.0);
    }
    ```
    
- 아래 함수에 대해서 살펴보자. 저번에 봤던 타일에서 4분면을 나누던 함수랑 비슷한 구조다.
    - 일단 목적은 저번엔 각 칸을 회전하는 거였지만 이번엔 각 칸을 다르게(무작위로) 뒤집기 위함이다.
        - 0~0.25: 그대로/0.25~0.5: y 뒤집기/0.5~0.75: x뒤집기/0.75~: x,y뒤집기
    - 인덱스를 함수 안에서 결정하지 않고 받아서 결정한다. 이 예제에서는 위에서 봤던 `ipos = floor(st)` 로 인덱스를 받고 있다.
    - `_index = fract(((_index-0.5)*2.0));` : 이건 `fract(index * 2.0)` 이랑 같은 코드인데, 이러면 0.0~1.0이 있다고 치면 0.0~0.49와 0.5~0.99의 패턴이 동일해진다. 패턴이 2번 반복되는 것.
    
    ```glsl
    // Author @patriciogv - 2015
    // Title: Truchet - 10 print
    
    vec2 truchetPattern(in vec2 _st, in float _index){
        _index = fract(((_index-0.5)*2.0));
        if (_index > 0.75) {
            _st = vec2(1.0) - _st;
        } else if (_index > 0.5) {
            _st = vec2(1.0-_st.x,_st.y);
        } else if (_index > 0.25) {
            _st = 1.0-vec2(1.0-_st.x,_st.y);
        }
        return _st;
    }
    ```