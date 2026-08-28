# llama-cpp-python · CUDA 便携版 (SuperMate 自编译)

> **"官方轮子在你机器上崩了？这个不会。"**
> 为没有 AVX-512 指令集的 CPU 而生 —— llama-cpp-python 0.3.35 · Windows · CUDA 13 · 便携指令集

---

## 解决了什么问题

官方 abetlen 轮子（cu124 / cu130 等）以 **AVX-512 指令集编译**。在不支持 AVX-512 的 CPU（如
Intel i7-14700KF / i5-13400 等消费级 Raptor Lake）上，加载模型即崩溃：

```
Windows Error 0xc000001d (非法指令) — llama_init_from_model
```

本包以 **`GGML_NATIVE=OFF`（便携指令集）** 重新编译，任何 CPU 均可运行，
同时保留完整 CUDA GPU 加速。

## 特性

| 项 | 值 |
|---|---|
| 版本 | llama-cpp-python **0.3.35**（支持 **qwen35** 架构：Qwen3.5 / Qwen3.8 系列） |
| 平台 | Windows x64 · Python 3.12（`py3-none` 通用 ABI） |
| GPU 覆盖 | **RTX 20 系 ~ 50 系**（sm_75 / 80 / 86 / 89 / 90 / 100 / **120**） |
| CUDA 运行时 | **已捆绑**（cudart64_13 / cublas64_13 / cublasLt64_13），**无需预装 CUDA** |
| CPU 兼容 | 任意 x64 CPU（无 AVX-512 要求） |
| 体积 | 约 631 MB（含多架构 GPU 内核 + CUDA 运行时） |

## 安装

```powershell
pip install llama_cpp_python-0.3.35-py3-none-win_amd64.whl
```

### ComfyUI 中使用

把 `ComfyUI_SuperMate-LLM-Nodes` 文件夹放入 `ComfyUI/custom_nodes/`，重启 ComfyUI：

| 节点 | 功能 |
|---|---|
| **SuperMate LLM Loader** | 加载 `models/LLM/` 下的 GGUF（Qwen3.8 等），`n_gpu_layers=-1` 全量进 GPU；选 `mmproj` 启用视觉 |
| **MiniMax 提示词增强** | 一张图 / 一句话 → 完整 **MiniMax H3 提示词**。模式：图生视频（接图看图写，参考图用途=场景参考/人物参考）/ 文生视频 / 续段；不接图即纯文本 |
| **SuperMate 图片反推** | 图片 → 详细描述提示词（自动缩放到 1024，约 13 倍提速；含画面比例） |
| **SuperMate Text Gen** | 通用文本生成（口播文案 / 翻译 / 总结） |
| **SuperMate LLM Unload** | 释放显存。接在工作流链尾（提示词/text 输出 → 输入）即 LLM 用后自动卸载；也可独立放在 H3 工作流开头，执行时先卸载残留 LLM 再生成 |

模型放法：GGUF 文件（如 `Qwen3.8-27B-UD-IQ3_XXS.gguf` + 视觉 `mmproj-BF16.gguf`）
放入 `ComfyUI/models/LLM/` 即可。

## 附带工作流（已实测跑通）

放在 `ComfyUI/user/default/workflows/SuperMate/`，Workflow → Open 即可打开：

| 工作流 | 用法 |
|---|---|
| **SuperMate-MiniMax提示词增强(UI).json** | 传图 + 填需求（如"根据图片内容设计一段vlog视频"）→ 约 1 分钟出完整 H3 提示词（分镜/转场/比例/时长）；不接图（断开 LoadImage 连线）即纯文本，可切换 文生视频/图生视频/续段 |
| **SuperMate-图片反推(UI).json** | 传图 → 反推详细描述 → 预览。约 35 秒 |

> **GitHub Release 资产名说明**：受 GitHub 平台限制（资产名仅支持 ASCII），Release 里的工作流文件名为
> `SuperMate-MiniMaxPrompt-UI.json`（= 提示词增强）与 `SuperMate-ImageToPrompt-UI.json`（= 图片反推）；
> 内容与上方中文名一致，下载后可直接放入 `ComfyUI/user/default/workflows/SuperMate/`。

注意事项：**一次只跑一个工作流**（Qwen3.8 27B 含视觉约 13GB 显存，16GB 卡需留余量）；输出在只读文本区，可一键复制。

## 自编译复现参数

```powershell
$env:CMAKE_GENERATOR = "Ninja"
$env:CMAKE_ARGS = '-DGGML_NATIVE=OFF -DGGML_CUDA=ON `
  "-DCMAKE_CUDA_COMPILER=C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v13.3/bin/nvcc.exe" `
  "-DCMAKE_CUDA_ARCHITECTURES=75;80;86;89;90;100;120"'
pip wheel llama-cpp-python==0.3.35 --no-binary llama-cpp-python
```

构建环境：CUDA Toolkit 13.3 + MSVC 2022 + CMake/Ninja（scikit-build-core 自动拉取）。

## 实测

- RTX 5080（sm_120）· Qwen3.8-27B (IQ3_XXS, 11.9GB) · 全量 GPU 层
- 模型加载 + 上下文：约 8 秒
- 生成：约 40+ tokens/s
- 无 AVX-512 CPU（i7-14700KF）验证通过（修复 0xc000001d）

## 许可证与致谢

- 代码：MIT（llama.cpp / llama-cpp-python 均为 MIT）
- CUDA 运行时：NVIDIA 允许再分发
- 致谢：abetlen/llama-cpp-python · llama.cpp · 秋叶绘世启动器 · Work-Fisher 整合包基础
