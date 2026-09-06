---
title: "Shapes"
date: 2026-09-05 00:00:00 +0900
categories: [Shader, The book of shaders]
tags: [shader, study]     # TAG names should always be lowercase
---

## Shapes

### Rectangle
- 저자가 박스 그리는 방법에 대해서 설명중
- 이건 좌측과 하단에 선을 그리는 코드다. 주석을 풀면 상단과 우측에 선을 그린다.
    - step()함수를 통해서 st.x와 st.y의 값이 0.1보다 큰지 확인한다. step()은 크면 1, 작으면 0을 반환한다. 
    그리고 이 x와 y를 곱하면 둘다 1인 곳`(x ≥ 0.1 && y ≥ 0.1)`은 1(흰색)이 되고 
    둘 중에 하나라도 0인 곳`(x < 0.1 || y < 0.1)`은 0이 된다.
    - 상단과 하단은 1.0 - st를 하면 된다.
    
    ```glsl
    #ifdef GL_ES
    precision mediump float;
    #endif
    
    uniform vec2 u_resolution;
    uniform vec2 u_mouse;
    uniform float u_time;
    
    void main(){
        vec2 st = gl_FragCoord.xy/u_resolution.xy;
        vec3 color = vec3(0.0);
    
        // bottom-left
        vec2 bl = step(vec2(0.1),st);
        float pct = bl.x * bl.y;
    
        // top-right
        // vec2 tr = step(vec2(0.1),1.0-st);
        // pct *= tr.x * tr.y;
    
        color = vec3(pct);
    
        gl_FragColor = vec4(color,1.0);
    }
    ```
    
#### Rectangle Function
- 예제를 보고 사각형을 함수화 해서 만들어봤다. 결과는 파란색 테두리 사각형과 노란색 사각형이 나온다.
    - 위의 예제에서 좌표가 0.1보다 큰지 확인하는 코드로 큰 사각형(size - thickness)에서 작은 사각형(size - thickness)을 빼서 그 사각형 사이만 남기는 방식으로 테두리 사각형을 만들 수 있다.
    - smoothstep()으로 만들면 테투리가 부드러운 사각형을 만들 수 있다.
    
    ![Draw Two Ractangle](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/Draw Two Ractangle.png)
    
    ```glsl
    // Author @patriciogv - 2015
    // http://patriciogonzalezvivo.com
    
    #ifdef GL_ES
    precision mediump float;
    #endif
    
    uniform vec2 u_resolution;
    uniform vec2 u_mouse;
    uniform float u_time;
    
    float fillRectSmooth(vec2 coord, vec2 margin, float smoothness) {
        vec2 bl = smoothstep(margin - smoothness, margin + smoothness,coord);       // 왼쪽/아래 경계 안쪽인가
        vec2 tr = smoothstep(margin - smoothness, margin + smoothness, 1.0 - coord); // 오른쪽/위 경계 안쪽인가
        return bl.x * bl.y * tr.x * tr.y;    // 네 조건 다 만족해야 내부
    }
    
    float fillRect(vec2 coord, vec2 margin) {
        vec2 bl = step(margin, coord);       // 왼쪽/아래 경계 안쪽인가
        vec2 tr = step(margin, 1.0 - coord); // 오른쪽/위 경계 안쪽인가
        return bl.x * bl.y * tr.x * tr.y;    // 네 조건 다 만족해야 내부
    }
    
    float drawRectangle(vec2 coord, vec2 size, float thickness) {
        float outer = fillRectSmooth(coord, size - thickness, 0.0);
        float inner = fillRectSmooth(coord, size + thickness, 0.0);
        return outer - inner; // 큰 사각형에서 작은 사각형을 빼면 테두리만 남음
    }
    
    float drawRectangleFull(vec2 coord, vec2 size)
    {
        float rect = fillRectSmooth(coord, size, 0.001);
        return rect; 
    }
    
    void main(){
        vec2 st = gl_FragCoord.xy/u_resolution.xy;
        vec3 color = vec3(0.0);
    
        // bottom-left
        //vec2 bl = step(vec2(0.1),st);
        //float pct = bl.x * bl.y;
    
        // top-right
        // vec2 tr = step(vec2(0.1),1.0-st);
        // pct *= tr.x * tr.y;
    
        color = vec3(0.000,0.572,0.975); 
        vec3 color2 = vec3(0.940,0.774,0.148);
        
        vec3 resColor = vec3(0.0);
        resColor = mix(resColor, color, drawRectangle(st, vec2(0.1,0.1), 0.020));
        resColor = mix(resColor, color2, drawRectangleFull(st+vec2(-0.290,0.050), vec2(0.380,0.300)));
        
        gl_FragColor = vec4(resColor,1.0);
    }
    ```

### Circles
- 다음은 원그리기에 대해서 설명하고 있다. 원을 그리려면 원의 중심부터 거리를 계산하여 그리는 방식이 있다.
- `distance(), length(), sqrt()` 함수를 통해서 거리를 구할 수 있다. 다음은 각 3개의 함수로 같은 결과를 내는 코드다.
    - `distance()` : 두 점 사이의 거리를 반환한다. 이 함수의 내부 코드는 두 점을 뺀 벡터의 거리를 구하는 것.
        - `distance(a, b) == length(a - b)`
    - `length()` : 벡터의 크기(원점에서부터의 거리)를 반환한다.
    - `sqrt()` : 제곱근. 아래 코드에서는 직접 거리를 구하는 공식을 계산하는데에 사용하고 있다. `length()` 의 내부 코드와 같다.
    
    ```glsl
    // Author @patriciogv - 2015
    // http://patriciogonzalezvivo.com
    
    #ifdef GL_ES
    precision mediump float;
    #endif
    
    uniform vec2 u_resolution;
    uniform vec2 u_mouse;
    uniform float u_time;
    
    void main(){
        vec2 st = gl_FragCoord.xy/u_resolution;
        float pct = 0.0;
    
        // a. The DISTANCE from the pixel to the center
        pct = distance(st,vec2(0.5));
    
        // b. The LENGTH of the vector
        //    from the pixel to the center
        // vec2 toCenter = vec2(0.5)-st;
        // pct = length(toCenter);
    
        // c. The SQUARE ROOT of the vector
        //    from the pixel to the center
        // vec2 tC = vec2(0.5)-st;
        // pct = sqrt(tC.x*tC.x+tC.y*tC.y);
    
        vec3 color = vec3(pct);
    
        gl_FragColor = vec4( color, 1.0 );
    }
    ```
    
#### Distance field
- 이런 코드가 있는데 하나씩 살펴보자
    
    ```glsl
    pct = distance(st,vec2(0.4)) + distance(st,vec2(0.6));
    pct = distance(st,vec2(0.4)) * distance(st,vec2(0.6));
    pct = min(distance(st,vec2(0.4)),distance(st,vec2(0.6)));
    pct = max(distance(st,vec2(0.4)),distance(st,vec2(0.6)));
    pct = pow(distance(st,vec2(0.4)),distance(st,vec2(0.6)));
    ```
    
    - `pct = min(distance(st,vec2(0.4)),distance(st,vec2(0.6)));` : 두 원의 합집합(or). 두 점 중 더 가까운 쪽의 거리를 고른다.  원이 겹쳐진 것 처럼 보인다.
        - `step(0.5(경계값), 1.0 - pct)` 함수를 통해서 A안에 있거나 B안에 있으면 1.0이 된다. 둘 중 하나만 경계값보다 작아도 충족.
        
        ![Distance Min](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/Distance Min.png)
        
    - `max(distance(st,vec2(0.4)),distance(st,vec2(0.6)));` : 두 원의 교집합(and). 두 점 중 더 먼 쪽의 거리를 고른다.
        - 둘 다 경계값보다 작아야 충족.
        
        ![Distance Max](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/Distance Max.png)
        
    - `pct = distance(st,vec2(0.4)) + distance(st,vec2(0.6));`  타원의 정의. 두 정점으로부터 거리의 합이 일정한 점들의 집합.
        - [Ellipse Animation](https://youtu.be/XLjnTgXXgXk?si=DFiKpWFTykChp4y8)
        
        ![Distance Ellipse](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/Distance Ellipse.png)
        
    - `pct = distance(st,vec2(0.4)) * distance(st,vec2(0.6));` : 카시니 타원, 땅콩 모양. 두 점까지의 거리의 곱이 일정한 점들.
        - [Cassini ovals](https://youtu.be/bhTkPJ0ty1k?si=t84uLik99PzRSx1j)
        
        ![Distance Cassini Ellipse](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/Distance Cassini Ellipse.png)
        
    - `pct = pow(distance(st,vec2(0.4)),distance(st,vec2(0.6)));` : 한쪽 거리를 지수로 사용하여 다른쪽의 거리를 왜곡함. 두번째 인자가 화면의 위치에 따라 계속 바뀌어서 위치마다 휘어지는 정도가 다르다.
        - a<1: b(a에 곱하는 횟수)가 커질 수록 점점 0에 가까워짐 = distB에서 멀어질 수록 검정(0)에 가까워짐
        - a>1: b가 커질 수록 점점 값이 커짐 = 여기부턴 값이 커지는데 색상은 1까지라 흰색(1)
        - a=1: 몇 번을 곱하든 항상 1 = 거리가 1인 원 둘레는 흰색(1)
        - b가 0에 가까우면 결과는 1 = distB 근처는 흰색(1)
        
        | ![Distance Pow](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/Distance Pow.png) | ![Distance Pow Gray](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/Distance Pow Gray.png) |
        
        
#### Draw Circle(use dot)
- 다음은 내적을 통한 원그리기 함수에 대한 코드다.
    - `sqrt()` 와 이 함수를 사용하는 함수들은 비용이 많이 들 수 있다.
    
    ```glsl
    float circle(in vec2 _st, in float _radius){
        vec2 dist = _st-vec2(0.5);
        return 1.-smoothstep(_radius-(_radius*0.01),
                                _radius+(_radius*0.01),
                                dot(dist,dist)*4.0);
    }
    ```
    
    - `vec2 dist = _st-vec2(0.5);` : 화면 중심을 기준으로 한 좌표를 구한다.
    - `dot(dist, dist)` : 내적은 각 벡터의 같은 성분끼리 각각 곱해서 모두 더하는 것.
    이 코드는 같은 벡터끼리 내적하는 코드인데, 이걸 풀자면  `dist.x*dist.x + dist.y***dist.y` 이렇게 된다. 이 말은 거리를 구하는 공식에서 제곱근 안씌운 버전이 된다. 거리가 특정 값보다 작은지 큰지만 판별하는 거면 굳이 sqrt를 쓰지않고 거리와 반지름을 제곱해서 써도 결과가 같다.
    - `* 4.0` : 숫자의 범위를 0~1로 맞추기 위한 것. `$2^2$` 와 같은 값이라 볼 수 있다.
    - `smoothstep()` :`_radius*0.01` (반지름의 1%)만큼 부드럽게 처리한다. 원의 작으면 경계값도 얇아야 되고 원이 커지면 경계도 두꺼워져야 경계부분이 자연스러워 진다.
    - `1.-smoothstep()` : 결과 뒤집기. 안쪽이 1.0, 바깥쪽이 0.0.
  
### Useful properties of a Distance Field
- 거리함수를 통해서 다양한 패턴을 만드는 코드
    - `abs(st)`: 네 사분면을 하나로 접기. abs는 음수를 양수로 뒤집어서 왼/오,위/아래에 있던 부분이 오른쪽 위 하나로 겹쳐진다.
    - 거리함수를 통한 패턴 만들기
        - `d = length(abs(st) - 0.3)` : 네 사분면에서 각각 (+-0.3, +-0.3)점까지의 거리를 구한다. 네 개의 원이 배치됨
        - `d = length( min(abs(st)-.3,0.));` ,`d = length( max(abs(st)-.3,0.));` : 네 점까지의 거리. 모서리가 둥글게 부푼 도형이 나온다.
        - max: 양수만 남긴다. 안쪽 영역은 전부 0이 되어 사각형 바깥 영역만 남기는 것. 모서리가 둥근 사각형.
        d = 0인 지점: 각진 사각형/ d > 0 지점: 모서리 부분에서만 둥글게 원호를 그림
        모서리에서 사방으로 뻗어나가는 같은 거리의 점을 모으면 모서리 중심의로 한 원호가 생긴다.
        - min: 양수를 0으로 만든다. 사각형 안쪽만 남기는 것. 십자형태
    - 도형 시각화
        - `fract(d*10.0)` : 거리를 반복시켜서 등고선처럼 시각화 한다. 0~1을 반복해서 거리가 같은 지점끼리 띠 모양의 링이 생김
        - `gl_FragColor = vec4(vec3( step(.3,d) ),1.0);` : d가 0.3보다 크면 흰색, 작으면 검정으로 도형 바깥은 흰색 안쪽은 검정이 된다.
        - `gl_FragColor = vec4(vec3( step(.3,d) * step(d,.4)),1.0);` : d가 0.3~0.4사이인 테투리만 남는다.
        - `gl_FragColor = vec4(vec3( smoothstep(.3,.4,d)* smoothstep(.6,.5,d)) ,1.0);` : 이것도 테두리 만들기 인데 smoothstep버전
    
    ```glsl
    #ifdef GL_ES
    precision mediump float;
    #endif
    
    uniform vec2 u_resolution;
    uniform vec2 u_mouse;
    uniform float u_time;
    
    void main(){
        vec2 st = gl_FragCoord.xy/u_resolution.xy;
        // 종횡비 보정
        st.x *= u_resolution.x/u_resolution.y;
        vec3 color = vec3(0.0);
        float d = 0.0;
    
        // Remap the space to -1. to 1.
        // 이러면 화면 정중앙이 (0,0)임
        st = st *2.-1.;
    
        // Make the distance field
        d = length( abs(st)-.3 );
        // d = length( min(abs(st)-.3,0.) );
        // d = length( max(abs(st)-.3,0.) );
    
        // Visualize the distance field
        gl_FragColor = vec4(vec3(fract(d*10.0)),1.0);
    
        // Drawing with the distance field
        // gl_FragColor = vec4(vec3( step(.3,d) ),1.0);
        // gl_FragColor = vec4(vec3( step(.3,d) * step(d,.4)),1.0);
        // gl_FragColor = vec4(vec3( smoothstep(.3,.4,d)* smoothstep(.6,.5,d)) ,1.0);
    }
    ```

### Polar shapes
- 다음 코드는 좌표를 각도로 만들어서 cos()을 통해서 다양한 모형을 만드는 코드다. 주석풀면서 하나씩 체크할 수 있다.
    
    ```glsl
    // Author @patriciogv - 2015
    // http://patriciogonzalezvivo.com
    
    #ifdef GL_ES
    precision mediump float;
    #endif
    
    uniform vec2 u_resolution;
    uniform vec2 u_mouse;
    uniform float u_time;
    
    void main(){
        vec2 st = gl_FragCoord.xy/u_resolution.xy;
        vec3 color = vec3(0.0);
    
        vec2 pos = vec2(0.5)-st;
    
        float r = length(pos)*2.0;
        float a = atan(pos.y,pos.x);
    
        float f = cos(a*3.);
        // f = abs(cos(a*3.));
        // f = abs(cos(a*2.5))*.5+.3;
        // f = abs(cos(a*12.)*sin(a*3.))*.8+.1;
        // f = smoothstep(-.5,1., cos(a*10.))*0.2+0.5;
    
        color = vec3( 1.-smoothstep(f,f+0.02,r) );
    
        gl_FragColor = vec4(color, 1.0);
    }
    ```
    
- 위 예제 코드로 전에 나왔던 `plot()` 을 통해서 윤곽선만 그리는 + 회전 애니메이션이 들어간 코드다.
    
    ```glsl
    // Author @patriciogv - 2015
    // http://patriciogonzalezvivo.com
    
    #ifdef GL_ES
    precision mediump float;
    #endif
    
    #define PI 3.14159265359
    
    uniform vec2 u_resolution;
    uniform vec2 u_mouse;
    uniform float u_time;
    
    float plot(vec2 st, float pct, float thickness)
    {
        return smoothstep(pct - thickness, pct, st.y)
            - smoothstep(pct, pct + thickness, st.y);
    }
    
    void main(){
        vec2 st = gl_FragCoord.xy/u_resolution.xy;
        vec3 color = vec3(0.0);
    
        vec2 pos = vec2(0.5)-st;
        
        float r = length(pos)*2.0;
        float a = atan(pos.y,pos.x);
        a += u_time;
        
        float f = cos(a*3.);
        f = abs(cos(a*3.));
            //f = abs(cos(a*2.5))*.5+.3;
        // f = abs(cos(a*12.)*sin(a*3.))*.8+.1;
        // f = smoothstep(-.5,1., cos(a*10.))*0.2+0.5;
    
        //color = vec3( 1.-smoothstep(f,f+0.02,r) );
        color = vec3( plot(vec2(r, r), f, 0.02));
        
        gl_FragColor = vec4(color, 1.0);
    }
    ```

### Combining powers
- 다음 코드는 다각형을 만드는 코드다 `N` 의 숫자에따라 다각형이 생긴다.
    - `float a = atan(st.x,st.y)+PI;` : 원래 각도를 구할때는 `atan(y,x)` 를 사용했었다. 하지만 여기서는 `atan(x,y)`인데 이러면 각도의 기준 방향이 90도 돌아가게 된다.
    `+PI` 를 하는 이유는 각도 범위를 양수로 만들기 위함이다.
    - `d = cos(floor(.5+a/r)*r-a)*length(st);` : 실제 거리(length)를, 방향에 따라 cos으로 눌러서(투영해서) 보정한 값이 d다. 이 d를 모든 픽셀에 대해 계산하고, 원 그릴 때처럼 특정 threshold(0.4) 기준으로 안/밖을 나누면, 방향마다 눌리는 정도가 달라서 원이 아니라 각진 다각형이 나온다.
        - `floor(.5+a/r)*r` : 이 픽셀이 몇 번째 조각(slice)에 있는지 찾고, 그 조각의 정중앙 각도를 구함
        - `(조각 중앙각도) - a` : 정중앙에서 얼마나 벗어나 있는지 각도의 차이 구하기
        - `cos(각도차이) * length(st)` : 투영비율(cos)에 거리를 곱해서 중심에서부터 변까지의 거리를 구함
    
    ```glsl
    #ifdef GL_ES
    precision mediump float;
    #endif
    
    #define PI 3.14159265359
    #define TWO_PI 6.28318530718
    
    uniform vec2 u_resolution;
    uniform vec2 u_mouse;
    uniform float u_time;
    
    // Reference to
    // http://thndl.com/square-shaped-shaders.html
    
    void main(){
        vec2 st = gl_FragCoord.xy/u_resolution.xy;
        st.x *= u_resolution.x/u_resolution.y;
        vec3 color = vec3(0.0);
        float d = 0.0;
    
        // Remap the space to -1. to 1.
        st = st *2.-1.;
    
        // Number of sides of your shape
        int N = 3;
    
        // Angle and radius from the current pixel
        float a = atan(st.x,st.y)+PI;
        float r = TWO_PI/float(N);
    
        // Shaping function that modulate the distance
        d = cos(floor(.5+a/r)*r-a)*length(st);
    
        color = vec3(1.0-smoothstep(.4,.41,d));
        // color = vec3(d);
    
        gl_FragColor = vec4(color,1.0);
    }
    
    ```