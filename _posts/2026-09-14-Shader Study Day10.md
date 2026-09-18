---
title: "Fractal Brownian Motion"
date: 2026-09-14 00:00:00 +0900
categories: [Shader, The book of shaders]
tags: [shader, study]     # TAG names should always be lowercase
---

## Fractal Brownian Motion
- 진폭과 주파수를 이용한 간단한 1차원 웨이브 식
    
    ```glsl
    float amplitude = 1.;
    float frequency = 1.;
    y = amplitude * sin(x * frequency);
    ```
    
- 웨이브는 또다른 웨이브를 만들어 더할 수 있다. 이를 superpotion이라고 부른다. 다음 코드는 여러 개의 sin 웨이브를 겹치는 코드다.
    - 3번째 줄까지가 위의 코드에서 본 기본 웨이브
    - `y+=...` 코드부터가 웨이브에 파동을 계속 더하는 코드이다.
        - 주파수 배율: `*2.1`, `*1.72`, `*2.221`, `*3.1122` 이 값을 통해서 웨이브의 각 주파수를 다르게 만든다. 웨이브의 간격이 달라진다.
        - 진폭: `*4.5`, `*4.0`, `*5.0`, `*2.5` 이 값을 통해서 각 웨이브마다 높이를 다르게 한다.
        - 시간 배율: `t`, `t*1.121`, `t*0.437`, `t*4.269` 이 값을 통해서 각 웨이브마다 서로 다른 속도로 움직이게 한다.
    - `y *= amplitude*0.06` 최종 크기를 재조정해준다. 위 진폭들을 그대로 사용하면 크기가 커져 표현하기 어려울 수 있다.
    
    ```glsl
    float amplitude = 1.;
    float frequency = 1.;
    y = sin(x * frequency);
    float t = 0.01*(-u_time*130.0);
    y += sin(x*frequency*2.1 + t)*4.5;
    y += sin(x*frequency*1.72 + t*1.121)*4.0;
    y += sin(x*frequency*2.221 + t*0.437)*5.0;
    y += sin(x*frequency*3.1122+ t*4.269)*2.5;
    y *= amplitude*0.06;
    ```
    
- 이제 sin대신 노이즈를 사용해 보자. 노이즈의 무작위성을 이용하여 규칙적인 sin대신 더 다양한 웨이브를 얻을 수 있다.
    - 노이즈의 반복을 더하고(octave), 일정한 간격으로 주파수를 연속적으로 증가 시키고(lacunarity), 진폭을 감소시킨다(gain). 이러한 테크닉을 **fractal Brownian Motion(fBM)**이라고 한다.
    
    ```glsl
    // Properties
    const int octaves = 1;
    float lacunarity = 2.0;
    float gain = 0.5;
    //
    // Initial values
    float amplitude = 0.5;
    float frequency = 1.;
    //
    // Loop of octaves
    for (int i = 0; i < octaves; i++) {
        y += amplitude * noise(frequency*x);
        frequency *= lacunarity;
        amplitude *= gain;
    }
    ```
    
- 위 기법을 사용한 예제를 보자.
    - `value += amplitude * noise(st);` : 현재 진폭만큼 노이즈를 더한다.
    - `st *= 2.0;` : 다음 겹을 위해 좌표를 2배 확대한다. 위의 `frequency *= lacunarity;` 코드와 같은 부분이다.
    - `amplitude *= 0.5;` : 다음 겹을 위해 진폭을 감소 시킨다. 위의 `*= gain`과 같은 부분이다.
    - 반복마다 좌표를 확대하고 영향력은 줄어든다. 
    첫번째 반복(큰 진폭, 낮은 주파수)로 전체적인 흐름과 윤곽을 결정하고, 뒤로 갈수록(작은 진폭, 높은 주파수) 자잘한 디테일을 더함.
    - 구름같은 텍스쳐가 만들어진다.
    
    ```glsl
    // Author @patriciogv - 2015
    // http://patriciogonzalezvivo.com
    
    #ifdef GL_ES
    precision mediump float;
    #endif
    
    uniform vec2 u_resolution;
    uniform vec2 u_mouse;
    uniform float u_time;
    
    float random (in vec2 st) {
        return fract(sin(dot(st.xy,
                                vec2(12.9898,78.233)))*
            43758.5453123);
    }
    
    // Based on Morgan McGuire @morgan3d
    // https://www.shadertoy.com/view/4dS3Wd
    float noise (in vec2 st) {
        vec2 i = floor(st);
        vec2 f = fract(st);
    
        // Four corners in 2D of a tile
        float a = random(i);
        float b = random(i + vec2(1.0, 0.0));
        float c = random(i + vec2(0.0, 1.0));
        float d = random(i + vec2(1.0, 1.0));
    
        vec2 u = f * f * (3.0 - 2.0 * f);
    
        return mix(a, b, u.x) +
                (c - a)* u.y * (1.0 - u.x) +
                (d - b) * u.x * u.y;
    }
    
    #define OCTAVES 6
    float fbm (in vec2 st) {
        // Initial values
        float value = 0.0;
        float amplitude = .5;
        float frequency = 0.;
        //
        // Loop of octaves
        for (int i = 0; i < OCTAVES; i++) {
            value += amplitude * noise(st);
            st *= 2.;
            amplitude *= .5;
        }
        return value;
    }
    
    void main() {
        vec2 st = gl_FragCoord.xy/u_resolution.xy;
        st.x *= u_resolution.x/u_resolution.y;
    
        vec3 color = vec3(0.0);
        color += fbm(st*3.0);
    
        gl_FragColor = vec4(color,1.0);
    }
    
    ```
    
    - 이러한 기법은 절차적인 landscapes만들기에 사용된다. fBM은 산맥이 침식되는 자기유사성과 비슷하게 이루어져 있다.

### Turbulence

- 비슷한 테크닉으로, 기본적으로 fBM이지만 부호있는 노이즈를 이용해 샤프한 느낌을 내는 방법이다.
    - simple noise는 값을 -1~1사이를 반환한다. 그리고 이 값을 abs를 통해 음수였던 부분을 양수로 만든다. 음수였던 부분을 양수로 만들면 방향이 갑자기 꺾이는 듯한 느낌을 낼 수 있다(sin을 abs로 감싸면 튀어오르는 느낌을 주는 것 처럼).
    
    ```glsl
    for (int i = 0; i < OCTAVES; i++) {
        value += amplitude * abs(snoise(st));
        st *= 2.;
        amplitude *= .5;
    }
    ```
    

### Ridge

- 또다른 비슷한 테크닉으로, 급하게 꺾이는 지점을 뒤집어서 더 날카로운 ridges를 만드는 방법이다.
    - `h = abs(h);` : 전에 `abs(snoise(…))` 하던 부분이랑 같은 부분이다.
    - `h = offset - h;` : 위아래를 뒤집는다. 가장 높은 지점을 뒤집는 것.
    - `h = h * h;` : 더 날카롭게 만드는 부분. 제곱해서 대비를 더 높이는 것과 같다.
    - `sum += n*amp*prev;` : prev에 이전 루프에서 계산된 ridge값을 저장한다. 그리고 이번 루프에서 이전 루프가 얼마나 강했는지에 따라 다시 한번 곱해주는 것이다. 이렇게하면 이전 루프에서 강했던 ridge는 이번 루프에서 더 세게 더해지고, 반대면 더 약하게 된다.
    
    ```glsl
    // Author: @patriciogv - 2015
    // Tittle: Ridge
    
    // Ridged multifractal
    // See "Texturing & Modeling, A Procedural Approach", Chapter 12
    float ridge(float h, float offset) {
        h = abs(h);     // create creases
        h = offset - h; // invert so creases are at top
        h = h * h;      // sharpen creases
        return h;
    }
    
    float ridgedMF(vec2 p) {
        float lacunarity = 2.0;
        float gain = 0.5;
        float offset = 0.9;
    
        float sum = 0.0;
        float freq = 1.0, amp = 0.5;
        float prev = 1.0;
        for(int i=0; i < OCTAVES; i++) {
            float n = ridge(snoise(p*freq), offset);
            sum += n*amp;
            sum += n*amp*prev;  // scale by previous octave
            prev = n;
            freq *= lacunarity;
            amp *= gain;
        }
        return sum;
    }
    ```
    

## Domain Warping

- 아래 코드는 구름같은 질감을 만들어내는 코드다.
- fbm함수에서 보면 `_st = rot * _st * 2.0 + shift;` 이 부분이 추가되었다. 이전에는 매 루프마다 확대(`st *= 2.0`)를 하였지만, 여기서는 2x2회전 행렬을 통해서 루프마다 회전도 해주고 있다. 그리고 shift를 더해 루프마다 다른 영역의 노이즈가 참조되도록 하여있다. 이 부분은 패턴이 겹치는 것을 방지하는 장치인 것.
- main함수의 `q` 를 보면  fbm함수를 2번 불러서 x, y에 각각 다른 노이즈를 넣어주고 있다.
- main함수의 r을 보면 여기서는 이전에 만든 `q` 를 가지고 st와 더해주고 있다. 이전의 노이즈에서 좌표에 그라디언트를 더했던 것과 같은 방법이다. 좌표를 왜곡시키는 것.
- 그리고 `f` 에서 한 번 더 fbm함수를 통해서 `r`에서 만든 벡터를 왜곡시킨다.
- 그리고 mix를 통해서 색을 입혀주고 있다.
    - `color = mix(……, clamp((f*f)*4.0,0.0,1.0))`
    - `color = mix(….,clamp(length(q),0.0,1.0));` , `color = mix(…, clamp(length(r.x),0.0,1.0));` : `q`, `r` 벡터의 크기(length)를 사용해여 왜곡이 강했던 곳(소용돌이가 심한 곳)에 다른 색조를 씌운다.
- `vec4((f*f*f+.6*f*f+.5*f)*color,1.);` : f를 세제곱, 제곱, 기본 값을 섞어서 밝기를 조절하는 커브를 하나 더 만들어 색에 곱해준다.

    <canvas id="domain warping inigo quiles" style="width:100%; max-width:300px; aspect-ratio:1/1; display:block; margin:20px auto; background:#222;"></canvas>

    {% include glsl-boilerplate.html %}

    <script>
    window.runGLSL("domain warping inigo quiles", `
        precision mediump float;uniform vec2 u_resolution;uniform vec2 u_mouse;uniform float u_time;float random (in vec2 _st) {    return fract(sin(dot(_st.xy,                         vec2(12.9898,78.233)))*        43758.5453123);}float noise (in vec2 _st) {    vec2 i = floor(_st);    vec2 f = fract(_st);    float a = random(i);    float b = random(i + vec2(1.0, 0.0));    float c = random(i + vec2(0.0, 1.0));    float d = random(i + vec2(1.0, 1.0));    vec2 u = f * f * (3.0 - 2.0 * f);    return mix(a, b, u.x) +            (c - a)* u.y * (1.0 - u.x) +            (d - b) * u.x * u.y;}float fbm ( in vec2 _st) {    float v = 0.0;    float a = 0.5;    vec2 shift = vec2(100.0);    mat2 rot = mat2(cos(0.5), sin(0.5),                    -sin(0.5), cos(0.50));    for (int i = 0; i < 5; ++i) {        v += a * noise(_st);        _st = rot * _st * 2.0 + shift;        a *= 0.5;    }    return v;}void main() {    vec2 st = gl_FragCoord.xy/u_resolution.xy*3.;    vec3 color = vec3(0.0);    vec2 q = vec2(0.);    q.x = fbm( st + 0.00*u_time);    q.y = fbm( st + vec2(1.0));    vec2 r = vec2(0.);    r.x = fbm( st + 1.0*q + vec2(1.7,9.2)+ 0.15*u_time );    r.y = fbm( st + 1.0*q + vec2(8.3,2.8)+ 0.126*u_time);    float f = fbm(st+r);    color = mix(vec3(0.101961,0.619608,0.666667),                vec3(0.666667,0.666667,0.498039),                clamp((f*f)*4.0,0.0,1.0));    color = mix(color,               vec3(0,0,0.164706),                clamp(length(q),0.0,1.0));    color = mix(color,               vec3(0.666667,1,1),                clamp(length(r.x),0.0,1.0));    gl_FragColor = vec4((f*f*f+.6*f*f+.5*f)*color,1.);}
    `);
    </script>

```glsl
// Author @patriciogv - 2015
// http://patriciogonzalezvivo.com

#ifdef GL_ES
precision mediump float;
#endif

uniform vec2 u_resolution;
uniform vec2 u_mouse;
uniform float u_time;

float random (in vec2 _st) {
    return fract(sin(dot(_st.xy,
                            vec2(12.9898,78.233)))*
        43758.5453123);
}

// Based on Morgan McGuire @morgan3d
// https://www.shadertoy.com/view/4dS3Wd
float noise (in vec2 _st) {
    vec2 i = floor(_st);
    vec2 f = fract(_st);

    // Four corners in 2D of a tile
    float a = random(i);
    float b = random(i + vec2(1.0, 0.0));
    float c = random(i + vec2(0.0, 1.0));
    float d = random(i + vec2(1.0, 1.0));

    vec2 u = f * f * (3.0 - 2.0 * f);

    return mix(a, b, u.x) +
            (c - a)* u.y * (1.0 - u.x) +
            (d - b) * u.x * u.y;
}

#define NUM_OCTAVES 5

float fbm ( in vec2 _st) {
    float v = 0.0;
    float a = 0.5;
    vec2 shift = vec2(100.0);
    // Rotate to reduce axial bias
    mat2 rot = mat2(cos(0.5), sin(0.5),
                    -sin(0.5), cos(0.50));
    for (int i = 0; i < NUM_OCTAVES; ++i) {
        v += a * noise(_st);
        _st = rot * _st * 2.0 + shift;
        a *= 0.5;
    }
    return v;
}

void main() {
    vec2 st = gl_FragCoord.xy/u_resolution.xy*3.;
    // st += st * abs(sin(u_time*0.1)*3.0);
    vec3 color = vec3(0.0);

    vec2 q = vec2(0.);
    q.x = fbm( st + 0.00*u_time);
    q.y = fbm( st + vec2(1.0));

    vec2 r = vec2(0.);
    r.x = fbm( st + 1.0*q + vec2(1.7,9.2)+ 0.15*u_time );
    r.y = fbm( st + 1.0*q + vec2(8.3,2.8)+ 0.126*u_time);

    float f = fbm(st+r);

    color = mix(vec3(0.101961,0.619608,0.666667),
                vec3(0.666667,0.666667,0.498039),
                clamp((f*f)*4.0,0.0,1.0));

    color = mix(color,
                vec3(0,0,0.164706),
                clamp(length(q),0.0,1.0));

    color = mix(color,
                vec3(0.666667,1,1),
                clamp(length(r.x),0.0,1.0));

    gl_FragColor = vec4((f*f*f+.6*f*f+.5*f)*color,1.);
}

```