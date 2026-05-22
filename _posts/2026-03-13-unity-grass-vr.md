---
layout: post
title: "Grass in Unity VR - A painful experience"
subtitle: "Or how to decrease your FPS by 90% with terribly looking grass."
date: 2026-03-13 10:00:00 +0200
tags: [gamedev, optimization, vr, unity, coding]
excerpt: "When games become just tools for data acquisition, just as in my case, you can't really focus on becoming a better game developer..."
---

## How did we get here?

When games become just tools for data acquisition, just as in my case, you can't really focus on becoming a better game developer or that's how I thought a few months ago, when my first VR game played at astonishing 9 FPS on an actual VR headset. The moment could've been easily avoided if my team received the headset for testing before rolling out the game, but this was just another sorry reason for delivering totally unstable piece of s~~hit~~oftware. The aforementioned prototype was built in Godot and I really loved the way it looked back then, but the performance was far too bad and after testing it, we've come to some conclusions:

- The grass in its current form impacts the performance too much, even after really aggressive distance culling
- We have no idea how to do it differently while preserving the looks
- VR is far more demanding in terms of optimization and besides needing to render two separate, high resolution frames for both of the eyes, it should achieve performance of around 72-90 FPS
- Godot is not quite production ready for 3D games and presents a drop below 60 fps at around 6 times fewer objects visible than Unity

The last of these points got solved accidentally - suddenly I was required to get to know Unity for work and this way I've managed to kill two birds with one stone. Before moving onto the problems, I'll show you how the grass prototype in Godot did look like:

![Early test of the grass in Godot prototype of the game](/assets/images/godot_grass_early.jpg)

### Some time later

Let's do a time skip and move a month further, to the moment when I've already been quite fluent in Unity and decided to go back to the orphaned project (to be totally honest, my friends that supervise the project asked me to get going, because they wanted to use the software to conduct research on the patients). The first Unity version didn't contain grass (as I was too scared I'll take hours to code it and will have to, once again, get rid of it) and looked far worse than the Godot prototype, but it had far better performance, even with some overhead for additional graphical fireworks. We've skipped quite a lot of time in this time skip and my very-well-organized self managed to find time to once again play The Legend of Zelda: Breath of the Wild while finding no time to focus on my research. I really loved the grass and even felt sad that I couldn't implement such grass in my game. And that's when it hit me - Nintendo Switch hardware is quite shitty when compared to the PCs! That's how I fell into a rabbit hole and spent far too much time researching optimization techniques used in that game.

## Exploring the rabbit hole

The first, most shallow exploration led me to watching [a great tutorial by Daniel Ilett](https://www.youtube.com/watch?v=MeyW_aYE82s) but after a short research, the geometry shaders approach presented in the video turned out to be too computationally expensive.

Diving deeper, watching the dev streams from various developers and a lot of tutorials ([this one stands out the most](https://www.youtube.com/watch?v=Y0Ko0kvwfgA)) made me realize that the grass implementation from Godot was one of the worst possible ideas. What were the issues?

- each of the blades consisted of 32 triangles, which after multiplying by 100 000 blades rendered at one time accumulated to 3 200 000 triangles solely from grass,
- the whole idea of rendering each blade as a separate object.

This way, each of the blades was being separately sent from CPU to GPU, which resulted in 100x too many draw calls for proper performance. What ideas did I get from those streams? First of all, I got to know a technique called GPU Instancing, which allows to instance similar objects at once using GPU and avoid sending them one by one from CPU. This way, the GPU receives a whole batch of models at once and is responsible for proper instancing of game objects. The other optimization was to make blade consist of smaller amount of tris and apply some computationally cheaper perturbations to make it look good. I think it's worth mentioning that I've went full Dunning-Kruger on this one and with insufficient knowledge, I fully dove into the GPU instancing. I might seem a little harsh on myself, but after implementing the solution, I came across Unity Docs (which I should've searched **before** doing anything) and discovered that GPU instancing is not the only way of achieving what I wanted to do, especially for Universal Render Pipeline, which I'm using. The whole idea wasn't bad, but it turns out that Unity has an easier way of handling this - GPU Resident Drawer which underneath handles the GPU instancing. The same doc page referenced me to use Scriptable Render Pipeline batcher, but I didn't really have time to experiment with this feature yet.

Despite discovering that the instancing process could be streamlined, I committed to improving existing logic and developing a simple shader for the grass. I needed to resort to some assistance from AI models, because I wanted to avoid painting new mask texture and use existing grass texture (used alongside dirt texture) as a marker of the area that should be populated by the grass. After some attempts of translating the Godot shader to the Unity system and improving the shader logic, I ended up with something like this:

```hlsl
Shader "Custom/StylizedGrass"
{
    Properties
    {
        _BaseColor ("Base", Color) = (0.15, 0.4, 0.1, 1)
        _TipColor ("Tip", Color) = (0.6, 0.9, 0.3, 1)
        _BaseColorAlt ("Base (Variation)", Color) = (0.18, 0.35, 0.08, 1)
        _TipColorAlt ("Tip (Variation)", Color) = (0.7, 0.85, 0.25, 1)
        _ColorVariation ("Tint Variation Strength", Range(0,1)) = 0.5
        _BrightnessVariation ("Brightness Variation", Range(0,0.5)) = 0.15
        _WindStrength ("Wind Strength", Float) = 0.15
        _WindSpeed ("Wind Speed", Float) = 1.5
        _CullDistance ("Cull Distance (m)", Float) = 20
        _FadeStart ("Fade Start (m)", Float) = 16
    }
    SubShader
    {
        Tags { "RenderType"="Opaque" "RenderPipeline"="UniversalPipeline" "Queue"="Geometry" }
        Cull Off

        Pass
        {
            Name "ForwardLit"
            Tags { "LightMode"="UniversalForward" }

            HLSLPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #pragma target 4.5
            #pragma multi_compile_fog

            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"

            StructuredBuffer<float4x4> _Matrices;

            CBUFFER_START(UnityPerMaterial)
                float4 _BaseColor;
                float4 _TipColor;
                float4 _BaseColorAlt;
                float4 _TipColorAlt;
                float _ColorVariation;
                float _BrightnessVariation;
                float _WindStrength;
                float _WindSpeed;
                float _CullDistance;
                float _FadeStart;
            CBUFFER_END

            struct Attributes
            {
                float4 positionOS : POSITION;
                float2 uv         : TEXCOORD0;
                UNITY_VERTEX_INPUT_INSTANCE_ID
            };

            struct Varyings
            {
                float4 positionHCS : SV_POSITION;
                float2 uv          : TEXCOORD0;
                float3 normalWS    : TEXCOORD1;
                float  fogCoord    : TEXCOORD2;
                float2 variation   : TEXCOORD3;
                UNITY_VERTEX_OUTPUT_STEREO
            };

            float Hash21(float2 p)
            {
                return frac(sin(dot(p, float2(12.9898, 78.233))) * 43758.5453);
            }

            Varyings vert(Attributes IN, uint instanceID : SV_InstanceID)
            {
                Varyings OUT;
                UNITY_SETUP_INSTANCE_ID(IN);
                UNITY_INITIALIZE_VERTEX_OUTPUT_STEREO(OUT);

                float4x4 mat = _Matrices[instanceID];

                float3 rootWS = float3(mat._m03, mat._m13, mat._m23);
                float dist = distance(_WorldSpaceCameraPos, rootWS);
                float fade = 1.0 - smoothstep(_FadeStart, _CullDistance, dist);
                float4 posOS = IN.positionOS;
                posOS.y *= fade;

                float3 worldPos = mul(mat, posOS).xyz;

                float2 rootXZ = float2(mat._m03, mat._m23);
                OUT.variation.x = Hash21(rootXZ);
                OUT.variation.y = Hash21(rootXZ + 17.31);

                float wind = sin(_Time.y * _WindSpeed + worldPos.x * 0.5 + worldPos.z * 0.3);
                worldPos.x += wind * _WindStrength * IN.uv.y;
                worldPos.z += wind * 0.5 * _WindStrength * IN.uv.y;

                OUT.positionHCS = TransformWorldToHClip(worldPos);
                OUT.uv = IN.uv;
                OUT.normalWS = float3(0, 1, 0);
                OUT.fogCoord = ComputeFogFactor(OUT.positionHCS.z);
                return OUT;
            }

            half4 frag(Varyings IN) : SV_Target
            {
                UNITY_SETUP_STEREO_EYE_INDEX_POST_VERTEX(IN);

                half tintHash = IN.variation.x * _ColorVariation;
                half3 baseCol = lerp(_BaseColor.rgb, _BaseColorAlt.rgb, tintHash);
                half3 tipCol  = lerp(_TipColor.rgb,  _TipColorAlt.rgb,  tintHash);
                half3 col = lerp(baseCol, tipCol, IN.uv.y);

                col *= 1.0 + (IN.variation.y * 2.0 - 1.0) * _BrightnessVariation;

                Light mainLight = GetMainLight();
                half ndotl = saturate(dot(IN.normalWS, mainLight.direction)) * 0.5 + 0.5;
                col *= mainLight.color * ndotl;
                col = MixFog(col, IN.fogCoord);
                return half4(col, 1);
            }
            ENDHLSL
        }
    }
    FallBack Off
}
```

The core logic is quite simple, but also uses some AI support for achieving the effect I want - I had no idea how to introduce random variation of the per-grass color and height and that's where the hashing idea came from. Even though I understand the code and idea, I wouldn't have thought of that myself and I don't know if that's the way you're supposed to do it. It works for me and will probably work for you, but if someone who's better at shading than me contacts me, I'll be happy to get to know if that's the right approach. To make the grass look better, similarly to the Godot prototype, I've introduced sin based wind and gradient based coloring.

The other thing I learned when trying to make this work was `#pragma multi_compile_fog` which allowed the fog to influence the grass - it's required due to rendering order and the grass in this approach was rendered after the fog was applied. When it comes to spawning and rendering the grass, the first important snippet is generating a mesh in the code:

```cs
static Mesh CreateGrassBlade(float width, float height, float bend)
    {
        float halfBase = width * 0.5f;
        float halfMid = width * 0.4f;
        float midY = height * 0.5f;
        float midZ = bend * 0.25f;
        float tipZ = bend;

        var mesh = new Mesh { name = "GrassBlade" };
        mesh.vertices = new[]
        {
            new Vector3(-halfBase, 0f,     0f),
            new Vector3( halfBase, 0f,     0f),
            new Vector3(-halfMid,  midY,   midZ),
            new Vector3( halfMid,  midY,   midZ),
            new Vector3( 0f,       height, tipZ),
        };
        mesh.triangles = new[] { 0, 2, 1, 1, 2, 3, 2, 4, 3 };
        mesh.uv = new[]
        {
            new Vector2(0f, 0f),
            new Vector2(1f, 0f),
            new Vector2(0f, 0.5f),
            new Vector2(1f, 0.5f),
            new Vector2(0.5f, 1f),
        };
        mesh.RecalculateNormals();
        mesh.RecalculateBounds();
        return mesh;
    }
```

The other important code segment is responsible for building the grass data for the shader and instancing mechanism. The `Build()` function samples base texture of the terrain and generates mask which determines which parts of the world should be populated with the grass. Besides that, it generates a buffer containing data on culling and height falloff on the edges of grassy areas. After generating such data, update calls:

```cs
Graphics.DrawMeshInstancedIndirect(activeMesh, 0, grassMaterial, bounds, argsBuffer);
```

Which results in such look:



## Are we done?

Not remotely, but grass is far from being an important part of this project, which is only a tool for medical research. **The main goal of implementing this grass system was achieved - the performance is VR friendly, however** I'm not entirely happy with the looks of the grass, as well as the whole project, and I can't really figure out what looks off and how to fix it. I'll keep digging because I'm using similar graphics style for another project which will be further developed. I'll try to keep you updated about the progress both in terms of graphics and the core functionality. Feel free to share your thoughts, improvement ideas or feedback overall. I'll also post the link to the grass generation repo when this iteration of the project ends.

**Disclaimer:** Please, don't treat this as a full tutorial - I intend for this post to be a means of experience exchange, place for you to learn from my mistakes on your game dev journey, but it's far from proper explanation of the used techniques, mostly because the whole solution is suited to my niche needs.
