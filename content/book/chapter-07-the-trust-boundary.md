---
title: "Chapter 7: The Trust Boundary"
description: "Identity, policy, and the question of who the agent is acting for"
date: 2026-04-07
weight: 7
tags: ["trust", "identity", "policy", "governance"]
---

## Identity, policy, and the question of who the agent is acting for

---

## Act 1: The Story

Nadia Okafor stopped trying to persuade her security director twenty minutes into the meeting. She had come in with a forty-slide deck about agent governance. She closed the laptop after slide eleven.

"Let me ask you a different question," she said. "If I told you that next week, a system in this building was going to send emails to our customers, query our core banking records, and take actions against external APIs — all without a human in the loop on every action — what would you need from me before you'd let that system run?"

The director, a man in his fifties who had spent a long career saying no to things before saying yes to them, considered the question for longer than was comfortable.

"I would need to know who it was acting for. I would need a log of what it did. And I would need to know that your team could turn it off before I could."

"What else?"

"I would need policies I could read. Not prompts. Policies."

"Why policies and not prompts?"

"Because prompts are preferences. Policies are rules. I need rules."

Nadia wrote this down. It was the sentence she had been looking for — not in the deck, not in the vendor literature, not in the half-dozen regulator guidance documents she had read over Christmas. *Prompts are preferences. Policies are rules.*

"That is what I am trying to build," she said. "Will you help me define what good looks like?"

He did not say yes. He said, "Send me what you have and I will mark it up."

It was January 2026. It was, in her experience, how things started moving at the bank.

---

Three days later, Sam Kovic was on a video call with Nadia from San Francisco.

"I want to show you something I'm stuck on," Nadia said. "Not the framework. The identity model. Specifically, token binding."

"Go ahead."

"AgentCore gives you workload identity per agent, and the docs say it cryptographically binds the user's token to the agent's workload identity. I believe the claim. I want to understand what happens when an agent delegates to another agent. If Agent A is acting for me, and Agent A calls Agent B to do part of the work, is Agent B acting for me, or for Agent A?"

Sam thought about this. "AgentCore's current model is that B inherits A's user token, so B is acting for you. But the audit trail shows two workload identities. A's identity invoked B. B's identity called the tool. Both are tied to your user identity at the top of the chain."

"And if B calls C?"

"Same pattern. Chain of workload identities, single user identity at the root."

"What if C is outside my organization?"

"That's the part the docs don't answer cleanly. A2A — the agent-to-agent protocol — is supposed to carry identity across the boundary. I don't think the implementations are there yet."

Nadia was quiet. "So if I deploy a multi-agent system today, and one of those agents reaches out to a partner's agent, I cannot cleanly attribute the action at the partner's system to a specific user in mine."

"Not cleanly. You can attribute it to your *agent*. Whether that's enough depends on who's asking."

"It's not enough for a regulator."

"I know."

"What do I do?"

"You restrict your multi-agent systems to within your own trust boundary. No cross-organization delegation until the identity story is better. Build as if the protocol will never fix this, and if the protocol does, you loosen."

She wrote that down too.

---

David Almeida, in London, had been having the same conversation from the other side.

In late January, David's team held a working session about the next protocol revision. The topic was a proposal to bring tenant identity into the protocol itself — to require that every MCP request carry a tenant identifier, signed in a way the server could verify. Half the team supported it. The other half did not.

David's counterpart on the interoperability team argued against it: "If we put tenant identity in the protocol as required, we break every existing server. Thousands of them. Servers in hobbyist GitHub repos, servers in enterprise deployments, servers written in seven programming languages by people who will never read our changelog. We will fragment the ecosystem to fix a problem that should be fixed at the implementation layer."

"The implementation layer keeps failing to fix it."

"Then it will keep failing. That is a signal to provide better guidance, better reference implementations, better linters. It is not a signal to break the protocol."

David did not win the argument that afternoon. The team agreed to add tenant identity as an *optional* extension, documented in the specification, supported by reference implementations, and expected to become required in some future revision that nobody would commit to scheduling. It was a classic protocol compromise: better than nothing, worse than necessary, defensible.

David decided he would tell Nadia the truth: the protocol was going to continue under-specifying trust for at least another year, and her framework was going to have to assume that the protocol would not help her.

---

Over the following three weeks, Nadia circulated drafts of her framework to her security director, her compliance lead, two people on her engineering leadership team, and — at Sam's urging — to Marcus Chen. Marcus responded on a Saturday morning with twenty-three comments, most of them uncomfortable.

Comment seven: *You describe "cryptographic binding" but do not specify the threat model. Binding against what? A confused deputy inside your own app? A compromised vendor? An insider? These require different guarantees.*

Comment twelve: *Policy outside the reasoning loop is necessary but your draft implies it is sufficient. Cedar does not know what the agent means, only what the agent is about to do. A well-instrumented agent can describe its intended action honestly and still be manipulated into performing it. The policy engine is a fence, not a judge.*

Comment nineteen: *The HITL section treats "consequences of being wrong" as if it were an assessable property of each action. In practice, operators cannot assess this in advance for novel actions. You need a fallback posture for the action the operator did not anticipate.*

Comment twenty-three: *You are writing this framework as if it will be adopted. It will not be adopted in the form you wrote it. The useful question is: which parts will survive contact with actual deployment, and which will be discarded under production pressure? Design the framework so that the parts that get discarded are not the parts that were protecting you.*

Nadia read this last comment four times. Then she revised the framework's structure. She split it into a *mandatory core* — the smallest set of guarantees that could not be compromised — and an *extended guidance* layer that captured everything else. The mandatory core had seven items. If any one of them was missing, the deployment did not proceed.

She sent the revision to Marcus, Sam, and her security director on the same email. Sam replied within an hour: *The mandatory core is right. This is the version I would defend.* Marcus replied the next day: *Better. Still incomplete. Ship it.* Her security director replied three days later with nine additional comments, all of which she accepted.

The framework was not a standard. It was a working document, used inside one company, circulated among a small network of people who were trying to solve the same problem at other companies. Sam began using its mandatory core as a checklist in his own deployment reviews. A colleague of Marcus's at a Seattle hospital adopted a modified version. A bank in Singapore asked Nadia for a copy after a chance conversation at a conference.

Elena read the mandatory core the week Sam sent it to her. She printed it and marked it up. Of the seven items, her rebuild already satisfied three. She added the other four to her engineering plan.

---

## Act 2: The Architecture

### The Three Identities, and What Binding Actually Means

Every agent action involves three identities. **User identity** — typically an OAuth 2.0 or OIDC token representing the human on whose behalf the agent acts. **Agent identity** — a workload identity scoped to a specific agent instance, often bound to a specific session or task. **Organizational identity** — the tenant the agent is operating within, which determines which data is in scope and which policies apply.

The architectural commitment is not merely that all three identities exist. It is that they are cryptographically bound at the infrastructure layer, below the application, such that the application cannot present an inconsistent combination even by accident.

Concretely, in AgentCore Identity: when a user initiates an agent session, the user's OAuth token is issued bound to a specific workload identity for the session's agent. The binding is enforced by the gateway: subsequent tool calls that present the user token must also present a signed assertion from the matching workload identity, or they are rejected before reaching any application code.

The property this buys is narrow but important. It does not prevent an agent from misusing its legitimate access. It does prevent an application bug — a cache key typo, a retrieval filter applied at the wrong layer, a debug tool re-enabled by an environment variable — from causing one user's context to reach another user's session. Elena's November incident was possible because her isolation was enforced at the retrieval filter; binding at the chunk-access layer would have made the debug tool return nothing rather than return the wrong thing.

What it does not solve, and what no current implementation solves well, is cross-organizational delegation. When an agent in Organization A invokes an agent in Organization B through A2A, the identity story becomes ambiguous. Nadia's framework handles this by forbidding cross-organizational multi-agent delegation within its mandatory core. Sam handles it by refusing customer requests that require it.

### Policy as Enforcement, Not Instruction

The second architectural commitment is that policy is enforced by an engine outside the agent's reasoning loop, not by instructions inside the prompt.

AWS Cedar is the most prominent implementation. Cedar policies are declarative statements written in a formal policy language, compiled, versioned, and evaluated by a policy decision point that sits between the agent and its tools. The agent's prompt cannot override the decision.

Open Policy Agent (OPA) fills the same role in ecosystems outside AWS. Both treat policy as code — reviewable, testable, versioned, deployed through a pipeline separate from the agent's own code or prompts.

Marcus's twelfth comment on Nadia's draft captures the limit of this architecture. Policy engines enforce on properties they can evaluate — identity, action type, resource, destination. They do not evaluate intent. An agent that has been manipulated into using a legitimate tool for an illegitimate purpose — sending an email to an approved address with exfiltrated content embedded in it — is not caught by policy. Policy is a fence. It is not a judge. The fences must be drawn knowing this.

### The Human-in-the-Loop Spectrum

The spectrum's poles are visible in products. Claude Code, by default, requests approval on every Bash invocation, every file write, every destructive action — roughly one approval per minute of active coding. Codex cloud accepts a task, runs for minutes to hours in an isolated sandbox, and returns a result the operator reviews asynchronously.

Between these poles lies most production deployment. A customer-support agent may auto-send responses below a confidence threshold while escalating edge cases. A financial-analysis agent may execute read-only queries freely while requiring approval for any action that modifies data.

Nadia's framework requires this declaration to be written per action-class, versioned, and tied to policy. It requires a *fallback posture* for actions not anticipated at declaration time: if an agent proposes an action that does not match any declared class, the default is synchronous human review.

The two failure modes at the edges of the spectrum are well-documented. Over-approval produces fatigue: operators approve reflexively, the approval becomes theater, the safety property disappears. Under-approval produces drift: incremental autonomy expansion without explicit review, until the agent is taking actions no one would have approved if asked at the beginning.

### The Four Layers of Trust

A diagnostic taxonomy used by Nadia's framework: agent trust decomposes into four layers.

**Model trust** — will the model follow instructions under adversarial input? Established through evaluation, red-teaming, and the operator's judgment of model suitability.

**Tool trust** — are the tools the agent calls safe and authentic? Established through package signing, integrity verification, provenance, and audit of tool invocations. The Asana incident was a tool-trust failure at the multi-tenant layer.

**Data trust** — is the context the agent consumes clean of injection? Established through provenance tracking on retrieved content, explicit trust labels on untrusted input (web content, user messages, external documents).

**Identity trust** — is the agent acting for who it claims? Established through the binding architecture described above, and through audit infrastructure that makes every action attributable to a user, an agent, and a tenant.

The diagnostic use: in any incident, identify which layer failed. Elena's November incident was a tool-trust failure (a debug tool shipped to production) compounded by an identity-trust failure (no tenant binding at the chunk layer). Asana was a tool-trust failure at the tenant boundary.

A deployment that achieves three layers and fails at the fourth is compromised. The taxonomy does not prevent failures; it makes them legible.

### The Shape of the Gap

What remains, even with binding, policy, declared HITL, and the four-layer diagnostic, is a gap the framework does not close. Agent trust architecture assumes an operator who can define policy, declare action classes, review audit trails, and defend deployment decisions. It assumes an organization.

A large fraction of the agents deployed in 2026 do not have an organization. They run on individual developers' laptops, in personal assistant deployments, on home servers. Their operators are individuals, often with no security background, following install instructions from GitHub README files.

Marcus had been scanning the public internet for these instances since November. He was in the process of writing up what he had found. The framework Nadia had written did not describe the world Marcus was documenting. A different architecture, or a different failure pattern, governed there.

The next chapter is about what Marcus found.

---

*Next: [Chapter 8 — The Personal Agent](/book/chapter-08-the-personal-agent/)*
