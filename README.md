> [【从UnityURP开始探索游戏渲染】](https://github.com)**专栏-直达**

# **漫反射基本流程**

漫反射遵循兰伯特定律(Lambert's Cosine Law)，其核心流程如下：

* ‌**法线准备**‌：获取表面法线向量(通常来自顶点法线或法线贴图)
* ‌**光源方向计算**‌：确定光源到表面点的单位方向向量
* ‌**点积运算**‌：计算法线向量与光源方向的点积(N·L)
* ‌**能量约束**‌：使用saturate函数将结果限制在[0,1]范围
* ‌**颜色混合**‌：将结果与光源颜色和表面反照率(albedo)相乘

## 数学表达式：

$漫反射 = 光源颜色 × 表面反照率 × max(0, N·L)$

# **漫反射基本原理**

漫反射遵循兰伯特定律(Lambert's Law)，描述光线在粗糙表面均匀散射的现象。其核心特点是：

* 光线入射角度影响反射强度
* 表面法线方向决定反射分布
* 与观察角度无关的各向同性反射

## 兰伯特定律 Lambert’s law

$Cdiffuse=(Clight\*Mdiffuse)max(0,n·I）$

```
|  |  |
| --- | --- |
|  | fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * saturate(dot(worldNormal, worldLight)); |
```

## 半兰伯特

$Cdiffuse=(Clight\*Mdiffuse)(a(n·I)+b)$

绝大多数a和b都为0.5

$Cdiffuse=(Clight\*Mdiffuse)(0.5(n·I)+0.5)$

# **Unity URP中的实现细节**

## **核心实现位置**

URP中的漫反射计算主要分布在以下文件：

* `Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl`

## **实现原理**

* ‌**法线处理**‌：
  + 世界空间法线转换
  + 法线贴图支持
  + 双面渲染处理
* ‌**光源计算**‌：
  + 主光源方向计算
  + 附加光源循环处理
  + 点光源/聚光灯方向计算
* ‌**漫反射核心**‌：
  + 使用half精度优化
  + 能量守恒处理
  + 多光源叠加支持

## **关键代码实现**

* **UnityURP\_漫反射实现**

  ```
  |  |  |
  | --- | --- |
  |  | hlsl |
  |  | // 简化版Lambert漫反射实现 |
  |  | half3 DiffuseLighting(Light light, half3 normalWS, half3 albedo) |
  |  | { |
  |  | half NdotL = saturate(dot(normalize(normalWS), light.direction)); |
  |  | return light.color * albedo * NdotL; |
  |  | } |
  |  |  |
  |  | // 完整光照计算入口 |
  |  | half3 UniversalFragmentBlinnPhong(InputData inputData, SurfaceData surfaceData) |
  |  | { |
  |  | // 初始化光照结果 |
  |  | half3 color = surfaceData.emission; |
  |  |  |
  |  | // 主光源漫反射计算 |
  |  | Light mainLight = GetMainLight(); |
  |  | color += DiffuseLighting(mainLight, inputData.normalWS, surfaceData.albedo); |
  |  |  |
  |  | // 附加光源计算 |
  |  | uint pixelLightCount = GetAdditionalLightsCount(); |
  |  | for (uint lightIndex = 0; lightIndex < pixelLightCount; ++lightIndex) |
  |  | { |
  |  | Light light = GetAdditionalLight(lightIndex, inputData.positionWS); |
  |  | color += DiffuseLighting(light, inputData.normalWS, surfaceData.albedo); |
  |  | } |
  |  |  |
  |  | return color; |
  |  | } |
  ```

# **快速调用方法**

在URP着色器中调用漫反射的标准方式：

## ‌**自定义着色器方式**‌：

```
|  |  |
| --- | --- |
|  | hlsl |
|  | // 片元着色器示例 |
|  | half4 frag(Varyings input) : SV_Target |
|  | { |
|  | // 初始化表面数据 |
|  | SurfaceData surfaceData; |
|  | InitializeStandardLitSurfaceData(input.uv, surfaceData); |
|  |  |
|  | // 准备光照输入数据 |
|  | InputData inputData; |
|  | InitializeStandardLitInputData(input, surfaceData.normalTS, inputData); |
|  |  |
|  | // 计算漫反射光照 |
|  | half4 color = UniversalFragmentBlinnPhong(inputData, surfaceData); |
|  |  |
|  | return color; |
|  | } |
```

## ‌**Shader Graph可视化方式**‌：

* 使用"Dot Product"节点计算N·L
* 使用"Multiply"节点混合颜色
* 连接至"Fragment"输出的Base Color通道

# **URP选择此方案的原因**

## ‌**性能优化**‌：

* 使用half精度计算
* 内置光照循环优化
* 最小化分支预测

## ‌**物理一致性**‌：

* 线性空间计算
* 正确的光照衰减（通过颜色与距离和阴影衰减相乘做到）

## ‌**扩展性**‌：

* 支持多光源场景
* 与PBR工作流兼容
* 可扩展自定义光照模型

## ‌**跨平台支持**‌：

* 适配移动端TBDR架构
* 支持SRP Batcher优化
* 兼容各种渲染路径

在URP中，这种实现方式既保持了经典光照模型的直观性，又通过现代渲染管线的优化手段确保了高性能表现，特别适合需要跨平台部署的项目。

---

> [【从UnityURP开始探索游戏渲染】](https://github.com):[鹊桥加速](https://queqiao6.com)**专栏-直达**

（欢迎*点赞留言*探讨，更多人加入进来能更加完善这个探索的过程，🙏）
