# Unity2021性能优化

## 1.项目环境

### PC环境

Windows10专业版
CPU	Intel(R) Core(TM) i7-7700 CPU @ 3.60GHz   3.60 GHz
GPU	NVIDIA GeForce GTX 1060 6GB
RAM	16.0 GB 2400MHz

### 软件环境

Unity版本：2021.3.3f1c1

模板：3D(URP)

资源：
SUNTAIL - Stylized Fantasy Village V1.3
UnitySetup-iOS-Support-for-Editor-2021.3.1f1c1
UnitySetup-Android-Support-for-Editor-2021.3.1f1c1

## 2.资源导入及编辑器设置

### 资源相关

1.导入 SUNTAIL - Stylized Fantasy Village 1.1.3.unitypackage

2.根据流程进行项目设置

```
URP:
1. Set the Color Space to Linear (Edit > Project Settings > Player > Other Settings)
2. Unpack the URP package, depending on your editor version ("SRP Packages" folder)
3. Set the "SuntailUniversalRenderPipelineAsset" as default or keep your own (Edit > Project Setiings > Graphics > Scriptable Render Pipeline Settings)
```

![image-20220525102901236](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220525102901236.png)

3.在Project Settings 中将 Tag and Layer 设置和 Quality 设置替换为资源设置。

### 编辑器相关

1.Build Settings中设置目标平台，勾选Development Build 和 Autoconnect Profiler，方便烧录到真机上是查看调试信息

## 3.Unity工作流

### Unity工程目录结构及用途

| Type                   |                                                              |
| ---------------------- | ------------------------------------------------------------ |
| Asset文件夹            | 存储和重用的项目资产（需要代码托管）                         |
| Library文件夹          | 项目内部资产数据信息的目录（不需要代码托管，无法打开工程可以删除此文件夹重新生成） |
| Packages文件夹         | 项目的包文件信息（建议代码托管）                             |
| Project Settings文件夹 | 项目设置的信息（建议代码托管）                               |
| UserSettings文件夹     | 用户设置信息（建议代码托管）                                 |
| Temp文件夹             | 使用Unity编辑器打开项目时的临时数据，一旦关闭Unity编辑器也会被删除（不需要代码托管） |
| Logs文件夹             | 项目的日志信息（不包含编辑器日志信息）（不需要代码托管）     |



### Unity Assets目录中的特殊文件夹及用途

| Type                           |                                                              |
| ------------------------------ | ------------------------------------------------------------ |
| Editor文件夹                   | 可以多个                                                     |
| Editor Default Resources文件夹 | 根目录唯一                                                   |
| Gizmos文件夹                   | 根目录唯一                                                   |
| Plugins文件夹                  | 2019后已删除，但仍可使用，仍能保障其中代码编译的优先顺序     |
| Resources文件夹                | 可以多个，建议正式项目中不要有此文件夹，使用Asset bundle进行构建和加载资源 |
| Standard Assets文件夹          | 根目录唯一，其中代码编译优先                                 |
| StreamingAssets文件夹          | 根目录唯一                                                   |

### 忽略导入的文件夹

1. 隐藏的文件夹

2. 以"."开头的文件和文件夹

3. 以"~"结尾的文件和文件夹

4. 扩展名为cvs的文件和文件夹

5. 扩展名为.tmp的文件夹




**Assets目录结构设计（个人习惯，不标准）**

**一级目录设计原则：**

1. 目录尽可能少

2. 区分编辑模式与运行模式

3. 区分工程大版本

4. 访问场景文件、全局配置文件便捷

5. 不在一级目录做资源类别区分，只有Video类视频直接放到StreamAssets下


**二级目录设计原则：**

1. 只区分资源类型

2. 资源类型大类划分要齐全

3. 不做子类型区分

4. 做功能区分

5. 不做生命周期区分

**三级目录设计原则：**

1. Audio/Texture/Models三级目录做子类型区分
2. 其他类型资源可按功能模块/生命周期区分


**四级目录设计原则：**

1. 只有Audio/Texture/Models做四级目录，可按功能和模块/生命周期划分


## 4.资源导入工作流

### 1.手动编写工具 

**优点：**根据项目特点自定义安排导入工作流，并且可以和后续资源制作与大包工作流结合

**缺点：**存在开发和维护成本，会让编辑器菜单界面变得复杂，对新人理解工程不友好

**AssetPostprocessor：**

所有的资源类型都有对应的OnPreprocess接口，它会在资源被导入时调用

编写编辑器代码继承AssetPostprocesser对象，获取Importer自定义完成导入设置修改

伪代码：

```c#
public class XXXAssetPostprocessor : AssetPostprocessor
{
        public void OnPreprocessXXXAsset()
        {
                XXXAssetImporter xxxImporter = (XXXAssetImporter)assetImporter;
                xxxImporter.属性 = xxx
                ...
                xxxImporter.SaveAndReimport();
        }
}
```

参考资料：

[资源审核 - Unity 手册 (unity3d.com)](https://docs.unity3d.com/cn/2021.2/Manual/BestPracticeUnderstandingPerformanceInUnity4.html)

[UnityEditor.AssetPostprocessor - Unity 脚本 API (unity3d.com)](https://docs.unity3d.com/cn/2021.2/ScriptReference/AssetPostprocessor.html)



**注意：**

1.这种方式实现了资源的统一设置，但有时同类资源也有不同的导入设置，这是需要通过不同的文件夹对不同配置需求的同类资源进行区分，或者通过资源文件名区分，这样导入工具就还可以通过对路径的管理，通过做表达式或通配符进行分类配置。并对导入资源设置做持久化，可以继承[ScriptableObject](https://docs.unity3d.com/cn/current/Manual/class-ScriptableObject.html)对象来持久化设置，也可以与第二种方法结合使用Presets来完成，Presets是从ScriptableObject继承而来。

![image-20220530141849581](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220530141849581.png)

2.这样就解决了在资源导入时的资源统一设置问题和同类资源不同导入设置的问题，但在编辑时可能涉及人为误修改导致打包时出现问题，如果仅在打包时重新统一资源有可能导致有些问题只在打包后才出现，所以建议使用新的接口来保证资源统一

**AssetsModifiedProcessor**可以检测在编辑器下，被添加、删除、修改、移动的资源，并回调OnAssetsModified接口，通过该接口可以在资源修改时做相应的资源导入设置

```C#
AssetsModifiedProcessor（新试验接口）:
/// 资源被添加、删除、修改、移动时回调该对象的OnAssetsModified接口
/// <summary>
/// The OnAssetsModified method is called whenever an Asset has been changed in the project.
/// This methods determines if any Preset has been added, removed, or moved
/// and updates the CustomDependency related to the changed folder.
/// </summary>
void OnAssetsModified(string[] changedAssets, string[] addedAssets, string[] deletedAssets, AssetMoveInfo[] movedAssets)
{

        ...
} 
```

参考资料：

[Unity - Manual: Applying default presets to Assets by folder (unity3d.com)](https://docs.unity3d.com/Manual/DefaultPresetsByFolder.html)

### 2.利用Presets功能

Unity2018时加入的功能

**Presets**是将相同属性设置跨多个组件、资源或项目设置保存和应用的资源，该资源运行时没有效果，仅能在Uity编辑器下使用，不会随应用程序导出。

**优点：**使用简单方便，只需要Assets目录结构合理规范即可

**缺点：**无法和后续工作流整合，只适合做资源导入设置。

![image-20220530142445452](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220530142445452.png)

任意资源右上角的都有Presets按钮，打开Presets界面后可以序列化存储当前资源的设置

![image-20220530142624497](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220530142624497.png)

![image-20220530142650421](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220530142650421.png)

这样就可以很方便地应用此资源的设置到其他资源上。

同样资源的导入设置也可以持久化成Presets，而且可以将Presets资源添加到Project Settings -> Preset Manager中

![image-20220530143040629](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220530143040629.png)

这样就可以将统一的资源设置应用到导入的新对象或新资源上

在Preset Manager中可以为每一个Presets添加高级过滤搜索选项，具体支持的符号和通配符如下图：

![image-20220530143256371](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220530143256371.png)

对于在第1种方法中遇到的资源变化问题，也可以同样使用OnAssetsModified接口进行完善，重新进行导入设置
只需将[Unity - Manual: Applying default presets to Assets by folder (unity3d.com)](https://docs.unity3d.com/Manual/DefaultPresetsByFolder.html)中的代码文件放到Editor文件夹下，同时删除Preset Manager中的默认预设并将Preset放到对应资产文件夹中即可。

![image-20220530143842340](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220530143842340.png)

如上图所示，Doors文件夹中的文件不论进行什么修改，都会自动设置为与文件夹下的DoorSound Preset一致，任何人为的修改都会被忽略

[Supporting presets - Unity 手册 (unity3d.com)](https://docs.unity3d.com/cn/2021.3/Manual/SupportingPresets.html)

[Applying default presets to Assets by folder - Unity 手册 (unity3d.com)](https://docs.unity3d.com/cn/2021.3/Manual/DefaultPresetsByFolder.html)



### 3.利用AssetGraph工具

优点：功能全，覆盖Unity资源工作流全流程，节点化编辑，直观

缺点：有一定上手成本，一些自定义生成节点也需要开发，不是Unity标准包，Unity新功能支持较慢。

通过Package Manager -> Add package from git URL...安装AssetGraph

AssetGraph仓库地址：[Unity-Technologies/AssetGraph: Visual Workflow Automation Tool for Unity. (github.com)](https://github.com/Unity-Technologies/AssetGraph)

AssetGraph可以可视化编辑资源流，只需要添加OverWrite Import Settings节点就可以进行资源统一设置，也同样支持各种过滤方式

可以勾选Use As Postprocessor来检测新资源导入的回调流程，并可以通过点击Execute按钮执行整个资源处置流程

![image-20220530145834893](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220530145834893.png)

![image-20220530145909908](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220530145909908.png)

## 5.项目资源分布

1. 根据文件类型分类查看资源分布情况，该资源中材质、纹理和音频占比较大，为主要优化方向，Script及Shader占比较少，优化优先级较低。

2. 关注场景资源中的摄像机数量 (影响渲染流程复杂度) 和灯光数量 (影响光照复杂度及阴影复杂度)。
   	此资源中灯光数量较多，优化优先级较高。

## 6.Quality Settings

1. 检查目标优化平台对应的Quality Setting 以及渲染管线与编辑器平台的差异

2. 渲染管线在对应目标平台的设置 (使用到的管线类型，使用哪些Render Feature，开启哪些中间纹理和管线内置功能)
   如Rendering Path选项为Deferred延迟渲染，会给移动平台带来带宽和显存的压力，优化优先级较高。

## 7.基础信息

1. 将编辑器切换至目标优化平台，查看是否显示正常并运行。

2. 查看编辑器自带的Static信息，统计当前场景平均面数、面数峰值、渲染批次数等信息。

3. 查看Profiler数据，并记录。


## 8.静态资源优化

静态资源的Unity官方步骤为Import Create Build Distribute Load五个步骤。优化也将从五个步骤分别着手。
外部导入资源：模型网格Mesh、纹理、音乐音效、字体、动画、视频等。
内部创建资源：Prefab、Animation Controller、Timeline、RenderTexture、ParticleSystem、VFX等。

### 资源设置检查

[UPR - Unity专业性能优化工具](https://upr.unity.cn/instructions/assetchecker)

使用Unity工具UPR的Asset Checker进行资源监测，生成配置文件后检测Unity工程。

```
assetcheck.exe --project="C:\Users\ianxhtian\Documents\Unity2022_SUNTAIL_Stylized_Fantasy_Village_Optimization" --includePaths="Assets" ----projectId="77fe08eb-3822-4536-80a2-5bba53a4b17f"
```

![image-20220526145335648](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220526145335648.png)

Asset Checker会对相应资源提出优化建议

### Audio文件优化

资源检测报告：

![image-20220526145519612](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220526145519612.png)

1.可以看到该类资源没有开启ForceToMono选项，但该类资源为左右声道声音相同的立体声，如不需要双声道的音频，可以强制转换为单声道节省声音的内存占用，降低包体大小（尤其是在移动端，几乎无法察觉）。

2.尽可能选择未压缩的Wav文件作为音频源文件，根据不同平台的支持和不同的用途进行不同的压缩。

| 音频压缩格式 | 特点                                            |
| ------------ | ----------------------------------------------- |
| Vorbis       | 移动端通用的压缩格式                            |
| ADPCM        | 压缩比较差但解码速度快 适合快速多次播放的短音频 |
| MP3          | 一般可用于ios平台 也适用于不循环的音频          |

3.音频文件的采样率
移动平台一般不需要48000Hz采样率的音频，建议设置为22050Hz（对音质影响最小的最低设置）来减少不必要的浪费。
具体操作为修改 Sample Rate Setting 为 Override Sample Rate 进行复写操作

![](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220526145534610.png)

4.不同音乐音效类型，要使用不同的Load Type

| Load Type            | 特点                                               |
| -------------------- | -------------------------------------------------- |
| Decompress On Load   | 默认选项 适用于音频压缩后大小<200kb的音效文件      |
| Compressed in Memory | 适用于音频压缩后大小>200kb的音效文件               |
| Streaming            | 适用于背景音效、音乐、较长较大文件，避免加载时卡顿 |

5.在游戏静音时，不能只简单地将音量调整为0，应销毁音频AudioSource组件，将其从内存中完全卸载

6.音乐音频一般不会成为性能瓶颈，但经过优化可以减少内存的使用和包体的大小

7.如使用复杂，第三方逻辑库的情况除外

### Audio文件优化结果



![image-20220526164538437](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220526164538437.png)

优化前Audio Memery使用了**76.4MB**的内存

![image-20220526164951708](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220526164951708.png)

优化后Audio Memery只使用了**7.3MB**的内存，**降低了90%**

只**增加了2%**的总Audio Cpu消耗用于Streaming，这个交换是十分划算的

通过对音频文件大小的优化，使包体大小从**560MB减少至542MB**

### Modle文件优化

#### Unity模型导入流程

![image-20220526195012289](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220526195012289.png)

尽量使用fbx模型文件作为导入链

##### DCC工具导出设置

1. 在DCC中统一单位

2. 导出的网格必须是多边形拓扑，不能是贝塞尔曲线、样条曲线、细分曲面等

3. 在导出之前确保所有变形体Deformers都烘焙到网格模型上

4. 不要将模型使用的纹理随模型导出

5. 如果需要导入Blend shape normals时，必须指定光滑组Smooth groups

6. 导出时不要携带如摄像机、灯光、材质等场景信息


##### 原始模型对性能的影响

1. 最小化面数，不需要微三角形面（一个三角面只包含个位数的像素，可使用LOD或者美术调整），三角面尽量分布均匀

2. 合理的拓扑结构与平滑组，尽可能是闭包（会产生额外三角形定点和边，影响Unity烘焙）

3. 尽量少的材质个数（多的话会引起Shader和贴图的暴涨）

4. 尽可能少的蒙皮网格（同一个模型下尽量使用相同的蒙皮网格）

5. 尽可能少的骨骼数量（过多的骨骼可能引起蒙皮动画的CPU和GPU的性能瓶颈）

6. FK与K节点分离，导出时删除K骨骼节点


##### 模型导入设置

| Scene              |                   |
| ------------------ | ----------------- |
| Import Blendshapes | 默认开启 建议关闭 |
| Import Visibility  | 默认开启 建议关闭 |
| Import Lights      | 默认开启 建议关闭 |
| Import Cameras     | 默认开启 建议关闭 |

| Meshes           |                                          |
| ---------------- | ---------------------------------------- |
| Mesh Compression | 默认关闭 开启须确认模型是否正确          |
| Read/Whrite      | 默认关闭 做skin mesh动画或需要修改时开启 |
| 其余默认         |                                          |

| Geometry |      |
| -------- | ---- |
| 默认     |      |

#### Modle资源检测报告

![image-20220526201416007](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220526201416007.png)

- 将所有静态模型的Animation Type设置为None

- 将Animation标签下的ImportAnimation取消

- 将Materials标签下的Material Creation Mode设置为None


![image-20220526201543684](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220526201543684.png)

限制为500顶点，过于严苛，无修改必要

![image-20220526201556629](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220526201556629.png)

### Modle文件优化结果

优化结果差别不大，因模型作者制作时十分规范无优化空间，且场景模型源文件只50MB优化度有限，优化的重点应该在运行时模型内存（350MB左右）



### Texture文件优化

#### 纹理的基础概念

##### Unity纹理类型

| Type                      |                                                          |
| ------------------------- | -------------------------------------------------------- |
| Default                   | 默认的纹理类型格式（绝大多数资源使用）                   |
| Normal map                | 法线贴图，可将颜色通道转换为适合实时法线贴图格式         |
| Editor GUI and Legacy GUI | 在编辑器GUI控件上使用纹理请选择此类型                    |
| Sprite(2D and UI)         | 在2D游戏中使用的精灵(Sprite)或UGUI使用的纹理请选择此类型 |
| Cursor                    | 鼠标光标自定义纹理类型                                   |
| Cookie                    | 用于光照Cookie剪影类型的纹理                             |
| Lightmap                  | 光照贴图类型的纹理，编码格式取决于不同的平台             |
| Single Channel            | 如果原始图片文件只有一个通道，请选择此类型               |



##### 纹理大小

选择合适纹理大小应尽量遵循以下经验：

1. 不同平台、不同硬件配置选择不同的纹理大小，Unity下可以采用bundle变体设置多套资源、通过Mipmap限制不同平台加载不同level层级的贴图。

2. 根据纹理用途的不同选择不同的纹理加载方式，如流式纹理加载Texture Streaming、稀疏纹理Sparse Texture、虚拟纹理VirtualTexture等方式。

3. 不能让美术人员通过增加纹理大小的方式增加细节，可以选择细节贴图DetailMap或增加高反差保留的方式。

4. 在不降低视觉效果的情况下尽量减小贴图大小，最好的方式是纹理映射的每一个纹素的大小正好符合屏幕上显示像素的大小，如果纹理小了会造成欠采样，纹理显示模糊，如果纹理大了会造成过采样，纹理显示噪点。这一点做到完美平衡很难保障，可以充分利用Unity编辑器下SceneView->DrawMode->Mipmap来查看在游戏摄像机视角下哪些纹理过采样，哪些纹理欠采样，进而来调整纹理大小。




##### 纹理颜色空间

默认大多数图像处理工具都会使用sRGB颜色空间处理和导出纹理。但如果你的纹理不是用作颜色信息的话，那就不要使用sRGB空间，如金属度贴图、粗糙度贴图、高度图或者法线贴图等。一旦这些纹理使用sRGB空间会造成视觉表现错误。



##### 纹理压缩

纹理压缩是指图像压缩算法，保持贴图视觉质量的同时，尽量减小纹理数据的大小。默认情况下我们的纹理原始格式采用PNG或TGA这类通用文件格式，但与专用图像格式相比他们访问和采样速度都比较慢，无法通用GPU硬件加速，同时纹理数据量大，占用内存较高。所以在渲染中我们会采用一些硬件支持的纹理压缩格式，如ASTC 、ETC、ETC2、DXT等。

如下图为未压缩、ETC2、ASTC6x6三种格式文件大小对比，我们可以看到在质量相差不大的情况下ASTC6x6压缩下大小可以减少到接近未压缩的十分之一。

![img](https://i0.hdslb.com/bfs/article/eec3ca42c56311ed1a9a342387478fbde0689a87.png@942w_315h_progressive.webp)


​                                                                                                   未压缩、ETC2、ASTC6x6三种格式文件大小对比



##### 纹理图集

纹理图集是一系列小纹理图像的集合，

优点：

1. 采用共同纹理图集的多个静态网格资源可以进行静态合批处理，减少DrawCall调用次数。

2. 纹理图集可以减少碎纹理过多，因为他们打包在一个图集里，通过压缩可以更有效的利用压缩，降低纹理的内存成本和冗余数据。


缺点

1. 美术需要合理规划模型，并且要求模型有相同的材质着色器，或需要制作通道图去区分不同材质。制作和修改成本较高。




##### 纹理过滤

- Nearest Point Filtering：临近点采样过滤最简单、计算量最小的纹理过滤形式，但在近距离观察时，纹理会呈现块状。

- Bilinear Filtering：双线性采样过滤会对临近纹素采样并插值化处理，对纹理像素进行着色。双线性过滤会让像素看上去平滑渐变，但近距离观察时，纹理会变得模糊。

- Trilinear Filtering：三线性过滤除与双线性过滤相同部分外，还增加了Mipmap等级之间的采样差值混合，用来平滑过度消除Mipmap之间的明显变化。

- Anisotropic Filtering：各向异性过滤可以改善纹理在倾斜角度下的视觉效果，跟适合用于地表纹理。




##### 纹理Mipmap

Mipmap纹理

逐级减低分辨率来保存纹理副本。相当于生成了纹理LOD，渲染纹理时，将根据像素在屏幕中占据的纹理空间大小选择合适的Mipmap级别进行采样。

优点：

- GPU不需要在远距离上对对象进行全分辨率纹理采样，因此可以提高纹理采样性能。

- 同时也解决了远距离下的过采样导致的噪点问题，提高的纹理渲染质量。


缺点：

- 由于Mipmap纹理要生成低分辨率副本，会造成额外的内存开销。




#### 纹理导入设置

详见[Texture Import Settings - Unity 手册 (unity3d.com)](https://docs.unity3d.com/cn/current/Manual/class-TextureImporter.html)

**Texture Shape**

| Type     |                                                              |
| -------- | ------------------------------------------------------------ |
| 2D       | 最常用的2D纹理，默认选项                                     |
| Cube     | 一般用于天空和与反射探针，默认支持Default、Normal、Single Channel几种类型纹理，可以通过Assets > Create > Legacy > Cubemap生成，也可以通过C#代码 Camera.RenderToCubemap在脚本中生成 |
| 2D Array | 纹理数组，可以极大提高大量相同大小和格式的纹理访问效率，但需要特定平台支持，可以通过引擎SystemInfo.supports2DArrayTextures 接口运行时查看是否支持。 |
| 3D       | 通过纹理位图方式存储或传递一些3D结构话数据，一般用于体积仿真，如雾效、噪声、体积数据、距离场、动画数据等信息，可以外部导入，也可运行时程序化创建。 |



**sRGB**

见上方纹理颜色空间



![image-20220527110454976](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220527110454976.png)

**Alpha Source**

默认选择Input Texture Alpha就好，如果确定不使用原图中的Alpha通道，可以选择None。另外From Gray Scale我们一般不会选用

**Alpha Is Transparency**

指定Alpha通道是否开启半透明，如果位图像素不关心是否要半透明可以不开启此选项。这样Alpha信息只需要占1bit。节省内存

**Ignore Png file gamma**

是否忽略png文件中的gamma属性，这个选项是否忽略取决于png文件中设置不同gamma属性导致的显示不正常，一般原图制作流程没有特殊设置，这个选项一般默认就好。 

![image-20220527110624107](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220527110624107.png)

**Non-Power of 2**

见上方纹理大小

**Read/Write**

开启此选项会导致内存量增加一倍，默认不开启，除非脚本逻辑中需要动态读写该纹理时需要打开此选项。

**Streaming Mipmaps**

使用默认设置

**Virtual Texture Only**

使用默认设置

**Generate Mip Maps**

什么时候不需要生成MipMaps？

1.2D场景

2.固定视角，摄像机无法缩放远近

**Border Mip Maps** 

默认不开启，只有当纹理的是Light Cookies类型时，开启此选项来避免colors bleeding现象导致颜色渗透到较低级别的Mip Level纹理边缘上

**MipMap Filtering**

| Type   |                                            |
| ------ | ------------------------------------------ |
| Box    | 最简单，随尺寸减小，Mipmap纹理变得平滑模糊 |
| Kaiser | 避免平滑模糊的锐化过滤算法                 |

**Mip Maps Preserve Coverage**

只有需要纹理在开启mipmap后也需要做Alpha Coverage时开启。默认不开启。

**Fadeout MipMaps**,

纹理Mipmap随Mip层级淡化为灰色，一般不开启，只有在雾效较大时开启不影响视觉效果。 



**纹理过滤设置**

1. 使用双线性过滤平衡性能和视觉质量。

2. 有选择地使用三线性过滤，因为与双线性过滤相比，它需要更多的内存带宽。

3. 使用双线性和 2x 各向异性过滤，而不是三线性和 1x 各向异性过滤，因为这样做不仅视觉效果更好，而且性能也更高。

4. 保持较低的各向异性级别。仅对关键游戏资源使用高于 2 的级别。


**纹理压缩设置**

详见：[Recommended, default, and supported texture formats, by platform - Unity 手册 (unity3d.com)](https://docs.unity3d.com/cn/current/Manual/class-TextureImporterOverride.html)

![image-20220527111637807](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220527111637807.png)

![img](https://i0.hdslb.com/bfs/article/544df923ae4e751ce401f95ebd0b51942c25bef4.png@942w_425h_progressive.webp)

​																	各个平台默认纹理压缩格式

#### 其他纹理类型问题

1. 纹理图集大小设置不合理，图集利用率低

2. 大量只有颜色差异的图片

3. UI背景贴图而不采用9宫格缩放的图

4. 纯色图没有使用Single Channel

5. 不合理的半透明UI，占据大量屏幕区域，造成Overdraw开销

6. 大量2D序列帧动画，而且图片大，还不打图集

7. 不合理的通道图利用方案




#### Texture资源检测报告

![image-20220530094330395](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220530094330395.png)

长宽不是2的幂次的纹理资源会导致内存占用过大且无法压缩

![image-20220530094447748](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220530094447748.png)

纹理的alpha通道不包含有效信息是将Alpha Source设置为None来节省内存

![image-20220530094551832](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220530094551832.png)

有些纯色纹理可以通过降低纹理大小或者直接用参数代替

![image-20220530094730629](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220530094730629.png)

检查Repeat Wrap模式是否设置正确，错误时更改为Clamp Wrap模式

![image-20220530094916509](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220530094916509.png)

重复纹理改为公用并删除节省资源占用和包体大小

![image-20220530094951844](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220530094951844.png)

检查纹理的大小，结合Unity编辑器判断是否过采样，并将过采样的纹理大小降低

### Texture文件优化结果

![image-20220530095609315](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220530095609315.png)

![image-20220530095536262](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220530095536262.png)

纹理的内存使用减少140MB

![image-20220530095738047](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220530095738047.png)

![image-20220530095752691](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220530095752691.png)

包体大小降低120MB

优化前后画面无肉眼可见差异

### Animation文件优化

#### Rig



##### Animation Type

| Type     |                        |
| -------- | ---------------------- |
| None     | 无动画                 |
| Legacy   | 旧版动画，请勿使用     |
| Generic  | 通用骨骼框架，默认使用 |
| Humanoid | 人形骨骼框架           |

##### 选择原则

- 无动画选择None
- 非人形动画选择Generic

- 人形动画

- 形动画需要Kinematices或Animation Retargeting功能，或者没有有自定义骨骼对象时选择Humanoid Rig

- 其他都选择Generic Rig，在骨骼数差不多的情况下,Generic Rig会比Humanoid Rig省30%甚至更多的CPU的时间。



**优化重点：**

##### Skin Weights

默认4根骨头，但对于一些不重要的动画对象可以减少到1根，节省计算量（如体育游戏观众席），也可以在Unity Project Settings->Quality中为不同平台做不同设置

##### Optimize Bones

建议开启，在导入时自动剔除没有蒙皮顶点的骨骼

##### Optimize Game Objects

在Avatar和Animatior组件中删除导入游戏角色对象的变换层级结构，而使用Unity动画内部结构骨骼，消减骨骼transform带来的性能开销。可以提高角色动画性能, 但有些情况下会造成角色动画错误，这个选项可以尝试开启但要看表现效果而定。注意如果你的角色是可以换装的，在导入时不要开启此选项，但在换装后在运行时在代码中通过调用AnimatorUtility.OptimizeTransformHierarchy接口仍然可以达到此选项效果。



#### Animation

##### Bake Animation

只对应于maya等dcc工具的原始文件格式，并且文件中使用了布料流体等烘培动画时才有用（fbx文件时显示禁用）。

##### Resmple Curves

将动画曲线重新采样为四元数数值，并为动画每帧生成一个新的四元数关键帧，仅当导入动画文件包含尤拉曲线时才会显示此选项

##### Anim.Compression

| Type                               |                                                             |
| ---------------------------------- | ----------------------------------------------------------- |
| Off                                | 不压缩,质量最高，内存消耗最大                               |
| Keyframe Reduction                 | 减少冗余关键帧，减小动画文件大小和内存大小。                |
| Keyframe Reduction and Compression | 减小关键帧的同时对关键帧存储数据进行压缩，只影响文件大小。  |
| Optimal                            | 仅适用于Generic与Humanoide动画类型，Unity决定如何进行压缩。 |

1. 无论压不压缩，Unity中的动画都与dcc中的原始动画有些许区别

2. Keyframe Reduction 不同Error是按百分比和当前单位计算


##### Animation Custom Properties

导入用户自定义属性，一般对应DCC工具中的extraUserProperties字段中定义的数据

#### 动画曲线数据信息

| Type         |                                                              |
| ------------ | ------------------------------------------------------------ |
| Curves Pos   | 位置曲线数量                                                 |
| Quaternion   | 四元数曲线 Resample Curves开启会有                           |
| Euler        | 尤拉曲线                                                     |
| Scale        | 缩放曲线                                                     |
| Muscles      | 肌肉曲线，Humanoid类型下会有                                 |
| Generic      | 一般属性动画曲线，如颜色，材质等                             |
| PPtr         | 精灵动画曲线，一般2D系统下会有                               |
| Curves Total | 曲线总数（越大时播放性能越差）                               |
| Constant     | 优化为常数的曲线（Keyframe Reduction中Error越大数量越多，一般不参与采样，有更高的效率） |
| Dense        | 使用了密集数据（线性插值后的离散值）存储，选择Optimal时自动选择 |
| Stream       | 使用了流式数据（插值的时间和切线数据）存储，选择Optimal时自动选择 |

#### 动画文件导入设置优化后信息查看原则

1. 效果差异（与原始制作动画差异是否明显）

2. 曲线数量（总曲线数量与各种曲线数显，常量曲线比重大更好）

3. 动画文件大小（以移动平台为例，动画文件在小几百k或更少为合理，查过1M以上的动画文件考虑是否进行了合理优化）


### Animation文件优化结果

![image-20220530104311605](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220530104311605.png)

项目中仅有三个动画文件且曲线数量较少，无优化空间。



## 9.零碎性能问题

### 性能问题可能的情况

瓶颈可能性按由高到低的顺序排列（个人经验总结）

- CPU利用率

- 带宽利用率

- CPU/GPU强制同步

- 片元着色器指令

- 几何图形到CPU到GPU的传输

- 纹理CPU到GPU的传输

- 顶点着色器指令

- 几何图形复杂性




**1.Unity2017后Transform的储存方式更改为连续存储，导致是否有父节点对游戏Transform的修改产生影响**

Unite提供的在1000万次测试后数据为下：

![image-20220530155400363](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220530155400363.png)

效率差距近十几倍

**2.如果游戏场景里有很多GameObject，搜索其中一个或一些使用tag搜索效率极高**

| 对象数量 | 搜索方法              | 执行1000次     | 执行10000次   | 执行100000次 |
| -------- | --------------------- | -------------- | ------------- | ------------ |
| 100      | Find                  | 0.002197409秒  | 0.0190155秒   | 0.1939484秒  |
| 100      | FindGameObjectWithTag | 0.0002516746秒 | 0.001338196秒 | 0.01151428秒 |
| 100      | FindObjectOfType      | 0.2374741秒    | 2.544969秒    | 5.801984秒   |
| 1000     | Find                  | 0.037534秒     | 0.357311秒    | 3.543825秒   |
| 1000     | FindGameObjectWithTag | 0.000655秒     | 0.002315秒    | 0.018519秒   |
| 1000     | FindObjectOfType      | 0.157907秒     | 1.552248秒    | 16.20919秒   |
| 10000    | Find                  | 0.429361秒     | 4.274960秒    | 42.64132秒   |
| 10000    | FindGameObjectWithTag | 0.000346秒     | 0.001686秒    | 0.016235秒   |
| 10000    | FindObjectOfType      | 1.375987 秒    | 14.24521秒    | 太长省略     |



## 10.物理

### Unity中的物理解决方案

1. Box2D
2. Nvidia PhysX

3. Unity Physics

4. Havok Physics for Unity


### Unity中的物理组件Collider部分的优化

- Trigger与Collider

- Trigger对象的碰撞会被物理引擎所忽略，通过OnTriggerEnter/Stay/Exit函数回调

- Collider对象由物理引擎触发碰撞，通过OnCollisionEnter/Stay/Exit函数回调

- Trigger对象不需要RigidBody组件，Collider对象必须至少有一个Collider对象有RigidBody组件

- Trigger对象更高效


### Unity中的物理组件Collider部分的优化

- 尽量少使用MeshCollider，可以用简单Collider代替，即使用多个简单Collider组合代替也要比复杂的MeshCollider来的高效
- MeshCollider是基于三角形面的碰撞
- MeshCollider生成的碰撞体网格占用内存也较高
- MeshCollider即使要用也要尽量保障其是静态物体
- 可以通过PlayerSetting选项中勾选Prebake Collision Meshes选项来在构建应用时预先Bake出碰撞网格。

### Unity中的物理组件RigidBody部分的优化

- Kinematic与RigidBody
- Kinematic对象不受物理引擎中力的影响，但可以对其他RigidBody施加物理影响。

- RigidBody完全由物理引擎模拟来控制，场景中RigidBody数量越多，物理计算负载越高

- 勾选了Kinematic选项的RigidBody对象会被认为是Kinematic的，不会增加场景中的RigidBody个数

- 场景中的RigidBody对象越少越好


### Unity中的RayCast与Overlap部分的优化

- Unity物理中RayCast与Overlap都有NoAlloc版本的函数，在代码中调用时尽量用NoAlloc版本，这样可以避免不必要的GC开销

- 尽量调用RayCast与Overlap时要指定对象图层进行对象过滤，并且RayCast要还可以指定距离来减少一些太远的对象查询

- 此外如果是大量的RayCast操作还可以通过RaycastCommand的方式批量处理，充分利用JobSystem来分摊到多核多线程计算。


## 11.动画

### Animation的一些细节

1. 播放单个AnimationClip速度，Legacy Animation系统更快，因为老系统是直接采样曲线并直接写入对象Transform
2. 针对动画的缩放曲线比位移、旋转曲线开销更大
3. 常数曲线不会每帧写入场景，更高效

### Animator的一些细节 

1. 不要使用字符串来查询Animator
2. 使用曲线标记来处理动画事件
3. 使用Target Marching函数来协助处理动画
4. 将Animator的CullingMode设置成Based On Renderers来优化动画，并禁用SkinMesh Renderer的Update When Offscreen属性来让角色不可见时动画不更新

![img](https://i0.hdslb.com/bfs/article/e56e0dcd48b848a3557e512d9582cc346f3059de.png@942w_597h_progressive.webp)

Internal Animation Update

### Animator VS Animation 

- Animation可以将任何对象属性制作成Animation Clip, Animator是将Animaiton Clip组织到状态机流程图中使用

- Animation与Animator播放动画时的效率是有个临界点的，这个临界点是根据动画曲线条数来的，当动画曲线条数小于这个临界点时Animation快，当动画曲线条数大于这个临界点时Animator快

- 当Cpu核数较少时，Animation播放动画有优势，当Cpu核数较多时，Animator表现会更好

- Animator Controller Graph中的所有动画节点的Animation Clip都会载入到内存中，当有海量动画状态机节点时，内存开销较大


### Playable API VS Animator 

**Playable API优点**

| Playable API优点                                             |
| ------------------------------------------------------------ |
| 支持动态动画混合，可为场景中的对象提供自己的动画，并可以动态添加到PlayableGraph当中使用 |
| 允许创建播放单个动画，而并不会产生创建和管理AnimatorController资源所涉及的开销，可更加灵活的控制PlayableGraph的数据流，可以插入自定义的AimationJob。 |
| 可以控制动画文件加载策略，按需加载、异步加载等               |
| 允许用户动态创建混合图，并直接逐帧控制混合权重（甚至可以混合AniationClip与AnimatorController动画） |
| 可以运行时动态创建，根据条件添加可播放节点。而不需要提前提供一套PlayableGraph运行时启动和禁用节点，可以做到自由度更高的override机制 |
| 可加载自定义配置数据，更加方便的和其他游戏系统整合           |

**Playable API缺点**

| **Playable API缺点**             |
| -------------------------------- |
| 没有直接使用Animator直观         |
| 混合模式没有现成的，需要自己实现 |
| 需要开发更多的配套工具           |
| 有一定学习成本                   |

### 解决方案选择 

1.一些简单、少量曲线动画可以使用Animation或动画区间库如Dotween\iTween等完成，如UI动画，Transform动画等。

2.角色骨骼蒙皮动画如果骨骼较少，Animation Clip资源不多，对动画混合表现要求不高的项目可以采用Legacy Animation。注意控制总体曲线数量

3.一些角色动画要求与逻辑有较高的交互、并且动画资源不多的项目可以直接用Animator Graph完成

4.一些动作游戏，对动画混合要求较高、有一些高级动画效果要求、动画资源量庞大的项目，建议采用Animator+Playable API扩展Timeline的方式完成。

## 12.性能优化

### Unity下常见的等待函数

| Type                                                |                                                              |
| --------------------------------------------------- | ------------------------------------------------------------ |
| WaitForTargetFPS                                    | 等待达到目标帧率，一般这种情况CPU与GPU都没什么负载问题（垂直同步中出现） |
| Gfx.WaitForGfxCommandsFromMainThread/WaitForCommand | 渲染线程已经准备接受新的渲染命令，一般瓶颈在CPU              |
| Gfx.WaitForPresentOnGfxThread/VaitForPresent        | 主线程等待渲染线程绘制完成，一般瓶颈在GPU                    |
| WaitForJobGroupID                                   | 等待工作线程完成，一般瓶颈在CPU                              |



### 静态资源优化后Profiler结果

![image-20220531091510033](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220531091510033.png)

![image-20220531091616601](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220531091616601.png)

### Xcode Metal Capture报告

![image-20220531101958309](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220531101958309.png)

### 工程优化重点

1. GPU与带宽可能是主要瓶颈

2. 渲染流程与效果优化

3. 渲染中生成资源优化

4. DrawCall SetPassCall过多

5. 片元着色器

6. 渲染三角形（最后考虑）


### SSAO优化

![image-20220531104611751](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220531104611751.png)

#### 1.**DownSample**  

降低中间生成纹理的分辨率，影响性能，URP下默认为1/2，可视情况改为1/4（与Intensity调节配合保证效果）

原设置：

![image-20220531105021058](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220531105021058.png)

修改为：

![image-20220531105046546](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220531105046546.png)

其次URP中只对AO第一张**m_AOPassDescripto**r进行了降采样处理，可修改为每一步都进行降采样处理，也可以拓展SSAO参数暴露到编辑器中，指定每一步采样各自的降采样率来达到分别控制的效果

原设置：

![image-20220531105234695](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220531105234695.png)

修改为：

![image-20220531105301765](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220531105301765.png)



#### 2.**After Opaque**

将计算与应用SSAO放在不透明物体渲染之后，以改善在tiled-based架构GPU上的渲染性能（可能会造成物理理论上的不精确，但移动端建议开启）

#### 3.Source

项目为延迟渲染，默认生成Depth和Normals信息，无法修改

#### 4.Intensity

AO强度设置，只影响混合系数，不影响性能，搭配其他优化进行使用

#### 5.Radius

采样半径，数值越大性能消耗越大，在可接受的视觉效果范围内尽量调低半径提升强度

#### 6.Direct Lighting Strength

指与受直接光光照的像素混合的系数，用来调节表现效果，不影响性能

#### 7.Sample Count

采样次数，直接影响采样循环数，影响性能开销，尽量保证最小采样次数

#### 最终修改结果

![image-20220531110319565](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220531110319565.png)

**开启降采样并将降采样系数调整为1/4，并应用至所有采样步骤。**

**降低采样次数和半径，并提升强度来平衡性能与效果表现**

#### 最终优化结果

![image-20220531111448355](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220531111448355.png)



**SSAO的耗时由24ms降低至1.8ms，共降低约22ms**

**由于SSAO占用的资源减少空出带宽，减少了其它环节的阻塞，渲染总流程耗时由60ms降低至25ms，共降低约35ms**



![image-20220531123358597](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220531123358597.png)



**UPR报告显示平均帧率由12.82提升为20.46，提高约8帧**



![image-20220531123451834](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220531123451834.png)



**纹理资源的内存占用也降低了31MB**



#### 优化前后画面对比



![image-20220531125058453](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220531125058453.png)

![image-20220531125114359](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220531125114359.png)

**帧数明显提升，表现效果无可明显察觉的差异**



#### SSAO进一步优化方法

1. 使用HBAO(horizon-based ambient occlusion)或GTAO(Ground Truth Ambient Occlusion)方案替代SSAO
2. 针对SSAO的Shader:指令做进一步优化

3. 可以采用烘焙AO到光照贴图的方案替换SSAO方案



### AA反走样优化

#### AA种类

| Type |                                                              |
| ---- | ------------------------------------------------------------ |
| SSAA | 超级采样抗锯齿Super-Sampling Anti-Aliasing                   |
| MSAA | 多重采样抗锯齿Multi Sampling Anti-Aliasing                   |
| FXAA | 快速近似抗锯齿Fast Approximate Anti-Aliasing                 |
| SMAA | 增强子像素形变抗锯齿Enhanced Subpixel Morphological Anti-Aliasing |
| TAA  | 时间序列抗锯齿Temporal Anti-Aliasing                         |
| DLSS | 基于深度学习的超级采样Deep Learning Super Sampling           |

#### AA优缺点分析

| 方案 | URP支持                                                      | 优点                                       | 缺点                                                         |
| ---- | ------------------------------------------------------------ | ------------------------------------------ | ------------------------------------------------------------ |
| MSAA | URP默认支持，适合绝大多数非延迟渲染游戏                      | 显卡硬件支持，反走样效果较好               | 仅支持前向渲，静态画面表现最好，运动画面一般，支持MRT的情况下效率较差，只能消除Geometryl的反走样，对于高光像素部分无能为力 |
| FXAA | URP默认支持，有Quality与Consolei两个不同版本，Quality版本需要PPV2，适合一些对低端硬件有要求、对画面模糊不敏感的特殊游戏 | 后处理支持，开销非常小，适合移动端         | 只有一遍pass，没有格外像素辅助，在高频颜色变化快的地方，动态场景会出现闪烁，对图形所有颜色边缘进行柔化处理，导致画面整体较模糊 |
| SMAA | URP默认支持，适合一些对低端硬件有要求的特殊游戏              | 后处理支持，开销相对较小，适合低端PC端     | 没有格外像素辅助，在高频颜色变化快的地方，动态场景会出现闪烁，三次pass,开销相对FXAA高一些，切换RT开销在手机上可能造成开销，边缘处理比FXAA精细些，但依然有模糊表现 |
| TAA  | URP未来支持，可用PPV2扩展支持                                | 支持延迟渲染，反走样效果较好，性能开销较小 | 动态场景下，低帧率下可能出现鬼影无法处理半透明物体、贴图序列帧动画物体需要额外内存开销，需要MotionVector Buffer |

**平均效率：FXAA>SMAA>TAA>MSAA**

**平均质量：MSAA>SMAA>TAA>FXAA**



#### 优化部分

![image-20220531142327051](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220531142327051.png)

从Xcode Metal Capture报告中可以看出，项目中使用SMAA，耗时为3.24ms

于是有以下三种优化选择：

1.

![image-20220531142536516](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220531142536516.png)

![image-20220531142756892](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220531142756892.png)

在工程Camera中，可以调节SMAA的质量选项为Low后，SMAA耗时由3.24ms降低至2.52ms，主要下降的部分为SMAA的第二个pass

![image-20220531143332988](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220531143332988.png)

通过查看SMAA shader的源码可以得知，不同质量挡位之间只有SMAA的阈值（SMAA_MAX_SEARCH_STEP）和计算权重的步长（SMAA_DISABLE_DIAG_DETECTION）的差异。

如果想继续优化，可以增加SMAA的阈值，调低计算权重的步长，但可能会影响画面的表现效果损失。

由于SMAA需要三遍pass，所以就算继续优化shader指令，也不会有太大的性能提升。

2.

使用只进行一遍pass的FXAA，但对画质影响较大，看起来远处十分模糊

3.由于移动端ppi较高，有时可以选择关闭AA，画面也无明显差异

总结：

| 设备性能     | 方案                                                         |
| ------------ | ------------------------------------------------------------ |
| 高端移动设备 | 采用优化后的SMAA或FXAA，Unity默认URP下的TAA成熟后，根据性能与表现均衡选择采用 |
| 中端移动设备 | 采用FXAA方式或关闭AA渲染                                     |
| 低端移动设备 | 关闭AA渲染                                                   |

#### 最终修改结果

通过对比三种方案效果，由于画面差距不明显，最终选择关闭AA反走样

#### 最终优化结果

![image-20220531155042494](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220531155042494.png)

**UPR报告显示平均帧率由20.46提升为24.96，提高约4.5帧**

![image-20220531155148343](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220531155148343.png)

**纹理资源的内存占用也降低了20MB**

![image-20220531125114359](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220531125114359.png)

![image-20220531155412309](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220531155412309.png)

**帧数明显提升，表现效果无可明显察觉的差异**



### PostProcess后处理优化

#### PostProcess常见种类

| 色彩校正与增强                                               | 画面效果增强                                                 | 镜头效果                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Channel Mixer<br />Color Adjustment<br />Color Curves<br/>Lift,Gamma<br />and Gain<br />Shadows/Midtones/Highlights<br/>Tonemapping<br />White Balance<br />Split Toning | Bloom<br />Chromatic Aberration<br />Depth Of Field<br />Film Grain、<br/>Motion Blur | Lens Disortion<br />Panini Projection<br />Vignette<br />Lens Flare |
| 开销较低，但不要胡乱堆砌<br />尽量在原始美术资源上做调整     | 需要额外的RanderPass处理<br />采样开销比较高<br />整体开销较大<br />如需要开启，要使用LocalVolume在不同逻辑下开启<br />不要使用GlobalVolume一直开启 | shader计算量比较大介于前两者之间<br />使用LocalVolume在不同逻辑下开启 |

#### Unity URP下支持的后处理效果列表

| 效果列表                      | 描述               | 移动端性能开销              | 常用性 | 优化方式                                                     |
| ----------------------------- | ------------------ | --------------------------- | ------ | ------------------------------------------------------------ |
| Bloom                         | 泛光、镜头污垢     | 中高                        | 高     | 降采样、降低迭代次数、替换2遍模糊pass算法                    |
| Channel Mixer                 | 通道混合器         | 非常低                      | 低     | 几乎无，更常用HUE调整                                        |
| Chromatic Aberration          | 散色像差           | 中低（需要三遍采样）        | 低     | 几乎无                                                       |
| Color Adjustment              | 颜色调整           | 中                          | 高     | ColorGradingMode(HDR or LDR)                                 |
| Color Curves                  | 颜色曲线           | 低                          | 低     | 几乎无                                                       |
| Depth Of Field                | 景深               | 非常高                      | 中低   | 切换Gaussian与Bokeh的景深模式（一般过场动画使用）            |
| Film Grain                    | 胶片颗粒           | 中（需要采样多张Lookup贴图) | 低     | 几乎无                                                       |
| Lens Disortion                | 镜头失真           | 中低                        | 低     | 几乎无                                                       |
| Lift,Gamma,andGain            | 提升、伽马与增益   | 非常低                      | 低     | 几乎无                                                       |
| Motion Blur                   | 运动模糊           | 高（需要MotionVector)       | 中低   | 质量分级、Intensity强度.Clamp摄像机旋转产生的速度可以具有的最大长度（过场动画或局部逻辑开启） |
| Panini Projection             | Panini投影         | 中                          | 低     | 几乎无                                                       |
| Shadows、Midtones、Highlights | 阴影、中间调与高光 | 中低                        | 低     | 几乎无                                                       |
| Split Toning                  | 拆分着色           | 非常低                      | 低     | 几乎无                                                       |
| Tonemapping                   | 色调映射           | 中                          | 中     | 几乎无                                                       |
| Vignette                      | 渐晕               | 中                          | 低     | 调节Intensity与Smoothness                                    |
| White Balance                 | 白平衡             | 中低                        | 低     | 几乎无                                                       |
| Lens Flare                    | 镜头光晕           | 中高                        | 中     | Occlusion设置与光晕数量                                      |

#### 优化部分

![image-20220531160608224](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220531160608224.png)

为了资源效果表现，项目原有较多后处理效果

![image-20220531160755875](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220531160755875.png)

将效果可控的部分和性能占用较大的动态模糊部分Remove，正式项目中应修改原始美术资源达到删除部分的视觉效果，
本项目不方便修改美术资源，所以通过调节Color Adjustments的Post Exposure数值使Vignette删除前后画面效果差距减小

![image-20220531161248629](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220531161248629.png)

调整URP Asset中的后处理选项，将HDR改为LDR（如果为支持浮点精度运算的设备则可以开启HDR），并降低size，开启Fast sRGB

![image-20220531161326648](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220531161326648.png)

对于bloom部分的优化，选择降低第一遍pass的采样分辨率和最大downscale迭代次数

![image-20220531161926556](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220531161926556.png)

如需进一步优化Bloom的其他pass，可以参考Unreal官方的次时代手机渲染GDC演讲

[GDC 2014 (myqcloud.com)](https://cdn2-unrealengine-1251447533.file.myqcloud.com/Resources/files/GDC2014_Next_Generation_Mobile_Rendering-2033767592.pdf)

#### 最终优化结果

![image-20220531170010988](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220531170010988.png)

**UPR报告显示平均帧率由24.96提升为25.78，提高约0.82帧**

![image-20220531170120662](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220531170120662.png)

**纹理资源的内存占用也降低了5MB**

![image-20220531155412309](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220531155412309.png)

![image-20220531170231461](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220531170231461.png)

**画面色彩稍有区别，正式项目应当调整美术资源消除优化影响**



### Culling剔除

#### Unity中的剔除

| 剔除种类 |                                                              |
| -------- | ------------------------------------------------------------ |
| 像素剔除 | 摄像机平截头体剔除、Back-face Culling、Early-Z、Pre-Z Pass   |
| 网格剔除 | Layer Mask、可见距离剔除、Occlusion                          |
| 灯光剔除 | Tile-Based Deferred Rendering、Forward+（均依赖额外图形库和硬件架构） |
| 场景剔除 | Additive Scene                                               |

#### 用户拓展剔除

| 场景数据结构 | GPU Culling                   |
| ------------ | ----------------------------- |
| Octree       | Hi-Z Pass                     |
| BSP Tree     | Temporal Reprojection Culling |
| Portal       | Cluster                       |
| Voxelization | Tile-based Visible Buffer     |
| SDF          | ……                            |
| ……           | ……                            |

### Simplization简化

#### Unity中的简化

| Simplization                                      |
| ------------------------------------------------- |
| Project Settings -> Quality Settings              |
| 通过烘焙光照简化实时光照(移动光源较少)            |
| 通过BoundingBox或替代体碰撞代替Mesh碰撞           |
| 通过Local Volume代替Global Volume                 |
| RayCast代替SphereCast、CapsuleCasts等             |
| 纹理文字代替系统文字                              |
| Mesh LOD                                          |
| Shader LOD                                        |
| HLOD                                              |
| **通过Camer Override代替URP管线中的一些通用设置** |
| 各种OnDemand更新或分级设置接口                    |
| …………                                              |

#### 用户扩展简化

| 用户扩展简化              |
| ------------------------- |
| 场景简化数据结构          |
| 第三方LOD方案             |
| Mesh Impostor             |
| Animation LOD             |
| 骨骼LOD                   |
| 2D寻路代替Navigation Mesh |
| 扩展类似OnDemand接口      |

### Batching

#### 资源Batching

##### Mesh:

- Mesh.CombineMesh,合并静态网格对象(合并对象较大可能造成剔除不掉或overdraw的问题，内存也会增加一定开销)
- Submeshes->Single Mesh,合并材质与贴图，不同材质通过通道图标记

##### Texture:

- AtlasTexture,通过纹理坐标映射多张贴图
- TextureArray纹理数组

##### Shader变量与材质属性

- Material Property Block(Build In管线)
- Const buffer(SRP管线)



#### Draw call Batching

##### Static Batching

- StaticBatchingUtility.Combine（可运行时合批）

  **限制：**

1. 额外内存开销
2. 64000个顶点限制
3. 影响Culling剔除

##### Dynamic Batching

​		**限制：**

- 合批不超过900个顶点属性，（非900个顶点，例如有位置颜色法线UV坐标四个属性时，合批不超过900/4）
- 除了渲染阴影对象外，相同材质，不同材质实例也不能合并
- 具有光照贴图的游戏对象如果有附加渲染器参数时，如果需要动态合批这些对象
- 他们必须指向相同的光照贴图位置。
- 有多Shader Pass的游戏对象无法做动态合批
- 受多个光照影响的游戏对象，满足动态合批条件合批后，只会受一个光源的影响
- 延迟渲染下不支持动态合批
- CPU开销可能会增大，需要测试开启使用

#### GPU Instancing

- DrawMeshInstanced（有实例数量限制）
- DrawMeshInstancedIndirect
- DrawMeshInstancedProcedural

参考：[Unity - Scripting API: Graphics.DrawMeshInstancedIndirect (unity3d.com)](https://docs.unity3d.com/ScriptReference/Graphics.DrawMeshInstancedIndirect.html)

​		**限制：**

- 图形API版本要求
- 与SRPBatcher不兼容
- 不同绘制API的参数与绘制个数不同
- 渲染顶点数较少的网格时，效率可能会较差。（测试使用)

#### Set Pass call Batching

##### SRP Batching

##### Const Buffer

- UnityPerCamera
- UnityPerFrame
- UnityPerPass
- UnityPerDraw
- UnityPerDrawRare
- **UnityPerMaterial**

#### 优化顺序

资源Batching > SRP Batching = Static Batching > GPU Instancing > Dynamic Batching

#### 合批失败原因

1.可参考Unity官方Git仓库工程（版本较旧，不含URP，HDRP管线，SRP Batcher的失败原因）

[Unity-Technologies/BatchBreakingCause: This project demonstrates different cases when Unity has to break a batch while rendering. (github.com)](https://github.com/Unity-Technologies/BatchBreakingCause)

2.FrameDebuger中抓取的管线信息

![image-20220811163633955](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220811163633955.png)

#### 较全原因汇总

| 共42个                                                       |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| “An object is affected by multiple forward lights.”          | 此物体受到多个前向灯光的影                                   |
| “Objects have different materials.”                          | 此物体有不同的材质                                           |
| “An object is using a multi-pass shader.”                    | 此物体使用了多pass着色器                                     |
| “An object has odd negative scaling.”                        | 此物体Trasform的Scale使用了负数                              |
| “Either objects have different \”Receive Shadows\“ settings<br />or some objects are within the shadow distance, while some other objects are not.” | 此物体接收阴影的设置不同，或者物体有不同的的阴影距离设置     |
| “Objects are affected by different forward lights.”          | 此物体受到不同的前向灯光影响                                 |
| “Objects are on different lighting layers.”                  | 物体在不同的Lighting Layer上                                 |
| “Objects have different \”Cast Shadows\“ settings.”          | 此物体有不同的投影体设置                                     |
| “Objects either have different shadow caster shaders, or have different shader properties / keywords that affect the output of the shadow caster pass.” | 此物体有不同的投影着色器设置或者有不同的着色器属性或者关键字影响Shadow Caster Pass的输出 |
| “The shader explicitly disables batching with the \”DisableBatching\“ tag.” | 着色器中显式设置了DisableBatching的标记                      |
| “Objects have different MaterialPropertyBlock set.”          | 此物体有不同的MaterialPropertyBlock集合                      |
| “Non-instanced properties set for instanced shader.”         | Instanced的着色器有非instanced属性集                         |
| “Objects are lightmapped.”                                   | 物体使用了不同的LightMap或者虽然使用相同的LightMap但使用的UV不同 |
| “Objects are affected by different light probes.”            | 此物体受到不同的光照探针影响                                 |
| “Objects are shadowed by baked occlusions and have different occlusion factors.” | 此物体烘焙了遮挡，并且设置了不同的遮挡因子                   |
| “Objects are affected by different reflection probes.”       | 此物体受到不同的反射探针影响                                 |
| “Rendering different meshes or submeshes with GPU instancing.” | 使用GPU实例化渲染不同的网格或子网格                          |
| “Objects have different batching-static settings.”           | 此物体有不同的静态合批设置                                   |
| “Objects belong to different static batches.”                | 此物体归属不同的Static Batches                               |
| "Dynamic Batching is turned off in the Player Settings or is disabled temporarily in the current context to avoid z-fighting.” | 在Player Settings中关闭了动态合批，或者在当前的环境中为了避免深度冲突而临时关闭了合批 |
| “There are too many indices (more than 32k) in a dynamic batch.” | 动态合批中有太多的索引（大于32k）                            |
| “A mesh renderer has additional vertex streams. Dynamic batching doesn‘t support such mesh renderers.” | Mesh Renderer具有其他顶点流。动态批处理不支持此类网格渲染器。 |
| “A submesh we are trying to dynamic-batch has more than 300 vertices.” | 动态合批超过300个顶点                                        |
| “A submesh we are trying to dynamic-batch has more than 900 vertex attributes.” | 动态合批超过900个顶点属性                                    |
| “This is the first draw call of a new shadow cascade.”       | 新阴影级联的第一次绘制调用                                   |
| “The material doesn‘t have GPU instancing enabled.”          | 材质未启用GPU Instancing功能                                 |
| “Objects are rendered using different rendering functions. This can happen if the type of renderer is different (eg Mesh/Skinned Mesh) or when using different settings within the same renderer, such as Sprite Masking.” | 使用不同的渲染。如果渲染器的类型不同（例如网格/蒙皮网格），或者在同一渲染器中使用不同的设置（例如精灵遮罩），则可能会发生这种情况。 |
| “Objects have different batching keys. This is usually caused by using different vertex streams on Particle Systems, or by mixing Lines and Trails, or by mixing lit and unlit geometry.” | 此对象具有不同的Batching Keys。 这通常是由于在粒子系统上使用不同的顶点流，或混合线和轨迹，或混合Lit和Unlit的几何体造成的。" |
| “Mesh uses 32 bit index buffer.”                             | Mesh使用了32位的索引缓冲                                     |
| “Submesh has non-zero base vertex.”                          | 子网格对象有非0的基础顶点， submesh.BaseVertexLocation != 0  |
| “The previous instanced draw call has reached its maximum instance count.” | 先前的InstanceDrawCall已经达到了Instance的最大数量           |
| “Motion Vector rendering doesn‘t support batching.”          | Motion Vector的渲染不支持Batching                            |
| “When using late latching, children of an XR late latched GameObject do not use batching.” | 使用late latching时，XR late latched GameObject的子级不能合批 |
| “Objects have different bounds and bounds instancing is disabled.” | 对象具有不同的包裹体，那么包裹体实例化被禁用                 |
| “SRP: Node have different shaders.”                          | 节点具有不同的着色器                                         |
| “SRP: Node use multi-pass shader”                            | 节点使用了多Pass着色器                                       |
| “SRP: Node use different shader keywords”                    | 节点使用了不同的着色器关键字                                 |
| “SRP: End of the batch flush”                                | Batch Flush结束                                              |
| “SRP: Node is not compatible with SRP batcher”               | 节点与SRP Batcher不兼容                                      |
| “SRP: Node material requires device state change”            | 节点材质需要改变渲染设备状态                                 |
| “SRP: First call from ScriptableRenderLoopJob”               | ScriptableRenderLoopJob第一次调用                            |
| “SRP: This material has custom buffer override”              | 材质有自定义重写的Buffer                                     |



### 项目优化分析

![image-20220819143843886](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220819143843886.png)

在Xcode frame capture中performance的timeline中，顶点着色器阶段Deferred Shading、Shadows Draw、G buffer目前耗时占比较大，其中Shadows Draw的耗时最高。在片元着色器阶段，结果也相似。

可以看出目前的渲染批次数过大，顶点数过大，很有可能是场景复杂度过高所导致的，所以需要在不影响表现的情况下做充分的简化和剔除，最后有需要再进行合批的优化。

![image-20220819144610080](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220819144610080.png)

项目中当前视角下，渲染批次有5400个左右，顶点数1.9M，三角形面数2.8M，投影体2700个左右

![image-20220816160517122](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220816160517122.png)

通过编辑器下的线框模式，可以大致获取场景中的顶点三角形分布，可以看到大面积的植被、房屋、树木是顶点三角形分布的重点，且远处的山峰也是真实模型渲染的，有较高的优化优先度。

![image-20220816160649384](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220816160649384.png)

在SRP渲染管线下，Randering Debugger的Overdraw下可以看到整个场景的Overdraw信息，可以看出场景有较多的Overdraw，这部分需要使用剔除进行优化。

![image-20220816161453211](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220816161453211.png)

通过Randering Debugger的阴影调试图，可以看到项目中的级联阴影可视视距较远，这也是投影体有2700多个的原因

![image-20220816161535747](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220816161535747.png)

从上方观察级联阴影的范围也可以发现这点。



### Simplization优化

#### 远景简化

![image-20220819145745787](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220819145745787.png)

- 先将场景中除远景和光源以外的部分隐藏，并在场景中添加一个反射探针烘焙CubeMap，选择探针Type为Baked，如果没有Bake按钮，则需要在编辑器菜单中的Windows->Randerring->Lightning选项中的Scene标签中，将Auto Generate复选框取消勾选。
- 接下来设置合适的裁剪距离来裁剪反射的平面，并按需要设置生成贴图的分辨率，如不需要HDR则可以关闭HDR选项，这样烘焙出的PNG格式的非HDRCubeMap就会更加小。
- 创建新材质选择shader为Skybox->CubeMap，并在贴图中选择刚才反射探针生成的Cubemap
- 替换Windows->Randerring->Lightning->Environment标签中的Skybox Material

![image-20220817105644791](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220817105644791.png)

经过远景简化后，同位置三角形数量从**1.9M**降低到**1.7M**，顶点部分数量由2.9M降低到**2.5M**



#### 中景简化

![image-20220819141609321](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220819141609321.png)



- 项目中的模型都已经有LODGroup，且已经有美术制作的LOD模型，所以不需要再生成LOD模型。

- 但是原始工程项目中的LOD设置并不合理，如只会出现在室内场景的物体的LOD裁剪层级和简化模型层级级别距离设置很远，无法做到合理的简化与剔除，Overdraw过多造成绘制浪费。

- 项目中的场景建筑是由多个模型块拼成的，但第一层级的LOD距离设置也过远，植被也存在种问题。

- 这些问题造成项目有5400左右的渲染批次，2.8M顶点，对象的投影体个数有2700个左右，导致阴影绘制阶段的压力。

  

![image-20220819141648125](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220819141648125.png)



- 优化过程对项目中部分场景中不必要的LOD级别上的模型不显示投影，如抽屉，桌子上的物体等，只在LOD0层级原始模型下才显示投影。

- 还有部分静态光源下的对象，已经在投影体下了，没必要再进行二次绘制，可以直接关闭投影，接触面的暗部可以由SSAO进行绘制。

  

![image-20220819141752774](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220819141752774.png)



经过LOD层级距离调整，投影体调整和可视距离调整，场景中的渲染批次由**5400**左右降低至**2400**左右，三角形由**1.9M**降低到**1.1**M，顶点数由**2.8M**降低到**1.7M**，投影体个数由**2700**降低到**1200**。



**注意：**

在Project Settings的Quality中，有各平台的 **LOD Bias** 设置，该设置会导致项目原有LOD层级距离出现偏差，LOD Bias 的数值设置大于1倾向于更多细节，小于1则相反。

![image-20220822153433684](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220822153433684.png)



- 此外，对于中远景的大物体还有  Imposters 的解决方案。
- 目前主流的替代体方案为八面体或者半球烘焙的替代体，主要是基于GPU Gems中的True  Imposters与Ryan Brucks的Octahedral Impostors。
- 两种算法的地址：

[	Chapter 21. True Impostors | NVIDIA Developer](https://developer.nvidia.com/gpugems/gpugems3/part-iv-image-effects/chapter-21-true-impostors)

[	Octahedral Impostors (shaderbits.com)](https://shaderbits.com/blog/octahedral-impostors/)

- 如需要深度大场景优化，可以使用此方案代替HLOD方案。

  

### Culling优化

![image-20220822154413045](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220822154413045.png)

- 原始项目中，作者已经制作了剔除部分的内容，所以关于Culling的优化是在原有剔除上做补充，应该不会对项目的性能有太大的影响。
- 场景遮挡剔除和入口剔除两种方法都在Unity中原生提供，主要是为了防止完全被遮挡的物体重复渲染浪费资源。
- 在该工程的延时渲染下，不透明对象已经做了像素的深度检测，会剔除被遮挡的像素进入渲染管线，但延时渲染下的投影阶段和半透明物体的绘制仍然是前向渲染的，因此遮挡剔除和窗口两种CPU端的剔除方法主要就是为了优化前向渲染时不做Pre Z Pass，GPU性能瓶颈的情况。
- 遮挡剔除只适用于静态遮挡体，但对于门窗这类的动态遮挡体，则需要Portal Culling脚本进行控制，使其随门窗开关显示或剔除。

![image-20220822155755914](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220822155755914.png)



- 优化部分，将项目中所有的门添加 Portal Culling 脚本控制屋内物体的显示与剔除，未避免视觉穿帮，需要在门开启开门动画时显示，在门完全关闭后剔除。
- 光影剔除部分，因为将远景山峰的Mesh替换为CubeMap，远景山峰的光照就可以关闭了。
- 项目的地形相对比较平坦，没有需要投影体的必要，只需要接收其他物体的投影，不需要自投影。

![image-20220822182134660](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220822182134660.png)

SRP下的LightLayer功能可以用来进行光影剔除，但由于项目中并没有开启LightLayer功能，场景中的物体也没有室内室外LayerMask，直接修改会导致大量资源发生变动，所以这里使用脚本进行控制。

![image-20220822182340147](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220822182340147.png)

通过在场景所有点光源上挂载LightCulling脚本实现限制场景内的动态光源数量，并通过设置灯光的剔除距离和可以产生投影的距离控制光源影响物体个数。

详细参考：[Light Layers | High Definition RP | 7.1.8 (unity3d.com)](https://docs.unity3d.com/Packages/com.unity.render-pipelines.high-definition@7.1/manual/Light-Layers.html)

![image-20220822182547406](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220822182547406.png)

降低近处场景的Shadow Resolution和Shadow Atlas Resolution来在不怎么影响整体光影的情况下降低场景的性能开销

![image-20220822182906222](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220822182906222.png)

优化后编辑器模式下，场景中的渲染批次由**2400**左右降低至**750**左右，三角形由**1.1M**降低到**0.6**M，顶点数由**1.7M**降低到**1.0M**，投影体个数由**1200**降低到**727**。

![image-20220822183133867](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220822183133867.png)

在Xcode frame capture中，在Ipad pro2017下渲染耗时已经由最初的**80ms**优化至接近**20ms**。

### Batching优化

由于项目使用第三方shader插件进行shader代码生成，shader类型若修改为支持Native RenderPass的Lit类型，会导致所有资源的引用断开，这部分优化应在项目初始时对美术规范进行限制，使资源能够支持Native RenderPass，才可以进行动态合批和阴影合批等Unity所支持的现代图形API的功能

Randerpass详细参考：

[Universal Renderer | Universal RP | 13.1.8 (unity3d.com)](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@13.1/manual/urp-universal-renderer.html)

[Vulkan Multipass Mobile Deferred Done Right (khronos.org)](https://www.khronos.org/assets/uploads/developers/library/2017-gdc/GDC_Vulkan-on-Mobile_Vulkan-Multipass-ARM_Mar17.pdf)

[Unity - Scripting API: Rendering.ScriptableRenderContext.BeginRenderPass (unity3d.com)](https://docs.unity3d.com/ScriptReference/Rendering.ScriptableRenderContext.BeginRenderPass.html)



### 内存优化

| Unity 中的内存 |                                                              |
| -------------- | ------------------------------------------------------------ |
| 托管内存       | 使用托管堆或垃圾收集器自动分配和管理的内存，也包括脚本堆栈和虚拟机内存 |
| C#非托管内存   | 可以在C#下与Unity Collection:名字空间和包结合使用，不使用垃圾收集器管理的内存部分 |
| Native内存     | Unity 用于运行引擎的C++内存                                  |

- 托管内存使用方便，但内存的分配方式和释放不可预期，Unity使用的是 **Boehm** 保守回收器，内存的管理与分配上比较保守，因此一些使用不当会造成较多内存的浪费，也可能造成性能的问题。
- C#非托管内存一般在C#下使用 Unity Collections 命名空间与包结合使用，也是将来使用dos程序加速准备的数据结构访问的内存。

<img src="C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220823170131323.png" alt="image-20220823170131323" style="zoom: 67%;" />

详细参考：[Unity Collections package | Collections | 1.4.0 (unity3d.com)](https://docs.unity3d.com/Packages/com.unity.collections@1.4/manual/index.html)

| Unity外的内存 | 描述                                                  |
| ------------- | ----------------------------------------------------- |
| 显存          | 渲染资源占用的部分，若是移动平台该显存与内存共享      |
| 应用程序框架  | 不能通过Unity提供的工具获取                           |
| 第三方库      | 不能通过Unity提供的工具获取                           |
| 系统内存      | 不能通过Unity提供的工具获取，操作系统为应用分配的内存 |

### Unity中的内存工具

#### Unity Profiler

![image-20220823171804098](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220823171804098.png)

- Unity Profiler下的Memory标签，列举了Unity当下内存使用的追踪状态，包括各类内存的分配与使用情况，以及当前Unity下分配的对象与资源占用的内存情况
- 注：Unity2021以后的版本不再提供对象抓取快照功能，而是使用新的Memory Profiler直接抓取内存整体快照。

![image-20220823171905671](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220823171905671.png)

- Profiler Memory标签下默认提供的指标并不全面，如需要更详细的信息需添加自定义Profiler Modules来加入更多的Memory Counter指标，也可以进一步扩展Profiler Modules，通过对应的系统接口提供更详细的内存指标加载到Profiler中，具体可查看Unity新工具下的自定义Profiler相关的内容。

#### Memory Profiler

在Windows下的Memory Profiler可以抓取一帧的内存数据，并可以进行多帧数据的对比Unity资源与对象的差异

![image-20220823172123809](C:\Users\ianxhtian\AppData\Roaming\Typora\typora-user-images\image-20220823172123809.png)

在Summary标签下可以查看内存数据的概括和内存分配的树图

在Objects and Allocations标签下可以查看具体对象内存分配情况快照，并可以通过链接直接找到原工程中对应的对象和资源

在
