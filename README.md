<div align="center">

```text
   ██████╗██╗    ██╗███╗   ███╗
  ██╔════╝██║    ██║████╗ ████║
  ██║     ██║ █╗ ██║██╔████╔██║
  ██║     ██║███╗██║██║╚██╔╝██║
  ╚██████╗╚███╔███╔╝██║ ╚═╝ ██║
   ╚═════╝ ╚══╝╚══╝ ╚═╝     ╚═╝

          CAUSAL WORLD MODELS
```

### From token-by-token prediction to one-step latent world simulation

**No autoregressive rollout. No KV cache. One structured transition of the entire latent world.**

</div>

---

## What is CWM?

**CWM (Causal World Models)** is an experimental neural architecture that predicts how a structured world changes instead of predicting the next token.

The model compresses an input sequence into a fixed set of latent object slots, infers directed interactions between those objects, and advances the complete latent state in a single parallel transition:

```text
World at T0
    │
    ▼
Object Encoder ──► Directed Interaction Graph ──► Latent Physics Engine
                                                        │
                                                        ▼
                                               Predicted World at T1
                                                        │
                                                        ▼
                                                 Energy Critic
```

Dense Transformer attention introduces a token-pair term that grows quadratically with sequence length. CWM replaces token-to-token global attention with linear token-to-slot coarse-graining and performs relational reasoning over a small, fixed object set.

For a fixed slot budget, one future-state prediction requires **one constant-depth latent transition**—effectively **O(1) sequential decoding steps** with respect to the number of tokens that an autoregressive model would have to emit.

> **Complexity note:** CWM does not claim literal O(1) arithmetic for arbitrary inputs. Encoding remains linear in input length, and the dense interaction graph is quadratic in the small number of object slots. The O(1) claim refers specifically to sequential decoding depth for a single latent future step.

---

## Core Advantages

| Capability | CWM approach |
|---|---|
| **One-step inference** | Predicts the complete latent successor state in one parallel transition |
| **No KV cache** | Does not retain an ever-growing token history during latent forecasting |
| **Linear token memory** | Token-side memory grows linearly with input length for a fixed object-slot budget |
| **Object-centric reasoning** | Compresses syntax into persistent macro-object slots |
| **Directed interactions** | Uses separate source and target projections for asymmetric influence vectors |
| **Latent physics** | Learns residual state changes rather than autoregressive token continuations |
| **Energy-based ranking** | Scores candidate futures by latent surprise energy; lower is more compatible |
| **Ultra-compact deployment** | The current interactive demonstration checkpoint contains approximately **600k parameters** |
| **Legacy-GPU friendly** | Demonstrated locally on an NVIDIA GeForce GTX 750 Ti |

In terms of token-sequence length (N), the encoder avoids an (N \times N) token-attention matrix. Its token-side activation memory is linear in (N) when the latent object budget is fixed. Object interaction is performed afterward on a much smaller slot graph.

---

## Blind OOD Test Results

The local blind out-of-distribution examination presented the model with three competing futures for an unseen high-pressure destruction scenario. The model assigned the lowest surprise energy to the physically directed destructive outcome.

| Candidate future | Surprise energy (MSE) | Rank | Interpretation |
|---|---:|---:|---|
| 💥 **Directed destruction** | **0.000099** | **1** | Physically structured transition selected |
| 🧊 Static freezing | 0.000151 | 2 | Identity shortcut rejected |
| 🌀 Chaotic nonsense | 0.000289 | 3 | Unstructured change rejected |

```text
E(directed destruction) < E(static freezing) < E(chaotic nonsense)
```

### Local inference latency

> **27.9 ms for one complete latent transition** on an **NVIDIA GeForce GTX 750 Ti**, a 2014-era consumer GPU.

The result is especially important because the model does not merely prefer change over stasis: it also assigns substantially higher energy to incoherent change. The learned transition therefore distinguishes **structured dynamics** from both identity collapse and arbitrary noise in the reported blind test.

These figures are preliminary local measurements from the current demonstration checkpoint. Repeated-seed statistics, standardized hardware benchmarks, and matched architectural baselines are planned for the public research release.

---

## Architecture at a Glance

1. **CWMObjectEncoder**  
   Converts tokens into a fixed tensor of latent macro-objects using local coarse-graining, competitive slot assignment, and an importance gate.

2. **CausalGraphLayer**  
   Builds a directed interaction vector for every ordered object pair through separate source and target projections.

3. **LatentPhysicsEngine**  
   Aggregates incoming and outgoing effects, predicts a latent state delta, and applies a stable residual transition.

4. **CWMCritic**  
   Measures the surprise energy between the predicted future and an encoded candidate outcome.

The current prototype is deliberately small and interpretable enough to run on low-power hardware while retaining an explicit object–relation–transition structure.

---

## Interactive Demonstration

The closed demonstration application provides:

- a built-in high-pressure reservoir grand examination;
- custom T0 world input through physics text, code-like state variables, and ASCII space;
- one-step latent prediction;
- energy comparison across destruction, static freezing, and chaotic futures;
- a human-readable verdict without exposing internal latent tensors.

The demonstration is intended for private technical presentations, research review, and investor sessions.

---

## Technical Paper

The full technical report is available in the repository root:

### [📄 CWM Technical Paper — PDF](./CWM_Technical_Paper.pdf)

The report contains the mathematical formulation of the object encoder, directed interaction layer, residual latent physics engine, energy critic, computational-complexity analysis, high-dynamics synthetic dataset, and blind OOD evaluation.

**arXiv release: coming soon.**

---

## Research Status

CWM is an active experimental research architecture. The present results demonstrate a compact non-autoregressive latent transition system on a controlled high-dynamics benchmark. They should not yet be interpreted as proof of general physical reasoning, formal causal identification, or universal superiority over Transformer-based systems.

Current research priorities include:

- byte-pair and code-aware tokenization;
- real execution traces and physical simulator data;
- contrastive negative futures and stronger anti-collapse objectives;
- sparse directed object graphs;
- multi-step latent rollouts;
- controlled comparisons with Transformer, recurrent, graph, and state-space baselines;
- intervention-based evaluation of learned directed edges.

---

## Source Availability Disclaimer

> **The CWM mathematical core is temporarily closed-source while official academic moderation, intellectual-property review, and publication preparation are completed.**

The interactive demonstration is available for controlled demonstrations. It exposes the user-facing inference workflow and energy verdicts without publishing the internal implementation of the latent transition kernel.

Source availability, reproducibility artifacts, trained checkpoints, and licensing terms will be announced after the academic review process.

---
### 🌐 Architectural Scalability & Ecosystem

The CWM (Causal World Model) core presented in this repository is designed as a foundational, high-efficiency latent engine. To demonstrate its viability for real-world artificial general intelligence (AGI) applications, we have successfully integrated this $O(1)$ latent simulation core into our private agentic operating framework, **SWILL v8.0** (an extensive infrastructure consisting of 40k+ lines of asynchronous Python code across 700+ isolated modules).

Within the SWILL ecosystem, the CWM core powers:
* **The Curiosity Engine:** Maximizing cognitive progress and dynamically adapting exploration goals based on the system's runtime stress levels.
* **The Epistemic Arena:** Enabling a decentralized swarm of agents to run parallel hypothesis validation and consensus filtering via automated $O(1)$ prediction loops, completely neutralizing autoregressive context explosion and token costs.

### 🔒 Access & Collaboration

To maintain strict IP protection, compute-cost efficiency, and align with responsible Deep Tech development, the complete codebase for the SWILL infrastructure and raw CWM weights are currently kept private. 

However, we are fully open to technical feedback, academic collaboration, and pre-seed research grants. If you are an AI researcher, an arXiv endorser (category `cs.LG`), or a strategic ecosystem partner interested in seeing a live technical review of the integrated CWM+SWILL swarm framework, please reach out directly via the public email listed on this GitHub profile. 

Let's discuss how we can completely break the Transformer bottleneck together.


<div align="center">

### CWM-Core

**Stop predicting the next token. Start predicting the next world.**

</div>
