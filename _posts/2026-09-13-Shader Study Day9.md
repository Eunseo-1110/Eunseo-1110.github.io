---
title: "Cellular Noise"
date: 2026-09-13 00:00:00 +0900
categories: [Shader, The book of shaders]
tags: [shader, study]     # TAG names should always be lowercase
---

- CellularNoise를 만들기 위해서는 우선 알아야하는게 반복이다. 
GLSL에서 for을 사용할 때 비교하는 숫자는 상수여야한다. 동적 루프를 사용해선 안되며, 반복 되는 숫자는 고정되어 있어야한다.

## Points for a distance field

- Cellular Noise는 거리장을 베이스로 한다.
- 아래 코드는 몇 개의 기준점을 두고, 화면의 지점마다 가장 가까운 점의 거리를 계산하여 거리를 얻는다. 그리고 그 거리를 통해서 기준점을 기준으로 화면을 쪼개는 코드다.

```glsl
// Author: @patriciogv
// Title: 4 cells DF

#ifdef GL_ES
precision mediump float;
#endif

uniform vec2 u_resolution;
uniform vec2 u_mouse;
uniform float u_time;

void main() {
    vec2 st = gl_FragCoord.xy/u_resolution.xy;
    st.x *= u_resolution.x/u_resolution.y;

    vec3 color = vec3(.0);

    // Cell positions
    vec2 point[5];
    point[0] = vec2(0.83,0.75);
    point[1] = vec2(0.60,0.07);
    point[2] = vec2(0.28,0.64);
    point[3] =  vec2(0.31,0.26);
    point[4] = u_mouse/u_resolution;

    float m_dist = 1.;  // minimum distance

    // Iterate through the points positions
    for (int i = 0; i < 5; i++) {
        float dist = distance(st, point[i]);

        // Keep the closer distance
        m_dist = min(m_dist, dist);
    }

    // Draw the min distance (distance field)
    color += m_dist;

    // Show isolines
    // color -= step(.7,abs(sin(50.0*m_dist)))*.3;

    gl_FragColor = vec4(color,1.0);
}

```

## Tiling and iteration

- 반복문과 배열은 GLSL에서 좋지 않다. 많은 인스턴스에서 반복문을 사용하면 쉐이더의 성능이 크게 저하된다. 이 말은 우리는 위 코드에서 사용한 접근법을 많은 양의 포인트에서는 사용할 수 없다. 다른 방법을 찾아야하는데, 하나는 GPU의 병렬처리를 사용하는 것이다.
- 방법 중 하나는 공간을 타일형태로 나누는 것이다. 모든 픽셀이 한 점까지의 거리를 계산할 필요는 없다.
- 셀사이의 경계에서의 이상현상을 피하기 위해 우리는 인접한 셀의 거리를 확인해야한다.
- 각 픽셀은 9개의 위치를 확인해야하는데, 자기 셀의 점과 그 주변의 8개의 점이다.
- vec2랜덤 포인트를 만든다. 각 타일마다 랜덤한 위치의 포인트를 만들어서, 각 픽셀은 타일안에서 거리가 랜덤포인트로부터 얼마나 되는지 체크한다.
- 그리고 우리는 인접 타일에서 의 거리도 알아야하기 때문에 여기서 반복문을 쓸거다. 인접타일은 x축의 -1(left), 1(right)과 그리고 y축의 -1(b),1(t)로, 3x3 9개의 타일로 이루어진 타일에 대해서 반복문을 돌린다.
- 이제 인접 타일의 랜덤 위치를 얻고싶으면 랜덤에 타일과 반복문으로 만든 xy좌표를 넣으면 된다.
- 이제 해당지점 까지의 거리를 계산하고 가장 가까운 거리를 얻으면 된다.

- 위에서 설명한 것을 구현한 코드다.
    
    ```glsl
    // Author: @patriciogv
    // Title: CellularNoise
    
    #ifdef GL_ES
    precision mediump float;
    #endif
    
    uniform vec2 u_resolution;
    uniform vec2 u_mouse;
    uniform float u_time;
    
    vec2 random2( vec2 p ) {
        return fract(sin(vec2(dot(p,vec2(127.1,311.7)),dot(p,vec2(269.5,183.3))))*43758.5453);
    }
    
    void main() {
        vec2 st = gl_FragCoord.xy/u_resolution.xy;
        st.x *= u_resolution.x/u_resolution.y;
        vec3 color = vec3(.0);
    
        // Scale
        st *= 3.;
    
        // Tile the space
        vec2 i_st = floor(st);
        vec2 f_st = fract(st);
    
        float m_dist = 1.;  // minimum distance
    
        for (int y= -1; y <= 1; y++) {
            for (int x= -1; x <= 1; x++) {
                // Neighbor place in the grid
                vec2 neighbor = vec2(float(x),float(y));
    
                // Random position from current + neighbor place in the grid
                vec2 point = random2(i_st + neighbor);
    
                // Animate the point
                point = 0.5 + 0.5*sin(u_time + 6.2831*point);
    
                // Vector between the pixel and the point
                vec2 diff = neighbor + point - f_st;
    
                // Distance to the point
                float dist = length(diff);
    
                // Keep the closer distance
                m_dist = min(m_dist, dist);
            }
        }
    
        // Draw the min distance (distance field)
        color += m_dist;
    
        // Draw cell center
        color += 1.-step(.02, m_dist);
    
        // Draw grid
        color.r += step(.98, f_st.x) + step(.98, f_st.y);
    
        // Show isolines
        // color -= step(.7,abs(sin(27.0*m_dist)))*.5;
    
        gl_FragColor = vec4(color,1.0);
    }
    
    ```
    
- 예시코드를 보고 경계선을 강조하는 듯한 느낌으로 만들어 보았다. (주석으로 다른 거리 실험도 해봤다)
    - 2번째로 작은 거리를 구해서 그 값에서 가장 작은 거리를 뺀 것.
    - 이렇게하면 셀의 정중앙으로 갈 수록 값이 커지고 셀 경계선 근처로 갈수록 값이 작아진다.
    - 중앙으로 갈수록 가장 작은 거리(자기 점까지의 거리)는 아주 가깝지만 2등 거리(두 번째로 가까운 다른 셀의 점까지의 거리)는 다른 셀 점이니까 거리가 멀게 나온다.
    
    <canvas id="tiling and iteration edge" style="width:100%; max-width:300px; aspect-ratio:1/1; display:block; margin:20px auto; background:#222;"></canvas>

    {% include glsl-boilerplate.html %}

    <script>
    window.runGLSL("tiling and iteration edge", `
        precision mediump float;uniform vec2 u_resolution;uniform vec2 u_mouse;uniform float u_time;vec2 random2( vec2 p ) {    return fract(sin(vec2(dot(p,vec2(127.1,311.7)),dot(p,vec2(269.5,183.3))))*43758.5453);}void main() {    vec2 st = gl_FragCoord.xy/u_resolution.xy;    st.x *= u_resolution.x/u_resolution.y;    vec3 color = vec3(.0);     st *= 3.;    vec2 i_st = floor(st);    vec2 f_st = fract(st);    float m_dist = 1.;      float m_dist2 = 1.;    for (int y= -1; y <= 1; y++) {        for (int x= -1; x <= 1; x++) {            vec2 neighbor = vec2(float(x),float(y));            vec2 point = random2(i_st + neighbor);            point = 0.5 + 0.5*sin(u_time + 6.2831*point);            vec2 diff = neighbor + point - f_st;            float dist = length(diff);            float new_dist = min(m_dist, dist);            if (new_dist < m_dist)           {            	m_dist2 = m_dist;	                }           m_dist = new_dist;        }    }    color += 1.0 - (m_dist2 - m_dist);    color += 1.-step(.02, m_dist);    color.r += step(.98, f_st.x) + step(.98, f_st.y);    gl_FragColor = vec4(color,1.0);}
    `);
    </script>


    ```glsl
    // Author: @patriciogv
    // Title: CellularNoise
    
    #ifdef GL_ES
    precision mediump float;
    #endif
    
    uniform vec2 u_resolution;
    uniform vec2 u_mouse;
    uniform float u_time;
    
    vec2 random2( vec2 p ) {
        return fract(sin(vec2(dot(p,vec2(127.1,311.7)),dot(p,vec2(269.5,183.3))))*43758.5453);
    }
    
    void main() {
        vec2 st = gl_FragCoord.xy/u_resolution.xy;
        st.x *= u_resolution.x/u_resolution.y;
        vec3 color = vec3(.0);
    
        // Scale
        st *= 3.;
    
        // Tile the space
        vec2 i_st = floor(st);
        vec2 f_st = fract(st);
    
        float m_dist = 1.;  // minimum distance
        float m_dist2 = 1.;  // minimum distance
    
        for (int y= -1; y <= 1; y++) {
            for (int x= -1; x <= 1; x++) {
                // Neighbor place in the grid
                vec2 neighbor = vec2(float(x),float(y));
    
                // Random position from current + neighbor place in the grid
                vec2 point = random2(i_st + neighbor);
    
                // Animate the point
                point = 0.5 + 0.5*sin(u_time + 6.2831*point);
    
                // Vector between the pixel and the point
                vec2 diff = neighbor + point - f_st;
    
                // Distance to the point
                float dist = length(diff);
                // Chebyshev distance
                //dist = max(abs(diff.x), abs(diff.y));
                // Manhattan distance
                //dist = abs(diff.x)+ abs(diff.y);
                
                // Keep the closer distance
                float new_dist = min(m_dist, dist);
                if (new_dist < m_dist)
                {
                    m_dist2 = m_dist;	    
                }
                m_dist = new_dist;
            }
        }
    
        // Draw the min distance (distance field)
        color += 1.0 - (m_dist2 - m_dist);
    
        // Draw cell center
        color += 1.-step(.02, m_dist);
    
        // Draw grid
        color.r += step(.98, f_st.x) + step(.98, f_st.y);
    
        // Show isolines
        // color -= step(.7,abs(sin(27.0*m_dist)))*.5;
    
        gl_FragColor = vec4(color,1.0);
    }
    
    ```
    

## Voronoi Algorithm

- 처음에 봤던 코드와 비슷하다. if문을 통해서 거리에서 추가로 어느 점이었는지도 기억하는 코드가 추가 되었다.
    - 이전 코드에서는 `m_dist = min(m_dist, dist);` 를 통해서 가장 작은 거리값을 갱신했었다. 이번에는 if문으로 바꾸고 그 거리가 어디 점이었는지(`m_point` )도 저장한다.
    - 그리고 그 `m_point` 를 color r,g채널에 넣어서 영역마다 색을 칠하고 있다.
    
    ```glsl
    // Author: @patriciogv
    // Title: 4 cells voronoi
    
    #ifdef GL_ES
    precision mediump float;
    #endif
    
    uniform vec2 u_resolution;
    uniform vec2 u_mouse;
    uniform float u_time;
    
    void main() {
        vec2 st = gl_FragCoord.xy/u_resolution.xy;
        st.x *= u_resolution.x/u_resolution.y;
    
        vec3 color = vec3(.0);
    
        // Cell positions
        vec2 point[5];
        point[0] = vec2(0.83,0.75);
        point[1] = vec2(0.60,0.07);
        point[2] = vec2(0.28,0.64);
        point[3] =  vec2(0.31,0.26);
        point[4] = u_mouse/u_resolution;
    
        float m_dist = 1.;  // minimum distance
        vec2 m_point;        // minimum position
    
        // Iterate through the points positions
        for (int i = 0; i < 5; i++) {
            float dist = distance(st, point[i]);
            if ( dist < m_dist ) {
                // Keep the closer distance
                m_dist = dist;
    
                // Kepp the position of the closer point
                m_point = point[i];
            }
        }
    
        // Add distance field to closest point center
        color += m_dist*2.;
    
        // tint acording the closest point position
        color.rg = m_point;
    
        // Show isolines
        color -= abs(sin(80.0*m_dist))*0.07;
    
        // Draw point center
        color += 1.-step(.02, m_dist);
    
        gl_FragColor = vec4(color,1.0);
    }
    
    ```