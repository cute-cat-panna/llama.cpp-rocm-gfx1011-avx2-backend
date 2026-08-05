# llama.cpp ROCm 后端包 (gfx1011 / AMD BC-160)

自编译 llama.cpp (build `42fc24306`) HIP 后端,目标架构 **gfx1011** (AMD BC-160,Radeon Pro VII 同芯)。
已内置完整 ROCm 7.14 运行时(含 110 个 gfx1011 rocblas 内核),**运行时无需安装/配置 ROCm 或 therock**,解压即用。

## 适用条件

- GPU 必须是 **gfx1011** (AMD BC-160)。本包为单一架构编译,其它 GPU 上无法加载 HIP 模块。
- LM Studio >= 0.4.0+15 (后端目录已有 `win-llama-rocm-vendor-v6` vendor 包)。

## 替换步骤

1. **备份**:把 `llama.cpp-win-x86_64-amd-rocm-avx2-2.27.1` 整个目录复制一份(如加 `.bak` 后缀)。
2. **覆盖**:把本包内所有文件/文件夹复制进
   `C:\Users\<用户名>\.lmstudio\extensions\backends\llama.cpp-win-x86_64-amd-rocm-avx2-2.27.1\`
   ,同名文件全部覆盖。
   > 若弹出"目标已存在"提示,选择覆盖。**保留** LM Studio 原有文件不要删:
   > `backend-manifest.json`、`display-data.json`、`engine-protocol-server-artifacts.json`、
   > `llm_engine.dll`、`llm_engine_rocm.node`、`liblmstudio_bindings_rocm.node`、
   > `ggml_llamacpp.dll`、`lmstudiocore.dll`。
3. **保留 vendor 包**:不要删除 `..\vendor\win-llama-rocm-vendor-v6`。运行时 DLL 会优先从后端目录加载,自动盖过 vendor 中的旧版 ROCm。
4. **改 manifest**:编辑 `backend-manifest.json`,在 `gpu.targets` 中加 `"gfx1011"`(否则 LM Studio 可能不认该 GPU):

   ```json
   "targets": [
     "gfx1011",
     "gfx1030",
     "gfx1100",
     ...
   ]
   ```

5. 重启 LM Studio,重新加载模型。

## 验证

- 底部/M 标显示引擎为 `llama.cpp`,GPU 显示 `AMD BC-160`。
- 服务器日志:`.lmstudio\server-logs\` 中可见 `llama-server ... listening on http://127.0.0.1` 即表示走本包。

## 实测参考(gemma-4-E4B Q6_K,7.5B 参数,gfx1011)

| 项 | 速度 |
| --- | --- |
| prompt 处理 (pp32) | ~137 t/s |
| 生成 (tg32) | ~42 t/s |
| 8K 上下文长对话 | ~20-22 t/s (gfx1011 无 flash-attention,长上下文生成速度按上下文增长下降) |

## 内容说明

- 构建产物:`llama-server.exe`、`llama-server-impl.dll`、`llama.dll`、`llama-common.dll`、`mtmd.dll`、`ggml.dll`、`ggml-base.dll`、`ggml-cpu.dll`、`ggml-hip.dll`、`llama-cli.exe`、`llama-bench.exe` 等
- ROCm 运行时:`amdhip64_7.dll`、`rocblas.dll` + `rocblas/library`(含 110 个 gfx1011 内核)、`hipblas.dll`、`libhipblaslt.dll` + `hipblaslt/library`、`rocsolver.dll`、`amd_comgr.dll`、`rocm_kpack.dll`
- VC 运行库:`MSVCP140.dll`、`VCRUNTIME140.dll`、`VCRUNTIME140_1.dll`

## 更新后端版本时

LLM 引擎/服务器文件(`llama-server.exe`、`llama.dll`、`ggml*.dll` 等)会随上游 llama.cpp 更新重新编译;替换时同上覆盖即可。vendor 包与 `llm_engine*.node` 等 LM Studio 文件请保持原样。