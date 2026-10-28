# 面向空天地一体化车联网的大小模型协同威胁诱骗

**本仓库整理空天地一体化车联网（Space–Air–Ground Integrated Vehicular Networks，SAGVNs）大小模型协同诱骗研究的实验代码、配置和结果。**

**模型文件说明：** 由于微调模型文件总大小超过 40 GB，`LLM and SLM/` 目录未包含在本公开仓库中。研究所需的相关微调模型文件可通过邮件联系作者咨询：**[hshen@njtech.edu.cn](mailto:hshen@njtech.edu.cn)**。

## 项目简介

本项目面向地面、空中和空间异构网络中的协同威胁诱骗场景，结合**边缘侧大语言模型（LLM）推理与任务编排、本地小语言模型（SLM）蜜罐响应生成，以及共享策略原子知识库**。边缘 LLM 从知识库检索并组织相关策略原子，将其转化为跨层协同任务；端侧 SLM 利用下发任务中的策略语义生成与协议、环境一致的诱骗响应，**不要求每个端侧节点直接访问知识库**。经过验证的威胁知识可进一步支持策略原子的更新与复用。

完整研究框架还包括联盟链支持的跨域协调、证据溯源，以及针对威胁捕获者和策略协作者的双角色激励机制。**本 README 仅依据现有目录说明列出相应代码和实验资料；不能据此认定完整框架的所有模块均已包含在待公开的代码中。**

### 研究流程概览

```text
蜜罐交互记录 / 威胁摘要
          |
          v
    共享策略原子知识库
    （BM25 / MCP 检索）
          |
          v
 边缘 LLM：跨层威胁推理与任务编排
          |
      协同任务策略语义
          |
          v
 端侧 SLM：协议与环境适配的诱骗响应
          |
          v
      交互日志与实验评估
          |
     可信知识更新 / 激励分析
```

上图为**功能层面**的研究流程，不意味着所有组件均已整合为单一可执行程序。若后续发布智能合约、证据恢复模块或 Oracle 相关代码，应另行注明对应目录和使用方法。

## 仓库目录

以下结构以当前提供的目录清单为依据，不同发布版本的实际文件可能有所差异。

```text

├── AdvancedshelLM/             # Manager–Worker 对比方法及适配代码
├── LLMHoneypot/
│   ├── codebase/
│   │   └── LLM_Plugins/         # Manager 逻辑与角色配置
│   ├── honeypot_exp/           # 统一实验框架与评估程序
│   ├── incentive_common.py     # 信誉与激励计算公共模块
│   └── incentive_redesign/     # 激励实验输出
├── Experimental results/       # 日志、汇总数据、绘图脚本和结果图
└── PROMPTS.md                  # 代表性提示词及 MCP 检索接口说明
```

### 1. `LLMHoneypot/`：核心实验框架

`LLMHoneypot/honeypot_exp/` 包含统一的实验入口和核心模块：

| 文件或目录 | 主要用途 |
| --- | --- |
| `main.py`、`config.py` | 实验入口及运行参数配置 |
| `tasks.py` | 基准任务和攻击阶段判定规则 |
| `experiment.py`、`model.py` | 会话执行及模型请求 |
| `reporter.py`、`baseline1.py` | 结果导出及静态回放基线 |
| `advanced_shellm_adapter.py` | AdvancedShelLM 对比实验适配层 |
| `strategy_atoms.json` | 策略原子知识库 |
| `strategy_atoms_manifest.json` | 知识库版本、分布及哈希元数据 |
| `strategy_retrieval.py` | 基于 BM25 的策略原子检索 |
| `mcp_rag_server.py` | MCP 检索服务 |
| `kb_sensitivity/` | 知识库可用性敏感性实验输入 |
| `build_*.py`、`make_*.py` | 知识库构建和规模变体生成 |
| `run_kb_sensitivity.ps1` | 知识库敏感性实验 PowerShell 入口 |
| `validate_strategy_atoms.py` | 策略原子知识库校验 |
| `rescore_checkpoint.py` | 已保存实验记录的离线重新评分 |
| `benchmark_*.py` | 时延与开销专项测量 |
| `requirements*.txt` | 各类实验所需依赖 |

其他相关组件包括 `LLMHoneypot/codebase/LLM_Plugins/` 下的 `manager_agent.py`、`personalities/`，以及 `LLMHoneypot/incentive_common.py` 和 `LLMHoneypot/incentive_redesign/`。

### 2. `AdvancedshelLM/`：Manager–Worker

该目录保存 Manager–Worker 协同方式的适配、提示词与评估代码：

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

其中，**Worker** 生成模拟终端响应；**Manager** 提供角色和场景指导，并检查或修正响应一致性。适配层为不同网络层级维护会话和文件系统状态。

**注意：** 该目录**不能单独运行**。其中的入口依赖 `model.py`、`tasks.py`、`strategy_retrieval.py` 等模块，这些模块列于 `LLMHoneypot/honeypot_exp/`。有关上游项目请参考 [shelLM / AdvancedShelLM](https://github.com/stratosphereips/shelLM/tree/main/AdvancedshelLM)。公开第三方代码前需核对其许可证并保留必要的来源说明。

### 3. `Experimental results/`：实验数据与图表

| 文件 | 内容 |
| --- | --- |
| `checkpoint.jsonl` | Baseline-2/3/4 与 Proposed-2/5 的实验记录 |
| `summary.csv` | 上述方法的指标汇总 |
| `session.csv` | Baseline-2/3 和 Proposed-2/5 的逐任务会话记录 |
| `baseline1_session.csv` | 静态基线会话记录 |
| `node_behavior_per_round.csv` | 逐轮节点信誉及奖励数据 |
| `Knowledge-bise size.csv` | 知识库规模/可用性实验汇总（保留现有文件名） |
| `task1.xlsx`～`task4.xlsx` | 时延图对应的数据表 |
| `fig7ab.json`、`fig7ab.py`、`fig8ab.py` | 模型及诱骗效果图的数据与绘图程序 |
| `fig9a.py`～`fig9d.py` | Token 开销和性能对比绘图 |
| `fig10abc.py`、`fig11.py` | 信誉与激励机制绘图 |
| `fig12a.py`～`fig12d.py` | 区块链/各层节点时延绘图 |
| `plot_kb_size_depth.py`、`plot_kb_size_duration.py` | 知识库敏感性绘图 |
| `fig*.pdf`、`fig*.png` | 已导出的图表 |
| `Completed attack stages.pdf`、`Session duration.pdf` | 知识库可用性实验结果图 |

**不同结果文件覆盖的方法集合并不完全相同。** 例如，不能假定 `summary.csv` 或 `session.csv` 包含所有基线的逐任务数据。

### 4. `PROMPTS.md`：提示词与检索接口

该文件说明以下代表性配置：

- **边缘 LLM 任务编排：** 系统提示词、输入模板、Web API/SSH 威胁摘要示例。
- **端侧 SLM 响应生成：** Web API 蜜罐角色、响应约束，以及 `attack_vectors`、`exploited_vulns` 等模板字段。
- **MCP 检索：** 服务端说明、`search_strategy_atoms` 工具、Top-5 检索参数和返回元数据。

### 5. 微调模型文件（未上传）

本研究使用的 `LLM and SLM/` 目录原本包含以下微调模型文件：

- `Llama-2-7B-lora/`
- `Llama-3-8B-lora/`
- `Qwen3-4B-lora/`
- `Qwen3-8B-lora/`

**由于上述模型文件总大小超过 40 GB，本仓库不提供 `LLM and SLM/` 目录及其中的模型权重。** 实验代码、配置说明与结果文件仍按照本 README 所列目录组织。**如需获取上述微调模型文件，请联系作者：[hshen@njtech.edu.cn](mailto:hshen@njtech.edu.cn)**，咨询可用的获取方式。请同时遵守相关基础模型的使用许可。

## 实验方法与对比设置

| 方法 | 实验配置 |
| --- | --- |
| Baseline-1 | Cowrie/Beelzebub 风格静态回放；在该实现中不产生 LLM token |
| Baseline-2 | GPT-4o 模型蜜罐 |
| Baseline-3 | 归档的 DeepSeek-V4-Flash 基线 |
| Baseline-4 / AdvancedShelLM | GPT-5.1 驱动的 Manager–Worker 多智能体对比方法 |
| Proposed-2 | Qwen3-8B，无跨层协调 |
| Proposed-5 | 以 Qwen3-8B 为基础、结合策略检索与边缘任务指导的协同配置 |

**现有 README 对 Proposed-5 的具体实验流程描述为：** 先利用 Qwen3-8B 的初始交互获得命令/响应上下文，再执行确定性 Top-5 策略检索、一次配置为 GPT-5.1 的边缘编排调用，随后生成经过协调的 Qwen3-8B 响应。这是**目录说明中给出的实验配置**，不代表总体研究架构的所有部署都采用 GPT-5.1。

本研究还使用 Qwen3-4B、Llama-2-7B 和 Llama-3-8B 等微调变体；相关模型文件未在仓库中上传，其实验结果是否包含在特定 CSV 汇总中，应以对应数据文件为准。

## 环境配置与复现说明

### 依赖安装

请根据选定的实验类型配置 Python 环境，并使用 `LLMHoneypot/honeypot_exp/` 下相应的 `requirements*.txt` 安装依赖。具体运行参数以 `main.py`、`config.py` 为准。**运行涉及微调 SLM 的实验还需要相应模型文件；这些文件未随仓库上传，获取方式见上文“微调模型文件（未上传）”。** 仅依据当前目录介绍，尚无法确认整个系统能够通过单条命令完整复现。

### 模型服务与凭据

代码目录说明使用环境变量配置模型接口。例如，在 PowerShell 中：

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

以上均为**配置示例，并非可公开调用的真实接口地址**。现有目录文档记录了部分实验使用 PAI-EAS 路由 Qwen、通过远程 GPT-5.1 服务进行边缘编排；另有本地 SLM 推理环境使用 **vLLM（OpenAI-compatible API）和 2 × NVIDIA GeForce RTX 4090 GPU**。由于服务提供商、推理环境和时延测量边界不同，复现实验时需逐次记录实际的 `model.py`/`config.py` 配置、模型版本、推理服务和硬件条件，不能将 PAI-EAS 与 vLLM 结果直接视为同一测量条件。

### 建议的复现顺序

1. 检查 `LLMHoneypot/honeypot_exp/config.py`，选择方法、任务及模型接口。
2. 使用 `validate_strategy_atoms.py` 检查知识库，并核对 manifest；变更知识库前保存对应版本。
3. 根据 `honeypot_exp/main.py` 支持的参数执行实验；如需使用 AdvancedShelLM，确保共享依赖模块及上游依赖已配置。
4. 对知识库可用性实验，检查 `kb_sensitivity/` 和 `run_kb_sensitivity.ps1`，分别运行 120/90/60/30/5 个原子的配置。
5. 将输出与 `Experimental results/` 中对应的 CSV/JSONL 比对，并使用绘图脚本生成图表。

如条件允许，应随结果记录随机种子、任务集合、原子子集选取方式、重复次数、模型提供方和各项指标定义。
