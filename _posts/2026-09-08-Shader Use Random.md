---
title: "Use Random"
date: 2026-09-08 00:00:00 +0900
categories: [Shader, The book of shaders]
tags: [shader, study]     # TAG names should always be lowercase
---

## 랜덤 사용해보기

- 랜덤 챕터의 예제 코드와 화면으로 랜덤한 가로줄이 랜덤한 색상으로 랜덤 속도로 이동하는 화면을 구성해봤다.
    - 만들면서 제일 문제였던게 초반에는 선이 잘 움직이다가 어느 순간부터 검은색(초반에 테스트는 배경이 검은색이였음)으로 나오는 문제가 있었다.
        - 이유는 선을 움직이려고 `t = u_time ...` 이 값은 계속 커지는데 랜덤함수에 들어가는 값이 이 값과 `st.x` 를 합해서 정수로 만들기 때문이였다. 값이 커지는데 랜덤함수에서 이 값을 또 엄청 큰 값을 곱하면서 값이 엄청커져서 float정밀도가 무너지는 이유였다.
        - 그래서 처음엔 `u_time` 을 쓰는 쪽에서 `mod` 를 적용했었다. 근데 이러니까 중간중간 mod로 값이 초기화될 때 이동이 튀는 문제가 생겼었다. 그래서 mod를 `u_time` 에 말고 `line`함수의 `floor(st.x + t)` 부분에 mod를 사용하도록 바꾸었다. 랜덤에 들어가는 함수의 값만 줄이면 정밀도 문제를 좀 더 나중으로 미룰 수 있으니까.
    
    <canvas id="random-line" style="width:100%; max-width:300px; aspect-ratio:1/1; display:block; margin:20px auto; background:#222;"></canvas>

    {% include glsl-boilerplate.html %}

    <script>
    window.runGLSL("random-line", `
        #ifdef GL_ES
        precision mediump float;
        #endif
        
        #define PI 3.14159265358979323846

        uniform vec2 u_resolution;
        uniform vec2 u_mouse;
        uniform float u_time;

        vec3 hsb2rgb( in vec3 c ){
            vec3 rgb = clamp(abs(mod(c.x*6.0+vec3(0.0,4.0,2.0),
                                    6.0)-3.0)-1.0,
                            0.0,
                            1.0 );
            rgb = rgb*rgb*(3.0-2.0*rgb);
            return c.z * mix(vec3(1.0), rgb, c.y);
        }
        
        float random (in float v) {
            return fract(sin(v * 10000.0));
        }
        
        float random (in vec2 _st) {
            return fract(sin(dot(_st.xy,
                                vec2(12.9898,78.233)))*
                43758.5453123);
        }
        
        float line(vec2 st, float r, float t)
        {
            float p = floor(st.x + t);
            return step(r, random(mod(p, 50.0)));
        }
        
        void main()
        {
            vec2 st = gl_FragCoord.xy/u_resolution.xy;
            st *= vec2(10.0, 50.0);
        
            // 격자 나누기
            vec2 ipos = floor(st);  // integer
            vec2 fpos = vec2(st);  // fraction
        
            float randY = random(1.0 + ipos.y);
            float t = u_time * 10.0 * randY;
                
            vec3 lineColor = hsb2rgb(vec3(random(1.25 + ipos.y), 1.0, 1.0));
            vec3 color = vec3(0.0);
            color = mix(vec3(1.0), lineColor, line(vec2(st.x, st.y), random(vec2(ipos.y, 12.12)), -t));
            
            gl_FragColor = vec4(color,1.0);
        }
    `);
    </script>

    ```glsl
    #define PI 3.14159265358979323846
    
    vec3 hsb2rgb( in vec3 c ){
        vec3 rgb = clamp(abs(mod(c.x*6.0+vec3(0.0,4.0,2.0),
                                 6.0)-3.0)-1.0,
                         0.0,
                         1.0 );
        rgb = rgb*rgb*(3.0-2.0*rgb);
        return c.z * mix(vec3(1.0), rgb, c.y);
    }
    
    float random (in float v) {
        return fract(sin(v * 10000.0));
    }
    
    float random (in vec2 _st) {
        return fract(sin(dot(_st.xy,
                             vec2(12.9898,78.233)))*
            43758.5453123);
    }
    
    float line(vec2 st, float r, float t)
    {
        float p = floor(st.x + t);
        return step(r, random(mod(p, 50.0)));
    }
    
    void mainImage( out vec4 fragColor, in vec2 fragCoord )
    {
        vec2 st = fragCoord.xy/iResolution.xy;
        st *= vec2(10.0, 50.0);
    
        // 격자 나누기
        vec2 ipos = floor(st);  // integer
        vec2 fpos = vec2(st);  // fraction
    
        float randY = random(1.0 + ipos.y);
        float t = iTime * 10.0 * randY;
    		
        vec3 lineColor = hsb2rgb(vec3(random(1.25 + ipos.y), 1.0, 1.0));
        vec3 color = vec3(0.0);
        color = mix(vec3(1.0), lineColor, line(vec2(st.x, st.y), random(vec2(ipos.y, 12.12)), -t));
        
        fragColor = vec4(color,1.0);
    }
    ```