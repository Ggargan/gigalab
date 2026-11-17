---
layout: post
title: "Running a Local LLM on a Snapdragon X Elite Windows PC"
date: 2026-11-17
categories: [ai, homelab]
tags: [local-llm, windows-arm64, snapdragon-x-elite, qualcomm, ollama, llama-cpp]
description: "A hands-on lab to evaluate local LLM inference on Windows ARM64, including model compatibility, performance, memory usage, GPU/NPU utilization and offline operation."
---

# Running a Local LLM on a Snapdragon X Elite Windows PC

> **Article status:** Work in progress  
> **Last test completed:** Not started  
> **Current conclusion:** Pending validation

Running a Large Language Model locally is increasingly accessible, but most practical guidance assumes an x86 computer equipped with an NVIDIA GPU.

This lab explores a different platform: a Windows on ARM laptop powered by Qualcomm Snapdragon X Elite, with 64 GB of shared system memory, an integrated Adreno GPU and a Hexagon NPU.

The goal is not simply to prove that a model can start. The objective is to determine which configurations are genuinely usable and which hardware accelerators are actually used during inference.

## Lab objectives

- Verify runtime compatibility with Windows ARM64.
- Install and validate a local LLM runtime.
- Run models from different parameter classes.
- Record model size, load time, memory use and generation speed.
- Observe CPU, GPU and NPU utilization.
- Test the local API.
- Confirm whether inference continues without Internet connectivity.
- Separate documented support from observed behavior.

---

## 1. Test environment

Before installing an LLM runtime or downloading a model, we captured a hardware baseline of the test system. This baseline will later allow us to compare CPU, memory, GPU and NPU utilization during model loading and inference.

### 1.1 Hardware overview

The test system is a Microsoft Surface Laptop, 7th Edition based on the Qualcomm Snapdragon X Elite platform. It combines a 12-core ARM64 processor, 64 GB of LPDDR5 memory, an integrated Qualcomm Adreno GPU and a dedicated Qualcomm Hexagon NPU.

<a href="{{ '/assets/images/local-llm/01-hardware-overview.png' | relative_url }}"
   class="article-image-link"
   data-lightbox
   aria-label="Open Snapdragon X Elite hardware overview screenshot">
  <img src="{{ '/assets/images/local-llm/01-hardware-overview.png' | relative_url }}"
       alt="Windows System About page showing the Snapdragon X Elite processor, 64 GB RAM, Qualcomm Adreno GPU and NVMe storage"
       class="article-image">
</a>

*Figure 1 - Hardware overview of the Snapdragon X Elite test system.*

| Component | Configuration |
|---|---|
| Device | Microsoft Surface Laptop, 7th Edition |
| CPU | Qualcomm Snapdragon X Elite X1E80100 |
| CPU cores | 12 |
| Architecture | ARM64 |
| Memory | 64 GB LPDDR5 |
| Memory speed | 8448 MT/s |
| GPU | Qualcomm Adreno X1-85 |
| NPU | Qualcomm Hexagon |
| Storage | Approximately 1 TB NVMe SSD |
| Operating system | Windows 11 ARM64 |

### 1.2 CPU baseline

The Snapdragon X Elite X1E80100 exposes 12 cores and 12 logical processors to Windows. Before starting the inference tests, CPU utilization was captured to establish an idle baseline.

<a href="{{ '/assets/images/local-llm/02-cpu-baseline.png' | relative_url }}"
   class="article-image-link"
   data-lightbox
   aria-label="Open Snapdragon X Elite CPU baseline screenshot">
  <img src="{{ '/assets/images/local-llm/02-cpu-baseline.png' | relative_url }}"
       alt="Windows Task Manager CPU view showing the 12-core Snapdragon X Elite X1E80100 processor before LLM inference"
       class="article-image">
</a>

*Figure 2 - Snapdragon X Elite CPU baseline before loading a local LLM.*

During the baseline capture, Windows reported:

- 12 cores;
- 12 logical processors;
- a base frequency of approximately 3.42 GHz;
- virtualization enabled.

The same Task Manager view will be captured while an LLM is actively generating tokens.

### 1.3 Memory baseline

Memory capacity is particularly important for local LLM inference, especially for quantized models that would not normally fit into the dedicated VRAM of a consumer GPU. The test system contains 64 GB of LPDDR5 memory running at 8448 MT/s.

<a href="{{ '/assets/images/local-llm/03-memory-baseline.png' | relative_url }}"
   class="article-image-link"
   data-lightbox
   aria-label="Open system memory baseline screenshot">
  <img src="{{ '/assets/images/local-llm/03-memory-baseline.png' | relative_url }}"
       alt="Windows Task Manager showing 64 GB LPDDR5 memory running at 8448 MT/s before local LLM testing"
       class="article-image">
</a>

*Figure 3 - System memory baseline before loading the LLM runtime.*

This screenshot will later be compared with memory consumption while:

1. the runtime is idle;
2. a model is loaded;
3. the model is actively generating tokens.

### 1.4 Storage baseline

The system uses an NVMe SSD with approximately 954 GB of usable capacity. Local models can consume several gigabytes or tens of gigabytes depending on parameter count and quantization, so storage capacity is also relevant to the lab.

<a href="{{ '/assets/images/local-llm/04-storage-baseline.png' | relative_url }}"
   class="article-image-link"
   data-lightbox
   aria-label="Open NVMe storage baseline screenshot">
  <img src="{{ '/assets/images/local-llm/04-storage-baseline.png' | relative_url }}"
       alt="Windows Task Manager storage view showing the NVMe SSD used to store local LLM models"
       class="article-image">
</a>

*Figure 4 - NVMe storage used for the local LLM environment.*

The actual disk footprint of each downloaded model will be recorded later in the experiment.

### 1.5 Qualcomm Adreno GPU baseline

The Snapdragon X Elite platform includes an integrated Qualcomm Adreno X1-85 GPU. Unlike a traditional discrete GPU with dedicated VRAM, Windows exposes shared system memory to the integrated GPU.

<a href="{{ '/assets/images/local-llm/05-gpu-baseline.png' | relative_url }}"
   class="article-image-link"
   data-lightbox
   aria-label="Open Qualcomm Adreno GPU baseline screenshot">
  <img src="{{ '/assets/images/local-llm/05-gpu-baseline.png' | relative_url }}"
       alt="Windows Task Manager showing Qualcomm Adreno X1-85 GPU utilization and shared GPU memory before LLM inference"
       class="article-image">
</a>

*Figure 5 - Qualcomm Adreno X1-85 GPU baseline before LLM inference.*

Windows reports up to approximately 47.5 GB of shared GPU memory. This does not by itself mean that an LLM runtime can use that memory efficiently or that inference will automatically be accelerated by the Adreno GPU. GPU acceleration will therefore be treated as an experimental result rather than an assumption.

### 1.6 Qualcomm Hexagon NPU baseline

The Snapdragon X Elite also includes a dedicated Qualcomm Hexagon NPU. The presence of an NPU does not automatically mean that a generic LLM runtime can use it, so NPU utilization was captured before running any local model.

<a href="{{ '/assets/images/local-llm/06-npu-baseline.png' | relative_url }}"
   class="article-image-link"
   data-lightbox
   aria-label="Open Qualcomm Hexagon NPU baseline screenshot">
  <img src="{{ '/assets/images/local-llm/06-npu-baseline.png' | relative_url }}"
       alt="Windows Task Manager showing Qualcomm Snapdragon X Elite NPU utilization before local LLM inference"
       class="article-image">
</a>

*Figure 6 - Qualcomm Hexagon NPU baseline before starting local LLM inference.*

At baseline, Task Manager reported essentially no active NPU compute workload. During inference, the same view will be captured again and compared with this baseline.

Task Manager activity alone will not be considered definitive proof of NPU offloading. Where possible, runtime logs or backend diagnostics will also be used to identify the hardware performing inference.

### 1.7 Software and ARM64 baseline

The Windows System About page confirms that the laptop is running a 64-bit operating system on an ARM-based processor. The complete system-information screenshot has been retained separately and will be added to this section as `07-system-info.png` after the Windows build information has been finalized for publication.

| Component | Version |
|---|---|
| Windows edition | Windows 11 Enterprise |
| Windows version | 26H2 |
| System type | 64-bit operating system, ARM-based processor |
| OS build | `[TO COMPLETE]` |
| Runtime | `[TO COMPLETE]` |
| Runtime version | `[TO COMPLETE]` |
| GPU driver | 31.0.133.1 |
| NPU driver | 30.0.220.3000 |

### 1.8 Baseline test record

| Test condition | Value |
|---|---|
| Date and time | `[TO COMPLETE]` |
| Power source | `[Battery / AC]` |
| Windows power mode | `[TO COMPLETE]` |
| Applications running | `[TO COMPLETE]` |
| Available memory before testing | `[TO COMPLETE]` |
| Notes | `[TO COMPLETE]` |

### 1.9 Why capture a baseline?

These screenshots establish the **before** state of the experiment. The same measurements will be repeated during sustained inference.

| Resource | Baseline | During inference |
|---|---|---|
| CPU | Figure 2 | To be measured |
| Memory | Figure 3 | To be measured |
| Adreno GPU | Figure 5 | To be measured |
| Hexagon NPU | Figure 6 | To be measured |

This provides a consistent way to distinguish between hardware that is merely available to Windows and hardware that is actually exercised by the selected LLM runtime.

---

## 2. Runtime selection

### 2.1 Candidates

| Runtime | Windows ARM64 | CPU | Adreno GPU | Hexagon NPU | Local API | Outcome |
|---|---|---|---|---|---|---|
| Ollama | `[VERIFY]` | `[VERIFY]` | `[VERIFY]` | `[VERIFY]` | `[VERIFY]` | `[PENDING]` |
| llama.cpp | `[VERIFY]` | `[VERIFY]` | `[VERIFY]` | `[VERIFY]` | `[VERIFY]` | `[PENDING]` |
| ONNX Runtime / GenAI | `[VERIFY]` | `[VERIFY]` | `[VERIFY]` | `[VERIFY]` | `[VERIFY]` | `[PENDING]` |
| Qualcomm AI runtime | `[VERIFY]` | `[VERIFY]` | `[VERIFY]` | `[VERIFY]` | `[VERIFY]` | `[PENDING]` |

### 2.2 Selection criteria

- Native or supported Windows ARM64 execution.
- Quantized model support.
- Repeatable installation.
- Local command-line interface.
- Local API availability.
- Observable hardware acceleration.
- Active maintenance and adequate documentation.

### Decision record

- **Selected runtime:** `[TO COMPLETE]`
- **Selected version:** `[TO COMPLETE]`
- **Reason for selection:** `[TO COMPLETE]`
- **Known limitations:** `[TO COMPLETE]`
- **Official references:** `[ADD LINKS]`

> **Evidence classification**  
> **Documented:** `[TO COMPLETE]`  
> **Observed:** `[TO COMPLETE]`  
> **Not verified:** `[TO COMPLETE]`

---

## 3. Runtime installation

### 3.1 Installation procedure

```powershell
# Insert the exact installation commands used during the lab.
```

### 3.2 Installation validation

```powershell
# Insert the command used to display the runtime version or status.
```

Expected or observed output:

```text
[PASTE SANITIZED OUTPUT]
```

### Installation record

| Check | Result |
|---|---|
| Native ARM64 package | `[YES / NO / UNKNOWN]` |
| Installation successful | `[YES / NO]` |
| Restart required | `[YES / NO]` |
| Runtime service created | `[YES / NO / N/A]` |
| Local port | `[TO COMPLETE]` |
| Installation footprint | `[TO COMPLETE]` |
| Issues encountered | `[TO COMPLETE]` |

### Screenshot

- `06-runtime-installation.png`

---

## 4. Model selection

### 4.1 Candidate models

| Test ID | Model | Parameters | Quantization | Download size | Expected memory | Status |
|---|---|---:|---|---:|---:|---|
| T01 | `[TO COMPLETE]` | 7B/8B | `[TO COMPLETE]` | `[TO COMPLETE]` | `[TO COMPLETE]` | Planned |
| T02 | `[TO COMPLETE]` | 14B | `[TO COMPLETE]` | `[TO COMPLETE]` | `[TO COMPLETE]` | Planned |
| T03 | `[TO COMPLETE]` | 30B/32B | `[TO COMPLETE]` | `[TO COMPLETE]` | `[TO COMPLETE]` | Optional |

### 4.2 Model download

```powershell
# Insert the exact model download command.
```

### Model record

- **Model repository/source:** `[TO COMPLETE]`
- **Model license:** `[TO COMPLETE]`
- **Model identifier:** `[TO COMPLETE]`
- **Quantization:** `[TO COMPLETE]`
- **On-disk size:** `[TO COMPLETE]`
- **Download time:** `[TO COMPLETE]`
- **Model checksum, if recorded:** `[TO COMPLETE]`

---

## 5. Test methodology

To make the comparison repeatable, each model should be tested under the same conditions.

### 5.1 Test conditions

- Connect the laptop to AC power.
- Select the same Windows power mode.
- Close nonessential applications.
- Record available memory before each test.
- Execute the same prompts.
- Repeat each measurement at least three times when practical.
- Distinguish cold model load from warm inference.

### 5.2 Standard prompts

#### Prompt A: concise technical explanation

```text
Explain the difference between a virtual machine and a container in five concise bullet points.
```

#### Prompt B: longer generation

```text
Write a technically accurate introduction to a guide about building a site-to-site VPN between Azure and a UniFi gateway. Use approximately 500 words and do not invent configuration values.
```

#### Prompt C: code generation

```text
Create a PowerShell script that reports the operating system, processor architecture, total memory and available memory. Include error handling and explanatory comments.
```

#### Prompt D: Italian-language test

```text
Spiega in italiano, con un esempio pratico, la differenza tra inferenza locale, GPU acceleration e NPU acceleration.
```

### 5.3 Measurement method

- **Load time:** `[METHOD/COMMAND]`
- **Time to first token:** `[METHOD/COMMAND]`
- **Generation speed:** `[METHOD/COMMAND]`
- **Memory consumption:** Windows Task Manager / `[OTHER TOOL]`
- **CPU utilization:** Windows Task Manager / `[OTHER TOOL]`
- **GPU utilization:** Windows Task Manager / `[OTHER TOOL]`
- **NPU utilization:** Windows Task Manager / `[OTHER TOOL]`

---

## 6. Test execution records

Copy this subsection once for every model tested.

### Test `[T01]` - `[MODEL NAME]`

#### Configuration

| Setting | Value |
|---|---|
| Runtime and version | `[TO COMPLETE]` |
| Model | `[TO COMPLETE]` |
| Parameters | `[TO COMPLETE]` |
| Quantization | `[TO COMPLETE]` |
| Context length | `[TO COMPLETE]` |
| Thread count | `[TO COMPLETE]` |
| Accelerator/backend | `[TO COMPLETE]` |
| Command line | `[TO COMPLETE]` |

#### Commands

```powershell
# Exact command used to start the model.
```

#### Results

| Metric | Run 1 | Run 2 | Run 3 | Average |
|---|---:|---:|---:|---:|
| Cold load time | `[ ]` | `[ ]` | `[ ]` | `[ ]` |
| Time to first token | `[ ]` | `[ ]` | `[ ]` | `[ ]` |
| Generation speed, tokens/s | `[ ]` | `[ ]` | `[ ]` | `[ ]` |
| Peak RAM usage | `[ ]` | `[ ]` | `[ ]` | `[ ]` |
| Peak CPU utilization | `[ ]` | `[ ]` | `[ ]` | `[ ]` |
| Peak GPU utilization | `[ ]` | `[ ]` | `[ ]` | `[ ]` |
| Peak NPU utilization | `[ ]` | `[ ]` | `[ ]` | `[ ]` |

#### Qualitative assessment

- **Output accuracy:** `[TO COMPLETE]`
- **Instruction following:** `[TO COMPLETE]`
- **Italian-language quality:** `[TO COMPLETE]`
- **Responsiveness:** `[TO COMPLETE]`
- **Thermal/noise observations:** `[TO COMPLETE]`
- **Errors or instability:** `[TO COMPLETE]`

#### Evidence

- `07-t01-terminal-output.png`
- `08-t01-resource-utilization.png`
- **Sanitized output:** `[LINK OR CODE BLOCK]`

#### Test conclusion

> `[SUPPORTED CONCLUSION BASED ONLY ON THE RECORDED EVIDENCE]`

---

## 7. GPU and NPU verification

Task Manager activity alone can indicate utilization, but it does not always prove which operations were offloaded. Conclusions in this section should therefore identify the strength of the evidence.

### 7.1 Adreno GPU

- **Runtime documentation states:** `[TO COMPLETE]`
- **Configured backend:** `[TO COMPLETE]`
- **Task Manager observation:** `[TO COMPLETE]`
- **Runtime logs/diagnostics:** `[TO COMPLETE]`
- **CPU-only comparison:** `[TO COMPLETE]`
- **Conclusion:** `[CONFIRMED / INDICATED / NOT OBSERVED / NOT VERIFIED]`

### 7.2 Hexagon NPU

- **Runtime documentation states:** `[TO COMPLETE]`
- **Configured backend:** `[TO COMPLETE]`
- **Task Manager observation:** `[TO COMPLETE]`
- **Runtime logs/diagnostics:** `[TO COMPLETE]`
- **Comparison with NPU disabled, if possible:** `[TO COMPLETE]`
- **Conclusion:** `[CONFIRMED / INDICATED / NOT OBSERVED / NOT VERIFIED]`

---

## 8. Model comparison

| Model | Quantization | Disk size | Peak RAM | Load time | Tokens/s | CPU | GPU | NPU | Practical assessment |
|---|---|---:|---:|---:|---:|---:|---:|---:|---|
| `[MODEL 1]` | `[ ]` | `[ ]` | `[ ]` | `[ ]` | `[ ]` | `[ ]` | `[ ]` | `[ ]` | `[ ]` |
| `[MODEL 2]` | `[ ]` | `[ ]` | `[ ]` | `[ ]` | `[ ]` | `[ ]` | `[ ]` | `[ ]` | `[ ]` |
| `[MODEL 3]` | `[ ]` | `[ ]` | `[ ]` | `[ ]` | `[ ]` | `[ ]` | `[ ]` | `[ ]` | `[ ]` |

### Recommended configuration

- **Best interactive model:** `[TO COMPLETE]`
- **Best quality model:** `[TO COMPLETE]`
- **Best speed/quality compromise:** `[TO COMPLETE]`
- **Configuration to avoid:** `[TO COMPLETE]`
- **Reasoning:** `[TO COMPLETE]`

---

## 9. Local API test

### Request

```powershell
# Insert the exact local API request.
```

### Response

```json
{
  "result": "[SANITIZED RESPONSE PLACEHOLDER]"
}
```

### API record

| Check | Result |
|---|---|
| API reachable on localhost | `[YES / NO]` |
| Authentication enabled | `[YES / NO / N/A]` |
| Interfaces bound | `[TO COMPLETE]` |
| Average response time | `[TO COMPLETE]` |
| Remote LAN access enabled | `[YES / NO]` |

> Do not expose the local API beyond the trusted network without authentication, access controls and an explicit security design.

---

## 10. Offline inference test

### Procedure

1. Confirm that the runtime and model are already installed.
2. Record the initial network state.
3. Disconnect Wi-Fi and any wired network connection.
4. Start or restart the local runtime.
5. Submit the standard prompts.
6. Confirm whether responses are generated successfully.
7. Reconnect the network after completing the test.

### Result

- **Runtime started offline:** `[YES / NO]`
- **Model loaded offline:** `[YES / NO]`
- **Inference completed offline:** `[YES / NO]`
- **Unexpected network dependency:** `[TO COMPLETE]`
- **Conclusion:** `[TO COMPLETE]`

### Evidence

- `09-offline-inference.png`

---

## 11. Problems and troubleshooting

| Issue | Symptom | Cause | Resolution | Status |
|---|---|---|---|---|
| `[TO COMPLETE]` | `[ ]` | `[ ]` | `[ ]` | `[Resolved/Open]` |

### Useful diagnostic commands

```powershell
# Add only commands that were actually used and validated.
```

---

## 12. Final conclusions

### What worked

- `[TO COMPLETE]`

### What did not work

- `[TO COMPLETE]`

### Documented versus observed

| Capability | Vendor/runtime documentation | Lab observation | Final status |
|---|---|---|---|
| Windows ARM64 runtime | `[ ]` | `[ ]` | `[ ]` |
| CPU inference | `[ ]` | `[ ]` | `[ ]` |
| Adreno acceleration | `[ ]` | `[ ]` | `[ ]` |
| Hexagon NPU acceleration | `[ ]` | `[ ]` | `[ ]` |
| Offline inference | `[ ]` | `[ ]` | `[ ]` |
| Local API | `[ ]` | `[ ]` | `[ ]` |

### Final recommendation

> `[ADD THE FINAL PRACTICAL RECOMMENDATION AFTER ALL TESTS]`

---

## 13. Next steps

Possible follow-up labs:

- Add a local Web UI.
- Build a local RAG pipeline.
- Index selected GigaLab articles and technical runbooks.
- Compare native Windows execution with WSL where technically relevant.
- Evaluate power consumption and battery impact.
- Integrate the local API with PowerShell or a small application.

---

## References

Add only sources that were actually used and preserve direct links to the original documentation.

1. `[Runtime official documentation]`
2. `[Model card and license]`
3. `[Qualcomm platform/runtime documentation]`
4. `[Microsoft Windows ARM64 or AI documentation]`

---

*GigaLab - Hands-on cloud, hybrid infrastructure, networking, security and real-world troubleshooting.*
