---
title: "Color"
date: 2026-09-03 00:00:00 +0900
categories: [Shader, The book of shaders]
tags: [shader, study]     # TAG names should always be lowercase
---

## Color

- vec타입에 대해서 설명하고 있다.
- vec 내부데이터는 C언어의 struct처럼 접근하는 것과 비슷하다.
- vec타입의 값에 접근하는 방식은 배열처럼 인덱스로 접근하기, xyzw, rgba, stpq(텍스쳐 공간 좌표에 사용)로 접근할 수 있다.
- GLSL의 벡터는 속성을 원하는 순서대로 조합하여 접근할 수 있다. 이러한 기술을 swizzle이라고 부른다.

```glsl
vec3 yellow, magenta, green;

// Making Yellow
yellow.rg = vec2(1.0);  // Assigning 1. to red and green channels
yellow[2] = 0.0;        // Assigning 0. to blue channel

// Making Magenta
magenta = yellow.rbg;   // Assign the channels with green and blue swapped

// Making Green
green.rgb = yellow.bgb; // Assign the blue channel of Yellow (0) to red and blue channels
```

### mix()
- GLSL에는 컬러를 섞는 함수 `mix()` 가 있는데 이건 두 값을 백분율에 따라 혼합해 주는 함수다.
- 아래 코드는 시간에 따라 sin을 통해 컬러가 변하는 코드다.

```glsl
#ifdef GL_ES
precision mediump float;
#endif

uniform vec2 u_resolution;
uniform float u_time;

vec3 colorA = vec3(0.149,0.141,0.912);
vec3 colorB = vec3(1.000,0.833,0.224);

void main() {
    vec3 color = vec3(0.0);

    float pct = abs(sin(u_time));

    // Mix uses pct (a value from 0-1) to
    // mix the two colors
    color = mix(colorA, colorB, pct);

    gl_FragColor = vec4(color,1.0);
}
```

#### Playing with gradients
- 다음 예제 코드는 저번에 했던 선그리기 예제에 mix를 통해서 배경색을 섞는 것. vec3를 통해서 각 속성별로 선을 정하고 채널마다 다른 방식으로 색을 섞는 코드다.
- 주석으로 비활성화 되어있는 `prt.r,prt.g,prt.b` 를 풀면 선이 추가되고 배경색이 변화한다.
    - `pct.r = smoothstep(0.0,1.0, st.x);` : 빨간색 S자 곡선이 추가됨. 배경색: 앞쪽이 좀 더 파란색 원색에 가까워지고, 뒤로 갈수록 빨간색이 섞여 주황색이 됨.
    - `pct.g = sin(st.x*PI);` : 초록색 봉우리선이 추가됨. 배경색이 파랑→초록→빨강이됨
    - `pct.b = pow(st.x,0.5);` : 파란색선이 초반에 급상승 되게 휘어짐. 배경은 파랑→노랑.
- 여기서  `color = mix(colorA, colorB, pct);` 채널(rgb)별로 다르게 색을 섞는다.
    - 참고로 pct는 vec3인데 이러면 각 채널에 대해서 각 비율로 섞인다. color.r = mix(colorA.r, colorB.r, pct.r) 이렇게 섞이는 것.
- 그리고 다음 코드 3줄은 저번 챕터에 사용했던 plot()함수로 선을 그리는 것
    - 저번 챕터의 예저에서 봤던 `color=(1.0-pct)**color+pct**vec3(0.0,1.0,0.0);` 이 코드가 `mix()`내부코드와 같은 공식이다.
- 이렇게 색을 섞는 비율을 선으로 시각화하고 배경은 색이 섞이는 것을 볼 수 있는 코드다.
- ColorA는 보라색이 섞인 파란색이다.

```glsl
#ifdef GL_ES
precision mediump float;
#endif

#define PI 3.14159265359

uniform vec2 u_resolution;
uniform vec2 u_mouse;
uniform float u_time;

vec3 colorA = vec3(0.149,0.141,0.912);
vec3 colorB = vec3(1.000,0.833,0.224);

float plot (vec2 st, float pct){
    return  smoothstep( pct-0.01, pct, st.y) -
            smoothstep( pct, pct+0.01, st.y);
}

void main() {
    vec2 st = gl_FragCoord.xy/u_resolution.xy;
    vec3 color = vec3(0.0);

    vec3 pct = vec3(st.x);

        //pct.r = smoothstep(0.0,1.0, st.x);
        //pct.g = sin(st.x*PI);
        //pct.b = pow(st.x,0.5);

    color = mix(colorA, colorB, pct);

    // Plot transition lines for each channel
    color = mix(color,vec3(1.0,0.0,0.0),plot(st,pct.r));
    color = mix(color,vec3(0.0,1.0,0.0),plot(st,pct.g));
    color = mix(color,vec3(0.0,0.0,1.0),plot(st,pct.b));

    gl_FragColor = vec4(color,1.0);
}

```

- 예제 코드를 보고 하늘의 색처럼 변하는(밤→일출→낮→일몰) 코드를 만들어봄
    - 처음에 시간계산을 sin으로 사용하다가 실행해보니까 정→역순으로 재생되길래, fract로 바꾸었다.

```glsl
#define PI 3.14159265359

void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    // 일출: 밤 > 노랑 > 파랑
    // 일몰: 파랑 > 주황 > 밤
    vec3 sunColor = vec3(0.000,0.755,0.912);   // 낮
    vec3 nightColor = vec3(0.0); // 밤
    vec3 sunriseColor = vec3(1.000,0.955,0.060); // 일출
    vec3 sunsetColor = vec3(1.000,0.344,0.277);  // 일몰

    vec2 st = fragCoord.xy/iResolution.xy;
    vec3 color = vec3(0.0);

    vec3 pct = vec3(st.y);
    pct *= smoothstep(0.0,1.0, st.y);

    // 4구간
    float line = 1.0 / 4.0;
    float line2 = line * 2.0;
    float line3 = line * 3.0;
    float t = fract(iTime * 0.1); // abs(sin(iTime * 0.25));
    if (t < line)
    {
        float progress = (t) / line;
        color = mix(nightColor, sunriseColor, progress);
    }
    else if (t < line2)
    {
        float progress = (t - line) / line;
        color = mix(sunriseColor, sunColor, progress);
    }
    else if (t < line3)
    {
        float progress = (t - line2) / line;
        color = mix(sunColor, sunsetColor, progress);
    }
    else
    {
        float progress = (t - line3) / line;
        color = mix(sunsetColor, nightColor, progress);
    }
    
    color = mix(color, color * 0.5,pct);
    
    fragColor = vec4(color,1.0);
}
```

### HSB
- HSB: 색조(Hue), 채도(Saturation), 밝기(Brightness)의 약자로 색상을 표현하는데 흔히 사용되는 방식.
    - 이런식으로 사용하면 빨간색을 유지하면서 밝기만 낮추고 싶으면 B값만 낮추면 되서 직관적으로 사용할 수 있다.
    - H: 색상환 각도, 빨노초파보 0~1로 표현
    - B: [검정] 0~1 [원래색]
    - S: 낮으면 흐릿한 회색빛, 높으면 원색
    
    ![Color Wheel](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/Color Wheel.png)
    
- `rgb2hsb, hsb2rgb` 함수로 RGB와 HSB를 오갈 수 있다.
    - 내부 코드는 색상환의 각도를 구하는 벡터연산이다.
- `hsb2rgb` 함수에 대해서 알아보자
    - 첫번째 인자: Hue. 0~1로 갈수록 빨노초파보빨 순으로 바뀐다.
    - 두번째 인자: Saturation. 줄이면 색상이 뿌옇게 된다.
    - 세번째 인자: Britghtness: 밝기 검은색(0)에서 원래색(1)이 된다.

```glsl
#ifdef GL_ES
precision mediump float;
#endif

uniform vec2 u_resolution;
uniform float u_time;

vec3 rgb2hsb( in vec3 c ){
    vec4 K = vec4(0.0, -1.0 / 3.0, 2.0 / 3.0, -1.0);
    vec4 p = mix(vec4(c.bg, K.wz),
                    vec4(c.gb, K.xy),
                    step(c.b, c.g));
    vec4 q = mix(vec4(p.xyw, c.r),
                    vec4(c.r, p.yzx),
                    step(p.x, c.r));
    float d = q.x - min(q.w, q.y);
    float e = 1.0e-10;
    return vec3(abs(q.z + (q.w - q.y) / (6.0 * d + e)),
                d / (q.x + e),
                q.x);
}

//  Function from Iñigo Quiles
//  https://www.shadertoy.com/view/MsS3Wc
vec3 hsb2rgb( in vec3 c ){
    vec3 rgb = clamp(abs(mod(c.x*6.0+vec3(0.0,4.0,2.0),
                                6.0)-3.0)-1.0,
                        0.0,
                        1.0 );
    rgb = rgb*rgb*(3.0-2.0*rgb);
    return c.z * mix(vec3(1.0), rgb, c.y);
}

void main(){
    vec2 st = gl_FragCoord.xy/u_resolution;
    vec3 color = vec3(0.0);

    // We map x (0.0 - 1.0) to the hue (0.0 - 1.0)
    // And the y (0.0 - 1.0) to the brightness
    color = hsb2rgb(vec3(st.x,1.0,st.y));

    gl_FragColor = vec4(color,1.0);
}
```

#### HSB in polar coordinates
- 기존에 있던 RGB를 색상환으로 표시하는 예제를 RYB로 바꿔보았다.
    
    ```glsl
    #ifdef GL_ES
    precision mediump float;
    #endif
    
    #define TWO_PI 6.28318530718
    
    uniform vec2 u_resolution;
    uniform float u_time;
    
    //  Function from Iñigo Quiles
    //  https://www.shadertoy.com/view/MsS3Wc
    vec3 hsb2rgb( in vec3 c ){
        vec3 rgb = clamp(abs(mod(c.x*6.0+vec3(0.0,4.0,2.0),
                                    6.0)-3.0)-1.0,
                            0.0,
                            1.0 );
        rgb = rgb*rgb*(3.0-2.0*rgb);
        return c.z * mix( vec3(1.0), rgb, c.y);
    }
    
    float angle2RYB(float angle)
    {
        // 노란색이 초록색 자리로
        // 빨강 0
        // 주황 0.083 -> 0.167
        // 노랑 0.167 -> 0.333
        // 초록 0.333 -> 0.500
        // 파랑이랑 보라가 조금씩 밀림
        // 시안 0.500 -> 0.583
        // 파랑 0.667 -> 0.667
        // 보라 0.833 -> 0.833
        // 마젠타 0.833~0.917 0.917
        if (angle <= 0.0)
        {
            // 빨강
            return 0.0;
        }
        else if (angle <= 0.167)
        {
            float progress = angle / (0.167);
            // 주황
            //return 0.083;    
            return mix(0.0, 0.083, progress);
        }
        else if (angle <= 0.333)
        {
            float progress = (angle - 0.167) / (0.333 - 0.167);
            // 노랑
            return mix(0.083, 0.167, progress); 
        }
        else if (angle <= 0.500)
        {
            float progress = (angle - 0.333) / (0.500 - 0.333);
            // 초록
            //return 0.333;    
            return mix(0.167, 0.333, progress); 
        }
        else if (angle <= 0.583)
        {
            float progress = (angle - 0.500) / (0.583 - 0.500);
            // 시안
            //return 0.500;    
            return mix(0.333, 0.500, progress); 
        }
        else if (angle <= 0.667)
        {
            float progress = (angle - 0.583) / (0.667 - 0.583);
            // 파랑
            //return 0.667;    
            return mix(0.500, 0.667, progress); 
        }
        else if (angle <= 0.833)
        {
            float progress = (angle - 0.667) / (0.833 - 0.667);
            // 보라
            //return 0.759;    
            return mix(0.667, 0.833, progress); 
        }
        else if (angle <= 0.917)
        { 
            float progress = (angle - 0.833) / (0.917 - 0.833);
            // 마젠타
            //return 0.833;    
            return mix(0.833, 0.917, progress); 
        }
        else
        {
            // 다시 빨강
            float progress = (angle - 0.917) / (1.0 - 0.917); 
            return mix(0.917, 1.0, progress); 
        }
            
        
        return 0.0;
    }
    
    void main(){
        vec2 st = gl_FragCoord.xy/u_resolution;
        vec3 color = vec3(0.0);
    
        // Use polar coordinates instead of cartesian
        vec2 toCenter = vec2(0.5)-st;
        float angle = atan(toCenter.y,toCenter.x);
        float radius = length(toCenter)*2.0;
    
        float newHue = angle2RYB(angle/TWO_PI + 0.5);
        // Map the angle (-PI to PI) to the Hue (from 0 to 1)
        // and the Saturation to the radius
        float inRad = step(1.0, radius);
        color = mix(hsb2rgb(vec3(newHue,radius,1.0)),vec3(1.0,1.0,1.0), inRad);
    
        gl_FragColor = vec4(color,1.0);
    }
    
    ```
    
    - 기존 결과
        
        ![Color HSB Exam Origin](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/Color HSB Exam Origin.png)
        
    - 결과는 이렇게 표시된다. 예시의 사진에서는 파란색이 중앙아래에 있었는데 찾아보니까 파란색은 240도에 보통 표시한다고해서 240도에 뒀다.
        
        ![Color HSB Exam Result](https://github.com/Eunseo-1110/jfourteen_ImageStorage/raw/main/ImageStorage/Color HSB Exam Result.png)
        
    - 시안이랑 마젠타도 넣었는데 굳이 필요 하진 않은 것 같다.
    - 그리고 매핑하는 부분을 step함수써서 하도록 바꿔보았다. 분기문은 gpu의 병렬 처리 효율이 떨어져서 성능상 step을 사용하는게 좋다.
        
        ```glsl
        float angleNormalize(float angle, float low, float max)
        {
            return (angle - low) / (max - low);
        }
        
        float angle2RYB(float angle)
        {
            // 노란색이 초록색 자리로
            // 빨강 0
            // 주황 0.083 -> 0.167
            // 노랑 0.167 -> 0.333
            // 초록 0.333 -> 0.500
            // 파랑이랑 보라가 조금씩 밀림
            // 시안 0.500 -> 0.583
            // 파랑 0.667 -> 0.667
            // 보라 0.833 -> 0.833
            // 마젠타 0.833~0.917 0.917
            
            float result = 0.0;
            // 주황
            result += (step(0.0, angle) - step(0.167, angle)) * mix(0.0, 0.083, angleNormalize(angle, 0.0, 0.167));
            // 노랑
            result += (step(0.167, angle) - step(0.333, angle)) * mix(0.083, 0.167, angleNormalize(angle, 0.167, 0.333));
            // 초록
            result += (step(0.333, angle) - step(0.500, angle)) * mix(0.167, 0.333, angleNormalize(angle, 0.333, 0.500));
            // 시안
            result += (step(0.500, angle) - step(0.583, angle)) * mix(0.333, 0.500, angleNormalize(angle, 0.500, 0.583));
            // 파랑
            result += (step(0.583, angle) - step(0.667, angle)) * mix(0.500, 0.667, angleNormalize(angle, 0.583, 0.667));
            // 보라
            result += (step(0.667, angle) - step(0.833, angle)) * mix(0.667, 0.833, angleNormalize(angle, 0.667, 0.833));
            // 마젠타
            result += (step(0.833, angle) - step(0.917, angle)) * mix(0.833, 0.917, angleNormalize(angle, 0.833, 0.917));
            // 빨강
            result += (step(0.917, angle) - step(1.0, angle)) * mix(0.917, 1.0, angleNormalize(angle, 0.917, 1.0));
            return result;
        }
        ```

---
- 색 관련으로 들어가니까 어렵다..