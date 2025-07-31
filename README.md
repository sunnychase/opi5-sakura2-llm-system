# Orange Pi 5 Plus Local LLM Deployment System

## 📦 Overview

This build transforms the Orange Pi 5 Plus (32GB LPDDR4x) into a high-performance, portable AI workstation using:
- A bootable 8TB NVMe SSD (via USB-C)
- Sakura-II M.2 AI accelerator (installed in PCIe x4 slot)
- Local quantized LLMs (cassette-style model switching)
- Hardware-accelerated Chromium for ChatGPT
- AnythingLLM for offline PDF/document Q&A

## 🧠 Core Goals
- Run LLMs locally with offline inference via Sakura-II
- Fast storage access using external USB-C NVMe SSD
- Load models into RAM and execute inference via accelerator
- Switch LLMs via GUI-style interface (LLM Cassette Loader)
- Keep OS and models on separate SSD partitions

---

## 🧩 Parts List

| Item | Description |
|------|-------------|
| SBC | Orange Pi 5 Plus 32GB LPDDR4x |
| AI Accelerator | Sakura-II M.2 (EdgeCortix) |
| SSD | 8TB NVMe M.2 (WD_BLACK SN770, connected via USB-C 3.2 Gen 2 enclosure) |
| Enclosure | NVMe SSD Type-C 10Gbps Aluminum Case (shown in image) |
| Cooling | Heatsink + Fan (for SSD and CPU), optional heatsink for Sakura-II |
| Power | 5V/5A USB-C PD adapter with PD support |
| OS Boot | Ubuntu 24.04 (bootable from USB-C NVMe SSD) |
| Display | HDMI or USB-C display connection |
| Input | USB keyboard/mouse |
| Optional | USB 3.0 hub, WiFi 6E onboard, BT 5.3, GPIO access pins |

---

## ⚙️ Storage Layout

| Partition | Purpose | Filesystem | Size |
|-----------|---------|------------|------|
| `/boot` | OS Bootloader | ext4 | 1 GB |
| `/` | Ubuntu 24.04 root | ext4 | 60 GB |
| `/llm_models` | Quantized LLMs | ext4 | Remaining (~7.9 TB) |

---

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

---

## 🧠 AnythingLLM Setup

1. Install via Docker or native build on Ubuntu.
2. Set documents folder to `/llm_models/docs`.
3. Use local LLM from the selected quantized model.
4. Enable GPU execution where supported via ONNX/accelerator.
5. Configure chat embedding: pdfplumber, tabulate, SymPy.

---

## 🌐 Chromium GPU Acceleration Fix

Enable Vulkan + GPU compositing:

```bash
chromium-browser --enable-features=VaapiVideoDecoder --use-gl=egl --enable-zero-copy --enable-gpu-rasterization --enable-oop-rasterization --ignore-gpu-blocklist
```

Validate GPU use:
```bash
chrome://gpu
```

Install latest `mesa`, `libva`, and V4L2-request drivers if needed.

---

## 📊 Performance Benchmarks

| Component | Interface | Speed (R/W MB/s) |
|-----------|-----------|------------------|
| USB-C NVMe SSD | USB 3.2 Gen 2 (10 Gbps) | 850–1000 MB/s |
| PCIe Sakura-II | Gen 2 x4 (2 GB/s theoretical) | 1000–1600 TOPS peak |
| eMMC | Internal | 150–300 MB/s |
| microSD | UHS-I (best case) | 50–90 MB/s |

---

## 🧊 Cooling Notes

- Use heatsinks on CPU, SSD, and optionally the Sakura-II (if under sustained load).
- Ensure airflow from side-mounted or top-down fan.

---

## 🔁 LLM Cassette Changer Logic

- Script to unmount any existing models from `/llm_models/active`
- Symlink selected model from `/llm_models/collection/MODEL_NAME` to `/llm_models/active`
- Load into memory and start inference server

---

## 🔐 Security Notes

- Disable SSH root login if exposed to external network.
- Keep LLM ports internal or password-protected.
- Use cronjob to restart system on memory overflow or GPU stall.

---

## 📥 Recommended Tools

- `llama.cpp`, `GPT4All`, `Ollama` for local inference
- `pdfplumber` + `tabulate` for doc Q&A
- `lmdeploy` and `text-generation-webui` for flexible GPU/CPU switching
- `glmark2-es2` to benchmark GPU (if needed)
