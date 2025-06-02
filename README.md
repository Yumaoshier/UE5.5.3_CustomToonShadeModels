# UE5.5.3_CustomToonShadeModels
这里是UE5.5.3版本的自定义渲染卡通模型UE部分的代码，build完成即可创建项目使用自定义的卡通渲染模型。
修改的文件有：
--自定义渲染模型--
1.新增渲染模型相关：
  1.1. 增加枚举类型
    a. EngineTypes.h
    b. MaterialShader.cpp
    c. PixelInspectorResult.h (添加宏，增加PixelInspector（像素检查器）对新光照模型的解析枚举数字)
    d. PixelInspectorResult.cpp (为增加的宏解析定义新增加的光照模型枚举)
  1.2. 修改Shader编译部分（添加着色器环境定义）
    e. MaterialHLSLEmitter.cpp (往Shader里添加宏定义,让Shader走新的分支判断)
    f. HLSLMaterialTranslator.cpp (添加Shader中的宏定义)
2.新增自定义材质属性相关：
  a. SceneTypes.h (添加自定义参数)
  b. Material.h
  c. MaterialExpressionMakeMaterialAttributes.h
  d. HLSLMaterialTranslator.cpp
  e. MaterialAttributeDefinitionMap.cpp (初始化自定义参数，Guid,类型，属性名等; 给editor设置显示参数名，可以给已有的参数为不同的渲染模型定义不同的显示名称)
  f. Material.cpp
  g. MaterialExpressions.cpp
  h. DatasmithMaterialExpressions.h
  i. DatasmithMaterialExpressions.cpp
  j. DMMaterialUtils.cpp
  k. MaterialShared.h
  l. ExportMaterialProxy.h
  m. MaterialBakingModule.cpp
  n. MaterialEditor.cpp
  o. MaterialGraph.cpp
  p. MaterialGraphNode_Root.cpp (其中CreateInputPins里，如果想面板默认连接的pin是3位数而不是默认1位，则需要加；而且这里如果加的话，则新加的属性都需要加类型，Float也需要)
  q. MaterialCachedData.cpp
  r. MaterialExpressionHLSL.cpp
  s. MaterialTemplate.ush (添加模板函数，对应d. HLSLMaterialTranslator.cpp (下面也有重复的，主要让加进去的参数可以让shader获得，需要注意区分Vertex Shader获取还是Pixel Shader获取))
  t. MaterialUtilities.cpp
3.给新增的渲染模型相关数据写入GBuffer用于后续渲染算法：
  a. ShaderMaterial.h (启用CustomData GBuffer)
  b. ShaderMaterialDerivedHelpers.cpp
  c. ShaderGenerationUtil.cpp (开启GBuffer的Custom槽)
4.Shader 定义着色模型：
  a. ShadingCommon.ush (添加Shader对新模型的定义, 因为ShadingModelID都储存在GBufferB的a通道的低4位里，总共只能有16个ShadingModelID，这里UE5.5.3已有13个模型，所以一般自定义模型只能再添加3个模型；不过这里后面我会利用宏定义来用一个ShadingModelID扩展多个渲染模型，这样可以添加3个以上模型)
5.开启Shader编译检查：
  a. ConsoleVariables.ini (编译shader是在 UE 里进行的，C++ 改完之后修改 .ush/usf 文件只需进入UE，随时使用Ctrl + Shift + . 即可编译)
6.定义ShadingModel的Shader宏:
  a. Definitions.usf (定义与声明设置好的ShadingModel)
7.Shader设置自定义接口的数据: 
  a. BasePassCommon.ush (自定义数据接口打开，开启GBuffer的CustomData写入权限)
  b. ShadingModelsMaterial.ush (GBuffer中添加自定义数据)
8.基础着色实现(光照渲染):
  a. BasePassPixelShader.usf
  b. ClusteredDeferredShadingPixelShader.usf (新添加的ShadingModel并入光照计算)
  c. ToonShadersCommon.ush (自定义的脚本，主要用来放编码解码函数，为了压缩数据体积, 添加到 Engine\Shaders\Private\ 目录下)
  d. DeferredShadingCommon.ush (#include "ToonShadersCommon.ush")
  e. MaterialTemplate.ush (添加模板函数)
  f. ShadingModels.ush (卡通渲染修改函数文件，添加在函数IntegrateBxDF 上方,这里就是添加渲染模型算法的地方)
  g. DeferredLightingCommon.ush (后向渲染光照计算入口)
9.其他一些处理：
  a. MaterialIRToHLSLTranslator.cpp
  b. ShaderGenerationUtil.cpp

--自定义Pass--
10.添加自定义Pass实现描边:
  添加Mesh Draw Pass：
    1.Mesh Draw Pass的Processor类
    2.定义VS和PS的类，以及Shader的实现
    3.抵用RDG的Render函数
  10.1. Mesh Draw Pass的开发
    a. MeshPassProcessor.h (定义Pass枚举; 修改静态检查中Pass的数量)
    b. 创建新的.h文件和.cpp文件，用来存放新的Processor类和Shader类的实现(可参照Custom Depth Pass):
      b1. OutlinePassRendering.h (实现Processor类的声明，包含一个构造函数和两个成员函数; VS和PS两个Shader类的实现)
      b2. OutlinePassRendering.cpp (将对应的Shader文件绑定到这个Shader类上，这样在编译之后UE才能找到对应的Shader文件; 路径不需要加Shaders文件夹；对Processor中三个函数的实现——构造函数中主要完成对渲染状态的重置和清零；在AddMeshBatch中，则主要用来收集需要在该Pass中进行绘制的Mesh，同时调用Process函数，实现主要的功能；在Process中，获取Shader，设置渲染状态，以及调用BuildMeshDrawCommands构建渲染指令，而这些渲染指令则是最终我们在RDG中去执行的渲染指令)
    c. 新建一个usf文件用来实现描边绘制的Shader (UnrealEngine\Engine\Shaders\Private\Custom\OutlinePassShader.usf)
    d. 声明和实现Render函数
      d1. DeferredShadingRenderer.h (声明Render函数)
      d2. OutlinePassRendering.cpp (将Render的函数依旧放入Processor的CPP文件中，由于需要在RDG中传入绘制所需要的View以及SceneTexture，在此处还需使用Shader宏来定义UniformBuffer，以便传入绘制所需要的参数)
      d3. SceneVisibility.cpp (计算动态网格相关性; 在可见相关性函数中添加渲染指令的构建条件)
      d4. DeferredShadingRenderer.cpp (在Render主函数中调用刚刚实现的Render函数)
11.在Material Editor添加开关 (开启描边Pass)：
  11.1. 添加属性：(这里是bHasOutline，可参考TwoSided)
    a. Material.h
    b. MaterialInstance.h
  11.2. 声明和重载Get方法：
    a. MaterialInterface.h
    b. MaterialInterface.cpp
    c. Material.h
    d. Material.cpp
    e. MaterialInstance.h (Material Instance Editor的时候需要)
    h. MaterialInstanceDynamic.h
  11.3. 实现对应的Get:
    a. MaterialShared.h
    b. MaterialShared.cpp
  11.4. 控制渲染管线:
    a. MaterialRelevance.h
    b. SceneVisibility.cpp (可见性渲染指令)
    c. StaticMeshBatch.h
    d. StaticMeshBatch.cpp
    e. PrimitiveSceneInfo.cpp
12.在MaterialInstance Editor上添加开关(可区别于母材质，单独在子材质开启描边Pass):
  材质实例的面板不是直接调用反射根据类中的成员自动生成的，需要自己绘制相对应的UI布局
  a. FMaterialInstanceBasePropertyOverrides (添加相对应的属性，来对应材质实例重写母材质参数的情况)
  b. FMaterialInstanceParameterDetails (添加相关函数)
  c. FMaterialInstanceParameterDetails (编写界面)
  d. MaterialUtilities.cpp
  e. MaterialEditor.h
  f. PreviewMaterial.cpp
  g. MaterialInstance.cpp
  h. MaterialInstance.h
  i. MaterialInterface.cpp
  j. MaterialShared.cpp
  k. ShaderMaterial.h

--按照自定义Pass Material Editor开关的方法扩展ShadingModelID(这个开关不用额外Pass),使其兼容不同的自定义渲染模型（上述提到的，利用宏定义开关来增加超过3个自定义模型而不浪费ShadingModelID的方法）--
--这里用SHADINGMODELID_CELTOONSKIN来举例，把CELTOONSKIN模型跟SHADINGMODELID_CELTOON放在一起，共同使用同一个ShadingModelID,用材质面板上的bIsToonSkin来切换--
13. ShadingModelID原理 (扩展ShadingModelID的思路)：
  Shader (ShadingCommon.ush) 里定义的SHADINGMODELID_CELTOONSKIN，用PixelInspectorResult.h里定义的PIXEL_INSPECTOR_SHADINGMODELID_CELTOONSKIN和EngineTypes.h的MSM_CelToonSkin（enum EMaterialShadingModel）联系起来，然后MSM_CelToonSkin再通过HLSLMaterialTranslator.cpp和MaterialHLSLEmitter.cpp里的环境定义和Definitions.usf里定义的宏MATERIAL_SHADINGMODEL_CELTOONSKIN联系起来
	a. Definitions.usf：MATERIAL_SHADINGMODEL_CELTOONSKIN宏跟GBuffer有关
	b. ShadingCommon.ush：SHADINGMODELID_CELTOONSKIN跟渲染模型有关，存储在GBuffer.ShadingModelID里
	c. EngineTypes.h：MSM_CelToonSkin跟材质属性和页面显示选择渲染模型有关
 结论：也就是说想要去掉ShadingCommon.ush里的SHADINGMODELID_CELTOONSKIN，EngineTypes.h里定义的MSM_CelToonSkin和Definitions.usf里定义的MATERIAL_SHADINGMODEL_CELTOONSKIN也得去掉，因为他们都是唯一对应联系的，并且PixelInspectorResult.h里定义的PIXEL_INSPECTOR_SHADINGMODELID_CELTOONSKIN也得去掉，否则会根据宏定义数字找到下面的渲染模型；PixelInspectorResult.cpp中DecodeShadingModel方法会用到PIXEL_INSPECTOR_SHADINGMODELID_CELTOONSKIN来获取对应的enum EMaterialShadingModel MSM_CelToonSkin，但是该方法只用于DcodeGBuffer BCDE,如果没有对GBufferBCDE做特殊操作可以直接去掉。因为全部去掉了，上述3个地方及其应用需要用额外的方法来代替区分(这里的难点是需要在Shader和CPP脚本中都能获取到判断该渲染模型的方法)，这里我从自定义Pass中定义的材质开关属性及其宏定义中获得灵感，用同样的方法只是不用于创建Pass，而是用来区别渲染模型，因为它可以从Shader和cpp脚本中直接获得(下面会有详细介绍)。

14. 用材质Editor开关属性判断同一ShadingModelID下不同的渲染模型：
  14.1. 定义材质Editor字段属性：
    a. Material.h
    b. MaterialInstance.h
    c. MaterialInterface.h
    d. MaterialInterface.cpp
    e. Material.cpp
    f. MaterialInstance.cpp
    g. MaterialInstanceDynamic.h
    h. MaterialShared.h
    i. MaterialShared.cpp (用 SET_SHADER_DEFINE(OutEnvironment, MATERIAL_ISTOONSKIN, IsToonSkin()) 把在j. ShaderMaterial.h中定义的MATERIAL_ISTOONSKIN结构体字段定义成Shader Compiler的宏定义，这样即可在各个需要区分渲染模型的Shader文件中直接用#if语句来判断)
    j. ShaderMaterial.h (定义结构体FShaderMaterialPropertyDefines： MATERIAL_ISTOONSKIN)
  14.2. 在MaterialInstance Editor上也添加扩展ShadingModel开关(字段属性及其方法)
    a. MaterialInstanceBasePropertyOverrides.h (添加相对应的属性，来对应材质实例重写母材质参数的情况)
    b. MaterialShared.cpp (初始化在MaterialInstanceBasePropertyOverrides.h定义的properties)
    c. MaterialEditorInstanceDetailCustomization.h (DECLARE_OVERRIDE_MEMBER_FUNCS(IsToonSkin))
    d. MaterialEditorInstanceDetailCustomization.cpp (IMPLEMENT_OVERRIDE_MEMBER_FUNCS(IsToonSkin, true); FMaterialInstanceParameterDetails编写界面: CREATE_BASE_OVERRIDE_ROW(IsToonSkin, IsToonSkin, IsToonSkin))
    e. MaterialUtilities.cpp
    f. MaterialEditor.h
    g. PreviewMaterial.cpp
    h. MaterialInstance.cpp
    i. ShaderMaterial.h 
    j. Material.cpp (IsPropertyActive_Internal给这个方法加一个参数用来判断是否是IsToonSkin)
      
      
