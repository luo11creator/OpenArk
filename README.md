# OpenArk 机器人智控系统技术文档

**项目名称**：OpenArk灵枢：基于 DAYU200 与 Wheeltec ROS 机器人的原生 AI 协同控制系统
**文档版本**：V1.1
**硬件环境**：

- 控制端：OpenHarmony DAYU200 (RK3568，OpenHarmony ≥ 5.0 Release)
- 机器人端：ROS 小车 (Ubuntu 18.04 + ROS Melodic + ARM64)   || 水下机器人（由于水下机器人不好演示效果，这里采用ROS小车完成项目演示流程，水下机器人功能同理）
- 传感器：Intel RealSense D435i 深度相机 / 水下光学摄像机

------

## 0. 赛题符合性与合规性声明

### 0.1 与赛项内容方向的符合性（对应条款 (1)）

本作品围绕 OpenHarmony 原生 AI 与机器人控制展开，针对**国家海洋强国战略**中的水下探测痛点，实现：

- **基于 OpenHarmony 的端侧 AI 推理**：使用 MindSpore Lite 实现水下图像增强与目标检测的一体化处理；
- **与 ROS 机器人的实时协同控制**：构建水下/潮湿环境下的视觉感知闭环；
- **面向实际场景的应用**：解决水下光学图像失真、模糊导致的探测难题，服务于水下巡检、生物监测等场景。

作品名称“OpenArk 灵枢”体现了 OpenHarmony 生态与智能中枢（灵枢）的结合，完全符合赛项要求。

<img width="1266" height="744" alt="image" src="https://github.com/user-attachments/assets/783be07a-d668-4b22-9281-644d76dd0dd3" />


### 0.2 原创性与知识产权声明（对应条款 (2)）

1. **系统架构原创**：跨端通信协议、NAPI 高性能封装、UI 交互及闭环控制逻辑均由团队独立设计实现。

2. 核心算法说明

   - 采用老师的 **LUIEO（Low-light Underwater Image Enhancement and Object detection）** 模型。

  <img width="1213" height="720" alt="image" src="https://github.com/user-attachments/assets/f0a66d78-363c-4e93-8747-5564017afb8e" />


   - 该模型基于物理成像模型与深度学习的融合，已获指导老师明确授权用于本次比赛。
   - 团队在此基础上完成了**模型轻量化裁剪、MindSpore 迁移适配及端侧 NPU/CPU 加速优化**。

3. 本项目承诺不侵犯第三方知识产权，所有引用均已标注。



### 0.3 第三方开源组件说明（对应条款 (3)）

本项目遵循“允许使用第三方开源组件，但需遵从协议并在文档中说明”的原则。
下表列出主要第三方组件（示例，需你根据实际补全/修正版本号与协议类型）：

| 组件名称                   | 作用/用途                                       | 代码占比（粗略）  | 协议类型                           |
| -------------------------- | ----------------------------------------------- | ----------------- | ---------------------------------- |
| OpenHarmony 5.0 Release    | 系统运行环境、UI 框架、网络与蓝牙接口           | 不计入参赛代码    | Apache 2.0                         |
| MindSpore Lite (NDK)       | 端侧 AI 推理引擎，加载 `.ms` 模型推理           | ~10%              | Apache 2.0                         |
| ROS Melodic                | 机器人操作系统中间件，话题通信、节点管理        | 不计入参赛代码    | BSD                                |
| librealsense2              | RealSense D435i 驱动与 SDK                      | <5%（接口级调用） | Apache 2.0                         |
| realsense-ros              | 将 RealSense 相机数据发布为 ROS 话题            | <5%               | Apache 2.0                         |
| web_video_server           | 将 ROS 图像话题转为 HTTP MJPEG 流               | <5%               | BSD / Apache 2.0（视具体项目而定） |
| OpenCV                     | C++ 端图像预处理（resize、normalize、色彩变换） | ~5%               | Apache 2.0 / BSD                   |
| 老师的模型（PyTorch 源码） | 视觉模型结构与预训练权重，作为算法基础          | ~20–30%           | 内部授权使用（需老师确认）         |

**说明方式**（在正式提交文档中需写明）：

- 每个组件的来源（GitHub 链接或官网）、功能用途；
- 在本项目中大约占据的代码/功能比例；
- 遵循的开源协议（并在仓库中附上对应 LICENSE 或链接）；
- 老师模型若未开源，则注明为“经授权的内部组件”，不对外再分发源码，仅提交与本作品必要的接口包装代码。

### 0.4 功能实现与文档一致性（对应条款 (4)）

- 文档中所有功能，将根据实际完成情况标注为：
  - **已实现功能**：在演示中可完整展示，代码可运行；
  - **部分实现功能**：实现主要流程但尚不完善（如缺少 UI 细节或异常处理），会在文档中说明；
  - **规划中/未实现功能**：在设计图中出现但当前版本未实现，文档中注明占比与重要程度。
- 若有文档中设计而未完成的功能（如后期扩展的人脸识别登录），会在“功能规划与未实现部分”章节单独列出，避免与评审产生偏差。

### 0.5 代码注释与规范性（对应条款 (5)）

- 核心模块要求做到：
  - 类和模块级注释：说明设计意图与职责；
  - 函数级注释：说明输入参数、输出与关键算法逻辑；
  - 关键算法（模型预处理、NMS、控制闭环等）均有简明的中英文注释。
- 采用统一的命名规范与目录结构（见「7. 源代码结构与复现说明」章节），提高可读性与可维护性。

------




## 1. 项目背景与需求分析

### 1.1 行业背景与痛点

21世纪是海洋的世纪，海洋是巨大的战略资源宝库。随着国家“海洋强国”战略的推进，水下目标检测技术的重要性日益凸显。

- **成像环境复杂**：水下图像成像是一个复杂的物理过程，受光吸收和散射的严重影响，导致图像普遍存在**色偏、低对比度、模糊**等问题。
- 现有技术局限
  - **声纳探测**：虽然探测距离远，但成像分辨率低，非专业人士难以标注，无法满足精细化作业需求。
  - **传统光学探测**：直接使用退化图像进行检测，精度极低；若采用“先增强后检测”的串行方案，增强模块无法获得检测任务的反馈，导致计算资源浪费且特征丢失。

### 1.2 项目解决方案

针对上述问题，本项目提出**基于 OpenHarmony 的水下图像增强与目标检测一体化方案**：

1. **物理约束融合**：建立精细的水下成像物理公式，为深度学习网络提供先验信息和物理约束，实现无配对数据下的自监督训练。
2. **端侧实时感知**：利用 OpenHarmony 的软硬协同能力，将高算力需求的增强与检测任务部署在边缘端（DAYU200），实现低延时控制。

<img width="1223" height="663" alt="image" src="https://github.com/user-attachments/assets/c051c3cc-4b9a-4882-9c38-a9935fa682cf" />




------



## 2. 实施路线图 (Roadmap)

### 阶段一：基础控制链路

- **目标**：建立 OpenHarmony 与 ROS 的蓝牙通信。
- **实现**：基于 Socket/HTTP 协议和蓝牙指令，实现运动指令下发操控ROS小车| 水下机器人移动 通过 BLE 发送 Hex 指令（需修正特征值 UUID 问题）。

<img width="864" height="995" alt="image" src="https://github.com/user-attachments/assets/d4e4da4c-770f-4848-954b-d8ba8bd4a597" />


### 阶段二：视频流回传与解码

- **目标**：解决 ROS 视频流在 OpenHarmony 端的实时显示，DAYU200 实时显示 D435i 画面并用于 AI 推理。。
- **技术点**：实现 MJPEG 协议的手动解析，直接获取 `ArrayBuffer` 供 AI 模块使用，避免 Base64 转换的性能损耗。

- **机器人端方案**：运行 `web_video_server` 或 `mjpg-streamer`，将 ROS 话题转为 HTTP MJPEG 流。

- DAYU200 端方案

  - **Socket 手动拉流**：建立 TCP 连接，发送 HTTP GET 请求。
  - **协议解析**：按字节流寻找 `FF D8` (头) 和 `FF D9` (尾)，截取 `ArrayBuffer`。
  - **闭环**：截取的 Buffer 直接喂给 AI 模型接口 `detectImageBuffer`。

<img width="1263" height="712" alt="image" src="https://github.com/user-attachments/assets/873f5ed9-517a-4d88-a698-0cb97d3c9c13" />




<img width="1265" height="712" alt="image" src="https://github.com/user-attachments/assets/a50b6c8b-05e7-4ad0-8999-9fdb3a034573" />


### 阶段三：LUIEO 模型端侧部署 (核心技术壁垒)

- **目标**：将 PyTorch 模型迁移至 OpenHarmony 并通过 Native C++ 加速。

- 流程

  1. **模型转换**：PyTorch -> ONNX (Opset 13) -> MindSpore Lite (.ms)。

  2. Native 封装

     - **预处理下沉**：C++ 实现 Resize、归一化（`tensorData[i] = srcPtr[srcIdx] / 255.0f`），效率提升显著。
     - **推理下沉**：调用 MindSpore Lite C++ API，减少 NAPI 层内存拷贝。
     - **后处理下沉**：在 C++ 层完成 NMS（非极大值抑制）与增强图像还原。

  3. 模型转化指令

     ```
     export LD_LIBRARY_PATH=../lib:$LD_LIBRARY_PATH
     ./converter_lite --fmk=ONNX --modelFile=LUIEO_all.onnx --outputFile=LUIEO_all --inputShape="images:1,3,256,256"
     ```

<img width="1266" height="792" alt="image" src="https://github.com/user-attachments/assets/4fa66396-3bd4-43e6-9bad-cd9cb1832589" />




### 阶段四：功能扩展

- 人脸识别
- ai小助手

------



## 3. 核心算法原理：LUIEO 一体化网络

### 3.1 总体网络架构

本项目采用**高质量水下光学图像生成模型**与**增强检测一体化模型**相结合的技术路线，网络结构具有良好的可解释性和鲁棒性。

- 编码器-解码器架构 (Encoder-Decoder)

  - **编码器**：从原始退化图像中提取多尺度特征 (E0*E*0 至 E5*E*5)。
  - **解码器**：通过上采样生成特征 (P1*P*1 至 P4*P*4)，并融合原始图像的重要特征。
  - **多分支输出**：解码器末端分为三个分支，分别输出**清晰图像 (Clean Image)**、**背景光 (Background)** 和 **透射率图 (Transmission Map)**。

- 物理约束机制

  - 引入水下成像物理公式（Jaffe-McGlamery 模型），利用输出的清晰图像、背景光和透射率反向合成退化图像，与输入图像计算重构损失。这种**物理约束**使得模型能够在无配对数据的情况下进行自监督学习。

  ```python
      def sigloss(self, input, target):
          if self.valid_mask:
              valid_mask = target > 0
              if self.max_depth is not None:
                  valid_mask = torch.logical_and(target > 0, target <= self.max_depth)
              input = input[valid_mask]
              target = target[valid_mask]
          if self.warm_up:
              if self.warm_up_counter < self.warm_iter:
                  g = torch.log(input + self.eps) - torch.log(target + self.eps)
                  g = 0.15 * torch.pow(torch.mean(g), 2)
                  self.warm_up_counter += 1
                  return torch.sqrt(g)
          g = torch.log(input + self.eps) - torch.log(target + self.eps)
          Dg = torch.var(g) + 0.15 * torch.pow(torch.mean(g), 2)
          # n, c, h, w = g.shape
          # norm = 1/(h*w)
          # Dg = norm * torch.sum(g**2) - (0.85/(norm**2)) * (torch.sum(g))**2
          return torch.sqrt(Dg)
  ```

  

### 3.2 目标检测头 (Detection Head)

- **特征融合**：采用路径聚合模块（类似于 PANet），融合 E5,P3,P2*E*5,*P*3,*P*2 特征，生成多尺度检测特征 D1*D*1 至 D4*D*4。
- **解耦检测头**：采用无锚框（Anchor-free）设计，通过两个独立的卷积层分别输出**类别分类**和**边界框回归**，大幅降低了计算复杂度。

### 3.3 轻量化设计

- **Backbone**：基于 **MobileNetV3** 和 **反向残差模块 (Inverted Residuals)** 构建，专为移动端设计。
- **优势**：相比传统的“增强+检测”串行网络，一体化网络共享特征提取层，内存占用降低约 50%，推理速度满足水下机器人实时作业需求。

<img width="1266" height="792" alt="image" src="https://github.com/user-attachments/assets/ed215759-18a7-4754-9c93-73b89006028d" />


------





## 4. OpenHarmony 侧原生 AI 实现

### 4.1 C++ NDK + MindSpore Lite 集成

- 在 OpenHarmony 工程中新增

  ```
  native
  ```

  模块：

  - 引入 `libmindspore_lite_ndk.so` 及相关头文件；
  - CMake 配置链接 MindSpore Lite 和 OpenCV（如有）；

- 封装

  ```
  libentry.so
  ```

  - 提供 `Detect(uint8_t* buf, int len, int width, int height)` C 接口；
  - 内部完成：
    - Buffer → Tensor 的转换（YCbCr/RGBA → RGB）；
    - resize、归一化、chw 排布；
    - session.run(model) 推理；
    - NMS 后处理与坐标还原；
    - 将检测结果序列化为 NAPI 能理解的数组/对象。

### 4.2 NAPI 层接口设计

- ```
  napi_init.cpp
  ```

  中导出：

  - `napi_value Detect(napi_env env, napi_callback_info info)`;

- ArkTS 中通过：

  ```
  import native from 'libentry.so';
  
  let results = native.detect(imageBuffer);
  ```

- 性能优化：

  - 避免频繁创建/销毁 Session 与 Tensor，尽量复用；
  - 尽可能将内存拷贝集中到 C++ 层完成，减少 JS 与 Native 之间数据来回。

------



## 5. 常见问题排查 (Troubleshooting)

### 5.1 NAPI 运行时依赖缺失 (Runtime Dependency Missing)

**现象描述**：
APP 启动或调用 AI 功能时崩溃，日志报错：

- `LoadNativeModule failed`
- `dlopen failed: library "libmindspore_lite_ndk.so" not found`
- `dlopen failed: library "libc++_shared.so" not found`

**原因分析**：
这是典型的**运行时链接错误**。

1. **链接关系**：你的业务动态库 `libentry.so` 在编译时链接了 `libmindspore_lite_ndk.so`（这就像借条上写着欠了 MindSpore 的“钱”）。
2. **加载失败**：当系统加载器 (Linker) 加载 `libentry.so` 时，会去寻找它依赖的库。如果这些依赖库没有被打包进 Hap 包的 `libs/arm64-v8a` 目录中，加载器就会找不到文件，从而拒绝加载 `libentry.so`。

**解决方案**：

1. **检查 CMakeLists.txt**：确保正确配置了 `target_link_libraries`。
2. **配置动态库打包**：
   在 `entry/build-profile.json5` 或模块级 `build-profile.json5` 中，确保第三方库被正确引用。
3. **手动拷贝 (最稳妥)**：
   将 `libmindspore_lite_ndk.so` 和 `libc++_shared.so` (如果需要) 放置在工程的 `entry/libs/arm64-v8a/` (或对应架构) 目录下。OpenHarmony 构建系统会自动将这些 `.so` 文件打包进应用，确保运行时可见。

### 5.2 NAPI 类型断言失败

**现象**：`napi assert failed`。
**解决**：检查 ArkTS 传入 NAPI 的参数类型。例如，C++ 期望接收 `napi_object` (ArrayBuffer)，但 JS 端传入了 `undefined` 或 `string`。务必在 C++ 端增加 `napi_typeof` 检查。

------



## 6. 实验设计与测试评估（对应技术报告要求）

内容可包括：

1. **实验环境**：DAYU200 平台参数、CPU 核数、内存，ROS 小车硬件配置；
2. **测试场景**：室内走廊、光线较暗/有水渍地面、有障碍物等；
3. 评估指标
   - 检测精度：mAP、召回率；
   - 实时性：FPS（DAYU200 上整链路推理 + 显示）、端到端时延；
   - 稳定性：长时间运行（>30min）是否出现崩溃、内存泄漏。
4. **结果分析**：对比“无增强直接检测 vs 增强+检测”的效果差异、对比 CPU 优化前后的 FPS 改善等。

------

## 7. 源代码结构与复现说明（对应“可复现”要求）



在 README 或技术报告中，需要说明：

- 环境依赖与版本；
- 训练脚本如何运行（如需的话）；
- 如何从源码构建并在 DAYU200 + ROS 小车上运行整个系统。

------

## 8. 创新点与应用价值总结

可以从以下几个维度提炼：

1. **异构融合创新**：首次将 OpenHarmony 原生 AI 与传统 ROS 机器人深度打通，提供可复用的参考架构。
2. **原生 AI 深度使用**：完整链路覆盖“PyTorch 训练 → ONNX → MindSpore Lite → NAPI C++ 接口 → ArkTS 调用”。
3. **场景针对性增强模型**：针对水下/潮湿退化场景的 MobileViT 联合增强+检测模型，提高感知鲁棒性。
4. **端侧闭环控制**：在 DAYU200 上实现 AI 判决并直接驱动 ROS 小车运动，时延小、可靠性高。
5. **生态贡献**：所积累的 NAPI + MindSpore Lite 封装代码，可开源为 OpenHarmony 开发者提供一个“从模型到应用”的完整范例。

------

## 9. 交付物对应关系（初赛 / 决赛）

### 9.1 初赛交付物

- **代码**：OHOS App + ROS 端 Demo 代码；
- **设计文档**：本技术文档（或其精简版），包含架构、类图、流程图、接口设计；
- **演示视频**：机器人场景运行演示（含 UI、控制、检测效果）。

### 9.2 决赛交付物

- **完整作品代码**：在初赛基础上完善，并整理为可编译运行的工程；

- 技术报告（≥2000 字）

  ：按大赛要求，包含：

  - 研究背景
  - 方法论（系统架构、模型算法）
  - 实验设计（场景、数据、指标）
  - 模型训练（若涉及重新训练）
  - 测试评估（性能与效果）
  - 创新点
  - 应用价值
  - 与 OpenHarmony、原生 AI 的融合深度

- **PPT 路演材料**：介绍代码架构设计、功能实现、亮点与演示截图。
