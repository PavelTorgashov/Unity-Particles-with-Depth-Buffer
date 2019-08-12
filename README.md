# Unity-Particles with Depth Buffer
**Shader for particles with writing to z-buffer.**

The standard particle shader in Fade or Transparent mode does not set the depth buffer.
If this does not suit you, and you need depth for semitransparent particles, then you can go the following way.

1) From the set of standard shaders we take the particle shader we need. I took Particle Add as an example.

2) In this shader we make the second pass. This passage is tagged with "LightMode" = "ShadowCaster" and will be used when building the depth map:

```
                Pass
                {
                    Tags { "LightMode" = "ShadowCaster" "RenderType" = "Opaque" }
                    ColorMask 0
                    AlphaToMask On
 
                    CGPROGRAM
                    #pragma vertex vert
                    #pragma fragment frag
 
                    #include "UnityCG.cginc"
 
                    sampler2D _MainTex;
                    fixed4 _TintColor;
                    float4 _MainTex_ST;
 
                    struct appdata_t {
                        float4 vertex : POSITION;
                        fixed4 color : COLOR;
                        float2 texcoord : TEXCOORD0;
                    };
 
                    struct v2f {
                        float4 vertex : SV_POSITION;
                        fixed4 color : COLOR;
                        float2 texcoord : TEXCOORD0;
                    };
 
                    v2f vert(appdata_t v)
                    {
                        v2f o;
                        o.vertex = UnityObjectToClipPos(v.vertex);
                        o.color = v.color;
                        o.texcoord = TRANSFORM_TEX(v.texcoord, _MainTex);
                        return o;
                    }
 
                    fixed4 frag(v2f i) : SV_Target
                    {
                        fixed4 col = 2.0f * i.color * _TintColor * tex2D(_MainTex, i.texcoord);
                        return col;
                    }
                    ENDCG
                }
```

This passage does nothing special, it only brings the object into the depth map. In this case, cutting by alpha is used. That is, in the depth map, the particle will be rendered in AlphaCut mode.

3) Create a new material, assign a new shader to it, assign a material for particles.

As result, normal camera view:
![alt text](https://github.com/PavelTorgashov/Unity-Particles-with-Depth-Buffer/blob/master/Env/%D0%A1%D0%BA%D1%80%D0%B8%D0%BD%D1%88%D0%BE%D1%82%202019-08-12%2017.05.56.png) 

Depth map camera view:
![alt text](https://github.com/PavelTorgashov/Unity-Particles-with-Depth-Buffer/blob/master/Env/%D0%A1%D0%BA%D1%80%D0%B8%D0%BD%D1%88%D0%BE%D1%82%202019-08-12%2017.35.11.png)

