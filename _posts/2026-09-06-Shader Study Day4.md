---
title: "Matrices"
date: 2026-09-06 00:00:00 +0900
categories: [Shader, The book of shaders]
tags: [shader, study]     # TAG names should always be lowercase
---

## Matrices

### Translate

- 전 챕터에서 모형을 그렸으니 이제 그 모형을 옮기는 트릭에 대해서 설명하고 있다.
그 방법은 좌표계 자체를 옮기는 것. `st` 에 더하기만하면 좌표계가 옮겨진다.
- 아래 코드는 Translate예제를 참고해서 움직임을 Y축 기준으로 십자가가 핑퐁되는 느낌이 되게 만들었다.

    <canvas id="pingpong-cross" style="width:100%; max-width:300px; aspect-ratio:1/1; display:block; margin:20px auto; background:#222;"></canvas>

    {% include glsl-boilerplate.html %}

    <script>
    window.runGLSL("pingpong-cross", `
        precision mediump float; uniform vec2 u_resolution; uniform float u_time; float box(in vec2 _st, in vec2 _size){ _size = vec2(0.5) - _size*0.5; vec2 uv = smoothstep(_size,_size+vec2(0.001),_st);uv *= smoothstep(_size,_size+vec2(0.001),vec2(1.0)-_st);return uv.x*uv.y;}float cross(in vec2 _st, float _size){return  box(_st, vec2(_size,_size/4.)) +box(_st, vec2(_size/4.,_size));}void main(){vec2 st = gl_FragCoord.xy/u_resolution.xy;vec3 color = vec3(0.0);float y = sin(u_time) * 0.5;float x = (abs(fract(u_time * 0.5) * 2.0 - 1.0)) - 0.5;vec2 translate = vec2(x, y);st += translate *0.75;color += vec3(cross(st,0.25));gl_FragColor = vec4(color,1.0);}
    `);
    </script>


    ```glsl
    // Author @patriciogv ( patriciogonzalezvivo.com ) - 2015
    
    #ifdef GL_ES
    precision mediump float;
    #endif
    
    uniform vec2 u_resolution;
    uniform float u_time;
    
    float box(in vec2 _st, in vec2 _size){
        _size = vec2(0.5) - _size*0.5;
        vec2 uv = smoothstep(_size,
                            _size+vec2(0.001),
                            _st);
        uv *= smoothstep(_size,
                        _size+vec2(0.001),
                        vec2(1.0)-_st);
        return uv.x*uv.y;
    }
    
    float cross(in vec2 _st, float _size){
        return  box(_st, vec2(_size,_size/4.)) +
                box(_st, vec2(_size/4.,_size));
    }
    
    void main(){
        vec2 st = gl_FragCoord.xy/u_resolution.xy;
        vec3 color = vec3(0.0);
    
        // To move the cross we move the space
        float y = sin(u_time) * 0.5;
        // *2.0하면 fract가 만든(0~1)값이 (0~2)가 된다
        // 위 값에 -1을 하면 (-1~1)이 된다
        // 위 값에 abs를 씌우면 1-0-1-0을 반복한다
        float x = (abs(fract(u_time * 0.5) * 2.0 - 1.0)) - 0.5;
        vec2 translate = vec2(x, y); //vec2(cos(u_time * s),sin(u_time * s));
        st += translate *0.75;
    
        // Show the coordinates of the space on the background
        // color = vec3(st.x,st.y,0.0);
    
        // Add the shape on the foreground
        color += vec3(cross(st,0.25));
    
        gl_FragColor = vec4(color,1.0);
    }
    ```
    

### Rotations

- 회전또한 좌표계를 움직여야한다. 회전을 하려면 matrix를 사용해야 한다.
- 행렬은 열과 행으로 구성된 숫자들의 집합인데 벡터는 특정 규칙에 따라 행렬과 곱해져서 벡터의 값을 특정한 방식으로 변경한다.
- glsl은 `mat2` , `mat3`,`mat4` 를 지원하고 행렬간 곱셈을 지원하고 특별한 행렬 함수(matrixCompMult)도 지원한다.
- 회전을 하기전에 우리는 모형을 (0,0)으로 옮기고 회전 한 뒤에 다시 원래 위치로 옮겨줘야 한다.
    - 3D하던 관점으로 적용해서 도형을 0,0으로 옮기고 회전을 하는 건가 싶었는데, 회전하고싶은 점을 그 점만큼 빼주는 느낌이 더 맞는 것 같다. 같은 말이긴 하지만 관점이 조금 다른 듯.
- 다음 함수를 통해서 2d회전 행렬을 얻을 수 있다. 이렇게 얻은 회전행렬을 벡터와 곱하면 좌표를 회전할 수 있는 것이다.
    
    ```glsl
    mat2 rotate2d(float _angle){
        return mat2(cos(_angle),-sin(_angle),
                    sin(_angle),cos(_angle));
    }
    ```
    
- 전에 만든 십자가를 이동하는 코드에 회전을 추가.
    - 원래 `st -= translate` 로 했었는데 이러니까 회전이 이상하게 된다. 왜그런지 봤더니 `box()` 함수 내부에 0.5지점에 그려지도록 되어 있어서 그런 것. 그래서 0.5를 넣으면 십자가가 회전하면서 이동을 했던 것이였다.

    <canvas id="pingpong-cross-rotate" style="width:100%; max-width:300px; aspect-ratio:1/1; display:block; margin:20px auto; background:#222;"></canvas>

    <script>
    window.runGLSL("pingpong-cross-rotate", 
    "#define PI 3.14159265359\n" + 
    "precision mediump float;uniform vec2 u_resolution;uniform float u_time;mat2 rotate2d(float _angle){return mat2(cos(_angle),-sin(_angle),sin(_angle),cos(_angle));}float box(in vec2 _st, in vec2 _size){_size = vec2(0.5) - _size*0.5;vec2 uv = smoothstep(_size,_size+vec2(0.001),_st);uv *= smoothstep(_size,_size+vec2(0.001),vec2(1.0)-_st);return uv.x*uv.y;} float cross(in vec2 _st, float _size){return  box(_st, vec2(_size,_size/4.)) +box(_st, vec2(_size/4.,_size));}void main(){vec2 st = gl_FragCoord.xy/u_resolution.xy;vec3 color = vec3(0.0);float y = sin(u_time) * 0.5;float x = (abs(fract(u_time * 0.5) * 2.0 - 1.0)) - 0.5;vec2 translate = vec2(x, y);translate *= 0.75;st += translate;st -= vec2(0.5);st = rotate2d( sin(u_time)*PI ) * st;st += vec2(0.5);color += vec3(cross(st,0.25));gl_FragColor = vec4(color,1.0);}"
    );
    </script>


    ```glsl
    void main(){
        vec2 st = gl_FragCoord.xy/u_resolution.xy;
        vec3 color = vec3(0.0);
    
        // To move the cross we move the space
        float y = sin(u_time) * 0.5;
        // *2.0하면 fract가 만든(0~1)값이 (0~2)가 된다
        // 위 값에 -1을 하면 (-1~1)이 된다
        // 위 값에 abs를 씌우면 1-0-1-0을 반복한다
        float x = (abs(fract(u_time * 0.5) * 2.0 - 1.0)) - 0.5;
        vec2 translate = vec2(x, y); //vec2(cos(u_time * s),sin(u_time * s));
        
        translate *= 0.75;
        
        st += translate;
        
        st -= vec2(0.5);
        st = rotate2d( sin(u_time)*PI ) * st;
        st += vec2(0.5);
        
        // Show the coordinates of the space on the background
        // color = vec3(st.x,st.y,0.0);
    
        // Add the shape on the foreground
        color += vec3(cross(st,0.25));
    
        gl_FragColor = vec4(color,1.0);
    }
    ```
    

### Scale

- 다음 함수를 통해서 스케일 행렬을 만들 수 있다. 크기 조절도 회전과 똑같이 원점을 맞춰주는 작업을 해야한다.
    
    ```glsl
    mat2 scale(vec2 _scale){
        return mat2(_scale.x,0.0,
                    0.0,_scale.y);
    }
    ```
    
- 회전행렬과 스케일행렬 같이 써보기
    - 변환행렬을 쓸 때는 순서에 주의해야한다. 변환행렬의 순서는 SRT(ScaleRotationTranslate)인데 행렬은 오른쪽부터 적용이라 반대로 사용하면 된다.
        - 만약 회전을 먼저하고 크기를 나중에 하면 회전된 축 방향으로 크기가 조절되게 된다(대부분 비스듬하게 찌그러짐).
    - 그리고 행렬끼리 곱한거에 벡터를 곱하면 된다.
    
    ```glsl
    void main(){
        vec2 st = gl_FragCoord.xy/u_resolution.xy;
        vec3 color = vec3(0.0);
    
        st -= vec2(0.5);
        st = rotate2d( sin(u_time)*PI ) * scale( vec2(sin(u_time)+1.0) ) * st;
        st += vec2(0.5);
    
        // Show the coordinates of the space on the background
        // color = vec3(st.x,st.y,0.0);
    
        // Add the shape on the foreground
        color += vec3(cross(st,0.2));
    
        gl_FragColor = vec4(color,1.0);
    }
    ```
    

---

- 행렬쪽은 원래 알고있던 지식이 있어서 쉽게 끝난 것 같다.
