# SIBR_viewers 项目功能与使用指南

## 1. 项目简介

`SIBR_viewers` 是 SIBR Core 的一个工程仓库，提供了基于图像的渲染（Image-Based Rendering, IBR）相关的核心库、交互式查看器、Gaussian Splatting 查看器，以及一组数据预处理工具。

这个仓库的代码大致可以分为四类：

1. 核心库：负责窗口、图形、相机、场景、渲染、视频、几何处理等基础能力。
2. 渲染项目：提供 `basic`、`ulr`、`gaussianviewer`、`remote` 等实际可运行的应用。
3. 数据预处理工具：用于数据集整理、裁剪、去畸变、相机格式转换、网格处理、Colmap 流程整理等。
4. 文档与资源：Doxygen 文档页面、着色器、示例资源和配置。

## 2. 目录结构概览

### 2.1 核心模块

`src/core/` 下的主要模块如下：

- `system`：命令行参数、数学工具、字符串、矩阵、计时、线程辅助等基础设施。
- `graphics`：窗口、OpenGL 纹理、RenderTarget、Shader、Mesh、GUI 辅助等。
- `renderer`：深度渲染、点渲染、法线渲染、阴影、纹理网格渲染、Poisson 等渲染器。
- `raycaster`：基于 Embree 的射线求交、相机射线、体素和几何辅助。
- `scene`：场景解析、输入图像、代理网格、相机集合、渲染目标纹理。
- `view`：交互相机、轨道球、场景调试视图、多视图管理、图像网格等。
- `assets`：输入相机、图像列表、资源定位、UV 展开等数据资源管理。
- `imgproc`：图像裁剪、网格贴图、Poisson 重建、MRF、去畸变等。
- `video`：视频读写与编码、视频工具、FFmpeg 编码器。

### 2.2 项目模块

`src/projects/` 下的主要项目如下：

- `basic`：基础点云 / 网格查看器。
- `ulr`：Unstructured Lumigraph Rendering 相关查看器。
- `gaussianviewer`：3D Gaussian Splatting 模型查看器。
- `remote`：远程 Gaussian 查看与交互相关应用。
- `dataset_tools`：数据预处理工具集合。

## 3. 主要功能总结

## 3.1 基础查看器

### `SIBR_PointBased_app`

用途：

- 加载标准 SIBR 数据集。
- 以点为中心的方式查看场景。
- 适合快速检查数据集、相机、场景结构。

常用参数：

- `--path`：数据集根目录，必需。
- `--mesh`：可选的网格路径。
- `--offscreen`：离屏运行。
- `--nogui`：关闭 ImGui。
- `--pathFile`：离线路径渲染输入。

示例：

```bash
install/bin/SIBR_PointBased_app --path /path/to/dataset
```

### `SIBR_texturedMesh_app`

用途：

- 加载标准 SIBR 数据集并显示贴图网格。
- 适合检查重建网格、纹理贴图和基本视角切换。

常用参数：

- `--path`：数据集根目录，必需。
- `--mesh`：自定义网格路径。
- `--texture`：自定义纹理路径。
- `--noScene`：禁用部分场景逻辑。

示例：

```bash
install/bin/SIBR_texturedMesh_app --path /path/to/dataset
```

## 3.2 ULR 查看器

### `SIBR_ulr_app`

用途：

- 旧版 ULR 查看器。
- 适合兼容旧流程或进行算法对比。

### `SIBR_ulrv2_app`

用途：

- 更新版 ULR 查看器。
- 集成多种 ULR 实现方式。
- 支持交互查看与离线路径播放。

典型功能：

- 从标准 SfM / MVS 数据集中进行基于图像的重渲染。
- 支持从命令行读取路径文件做离线播放。

示例：

```bash
install/bin/SIBR_ulrv2_app --path /path/to/dataset
```

离屏路径渲染示例：

```bash
install/bin/SIBR_ulrv2_app \
  --path /path/to/dataset \
  --offscreen \
  --pathFile /path/to/camera_path.out
```

## 3.3 Gaussian Splatting 查看器

### `SIBR_gaussianViewer_app`

用途：

- 加载 3D Gaussian Splatting 训练输出结果。
- 交互查看 Gaussian splats、相机、场景结构。
- 支持窗口模式和离屏 / 无 GUI 验证模式。

本仓库当前已在 Ubuntu 上验证可用。

必需参数：

- `--path`：原始数据集根目录。
- `--model-path`：训练输出目录。
- `--iteration`：要加载的迭代轮次。

常用参数：

- `--device`：CUDA 设备编号。
- `--load_images`：加载原始图像用于场景概览。
- `--no_interop`：禁用 OpenGL / CUDA interop，某些环境更稳定。
- `--offscreen`：离屏运行。
- `--nogui`：关闭 GUI。

窗口模式示例：

```bash
install/bin/SIBR_gaussianViewer_app \
  --path /path/to/dataset_root \
  --model-path /path/to/training_output \
  --iteration 30000 \
  --no_interop
```

离屏验证示例：

```bash
install/bin/SIBR_gaussianViewer_app \
  --path /path/to/dataset_root \
  --model-path /path/to/training_output \
  --iteration 30000 \
  --offscreen --nogui --no_interop
```

本机验证示例：

```bash
install/bin/SIBR_gaussianViewer_app \
  --path /home/sil/workspace/gaussian-splatting/data/2024_06_04_13_44_39_undist \
  --model-path /home/sil/workspace/gaussian-splatting/data/output/demo \
  --iteration 30000 \
  --no_interop
```

## 3.4 Remote Gaussian 应用

### `SIBR_remoteGaussian_app`

用途：

- 用于远程 Gaussian 场景查看 / 交互。
- 适合与远程训练或远程场景服务协同使用。

说明：

- 该应用和 `gaussianviewer` 共用大量场景与渲染基础设施。
- 如果你在本地直接查看 Gaussian 模型，通常优先使用 `SIBR_gaussianViewer_app`。

## 3.5 数据预处理工具

`src/projects/dataset_tools/preprocess/` 下包含一批独立的可执行工具或 Python 脚本。

主要包括：

- `cameraConverter`
  用于相机路径或相机文件格式转换。

- `clippingPlanes`
  计算每张图像对应的近平面 / 远平面。

- `cropFromCenter`
  将多张图像裁剪到统一尺寸并保持居中。

- `distordCrop`
  去畸变并进一步裁剪图像。

- `prepareColmap4Sibr`
  将 Colmap 结果整理为 SIBR 可用格式。

- `nvmToSIBR`
  将 VisualSFM `.nvm` 转换为 SIBR 数据格式。

- `unwrapMesh`
  调用 `xatlas` 为网格生成 UV。

- `textureMesh`
  根据已标定相机和 UV 网格生成贴图。

- `alignMeshes`
  进行网格对齐。

- `tonemapper`
  旧版色调映射工具。

- `fullColmapProcess`
  基于 Python 的从图像到 Colmap 数据集的流程脚本。

### 典型预处理脚本

- `src/projects/dataset_tools/preprocess/converters/generate_list_images.py`
  生成 `list_images.txt`。

- `src/projects/dataset_tools/preprocess/converters/ibr_preprocess_rc_to_sibr.py`
  将 RealityCapture 输出整理为 SIBR 格式。

- `src/projects/dataset_tools/preprocess/converters/simplify_mesh.py`
  借助 MeshLab 简化网格。

- `src/projects/dataset_tools/preprocess/fullColmapProcess/fullColmapProcess.py`
  一键式 Colmap 处理流程。

## 4. Ubuntu 安装方法

以下步骤面向 Ubuntu 系统。

## 4.1 系统依赖安装

先更新包索引：

```bash
sudo apt update
```

建议安装以下构建和运行依赖：

```bash
sudo apt install -y \
  build-essential \
  cmake \
  git \
  pkg-config \
  ninja-build \
  libboost-all-dev \
  libeigen3-dev \
  libglew-dev \
  libglfw3-dev \
  libopencv-dev \
  libassimp-dev \
  libembree-dev \
  libavcodec-dev \
  libavdevice-dev \
  libavformat-dev \
  libavutil-dev \
  libswscale-dev \
  libxxf86vm-dev \
  libxinerama-dev \
  libxcursor-dev \
  libxi-dev \
  libgl1-mesa-dev \
  mesa-common-dev
```

如果你需要运行 Gaussian 查看器，还需要：

1. NVIDIA 驱动
2. CUDA Toolkit

建议确认：

```bash
nvidia-smi
nvcc --version
```

## 4.2 获取源码

```bash
git clone https://github.com/waynewei618/sibr_core.git
cd sibr_core
```

## 4.3 配置、编译与安装

如果你的显卡架构是 RTX 3090，对应 `86`：

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release -DCMAKE_CUDA_ARCHITECTURES=86
cmake --build build -j$(nproc)
cmake --install build
```

安装完成后，主要程序位于：

```bash
install/bin/
```

## 4.4 编译产物

常见可执行文件包括：

```bash
install/bin/SIBR_PointBased_app
install/bin/SIBR_texturedMesh_app
install/bin/SIBR_ulr_app
install/bin/SIBR_ulrv2_app
install/bin/SIBR_gaussianViewer_app
install/bin/SIBR_remoteGaussian_app
```

## 5. 数据格式与运行输入

### 5.1 标准 SIBR / ULR 数据集

通常需要：

- 图像目录
- 标定相机
- 稀疏 / 稠密重建结果
- 代理网格或点云

常见入口参数是：

```bash
--path /path/to/dataset
```

### 5.2 Gaussian Splatting 模型

通常需要两个路径：

1. 原始数据集根目录 `--path`
2. 训练输出目录 `--model-path`

例如训练输出目录下通常有：

- `cfg_args`
- `cameras.json`
- `point_cloud/iteration_xxxxx/point_cloud.ply`

## 6. 常见使用场景

### 6.1 检查普通数据集是否可视化正常

```bash
install/bin/SIBR_texturedMesh_app --path /path/to/dataset
```

### 6.2 检查点云 / 相机布局

```bash
install/bin/SIBR_PointBased_app --path /path/to/dataset
```

### 6.3 查看 Gaussian 训练结果

```bash
install/bin/SIBR_gaussianViewer_app \
  --path /path/to/dataset_root \
  --model-path /path/to/output \
  --iteration 30000 \
  --no_interop
```

### 6.4 离屏验证 Gaussian 模型能否加载

```bash
timeout 20s install/bin/SIBR_gaussianViewer_app \
  --path /path/to/dataset_root \
  --model-path /path/to/output \
  --iteration 30000 \
  --offscreen --nogui --no_interop
```

如果进程稳定运行并最终因 `timeout` 退出，通常说明模型加载、OpenGL 上下文、CUDA 路径和主循环都已打通。

## 7. 交互方式

SIBR 的交互核心是“主视图 + 可选调试视图 + 可切换相机控制模式”。

对于 `SIBR_gaussianViewer_app`，最常见的是在主视图中查看 Gaussian 渲染结果，并通过浮动菜单切换导航模式、调整显示参数、查看相机布局。

### 7.1 主视图中可以做什么

主视图里常见操作包括：

- 自由浏览 Gaussian 场景。
- 在不同相机控制模式之间切换。
- 跳转到输入相机位置。
- 调整导航速度。
- 调整 Gaussian 显示尺寸。
- 切换是否显示初始点云。
- 切换 fast culling。

除 Gaussian splats 外，还可以在菜单中切换为以 ellipsoid 形式显示 Gaussian。

### 7.2 导航模式

SIBR 内部支持多种交互相机模式，常见的有：

- `FPS`
  第一人称飞行模式，适合在场景中自由漫游。
- `Orbit`
  绕目标点轨道式观察，适合围绕局部区域查看。
- `Interp.`
  插值相机模式，适合观察输入相机之间的视角过渡。
- `Trackball`
  轨道球模式，适合围绕一个关注点旋转、平移和缩放。
- `None`
  不启用交互控制。

默认通常是 `FPS`。

切换方式：

- 打开查看器后，在浮动菜单中找到相机控制相关面板。
- 在 `Mode` 下拉框中切换 `FPS / Orbit / Interp. / Trackball / None`。

### 7.3 FPS 模式常用快捷键

`FPS` 模式下，键盘控制如下：

- `W`：前进
- `S`：后退
- `A`：左移
- `D`：右移
- `Q`：下移
- `E`：上移
- `I`：抬头
- `K`：低头
- `J`：向左转
- `L`：向右转
- `U`：绕视线方向滚转
- `O`：反向滚转

### 7.4 Trackball 模式常用操作

`Trackball` 模式更适合观察单个物体或局部区域。

常用操作如下：

- 鼠标左键在视图中心区域拖动：围绕中心点旋转。
- 鼠标左键在视图边缘拖动：滚转视角。
- 鼠标右键在中心区域拖动：平移。
- 鼠标右键在边缘区域拖动：前进 / 后退。
- 鼠标滚轮：缩放。
- `Ctrl + 左键`：重新设置 Trackball 的中心点。

当你需要聚焦某个细节区域时，先用 `Ctrl + 左键` 重设中心，再用滚轮缩放，通常会比 `FPS` 更好用。

### 7.5 Snap to 与 Snap to closest

查看器支持快速跳到输入相机位置：

- `Snap to`
  直接跳到指定编号的输入相机。
- `Snap to closest`
  跳到离当前用户相机最近的输入相机。

这两个功能通常位于浮动菜单中，适合做以下检查：

- 当前观察位置是否接近真实采样视角。
- 相机位姿是否明显错位。
- 模型在训练视角附近和远离训练视角时的表现差异。

### 7.6 Top View 是什么

`Top View` 是一个额外的调试视图，不会替代主视图，而是增加一个从整体上观察场景和相机布局的窗口。

打开方式：

- `Views > Top view`

`Top View` 中通常会显示：

- SfM 点云。
- 输入相机的位置与方向。
- 当前用户相机的位置。

它的主要用途是：

- 看当前视角离输入相机有多远。
- 看输入相机的覆盖范围是否足够。
- 看当前观察位置是否已经跑出训练数据覆盖区域。
- 检查相机对齐和场景整体结构。

`Top View` 本身也是一个 3D 视图，可以像主视图一样导航，并支持：

- `FPS`
- `Trackball`
- `Orbit`

需要注意：

- 打开 `Top View` 后渲染通常会变慢一些。
- 在大场景或高分辨率下，额外视图会带来明显性能开销。

### 7.7 `--load_images` 与不加载的区别

`--load_images` 不会改变主视图里的 Gaussian 渲染结果。

它的主要作用是：

- 把输入图像也加载到查看器中。
- 让 `Top View` 或相关相机调试视图可以显示输入相机对应的原始图像。
- 方便检查相机对齐和图像内容。

因此：

- 不加 `--load_images`
  启动更轻，内存占用更低，适合单纯浏览模型。
- 加 `--load_images`
  更适合做数据集调试、相机检查和 `Top View` 相关分析。

如果你只是普通浏览场景，通常看不出视觉差异，这是正常现象。

### 7.8 其他常用 GUI 操作

浮动菜单中常见的可调项目包括：

- `Mode`
  切换导航模式。
- `Snap to`
  跳转到指定输入相机。
- `Snap to closest`
  跳转到最近输入相机。
- `Scaling Modifier`
  调整 Gaussian 显示尺寸。
- `fast culling`
  开关更激进的快速裁剪。
- 显示初始点云
  用于和 Gaussian 结果对照。

此外，`Menu > Display` 中通常可以关闭 `V-Sync`，在一些机器上这有助于释放帧率上限。

## 8. 常见问题

### 8.1 `--nogui` 或离屏模式崩溃

本仓库当前已经修复 `SIBR_gaussianViewer_app` 在 `--offscreen --nogui` 下的多处 ImGui 空上下文崩溃问题。

如果仍有异常，建议优先尝试：

```bash
--no_interop
```

### 8.2 CUDA 架构不匹配

如果编译时报 `Unsupported gpu architecture`，请根据你的 GPU 修改：

```bash
-DCMAKE_CUDA_ARCHITECTURES=XX
```

例如：

- RTX 3090: `86`
- 其他显卡请按实际 Compute Capability 调整

### 8.3 FFmpeg / Embree 版本兼容

在较新的 Ubuntu 上，系统通常提供新版 FFmpeg 和 Embree 4。
本仓库当前已经做了 Linux 侧兼容修正，可以在 Ubuntu 上正常编译和运行。

## 9. 建议使用顺序

如果你第一次接触这个项目，建议按下面顺序使用：

1. 先编译并安装整个工程。
2. 用 `SIBR_texturedMesh_app` 或 `SIBR_PointBased_app` 验证普通数据集。
3. 用 `SIBR_gaussianViewer_app` 验证 Gaussian 模型。
4. 如果需要离线流程，再使用 `dataset_tools` 的预处理工具。
5. 如果需要基于图像的重渲染，再尝试 `SIBR_ulrv2_app`。

## 10. 总结

`SIBR_viewers` 不是单一程序，而是一整套围绕 IBR 和 Gaussian Splatting 的：

- 核心图形 / 场景 / 相机基础库
- 多种交互式查看器
- 离屏与路径渲染能力
- 数据集预处理工具链

如果你的目标是：

- 查看普通重建结果：用 `basic`
- 做 ULR：用 `ulr`
- 查看 3D Gaussian Splatting：用 `gaussianviewer`
- 做远程交互：用 `remote`
- 整理数据：用 `dataset_tools`

这几个模块基本覆盖了该仓库的主要用途。
