# Large–Small Model Cooperative Threat Deception in Space–Air–Ground Integrated Vehicular Networks

**This repository collects experimental code, configurations, and results for research on large–small model cooperative threat deception in space–air–ground integrated vehicular networks (SAGVNs).**

**Model files:** The fine-tuned model files exceed 40 GB in total, so the `LLM and SLM/` directory is **not included** in this public repository. Researchers who need the corresponding fine-tuned model files may contact the author at **[hshen@njtech.edu.cn](mailto:hshen@njtech.edu.cn)** to inquire about availability.

## Project Overview

This project investigates coordinated threat deception across heterogeneous ground, air, and space networks. It combines **edge-side large language model (LLM) reasoning and task orchestration, local small language model (SLM) honeypot response generation, and a shared strategy-atom knowledge base**. The edge-side LLM retrieves and organizes relevant strategy atoms from the knowledge base and translates them into coordinated cross-tier tasks. On-device SLMs use the strategy semantics conveyed by those tasks to generate protocol- and environment-consistent deceptive responses; **individual endpoint nodes are not required to access the knowledge base directly**. Validated threat knowledge can subsequently support the refinement and reuse of strategy atoms.

The broader research framework also includes consortium-blockchain-supported cross-domain coordination and evidence provenance, as well as a dual-role incentive mechanism for threat captors and strategy collaborators. **This README describes the code and experimental materials identified in the available directory inventory; it should not be taken as confirmation that every module of the full framework is included in the public release.**

### Research Workflow at a Glance

```text
Honeypot interaction logs / threat summaries
                    |
                    v
       Shared strategy-atom knowledge base
              (BM25 / MCP retrieval)
                    |
                    v
      Edge-side LLM: cross-tier threat
          reasoning and task orchestration
                    |
             Coordinated task semantics
                    |
                    v
       Endpoint SLMs: protocol- and
     environment-adapted decoy responses
                    |
                    v
           Interaction logs and
           experimental evaluation
                    |
          Validated knowledge updates /
                incentive analysis
```

This diagram represents the **functional workflow** and does not imply that all components have been integrated into a single executable program. If smart contracts, evidence recovery modules, or Oracle-related code are released later, their directories and usage instructions should be documented separately.

## Repository Structure

The following structure is based on the current directory inventory. Actual contents may vary between releases.

```text
├── AdvancedshelLM/             # Manager–Worker baseline and adaptation code
├── LLMHoneypot/
│   ├── codebase/
│   │   └── LLM_Plugins/         # Manager logic and role configurations
│   ├── honeypot_exp/           # Unified experiment and evaluation framework
│   ├── incentive_common.py     # Shared reputation/incentive computations
│   └── incentive_redesign/     # Incentive experiment outputs
├── Experimental results/       # Logs, summary data, plotting scripts, and figures
└── PROMPTS.md                  # Representative prompts and MCP retrieval interface
```

### 1. `LLMHoneypot/`: Core Experimental Framework

`LLMHoneypot/honeypot_exp/` contains the unified experiment entry points and core modules:

| File or directory | Purpose |
| --- | --- |
| `main.py`, `config.py` | Experiment entry points and runtime configuration |
| `tasks.py` | Benchmark tasks and attack-stage evaluation rules |
| `experiment.py`, `model.py` | Session execution and model requests |
| `reporter.py`, `baseline1.py` | Result export and static replay baseline |
| `advanced_shellm_adapter.py` | Adapter for the AdvancedShelLM baseline |
| `strategy_atoms.json` | Strategy-atom knowledge base |
| `strategy_atoms_manifest.json` | Knowledge-base version, distribution, and hash metadata |
| `strategy_retrieval.py` | BM25-based strategy-atom retrieval |
| `mcp_rag_server.py` | MCP retrieval service |
| `kb_sensitivity/` | Inputs for knowledge-base availability sensitivity experiments |
| `build_*.py`, `make_*.py` | Knowledge-base construction and size-variant generation |
| `run_kb_sensitivity.ps1` | PowerShell entry point for knowledge-base sensitivity experiments |
| `validate_strategy_atoms.py` | Strategy-atom knowledge-base validation |
| `rescore_checkpoint.py` | Offline rescoring of saved experiment records |
| `benchmark_*.py` | Dedicated latency and overhead measurements |
| `requirements*.txt` | Dependencies for the various experiments |

Other related components include `manager_agent.py` and `personalities/` under `LLMHoneypot/codebase/LLM_Plugins/`, as well as `LLMHoneypot/incentive_common.py` and `LLMHoneypot/incentive_redesign/`.

### 2. `AdvancedshelLM/`: Manager–Worker Baseline

This directory contains adaptation, prompting, and evaluation code for the Manager–Worker cooperative approach:

```text
AdvancedshelLM/
├── main.py
├── config.py
├── experiment.py
├── advanced_shellm_adapter.py
├── manager_agent.py
├── baseline.yml
└── reporter.py
```

The **Worker** generates simulated terminal responses, while the **Manager** provides role and scenario guidance and checks or revises response consistency. The adapter maintains session and filesystem states for different network tiers.

**Note:** This directory **is not standalone**. Its entry points depend on modules such as `model.py`, `tasks.py`, and `strategy_retrieval.py`, which are listed under `LLMHoneypot/honeypot_exp/`. For the upstream project, see [shelLM / AdvancedShelLM](https://github.com/stratosphereips/shelLM/tree/main/AdvancedshelLM). Before redistributing third-party code, review the relevant license and retain required attribution.

### 3. `Experimental results/`: Data and Figures

| File | Contents |
| --- | --- |
| `checkpoint.jsonl` | Experiment records for Baselines 2/3/4 and Proposed-2/5 |
| `summary.csv` | Summary metrics for the above methods |
| `session.csv` | Per-task session records for Baselines 2/3 and Proposed-2/5 |
| `baseline1_session.csv` | Static-baseline session records |
| `node_behavior_per_round.csv` | Per-round node reputation and reward data |
| `Knowledge-bise size.csv` | Knowledge-base size/availability results (original filename retained) |
| `task1.xlsx`–`task4.xlsx` | Data tables for the latency figures |
| `fig7ab.json`, `fig7ab.py`, `fig8ab.py` | Data and plotting code for model and deception-effectiveness figures |
| `fig9a.py`–`fig9d.py` | Token-cost and performance-comparison plots |
| `fig10abc.py`, `fig11.py` | Reputation and incentive-mechanism plots |
| `fig12a.py`–`fig12d.py` | Blockchain/network-tier latency plots |
| `plot_kb_size_depth.py`, `plot_kb_size_duration.py` | Knowledge-base sensitivity plots |
| `fig*.pdf`, `fig*.png` | Exported figures |
| `Completed attack stages.pdf`, `Session duration.pdf` | Results of knowledge-base availability experiments |

**Different result files cover different subsets of the evaluated methods.** For example, `summary.csv` and `session.csv` should not be assumed to contain per-task records for every baseline.

### 4. `PROMPTS.md`: Prompts and Retrieval Interface

This file documents representative configurations for:

- **Edge-side LLM task orchestration:** System prompts, input templates, and Web API/SSH threat-summary examples.
- **Endpoint SLM response generation:** Web API honeypot roles, response constraints, and template fields such as `attack_vectors` and `exploited_vulns`.
- **MCP retrieval:** Server instructions, the `search_strategy_atoms` tool, Top-5 retrieval parameters, and response metadata.

### 5. Fine-Tuned Model Files (Not Uploaded)

The original `LLM and SLM/` directory contained the following fine-tuned model files:

- `Llama-2-7B-lora/`
- `Llama-3-8B-lora/`
- `Qwen3-4B-lora/`
- `Qwen3-8B-lora/`

**Because these model files exceed 40 GB in total, this repository does not include the `LLM and SLM/` directory or its model weights.** The experimental code, configuration notes, and result files are organized as listed above. **For inquiries about obtaining the fine-tuned model files, please contact the author at [hshen@njtech.edu.cn](mailto:hshen@njtech.edu.cn).** Any use or distribution of the models must also comply with the respective base-model licenses.

## Experimental Methods and Baseline Configurations

| Method | Experimental configuration |
| --- | --- |
| Baseline-1 | Cowrie/Beelzebub-style static replay; no LLM tokens are generated in this implementation |
| Baseline-2 | GPT-4o-based honeypot |
| Baseline-3 | Archived DeepSeek-V4-Flash baseline |
| Baseline-4 / AdvancedShelLM | GPT-5.1-powered Manager–Worker multi-agent baseline |
| Proposed-2 | Qwen3-8B without cross-tier coordination |
| Proposed-5 | Coordinated Qwen3-8B configuration with strategy retrieval and edge-side task guidance |

**The experimental workflow documented in the existing directory notes for Proposed-5 is as follows:** An initial Qwen3-8B interaction provides command/response context, followed by deterministic Top-5 strategy retrieval, a single edge-side orchestration call configured to use GPT-5.1, and subsequent coordinated Qwen3-8B response generation. This reflects **the configuration documented in the directory notes**, not a claim that every deployment of the overall research architecture uses GPT-5.1.

The research also uses fine-tuned variants of Qwen3-4B, Llama-2-7B, and Llama-3-8B. Their model files are not uploaded to this repository. Whether their evaluation results appear in a particular CSV summary should be checked against the corresponding data file.

## Environment Setup and Reproduction Notes

### Dependencies

Set up a Python environment for the selected experiment type and install the appropriate dependencies using the corresponding `requirements*.txt` files under `LLMHoneypot/honeypot_exp/`. Consult `main.py` and `config.py` for supported runtime parameters. **Experiments involving fine-tuned SLMs also require the corresponding model files, which are not included in this repository; see “Fine-Tuned Model Files (Not Uploaded)” above for contact information.** The current directory inventory alone does not establish that the entire system can be reproduced with a single command.

### Model Services and Credentials

The code directory notes describe environment-variable-based model service configuration. For example, in PowerShell:

```powershell
$env:OPENAI_API_KEY="<your-key>"
$env:DEEPSEEK_API_KEY="<your-key>"
$env:QWEN_EAS_TOKEN="<your-token>"
$env:ADVANCED_SHELLM_ROOT="<path-to-shelLM>/AdvancedshelLM"
$env:ADVANCED_SHELLM_BASE_URL="<your-compatible-API-endpoint>"
$env:ADVANCED_SHELLM_MODEL_NAME="gpt-5.1"
$env:QWEN_PROVIDER="pai-eas"
$env:QWEN_BASE_URL="<your-Qwen-API-endpoint>/v1"
$env:QWEN_MODEL_NAME="Qwen3-8B"
$env:QWEN_API_KEY_ENV="QWEN_EAS_TOKEN"
$env:QWEN_AUTHORIZATION_SCHEME="raw"
$env:EDGE_BASE_URL="<your-edge-LLM-API-endpoint>"
$env:EDGE_MODEL_NAME="gpt-5.1"
```

These are **configuration examples, not working public API endpoints**. The existing directory documentation records that some experiments routed Qwen through PAI-EAS and used a remote GPT-5.1 service for edge-side orchestration. A separate local SLM inference setup used **vLLM (OpenAI-compatible API) and two NVIDIA GeForce RTX 4090 GPUs**. Because model providers, inference environments, and latency measurement boundaries vary, reproduction attempts should record the actual `model.py`/`config.py` settings, model versions, inference services, and hardware conditions for each run. PAI-EAS and vLLM results should not be treated as measurements under identical conditions.

### Suggested Reproduction Workflow

1. Inspect `LLMHoneypot/honeypot_exp/config.py` to select the method, tasks, and model endpoint.
2. Use `validate_strategy_atoms.py` to check the knowledge base and its manifest; preserve the relevant version before modifying the knowledge base.
3. Run the experiment using parameters supported by `honeypot_exp/main.py`. For AdvancedShelLM, ensure that the shared dependency modules and the upstream dependencies are configured.
4. For knowledge-base availability experiments, inspect `kb_sensitivity/` and `run_kb_sensitivity.ps1`, then run the 120/90/60/30/5-atom configurations.
5. Compare the outputs with the corresponding CSV/JSONL files in `Experimental results/` and regenerate plots using the supplied plotting scripts.

Where feasible, record random seeds, task sets, atom-subset selection procedures, repetition counts, model providers, and metric definitions alongside the results. 
