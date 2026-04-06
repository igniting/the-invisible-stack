---
title: "Chapter 6: What Goes Wrong"
description: "Failure patterns, antipatterns, and the lessons nobody writes blog posts about"
date: 2026-04-06
weight: 6
linkTitle: "6. What Goes Wrong"
tags: ["failures", "security", "incidents", "tenant-isolation"]
---

## Failure patterns, antipatterns, and the lessons nobody writes blog posts about

---

## Act 1: The Story

At 6:47 AM on a Thursday in November 2025, Nadia Okafor was on her second cup of coffee when her phone rang.

It was her Head of Information Security. He did not call at 6:47 AM unless something was wrong.

"Our DLP flagged an outbound email from the support team last night," he said. "Standard pattern — an agent-generated response to a customer inquiry. The email contained what looks like fragments of customer data that don't belong to that ticket."

Nadia set down her coffee. "Whose data?"

"That's the issue. The fragments don't match any of our customers. They match data from what looks like a *different company's* tickets. Names, partial order IDs, email addresses."

Nadia's company had been piloting an AI-powered customer support agent from a startup — Elena Voss's startup — for six weeks. The pilot was limited to a single support team, handling low-risk customer inquiries for one product line. Nadia had insisted on restrictions. She had insisted on DLP scanning of all agent-generated external communications, because she did not trust the vendor's trust infrastructure. Her CISO had called this paranoid. This morning, it looked prescient.

"Pull the pilot," Nadia said. "Immediately. Disconnect the agent. I want logs of every external communication the agent has sent since the pilot started."

"Already disconnecting."

"And call the vendor. I want their CTO on the phone within the hour."

---

Elena Voss was in Berlin, three hours ahead of London, walking to the office when she saw the message from Nadia's security team. The subject line read: *URGENT: Suspected Cross-Tenant Data Leak — Immediate Response Required*.

She read the email standing on the sidewalk. Then she read it again. Then she ran the rest of the way to the office.

By the time she was at her desk, her phone was ringing. It was Nadia.

"Elena. We have a situation."

Elena tried to sound calm. "I just saw the email. I'm pulling up logs now. Tell me exactly what you're seeing."

Nadia told her. An outbound email, composed by Elena's agent, sent to a customer in London, containing three lines of content that mentioned a person nobody at Nadia's company had ever heard of, referencing an order number that did not match any of their order number formats, signed with a customer service representative's name from — Nadia had checked — one of Elena's other pilot customers, a French logistics company.

Elena knew what that meant before Nadia finished describing it.

"I'm going to investigate immediately," Elena said. "I will have a root cause analysis for you by end of day. I will be completely transparent about what happened. I'm — I'm very sorry, Nadia."

"I appreciate that, Elena. But you should understand: the pilot is suspended indefinitely. Whether we resume depends entirely on what your investigation finds and whether I can defend the answer to my board and to our regulators. If this is what it looks like, you have a material breach of our data processing agreement. I need you to understand the seriousness of that."

"I understand."

"I need to go. I have to brief my CEO."

The call ended. Elena stared at her laptop screen. Then she called Sam.

---

Sam Kovic was in San Francisco, where it was not quite 10 PM the night before. He picked up on the second ring.

"Sam, I need your help. I think we have a cross-tenant data leak."

Sam sat up. "Tell me what you know."

Elena told him. Sam asked her three questions.

"One. How are you isolating customer data in your vector store?"

"Namespaced by customer ID. Each customer's tickets are tagged with their tenant ID, and we filter by tenant ID on retrieval."

"Two. Where is the tenant ID filter applied — in the retrieval query, or after retrieval?"

Elena was quiet for a moment. "In the query. I think."

"You think."

"I'm pulling up the code."

"Three. When you do retrieval-augmented generation for a customer ticket, does the agent see retrieved chunks before or after the filter is applied?"

"Before."

"What?"

"Before. The agent — we send it a retrieval request, it gets back chunks, it decides which ones to use in its response."

"Elena. The agent is seeing chunks from other tenants *before* you filter by tenant ID."

"No. We filter. I'm sure we filter."

"Pull up the retrieval code."

She pulled it up. Then she was quiet for a very long time.

---

What Elena found, over the next four hours, was the following.

Her retrieval pipeline had two stages. In the first stage, a semantic search against the vector store returned the top twenty chunks most similar to the current customer's query. In the second stage, a filter was applied to keep only the chunks belonging to the current customer's tenant.

The filter worked. On the chunks that reached the final prompt, the filter was correct.

But Elena's team, three months earlier, had added an instrumentation layer. The instrumentation was meant to help them debug retrieval quality. It logged every candidate chunk returned by the semantic search — *before* the filter — along with metadata about which chunks were kept and which were discarded. The logs were written to a debug table in their main application database.

That would have been fine, except for one other thing.

Six weeks earlier, a different engineer on Elena's team had added a new tool to the agent: `get_retrieval_debug_context`. The tool had been added for an internal dogfooding experiment, to help the team understand why the agent was sometimes choosing unhelpful context. The tool was meant to be disabled in production. It had been disabled in production. For three weeks.

Then, in late October, an unrelated deployment had reset an environment variable. The tool had come back online. Nobody had noticed, because the tool was only called when the model decided to call it, and the model had not decided to call it for the first ten days after the deployment.

Then, on a Tuesday in early November, the model had called it. While processing a ticket for a London customer, the agent had apparently decided it needed "more context" and had called `get_retrieval_debug_context`. The tool had returned the full pre-filter candidate set — twenty chunks, from any tenant, semantically relevant to the customer's query. The agent had dutifully incorporated the most relevant chunks into its response, not knowing that some of those chunks came from a French logistics company's tickets.

The agent had not escaped the sandbox. The agent had not been manipulated by a prompt injection. The agent had done exactly what it was designed to do: look at its available tools, decide which ones were relevant to its task, call them, and use the results.

The failure was not the agent's. The failure was in the architecture around the agent. A debug tool, shipped to production by accident, had handed the agent an API that violated the tenant isolation guarantee that the rest of the system had been designed to enforce.

Elena wrote this up, honestly, in a six-page root cause analysis, and sent it to Nadia by 6 PM London time.

---

Nadia read the RCA twice. Then she forwarded it to Marcus Chen, with a short note: *You consulted on our pilot approval. I'd like your independent read on this vendor's security posture. How bad is it?*

Marcus read the RCA the next morning. Then he called Sam.

"Sam. Your friend Elena's company. I'm about to write a very unflattering security assessment of her platform for Nadia. I wanted to give you a heads-up."

"I appreciate that."

"The root cause analysis is honest. That helps her. But the architecture underneath is worse than the incident suggests. A debug tool got shipped to production and nobody noticed for three weeks. That's a symptom, not a root cause. The root cause is that her company doesn't have the processes or the architecture to prevent this class of failure. They don't have a tool registry with production allowlists. They don't have per-tenant isolation at the retrieval layer. They don't have an audit of what tools are exposed in which environments. The agent is the symptom. The harness is the problem."

"I know."

"I'm going to say that to Nadia. I'm going to recommend against resuming the pilot."

"I understand."

"Sam, one other thing. I think you should offer to help her rebuild. Her team is going to need someone who's done this before. You are that person."

"Why are you telling me this?"

"Because the alternative is that her company dies, and the ecosystem loses a founder who has now learned — very expensively — the exact lesson the rest of us are trying to teach. That's a waste. Help her rebuild. She'll do it right the second time."

---

Marcus's assessment to Nadia was four pages long, technically precise, and devastating. It identified nine distinct weaknesses in Elena's platform architecture, ranked by severity. It described the cross-tenant leak as "the surface manifestation of a systemic isolation failure" and recommended that Nadia not resume the pilot until Elena's company had rebuilt its retrieval and tool-exposure architecture from the ground up.

Nadia forwarded the assessment to Elena the same day, with one sentence: *I am sharing this so you understand what it will take for us to consider resuming.*

Elena read it three times. Then she forwarded it to her co-founder and to her head of engineering, with a meeting invite titled *Rebuild Plan — Monday 9 AM*. Then she called Sam.

"Sam. Will you help me?"

"Yes."

"I don't know where to start."

"Start by admitting you need to rebuild, not patch. You've been patching for eighteen months. This is the moment the patching stops."

"Okay."

"Then tell your team the same thing. Then tell your board. Then we'll build the architecture you should have built in the first place."

"How long will it take?"

"Three months to get to a production-ready trust architecture. Six months to rebuild the retrieval layer. You will lose customers in that time. You will burn a significant amount of your runway. The question is whether you survive the rebuild."

"I don't know if we survive the rebuild."

"Then you had better start the rebuild today."

---

Four weeks later, in mid-December, Nadia received an email from Elena. The subject line read: *Architecture rebuild — progress update and request for 30-minute review call*.

The email was not a sales email. It did not ask to resume the pilot. It described what Elena's team had done in four weeks: removed all debug tools from production, introduced a tool registry with per-environment allowlists, rebuilt the retrieval layer to apply tenant isolation at the query level rather than the post-retrieval filter level, added per-tenant namespacing in the vector store with hard cryptographic boundaries, introduced an audit log of every tool invocation with tenant ID and request ID, and engaged an external security firm to conduct a second assessment.

The email ended: *I am not asking to resume the pilot. I am asking for thirty minutes of your time to show you the architecture, get your feedback, and learn what else we need to do. If the answer is no, I understand. I am writing because you were the customer who caught the incident, and I want to earn back the right to your trust by showing you the work, not by asking for it.*

Nadia took the call.

Whether Elena's company survived was not yet clear, as of December 2025. But the email was the first moment Nadia had felt that Elena understood what had gone wrong. Not the incident. The architecture. The *harness*.

That was the moment the lesson of the book landed on Elena for the first time. It had cost her a contract, a quarter of her runway, and — for a few weeks in November — her conviction that she knew what she was doing.

It was worth every euro.

---

## Act 2: The Architecture

Elena's story contains, in microcosm, almost every category of harness failure the industry has produced. The failures divide into four families, which map directly onto the three layers of the harness plus a fourth category that cuts across all three.

### Family 1: Orchestration Failures

**The context bloat failure.** An agent runs for too many turns, accumulating context until the model's attention degrades and its responses become incoherent. Fix: context engineering as a first-class concern, with explicit turn budgets and summarization checkpoints.

**The infinite loop failure.** An agent gets stuck in a pattern where it repeatedly calls the same tool with slightly different arguments, never converging. Fix: loop detection at the harness level. The harness should track tool-call patterns and break loops even when the agent does not recognize them as loops.

**The silent drift failure.** An agent's behavior changes over time as the underlying model is updated, as tool definitions are modified, or as the system prompt is edited by well-meaning engineers. Fix: agent evaluation as continuous integration. Every model update, prompt change, and tool modification should trigger a regression suite.

**The reasoning failure.** An agent arrives at a wrong conclusion through a chain of reasoning that looks plausible at each step but compounds errors across steps. Fix: human-in-the-loop checkpoints for high-stakes decisions, critic models for lower-stakes ones.

### Family 2: Execution Failures

**The sandbox escape failure.** The agent, through creative use of its tools, finds a path that bypasses its intended isolation. It does not require the model to be adversarial — only that the model be goal-directed and that the sandbox have any reachable bypass. Fix: hardware-level isolation for anything touching untrusted input, and the operating assumption that the agent will attempt to route around any restriction it perceives as an obstacle.

**The permission sprawl failure.** An agent accumulates permissions over time, as new capabilities are added, and nobody audits the cumulative set. Fix: permission manifests, capability-based security, and reviews of the complete permission set each time a new tool is added.

**The supply chain failure.** An agent's tool, skill, or MCP server is backdoored through a compromised package. Fix: signed packages, SHA pinning, allowlists of trusted sources, and per-org package allowlists with integrity enforcement.

**The credential exposure failure.** An agent has access to secrets that end up in places they should not, through prompt context or log aggregation. Fix: secrets treated as injected runtime values, never placed in the agent's prompt context.

### Family 3: Trust Failures

**The cross-tenant leakage failure.** Data from one tenant reaches another tenant through an imperfectly enforced isolation boundary. Elena's case is the canonical example. Fix: isolation enforced at every layer that touches tenant data, with explicit tenant-ID propagation through every function call, every log entry, every tool invocation. *Defense in depth, applied consistently.*

**The prompt injection failure.** An agent processes untrusted input that contains instructions designed to manipulate the agent into taking an action the principal did not authorize. Fix: input provenance tracking, explicit distinction between trusted and untrusted content in the prompt, and human confirmation requirements that cannot be bypassed by the agent regardless of what the input contains.

**The audit gap failure.** An incident occurs, and the team cannot reconstruct what the agent did, why it did it, or what inputs led to its decision. Fix: structured traces of every agent turn — inputs, tool calls, tool results, model responses — stored durably, indexed by session ID and tenant ID, queryable by security teams without engineering intervention.

**The policy drift failure.** An agent's behavior gradually deviates from the policies it is supposed to enforce, because the policies live in prompts rather than in code. Fix: policy engines (Cedar, OPA) that enforce rules outside the agent's control, so that the agent *cannot* violate the policy regardless of what the prompt instructs it to do.

### Family 4: Economic Failures

**The retry storm failure.** An agent fails a tool call, retries, fails again, retries again. The retries compound across multiple agents running in parallel. Fix: per-session token budgets, per-tenant daily budgets, and alerting on cost anomalies before they become cost disasters.

**The compound cost of failure.** When an agent fails, the organization pays four times: once for the model tokens of the failed run, once for the engineering time to diagnose the failure, once for the customer impact, and once for the loss of trust that makes the next deployment slower. Fix: recognize that investment in orchestration, execution, and trust infrastructure is not overhead — it is the thing that keeps the compound cost of failure bounded.

### The Meta-Pattern

These four families share a common structure. In every case:

1. The agent does something that, from its local perspective, is reasonable.
2. The harness lacks a constraint, an observation, or an enforcement mechanism that would have caught the agent's action.
3. The consequence propagates to a part of the system the agent did not know existed.
4. The failure is only detected by an external observer — a DLP scanner, a customer complaint, a regulator, an invoice.

The common diagnosis is that the harness assumed the agent knew things it did not know, or would behave in ways it did not behave, or would stay within boundaries it did not stay within. The common fix is to move the constraint from the agent's judgment to the harness's enforcement — to treat the agent as a capable but untrustworthy component whose actions must be bounded by infrastructure, not by instruction.

Elena's incident sits squarely in this pattern. Her agent did something reasonable (called an available tool to get more context). Her harness lacked a constraint (no production allowlist on the debug tool). The consequence propagated (cross-tenant data reached a customer email). The failure was detected by an external observer (Nadia's DLP scanner). The fix was to move constraints from the agent's judgment to the harness's enforcement.

This is the architecture of trust.

---

*Next: [Chapter 7 — The Trust Boundary](/book/chapter-07-the-trust-boundary/)*
