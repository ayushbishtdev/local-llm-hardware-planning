# Local LLM Hardware Planning (2026)

A technical reference for sizing, building, and operating local LLM infrastructure. This repository provides exact VRAM requirement tables, memory bandwidth benchmarks, and Total Cost of Ownership (TCO) calculations for running 7B to 70B reasoning models locally without relying on expensive cloud APIs.

*Last updated: June 2026*

## What's in this repo

*   [**1. Sizing VRAM & Quantization**](#1-sizing-vram--quantization): Formulas for calculating exact model memory footprints.
*   [**2. Bandwidth vs. Capacity (Unified vs VRAM)**](#2-bandwidth-vs-capacity): Why 128GB of RAM can be slower than 32GB of VRAM.
*   [**3. GPU & Mini PC Benchmarks**](#3-gpu--mini-pc-benchmarks): RTX 5090 vs PRO 6000 vs Strix Halo vs DGX Spark.
*   [**4. Multi-GPU Builds & TCO**](#4-multi-gpu-builds--power-tco): PCIe bifurcation and 24/7 electricity costs.
*   [**Quick Reference Assets**](#quick-reference-assets): CSV tables and build BOMs.

---

## 1. Sizing VRAM & Quantization

Do not buy hardware based on native FP16 parameter counts. Standard 2026 inference utilizes 4-bit quantization (Q4_K_M), which slashes the base model weight memory footprint by roughly 75%. 

**The Calculation:**
*   `Parameters × 0.5 bytes = Base VRAM (at Q4)`
*   **KV Cache Tax:** Always add an extra 15% to 20% VRAM capacity to accommodate the context window. An 8K context typically uses 1GB-3GB, while a 128K context can consume over 20GB.

| Model Size | Native FP16 Size | Q4 Quantized Size | Target Hardware (Minimum) |
| :--- | :--- | :--- | :--- |
| **7B** | ~16 GB | ~6 GB | 8GB - 12GB VRAM |
| **13B** | ~30 GB | ~10 GB | 12GB - 16GB VRAM |
| **32B** | ~72 GB | ~22 GB | 24GB VRAM (RTX 3090/4090) |
| **70B** | ~160 GB | ~42 GB | 48GB VRAM (Dual GPU / Mac) |

*Format Selection:* Use **AWQ / GPTQ** for dedicated NVIDIA hardware to maximize token speed. Use **GGUF** for Apple Silicon or systems that may need to offload layers to system RAM.

**Full breakdown:** [How Much VRAM to Run Any LLM](https://aidevdayindia.org/blogs/run-local-llm-hardware-guide/vram-requirements-by-model-size.html)

---

## 2. Bandwidth vs. Capacity

Capacity dictates *if* a model loads; memory bandwidth dictates *how fast* tokens generate.

*   **Prompt Processing (TTFT):** Time-to-First-Token is compute-bound (reliant on matrix math cores).
*   **Token Generation Speed:** The autoregressive loop is strictly memory-bandwidth bound. 

A massive 128GB unified memory pool allows a 70B model to load, but its narrow bus (e.g., 256 GB/s) will output tokens much slower than a dedicated 32GB GPU pushing 1,800 GB/s on a smaller model. If a model exceeds your dedicated GPU VRAM, it triggers a PCIe spill into standard RAM, crashing generation speeds.

**Full breakdown:** [Why 128GB Can Run LLMs Slower Than 32GB](https://aidevdayindia.org/blogs/run-local-llm-hardware-guide/unified-memory-vs-vram-explained.html)

---

## 3. GPU & Mini PC Benchmarks

If tokens-per-second is your priority, use discrete GPUs. If massive single-pool capacity is required on a budget, unified memory Mini PCs are the 2026 standard.

| Hardware Platform | Memory Pool | Bandwidth | Best For |
| :--- | :--- | :--- | :--- |
| **NVIDIA RTX 5090** | 32GB VRAM | 1,800 GB/s | Premium consumer speed (Sub-32B models) |
| **NVIDIA RTX 3090 (Used)** | 24GB VRAM | 936 GB/s | Budget multi-GPU scaling |
| **NVIDIA PRO 6000** | 96GB VRAM | 960 GB/s | Enterprise 70B single-slot deployments |
| **NVIDIA DGX Spark** | 128GB Unified | 273 GB/s | High-capacity CUDA-native desktop AI |
| **AMD Strix Halo (Mini PC)** | 128GB Unified | ~256 GB/s | Cost-effective 70B inference ($2K-$4K) |
| **Apple Mac Studio (M4 Max)**| 128GB Unified | 546 GB/s | High-bandwidth unified memory setups |

**Full breakdowns:** [Best GPU for Local LLMs](https://aidevdayindia.org/blogs/run-local-llm-hardware-guide/best-gpu-for-local-llm.html) | [Mini PCs for Local LLMs](https://aidevdayindia.org/blogs/run-local-llm-hardware-guide/mini-pc-for-local-llm-inference.html)

---

## 4. Multi-GPU Builds & Power TCO

Two used RTX 3090s (yielding 48GB of effective VRAM via tensor parallelism) can outperform a turnkey enterprise AI box for under $2,000. However, you must account for 24/7 Total Cost of Ownership (TCO):

*   **Idle Power:** A dual-GPU rig idles at 120W–150W. A unified mini PC idles at just 15W–30W.
*   **Load Power:** Running a 70B model spikes a dual-GPU rig to 750W–850W.
*   **HVAC Overhead:** A 750W server acts like a space heater. Add 20% to 30% to your electricity bill strictly for ambient cooling compensation.

**Full breakdowns:** [Build a Multi-GPU Local LLM Rig Under $2K](https://aidevdayindia.org/blogs/run-local-llm-hardware-guide/build-local-llm-rig-multi-gpu.html) | [The Real Cost of a 24/7 Local LLM Box](https://aidevdayindia.org/blogs/run-local-llm-hardware-guide/local-llm-power-cooling-costs.html)

---

## Quick Reference Assets

*   [`data/vram-requirements-2026.csv`](data/vram-requirements-2026.csv): A raw structured dataset of FP16, Q8, and Q4 VRAM requirements for embedding into your own provisioning calculators.
*   [`CHEATSHEET.md`](CHEATSHEET.md): A dense 1-pager on hardware math (bandwidth vs capacity, Q4 parameter mapping).
*   [`multi-gpu-bom.md`](multi-gpu-bom.md): The minimum viable bill of materials and `vLLM` software execution flags for a dual-RTX 3090 build.

---

## Sources & Deeper Reading

All architecture guidelines and hardware tracking in this repository are based on 2026 empirical benchmarking by Ayush Bisht at AI DEV DAY.

*   [Best Hardware to Run Local LLMs in 2026](https://aidevdayindia.org/blogs/run-local-llm-hardware-guide/run-local-llm-hardware-guide.html)
*   [How Much VRAM to Run Any LLM](https://aidevdayindia.org/blogs/run-local-llm-hardware-guide/vram-requirements-by-model-size.html)
*   [Why 128GB Can Run LLMs Slower Than 32GB](https://aidevdayindia.org/blogs/run-local-llm-hardware-guide/unified-memory-vs-vram-explained.html)
*   [Quantization: Cut LLM VRAM by 75%](https://aidevdayindia.org/blogs/run-local-llm-hardware-guide/quantization-vram-savings-guide.html)
*   [Best GPU for Local LLMs](https://aidevdayindia.org/blogs/run-local-llm-hardware-guide/best-gpu-for-local-llm.html)
*   [Mini PCs for Local LLMs](https://aidevdayindia.org/blogs/run-local-llm-hardware-guide/mini-pc-for-local-llm-inference.html)
*   [Build a Multi-GPU Local LLM Rig](https://aidevdayindia.org/blogs/run-local-llm-hardware-guide/build-local-llm-rig-multi-gpu.html)
*   [The Real Cost of a 24/7 Local LLM Box](https://aidevdayindia.org/blogs/run-local-llm-hardware-guide/local-llm-power-cooling-costs.html)

## Contributing / Corrections

Hardware pricing and software orchestration (vLLM, llama.cpp) move fast. If you discover updated bandwidth throughputs or newer motherboard bifurcation standards, please open a PR to keep these specifications accurate.

---

**About the Author**
Created by Ayush Bisht, a Content Engineer and AI Tools Specialist focused on building scalable, private AI ecosystems. Find more hardware testing at [aidevdayindia.org](https://aidevdayindia.org/).
