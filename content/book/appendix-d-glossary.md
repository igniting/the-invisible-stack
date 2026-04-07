---
title: "Appendix D: Glossary"
description: "Key terms and concepts used throughout The Invisible Stack"
date: 2026-04-01
weight: 14
tags: ["reference", "glossary", "definitions"]
---

Definitions are given as used in this book. Where a term has a broader or contested meaning in the field, the scoped definition is noted.

---

## A

**A2A (Agent-to-Agent protocol)**
A proposed standard for agent-to-agent communication, complementary to MCP. Where MCP defines how agents consume tools and data sources, A2A defines how agents discover and delegate to other agents. Status as of early 2026: draft specification, limited production adoption.

**AGENTS.md**
A convention for documenting the capabilities, identity, and behavioral constraints of an agent in a machine-readable file stored alongside the codebase. Analogous to a robots.txt for agent orchestrators. Not yet a formal standard.

**AgentCore (AWS Bedrock AgentCore)**
A managed runtime service from Amazon Web Services that provides execution infrastructure (sandboxing, identity, memory, tool gateways) independent of orchestration framework. Represents the "runtime as a separate layer" category in the framework taxonomy.

**Agentic loop**
The core execution pattern of an LLM agent: (1) receive task, (2) reason about next step, (3) call tool, (4) observe result, (5) repeat until task is complete or a termination condition is met. Also called the ReAct loop (Reason + Act).

**Allowlist**
An explicit enumeration of permitted capabilities, identities, or actions. Contrasted with a denylist (enumeration of prohibited items). In security contexts, allowlists are preferred because they fail closed: anything not explicitly permitted is denied.

**Augmented LLM**
An LLM extended with retrieval (access to external knowledge), tools (ability to take actions), and memory (persistence across sessions). The foundation layer of the agent stack.

---

## C

**Cedar**
A policy language developed by AWS for expressing fine-grained authorization policies. Used in the trust layer examples in Chapter 7. Cedar policies are evaluated outside the reasoning loop, making them enforcement mechanisms rather than advisory guidance.

**Checkpoint**
In LangGraph, a serialized snapshot of the agent's state graph at a specific point in execution. Enables pause-and-resume, human-in-the-loop review, and debugging. The primary architectural differentiator of LangGraph relative to linear chain frameworks.

**ClawJacked**
The vulnerability class described in Chapter 8: an attacker compromises an OpenClaw instance's external-facing gateway and injects malicious skills, gaining persistent access to the agent's capabilities and memory.

**Claude Agent SDK**
Anthropic's framework for building agents using Claude models. Provides a permission hooks system for HITL enforcement, computer use integration, and a tool use interface. Framework binding: Anthropic Claude models.

**Cryptographic binding**
The practice of linking an identity claim (user, tenant, agent instance) to a cryptographic proof (signed token, certificate) so that the claim cannot be forged or replayed. Contrasted with application-level binding (convention, configuration).

**CrewAI**
An open-source multi-agent framework that uses role-based agent definitions and crew metaphors. Popular in 2024 as an alternative to LangChain for multi-agent coordination.

---

## D

**Data plane**
The layer of an agent system that handles actual data access: retrieval, storage reads and writes, memory operations. Contrasted with the control plane (routing, orchestration). Tenant isolation should be enforced at the data plane, not the control plane.

**Denylist**
An enumeration of prohibited capabilities, identities, or actions. Fails open: anything not explicitly denied is permitted. Inferior to allowlists for security-critical capability management.

**Dynamic prompt injection**
The practice of assembling a prompt from multiple sources at runtime, some of which may be externally controlled (retrieved documents, user inputs, API responses). Creates a prompt injection attack surface if assembled sources are not sanitized.

---

## E

**Execution layer**
The layer of an agent system that actually executes actions: spawning processes, writing files, calling APIs. Enforcement mechanisms placed at the execution layer cannot be bypassed by model reasoning errors or prompt injection. Contrasted with the orchestration layer.

**Exponential backoff**
A retry strategy where each successive retry waits twice as long as the previous attempt. Used for transient failures in distributed systems; relevant to agent deployments where tool calls may encounter rate limits or temporary unavailability.

---

## F

**Fail closed**
A security property: when a system encounters an unexpected condition, it denies access by default. An allowlist fails closed (unknown capability → denied). Contrasted with fail open (unknown capability → permitted).

**Firecracker**
An open-source microVM technology developed by AWS, originally for Lambda and Fargate. Provides strong isolation between workloads with startup times under 125ms and memory overhead of approximately 5MB per VM. Used as the execution primitive for agent sandboxing in deployments requiring stronger isolation than containers.

**Framework binding constraint**
The capability or infrastructure investment that makes switching away from a specific agent framework costly. Examples: model provider lock-in (OpenAI Agents SDK), graph-native state management (LangGraph), existing Azure infrastructure (Microsoft Agent Framework).

---

## G

**Google ADK (Agent Development Kit)**
Google's framework for building agents using Gemini models. Provides a hierarchical agent tree structure with deterministic routing and LLM-based routing options. Notable for integrating with Google Cloud infrastructure and Vertex AI.

**gVisor**
A user-space kernel that intercepts system calls to provide isolation without requiring hardware virtualization. Provides stronger isolation than standard Linux namespaces but weaker than Firecracker microVMs. Used in Google's container infrastructure.

---

## H

**Harness**
As used in this book: the complete trust and execution infrastructure surrounding an AI agent — the sandboxing, identity binding, policy enforcement, HITL gates, and audit mechanisms that make autonomous action safe to permit. Analogous to the harness a climber wears: not the thing that gets the work done, but the thing that makes the work survivable if something goes wrong.

**Hermes Agent**
A hypothetical agent capability described in Chapter 9 that can self-author skills in response to novel tasks, subject to a review loop before promotion to production. Represents the frontier of the static→dynamic→autonomous skills spectrum.

**HITL (Human-in-the-Loop)**
A design pattern where a human must approve or review an agent action before it is executed. The gate can be soft (notify and allow cancellation window) or hard (block execution until explicit approval is received). Placement at the execution layer rather than the orchestration layer is required for enforcement guarantees.

**Hugo**
A static site generator written in Go. Used to build this site. Hugo Stack is the theme used for this book's web presentation.

---

## I

**Identity binding**
The process of associating an authenticated identity (user, tenant, service) with an agent session or action in a way that is cryptographically verifiable. Required for attribution, accountability, and tenant isolation.

**Isolation hierarchy**
The four-level taxonomy of agent execution isolation described in Chapter 5:
- Level 1: Process isolation (OS-level, no hardware boundary)
- Level 2: Container isolation (Linux namespaces and cgroups)
- Level 3: gVisor (user-space kernel, system call interception)
- Level 4: MicroVM (Firecracker, hardware virtualization boundary)

---

## J

**JSON-RPC 2.0**
The wire protocol underlying MCP. A lightweight remote procedure call protocol using JSON encoding. Each request has a method name, parameters, and an ID; each response has a result or error and the corresponding ID.

---

## L

**LangChain**
The early-dominant Python framework for building LLM applications, introduced in late 2022. Used a linear chain abstraction. By 2024, complex workflows exposed limitations; LangGraph was introduced as an alternative for stateful, cyclical workflows.

**LangGraph**
A graph-based orchestration framework from LangChain Inc. Represents workflows as directed graphs with nodes (agent steps), edges (routing logic), and state (persisted as checkpoints). The primary architectural advance over LangChain for complex agent workflows.

**LangSmith**
An observability and evaluation platform from LangChain Inc. for tracing agent execution, evaluating outputs, and debugging failures. Represents the vendor observability layer.

---

## M

**MCP (Model Context Protocol)**
An open protocol, proposed by Anthropic and adopted across the industry, for connecting LLM agents to external tools and data sources. Uses JSON-RPC 2.0 as its wire protocol. Defines three primitives: tools (callable functions), resources (readable data), and prompts (reusable templates). Transports: stdio (local), SSE (server-sent events), Streamable HTTP (production).

**Memory (agent memory)**
Persistence mechanisms that allow agents to retain information across sessions. Three categories: in-context (within a single session's context window), external (vector databases, key-value stores), and parametric (embedded in model weights via fine-tuning).

**Microsoft Agent Framework**
The merger of AutoGen (multi-agent coordination) and Semantic Kernel (enterprise integration) announced in late 2025. Targets enterprise deployments with existing Azure and Microsoft 365 infrastructure.

**MicroVM**
A minimal virtual machine with hardware-enforced isolation between workloads. Firecracker is the primary example. Provides the strongest isolation available for agent execution at reasonable cost (approximately 5-10% performance overhead for I/O-bound workloads).

---

## N

**NemoClaw**
An enterprise-hardened fork of OpenClaw developed at NVIDIA, described in Chapter 8. Notable for retrofitting stronger isolation and enterprise identity bindings onto the OpenClaw architecture.

---

## O

**OPA (Open Policy Agent)**
An open-source policy engine that evaluates authorization decisions using the Rego policy language. Used as an alternative to Cedar for external policy enforcement in the trust layer. Policy decisions are made outside the model's reasoning loop.

**OpenAI Agents SDK**
OpenAI's framework for building agents, evolved from the experimental Swarm library. Provides a runner loop, handoff mechanism, and guardrails system. Framework binding: OpenAI models (with tool calling API).

**OpenClaw**
A hypothetical open-source personal agent platform described in Chapters 4 and 8, used to illustrate the security challenges of widely-deployed agents with self-managed infrastructure. Represents the class of "bring your own compute" agent deployments.

**Orchestration layer**
The layer of an agent system that decides what to do next: which tool to call, which sub-agent to delegate to, whether to ask a human for input. Enforcement mechanisms placed only at the orchestration layer can be bypassed by model reasoning errors. Contrasted with the execution layer.

---

## P

**Personal agent**
An agent that acts on behalf of a specific individual, with persistent memory of that person's preferences, calendar, contacts, and communications. Chapter 8's focus. Contrasts with task agents (single-task, stateless) and system agents (infrastructure operations).

**Policy enforcement point**
The component in a trust architecture that evaluates authorization policies and enforces decisions. Must be placed outside the reasoning loop (at the execution layer) to provide enforcement guarantees rather than advisory guidance.

**Prompt injection**
An attack where malicious instructions embedded in retrieved content, user input, or tool output cause the model to deviate from its intended behavior. The primary attack vector for agents that process untrusted external content.

**Provenance**
The documented history of an artifact's origin and transformations. In skills governance: which agent authored a skill, in which session, for which task, and whether it was human-reviewed before production use. Required for audit and rollback.

---

## R

**ReAct**
A prompting pattern (Reason + Act) that structures agent reasoning as alternating thought steps and action steps. The conceptual basis for the agentic loop, introduced in academic research and popularized by LangChain.

**Retrieval-augmented generation (RAG)**
The practice of retrieving relevant documents or data at query time and including them in the model's context window, enabling the model to answer questions about information not present in its training data. The foundation of the retrieval arm of the augmented LLM.

---

## S

**Sandboxing**
Constraining the execution environment of an agent to limit the impact of errors or malicious behavior. Implemented via process isolation, containers, gVisor, or microVMs (in increasing order of isolation strength). A requirement for Level 2 and above in the maturity model.

**Semantic Kernel**
Microsoft's framework for integrating LLMs into enterprise applications. Merged with AutoGen to form the Microsoft Agent Framework in late 2025.

**SKILL.md**
A convention for documenting a skill (reusable agent capability) in a structured Markdown file. Anatomy: front matter (name, description, activation criterion), inputs, outputs, side effects, tools used, examples. Progressive disclosure: the summary section is loaded first; full details are loaded only when the skill is activated.

**Skills (agent skills)**
Reusable, parameterized capability bundles that define how an agent should approach a class of task. Stored externally from the model, loaded at runtime when relevant. Contrasted with MCP tools (capability access) — skills describe strategy and intent, tools provide the mechanism.

**Streamable HTTP**
The third-generation MCP transport, superseding SSE. Enables bidirectional streaming over a single HTTP connection, eliminating the need for separate SSE and POST endpoints. Required for production MCP deployments as of early 2026.

**Swarm (OpenAI Swarm)**
An experimental multi-agent framework released by OpenAI in late 2024. Introduced handoffs (agent-to-agent delegation) and routines (skill-like instruction bundles). Deprecated in favor of the OpenAI Agents SDK.

---

## T

**Tenant isolation**
The property that data and actions belonging to one customer (tenant) are inaccessible to other tenants, enforced at the data layer rather than the application layer. A security requirement for any multi-tenant agent deployment.

**Three-layer model**
The book's central taxonomy of the agent stack:
1. **Orchestration layer** — what to do next (LangGraph, OpenAI Agents SDK, etc.)
2. **Execution layer** — actually doing it (tool calls, sandboxed execution)
3. **Trust layer** — enforcing what is permitted (identity, policy, HITL, audit)

**Tool (MCP tool)**
In MCP: a callable function exposed by an MCP server that an agent can invoke. Tools represent the action primitive — things the agent can do. Contrasted with resources (things the agent can read) and prompts (reusable instruction templates).

**Tool poisoning**
A supply-chain attack on MCP deployments: a malicious MCP server provides tools with names or descriptions designed to cause the agent to behave in ways the user did not intend. A variant of prompt injection via the tool description vector.

**Trust boundary**
The enforced limit of what an agent is permitted to do, implemented at the execution layer using cryptographic identity binding, policy enforcement, and HITL gates. The central concept of this book. The trust boundary is distinct from the capability boundary: the capability boundary is what the agent can do; the trust boundary is what it is permitted to do.

---

## W

**Workload identity**
A cryptographic identity assigned to a specific agent instance (rather than a user or service account), allowing audit logs to distinguish actions taken by different agent instances even within the same session or on behalf of the same user.

---
