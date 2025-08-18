# Orange Pi 5 Plus Local LLM Deployment System

## 📦 Overview

This build transforms the Orange Pi 5 Plus (32GB LPDDR4x) into a high-performance, portable AI workstation using:

- Orange Pi Plus (32GB)
  
  <center><img width="600" height="400" alt="image" src="https://github.com/user-attachments/assets/8c982324-3f88-4e3a-b179-7ebbc08d06fd" /></center>
  <img width="608" height="501" alt="image" src="https://github.com/user-attachments/assets/742f1fa3-9da6-417d-a2b3-8d221cffab7c" />

- A bootable 8TB NVMe SSD (via USB-C)
  
  <img width="605" height="300" alt="image" src="https://github.com/user-attachments/assets/3ddba983-8d3f-4a09-a711-1d65938b1ae1" />

- Sakura-II®️ M.2 AI accelerator (installed in PCIe x4 slot)
    
  <img width="1011" height="445" alt="image" src="https://github.com/user-attachments/assets/7123ad5a-c25f-424f-85d2-d8277da0164b" />

- Local quantized LLMs (downloaded from lmstudio.ai, huggingface.co, etc. and used in the cassette-style model switching GUI interface)
- Hardware-accelerated Chromium for ChatGPT
- AnythingLLM for offline PDF/document Q&A

## 🧠 Core Goals
- Run LLMs locally with offline inference via Sakura-II®️
- Fast storage access using external USB-C NVMe SSD
- Load models into RAM and execute inference via accelerator
- Switch LLMs via GUI-style interface (LLM Cassette Loader)
- Keep OS and models on separate SSD partitions

## 🧩 Parts List

| Item | Description |
|------|-------------|
| SBC | Orange Pi 5 Plus 32GB LPDDR4x |
| AI Accelerator | Sakura-II®️ M.2 (EdgeCortix) - https://www.edgecortix.com/en/press-releases/edgecortixs-sakura-ii-ai-accelerator-brings-low-power-generative-ai-to-raspberry-pi-5-and-other-arm-based-platforms|
| SSD | 8TB NVMe M.2 (WD_BLACK SN770, connected via USB-C 3.2 Gen 2 enclosure) |
| Enclosure | NVMe SSD Type-C 10Gbps Aluminum Case - SABRENT USB 3.2 Type-C Tool-Free Enclosure for M.2 PCIe NVMe and SATA SSDs (EC-SNVE) |
| Cooling | Heatsink + Fan (for SSD and CPU), optional heatsink for Sakura-II®️ |
| Power | Geekworm PD 27W 5.1V 5A USB-C Power Supply |
| OS Boot** | Ubuntu 24.04 (bootable from USB-C NVMe SSD) |
| Display | HDMI or USB-C display connection |
| Input | USB keyboard/mouse |
| Optional | USB 3.0 hub, WiFi 6E onboard, BT 5.3, GPIO access pins |

**Recommended Image for the best GUI (Desktop) experience with the Orange Pi 5 Plus, use the **Ubuntu 24.04 LTS Desktop with Linux 6.1** image built specifically for this SBC.

OS Boot Download Link (**Size:** 1.7 GB):
https://ubuntu.com/download/server/arm
https://joshua-riek.github.io/ubuntu-rockchip-download/boards/orangepi-5-plus.html

<img width="2307" height="1101" alt="image" src="https://github.com/user-attachments/assets/fecec922-1282-4c4f-a364-b0904c804195" />

## ⚙️ Storage Layout
Steps on how to boot from the Orange Pi's USB-C:
https://github.com/sunnychase/usbc-booting-opi5plus/blob/main/README.md

| Partition | Purpose | Filesystem | Size |
|-----------|---------|------------|------|
| `/boot` | OS Bootloader | ext4 | 830.5 MB |
| `/` | Ubuntu 24.04 root | ext4 | 48.3 GB |
| `/` | quantized_llm_models| ext4 | 1.8 TB |
| `/` | full_llm_models| ext4 | 3.6 TB |
| `/` | misc| ext4 | 1.3 TB |

## 🚀 LLM Deployment & Use

### LLMs in Cart (Quantized - GGUF Q4_K_M / Q5_0):
- Yi-34B (GPTQ)
- DeepSeek-Coder-6.7B
- WizardCoder-15B
- StarCoder2-7B
- WizardMath-7B
- LLaMA 3 8B Instruct
- Phi-3-mini
- CodeLlama-7B
- Mistral 7B Instruct
- TinyLlama 1.1B
- FinGPT
- FinBERT
- ClinicalCamel (if GGUF available)
- Meditron-7B

**Cassette Switch Mechanism:** A simple GUI using AnythingLLM or GPT4All desktop selector for choosing a single active model.

## 🧠 AnythingLLM Setup

1. Install via Docker or native build on Ubuntu.
2. Set documents folder to `/llm_models/docs`.
3. Use local LLM from the selected quantized model.
4. Enable GPU execution where supported via ONNX/accelerator.
5. Configure chat embedding: pdfplumber, tabulate, SymPy.

## 🌐 Chromium GPU Acceleration Fix

Enable Vulkan + GPU compositing:

https://github.com/sunnychase/chromium-gpu/tree/main

```bash
chromium-browser --enable-features=VaapiVideoDecoder --use-gl=egl --enable-zero-copy --enable-gpu-rasterization --enable-oop-rasterization --ignore-gpu-blocklist
```

Validate GPU use:
```bash
chrome://gpu
```

Install latest `mesa`, `libva`, and V4L2-request drivers if needed.

## 📊 Performance Benchmarks

| Component | Interface | Speed (R/W MB/s) |
|-----------|-----------|------------------|
| USB-C NVMe SSD | USB 3.2 Gen 2 (10 Gbps) | 850–1000 MB/s |
| PCIe Sakura-II®️ | Gen 2 x4 (2 GB/s theoretical) | 1000–1600 TOPS peak |
| eMMC | Internal | 150–300 MB/s |
| microSD | UHS-I (best case) | 50–90 MB/s |

## 🧊 Cooling Notes

- Use heatsinks on CPU, SSD, and optionally the Sakura-II®️ (if under sustained load).

  <img width="380" height="250" alt="image" src="https://github.com/user-attachments/assets/3942134c-5ed5-44d5-8e4f-cd7b8b94cb6b" />

- Ensure airflow from side-mounted or top-down fan.

  <img width="480" height="440" alt="image" src="https://github.com/user-attachments/assets/66638faf-2db2-48dd-9567-87fe4943da2b" />

## 🔁 LLM Cassette Changer Logic

- Script to unmount any existing models from `/llm_models/active`
- Symlink selected model from `/llm_models/collection/MODEL_NAME` to `/llm_models/active`
- Load into memory and start inference server

## 🔐 Security Notes

- Disable SSH root login if exposed to external network.
- Keep LLM ports internal or password-protected.
- Use cronjob to restart system on memory overflow or GPU stall.

## 📥 Recommended Tools

- `llama.cpp`, `GPT4All`, `Ollama` for local inference
- `pdfplumber` + `tabulate` for doc Q&A
- `lmdeploy` and `text-generation-webui` for flexible GPU/CPU switching
- `glmark2-es2` to benchmark GPU (if needed)

## 🧠 System Purpose & AI Model Overview

This Orange Pi 5 Plus build is engineered as a **modular, edge-AI workstation** that enables rapid local inference, debugging, and task automation using quantized large language models (LLMs). Key use cases include:

- 🔁 **Cassette-style LLM switching** via GUI: Load one model at a time depending on task needs.
- 📄 **Offline document interpretation** (PDF, DOCX): Summarize, extract data, or reason over engineering schematics.
- 🧠 **Symbolic and mathematical problem solving**: Integrated tools like SymPy and Math LLMs for engineering, control theory, or finance.
- 🐍 **Code generation**: Python/Bash code for GPIO, I2C, RS232/485, CAN protocols and automation tools.
- 📊 **Stock/market prediction**: Run FinGPT and similar models to predict price trends, sentiment, and GEX hedging.
- 💬 **Natural language reports and procedures**: Automatically generate operational procedures, system summaries, and markdown documentation.

### ⚙️ Included Local LLMs & Their Roles

| Model | Purpose | Size | Notes |
|-------|---------|------|-------|
| **WizardCoder-15B** | High-performance code and troubleshooting assistant | 15B | GPTQ Q4_K_M |
| **DeepSeek-Coder-6.7B** | Fast Python + logic generation | 6.7B | Code workflows |
| **Mistral 7B Instruct** | General-purpose reasoning | 7B | Default fallback |
| **Yi-34B GPTQ** | High-context reasoning and summarization | 34B | Heavyweight |
| **Phi-3 Mini** | Lightweight math + logic model | 1.8B | Fastest to load |
| **FinGPT** | Financial prediction and text analysis | ~7B | Focused |
| **WizardMath-7B** | Symbolic/advanced math | 7B | Engineering logic |
| **MusicGen / AudioLDM2** | Sample-conditioned music generation | <2B | Local WAV library supported |

These models are run via:
- 🧠 `llama.cpp`, `text-generation-webui`, `GPT4All`
- 🎛️ Selected via GUI for cassette-style switching (no concurrent runs)
- 📁 Stored on USB-C NVMe SSD, loaded to RAM as needed

## 🔧 System Summary

| Component | Description |
|----------|-------------|
| **Board** | Orange Pi 5 Plus (32GB LPDDR4x RAM) |
| **OS Boot** | Ubuntu 24.04 booted via USB-C NVMe SSD |
| **AI Accelerator** | Sakura-II®️ M.2 Edge AI Accelerator |
| **Storage** | 8TB NVMe SSD (USB-C Enclosure) |
| **Cooling** | Dedicated fan and heatsink for SSD and Sakura-II®️ |
| **LLM Software Stack** | AnythingLLM, GPT4All, Hugging Face Quantized Models |
| **Browser Stack** | Chromium w/ GPU acceleration enabled for ChatGPT and web LLM tools |
| **Cassette Switch System** | Manual LLM selector with on-demand quantized GGUF model loading |
| **Quantization Format** | Q4_K_M, Q5_0, Q8_0 as needed (ggml/gguf) |

## ⚙️ Performance Benchmarks

### 🧠 Model Inference

| Model | Size | Format | Avg. Speed (tokens/sec) | Notes |
|-------|------|--------|--------------------------|-------|
| Mistral-7B Instruct | 7B | Q4_K_M | 9–12 | Smooth UI |
| DeepSeek-Coder 6.7B | 6.7B | Q4_K_M | 8–10 | Great for Python |
| Phi-3-mini | 3.8B | Q4_K_M | 18–20 | Ultra low-latency |
| Yi-34B GPTQ | 34B | Q4_GGUF (external GPU/host RAM) | 2–4 | Slow w/o offload |
| WizardCoder-15B | 15B | Q4_K_M | 4–6 | Requires full RAM use |
| TinyLlama 1.1B | 1.1B | Q4_K_M | 20–25 | For mobile tasks |

> Note: Performance depends on model quantization, batch size, and RAM allocation. Phi-3-mini and TinyLlama perform best for latency-critical tasks.

### 📈 Storage I/O Performance

| Interface | Avg. Read Speed | Avg. Write Speed | Notes |
|-----------|------------------|-------------------|-------|
| **USB-C NVMe (Gen 2)** | 900–1050 MB/s | 850–950 MB/s | Optimal for LLM loads |
| **PCIe NVMe (x4)** | 1800–2200 MB/s | 1700–2000 MB/s | Used for accelerator |
| **eMMC** | 180–200 MB/s | 100–120 MB/s | Only used if USB fails |
| **microSD (UHS-I)** | 70–90 MB/s | 50–70 MB/s | Avoid for OS/LLM loading |

### 🖥️ System Use Cases

- 🔌 Local LLM chat (offline, private, uncensored)
- 📁 PDF and DOCX document QA, parsing, summarization
- 🧠 Coding assistant (Python, Bash, circuit scripts)
- 📊 Report generation and formatting (Markdown, LaTeX)
- 🔁 AI model swapping using "cassette change" system
- 🌐 Browser-based LLM via Chromium GPU acceleration

### 🧪 Stability and Heat Management

- CPU load averages: ~30–45% while idle w/ background services
- Load temps (w/ fan): 48–55°C (CPU), 50–58°C (Sakura-II®️), 45–52°C (SSD)
- No throttling detected with current heatsink + airflow setup

## ⚙️ Power Supply Specs
- **Model**: Geekworm PD 27W USB-C
- **Output**: 5.1V ⎓ 5A
- **Max Wattage**: 25.5W continuous (stable under load)

## 📦 Components and Estimated Power Draw

| Component                        | Voltage | Typical Current | Power (Watts) | Notes |
|----------------------------------|---------|------------------|----------------|-------|
| Orange Pi 5 Plus (32GB LPDDR4X) | 5.1V    | 2.5–3.5A         | 13–18W         | Under load (AI tasks) |
| NVMe SSD (8TB via USB-C)        | 5.1V    | 0.9–1.2A         | 4.5–6W         | Depends on SSD controller |
| Sakura-II®️ M.2 AI Accelerator    | 5.1V    | 1.0–1.5A         | 5–7.5W         | Inference mode |
| USB Peripherals (keyboard/mouse)| 5.1V    | 0.1–0.2A         | 0.5–1W         | Varies |
| Fan + Heatsink (Active Cooling) | 5.1V    | 0.15A            | ~0.75W         | For NVMe/Sakura cooling |

## 🔋 Total System Power Budget (Typical Load)
- **Estimated Max Draw**: ~26–30W
- **Notes**:
    - Slightly over spec under full load. Short bursts are okay.
    - Recommend active cooling and quality PD charger with heat protection.

## 🛡️ Power Recommendations
- For **sustained inference workloads**, consider a 5.1V/6A+ or 9V/3A USB-C PD.
- Monitor temps and current via inline USB meter.
- Avoid cheap cables or thin wires that cause voltage drop.

## 🧠 Performance Notes
- Power usage spikes when:
    - Loading LLMs into RAM (via AnythingLLM)
    - Running multi-threaded tasks across CPU and Sakura-II®️
- LLM caching to SSD increases NVMe power briefly.

## ✅ Recommended Workflows

- Use `llm-launcher.sh` to select LLM model before session.
- Access `AnythingLLM` via `localhost:3000` or remote IP.
- Load PDF/DOCX via drag & drop in GUI or auto ingestion folder.
- Monitor temp and resource stats using `htop` + `sensors`.
- Chromium flags enabled for `--enable-gpu-rasterization` + `--use-vulkan`

## 🧾 Final Notes

This Orange Pi 5 Plus system offers desktop-level generative AI performance with support for high-throughput SSD, quantized LLMs, and dedicated AI acceleration via Sakura-II®️. Ideal for offline use, research, coding, creative writing, and system troubleshooting.

## ©️ License
Sunny Chase, All Rights Reserved 2025
