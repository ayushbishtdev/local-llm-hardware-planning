# Local LLM Hardware Math Cheatsheet

Keep these baseline physics and math formulas in mind before provisioning any local AI hardware.

## 1. VRAM & Quantization Math
To size your hardware, do not look at FP16. Look at Q4 (4-bit).
*   **Formula:** `Model Parameters × 0.5 bytes = Q4 Weight Size`
*   **Context Window (KV Cache) Tax:** Add an additional 15% to 20% to your calculated base size to fit the context window.
*   *Example:* 32B Model × 0.5 = 16GB Base Weight. + 20% overhead = ~20GB to 22GB Required.

## 2. Bandwidth vs. Capacity
*   **Capacity (GB):** Determines if the model *can load*. If it spills over dedicated VRAM into system RAM over PCIe, generation speeds crash.
*   **Bandwidth (GB/s):** Determines *how fast* tokens generate. 
    *   *Discrete GPU:* Fast (e.g., RTX 5090 = 1,800 GB/s).
    *   *Unified Mini PC:* Slow but massive capacity (e.g., AMD Strix Halo = 256 GB/s).

## 3. Formats to Use
*   **GGUF:** Use on Apple Silicon (Metal) or when you need CPU/RAM layer offloading.
*   **AWQ / GPTQ:** Use natively on dedicated NVIDIA graphics cards to maximize CUDA acceleration.

## 4. Power Overhead (TCO)
*   When calculating your API vs Local break-even cost, do not forget idle power. 
*   A multi-GPU rig idles at 120W-150W (costing money 24/7). 
*   Always add 20%-30% to your power calculation to account for your room's HVAC system cooling the 750W+ heat output under load.
