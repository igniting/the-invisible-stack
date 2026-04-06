---
title: "Chapter 4: Whose Harness?"
description: "How every AI lab built the same thing differently — and what their choices reveal"
date: 2026-04-04
weight: 4
tags: ["SDKs", "frameworks", "strategy", "AgentCore"]
---

## How every AI lab built the same thing differently — and what their choices reveal

---

## Act 1: The Story

In the spring of 2025, Sam Kovic wrote a document he'd been putting off for months.

The document was a framework selection memo for his CEO. Its purpose was to answer a question Leo had been asking with increasing frequency: should we migrate off LangGraph? And if so, to what?

Sam had been on LangGraph for a year. It had served him well — the graph abstraction, the checkpointing, the human-in-the-loop interrupts had all solved real problems. But the landscape had changed. In the first four months of 2025, five of the largest technology companies on earth had each shipped a production-grade agent harness SDK. OpenAI released the Agents SDK in March. Anthropic renamed the Claude Code SDK to the Claude Agent SDK and dramatically expanded its scope. Google announced the Agent Development Kit at Cloud NEXT in April. Microsoft was in the middle of merging AutoGen and Semantic Kernel. AWS launched Bedrock AgentCore in preview in July.

This was not coordination. It was convergence — five companies arriving independently at the same conclusion at the same time: that the model alone was no longer the product. The product was the model plus the harness. Whoever controlled the harness controlled how agents were built, deployed, and governed. Nobody wanted to be the company that built the best engine but let someone else design the car.

Sam's job was to figure out which car his company should be driving. And after six weeks of evaluation, he had come to a surprising conclusion. The five SDKs were not simply competing products differing in API style and programming language support. They were architectural commitments that reflected deep, incompatible disagreements about what agents were, what they should be allowed to do, and who should be in control.

---

### Question One: What Should the Agent Come Equipped With?

Sam opened the Claude Agent SDK documentation and then opened the OpenAI Agents SDK documentation in adjacent browser tabs. The contrast was immediate.

The OpenAI Agents SDK had four primitives: Agents (models with instructions and tools), Handoffs (delegation between agents), Guardrails (input and output validation), Tracing (observability). There were no built-in tools for file access, code execution, or web browsing. If you wanted your agent to read a file, you wrote a file-reading tool. The philosophy was explicit: the SDK provides orchestration, the developer provides capabilities.

The Claude Agent SDK came equipped with the exact tools that powered Claude Code, Anthropic's own coding agent: Read, Write, Edit, Bash, Glob, Grep, WebSearch, WebFetch, AskUserQuestion. These were not example templates. They were production-grade tools, battle-tested across millions of real-world coding sessions, now exposed as a library. The developer's job was to select which tools to enable and write the prompt that directed the agent's work.

Sam built the same prototype in both SDKs — a documentation search agent that could find specific information in his company's knowledge base. In the Claude Agent SDK, it took about fifteen minutes. The built-in Glob and Grep tools handled the file discovery and search logic, the built-in Read tool handled file access, and the prompt provided the task description. The agent worked on the first try.

In the OpenAI Agents SDK, the same prototype took most of an afternoon. Sam had to write tools for file discovery, file reading, and text search. Each decision was small. The cumulative effect was significant.

Sam called David Almeida to talk through the implications.

"The OpenAI position is that intelligence is the bottleneck," David said. "Give a smart enough model the right tools, and it will figure out how to use them. The quality of the tool doesn't matter as much as the quality of the model's reasoning about when to use it."

"And the Anthropic position?"

"That tooling is the bottleneck. Even the smartest model produces poor results with poorly built tools. The difference between an agent that reliably completes tasks and one that sometimes completes tasks is often not the model — it's the tool. A Read tool that handles path resolution correctly, manages file size limits intelligently, and returns helpful error messages will produce dramatically better agent behavior than a hastily written Read tool, with the same model."

"Which is right?"

"Both. For different teams. If you have sophisticated engineers who can write good tools, you probably don't want the opinionated built-in tools — they'll get in your way. If you're shipping quickly and your agents do common things, the built-in tools will save you weeks of work and produce better results than you would build yourself."

### Question Two: Who Owns the Execution Loop?

When an agent runs — reasoning, calling tools, evaluating results, deciding what to do next — who is in charge? The developer, who designed the system? Or the model, which is reasoning in real time?

Google and Microsoft answered: the developer.

Google's Agent Development Kit was built around workflow agents — Sequential, Parallel, Loop — that let developers define deterministic pipelines. These were control flow primitives: the same for-loops, if-statements, and thread-pools that software engineers had been using for decades, now applied to agents. Alongside them, ADK supported LLM-driven routing: a model examined a request and decided which sub-agent to delegate to. But the sub-agents the model could route to were the ones the developer had defined. If a path didn't exist in the developer's design, the model couldn't take it.

Microsoft's Agent Framework took a similar approach with explicit dual modes. Workflow orchestration provided deterministic, business-logic-driven pipelines. Agent orchestration provided LLM-driven multi-agent patterns — group chat, debate, reflection. The developer chose which mode to use for each part of their system.

Both Google and Microsoft were building for enterprise customers — organizations that needed auditability, predictability, and the ability to explain what an agent did and why.

Anthropic answered differently: the model.

The Claude Agent SDK exposed the same execution loop that powered Claude Code. The developer provided a prompt, a set of allowed tools, and optional configuration. Then Claude ran. It read files if it needed to. It executed commands if it needed to. It decided what to do at each step based on its assessment of the task — not based on a graph the developer had designed in advance.

Sam tested both approaches on a real task from his backlog: an agent that processed incoming bug reports, investigated the codebase to identify the likely cause, and drafted a preliminary assessment. In Google's ADK, he designed a workflow with defined sub-agents for intake, search, analysis, and drafting. It worked well for bug reports that fit the expected shape but poorly for bug reports that required iterative exploration. In the Claude Agent SDK, the agent investigated bug reports by reading relevant files, searching for patterns, running commands to check assumptions — far more flexible, handling cases Sam hadn't anticipated, but also less predictable.

"Match the autonomy to the stakes," David had said. "For processes with compliance requirements, we want the developer to own the loop. For tasks that benefit from adaptability, we want the model to own the loop."

### Question Three: Is the Harness the Runtime?

Should the agent framework also be the system that managed the agent's lifecycle in production — its state, its persistence, its recovery, its operational infrastructure?

LangGraph said yes. The framework included checkpointing, durable execution, streaming, and human-in-the-loop interrupts. When Sam deployed a LangGraph agent, the framework was the runtime.

OpenAI said no. The Agents SDK was deliberately stateless. It managed the execution loop in memory but didn't persist state, checkpoint, or provide durable execution.

AWS said the runtime was a separate layer entirely. Amazon Bedrock AgentCore was not an agent framework at all. It was a runtime platform — a place where agents built on any framework (LangGraph, the OpenAI Agents SDK, the Claude Agent SDK, Google ADK, custom code) could be deployed and operated with enterprise-grade infrastructure. AgentCore provided the execution layer (microVM isolation), the identity layer (workload identity per agent), the memory layer (short-term and long-term stores), the observability layer (OpenTelemetry tracing), and the policy layer (Cedar-based access control).

This separation — framework for orchestration, platform for execution and trust — was the most architecturally significant choice any company had made, and it was the choice that resonated most with Sam's own thinking. The framework owns orchestration, the runtime owns execution and trust. A team could switch frameworks without changing their runtime. A runtime could add new security features without requiring changes to the agent's logic.

Sam called David to talk through the implications.

"You're moving from a world where your framework is your runtime to a world where your runtime is a separate concern," David said. "That's the right architectural direction. It means you can pick the best framework for each problem without being locked into that framework's runtime limitations."

"And the tradeoff is lock-in to the runtime."

"Yes. You're trading framework lock-in for runtime lock-in. The question is which one hurts more if you have to change it."

"Runtime lock-in," Sam said, "because the runtime is where the trust infrastructure lives. If I change frameworks, I have to rewrite agent logic — painful but bounded. If I change runtimes, I have to rebuild identity, policy, observability, and audit from scratch. That's existential."

---

In August 2025, while Sam was deep in his AgentCore prototype, he received a calendar invitation that changed how he thought about the entire evaluation.

The invitation was from Nadia Okafor — a VP of Engineering at a financial services company in London. She had been introduced to Sam by a former colleague. The meeting was scheduled for forty-five minutes. The subject line read: "Advice on agent framework selection."

Sam expected a standard vendor-adjacent conversation. What he got was something more interesting.

Nadia was not evaluating frameworks. She was trying to decide whether her company could deploy AI agents at all. She had fifteen years in regulated financial technology. She was skeptical by training and institutional obligation.

"I've looked at the agent SDKs," she said on the video call. "I can't tell from the documentation which ones I can trust. Everybody is comparing themselves on capability. Nobody is comparing themselves on trustworthiness. Can you help me understand what I'm looking at?"

"What do you mean by trustworthy?"

"I mean: if I deploy this in production, and something goes wrong — the agent leaks data, or makes a wrong decision, or takes an action it shouldn't have — can I explain to a regulator exactly what happened and why? Can I prove that the agent acted within its authorized boundaries? None of the SDK documentation I've read answers them."

Sam thought for a moment. The three design questions he had organized his evaluation around were the practitioner's questions. Nadia was asking the buyer's question: **who can you defend this to?**

"You're asking about the trust layer," Sam said.

"Yes. And I can't find it in any of these SDKs."

"That's because it's mostly not there yet. The frameworks provide orchestration. Some of them provide execution. A few of them provide pieces of trust — observability, some identity primitives. Nobody provides a comprehensive trust layer. It's the gap the industry hasn't filled."

"So what do I do?"

"You pick the runtime that gives you the most trust primitives, and you build the rest yourself. But you go in knowing that you're going to be doing significant engineering on the trust layer, because the vendors aren't doing it for you yet."

Nadia was quiet for a moment. "That's an expensive answer."

"It's the honest answer."

The conversation lasted an hour and forty minutes. By the end of it, Sam had changed the framing of his own memo. The three design questions were still the right questions. But they were nested inside a larger question that Nadia had articulated: **which of these choices can you defend?**

Not to an engineer. To a regulator, a board, a risk team, a customer. Which framework, which runtime, which architecture, can you stand behind and say: "We chose this because it lets us answer the questions that accountability requires."

---

Sam's memo for Leo recommended migrating from LangGraph to the Claude Agent SDK running on AWS Bedrock AgentCore.

The Claude Agent SDK provided the built-in tools his team needed for the coding and documentation agents that formed the core of his product. Its model-driven loop architecture matched the adaptive, exploratory nature of the tasks. AgentCore provided the runtime infrastructure for microVM isolation, identity, observability, and policy — the trust primitives his team would need to serve enterprise customers with regulated data.

The recommendation came with costs. Migrating from LangGraph to the Claude Agent SDK would require rewriting the core agent logic. The Claude Agent SDK was Anthropic-only, which meant the team was betting on Claude as the primary model. AgentCore was AWS-specific, which meant the team was betting on AWS as the primary cloud.

Sam wrote: *Every choice is a bet. The bets we're making are: that tool-rich agents are better than tool-sparse agents, that model-driven loops are better suited to our tasks than developer-owned loops, that separation of framework from runtime is the right architectural direction, and that the trust layer is where we need to invest our own engineering.*

Leo read the document on a Sunday night and called Sam on Monday morning.

"This is a six-month migration," Leo said.

"Four to five, if we do it right."

"And during those four to five months, we're going to ship fewer features."

"Yes."

"Why now? Why not in a year?"

"Because our product is becoming enterprise-first, and the trust layer is where enterprise customers differentiate vendors. If we don't build trust infrastructure now, we'll spend the next year losing deals to vendors who did."

Leo was quiet for a moment. "Approved. Do it right. Tell the team I'll defend the timeline."

---

One story from this period deserves extended attention. When OpenAI launched Codex CLI in April 2025, it was a lightweight, open-source coding agent that ran in the terminal. Built using o3 and o4-mini, it could read files, write code, run tests, and commit to Git — OpenAI's answer to Claude Code. Where Claude Code was a closed product, Codex CLI was open-source under the Apache 2.0 license.

By the fall of 2025, Codex had evolved further. GPT-5-Codex, a model specifically optimized for agentic coding, arrived in October. In February 2026, OpenAI released the Codex desktop app — and the ambitions became unmistakable. The app introduced worktrees (each agent working on an isolated copy of the codebase), Skills (bundled instructions, scripts, and resources extending Codex beyond code generation), and Automations (scheduled background tasks that ran continuously without human prompting).

OpenAI stated the thesis explicitly: "Codex is built on a simple premise: everything is controlled by code. The better an agent is at reasoning about and producing code, the more capable it becomes across all forms of technical and knowledge work."

Sam watched this evolution with fascination. It was the clearest expression of a belief he had been forming independently: the most capable general agents would emerge from coding agents. Anthropic was pursuing the same thesis from the other direction — starting with a coding agent (Claude Code) and generalizing it into a platform for building any kind of agent (the Claude Agent SDK). OpenAI started with a general model and specialized it into a coding surface that was becoming general again.

Both arrived at the same place: the coding agent as the prototype for the general agent.

He noted in his memo: *The agent ecosystem is converging on the coding agent as the canonical form. The best general agent in two years will look a lot like the best coding agent today.*

---

The Microsoft story deserves attention not for its technical choices but for what it revealed about organizational evolution in the agent era.

For two years, Microsoft had maintained two competing agent frameworks. AutoGen, born in Microsoft Research, was experimental and innovative — it pioneered multi-agent patterns like group chat, reflection, and dynamic workflow generation. Semantic Kernel, built by the Azure engineering team, was stable and enterprise-ready — it provided session management, type safety, telemetry, and deep integration with Microsoft's identity and compliance infrastructure.

In October 2025, Microsoft merged them into the **Microsoft Agent Framework**. Both original frameworks entered maintenance mode. The merger reflected a broader industry recognition that the age of framework experimentation was ending. What developers needed was not more experimentation but more reliability: production SLAs, enterprise security, long-term support.

The features Microsoft was leading with in its merged framework were not orchestration features. They were trust features: OpenTelemetry observability, Entra ID authentication, prompt injection protection, PII detection, task adherence monitoring.

Microsoft was competing on the trust layer, even if it wasn't using that term. It would not be the last.

---

## Act 2: The Architecture

At the end of Sam's memo, he included a decision framework organized not around features but around binding constraints:

*If your constraint is **time to prototype**, the Claude Agent SDK wins — built-in tools mean you can have a working agent in minutes.*

*If your constraint is **model flexibility**, the OpenAI Agents SDK or Google ADK wins — both are model-agnostic.*

*If your constraint is **enterprise compliance**, Microsoft Agent Framework wins — designed from the ground up for auditability, governance, and Azure security integration.*

*If your constraint is **production durability**, LangGraph wins — checkpointing, human-in-the-loop, and durable execution are core, not bolted on.*

*If your constraint is **operational security**, AgentCore wins — microVM isolation, workload identity, and policy enforcement at the runtime level.*

The mistake most teams make, Sam wrote, is choosing based on the demo — how fast can I get Hello World working? — rather than the production requirements. The framework that is easiest to start with is rarely the one that is easiest to operate.

Sam attached an appendix to the memo with a line that would show up, months later, in Nadia's own evaluation document: *The framework you can defend is the framework worth choosing.*

### AgentCore as a Separate Category

AgentCore does not fit in the SDK comparison because it is not an SDK. It is a runtime platform that hosts agents built on any SDK.

Its architecture provides:

- **Session isolation**: Each user session runs in a dedicated Firecracker microVM with isolated CPU, memory, and filesystem. Sessions last up to eight hours.
- **Framework agnosticism**: Supports agents built on Python frameworks — LangGraph, Strands, CrewAI, OpenAI Agents SDK, Google ADK, custom code.
- **Identity integration**: Assigns unique workload identity per agent. Cryptographically binds user tokens to agent workload identity, preventing cross-tenant token access.
- **MCP and A2A support**: Integrates both protocols within the runtime's security boundary.
- **Observability**: Built-in tracing, OpenTelemetry-compatible, exports to CloudWatch and external providers.

This separation of framework (orchestration) from runtime (execution and trust) was the architectural direction Sam bet on. It allowed the trust layer to evolve independently of the orchestration layer — and the trust layer, he was convinced, was where the industry's underinvestment would eventually be exposed.

The next chapter is about the first place it was exposed: the body problem.

---

*Next: [Chapter 5 — The Body Problem](/book/chapter-05-the-body-problem/)*
