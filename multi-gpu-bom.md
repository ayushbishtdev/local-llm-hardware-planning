# Sub-$2K Dual-GPU Local LLM Rig (BOM & Config)

To run a 70B model efficiently, you can combine two 24GB consumer GPUs rather than paying $4,699+ for an integrated enterprise box.

## Bill of Materials (Core Infrastructure)
1. **GPUs:** 2× Used NVIDIA RTX 3090s (yielding 48GB combined VRAM).
2. **Motherboard:** Enthusiast or Workstation board supporting **x8/x8 PCIe electrical bifurcation**. (Do not use boards that drop the second slot to x4 speeds).
3. **Power Supply:** 1200W to 1600W Tier-A PSU. Ensure independent PCIe power rails for each GPU, no daisy-chaining.
4. **Chassis:** Open-air frame or an extra-wide case with PCIe risers to prevent thermal throttling.

## Software Orchestration (vLLM)
You do not strictly need a physical NVLink bridge for standard inference. Standard PCIe 4.0 is enough. Use an engine like `vLLM` to manage tensor parallelism.

```bash
# Launching a model across two GPUs
python -m vllm.entrypoints.openai.api_server \
    --model your-70b-model-q4 \
    --tensor-parallel-size 2 \
    --port 8000
