---
published: false
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

### 1.1 Hardware baseline

| Component | Configuration |
|---|---|
| System model | `[TO COMPLETE]` |
| CPU | Qualcomm Snapdragon X Elite X1E80100 |
| CPU cores | 12 |
| Architecture | ARM64 |
| Memory | 64 GB LPDDR5 |
| Memory speed | 8448 MT/s |
| GPU | Qualcomm Adreno X1-85 |
| NPU | Qualcomm Hexagon |
| Storage | Approximately 1 TB NVMe SSD |
| Power mode | `[Battery / Balanced / Best performance]` |
| Power source during tests | `[Battery / AC]` |

### 1.2 Software baseline

| Component | Version |
|---|---|
| Windows edition | `[TO COMPLETE]` |
| Windows version | `[TO COMPLETE]` |
| OS build | `[TO COMPLETE]` |
| Runtime | `[TO COMPLETE]` |
| Runtime version | `[TO COMPLETE]` |
| GPU driver | `[TO COMPLETE]` |
| NPU driver | `[TO COMPLETE]` |

### 1.3 Baseline screenshots

- `01-cpu-baseline.png`
- `02-memory-baseline.png`
- `03-storage-baseline.png`
- `04-gpu-baseline.png`
- `05-npu-baseline.png`

```html
<!-- Replace PLACEHOLDER with the correct image and alt text. -->
<a href="{{ '/assets/images/local-llm/PLACEHOLDER.png' | relative_url }}"
   class="article-image-link"
   data-lightbox
   aria-label="Open test environment screenshot">
  <img src="{{ '/assets/images/local-llm/PLACEHOLDER.png' | relative_url }}"
       alt="PLACEHOLDER"
       class="article-image">
</a>
```

### Test record

- **Date and time:** `[TO COMPLETE]`
- **Battery or AC:** `[TO COMPLETE]`
- **Windows power mode:** `[TO COMPLETE]`
- **Applications running:** `[TO COMPLETE]`
- **Available memory before testing:** `[TO COMPLETE]`
- **Notes:** `[TO COMPLETE]`

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
