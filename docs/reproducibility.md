# Reproducibility

For each run, record the date and Colab GPU; Python, PyTorch, CUDA, ComfyUI, custom-node and notebook revisions; exact model repository/revision and quantization file; prompt, seed, dimensions, sampler, scheduler, CFG, steps and adapters; generation-cell runtime and total end-to-end runtime separately; output dimensions, file size, FPS, duration and frame count; peak GPU-wide memory measurement method and sampling interval; and safety scan status plus number of video frames inspected.

Do not infer runtime VRAM from model-file sizes. Distinguish PyTorch allocated/reserved memory from GPU-wide usage reported by `nvidia-smi`.
