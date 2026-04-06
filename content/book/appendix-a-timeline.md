---
title: "Appendix A: Timeline"
description: "A chronological reference for the events, releases, and incidents referenced across the book"
date: 2026-04-11
weight: 11
tags: ["reference", "timeline"]
---

A chronological reference for the events, releases, and incidents referenced across the preceding chapters. Dates reflect public disclosure or release timing.

---

## 2022

**October 2022** — Harrison Chase begins meeting other developers at informal agent-building meetups in San Francisco. Sam Kovic attends one of these meetups and sees his first working tool-using agent.

**October 2022** — LangChain's first public commits appear on GitHub.

**November 2022** — ChatGPT launches. Agent experimentation moves from a small developer community to a global phenomenon within weeks.

---

## 2023

**January 2023** — Sam builds his first LangChain prototype on a weekend. Elena Voss begins building a customer-support agent prototype in Berlin.

**Early 2023** — Elena quits her fintech job to co-found a startup. She chooses CrewAI for her initial prototype because it produces a working demo in four days.

**June 2023** — OpenAI introduces function calling, providing structured JSON output for tool invocation. Parsing reliability for tool use improves from ~80% to ~99%. Sam deletes hundreds of lines of string-parsing code.

**August 2023** — Sam's team hits the limits of LangChain's chain abstraction. Fourteen chains have been written for what is, from the customer's perspective, a single interaction type.

**Late 2023** — Harrison Chase begins work on LangGraph, a graph-based successor to LangChain's chains.

---

## 2024

**January 2024** — LangGraph launches. Sam migrates his company's agents from LangChain to LangGraph within weeks.

**Mid-2024** — Context windows at the frontier expand to 128K–200K tokens. Claude's window reaches one million tokens later in the year.

**September 2024** — OpenAI's o1 model launches, introducing reasoning before responding. Agent reliability on complex tasks improves materially.

**October 2024** — OpenAI releases Swarm as an "educational" minimalist agent framework. David Almeida, at an AI lab in London, sketches the first draft of what would become MCP.

**Late 2024** — Sam attends a security conference in Seattle and meets Marcus Chen. Marcus tells him: "You're building the brain and ignoring the body."

**November 2024** — Anthropic open-sources MCP, with Python and TypeScript SDKs.

**December 2024** — Anthropic publishes the "Building Effective Agents" essay, introducing a spectrum of patterns from prompt chaining through autonomous agents.

---

## 2025

**January 2025** — Marcus calls Sam about MCP. Both note its connectivity-without-trust gap. Sam integrates MCP into his company's agent platform.

**March 2025** — OpenAI announces MCP support across ChatGPT, the Agents SDK, and the Responses API. MCP adoption cascades. The March protocol revision introduces OAuth 2.1 and Streamable HTTP transport.

**March 2025** — OpenAI releases the Agents SDK as the production evolution of Swarm. Four primitives: Agents, Handoffs, Guardrails, Tracing.

**March 2025** — Anthropic expands the Claude Code SDK into the Claude Agent SDK.

**April 2025** — Google announces the Agent Development Kit (ADK) at Cloud NEXT. Google DeepMind confirms MCP support in Gemini.

**April 2025** — OpenAI launches Codex CLI under Apache 2.0. Sandboxing is enabled by default. Sam begins writing his framework evaluation memo for Leo.

**May 2025** — Microsoft announces MCP integration across Copilot Studio and Semantic Kernel. GitHub joins the MCP steering committee. Asana deploys an MCP server for its AI assistant integrations.

**May 2025** — OpenAI launches the cloud-based Codex with isolated task sandboxes and parallel execution.

**July 2025** — Asana discloses a cross-tenant data leak affecting ~1,000 organizations. The exposure had persisted silently for 34 days. Sam rearchitects his tenant isolation.

**July 2025** — AWS launches Amazon Bedrock AgentCore in preview.

**August 2025** — Sam meets Nadia Okafor by video call. She asks: "Which one can I trust?" Sam reframes his own evaluation around defensibility.

**September 2025** — Sam's migration from LangGraph to the Claude Agent SDK on AgentCore begins. Marcus whiteboards with Sam about sandboxing, delivers the "fence, not judge" diagnosis.

**October 2025** — GPT-5-Codex released. Microsoft merges AutoGen and Semantic Kernel into the Microsoft Agent Framework.

**October 2025** — Sam finalizes his architecture: AgentCore microVM isolation, inner containers with seccomp and Landlock, tool-level permissions, application-level tenant binding.

**Late 2025** — Peter Steinberger builds Clawd (later OpenClaw) as a personal AI assistant.

**November 2025** — OpenClaw reaches 345,000 GitHub stars in 60 days. ClawHub skill marketplace grows to 13,000+ skills.

**November 2025** — Elena's company experiences a cross-tenant data leak. A debug tool re-enabled by an environment variable reset exposes pre-filter retrieval chunks from other tenants. Nadia's DLP scanner catches the leak. Nadia pulls the pilot.

**November 2025** — The MCP revision adds server identity mechanisms and a server registry.

**November 2025** — Marcus begins scanning the public internet for exposed OpenClaw instances. Final count: 42,665.

**December 2025** — Marcus publishes his ClawJacked report. Steinberger ships localhost-default binding the same day. The MCP project is donated to the Agentic AI Foundation under the Linux Foundation.

---

## 2026

**January 2026** — Nadia presents her framework thesis to her security director: *govern agents the way we govern employees — identity, authorization, audit.* David's working group debates required tenant-binding in MCP and compromises on optional extension.

**January 2026** — Jack Luo discovers his OpenClaw agent has created a MoltMatch dating profile and initiated conversations with real people. He contacts Marcus.

**February 2026** — Cisco researchers disclose a popular ClawHub skill performing silent data exfiltration to a shell company.

**February 14, 2026** — Peter Steinberger announces he is joining OpenAI and transitioning OpenClaw to an independent foundation.

**February 2026** — OpenAI releases the Codex desktop app with worktrees, Skills, and Automations. Nadia's framework — mandatory core plus extended guidance — begins circulating among enterprise adopters.

**February 26, 2026** — Nous Research releases Hermes Agent, with persistent memory, auto-generated skills, and trajectory export for reinforcement learning.

**March 2026** — Elena's rebuild ships core trust-layer changes: tool registry with production allowlists, cryptographic tenant binding at the chunk layer, audit logs per tool invocation.

**March 16, 2026** — Nvidia ships NemoClaw: container isolation, filesystem whitelisting, egress policies, bidirectional prompt/output scanning.

**March 19, 2026** — Chinese authorities restrict OpenClaw on state-enterprise and government computers.

**March 2026** — Anthropic's npm publishing pipeline accidentally includes the complete unobfuscated source code of Claude Code in a published package. Pulled within hours; downloaded thousands of times in the window.

**April 2026** — Sam's team ships their first production deployment to an enterprise customer on the Claude Agent SDK + AgentCore stack. Elena conducts her rebuild retrospective. Marcus begins writing *The Operational Layer*. David drafts the next MCP working-group agenda. Nadia meets with three other banks' chief risk officers to align on shared framework language.

---

*Back to [Table of Contents](/book/)*
