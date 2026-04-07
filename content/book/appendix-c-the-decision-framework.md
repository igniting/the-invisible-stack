---
title: "Appendix C: The Decision Framework"
description: "A structured guide for evaluating and selecting agent frameworks and trust infrastructure"
date: 2026-04-01
weight: 13
tags: ["reference", "decision", "framework", "evaluation"]
---

A structured evaluation guide for teams choosing an agent framework, designing trust infrastructure, or assessing existing deployments. Use this alongside the three diagnostic questions and maturity model from Chapter 10.

---

## Part 1: Choosing an Orchestration Framework

### Step 1 — Characterize the task

| Question | If yes → | If no → |
|---|---|---|
| Does the task require persistent state across multiple turns? | Consider LangGraph or Claude Agent SDK | OpenAI Agents SDK or simple loop may suffice |
| Does the task require deterministic control flow with occasional LLM routing? | Consider Google ADK or Microsoft Agent Framework | Prefer LLM-driven orchestration |
| Does the task require human review before high-stakes actions? | Require HITL-capable framework (LangGraph checkpoints, Claude Agent SDK permission hooks) | Any framework |
| Is the deployment context enterprise with existing Azure/Microsoft stack? | Microsoft Agent Framework reduces integration burden | Framework choice is vendor-neutral |
| Is the team already operating Python-native infrastructure at scale? | LangGraph, Claude Agent SDK, Google ADK all fit | Consider TypeScript-native OpenAI Agents SDK |

### Step 2 — Apply the three binding questions

**What does the agent come equipped with?**

List every capability the agent will have: tools, MCP servers, memory access, network egress, filesystem paths. For each capability, ask: does the agent need this to complete the task, or does it need it because removing it would be inconvenient?

Strip every capability that fails this test. The attack surface of an agent is the set of capabilities it has been granted; reducing it before deployment is cheaper than recovering from it afterward.

**Who owns the execution loop?**

| Owner | Consequence |
|---|---|
| The framework vendor | You depend on their persistence, HITL, and audit semantics. Verify they match your requirements. |
| Your infrastructure | You own the operational burden. You also own the audit trail. |
| The model | No meaningful answer to accountability questions. Not acceptable for production. |

**Is the harness the runtime, or is the runtime separate?**

If the answer is "the harness is the runtime" — the framework manages execution, isolation, and persistence — ask what happens when you need to change frameworks. If the answer is "the runtime is separate" (AgentCore, a cloud function, your own container), you have portability and can upgrade the orchestration layer without migrating the trust layer.

### Step 3 — Rate against the maturity model

Determine which level your deployment target requires (see Chapter 10, Act 2), then verify the framework supports it:

| Required Level | Minimum Framework Capabilities |
|---|---|
| Level 1 (Supervised Production) | HITL checkpoint mechanism; structured output for audit |
| Level 2 (Sandboxed Autonomy) | Execution isolation or external runtime; declared HITL gates |
| Level 3 (Trusted Autonomy) | Cryptographic identity binding; policy enforcement outside reasoning loop; tenant-controlled audit |
| Level 4 (Self-Improving) | Provenance tracking for learned behaviors; rollback; sandbox for self-authoring |

No major vendor ships Level 3 or above out of the box as of early 2026.

---

## Part 2: Designing the Trust Layer

### The four trust layers

These four layers should be present in every production deployment. They are independent of orchestration framework choice.

**Layer 1: Model Trust**

The agent follows instructions from the system prompt. This is the weakest trust mechanism — it is advisory, not enforced. Use it for: capability declaration, persona definition, default behavior specification. Do not use it as the sole enforcement mechanism for any security property.

*Checklist:*
- [ ] System prompt reviewed and version-controlled
- [ ] Prompt injection mitigations in place (input sanitization, output parsing)
- [ ] System prompt does not contain secrets or credentials

**Layer 2: Tool Trust**

Tools define what the agent can physically do. Limit the tool surface to what the task requires. Use explicit allowlists at the framework level, not the model level. Tools should be the mechanism through which the model exercises authority, not the mechanism through which it is granted authority.

*Checklist:*
- [ ] Tool allowlist declared explicitly (not implicit)
- [ ] Each tool has a declared scope (filesystem paths, network destinations, database tables)
- [ ] Tool invocations logged with inputs and outputs
- [ ] No tool provides write access to its own logs or audit trail
- [ ] MCP server identity verified before trust is extended

**Layer 3: Data Trust**

Tenant isolation must be enforced below the application layer. Application-level isolation (prompt instructions, application code) is insufficient — it relies on the model not making mistakes, and on the application code not having bugs. Isolation below the application layer means the data is physically inaccessible to the wrong tenant even if the application layer is compromised.

*Checklist:*
- [ ] Tenant isolation enforced at storage layer (separate databases, cryptographic key separation, or equivalent)
- [ ] Retrieval pipeline tags each chunk with tenant identity before storage
- [ ] Retrieval pipeline filters by tenant identity as a hard constraint, not a preference
- [ ] Cross-tenant queries are architecturally impossible, not just disabled by configuration

**Layer 4: Identity Trust**

Every action taken by an agent should be attributable to: (a) the user who initiated the session, (b) the agent instance that took the action, and (c) the tenant to which the data belongs. This attribution must be enforced cryptographically, not by convention.

*Checklist:*
- [ ] Agent sessions are cryptographically bound to authenticated user identity
- [ ] Agent instance has a workload identity distinct from user identity
- [ ] Tenant binding is cryptographic, not application-level
- [ ] Audit log is append-only and tenant-controlled (not vendor-controlled)
- [ ] Audit log records: user, agent, tenant, tool called, inputs, outputs, timestamp

### The HITL spectrum

Human-in-the-loop is not binary. Design the appropriate intervention point for each action class:

| Action Class | Recommended HITL Gate |
|---|---|
| Read-only retrieval | No gate required |
| Write within same tenant | Audit log only |
| Write with external side effects (email, calendar) | Soft approval (notify and allow cancellation window) |
| Write with financial consequences | Hard approval before execution |
| Write affecting other tenants or users | Hard approval + compliance review |
| Irreversible actions | Hard approval + second approver |

Gates should be enforced at the execution layer, not the orchestration layer. An orchestration-layer gate can be bypassed by model error or prompt injection. An execution-layer gate cannot.

---

## Part 3: Evaluating an Existing Deployment

Use this checklist to assess a deployment already in production.

### The three diagnostic questions (operationalized)

**1. Where is the trust boundary?**

- [ ] Can you list every capability the agent currently has?
- [ ] Is the enforcement of that boundary below the model's reasoning (execution layer) or inside it (prompt instructions)?
- [ ] If the model made a mistake in its reasoning, could it exceed its intended authority?
- [ ] Is there a human gate before any irreversible action?

**2. Who is accountable, and can you prove it?**

- [ ] When an agent action caused a problem last month, could you produce a complete audit trail in under four hours?
- [ ] Does the audit trail include the inputs the model received, not just the outputs it produced?
- [ ] Is the audit trail stored in infrastructure you control, not only in vendor observability tools?
- [ ] Can you attribute every agent action to a specific user and a specific tenant?

**3. What compounds?**

- [ ] Does the agent learn from its own outputs (skills, memory, fine-tuning)?
- [ ] If yes, is every learned artifact human-reviewed before production use?
- [ ] Can you enumerate what the agent has learned since deployment?
- [ ] Can you roll back a learned behavior that was endorsed in error?

### The security questionnaire map

Enterprise security teams will ask the following. Have answers prepared before the first pilot.

| Security question | Where the answer lives |
|---|---|
| "How is the agent authenticated?" | Identity layer — user identity binding mechanism |
| "How is data isolation enforced between customers?" | Data layer — tenant isolation architecture |
| "Who can see customer data?" | Data layer — access controls + audit log |
| "What can the agent do without human approval?" | Tool layer — allowlist + HITL gates |
| "What do you log and for how long?" | Audit infrastructure — retention policy |
| "What happens if the agent makes a mistake?" | Incident response + rollback plan |
| "Has this been penetration tested?" | Security assessment — provide report |
| "Who is responsible if the agent causes a data breach?" | Legal + contractual — data processing agreement |

---

## Part 4: Skills Governance

### For human-authored skills

- [ ] Skills are stored in version control
- [ ] Skills require review before merging to production
- [ ] Each skill declares its activation criterion (when to use), side effects (what it will do), and dependencies (what tools it calls)
- [ ] Skills are tested in staging before promotion
- [ ] Deprecated skills are removed, not left dormant

### For agent-generated skills (if permitted)

**Recommendation: Do not permit agent-generated skills in production deployments where auditability is required. If permitted:**

- [ ] Agent-generated skills are quarantined to a review branch, not loaded automatically
- [ ] Every agent-generated skill requires human review before promotion to production
- [ ] Agent-generated skills include provenance metadata: which agent, which session, which task
- [ ] A rollback mechanism exists for skills promoted in error

### MCP server governance

- [ ] MCP server identity is verified before tools are invoked (OAuth 2.1 or equivalent)
- [ ] Each MCP server is listed in an explicit allowlist
- [ ] MCP server connections are logged
- [ ] MCP servers sourced from third parties are sandboxed or reviewed before use

---

## Part 5: The Vendor Evaluation Matrix

Use this matrix when comparing vendors for a specific component of the agent stack.

| Criterion | Questions to ask | Red flags |
|---|---|---|
| **Sandboxing** | What isolation mechanism? MicroVM, container, gVisor? | "We use containerization" without specifying namespace isolation |
| **Persistence** | Who owns the persistence layer? Is data encrypted at rest? | Vendor-controlled audit trail with no export mechanism |
| **Identity** | How are agents authenticated? How is tenant binding enforced? | Application-level tenant isolation without cryptographic binding |
| **HITL** | What is the interrupt mechanism? Is it enforcement or advisory? | "The model knows when to ask for help" |
| **Portability** | Can you export your configuration and data? What is the migration path? | Proprietary skill format with no export; vendor lock-in by design |
| **Incident response** | What is the SLA for breach notification? What tools do you provide for investigation? | No breach notification SLA; vendor-controlled forensics only |
| **Compliance** | SOC 2? ISO 27001? GDPR data processing agreement? | "We're working on it" — not acceptable for regulated industries |

---
