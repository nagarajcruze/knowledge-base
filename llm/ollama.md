# Ollama & Large Language Model (LLM) Fundamentals

## 1. What is Ollama?

**Ollama** is an open-source command-line tool and local runtime service that simplifies running Large Language Models (LLMs) locally. It packages model weights, configurations, and datasets into a single unified format, managing memory allocation and hardware acceleration automatically.

- **Primary Website**: [ollama.com](https://ollama.com)
- **Model Registry (Library)**: Browse and search for available models at [ollama.com/library](https://ollama.com/library).

### Local Execution Engine: llama.cpp
Ollama is powered by **llama.cpp**, a highly optimized C/C++ engine designed to execute LLM inferences with minimal overhead. It enables hardware acceleration on CPU and GPU systems (including CUDA, ROCm, and Apple Metal) out of the box.

---

## 2. Ollama CLI Command Reference

Execute these commands in your terminal to manage and run local models.

### 1. Run a Model (`ollama run`)
Downloads (if not already local) and launches an interactive chat session with the specified model.
- **Example**:
  ```bash
  ollama run qwen2.5-coder:7b
  ```

### 2. Pull a Model (`ollama pull`)
Downloads a model from the Ollama registry to your local machine without starting an active chat session.
- **Example**:
  ```bash
  ollama pull llama3:8b
  ```

### 3. List Local Models (`ollama list`)
Displays all models currently downloaded and available on your system.
- **Example**:
  ```bash
  ollama list
  ```

### 4. Remove a Model (`ollama rm`)
Deletes a downloaded model from local storage to free up disk space.
- **Example**:
  ```bash
  ollama rm phi3:mini
  ```

### 5. Check Running Models (`ollama ps`)
Lists the models currently loaded into memory (RAM/VRAM) and running.
- **Example**:
  ```bash
  ollama ps
  ```

### 6. Show Model Information (`ollama show`)
Displays details, parameters, license, and system prompt details for a downloaded model.
- **Example**:
  ```bash
  ollama show qwen2.5-coder:7b
  ```

### 7. Create a Custom Model (`ollama create`)
Allows you to create a customized model using a `Modelfile` (similar to a Dockerfile).
1. Create a `Modelfile` in your directory:
   ```dockerfile
   FROM llama3
   # Set the temperature parameter (higher = more creative)
   PARAMETER temperature 0.8
   # Set a custom system instructions prompt
   SYSTEM "You are a specialized DevOps assistant. Keep answers technical and concise."
   ```
2. Build the model:
   ```bash
   ollama create devops-assistant -f ./Modelfile
   ```
3. Run your custom model:
   ```bash
   ollama run devops-assistant
   ```

---

## 3. Model Formats: GGUF vs. Others

Different software engines and hardware setups require different file formats for storing and loading model weights.

| Format | Execution Engine | Primary Use Case | Hardware Context | Key Benefit |
|---|---|---|---|---|
| **GGUF** (GPT-Generated Unified Format) | `llama.cpp`, Ollama, KoboldCPP | Local consumer computers & homelabs. | CPU-only, or hybrid CPU/GPU setups. | **Layer Offloading**: Can split layers between CPU RAM and GPU VRAM if model exceeds VRAM size. |
| **Safetensors** / PyTorch | Hugging Face, vLLM, TGI | Training, fine-tuning, and enterprise serving. | High-end dedicated GPUs (NVIDIA A100, H100). | Raw unquantized format. Fast loading and safe (no arbitrary code execution compared to old `.bin` pickles). |
| **AWQ** (Activation-aware Weight Quantization) | vLLM, TensorRT-LLM | High-concurrency enterprise inference. | Dedicated GPU clusters (must fit entirely in VRAM). | 4-bit quantization optimized for GPU speed. Protects key weights to maintain high accuracy. |
| **GPTQ** (Generalized Post-Training Quantization) | vLLM, AutoGPTQ | General GPU-only inference. | Dedicated GPU (must fit entirely in VRAM). | Traditional 4-bit GPU quantization method. Slightly faster than AWQ but slightly less accurate. |

### When to use which?
- Use **GGUF** if you are running models on a **laptop, desktop, or homelab server** where you need CPU/RAM offloading because your GPU's VRAM is too small.
- Use **Safetensors** if you are **fine-tuning** or have plenty of GPU VRAM to run full-precision models.
- Use **AWQ** or **GPTQ** if you are hosting models in production on **dedicated cloud GPUs** (e.g. using `vLLM` to serve a web API).

---

## 4. Model Compression & Quantization

Raw LLM weights are represented in 16-bit floating-point precision (FP16). Loading these models is extremely memory-intensive.

**Quantization** is the process of compressing models by shrinking the precision of weights from 16-bit floats to smaller representation sizes like 8-bit or 4-bit integers.

### Quantization Naming Conventions
- **`deepseek.Q4_K_M`**: Represents a model quantized to **4-bits** using the **K-medium** quantization method.

### Precision vs. Intelligence Trade-offs
- **4-bit Quantization (`Q4`)**: Reduces file size and RAM footprint, allowing models to run on standard computers. Sufficient for text summaries and general conversations.
- **8-bit Quantization (`Q8`)**: Requires more RAM/VRAM but preserves logical reasoning and syntax structure. A minimum of `Q8` is highly recommended for coding, debugging, and advanced logical reasoning tasks.

---

## 5. Homelab Benchmarks (Optiplex Server)

The following models were tested on an Optiplex homelab server:

### Installed Models
```bash
$ ollama list
NAME                  ID              SIZE      MODIFIED
qwen2.5-coder:7b      dae161e27b0e    4.7 GB    42 minutes ago
qwen3.6:35b           07d35212591f    23 GB     15 hours ago
phi3:mini             4f2222927938    2.2 GB    17 hours ago
```

### Benchmark Results & Analysis

#### 1. Qwen 2.5 Coder (7B Parameter Model)
```text
total duration:       1m6.819380299s
load duration:        98.215436ms
prompt eval count:    58 token(s)
prompt eval duration: 869.853844ms
prompt eval rate:     66.68 tokens/s
eval count:           336 token(s)
eval duration:        1m5.088886202s
eval rate:            5.16 tokens/s
```
- **Takeaway**: Fast prompt ingestion (66.68 tok/s) and a usable generation rate (5.16 tok/s). This model size is highly optimized for interactive coding help on mid-range host setups.

#### 2. Qwen 3.6 (35B Parameter Model)
```text
$ ollama run qwen3.6:35b --verbose
total duration:       46.127416918s
load duration:        197.530746ms
prompt eval count:    11 token(s)
prompt eval duration: 599.114048ms
prompt eval rate:     18.36 tokens/s
eval count:           180 token(s)
eval duration:        45.191845345s
eval rate:            3.98 tokens/s
```
- **Takeaway**: Prompt evaluation rate (18.36 tok/s) and output generation speed (3.98 tok/s) are very slow. The large 23GB footprint exceeds standard consumer resources and hits hardware memory bandwidth bottlenecks, making it slow for real-time coding or chat loops.

---

## 6. Model Classifications: SLM, LLM, and Frontier Models

| Model Category | Size (Parameters) | Architecture Focus | Use Case Examples |
|---|---|---|---|
| **Small Language Models (SLMs)** | Typically < 10B | Highly specialized, lightweight, and fast. | Document routing, text classification, simple sentiment analysis. |
| **Large Language Models (LLMs)** | 10B to 100B+ | Generalists capable of handling broad knowledge domains and complex context layouts. | Multi-source data synthesis, customer support chatbots, data extraction. |
| **Frontier Models (FMs)** | Hundreds of Billions / Multi-modal | Cutting-edge systems designed for multi-step reasoning and multi-agent workflows. | Multi-step agentic operations, debugging complex software, autonomous incident response. |

---

## 7. Reasoning Models (LRMs) vs. Pattern Matching (LLMs)

### LLMs (Large Language Models)
- **Mechanism**: Predict the next token based on statistical pattern matching.
- **Style**: Instant "reflex-based" response.
- **Best For**: Creative text generation, simple summaries, translation.

### LRMs (Large Reasoning Models)
- **Mechanism**: Go beyond token prediction by adding a "System 2" thinking phase. Before generating final text, they construct internal plans, evaluate options, search paths, and perform calculations.
- **Training**: Built on pre-trained LLMs using datasets with "chains of thought" (step-by-step reasoning solutions). They are guided via **Reinforcement Learning from Human Feedback (RLHF)** and **Process Reward Models (PRMs)** that score the validity of *each individual step* rather than just grading the final output.
- **Inference-Time Compute**: Unlike standard LLMs, LRMs spend extra processing time ("thinking") at runtime to run search algorithms or write plans.
  - *Pros*: Extreme logical accuracy, decreases the need for complex prompt engineering (like writing "think step-by-step").
  - *Cons*: High response latency, high GPU processing costs, and increased operational costs.

---

## 8. Document Ingestion: Long Context vs. Cache Augmented Generation (CAG)

Feeding massive external datasets or document directories to a model is accomplished via two main methods:

### 1. Long Context
- **Method**: Insert all source files directly into the prompt history.
- **Drawbacks**:
  - **Cost & Latency**: High token counts require expensive and slower calculations for every subsequent query.
  - **Lost in the Middle**: Models frequently fail to accurately retrieve information located in the center of long contexts.

### 2. Cache Augmented Generation (CAG)
- **Method**: Utilizes the **Key-Value (KV) Cache**—the model's internal representation of processed text. The system processes the source documents once, generates the KV cache, and saves it. Subsequent queries reuse the pre-computed KV cache.
- **Workflow**:
  1. **Knowledge Preparation**: Format documents to fit the context.
  2. **Precomputation**: The model processes the text once to build the KV cache.
  3. **Inference**: Loads the cached state, enabling up to 10x to 40x speedups.

### Prompt Caching (CAG as a Service)
Commercially available prompt caching abstracts KV cache management for developers:
- **Automation**: The cloud provider handles the lifecycle of the KV cache under the hood based on prompt prefixes.
- **Efficiency**: Identical system prompt prefixes skip the document ingestion/re-processing phase entirely.
- **Savings**: Providers commonly offer discounts of up to **90%** for cached tokens.