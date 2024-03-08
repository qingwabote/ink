# 反射方程
$$
L_r(p, \omega_r) = \int_{\Omega^+} f_r(p, \omega_r, \omega_i) L_i(p, \omega_i)(n·\omega_i)d\omega_i
\hspace{100cm}
$$
*$f_r$: BRDF*  
*$L$: 辐射率 (Radiance)。反射方程是个递归方程，最终会递归到计算光源的 $L$，我猜游戏引擎中用光源的 RGB $LitColor$ 表示它的辐照度 (Irradiance)，那么光源在点 $p$ 的 $L_o = LitColor (n·\omega)$。可以看出如果光源垂直照射表面时辐射率最大，$n·\omega = 1$。*

[由于渲染方程和反射率方程都没有解析解，我们将会用离散的方法来求得这个积分的数值解](https://learnopengl-cn.github.io/07%20PBR/01%20Theory/)：
```c
int steps = 100;
float sum = 0.0f;
vec3 P    = ...;
vec3 Wo   = ...;
vec3 N    = ...;
float dW  = 1.0f / steps;
for(int i = 0; i < steps; ++i) 
{
    vec3 Wi = getNextIncomingLightDir(i);
    sum += Fr(p, Wi, Wo) * L(p, Wi) * dot(N, Wi) * dW;
}
```

# Cook-Torrance BRDF
Cook-Torrance BRDF 按比例拆成了漫反射和镜面反射两部分 $k_d + k_s = 1$：
$$
f_r = k_df_{d} + k_sf_{s}
\hspace{100cm}
$$
Cook-Torrance 镜面反射方程：
$$
f_{cook-torrance} = \frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}
\hspace{100cm}
$$
漫反射部分仍然使用 lambert：
$$
f_{lambert} = \frac{c}{\pi}
\hspace{100cm}
$$
*Where $c_{diff}$ is the diffuse albedo of the material. [Real Shading in Unreal Engine 4](https://blog.selfshadow.com/publications/s2013-shading-course/#course_content)* *[译文](https://zhuanlan.zhihu.com/p/121719442)*

[Fresnel 方程返回的是一个物体表面光线被反射的百分比，也就是我们反射方程中的参数 $k_s$](https://learnopengl-cn.github.io/07%20PBR/02%20Lighting/)，合并之：
$$
f_r = k_d\frac{c}{\pi} + \frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}
\hspace{100cm}
$$
那么反射方程被分割成了漫反射和镜面反射两部分
$$
\int_{\Omega} (k_d\frac{c}{\pi}) L_i(p,\omega_i) n \cdot \omega_i  d\omega_i + 
\int_{\Omega} (\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)})
L_i(p,\omega_i) n \cdot \omega_i  d\omega_i
\hspace{100cm}
$$
漫反射积分通过**漫反射贴图**来解，下面讨论如何解镜面反射积分

# Split Integral Approximation
<!-- $$
\int Env(l)BRDF_{env}(l,v,h)cos(\omega)d\omega \hspace{100cm}\\
\approx \left( 4\int Env(l)D_{env}(h)cos(\omega) d\omega\right)\left(\int BRDF_{env}(l,v,h)cos(\omega) d\omega\right)
\hspace{100cm}
$$
The first part represents **environment map filtering** (blurring), and the second part is the 
**Environment BRDF** *[Getting More Physical in Call of Duty: Black Ops II](https://blog.selfshadow.com/publications/s2013-shading-course/#course_content)*   -->
*[漫谈 split sum approximate](https://zhuanlan.zhihu.com/p/56063836)*

"Approximation in RTR (GAMES 202 P3 P5)"提到套用这样一个约等式：
$$
\int_{\Omega} f(x)g(x)dx \approx \frac{\int_{\Omega} f(x)dx}{\int_{\Omega} dx} \cdot \int_{\Omega} g(x)dx
\hspace{100cm}
$$
得到
$$
 \int_{\Omega} (\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)})
L_i(p,\omega_i) n \cdot \omega_i  d\omega_i \approx \frac{\int_{\Omega} L_i(p,\omega_i) d\omega_i}{\int_{\Omega} d\omega_i} \cdot \int_{\Omega} (\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}) n \cdot \omega_i  d\omega_i
\hspace{100cm}
$$

# Environment Specular BRDF
*For the diffuse response we used the classic Lambertian BRDF. This talk will be all about 
the **specular** response. [Getting More Physical in Call of Duty: Black Ops II](https://blog.selfshadow.com/publications/s2013-shading-course/#course_content)*

Black Ops 对 Cook-Torrance BRDF 进行了重构
$$
 \frac{DFG}{4(n \cdot l)(n \cdot v)} = \frac{\pi D}{4} \frac{F}{1} \frac{G}{(n \cdot l)(n \cdot v)} = D_{pl}FV
\hspace{100cm}
$$
*V: visibility function*  
然后展开$BRDF_{env}$
$$
\int D_{pl}FVcos(\omega) d\omega
\hspace{100cm}
$$
*pl: point light*

带入 Schlick-Fresnel
$$
F(l,h) = rf_0 + (1− rf_0 )(1− h ⋅l)^5 
\hspace{100cm}
$$
最终得到
$$
rf_0\int D_{env}V\cos(\omega)d\omega + (1 - rf_0)\int D_{env}V(1 - h⋅l)^5\cos(\omega)d\omega
\hspace{100cm}
$$
即
$$
rf_0 * a_1 + (1 - rf_0) * a_0
\hspace{100cm}
$$
使用魔法将 $a_0$, $a_1$ 拟合出来
```glsl
float3 EnvironmentBRDF( float g, float NoV, float3 rf0 ) {
    float4 t = float4( 1/0.96, 0.475, (0.0275 - 0.25 * 0.04)/0.96, 0.25 );
    t *= float4( g, g, g, g );
    t += float4( 0, 0, (0.015 - 0.75 * 0.04)/0.96, 0.75 );
    float a0 = t.x * min( t.y, exp2( -9.28 * NoV ) ) + t.z;
    float a1 = t.w;
    return saturate( a0 + rf0 * ( a1 - a0 ) );
}
```

# UE
重构 Cook-Torrance Specular BRDF
$$
\frac{DFG}{4(n \cdot l)(n \cdot v)} \approx D \frac{1}{(v \cdot n)(v \cdot l)}  FG
\hspace{100cm}\\~\\
D = GGX\_Mobile
\hspace{100cm}\\~\\
\frac{1}{(v \cdot n)(v \cdot l)} \approx 0.25 * roughness + 0.25
\hspace{100cm}\\~\\
FG \approx EnvBRDFApprox
\hspace{100cm}
$$

```glsl
half GGX_Mobile(half Roughness, float NoH)
{
    // Walter et al. 2007, "Microfacet Models for Refraction through Rough Surfaces"
	float OneMinusNoHSqr = 1.0 - NoH * NoH; 
	half a = Roughness * Roughness;
	half n = NoH * a;
	half p = a / (OneMinusNoHSqr + n * n);
	half d = p * p;
	// clamp to avoid overlfow in a bright env
	return min(d, 2048.0);
}
```
*<https://github.com/EpicGames/UnrealEngine/blob/c3caf7b6bf12ae4c8e09b606f10a09776b4d1f38/Engine/Shaders/Private/MobileGGX.ush>*

```glsl
half3 MobileSpecularGGXInner(half D, half3 EnvBrdf, half3 SpecularColor, half Roughness, half NoV, half NoL, half VoH)
{
#if MOBILE_HIGH_QUALITY_BRDF
	half Vis = Vis_SmithJointApprox_Mobile(Roughness * Roughness, NoV, NoL);
	half3 F = F_Schlick_Mobile(SpecularColor, VoH);
#else
	half Vis = (Roughness * 0.25 + 0.25);
	half3 F = EnvBrdf;
#endif

	return (D * Vis) * F;
}
```
*<https://github.com/EpicGames/UnrealEngine/blob/46544fa5e0aa9e6740c19b44b0628b72e7bbd5ce/Engine/Shaders/Private/MobileShadingModels.ush>*

```glsl
half3 EnvBRDFApprox( half3 SpecularColor, half Roughness, half NoV )
{
	// [ Lazarov 2013, "Getting More Physical in Call of Duty: Black Ops II" ]
	// Adaptation to fit our G term.
	const half4 c0 = { -1, -0.0275, -0.572, 0.022 };
	const half4 c1 = { 1, 0.0425, 1.04, -0.04 };
	half4 r = Roughness * c0 + c1;
	half a004 = min( r.x * r.x, exp2( -9.28 * NoV ) ) * r.x + r.y;
	half2 AB = half2( -1.04, 1.04 ) * a004 + r.zw;

	// Anything less than 2% is physically impossible and is instead considered to be shadowing
	// Note: this is needed for the 'specular' show flag to work, since it uses a SpecularColor of 0
	AB.y *= saturate( 50.0 * SpecularColor.g );

	return SpecularColor * AB.x + AB.y;
}
``` 
*<https://github.com/EpicGames/UnrealEngine/blob/ddf25c16cf3311401a7695d24b7682c653f9117d/Engine/Shaders/Private/BRDF.ush>*