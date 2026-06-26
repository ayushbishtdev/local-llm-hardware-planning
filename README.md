# Local LLM Hardware Sizing & Benchmarks (2026)

This repository provides empirical hardware constraints, VRAM sizing matrices, and power-draw baselines for running local LLMs in 2026. Designed for engineering teams bypassing cloud APIs, these references cover everything from unified-memory Mini PCs to multi-GPU tensor parallelism. Last updated: June 2026.

## What's in this repo

*   [VRAM Sizing by Parameter Count](#vram-sizing--quantization) — Exact gigabyte requirements for 7B to 671B models.
*   [Bandwidth vs. Capacity Rankings](#memory-bandwidth-vs-capacity) — Why 128GB of RAM can be slower than a 32GB GPU.
*   [Multi-GPU Rig Architecture](#building-a-multi-gpu-inference-rig) — The $2,000 dual-RTX 3090 blueprint.
*   [Power Draw & OpEx](#tco--power-consumption) — Idle vs. load wattage and HVAC overhead.
*   [Quick Reference Assets](#quick-reference-assets) — Downloadable CSVs and architecture cheatsheets.

---

## VRAM Sizing & Quantization

Parameter counts are highly misleading without factoring in quantization and KV cache overhead. Running at Q4_K_M (4-bit Medium) quantization cuts the required VRAM by roughly 75% compared to native FP16. You must calculate VRAM by multiplying the parameter count by the precision byte-weight (0.5 for 4-bit) and adding a mandatory 15–20% overhead for the context window cache. A quantized 70B model demands ~42GB to comfortably fit in active memory.

| Model Size | Native FP16 (No Quantization) | 8-Bit (Q8) | 4-Bit (Q4) | Minimum Hardware Target (Q4) |
| :--- | :--- | :--- | :--- | :--- |
| **7B** | ~16 GB | ~9 GB | ~6 GB | 8GB - 12GB VRAM |
| **13B** | ~30 GB | ~16 GB | ~10 GB | 12GB - 16GB VRAM |
| **32B** | ~72 GB | ~38 GB | ~22 GB | 24GB VRAM (RTX 3090/4090) |
| **70B** | ~160 GB | ~80 GB | ~42 GB | 48GB VRAM (Dual GPU / Mac) |
| **120B** | ~270 GB | ~135 GB | ~72 GB | 80GB - 96GB VRAM |
| **671B (MoE)** | ~1,400 GB | ~700 GB | ~380 GB | Server Cluster / Cloud API |

**Full breakdown:** [Exact VRAM calculation formulas and MoE sizing](https://aidevdayindia.org/blogs/run-local-llm-hardware-guide/vram-requirements-by-model-size.html)

---

## Memory Bandwidth vs. Capacity

Capacity acts as a hard ceiling (if the model doesn't fit, it spills over the PCIe bus, crashing generation speed). Bandwidth, however, strictly dictates your tokens-per-second output limit. A 128GB unified memory Strix Halo machine can hold massive models, but its ~256 GB/s bus will generate text far slower than a 32GB RTX 5090 pushing 1,800 GB/s. 

| Hardware Platform | Memory Capacity | Memory Bandwidth | Format Focus |
| :--- | :--- | :--- | :--- |
| **AMD Strix Halo (Ryzen AI Max+)** | Up to 128GB | ~256 GB/s | GGUF |
| **NVIDIA DGX Spark** | 128GB | 273 GB/s | AWQ / GPTQ |
| **Apple Mac Studio (M4 Max)** | Up to 128GB | 546 GB/s | GGUF (Metal) |
| **Discrete NVIDIA RTX 5090** | 32GB | 1,800 GB/s | AWQ / GPTQ (CUDA) |

**Full breakdown:** [Architectural differences between unified memory and VRAM](https://aidevdayindia.org/blogs/run-local-llm-hardware-guide/unified-memory-vs-vram-explained.html)

---

## Building a Multi-GPU Inference Rig

To run a 70B model efficiently without enterprise pre-built costs, developers use tensor parallelism across consumer GPUs. Sourcing two used RTX 3090s yields 48GB of high-speed VRAM for under $2,000. This requires a motherboard supporting PCIe bifurcation down to an x8/x8 electrical minimum to prevent PCIe bottlenecks, and a Tier-A 1200W–1600W power supply. 

Using `vLLM`, you can shard the model weights symmetrically across both cards:
```bash
python -m vllm.entrypoints.openai.api_server \
    --model neural-matrix-70b-q4 \
    --tensor-parallel-size 2 \
    --port 8000
