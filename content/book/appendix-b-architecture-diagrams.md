---
title: "Appendix B: Architecture Diagrams"
description: "Schematic representations of the major frameworks' internal designs"
date: 2026-04-01
weight: 12
tags: ["reference", "architecture", "diagrams"]
---

Schematic representations of the major frameworks' internal designs. These are pedagogical, not implementation-accurate; they omit details that would be essential for production but distracting here.

---

## B.1 — LangGraph (Stateful Graph)

```
             ┌──────────────────────────────────┐
             │         STATE (TypedDict)        │
             │  ticket, customer_id, history,   │
             │  retrieved_docs, draft, retries  │
             └──────────────────────────────────┘
                        ▲          │
              read/write│          │read/write
                        │          ▼
 ┌────────────┐     ┌─────────────────┐     ┌──────────┐
 │   START    │───▶ │   agent_node    │───▶ │   END    │
 └────────────┘     │  (calls model)  │     └──────────┘
                    └─────────────────┘
                        │        ▲
               tool_call│        │tool_result
                        ▼        │
                    ┌─────────────────┐
                    │   tool_node     │
                    │ (executes tool) │
                    └─────────────────┘
                        │
              interrupt │ (optional HITL)
                        ▼
                    ┌─────────────────┐
                    │ checkpointer    │──▶ SQLite/Postgres
                    │ (serialize)     │
                    └─────────────────┘
```

The state is the single source of truth. Nodes read and write the state. Conditional edges inspect the state to route. Checkpoints serialize the state after each node. Interrupts pause execution at marked nodes. HITL is a state transition like any other.

---

## B.2 — OpenAI Agents SDK (Runner Loop)

```
┌─────────────────────────────────────────────────────┐
│                     RUNNER                          │
│                                                     │
│   messages ──▶ [model] ──▶ response                 │
│                              │                      │
│         ┌────────────────────┤                      │
│         │                    │                      │
│    final text           tool_calls                  │
│         │                    │                      │
│         ▼                    ▼                      │
│      RETURN           [execute tools]               │
│                              │                      │
│                              ▼                      │
│                    append results to messages       │
│                              │                      │
│                              └──── LOOP ────────────┤
│                                                     │
└─────────────────────────────────────────────────────┘

Stateless between calls. No persistence. No interrupts.
Guardrails validate input before model, output after.
```

The Runner maintains an in-memory message list. Each turn: send to model, inspect response, either return or execute tools and loop. Handoffs are special tool calls that swap the active agent. Tracing captures the trajectory for debugging. Persistence is the developer's problem.

---

## B.3 — Claude Agent SDK (Tool-Equipped Loop)

```
┌────────────────────────────────────────────────────────┐
│                    query(prompt, config)               │
│                                                        │
│   ┌──────────────────────────────────────────────┐     │
│   │              Claude (model)                  │     │
│   │                                              │     │
│   │   ┌──────────────────────────────────┐       │     │
│   │   │   Built-in Tools (allowlist):    │       │     │
│   │   │   Read, Write, Edit, Bash,       │       │     │
│   │   │   Glob, Grep, WebSearch, ...     │       │     │
│   │   └──────────────────────────────────┘       │     │
│   │                                              │     │
│   │   ┌──────────────────────────────────┐       │     │
│   │   │   MCP servers (external tools)   │       │     │
│   │   └──────────────────────────────────┘       │     │
│   │                                              │     │
│   │   ┌──────────────────────────────────┐       │     │
│   │   │   Subagents (child processes)    │       │     │
│   │   └──────────────────────────────────┘       │     │
│   └──────────────────────────────────────────────┘     │
│                                                        │
│   Hooks: PreToolUse, PostToolUse, SessionStart         │
│   Permission framework: allow | deny | ask             │
│                                                        │
│   async stream of messages ──▶                         │
└────────────────────────────────────────────────────────┘
```

The SDK exposes Claude Code's engine as a library. Developer selects allowed tools, writes a prompt, consumes an async stream of messages. HITL is implemented as permission prompts at the tool level, not as graph interrupts. Sandboxing is the developer's responsibility (AgentCore, Bubblewrap, Seatbelt, etc.).

---

## B.4 — Google ADK (Hierarchical Agent Tree)

```
                ┌───────────────────┐
                │    Root Agent     │
                │    (LLM agent)    │
                └───────────────────┘
                 /        |         \
               /          |           \
             ▼            ▼            ▼
┌──────────────┐  ┌─────────────┐  ┌─────────────┐
│ Sequential   │  │   LLM       │  │  Parallel   │
│ Agent        │  │   Agent     │  │  Agent      │
│ (workflow)   │  │ (reasoning) │  │ (fan-out)   │
└──────────────┘  └─────────────┘  └─────────────┘
    │                 │                 │   │
    ▼                 ▼                 ▼   ▼
step 1→2→3        tool calls      worker  worker
                                  agents  agents
```

ADK organizes agents in a tree. Workflow agents (Sequential, Parallel, Loop) provide deterministic control flow. LLM agents provide dynamic routing. The developer designs the tree; the model fills in reasoning at LLM-agent nodes.

---

## B.5 — Microsoft Agent Framework (Dual Mode)

```
┌────────────────────────────────────────────────────┐
│                                                    │
│   Agent Orchestration        Workflow              │
│   (LLM-driven)               Orchestration         │
│                              (deterministic)       │
│                                                    │
│   - group chat               - sequential graphs   │
│   - debate                   - conditional branches│
│   - reflection               - parallel execution  │
│   - Magentic (lead agent)    - HITL approval gates │
│                                                    │
│              ↕ interop ↕                           │
│                                                    │
│   ┌────────────────────────────────────────┐       │
│   │   Trust infrastructure (Azure-native)  │       │
│   │  - Entra ID auth                       │       │
│   │  - OpenTelemetry tracing               │       │
│   │  - Responsible AI (prompt injection,   │       │
│   │    PII detection, task adherence)      │       │
│   └────────────────────────────────────────┘       │
└────────────────────────────────────────────────────┘
```

Merger of AutoGen (experimental multi-agent) and Semantic Kernel (enterprise workflow). Trust infrastructure is the distinctive investment.

---

## B.6 — AgentCore Runtime (Framework-Agnostic)

```
┌────────────────────────────────────────────────────┐
│                   AgentCore Runtime                │
│                                                    │
│  ┌──────────────────────────────────────────────┐  │
│  │         Firecracker microVM (per session)    │  │
│  │                                              │  │
│  │   ┌───────────────────────────────────┐      │  │
│  │   │  Agent (any framework):           │      │  │
│  │   │  LangGraph / OpenAI SDK /         │      │  │
│  │   │  Claude Agent SDK / ADK / custom  │      │  │
│  │   └───────────────────────────────────┘      │  │
│  │                                              │  │
│  │   isolated CPU, memory, filesystem           │  │
│  │   ephemeral, up to 8 hours                   │  │
│  └──────────────────────────────────────────────┘  │
│                                                    │
│  ┌──────────────────────────────────────────────┐  │
│  │  Identity (OAuth/JWT + workload identity)    │  │
│  │  Policy (Cedar)                              │  │
│  │  Observability (OpenTelemetry)               │  │
│  │  MCP / A2A gateways                          │  │
│  └──────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────┘
```

AgentCore separates the framework (orchestration) from the runtime (execution + trust). Any Python framework can run inside a microVM session.

---

## B.7 — The MCP Protocol Stack

```
┌─────────────────────────────────────────┐
│              AI Application             │
│         (Claude, ChatGPT, Gemini)       │
└─────────────────────────────────────────┘
                    │
              MCP Client
                    │  JSON-RPC 2.0
                    │  (stdio or HTTP)
                    │
┌─────────────────────────────────────────┐
│              MCP Server                 │
│                                         │
│   Tools      Resources      Prompts     │
│  (actions)   (read-only)  (templates)   │
└─────────────────────────────────────────┘
                    │
         External System / Data
    (GitHub, Slack, DB, filesystem...)
```

The three-layer protocol stack: AGENTS.md (configuration), MCP (tools), A2A (cross-agent collaboration).

---

## B.8 — The Isolation Hierarchy

```
Security Strength ▲

Level 4 │ Confidential Computing (SEV-SNP / TDX)
         │ hardware-encrypted memory, strongest guarantee
         │
Level 3 │ MicroVM (Firecracker / Kata Containers)
         │ separate guest kernel, hardware virtualization
         │ ~125ms boot, ~5MB overhead — GOLD STANDARD
         │
Level 2 │ User-space kernel (gVisor)
         │ syscall interception, no host kernel exposure
         │ ~100ms boot, 10-20% perf overhead
         │
Level 1 │ Standard Container (Docker/runc)
         │ shared host kernel, namespaces + cgroups only
         │ fast, cheap — INSUFFICIENT for AI code execution
         │
Level 0 │ No isolation
         │ (OpenClaw pre-NemoClaw)
         ▼

Performance / Simplicity ▲
```

---

*Back to [Table of Contents](/book/)*
