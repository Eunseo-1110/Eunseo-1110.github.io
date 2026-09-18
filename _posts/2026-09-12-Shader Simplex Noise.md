---
title: "Simplex Noise 분석"
date: 2026-09-12 12:00:00 +0900
categories: [Shader, The book of shaders]
tags: [shader, study]     # TAG names should always be lowercase
---

- `mod289, permute` : Random()을 대체하는 GPU최적화 해시 함수
    - `sin()` 대신 곱셈과 덧셈만으로 계산하게 되어있다. sin은 GPU에 따라 입력이 커지면 정밀도가 불안해질 수 있는데 다항식 기반으로 하면 값을 항상 0~289사이로 만들기 때문에 안정적이다.
- `const vec4 C` : 이 전의 skew함수의 `1.1547`(=`2/√3` ) 와 같은 계열의 숫자를 가지고 있다. 사각형을 정삼각형이 나오도록 기울이는 비율을 미리 계산해서 상수로 만들어 둔 것.
- `i, x0` : i는 몇 번째 칸인가, x0은 그 칸의 첫 모서리에서 지금 점까지의 벡터다. Gradient Noise에서 `f - vec2(0,0)` 로 구했던 모서리 → 점 벡터과 같은 개념
- `i1, x1, x2` : 삼각형의 나머지 두 모서리 찾기. i1은 위/아래 삼각형 판별, x1와 x2는 두번째,세번째 모서리에서 지금 점까지의 벡터다.
- `p` : 각 모서리에 무작위 값을 세팅한다.
- `x, h, ox, a0` : 무작위 값으로 부터 gradient벡터 만들기. `p` 로부터 41가지 방향 중 하나를 뽑아낸다.
- `m` : 각 모서리에서 멀어질수록, 그 모서리의 영향력이 원형으로 부드럽게 줄어들다가 특정 거리를 넘으면 0이 된다. `dot(x0,x0)` 은 sqrt를 생략하지 위한 거리의 제곱이고, 이 0.5-$거리^2$를 해서 가까울수록 양수, 멀수록 음수가 되게 만든다. 그리고 이걸 max를 통해서 음수를 잘라낸다.
그리고 m을 제곱을 2번하여 경계가 아주 부드럽에서 0에서 사그라들도록 만든다.
- `g` : gradient와 점 사이의 내적.
`g.x` 는 전에 봤던 dot(gradient벡터, 모서리→점벡터)를 풀어서 쓴 것. `a0`,`h` 로 gradient벡터의 x,y역할을 하고 x0과 내적하는 것이다.
- `130.0*dot(m, g)` : m(각 모서리에서 얼마나 가까운지 가중치)와 g(각 모서리의 gradient 내적값)를 내적하여 세 모서리의 기여도를 합산한다. 130.0은 결과값을 -1~1범위로 맞추기 위한 보정 상수다.

<canvas id="simplex noise Ashima Arts" style="width:100%; max-width:300px; aspect-ratio:1/1; display:block; margin:20px auto; background:#222;"></canvas>

{% include glsl-boilerplate.html %}

<script>
window.runGLSL("simplex noise Ashima Arts", `
    precision mediump float;uniform vec2 u_resolution;uniform vec2 u_mouse;uniform float u_time;vec3 mod289(vec3 x) { return x - floor(x * (1.0 / 289.0)) * 289.0; }vec2 mod289(vec2 x) { return x - floor(x * (1.0 / 289.0)) * 289.0; }vec3 permute(vec3 x) { return mod289(((x*34.0)+1.0)*x); }float snoise(vec2 v) {    const vec4 C = vec4(0.211324865405187,                        0.366025403784439,                        -0.577350269189626,                        0.024390243902439);    vec2 i  = floor(v + dot(v, C.yy));    vec2 x0 = v - i + dot(i, C.xx);    vec2 i1 = vec2(0.0);    i1 = (x0.x > x0.y)? vec2(1.0, 0.0):vec2(0.0, 1.0);    vec2 x1 = x0.xy + C.xx - i1;    vec2 x2 = x0.xy + C.zz;    i = mod289(i);    vec3 p = permute(            permute( i.y + vec3(0.0, i1.y, 1.0))                + i.x + vec3(0.0, i1.x, 1.0 ));    vec3 m = max(0.5 - vec3(                        dot(x0,x0),                        dot(x1,x1),                        dot(x2,x2)                        ), 0.0);    m = m*m ;    m = m*m ;    vec3 x = 2.0 * fract(p * C.www) - 1.0;    vec3 h = abs(x) - 0.5;    vec3 ox = floor(x + 0.5);    vec3 a0 = x - ox;    m *= 1.79284291400159 - 0.85373472095314 * (a0*a0+h*h);    vec3 g = vec3(0.0);    g.x  = a0.x  * x0.x  + h.x  * x0.y;    g.yz = a0.yz * vec2(x1.x,x2.x) + h.yz * vec2(x1.y,x2.y);    return 130.0 * dot(m, g);}void main() {    vec2 st = gl_FragCoord.xy/u_resolution.xy;    st.x *= u_resolution.x/u_resolution.y;    vec3 color = vec3(0.0);    st *= 10.;    color = vec3(snoise(st)*.5+.5);    gl_FragColor = vec4(color,1.0);}
`);
</script>

```glsl
#ifdef GL_ES
precision mediump float;
#endif

uniform vec2 u_resolution;
uniform vec2 u_mouse;
uniform float u_time;

// Some useful functions
vec3 mod289(vec3 x) { return x - floor(x * (1.0 / 289.0)) * 289.0; }
vec2 mod289(vec2 x) { return x - floor(x * (1.0 / 289.0)) * 289.0; }
vec3 permute(vec3 x) { return mod289(((x*34.0)+1.0)*x); }

//
// Description : GLSL 2D simplex noise function
//      Author : Ian McEwan, Ashima Arts
//  Maintainer : ijm
//     Lastmod : 20110822 (ijm)
//     License :
//  Copyright (C) 2011 Ashima Arts. All rights reserved.
//  Distributed under the MIT License. See LICENSE file.
//  https://github.com/ashima/webgl-noise
//
float snoise(vec2 v) {

    // Precompute values for skewed triangular grid
    const vec4 C = vec4(0.211324865405187,
                        // (3.0-sqrt(3.0))/6.0
                        0.366025403784439,
                        // 0.5*(sqrt(3.0)-1.0)
                        -0.577350269189626,
                        // -1.0 + 2.0 * C.x
                        0.024390243902439);
                        // 1.0 / 41.0

    // First corner (x0)
    vec2 i  = floor(v + dot(v, C.yy));
    vec2 x0 = v - i + dot(i, C.xx);

    // Other two corners (x1, x2)
    vec2 i1 = vec2(0.0);
    i1 = (x0.x > x0.y)? vec2(1.0, 0.0):vec2(0.0, 1.0);
    vec2 x1 = x0.xy + C.xx - i1;
    vec2 x2 = x0.xy + C.zz;

    // Do some permutations to avoid
    // truncation effects in permutation
    i = mod289(i);
    vec3 p = permute(
            permute( i.y + vec3(0.0, i1.y, 1.0))
                + i.x + vec3(0.0, i1.x, 1.0 ));

    vec3 m = max(0.5 - vec3(
                        dot(x0,x0),
                        dot(x1,x1),
                        dot(x2,x2)
                        ), 0.0);

    m = m*m ;
    m = m*m ;

    // Gradients:
    //  41 pts uniformly over a line, mapped onto a diamond
    //  The ring size 17*17 = 289 is close to a multiple
    //      of 41 (41*7 = 287)

    vec3 x = 2.0 * fract(p * C.www) - 1.0;
    vec3 h = abs(x) - 0.5;
    vec3 ox = floor(x + 0.5);
    vec3 a0 = x - ox;

    // Normalise gradients implicitly by scaling m
    // Approximation of: m *= inversesqrt(a0*a0 + h*h);
    m *= 1.79284291400159 - 0.85373472095314 * (a0*a0+h*h);

    // Compute final noise value at P
    vec3 g = vec3(0.0);
    g.x  = a0.x  * x0.x  + h.x  * x0.y;
    g.yz = a0.yz * vec2(x1.x,x2.x) + h.yz * vec2(x1.y,x2.y);
    return 130.0 * dot(m, g);
}

void main() {
    vec2 st = gl_FragCoord.xy/u_resolution.xy;
    st.x *= u_resolution.x/u_resolution.y;

    vec3 color = vec3(0.0);

    // Scale the space in order to see the function
    st *= 10.;

    color = vec3(snoise(st)*.5+.5);

    gl_FragColor = vec4(color,1.0);
}

```