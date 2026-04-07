---
title: "Chapter 10: The Third Layer"
description: "Where the characters end up, what the book claims, and the questions the reader carries forward"
date: 2026-04-01
weight: 10
linkTitle: "10. The Third Layer"
tags: ["trust", "conclusions", "maturity model"]
---

## Where the characters end up, what the book claims, and the questions the reader carries forward

---

On a Tuesday morning in early April 2026, Sam Kovic sat in his office with the door closed and read an internal Slack message that said his team's migration to the Claude Agent SDK on AgentCore had, after six months and two slipped deadlines, gone live in production for its first enterprise customer.

He did not feel triumphant. He felt the particular kind of flatness that follows a long engineering project reaching its endpoint — the flatness of a problem that has moved from active to maintained. He closed the laptop, walked to the window, and stood there for three minutes. Then he went back to his desk and wrote a message to his team that said *thank you* and *now we harden it* and *please take Friday off.*

Leo, his CEO, caught him in the hallway an hour later.

"Celebrating?"

"Friday. After people rest."

"Next six months. What do you see?"

Sam thought about this for longer than Leo probably expected. "We harden the trust layer. The orchestration works. The execution works. The trust layer is the one we built fastest because we had to, and it's the one I trust least. The next six months are about making it auditable enough that a customer like Nadia's could adopt it without writing their own framework on top."

"Defensive."

"It's the move I believe in."

Leo nodded and kept walking. He had, over the preceding year, learned to stop pushing Sam on the trust work.

Sam went back to his office and opened the migration document he had written the previous spring. He read his own summary of the three design questions — what does the agent come equipped with, who owns the execution loop, is the harness the runtime — and then read the line he had added at the end: *The framework you can defend is the framework worth choosing.*

He had written that line expecting to defend it to Leo, to customers, to his team. He had not expected to find himself, a year later, still thinking about what defensibility required. The answer, he now understood, was different from what he had imagined. Defensibility was not a property of the framework. It was a property of the trust infrastructure built around the framework. The framework was the easy part. The trust layer was the work.

He wrote a new line below the old one. *Build the framework for the task. Build the trust layer for the audit.*

---

Elena Voss, that same week, was in Berlin conducting a retrospective on her rebuild.

The retrospective was not a celebration. Her company had closed two customers since February, re-closed a third that had paused after her November incident, and had lost two that had decided during the rebuild that they could not wait. The net was negative. Her runway was shorter than it had been a year earlier. Her team was smaller. Her product was materially better.

The retrospective was structured around three questions her head of engineering had written on a whiteboard. *What did we learn? What did we pay for? What would we do differently?*

The team answered the first question at length. They had learned about tenant isolation at the chunk layer, about tool allowlists at the environment level, about audit logs as a product requirement rather than an engineering afterthought.

They answered the second question more quietly. They had paid in dollars — roughly a quarter of their runway. They had paid in customers. They had paid in morale. The head of engineering, Petra, said, without looking up: "We paid in the thing we cannot get back. We paid in time."

Sam had joined by video from San Francisco. When Elena asked what he would have done differently:

"You would not have built on CrewAI. But you know that. The real lesson is earlier. The real lesson is that you treated trust as a line item on a roadmap, and you needed to treat it as a column in your architecture document. Every feature you shipped should have had a trust-layer implication written next to it, and the implications should have been closed before the feature went live."

Elena nodded. "We survived."

"You survived. Most companies in your position did not."

He had watched the graveyard fill since November. Kestrel Insights — a legal-discovery startup that had raised a Series A on the strength of its agent-powered document review — had lost two Fortune 500 clients after a privilege-escalation incident in August 2025, failed to close its Series B in the resulting uncertainty, and quietly wound down that November. The founders were at larger companies now. The IP had sold in a distress auction to a law firm that shelved it. Nobody had written the obituary. That was the thing Sam noticed: the companies that died this way did not get postmortems. They just stopped.

Petra, after the meeting, asked Elena a precise question: "If you could have received this knowledge two years ago, at zero cost, would you have believed it enough to act on it?"

"I would have told myself that the roadmap is where good decisions go to die. The roadmap is the graveyard of the infrastructure you know you need and are pretending you can defer. And I would have told myself that this is not a lesson you can learn from someone else's warning. It is a lesson you pay for."

Petra wrote this down. Elena, watching her, recognized the moment. It was the moment someone younger than her absorbed a lesson Elena had paid for. If the knowledge transferred — if Petra, running her own company in five years, built the trust layer in the first quarter rather than the eighteenth — then Elena's November had not been wasted. It had been tuition paid, delivered to someone else's benefit, as so much tuition is.

Elena wrote her own note in her journal that night: *The lesson is not free. But it is transmissible.*

---

Marcus Chen, in Seattle, was reading a security researcher's disclosure post on a Monday morning in March when his phone buzzed with a message from a friend at Anthropic.

*Have you seen the npm thing?*

A researcher had discovered that an npm package published by Anthropic, as part of a routine Claude Code release, had inadvertently included the complete unobfuscated source code of Claude Code itself. The package had been available for a window of several hours before being pulled. In that window, it had been downloaded thousands of times.

Marcus read the post carefully. Then he sat back in his chair and, for the first time in months, laughed.

It was not a happy laugh. It was the laugh of a person who had been arguing for two years that the industry was under-investing in boring operational discipline, and who had just watched one of the industry's most sophisticated companies make a boring operational mistake. The mistake was not an architecture failure. It was not a novel attack. It was an npm publishing pipeline that had included files it should not have included, caught too late, pulled too slowly.

He wrote a post that afternoon. It was short — three paragraphs — and it made one point:

*The attack surface of AI agent infrastructure is not glamorous. It is npm packages, environment variables, cached responses, debug tools left in production, and the thousand small configurations that no one audits because they have worked this far. The industry has spent the past two years learning that the dangerous failures are not the exotic ones. They are the operational ones. The question is whether the industry will build the operational discipline that the architecture requires, or whether it will keep being surprised by the same category of boring failure until the boring failures stop being tolerated.*

---

David Almeida, in London, was drafting an agenda for a protocol working-group meeting scheduled for late April.

The agenda had three items. The first was tenant identity — the proposal to add required tenant binding to the protocol. The second was workload identity for cross-organizational agent delegation — the A2A problem Sam had flagged to Nadia. The third was skill provenance — the question Marcus had raised, of how agent-authored skills should be represented in the protocol.

David was no longer embarrassed by the protocol's gaps. He had, over the preceding year, come to understand that protocols evolve the way they evolve, through rounds of under-specification, incident, debate, compromise, and revision. The original MCP specification had been criticized for its trust gaps, and the criticisms had been fair. The criticisms had also been the mechanism by which the protocol improved.

He wrote a short reflection in a blog post in early April, titled *Protocols Are Taught By Their Failures*. Its central claim was that the design discipline for infrastructure protocols was not to anticipate every failure but to make the protocol cheap to evolve in response to failures that were, in any case, going to occur. He sent the post to Sam, with a note: *This is the conversation I'll be having for the rest of my career.*

Sam replied: *It's the conversation this whole industry is having. You're just having it more publicly than most.*

---

Nadia Okafor, on a Wednesday afternoon in April, was in a room with the chief risk officers of three other banks, and she was not comfortable.

The meeting had been organized by a financial-services industry association to compare notes on AI agent governance frameworks. The other banks had all adopted variants of the mandatory core Nadia had drafted. One had weakened it in a way she did not respect.

"You've made cryptographic identity binding optional in your mandatory core," she said to that bank's chief risk officer. "I'd like to understand why."

"Because our vendor market cannot supply it yet. If we require it, we cannot deploy any agents, and my leadership has been clear that not deploying is not an option."

"I understand the pressure. I'm asking whether your framework is protecting you if that requirement is optional. Because if it is optional, then it is not enforced, and if it is not enforced, then your deployment depends on application-level isolation, and application-level isolation is what failed at Asana, and at Elena Voss's company."

"I have the same list. I am telling you what my leadership will accept and what it will not."

The room was quiet. Nadia recognized that she was pushing a colleague whose situation was, in most ways, exactly like hers.

"Can I suggest a compromise?" she said. "Make it conditional. Deployments on vendors who provide binding are held to the mandatory core as written. Deployments on vendors who do not are flagged as accepting a specific, documented residual risk, with a scheduled review to escalate the requirement once vendors catch up."

The officer considered this. "That is what I was going to propose in our own framework. I was going to call it a *deferred-binding* posture."

"Then we phrase it together."

The meeting ended an hour later with a rough agreement on shared language. None of them expected that this would change the industry. All four of them understood that it was the kind of slow, unglamorous work through which the industry's trust standards would actually be built.

Nadia walked back to her office and thought about the conversation. She thought about Sam, whose framework had pointed her at the right questions. She thought about Marcus, whose comments on her draft had given it backbone. She thought about David, whose protocol revisions would eventually close some of the gaps her framework currently covered. She thought about Elena, whose incident had given her the concrete argument she had used to defend the framework to her own board.

None of them worked at her bank. All of them were, in some functional sense, her colleagues now. The network she had not set out to join had formed around a problem they were all trying to solve, and the solution was emerging, piece by piece, across their separate conversations.

She wrote in her notebook that evening: *The trust layer is not built by any single company. It is built by the conversations between companies.*

---

The book's central claim has been that what everyone calls "the agent framework" is actually three layers pretending to be one. The **orchestration layer** shapes how agents reason. The **execution layer** constrains what agents can physically touch. The **trust layer** governs whether agents' actions can be identified, authorized, audited, and defended.

The book has argued that these three layers are, in early 2026, at radically different stages of maturity. The orchestration layer is mature. The execution layer is adolescent. The trust layer is nascent. And this asymmetry has consequences: Elena's November incident, the Asana cross-tenant leak, the more than forty thousand exposed OpenClaw instances, Jack Luo's dating profile, the Cisco skill-telemetry findings.

### Three Diagnostic Questions

The reader of this book does not need predictions about which vendors will win. The reader needs diagnostic tools — questions that will remain useful regardless of which specific products survive.

**One: Where is the trust boundary?** What can the agent touch, and what enforces that boundary? Is it the model's judgment (insufficient), a prompt instruction (also insufficient), a sandbox that isolates physical blast radius, a policy engine that intercepts tool invocations outside the agent's reasoning loop, or a human in the loop who must approve certain classes of action? The best-architected systems will have boundaries at multiple layers, with the strongest boundaries enforced at infrastructure below the application.

**Two: Who is accountable, and can you prove it?** When the agent acts, can you trace its reasoning — what it read, what it decided, what it called, what it returned — in a durable audit trail that the organization controls, not the vendor? Can you attribute actions to a specific user, a specific agent instance, and a specific tenant? Can you defend the deployment to a regulator who has never heard of your agent framework?

**Three: What compounds?** Is the system becoming more capable over time — through learning loops, self-authored skills, accumulated memory — and if so, can you audit the compounding? Can you identify what the system learned, when it learned it, and why? Can you roll back learned behaviors you do not endorse?

Any agent system that cannot answer all three questions is not ready for production deployment in any context where actions have real consequences.

### A Maturity Model

**Level 0 — Demo.** The agent works in a controlled setting. No trust infrastructure. Appropriate for internal experimentation. Inappropriate for any deployment where failure has consequences.

**Level 1 — Supervised Production.** The agent operates in production, with every significant action reviewed by a human before execution. Appropriate for high-stakes, low-volume tasks. The limitation is scale.

**Level 2 — Sandboxed Autonomy.** The agent operates with execution isolation (microVM or equivalent) and declared HITL checkpoints for defined high-stakes actions. Appropriate for enterprise deployments handling regulated data.

**Level 3 — Trusted Autonomy.** The agent operates with cryptographically bound identity, policy enforcement outside the reasoning loop, tenant-controlled audit infrastructure, and declared fallback postures for unanticipated actions. Appropriate for mission-critical deployments in regulated industries. No major vendor currently ships this out of the box.

**Level 4 — Self-Improving Autonomy.** The agent learns from its own behavior, authors its own skills, and compounds capability over time, with governance infrastructure that makes the compounding auditable and reversible. This is the frontier. It will be built. It is not ready in early 2026.

Most production deployments in 2026 sit at Level 1 or Level 2. The enterprise frameworks emerging from companies like Nadia's are an attempt to define Level 3 in terms vendors can build toward.

### What the Third Layer Is Not

The trust layer is not the moat — not in the traditional business sense of a defensible competitive advantage. The trust primitives will be standardized. The enterprise frameworks will be shared. The protocols will evolve toward common ground. The vendors who win will not win because their trust infrastructure is uniquely theirs. They will win because they built it early, shipped it competently, and made it easy for their customers to adopt.

The trust layer is, instead, the table stakes. It is the infrastructure without which no vendor will survive in the enterprise market once the market matures. It is the work that every serious team will have to do, and most teams are deferring. The advantage available right now — in early 2026 — is not the advantage of uniqueness. It is the advantage of earliness.

### What the Reader Carries Forward

The reader who has reached this page has spent a long time inside one argument: that the agent framework is three layers, that the industry has invested in the first while neglecting the third, and that the next several years of this field's work will be in closing the trust gap.

If the argument is correct, the reader now carries a vocabulary — orchestration, execution, trust — with which to evaluate any agent system they encounter. They carry three diagnostic questions. They carry a maturity model. They carry the stories of five people who represent, in different ways, the tradeoffs the industry is navigating: a practitioner who migrated to the right framework for the wrong reasons and kept learning; a founder who paid for her deferred investments and survived; a security researcher who named the failures that no one wanted to name; a protocol designer who built the connective tissue and watched it be improved by its own failures; an enterprise buyer who refused to deploy what she could not defend.

None of them finished the work. None of them will. The work is generational.

---

### What 2028 Looks Like

The book should commit to predictions. Here are the ones this author will defend.

**The orchestration layer consolidates to three survivors.** By 2028, the open-source orchestration framework market will have collapsed to a small number of viable options — almost certainly the Claude Agent SDK, the OpenAI Agents SDK, and LangGraph as the remaining flexible-graph option. The rest — the frameworks that launched in 2023 and 2024 with compelling demos and thin maintenance communities — will be unmaintained or absorbed. The consolidation is already underway. Teams building new systems in 2026 are not picking from fifteen frameworks; they are picking from three or four. By 2028, the default choice for most practitioners will be whichever SDK ships with the model their company has standardized on. The orchestration layer will become invisible infrastructure.

**Microsoft wins enterprise, for now.** Among the major platform vendors, Microsoft has the strongest near-term position in enterprise agent deployment. Not because their technical architecture is superior — it is not — but because enterprise adoption is not primarily a technical decision. It is a procurement, compliance, and integration decision. Microsoft's existing relationships with enterprise IT, its presence in identity infrastructure through Azure AD, and its deep integration into the tools enterprises already use (Teams, Office, SharePoint, Azure) give it a distribution advantage that no model-native competitor has yet matched. The caveat is that this advantage is conditional on trust architecture maturity. If Microsoft's trust layer is incomplete when the first significant enterprise incidents occur — and they will occur — the advantage will narrow. Anthropic, OpenAI, and Google have time to close the gap if they invest in enterprise trust primitives rather than demo features.

**LangChain is absorbed or irrelevant by 2027.** LangGraph remains useful because it solves a real problem — explicit state management in complex graphs — that the SDK-native frameworks have not fully addressed. But LangChain, the framework that preceded it, has been in decline since mid-2024, when the SDK-native models eliminated its primary value proposition (easy model switching). The company behind LangChain has pivoted repeatedly — observability, deployment, evaluation tooling. One of these pivots will either produce a sustainable business or result in an acquisition. The open-source LangChain framework will be in maintenance mode within eighteen months of this book's publication.

**The trust layer becomes a regulatory requirement in two industries before 2028.** Financial services and healthcare will be the first verticals where agent trust architecture is mandated, not chosen. The first regulatory guidance will be derivative — an extension of existing data governance and operational risk requirements — but it will reference specific technical controls: cryptographic identity binding, policy enforcement outside the reasoning loop, audit trail requirements. Nadia's mandatory core, or something structurally equivalent to it, will appear in a DORA or OCC guidance document. When that happens, the vendors who have invested in trust infrastructure will have a compliance advantage that their competitors cannot replicate in six months. The window for building this infrastructure proactively — rather than reactively, after the guidance drops — is open now and will not stay open much longer.

**One major vendor gets acquired by a hyperscaler before 2028.** The model-native agent frameworks are expensive to maintain, expensive to distribute, and easier to build if you control the underlying compute. At least one of the independent agent infrastructure companies will be acquired by AWS, Google, or Microsoft in the next two years. The acquisition target is most likely a company whose technical advantage is in execution infrastructure (sandboxing, tool management) or trust infrastructure, rather than in orchestration, which is the layer most vulnerable to commoditization. The price will be high. The rationale will be the trust layer.

The reader who disagrees with these predictions has the same information this book has, and may well be right. These are not certainties. They are the conclusions that follow from taking the arguments of this book seriously, combined with the competitive dynamics visible in early 2026.

If the predictions are wrong in the details, they are right in the direction: the orchestration layer is consolidating, the enterprise market is being won by distribution and trust rather than by technical superiority, the independent frameworks are running out of runway to compete with SDK-native defaults, and the trust layer is about to become a regulatory requirement rather than a design choice.

The window for building the right infrastructure — before it becomes mandatory, before the incidents force it, before the competition has caught up — is the window this book was written inside. How much of it remains is a question the reader will have to answer from wherever they are standing.

