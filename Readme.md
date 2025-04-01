> This is a fork from [Const-me/Whisper](https://github.com/Const-me/Whisper), with some content translated into Chinese. In the text below, "我("I" in Chinese)" refers to the original author [Const-me](https://github.com/Const-me), not the owner of this forked project you're currently viewing.<br/><br/>
这是 [Const-me/Whisper](https://github.com/Const-me/Whisper) 的一个Fork，并且对部分内容进行了中文翻译，下文中的“我”均代指原作者 [Const-me](https://github.com/Const-me)，而不是现在所看到的这个fork项目的拥有者。
---
这个项目是 [whisper.cpp](https://github.com/ggerganov/whisper.cpp) 实现的一个 Windows 移植版，而 [whisper.cpp](https://github.com/ggerganov/whisper.cpp) 又是 [OpenAI's Whisper](https://github.com/openai/whisper) 自动语音识别 (ASR) 模型的 C++ 移植版。

# 快速开始指南

从本仓库的 Releases 部分下载 WhisperDesktop.zip，解压 ZIP 文件，然后运行 WhisperDesktop.exe。

在第一个界面将提示您下载一个模型。<br/>
我推荐使用 `ggml-medium.bin`（1.42GB 大小），因为我大部分时间都是用这个模型测试软件的。<br/>
![Load Model Screen](gui-load-model.png)

在下一个界面可以转录音频文件。<br/>
![Transcribe Screen](gui-transcribe.png)

还有一个界面允许从麦克风捕获并转录或翻译实时音频。<br/>
![Capture Screen](gui-capture.png)

# 特点

* 基于 DirectCompute 的供应商无关 GPGPU；该技术的另一个名称是“Direct3D 11 中的计算着色器”

* 纯 C++ 实现，除了基本的 OS 组件外，没有运行时依赖

* 比 OpenAI 的实现快得多。<br/>
在我的台式计算机上，使用 GeForce [1080Ti](https://en.wikipedia.org/wiki/GeForce_10_series#GeForce_10_(10xx)_series_for_desktops) GPU 和中等模型，一段 [3 分 24 秒的语音](https://upload.wikimedia.org/wikipedia/commons/1/1f/George_W_Bush_Columbia_FINAL.ogg)在 PyTorch 和 CUDA 上转录需要 45 秒，而使用我的实现和 DirectCompute 只需要 19 秒。<br/>
有趣的是：运行时依赖是 9.63 GB，而 `Whisper.dll` 只有 431 KB。

* 混合 F16 / F32 精度：Windows 从 D3D 版本 10.0 [需要支持](https://learn.microsoft.com/en-us/windows/win32/direct3ddxgi/format-support-for-direct3d-feature-level-10-0-hardware#dxgi_format_r16_floatfcs-54) `R16_FLOAT` 缓冲区

* 内置性能分析器，可以测量单个计算着色器的执行时间

* 低内存使用

* 用于音频处理的 Media Foundation，支持大多数音频和视频格式（值得注意的是不支持 Ogg Vorbis），以及大多数在 Windows 上工作的音频捕获设备（除了某些仅实现 [ASIO](https://en.wikipedia.org/wiki/Audio_Stream_Input/Output) API 的专业设备）。

* 用于音频捕获的语音活动检测。<br/>
该实现基于 Mohammad Moattar 和 Mahdi Homayoonpoor 于 2009 年发表的文章“[A simple but efficient real-time voice activity detection algorithm](https://www.researchgate.net/publication/255667085_A_simple_but_efficient_real-time_voice_activity_detection_algorithm)(一种简单但高效的实时语音活动检测算法)”。

* 易于使用的 COM 风格 API。有符合语言习惯的 C# 包装器，可[在 nuget 上获取](https://www.nuget.org/packages/WhisperNet/)。<br/>
1.10 版本[引入](https://github.com/Const-me/Whisper/tree/master/WhisperPS)了对 PowerShell 5.1 的脚本支持，这是较旧的“Windows PowerShell”版本，默认预安装在 Windows 上。

* 可用的预构建二进制文件

唯一支持的平台是 64 位 Windows。<br/>
应该可以在 Windows 8.1 或更新版本上运行，但我只在 Windows 10 上进行了测试。<br/>
该库需要支持 Direct3D 11.0 的 GPU，而在 2023 年，这基本上意味着“任何硬件 GPU”。<br/>
最新一款不支持 D3D 11.0 的 GPU 是来自 2011 年的 Intel [Sandy Bridge](https://en.wikipedia.org/wiki/Sandy_Bridge)。<br/><br/>
在 CPU 方面，该库需要支持 [AVX1](https://en.wikipedia.org/wiki/Advanced_Vector_Extensions) 和 [F16C](https://en.wikipedia.org/wiki/F16C)。

# 开发者指南

## 构建说明

1. 克隆此仓库

2. 在 Visual Studio 2022 中打开 `WhisperCpp.sln`。我使用的是免费的社区版，版本 17.4.4。

3. 切换到 `Release` 配置

4. 构建并运行 `CompressShaders` C# 项目，该项目位于解决方案的 `Tools` 子文件夹中。<br/>
要运行该项目，在 Visual Studio 中右键单击，选择“Set as startup project（设置为启动项目）”，然后在 Visual Studio 主菜单中选择“Debug / Start Without Debugging（调试 / 开始但不调试）”。
成功完成时，你应该会看到一个控制台窗口，显示如下类似的一行内容：<br/>
`Compressed 46 compute shaders, 123.5 kb -> 18.0 kb`

5. 构建 `Whisper` 项目以获取原生 DLL，或者构建 `WhisperNet` 来获取 C# 封装库和 NuGet 包，亦或是构建示例项目。

## 其他说明

如果您要在使用 Visual C++ 2022 或更新版本构建的软件中使用该库，您可能需要以 `.msm` 合并模块或者 [vc_redist.x64.exe](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist?view=msvc-170) 二进制文件的形式重新分发 Visual C++ 运行时 DLL。<br/>
如果要这样做，右键点击 `Whisper` 项目，选择“Properties（属性）”，然后转到 C/C++ > Code Generation（代码生成），
将“Runtime Library（运行库）”设置从 `Multi-threaded (/MT)` 切换为 `Multi-threaded DLL (/MD)`，
然后重新构建：这会使二进制文件变得更小。

该库包括 [RenderDoc](https://renderdoc.org/) GPU 调试器集成。<br/>
从 RenderDoc 启动您的程序时，按住 F12 键以捕获计算调用。<br/>
如果您要调试 HLSL 着色器，请使用 DLL 的调试版本，它包括着色器的调试版本，您将在调试器中获得更好的用户体验。

该仓库包含了许多仅用于开发的代码：
一些备用的模型实现、兼容 FP64 的计算着色器版本、调试跟踪工具以及用于比较跟踪结果的工具等。<br/>
这些内容通过预处理宏或 constexpr 标志被禁用，我希望保留这些内容是可以的。

## 性能说明

我家里可用的 GPU 选择有限。<br/>
具体来说，我已经针对以下设备进行了优化：
- nVidia 1080Ti
- Ryzen 7 5700G 内置的 Radeon Vega 8
- Ryzen 5 5600U 内置的 Radeon Vega 7

这里有一篇[总结](https://github.com/Const-me/Whisper/blob/master/SampleClips/summary.tsv)。

nVidia 在运行大型模型时的相对速度为 5.8，运行中型模型时为 10.6。<br/>
AMD Ryzen 5 5600U APU 在运行中型模型时的相对速度约为 2.2。虽然表现不是很优秀，但仍然远快于实时处理。

我还在 [nVidia 1650](https://en.wikipedia.org/wiki/GeForce_16_series#Desktop) 上进行了测试，其速度比 1080Ti 慢，但表现相当不错，远快于实时处理。<br/>
另外，我在 Core i7-3612QM 内置的 Intel HD Graphics 4000 上也进行了测试，中型模型的相对速度为 0.14，小型模型为 0.44。虽然明显慢于实时处理，但让我感到欣慰的是，我的软件甚至能够在这款于2012年[发布](https://ark.intel.com/products/64901)的移动集显上正常运行。

我不确定软件在独立 AMD GPU 或集成 Intel GPU 上的性能是否理想，因为我没有针对它们进行特定优化。<br/>
理想情况下，它们可能需要对几个计算开销最大的着色器作些许调整，比如 `mulMatTiled.hlsl` 和 `mulMatByRowTiled.hlsl`。<br/>
此外，可能还需要其他调整，例如 `Whisper/D3D/device.h` 头文件中的 `useReshapedMatMul()` 值。

我不知道该如何准确测量性能，但我感觉瓶颈可能是在内存，而不是计算性能。<br/>
Hacker News 上有人曾经在使用 GDDR6 内存的版本的 [3060Ti](https://en.wikipedia.org/wiki/GeForce_30_series#Desktop) 上[测试过](https://news.ycombinator.com/item?id=34408429) 。<br/>
相较于 1080Ti，这款 GPU 的 FP32 浮点运算性能是 1.3 倍，但显存带宽只有 0.92 倍。<br/>
在他们的测试中，这个应用在 3060Ti 上运行时大约慢了 10%。

## 进一步优化

我只花了几天时间优化这些着色器的性能。<br/>
可能还有很大的提升空间，这里有一些想法。

* 一些较新的 GPU，例如 Radeon Vega 或 nVidia 1650，在 FP16 性能上比 FP32 更高，但我的计算着色器目前仅使用 FP32 数据类型。<br/>
[Half The Precision, Twice The Fun](https://therealmjp.github.io/posts/shader-fp16/)

* 在当前版本中，FP16 张量使用着色器资源视图来对加载的值进行升精度（upcast），使用无序访问视图来对存储的值降精度（downcast）。<br/>
或许可以考虑切换到[字节地址缓冲区(byte address buffers)](https://learn.microsoft.com/en-us/windows/win32/direct3d11/direct3d-11-advanced-stages-cs-resources#byte-address-buffer)，加载/存储完整的 4 字节值，然后在 HLSL 中通过 `f16tof32` / `f32tof16` 内置函数进行升精度或降精度转换。

* 在当前版本中，所有的着色器都离线编译，`Whisper.dll` 中包含了 DXBC 字节。<br/>
HLSL 编译器 `D3DCompiler_47.dll` 是操作系统组件，编译速度非常快。<br/>
对于计算成本较高的计算着色器，或许值得考虑直接分发 HLSL 源代码而非 DXBC，并在启动时基于环境相关的宏（[macros](https://learn.microsoft.com/en-us/windows/win32/api/d3dcommon/ns-d3dcommon-d3d_shader_macro)）[编译](https://learn.microsoft.com/en-us/windows/win32/api/d3dcompiler/nf-d3dcompiler-d3dcompile)。

* 将整个项目从 D3D11 升级到 D3D12 可能是个好主意。<br/>
较新的 API 虽然更难使用，但包含了一些 D3D11 无法使用的潜在有用功能，例如：
[波操作内置函数（wave intrinsics）](https://github.com/Microsoft/DirectXShaderCompiler/wiki/Wave-Intrinsics) 和
[显式 FP16 支持（explicit FP16）](https://github.com/microsoft/DirectXShaderCompiler/wiki/16-Bit-Scalar-Types)。

## 缺少的功能

自动语言检测尚未实现。<br/>

在当前版本中，实时音频捕获的延迟较高。<br/>
具体来说，根据语音检测的情况，延迟大约在 5-10 秒之间。<br/>
至少在我的测试中，当我提供过短的音频片段时，模型表现不佳。<br/>
我暂时增加了延迟来解决问题，但从理想的用户体验来看，这需要一个更好的修复方法。


# 最后的话

从我的角度来看，这是一个无偿的业余爱好项目，我在 2022-23 年的冬季假期完成的。<br/>
代码可能存在一些漏洞。<br/>
该软件是按“原样”提供的，不附带任何形式的保证。<br/>

感谢 [Georgi Gerganov](https://github.com/ggerganov) 提供的 [whisper.cpp](https://github.com/ggerganov/whisper.cpp) 实现，
以及 GGML 二进制格式的模型。<br/>
我不编写 Python 程序，也对机器学习生态系统一无所知。<br/>
如果没有一个优秀的 C++ 参考实现来对比测试我的版本，我甚至不会开始这个项目。<br/>

whisper.cpp 项目中有一个示例，[使用](https://github.com/ggerganov/whisper.cpp/blob/master/examples/talk/gpt-2.cpp) 相同的 GGML 实现来运行另一个 OpenAI 的模型，[GPT-2](https://en.wikipedia.org/wiki/GPT-2)。<br/>
借助本项目中已实现的计算着色器和相关基础设施，支持该机器学习模型应该不难。

如果您觉得这个有用，我将非常感激，您可以考虑向“Come Back Alive”基金会捐款（请前往[原项目处](https://github.com/Const-me/Whisper?tab=readme-ov-file#final-words)进行捐赠，如果需要的话）。

---
重申：上述的“我”均指原作者 [Const-me](https://github.com/Const-me) 而非此项目的拥有者！

翻译使用翻译工具（带大语言模型 (Github Models 中的 [OpenAI GPT-4o](https://github.com/marketplace/models/azure-openai/gpt-4o) & [智谱清言](https://chatglm.cn/)中的智能体) 辅助），辅以人工校准及润色 (by [@hoiyuyiu](https://github.com/hoiyuyiu/))。<br/>
Translation uses translation tools (with LLM ([OpenAI GPT-4o · GitHub Models](https://github.com/marketplace/models/azure-openai/gpt-4o) & [ChatGLM](https://chatglm.cn/) Agent) assistance), supplemented by manual calibration and polishing (by [@hoiyuyiu](https://github.com/hoiyuyiu/)).
