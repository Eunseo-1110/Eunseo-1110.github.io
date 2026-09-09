---
title: "Patterns"
date: 2026-09-07 00:00:00 +0900
categories: [Shader, The book of shaders]
tags: [shader, study]     # TAG names should always be lowercase
---
    
## Patterns

- 쉐이더 프로그램은 픽셀별로 실행된다. 얼마나 도형을 반복해도 계산은 일정하게 유지된다. 이 말은 프래그먼트 쉐이더는 타일 패턴에 적합하다.
    - 일반 프로그래밍은 도형의 개수만큼 연산이 반복되어 계산량이 늘어나지만, 쉐이더에서는 도형이 반복되게 만든다고 해도 fract()연산이 한 번 추가되는 것이다. 픽셀하는 계산은 무엇을 그리던 내가 무슨 색인지만 지정하는 것.
- 우리의 전략은 좌표계를 곱하는 거다. 우리는 모형을 0~1사이로 그리고 그걸 반복해서 격자를 만들거다.
- 버전에 본 도형 패턴화 해본 코드
    
    ```glsl
    // Author @patriciogv - 2015
    
    #ifdef GL_ES
    precision mediump float;
    #endif
    
    uniform vec2 u_resolution;
    uniform float u_time;
    
    float circle(in vec2 _st, in float _radius){
        vec2 l = _st-vec2(0.5);
        return 1.-smoothstep(_radius-(_radius*0.01),
                                _radius+(_radius*0.01),
                                dot(l,l)*4.0);
    }
    
    void main() {
        vec2 st = gl_FragCoord.xy/u_resolution;
        vec3 color = vec3(0.0);
    
        st *= vec2(3.0,3.0);      // Scale up the space by 3
        st = fract(st); // Wrap around 1.0
    
        // Now we have 9 spaces that go from 0-1
    
        vec2 pos = vec2(0.5)-st;
        
        float r = length(pos)*2.0;
        float a = atan(pos.y,pos.x);
        float f = abs(cos(a*3.));
        
        color = vec3(0.817,0.830,0.540) * vec3( 1.-smoothstep(f,f+0.02,r) );
    
        gl_FragColor = vec4(color,1.0);
    }
    ```
    

### Offset patterns

- 벽돌과 같은 모양을 만들려면 어떻게 해야할까? mod()함수를 통해서 `mod(x, 2.0)` 같은 형태로 사용하면 0~2가 반복된다. 그래서 이 값이 1.0보다 작은지를 판별하면 홀수칸에 있는지 짝수칸에 있는지를 확인할 수 있다.
    - 아래의  brickTile함수를 살펴보면:
        - `_st *= _zoom;` : 좌표계를 확대한다
        - `_st.x += step(…` : 확대된 좌표계를 0~2로 반복시켜 1보다 크면 x좌표를 0.5만큼 민다
        - `fract(_st)` : 좌표계를 다시 0~1로 돌린다
    
    ```glsl
    // Author @patriciogv ( patriciogonzalezvivo.com ) - 2015
    
    vec2 brickTile(vec2 _st, float _zoom){
        _st *= _zoom;
    
        // Here is where the offset is happening
        _st.x += step(1.,mod(_st.y,2.0)) * 0.5;
    
        return fract(_st);
    }
    ```
    

#### mod의 반복에 대해서 정리

- `mod(x, n) < p` 형태로 되어있다. 
여기서 n을 바꾸면 반복 주기가 바뀌는데 예를 들어 3.0을 넣으면 0~3을 반복하게 된다.
그리고 이 값을 판별할 때 1.0이 아닌 예를들어 0.4같은 값을 사용한다면 전체 0~n구간 중 0.4만 칠하거나 하는 형태다. 비율이 바뀌는 것.
추가로 이 n을 3으로 두면 0~3이 되는데 판별하는 것도 1.0,2.0로 판별하면 세 구간으로도 나눌 수 있다.
    - `x` : 나눠지는 값(좌표)
    - `n`: 반복 주기
    - `p`: 비교 기준선

#### 시간에 따라 x와 y를 교차하면서 움직이기

- 예제의 코드를 활용하여 문제로 있던 것과 같은 화면을 만들었다.
    
    <canvas id="cross-xy" style="width:100%; max-width:300px; aspect-ratio:1/1; display:block; margin:20px auto; background:#222;"></canvas>

    {% include glsl-boilerplate.html %}

    <script>
    window.runGLSL("cross-xy", `
        precision mediump float;uniform vec2 u_resolution;uniform float u_time;vec2 brickTileMoveX(vec2 _st, float _zoom){_st *= _zoom;float t = step(1.0,mod(_st.y,2.0));_st.x += mix(u_time, -u_time, t);return fract(_st);}vec2 brickTileMoveY(vec2 _st, float _zoom){ _st *= _zoom;float t = step(1.0,mod(_st.x,2.0));_st.y += mix(u_time, -u_time, t);return fract(_st);}float box(vec2 _st, vec2 _size){ _size = vec2(0.5)-_size*0.5;vec2 uv = smoothstep(_size,_size+vec2(1e-4),_st);uv *= smoothstep(_size,_size+vec2(1e-4),vec2(1.0)-_st);return uv.x*uv.y;}void main(void){vec2 st = gl_FragCoord.xy/u_resolution.xy;vec3 color = vec3(0.0);vec2 stY = brickTileMoveY(st,10.0);vec2 stX = brickTileMoveX(st,10.0);float t = fract(u_time * 0.5);float useX = step(0.5, t);st = mix(stY, stX, useX);st = vec2(0.5) - st;color = step(0.3, vec3(length(st))); gl_FragColor = vec4(color,1.0);}
    `);
    </script>

    ```glsl
    // Author @patriciogv ( patriciogonzalezvivo.com ) - 2015
    #ifdef GL_ES
    precision mediump float;
    #endif
    uniform vec2 u_resolution;
    uniform float u_time;
    vec2 brickTileMoveX(vec2 _st, float _zoom){
        _st *= _zoom;
        // Here is where the offset is happening
        float t = step(1.0,mod(_st.y,2.0));
        
        _st.x += mix(u_time, -u_time, t);
    
        return fract(_st);
    }
    vec2 brickTileMoveY(vec2 _st, float _zoom){
        _st *= _zoom;
        // Here is where the offset is happening
        float t = step(1.0,mod(_st.x,2.0));
    
        _st.y += mix(u_time, -u_time, t);
    
        return fract(_st);
    }
    float box(vec2 _st, vec2 _size){
        _size = vec2(0.5)-_size*0.5;
        vec2 uv = smoothstep(_size,_size+vec2(1e-4),_st);
        uv *= smoothstep(_size,_size+vec2(1e-4),vec2(1.0)-_st);
        return uv.x*uv.y;
    }
    void main(void){
        vec2 st = gl_FragCoord.xy/u_resolution.xy;
        vec3 color = vec3(0.0);
        // Modern metric brick of 215mm x 102.5mm x 65mm
        // http://www.jaharrison.me.uk/Brickwork/Sizes.html
        // st /= vec2(2.15,0.65)/1.5;
    
        vec2 stY = brickTileMoveY(st,10.0);
        vec2 stX = brickTileMoveX(st,10.0);
        
        float t = fract(u_time * 0.5);
        float useX = step(0.5, t);
        st = mix(stY, stX, useX);
    
        st = vec2(0.5) - st;
        color = step(0.3, vec3(length(st)));
        //color = vec3(box(st,vec2(0.9)));
        // Uncomment to see the space coordinates
        //color = vec3(st,0.0);
        gl_FragColor = vec4(color,1.0);
    }
    ```
    

### Truchet Tiles

- 다음 함수를 살펴보자
    - 우선 들어오는 좌표를 한 번 더 2*2격자로 쪼갠다.
    - 그리고 mod를 통해서 2x2의 4칸에 각각 고유 번호를 매기는 것
        - `step(1.0, mod(_st.x,2.0);` : x가 오른쪽이면 1, 왼쪽이면 0
        - `step(1., mod(_st.y,2.0))*2.0;` : y가 위쪽칸이면 2, 아래쪽이면 0
        - 이 둘을 더하면 주석에 있는 좌표처럼 된다.
        - 왼쪽위: 2(0, 2) 오른쪽위: 3(1, 2)
        오른쪽위: 1(1, 0) 오른쪽아래: 0(0, 0)
    - `_st = fract(_st);`:그리고 좌표를 다시 0~1범위로 만든다
    - 그리고 인덱스별로 회전을 한다
        - 0번: 회전X
        - 1번: 90도회전
        - 2번: -90도회전
        - 3번: 180도회전
    
    ```glsl
    vec2 rotateTilePattern(vec2 _st){
        //  Scale the coordinate system by 2x2
        _st *= 2.0;
        //  Give each cell an index number
        //  according to its position
        float index = 0.0;
        index += step(1., mod(_st.x,2.0));
        index += step(1., mod(_st.y,2.0))*2.0;
        //      |
        //  2   |   3
        //      |
        //--------------
        //      |
        //  0   |   1
        //      |
        // Make each cell between 0.0 - 1.0
        _st = fract(_st);
        // Rotate each cell according to the index
        if(index == 1.0){
            //  Rotate cell 1 by 90 degrees
            _st = rotate2D(_st,PI*0.5);
        } else if(index == 2.0){
            //  Rotate cell 2 by -90 degrees
            _st = rotate2D(_st,PI*-0.5);
        } else if(index == 3.0){
            //  Rotate cell 3 by 180 degrees
            _st = rotate2D(_st,PI);
        }
        return _st;
    }
    ```
    
- 삼각형이 였던 코드를 반원으로 바꿔본 코드다.
    
    ```glsl
    void main (void) {
        vec2 st = gl_FragCoord.xy/u_resolution.xy;
    
        st = tile(st,3.0);
        st = rotateTilePattern(st);
    
        // Make more interesting combinations
        // st = tile(st,2.0);
        // st = rotate2D(st,-PI*u_time*0.25);
        // st = rotateTilePattern(st*2.);
        // st = rotate2D(st,PI*u_time*0.25);
    
        // step(st.x,st.y) just makes a b&w triangles
        // but you can use whatever design you want.
        //gl_FragColor = vec4(vec3(step(st.x,st.y)),1.0);
        st = vec2(1.0, 0.5) - st;
        float circle = step(length(st), 0.5);
        float halfcircle = step(0.0, st.x);
        float res = circle *  halfcircle;
        vec3 color = mix(vec3(1.0), vec3(0.0), res);
    
        gl_FragColor = vec4(color,1.0);
    }
    ```