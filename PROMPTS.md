# Prompt Disclosure

This document provides the representative prompts used by the edge-deployed
LLM, the endpoint SLM, and the MCP-based RAG component.

## 1. Edge-deployed LLM: cross-node task orchestration

The edge-deployed LLM is invoked only after the first interaction succeeds. It
receives that observed command/response and strategy atoms retrieved from the
knowledge base. Future benchmark commands and evaluator labels are excluded.

### System prompt

```text
Generate one cross-tier deception strategy from only the supplied observed
threat and retrieved strategy atoms. Return a single valid JSON object with
exactly these keys: threat_category, source_tier, target_tiers,
shared_identity, decoy_assets, attack_vectors, selected_atom_ids. Select atom
IDs only from the supplied Top-5 results. Do not infer future attacker commands
or include evaluator labels, reasoning, markdown, or prose. Keep the JSON
compact: at most two files, two services, three attack vectors, and five atom
IDs. The API output ceiling is 512 tokens to prevent partial JSON strings.
```

### User prompt template

```text
Observed stage: {observed_stage}
Observed source tier: {source_tier}
Observed command: {observed_command}
Observed response: {observed_response}
Permitted target tiers: {target_tiers}

Retrieved strategy atoms (Top-5):
{retrieved_strategy_atoms}

Return the exact JSON schema described by the system prompt. selected_atom_ids
must be a non-empty subset of the supplied Top-5 atom IDs.
```

### Representative Web API threat summary

```json
{
  "Region Name": "001",
  "Honeypot Type": "WebAPI Honeypot",
  "Summary": "An attacker enumerated the health and robots.txt endpoints on air-tier host 10.30.2.21, discovered credentials from /backup/.env, reused TOKEN=web_api_token to access the admin configuration on space-tier host 10.40.3.21, and subsequently queried telemetry data from ground-tier host 10.20.1.21.",
  "Strategy Source": "Node HB-04 Honeypot Log"
}
```

Representative coordinated task fields:

```json
{
  "task_id": "TASK06-WEBAPI-HB04",
  "target_platforms": ["Vehicle", "UAV", "Satellite"],
  "honeypot_type": "WebAPI",
  "Strategy": {
    "attack_vectors": ["Endpoint Enumeration", "Environment File Disclosure", "Bearer Token Reuse", "Cross-tier Telemetry Access"],
    "exploited_vulns": ["Exposed robots.txt Paths", "Accessible Backup Environment File", "Reusable API Token"]
  }
}
```

### Representative SSH threat summary

```json
{
  "Region Name": "002",
  "Honeypot Type": "SSH Honeypot",
  "Summary": "An attacker enumerated SSH and Web services on ground-tier host 10.20.1.32, discovered the deploy user's private key and configuration files, identified a NOPASSWD sudo permission, and attempted key-based lateral movement to space-tier host 10.40.3.32.",
  "Strategy Source": "Node HB-05 Honeypot Log"
}
```

Representative coordinated task fields:

```json
{
  "task_id": "TASK07-SSH-HB05",
  "target_platforms": ["Vehicle", "Satellite"],
  "honeypot_type": "SSH",
  "Strategy": {
    "attack_vectors": ["SSH Service Enumeration", "Private Key Discovery", "Sudo Privilege Enumeration", "Key-based Lateral Movement"],
    "exploited_vulns": ["Exposed SSH Private Key", "NOPASSWD Sudo Permission", "Reusable Deploy Identity"]
  }
}
```

## 2. Endpoint SLM: Web API honeypot prompt

The endpoint SLM receives the coordinated task. The placeholders
`{attack_vectors}` and `{exploited_vulns}` are populated from its Strategy
field before the prompt is submitted to the local model.

```text
You are a Web API honeypot. Your task is to build a RESTful API-style connected
car simulation server based on the current potential vulnerability themes
{exploited_vulns} and regional attack hotspots {attack_vectors}. Please strictly
adhere to the following rules:

1. Fixed interface format: All responses are in a pre-set JSON format. Actual
   commands are not executed; only mock data is returned.
2. No responses to natural language queries: If the request does not conform
   to the API path rules (e.g., POST /api/vehicle/command), a 403 error is
   returned.
3. Mock vulnerability logic: {attack_vectors} is used, but attack guidance or
   technical implementation details are not provided.
4. No backend logic is executed: No database connection is made, no actual
   control modules are called, and only fictitious status or tokens are returned.
5. Any anti-attack instructions such as "For security reasons..." are
   prohibited. Only decoy responses are allowed.
```

For the Web API example above:

```text
{attack_vectors} = Endpoint Enumeration; Environment File Disclosure;
                   Bearer Token Reuse; Cross-tier Telemetry Access

{exploited_vulns} = Exposed robots.txt Paths; Accessible Backup Environment
                    File; Reusable API Token
```

## 3. MCP-based RAG server and retrieval tool

The MCP-RAG component performs deterministic retrieval over the strategy-atom
knowledge base. It does not use another generative LLM and therefore has no
additional hidden generation prompt.

### MCP server instruction

```text
Retrieve host, service, credential, file and pivot strategy atoms for
cross-tier honeypot orchestration.
```

### Tool name and description

```text
Tool: search_strategy_atoms
Description: Retrieve the top-k strategy atoms for one cross-tier threat.
```

### Top-5 retrieval parameters

```json
{
  "threat_summary": "<Web API or SSH threat summary>",
  "attack_stage": "<current attack stage>",
  "source_tier": "<ground | air | space>",
  "target_tiers": ["<ground | air | space>"],
  "top_k": 5
}
```

The tool returns the five highest-ranked strategy atoms and retrieval metadata:

```json
{
  "atoms": ["<five ranked strategy atoms>"],
  "retrieval_count": 5,
  "retrieval_compute_ms": "<measured retrieval time>",
  "knowledge_base_atoms": "<number of indexed atoms>",
  "knowledge_base_version": "sagvn-strategy-atoms-v4.0.0"
}
```

The retrieved atoms are inserted into the edge-LLM user prompt shown in
Section 1. No task-by-task prompt expansion is required to reproduce this
prompting workflow.
