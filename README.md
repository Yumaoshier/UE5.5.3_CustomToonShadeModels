# UE5.5.3_CustomToonShadeModels
这里是UE5.5.3版本的自定义渲染卡通模型UE部分的代码，build完成即可创建项目使用自定义的卡通渲染模型。
修改的文件有：
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
  
