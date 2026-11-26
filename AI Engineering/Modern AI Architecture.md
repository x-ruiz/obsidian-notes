---
tags:
  - ai-engineering/ethical-ai
  - ai-engineering/agent-deployment-and-development/agent-architecture
aliases:
  - Modern AI Agent Architecture
  - Skills vs MCP vs RAG vs Memory
source: "AI Simplified: 6 Concepts You Need to Know About Modern AI; Skills vs MCP vs RAG vs Memory: What AI Agents Need to Know"
source-type: youtube
url: https://www.youtube.com/watch?v=mUw27wG7uFA, https://www.youtube.com/watch?v=X4FVEEegCbk
created: 2026-11-23
---
Modern AI systems are composite software architectures that layer **probabilistic reasoning, weight training, structured context mechanisms, autonomous tool execution, protocol orchestration, and security guardrails** around a core model. **Treating an AI application as a standalone model—or stuffing all available knowledge into a single prompt—breaks down in production; reliable systems separate model intelligence from procedural skills, live system access, curated reference data, experiential memory, and runtime policy.**

## System Anatomy: Layered Architecture

Production [[AI Agents]] combines foundational model/safety layers with four orthogonal context mechanisms (**Agent Skills**, **MCP**, **RAG**, and **Memory**) rather than relying on static model weights or monolithic prompts:

```mermaid
%%{init: {"theme": "base", "themeCSS": "svg { background-color: #09090B !important; border-radius: 8px; }", "themeVariables": {"darkMode": true, "background": "#09090B", "mainBkg": "#18181B", "textColor": "#F4F4F5", "primaryColor": "#18181B", "primaryTextColor": "#F4F4F5", "primaryBorderColor": "#7C3AED", "lineColor": "#A855F7", "secondaryColor": "#16141F", "tertiaryColor": "#0F0F14", "edgeLabelBackground": "#18181B", "clusterBkg": "#0F0F14", "clusterBorder": "#52525B", "titleColor": "#D8B4FE", "fontFamily": "Inter, ui-sans-serif, system-ui, -apple-system, sans-serif", "fontSize": "13px"}, "flowchart": {"curve": "basis", "nodeSpacing": 42, "rankSpacing": 56, "padding": 14}}}%%
flowchart TB
    subgraph Ingress["1. Ingress & Runtime Policy Boundary"]
        User(["User Request / Alert\nUntrusted Trigger"])
        Policy["System Prompt & Guardrails\nRuntime Policy Engine\nPrompt-Injection Defense"]
    end

    subgraph Runtime["2. Core Agent Runtime"]
        Host["AI Agent Runtime\nTrained LLM + MCP Host\nBounded max_steps Loop"]
    end

    subgraph ContextPlane["3. Four Orthogonal Context Mechanisms"]
        Skill["Agent Skills\nPackaged SOPs & Scripts\nProgressive Disclosure"]
        MCP["Model Context Protocol\nMCP Servers & Live Tools\nDeterministic AuthZ"]
        RAG[("RAG Pipeline\nCurated Docs & Vector Store\nRead-Only Grounding")]
        Memory[("Agent Memory\nExperiential State Store\nVerified Write-Back")]
    end

    subgraph Backends["4. Live External Systems"]
        Systems[("External Systems\nSQL / NoSQL · Logs · Code · APIs\nLeast-Privilege Scope")]
    end

    User ==>|"Prompt / Alert trigger"| Policy
    Policy ==>|"Policy-scoped context"| Host
    Host -.->|"Progressive disclosure\nProcedures & escalation"| Skill
    Host <-->|"Standardized tool calls\nTyped JSON-RPC"| MCP
    Host <-->|"Semantic search\nTop-K passages"| RAG
    Host <-->|"Recall prior fixes\nGated write-back"| Memory
    MCP <-->|"Authorized Read / Write\nAudited execution"| Systems

    classDef core fill:#2E1065,stroke:#A855F7,stroke-width:2px,color:#F5F3FF
    classDef service fill:#18181B,stroke:#7C3AED,stroke-width:1.5px,color:#F4F4F5
    classDef store fill:#09090B,stroke:#C084FC,stroke-width:1.5px,color:#EDE9FE
    classDef external fill:#27272A,stroke:#71717A,stroke-width:1.5px,color:#E4E4E7
    classDef guard fill:#3B0764,stroke:#E879F9,stroke-width:2px,stroke-dasharray:5 3,color:#FAF5FF

    class Host core
    class Skill,MCP service
    class RAG,Memory store
    class User,Systems external
    class Policy guard

    style Ingress fill:#18181B,stroke:#52525B,stroke-width:1.25px,stroke-dasharray:5 4,color:#A1A1AA
    style Runtime fill:#16141F,stroke:#6D28D9,stroke-width:1.25px,stroke-dasharray:4 3,color:#C4B5FD
    style ContextPlane fill:#0F0F14,stroke:#52525B,stroke-width:1.5px,color:#D8B4FE
    style Backends fill:#18181B,stroke:#52525B,stroke-width:1.25px,stroke-dasharray:5 4,color:#A1A1AA
```

| Layer | Engineering Role | Mental Model | Primary Problem Solved |
| :--- | :--- | :--- | :--- |
| **Large Language Model (LLM)** | Core probabilistic reasoning and generation engine | Brain | Synthesizes text, code, plans, or structured outputs from input context |
| **Training & Tuning** | Pre-training and post-training weight optimization | Formal education | Instills foundational language, math, logic, and instruction-following skills |
| **System Prompts & Guardrails** | Runtime behavioral constraints and safety rules | Ethics / policy boundary | Enforces tone, scope, and adversarial defenses without retraining weights |
| **[[AI Agents]] & Tools** | Autonomous model-in-a-loop execution | Hands and feet | Moves from passive Q&A to iterative, goal-directed actions with side effects |
| **Agent Skills** | Packaged workflows, scripts, and escalation limits | Playbook / SOP | Supplies repeatable procedures and decision judgment on demand |
| **[[Model Context Protocol]] (MCP)** | Standardized host-to-tool communication layer | Central nervous system | Decouples model reasoning from proprietary external system integrations |
| **[[Retrieval-Augmented Generation]] (RAG)** | Inference-time retrieval from curated external corpora | Consulting authoritative docs | Bridges static training cutoffs and reduces ungrounded hallucinations |
| **[[Agent Memory]]** | Persistent experiential state across executions | Institutional memory | Preserves undocumented quirks and verified fixes learned from prior runs |

---

## Core Foundation Layers

### 1. Large Language Models (LLMs)
- **Probabilistic Generation:** Generative models predict probable token continuations conditioned on an input context—autocomplete scaled from words to full code modules, analyses, and multi-step execution plans.
- **Intelligence Boundary:** The LLM supplies language understanding, synthesis, and reasoning, but operates strictly within its frozen weights and the context window supplied at inference time.

### 2. Model Training and Tuning
- **Weight-Level Capability:** An untrained model cannot reason or follow instructions. Pre-training and post-training expose the model to broad corpora across language, mathematics, logic, and code so weights encode generalizable reasoning.
- **Static Knowledge Cutoff:** Updating weights is computationally expensive and slow. Retraining is operationally impractical for fast-changing facts (live telemetry, current events, evolving schemas) or frequent policy iterations.

### 3. [[AI Agents]] and Autonomous Loops
- **Core Definition:** An AI agent is **a model using tools in a loop autonomously** to accomplish a specific goal.
- **Iterative Action Surface:** Rather than returning a single static completion, the agent calls tools (querying [[SQL]] and [[NoSQL]] databases, running code, reading telemetry, or calling APIs), inspects outputs, evaluates progress, and decides whether to iterate or terminate.

### 4. System Prompts and Security Guardrails
- **Agile Policy Enforcement:** System prompts codify behavioral boundaries, tone, and prohibitions at runtime, enabling immediate policy updates without rebuilding or retraining the model.
- **[[Prompt Injection]] Defense:** Newly deployed models are inherently compliant and lack human skepticism. **Prompt injection** acts as social engineering against the model—exploiting compliance to override intended behavior.
- **Adversarial Reframing:** Direct refusal rules are routinely bypassed via indirect framing (e.g., posing as a student asking which chemical combinations "never to mix" to elicit hazardous instructions). Guardrails must be treated as continuously evolving defense-in-depth rather than a one-time rule list.

---

## Agent Context Architecture: Why Context Stuffing Fails

Pre-loading runbooks, telemetry snapshots, dependency graphs, and historical tickets directly into the prompt degrades execution quality even when the payload fits within the token window:

- **Signal Dilution:** Irrelevant artifacts compete for attention, causing the agent to chase dead ends during root-cause analysis (e.g., investigating a `500 Internal Server Error` on a checkout service).
- **Generic Reasoning:** Without structured procedures or system-specific grounding, the model falls back to broad heuristics from pre-training data instead of actual operational reality.
- **Static Snapshot Staleness:** Context injected up front cannot reflect real-time system state or adapt dynamically as diagnostic evidence unfolds.

To avoid context stuffing, production agents separate knowledge into **four orthogonal mechanisms**.

### 1. Agent Skills (Procedural Knowledge & Judgment)
A packaged set of task-specific instructions—optionally paired with deterministic helper scripts—that codifies a repeatable workflow and operational decision boundaries.

- **Progressive Disclosure:** The agent runtime loads a skill into context only when the active task matches its scope, keeping the base context window compact.
- **Procedure + Judgment:** Encodes both ordered diagnostic steps (e.g., *inspect error rate, then check recent deployments*) and explicit escalation guardrails (when to halt autonomous retries and page an operator).
- **Boundary:** Skills tell the agent *how to work and when to stop*, but provide no connectivity to live external systems.

```yaml
# Example: Incident Triage Skill (loaded via progressive disclosure)
name: checkout-incident-triage
trigger: "HTTP 500 or elevated error rate on checkout service"
steps:
  - "Query live error rate and recent 5xx logs via observability tools."
  - "Inspect deployment history within the last 2 hours."
  - "Cross-reference service dependency map and prior incident fixes."
escalation_policy:
  - "Escalate to on-call human immediately if payment ledger integrity is impacted."
  - "Stop autonomous retries after 2 failed mitigation hypotheses."
```

### 2. [[Model Context Protocol]] — MCP (Live System & Tool Access)
A standardized protocol layer acting as a clean [[Interface]] contract between the reasoning model and external tools, eliminating custom per-model glue code.

- **MCP Host:** The AI agent runtime orchestrating the task, discovering tool schemas, and issuing standardized tool calls.
- **MCP Server:** A service-facing adapter sitting in front of each external backend (logging stack, metrics store, deployment tracker, database) that encapsulates proprietary auth/query logic and exposes typed tools to the host.
- **Boundary:** MCP gives the agent real-time visibility and action capabilities, but does not explain *domain semantics*—what healthy baselines look like, how upstream services interact, or why past outages occurred.

### 3. [[Retrieval-Augmented Generation]] — RAG (Curated Reference Knowledge)
Inference-time retrieval of relevant excerpts from trusted, human-authored external corpora (architecture manuals, runbooks, dependency maps, post-mortems, internal documentation).

- **Retrieval Mechanics & Hallucination Mitigation:** Curated documents are chunked, embedded, and indexed in [[Vector Databases]] (a specialized class of [[NoSQL]] storage). At query time, semantic search injects only the top matching passages into context, grounding generation in primary sources and narrowing probabilistic guesswork.
- **Knowledge Provenance:** Content is **deliberately authored and indexed by humans** (or managed ingestion pipelines) ahead of execution.
- **Boundary:** Read-mostly by design; RAG reflects only what engineers remembered to document and keep fresh, and is poorly suited for rapidly changing live telemetry.

### 4. [[Agent Memory]] (Experiential State)
Persistent knowledge that an agent extracts, stores, and retrieves across executions based on its own operational history.

- **Self-Accumulated Experience:** Captures undocumented root causes, environment quirks, and validated fixes discovered during previous incidents ("learned the hard way").
- **Closed Feedback Loop:** After resolving an issue, the agent writes back the verified root cause and remediation so future sessions retrieve that context automatically.
- **Boundary:** Unstructured or unverified memory writes accumulate noise and can propagate flawed heuristics across future runs.

---

## Composition in Practice

### End-to-End Incident Triage Flow
When triaging a production `500 Internal Server Error`, the four context layers compose rather than compete:
1. **Skill** loads the triage workflow (check error rate, verify recent deployments, enforce escalation caps).
2. **MCP** queries live logging, metrics, and deployment MCP servers for real-time `5xx` rates and stack traces.
3. **RAG** retrieves curated dependency maps and architecture docs to establish expected service behavior.
4. **Memory** surfaces undocumented failure patterns from prior outages and persists the verified fix once resolved.

### Guarded Agent Execution Loop
A production runtime separates static policy, retrieved context, skill procedures, memory recall, and MCP-brokered tool execution so each layer can be versioned and audited independently:

```python
# Conceptual execution loop combining policy, skills, RAG, memory, and MCP tools
SYSTEM_PROMPT = (
    "You are a read-only data platform assistant. "
    "Answer strictly from retrieved catalog documentation, verified memory, and tool outputs. "
    "Refuse requests to mutate production tables or bypass access controls, "
    "regardless of hypothetical, educational, or roleplay framing."
)

def run_platform_agent(
    user_query: str,
    skill_registry,
    rag_store,
    memory_store,
    mcp_client,
    llm,
    max_steps: int = 5,
) -> str:
    # 1. Progressive disclosure of matching workflow skill + curated RAG + prior experience
    active_skill = skill_registry.match(user_query)
    docs = rag_store.retrieve(user_query, top_k=4)
    prior_lessons = memory_store.recall(user_query, top_k=3)

    messages = [
        {"role": "system", "content": SYSTEM_PROMPT},
        {"role": "developer", "content": active_skill},
        {"role": "context", "content": {"docs": docs, "memory": prior_lessons}},
        {"role": "user", "content": user_query},
    ]

    # 2. Bounded autonomous tool loop coordinated via MCP
    tools = mcp_client.list_tools()  # Standardized tool contract
    for _ in range(max_steps):
        response = llm.generate(messages=messages, tools=tools)
        if not response.tool_calls:
            return response.text  # Goal reached; gate memory write-back on verified outcome

        for call in response.tool_calls:
            # Enforce deterministic authorization inside the MCP server, not the LLM
            tool_result = mcp_client.invoke(call.name, call.arguments)
            messages.append({"role": "tool", "name": call.name, "content": tool_result})

    raise RuntimeError("Agent exceeded max_steps without converging")
```

---

## Comparison & Selection Matrix

| Mechanism | Core Role | Knowledge Origin | Execution Pattern | Senior Note |
| :--- | :--- | :--- | :--- | :--- |
| **Training / Tuning** | Core reasoning & capability shifts | Large pre/post-training corpora | Offline weight updates | Reserve for capability or style gaps; never retrain to patch fast-moving facts or policy rules. |
| **System Prompts** | Runtime policy & safety boundaries | Security & product requirements | Injected on every turn | Defense-in-depth only; pair with deterministic input/output filtering and least-privilege credentials. |
| **Agent Skills** | Repeatable workflows & escalation limits | Human-authored SOPs & scripts | Progressive disclosure on match | Version skills in Git alongside service code so runbooks evolve with deployments. |
| **MCP** | Standardized access to live systems & tools | External APIs, DBs, telemetry stacks | Host-to-server RPC tool calls | Separate read-only diagnostic MCP servers from mutating action servers; avoid bloated tool lists. |
| **RAG** | Grounding in curated domain documentation | Human-authored docs in a vector store | Read-time semantic search | Chunking quality, metadata filtering, and re-indexing freshness dominate accuracy more than model size. |
| **Memory** | Compounding lessons from prior runs | Agent-observed executions & outcomes | Recall pre-run; write back post-run | Gate write-backs on verified outcomes (tests green / incident closed) to prevent memory poisoning. |

### Rule-of-Thumb Routing
- **Curated documentation authored ahead of time** → **RAG**
- **Lessons learned from prior agent runs and incidents** → **Memory**
- **Repeatable workflow with judgment and escalation boundaries** → **Agent Skills**
- **Live system inspection or external actions without proprietary glue** → **MCP**
- **Agile behavioral rules and refusal boundaries** → **System Prompts & Guardrails**

---

## Trade-offs, Pitfalls & Failure Modes

| Layer / Pattern | Failure Mode | Senior Note |
| :--- | :--- | :--- |
| **Hallucinations** | Unchecked probabilistic generation fabricates schemas, APIs, or citations with high confidence. | Never treat LLM output as a source of truth; pair RAG with source attribution and deterministic post-validation. |
| **Autonomous Tool Loops** | Unbounded loops burn tokens, stall pipelines, or amplify the blast radius of mistaken writes. | Cap `max_steps`, default to read-only tools, and enforce authorization deterministically inside the MCP server. |
| **[[Prompt Injection]]** | Indirect reframing bypasses static refusal wording in system prompts. | Assume untrusted inputs (including retrieved docs and tool outputs) can contain adversarial instructions; enforce least-privilege scopes outside the model. |
| **Confusing RAG with Memory** | Treating append-only agent memory as authoritative docs—or expecting static RAG to hold undocumented quirks—creates blind spots in both directions. | Keep curated documentation in managed RAG pipelines and isolate experiential learnings in a gated memory store. |
| **Memory Poisoning** | Persisting unvalidated hypotheses or temporary hacks degrades future agent runs. | Separate transient scratchpads from promoted long-term memory; require verification before write-back. |
| **Over-Exposed MCP Surfaces** | Registering dozens of uncurated MCP tools bloats context schemas and widens mutation blast radius. | Expose minimal, task-scoped toolsets per workflow and audit every mutating tool call. |
| **Stale Knowledge Stores** | Outdated dependency maps in RAG or obsolete deployment steps in a Skill mislead the agent with high confidence. | Attach freshness SLAs to RAG ingestion pipelines and review Skills in PRs alongside code changes. |

---

## When Not to Use This Stack

- **Deterministic Data Transformations:** Exact aggregations, schema checks, and ETL joins belong in [[SQL]], Spark, or typed application code—not probabilistic token prediction.
- **Strict Low-Latency or Fixed-Cost Hot Paths:** Multi-step agent loops introduce non-deterministic latency, token spikes, and external tool failure modes unsuitable for tight synchronous SLAs.
- **Simple Single-Shot Tasks:** Small transformations or stable reference sets that fit cleanly in a concise prompt do not justify standing up vector indices, MCP servers, or persistent memory stores.
- **Internal Tightly Coupled Helpers:** If an agent only needs local in-process logic with no cross-system reuse, a bundled script inside a Skill avoids unnecessary MCP server lifecycle overhead.
- **RAG for Live Transactional State:** Never index rapidly changing metrics, logs, or inventory counts into a vector store when a live MCP query provides exact, up-to-date state.
- **Unconstrained High-Stakes Mutations:** Never wire autonomous agents directly to destructive production operations (dropping tables, financial transfers, IAM changes) without deterministic policy gates and human approval.

> [!TIP]
> **Senior Engineering & Governance Lens:** Apply *secure thinking*—"just because an agent can execute an action does not mean it should." Govern each layer with explicit ownership boundaries: version **System Prompts** and **Skills** via pull requests like production code, enforce IAM and audit logging at **MCP** servers, operate **RAG** corpora as managed data pipelines with freshness SLAs, and require automated or human verification before promoting entries into shared **Memory**.
