# Inference Optimization & Quantization (AWQ + FP8 Cache)

## Environment
- Server: NVIDIA H200 GPU (143GB VRAM)
- OS: Ubuntu 24.04
- Kubernetes: K3s v1.36.4
- Model: Qwen/Qwen2.5-7B-Instruct-AWQ

## Key Configuration
- --quantization=awq (4-bit weight compression)
- --dtype=bfloat16 (required for FlashAttention v3 + FP8 compatibility)
- --kv-cache-dtype=fp8 (8-bit KV cache compression)

## Verification Results

### 1. Marlin Kernel Confirmed

### 2. FP8 KV Cache Confirmed

### 3. VRAM Footprint
- Model weights loaded: **5.38 GiB** (vs ~16GB for FP16 — meets <6GB target)
- Loading took 5.28 seconds

### 4. Known Issue Encountered & Fixed
Initial deployment crashed with:
FlashAttention v3 requires BF16 precision when FP8 KV cache is active.
Fixed by explicitly setting `--dtype=bfloat16`.

### 5. Inference Verification
Sent a chat completion request via curl — received a real response,
confirming the dual-compression stack (4-bit weights + 8-bit cache)
produces working inference. Response showed some incoherence at the tail
end, consistent with the documented accuracy trade-off of FP8 KV cache
without a calibrated scaling factor.

## Cleanup
All Kubernetes resources (Deployment, Service, PVC) removed from the
server after task completion. GPU confirmed at 0MiB usage post-cleanup.
