---
title: "Noise"
date: 2026-09-12 00:00:00 +0900
categories: [Shader, The book of shaders]
tags: [shader, study]     # TAG names should always be lowercase
---

- 노이즈를 만들기 위한 코드를 살펴보자
    - `y = rand(i)` : 이전 챕터에서 배웠던 rand에 정수를 넣으면 랜덤한 높이에서 1만큼 그려지는 값을 얻을 수 있다.
    - `y = mix(rand(i), rand(i + 1.0), f);` : 위처럼 만든 값을 mix를 통해서 보간하여 랜덤한 꺾인 그래프 같은 모습을 얻을 수 있다.
    - `y = mix(rand(i), rand(i + 1.0), smoothstep(0.,1.,f));` : smoothstep을 통해서 부드럽게 휘어진 그래프 같은 모습을 얻을 수 있다.
    
    ```glsl
    float i = floor(x);  // integer
    float f = fract(x);  // fraction
    y = rand(i); //rand() is described in the previous chapter
    y = mix(rand(i), rand(i + 1.0), f);
    y = mix(rand(i), rand(i + 1.0), smoothstep(0.,1.,f));
    ```
    
    - 커스텀 3차 곡선으로 보간할 수도 있다.
        
        ```glsl
        float u = f * f * (3.0 - 2.0 * f ); // custom cubic curve
        y = mix(rand(i), rand(i + 1.0), u); // using it in the interpolation
        ```
        

## 2D Noise

- 노이즈 보간은 각 점 사이를 보간해야한다. 2D에선 네 점. 3D에서는 8개의 코너가 있다(큐브 기준). 이러한 랜덤값을 보간하는 테크닉을 **value noise**라고 한다.
- 2D Noise 예시 코드를 살펴보자.
    - `vec2 pos = vec2(st*10.0);` : 일단 이 부분에서 좌표 공간을 스케일링하고 있다.
    - 그리고 위에서 본대로 `floor` , `fract` 함수를 통해서 좌표를 정수와 0~1사이의 타일링으로 만든다.
    - `random(i + vec2(…` : 이 부분은 칸의 꼭짓점 마다 무작위값을 뽑아내고 있다.
        - 왜 이렇게 하는가?  `rand(i)` 만 사용하면 다른 칸으로 넘어가는 순간 다른 무작위값을 만나 값이 끊긴다. 다른 랜덤값이 되는 것. 하지만 네 꼭짓점 모두 무작위값을 구해서 f값(좌표가 타일에서 몇 %에 위치해 있는지)를 mix의 얼마나 희석하는지 인자에 넣으면 각 꼭짓점이 보간되어 이어지는 값으로 만들 수 있다.
        결과적으로 한 칸안에서 a(0,0)→b(1, 0)→c(0,1)→d(1,1)이 보간되어 매끄럽게 변하는 것. 
        옆 칸으로 넘어가면 b→a로 넘어가는 것이기 때문에 넘어가도 끊기지 않고 이어지는 것이다. i가 `(1,1)→(2,1)`면, `넘어간 a(2,1)=원래b(1,1)+vec2(1,0)=(2,1)`
    - `mix(a, b, u.x) + …` : 이 부분은 네 꼭짓점을 보간하는 부분인데, 각 두 꼭짓점 사이를 x(가로)를 섞어서 값을 구하고 이 값에 세로 위치를 섞어서 최종값을 구하는 것.
    `mix(mix(a, b, u.x), mix(c, d, u.x), u.y);` : 이것과 똑같은 코드다.
    
    ```glsl
    #ifdef GL_ES
    precision mediump float;
    #endif
    
    uniform vec2 u_resolution;
    uniform vec2 u_mouse;
    uniform float u_time;
    
    // 2D Random
    float random (in vec2 st) {
        return fract(sin(dot(st.xy,
                                vec2(12.9898,78.233)))
                        * 43758.5453123);
    }
    
    // 2D Noise based on Morgan McGuire @morgan3d
    // https://www.shadertoy.com/view/4dS3Wd
    float noise (in vec2 st) {
        vec2 i = floor(st);
        vec2 f = fract(st);
    
        // Four corners in 2D of a tile
        float a = random(i);
        float b = random(i + vec2(1.0, 0.0));
        float c = random(i + vec2(0.0, 1.0));
        float d = random(i + vec2(1.0, 1.0));
    
        // Smooth Interpolation
    
        // Cubic Hermine Curve.  Same as SmoothStep()
        vec2 u = f*f*(3.0-2.0*f);
        // u = smoothstep(0.,1.,f);
    
        // Mix 4 coorners percentages
        return mix(a, b, u.x) +
                (c - a)* u.y * (1.0 - u.x) +
                (d - b) * u.x * u.y;
    }
    
    void main() {
        vec2 st = gl_FragCoord.xy/u_resolution.xy;
    
        // Scale the coordinate system to see
        // some noise in action
        vec2 pos = vec2(st*5.0);
        // Use the noise function
        float n = noise(pos);
    
        gl_FragColor = vec4(vec3(n), 1.0);
    }
    ```
    

### 노이즈의 그라디언트를 디스턴스 필드로 취급한다면? 무슨일이 일어날까

- 노이즈가 얼마나 가파르게 변하는지 계산해서, 그 값을 도형그릴 때 쓰던 거리처럼 취급해보자.
    - 노이즈의 경사가 급한 부분을 따라 자연스럽게 그려지는 선/무늬가 나올 것
    - `pos + gradient * 20.0;` : 좌표에 그라디언트를 더하면 그라디언트 방향으로 좌표를 밀 수 있다. 그라디언트가 강한 곳은 좌표가 더 많이 휘어지게 되는데 이러면 노이즈가 연기같은 느낌으로 왜곡이 된다.
    
    <div style="display: flex; justify-content: center; gap: 20px; flex-wrap: wrap; width: 100%;">
        <canvas id="noise gradient domain warping" style="width:100%; max-width:300px; aspect-ratio:1/1; display:block; margin:20px auto; background:#222;"></canvas>
        <canvas id="noise gradient" style="width:100%; max-width:300px; aspect-ratio:1/1; display:block; margin:20px auto; background:#222;"></canvas>
    </div>

    {% include glsl-boilerplate.html %}

    <script>
    window.runGLSL("noise gradient domain warping", `
        precision mediump float; uniform vec2 u_resolution; uniform vec2 u_mouse; uniform float u_time; float random (in vec2 st) { return fract(sin(dot(st.xy,                          vec2(12.9898,78.233)))* 43758.5453123);}float noise (in vec2 st) {vec2 i = floor(st);vec2 f = fract(st);float a = random(i);float b = random(i + vec2(1.0, 0.0));float c = random(i + vec2(0.0, 1.0));float d = random(i + vec2(1.0, 1.0));  vec2 u = f*f*(3.0-2.0*f);return mix(a, b, u.x) + (c - a)* u.y * (1.0 - u.x) + (d - b) * u.x * u.y; }void main() {vec2 st = gl_FragCoord.xy/u_resolution.xy;vec2 pos = vec2(st*5.0);float eps = 0.001;float gx = noise(pos + vec2(eps, 0.0)) - noise(pos);float gy = noise(pos + vec2(0.0, eps)) - noise(pos);vec2 gradient = vec2(gx, gy) / eps;float d = length(gradient);float angle = atan(gradient.y, gradient.x);vec3 color = vec3(0.0);color = vec3(fract(d * 10.0)); color = vec3(step(0.5, d)); color = vec3(step(sin(u_time)*0.5+0.5, d)); color = vec3(step(sin(u_time)*0.5+0.5, fract(d * 10.0)));    vec2 warped = pos + gradient * 20.0;	color = vec3(noise(warped));  gl_FragColor = vec4(color, 1.0);}
    `);
    </script>

    <script>
    window.runGLSL("noise gradient", `
        precision mediump float; uniform vec2 u_resolution; uniform vec2 u_mouse; uniform float u_time; float random (in vec2 st) { return fract(sin(dot(st.xy,                          vec2(12.9898,78.233)))* 43758.5453123);}float noise (in vec2 st) {vec2 i = floor(st);vec2 f = fract(st);float a = random(i);float b = random(i + vec2(1.0, 0.0));float c = random(i + vec2(0.0, 1.0));float d = random(i + vec2(1.0, 1.0));  vec2 u = f*f*(3.0-2.0*f);return mix(a, b, u.x) + (c - a)* u.y * (1.0 - u.x) + (d - b) * u.x * u.y; }void main() {vec2 st = gl_FragCoord.xy/u_resolution.xy;vec2 pos = vec2(st*5.0);float eps = 0.001;float gx = noise(pos + vec2(eps, 0.0)) - noise(pos);float gy = noise(pos + vec2(0.0, eps)) - noise(pos);vec2 gradient = vec2(gx, gy) / eps;float d = length(gradient);float angle = atan(gradient.y, gradient.x);vec3 color = vec3(0.0);color = vec3(fract(d * 10.0)); color = vec3(step(0.5, d)); color = vec3(step(sin(u_time)*0.5+0.5, d)); color = vec3(step(sin(u_time)*0.5+0.5, fract(d * 10.0)));    vec2 warped = pos + gradient * 20.0;	 gl_FragColor = vec4(color, 1.0);}
    `);
    </script>

    ```glsl
    void main() {
        vec2 st = gl_FragCoord.xy/u_resolution.xy;
    
        // Scale the coordinate system to see
        // some noise in action
        vec2 pos = vec2(st*5.0);
    
        float eps = 0.001;
        float gx = noise(pos + vec2(eps, 0.0)) - noise(pos);
        float gy = noise(pos + vec2(0.0, eps)) - noise(pos);
        vec2 gradient = vec2(gx, gy) / eps;
        
        float d = length(gradient);
        float angle = atan(gradient.y, gradient.x);
        
        vec3 color = vec3(0.0);
        color = vec3(fract(d * 10.0));
        //color = vec3(step(0.5, d));
        //color = vec3(step(sin(u_time)*0.5+0.5, d));
        //color = vec3(step(sin(u_time)*0.5+0.5, fract(d * 10.0)));
        //color = hsb2rgb(vec3((angle/TWO_PI) + 0.5, 1.0, 1.0));
        
        // domain warping
        vec2 warped = pos + gradient * 20.0;
        //color = vec3(noise(warped)); 
        
        gl_FragColor = vec4(color, 1.0);
    }
    ```
    

## Using Noise in Generative Designs

- 노이즈를 얻는 것에는 더 많은 방법이 있다.
- 1985년에 KenPerlin은 또다른 알고리즘이 구현되었는데 이를 **Gradient Noise** 라고 한다. 그는 값대신 랜덤 그라디언트를 보간했는데 이 그라디언트는 랜덤함수에서 2차원벡터로 반환되었다.
    - `random2(i + vec2(0.0,0.0))` : 모서리 마다 무작위 방향 벡터를 만든다
    - `f - vec2(0.0,0.0)` : 현재 fract로 구한 점의 모서리에서 점까지의 방향 벡터를 구한다.
        - 다른 모서리(`vec2(1.0,0.0)` )는 그 모서리에서 현재 점까지의 벡터를 구하려면 해당 모서리의 기준 벡터를 빼줘야 한다. 기존에 모서리를 구하기 위해 더해줬던 만큼.
    - 위의 두 벡터를 내적한다.
    - 두 벡터의 내적은 두 방향이 얼마나 같은 쪽을 향하는지 알려주는 것.
    이제 이 내적값이 현재 좌표가 각 모서리(무작위로 설정한 방향 벡터)와 얼마나 비슷한 방향을 가르키는지에 대한 값이 된다.
    - Gradient Noise는 모든 모서리 지점에서 값이 0이 되는데, Value Noise에서는 모서리에 랜덤한 값이 들어가서 이 값이 만약 극댓값/극솟값이 되면 타일링된게 티가 났었다. 하지만 Gradient Noise에서는 칸 내부에서만 극댓값/극솟값이 생기기때문에 타일이 덜 티나게 된다.
    
    ```glsl
    // Gradient Noise by Inigo Quilez - iq/2013
    // https://www.shadertoy.com/view/XdXGW8
    float noise(vec2 st) {
        vec2 i = floor(st);
        vec2 f = fract(st);
    
        vec2 u = f*f*(3.0-2.0*f);
    
        return mix( mix( dot( random2(i + vec2(0.0,0.0) ), f - vec2(0.0,0.0) ),
                            dot( random2(i + vec2(1.0,0.0) ), f - vec2(1.0,0.0) ), u.x),
                    mix( dot( random2(i + vec2(0.0,1.0) ), f - vec2(0.0,1.0) ),
                            dot( random2(i + vec2(1.0,1.0) ), f - vec2(1.0,1.0) ), u.x), u.y);
    }
    ```
    
### Rotate Noise
- 노이즈를 회전해서 얼룩말같은 줄무늬를 만드는 예제.
- 줄무늬함수
    - 좌표를 확대해서 sin의 반복주기로 줄무늬를 그린다.
    - 이 함수는 sin의 반복성을 통해서 줄무늬를 그린다. `sin(pos.x * 3.1415)` 를 통해서 원래는 넓은 간격을 압축하여 반복되게 한다.
    - fract타일링은 딱딱 끊어지게 그려졌지만 sin을 통해서 매끄럽게 이어지는 줄무늬를 얻을 수 있다.
    
    ```glsl
    float lines(in vec2 pos, float b){
        float scale = 10.0;
        pos *= scale;
        return smoothstep(0.0, 0.5+b*0.5, abs((sin(pos.x*3.1415)+b*2.0))*.5);
    }
    ```
    
- 메인함수
    - `st.yx` 는 x와 y를 뒤바꿔서 가로방향으로 그려지게끔 자리를 바꾼 것
    - 노이즈의 값을 구해서 이 노이즈 값을 회전값으로 사용하고 있다. 그리고 거기에 원래 좌표를 곱해주는데, 이러면 원래 좌표를 그 위치의 노이즈 값만큼 회전이 된다.
    - 회전 각도가 부드럽게 변하는 트릭이다. `line()` 함수는 원래 가로로 일직선인 무늬였는데 노이즈에 따라서 위치를 회전하여 휘어지는 무늬가 만들어진다.
    
    ```glsl
    vec2 pos = st.yx*vec2(10.,3.);
    pos = rotate2d( noise(pos) ) * pos; // rotate the space
    pattern = lines(pos,.5); // draw lines
    ```
        

## Simplex Noise

- Simplex Noise에 대해서
    - Simplex Noise는 사각형 대신 삼각형을 사용하여 더 효율적이고 자연스러운 노이즈를 만드는 것이다.
        
        ![Simplex Noise Triangle](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/SimplexNoise Triangle.png)
        
    - 기존의 노이즈들은 사각형의 꼭짓점을 모두 계산하고 보간하여 노이즈를 만들었었다. 하지만 삼각형을 사용하면 2D에서는 4개에서 3개, 3D에서는 8개에서 4개, 4D에서는 16개에서 4개로 줄어들게 되는거다.
    - 기존의 사각형 노이즈는 방향에 따라서 결이 보일 때가 있었지만 삼각형으로 사용하면 특정 방향으로 왜곡된 부분이 적어져서 어느 방향으로 봐도 더 고르게 보이는 노이즈를 만들 수 있다.
    - 이렇게 더 계산량적고 자연스러운 노이즈를 얻을 수 있다.
- 예시의 코드를 보자. 이 코드는 삼각형 격자를 만들어보는 코드다.
    - `skew()` : 이 함수는 사각형 격자를 기울여서 평행사변형으로 만드는 코드다.
        - `1.5747` 은 `2/√3` 인 숫자인데, 대각선으로 잘랐을 때 정삼각형이 나오도록 하는 배율이다. x축으로 공간을 1.1547만큼 더 늘리는 것.
        - 그리고 늘어난 x축 좌표의 절반만큼 y좌표에 더해서 줄마다 밀리게 만든다. (r.x가 커질 수록 기울어짐)
    - `simplexGrid()` : 위 함수로 만든 평행사면형을 삼각형으로 나누는 함수.
        - `if (p.x > p.y) {…` : 이 점이 평행사변형 안에서 위쪽 삼각형`(y>x)`에 있는지, 아래쪽 삼각형`(x>y)`에 있는지 판별한다.
        - 여기서 `xyz` 변수를 통해서 이 점이 삼각형 각 꼭짓점에 얼마나 가까운지를 나타낸다. 이를 무게중심좌표라고 부른다. 이 위치정보를 통해서 각 꼭짓점의 gradient 벡터와 내적해서 노이즈 값을 계산하는 것이다(이 예제에서는 노이즈 계산까지 하지 않는다.).
    
    ```glsl
    // Author @patriciogv - 2015 - patriciogonzalezvivo.com
    
    #ifdef GL_ES
    precision mediump float;
    #endif
    
    uniform vec2 u_resolution;
    uniform vec2 u_mouse;
    uniform float u_time;
    
    vec2 skew (vec2 st) {
        vec2 r = vec2(0.0);
        r.x = 1.1547*st.x;
        r.y = st.y+0.5*r.x;
        return r;
    }
    
    vec3 simplexGrid (vec2 st) {
        vec3 xyz = vec3(0.0);
    
        vec2 p = fract(skew(st));
        if (p.x > p.y) {
            xyz.xy = 1.0-vec2(p.x,p.y-p.x);
            xyz.z = p.y;
        } else {
            xyz.yz = 1.0-vec2(p.x-p.y,p.y);
            xyz.x = p.x;
        }
    
        return fract(xyz);
    }
    
    void main() {
        vec2 st = gl_FragCoord.xy/u_resolution.xy;
        vec3 color = vec3(0.0);
    
        // Scale the space to see the grid
        st *= 10.;
    
        // Show the 2D grid
        color.rg = fract(st);
    
        // Skew the 2D grid
        // color.rg = fract(skew(st));
    
        // Subdivide the grid into to equilateral triangles
        // color = simplexGrid(st);
    
        gl_FragColor = vec4(color,1.0);
    }
    
    ```