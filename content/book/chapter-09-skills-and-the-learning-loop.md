---
title: "Chapter 9: Skills and the Learning Loop"
description: "The abstraction every team invented independently — and the agent that took it further"
date: 2026-04-01
weight: 9
tags: ["skills", "learning", "Hermes", "SKILL.md"]
---

## The abstraction every team invented independently — and the agent that took it further

---

## Act 1: The Story

Elena Voss discovered skills by accident, in the third week of her rebuild, while she was trying to solve a problem she did not realize had a name.

It was late February 2026. Her company had committed to the Claude Agent SDK as part of the rebuild — Sam had pushed her toward it, Nadia's framework had pushed her toward it, and by the time Elena actually looked at the documentation, the decision had been made. She was porting the agent's instructions from her old CrewAI configuration when she hit the problem that had been hurting her for a year without her having articulated it.

The problem was that her agents needed to know things. Not facts — the model knew facts — but procedures. How her company's refund workflow actually worked, with all its exceptions and special cases. Which tone to use when a customer mentioned a competitor. When to escalate and when to hold. The conventions for referring to products by their internal names versus their customer-facing names. Her old CrewAI system had stuffed all of this into a single enormous system prompt that she had been editing nervously for months, afraid that adjusting one part would break another.

The Claude Agent SDK documentation pointed her at something called Agent Skills. She read the page twice and then wrote to Sam.

*Have you used these?*

*Yes. Extensively. Why?*

*I think they're what I've been trying to build in my system prompt.*

*They are. Start with the refund workflow. Make a skills directory, write a SKILL.md that describes when to use the skill and links to a procedure file, and see what happens.*

Elena spent the afternoon doing this. She created a directory called `refund-workflow`. Inside it, she wrote a short markdown file that described what the skill was for and when the agent should reach for it. She linked to a longer procedure document that laid out every exception and special case, and to a CSV that mapped internal product codes to customer-facing names. She configured the agent to load skills from the directory.

Then she tested it. The agent opened the skill only when the customer's message was actually about a refund. When it opened the skill, it loaded the procedure, used the CSV to translate product names, and produced a response that was better than anything her previous system had produced — not because the model had gotten smarter but because the model was now getting the right context at the right time, instead of all the context all the time.

Elena moved three more procedures into skills the next day. By the end of the week, she had twelve skills, and her system prompt had shrunk from 4,200 tokens to roughly 600. The agent's responses were more accurate. Her token bill dropped. And she could edit a single procedure without worrying about what it would break, because the procedures were now isolated from each other in a way they had never been when they all lived in one enormous prompt.

On the Friday of that week, she wrote to Sam again: *This is the abstraction I needed eighteen months ago.*

Sam wrote back: *It's the abstraction everyone needed eighteen months ago. Most of us didn't have it either.*

---

David Almeida, in London, had been noticing something strange over the preceding six months. He had been tracking how different agent projects exposed capabilities to their underlying models, and had observed five separate teams — none of whom, as far as he could tell, were coordinating — arrive at substantially the same design.

Anthropic had shipped Agent Skills: directories containing a `SKILL.md` file with a name, a description of when to use the skill, and references to supporting documents. OpenAI, in the Codex desktop app released in February, had introduced Skills: directories containing instructions, scripts, and resources, bundled to extend the agent's capabilities. OpenClaw, independently, had used the same structure for ClawHub — SKILL.md folders that users could install to add capabilities. Nous Research's Hermes Agent used an open standard called agentskills.io, which was — David noted with some amusement — a SKILL.md folder with a slightly different set of optional files. LangChain's DeepAgents had converged on the same pattern.

Five teams. Five products. Five slightly different names for the same abstraction. A markdown file inside a folder, optionally accompanied by scripts and reference material, designed to be loaded by an agent at runtime to acquire a specific capability.

David turned his observations into a public essay in early March, titled *The Package Manager We Keep Reinventing*. It ended with a question: why had this particular shape arrived, independently, in so many places, so quickly?

He offered four answers.

**Markdown is the lingua franca of language models.** A model's training data is rich in markdown. Asking a model to read a markdown document is asking it to do the thing it is most practiced at doing. A skill written in markdown is read by the model the way a person reads a recipe: fluently, in context, without a parsing step.

**Folders are the natural unit of context.** A capability rarely consists of a single instruction. It consists of instructions, supporting procedures, reference materials, and sometimes scripts. A folder groups these naturally, and makes the skill portable: a folder can be copied, versioned, shared, installed.

**Dynamic loading solves the context-window problem.** If every capability the agent might need were present in every prompt, the prompt would be enormous and expensive. If capabilities are loaded only when needed, the prompt stays small. Anthropic's measurements showed that skills could reduce the token cost of capability declaration by roughly 98% compared to equivalent tool specifications held in the context window.

**The format is composable.** Skills can reference other skills. A skill can list another skill as a prerequisite. The composition is recursive in the way the best abstractions are.

---

Sam read David's essay the morning it was published and immediately forwarded it to his engineering team. He wrote, in the forwarding note: *This is the abstraction the whole agent ecosystem is converging on. Our migration has been using it without us having a word for it. Now we have a word.*

By this point, Sam's own team was nine weeks into their Claude Agent SDK migration. They had accumulated eighteen skills. Five were small — capability descriptions for specific internal tools. Thirteen were substantial — procedural documents that captured the company's institutional knowledge about how to do specific customer workflows, rewritten as instructions an agent could follow.

Writing skills had forced his team to articulate procedures that had previously lived only in their senior engineers' heads. The process of turning tacit knowledge into an explicit skill file was, itself, valuable. His most senior customer engineer had spent two days writing a skill document. At the end of those two days, the skill document existed, and more importantly, the knowledge existed in a form that could be read by a new hire, reviewed in a pull request, updated through version control, and tested.

"Skills aren't just an agent abstraction," Sam told the team at their Monday planning meeting. "They're an organizational documentation abstraction. They're making us write down things we have known how to do but have never written down."

One of his engineers, a recent hire, said: "They're also making us realize how many of our procedures were wrong. I wrote one last week, and while I was writing it, I found three steps that contradicted each other."

Sam wrote this down. The skills abstraction was doing something beyond the agent — it was surfacing institutional contradictions that had been hidden by the fact that no one had previously been forced to make them explicit.

---

The escalation arrived on February 26, 2026, when Nous Research released Hermes Agent.

Hermes Agent was, on its surface, another personal-agent framework. It ran locally. It could connect to messaging channels. It supported more than forty built-in tools. It had an explicit migration path from OpenClaw, via a command Sam found either audacious or insolent depending on his mood: `hermes claw migrate`. None of this was remarkable.

What was remarkable was that Hermes wrote its own skills.

The architecture was straightforward in description and unusual in practice. After completing a task, the agent wrote a structured record of what it had attempted, what had worked, and what had failed. It stored this record in persistent memory — SQLite with full-text search, supplemented by summarization performed by the underlying model. When the agent encountered a similar task in a future session, it retrieved the relevant records and used them to adjust its approach. For patterns of recurrence, the agent went further: it wrote a new skill file describing the procedure it had learned, saved it to the skills directory, and loaded it on subsequent similar tasks.

Jeffrey Quesnelle, Nous Research's CEO, demonstrated the system by having Hermes Agent write a seventy-nine-thousand-word novel across iterative sessions, with each session building on skills generated in previous sessions. The novel was not remarkable as fiction. It was remarkable as evidence: the agent had, in the course of producing it, developed a library of self-authored skills about how to maintain narrative continuity, handle character voice, and recover from plot inconsistencies.

Sam read the Hermes Agent documentation on a Sunday afternoon, then read the source code on Sunday evening, then sent a long message to David.

*This is the thing we've been circling. Skills were a static abstraction: humans write them, agents read them. Hermes makes them dynamic: agents write them, agents read them. The loop closes. I've been trying to decide if this is a breakthrough or a warning.*

David replied the next morning: *Both. The architecture is elegant. The governance question is unanswered. How do you audit an agent whose behavior is determined, in part, by skills it wrote itself? Your review process assumes skills are human-authored artifacts. Hermes breaks that assumption.*

Sam wrote back: *What do you do if you're me, deploying agents into enterprise contexts where auditability is the whole ballgame?*

David's reply was short: *You do not let your agents write their own skills in production. Not yet. Maybe not for a long time.*

---

Sam and Elena talked about Hermes on a video call the following weekend.

"If Hermes works," Elena said, "it solves a problem I've been paying a tax on for a year. My agents handle thousands of similar support tickets every day. They don't learn from them. Every ticket is treated fresh. The same mistakes get made over and over. A learning loop that wrote skills from successful resolutions would compound."

"It would."

"So why am I not building it?"

"Because you cannot audit an agent whose skills it wrote itself. Your customers' security teams are going to ask how your agent makes decisions, and 'it taught itself' is not an answer they will accept. Not this year. Maybe not for several years."

"But the competitive pressure —"

"Will be real. If a competitor ships a self-learning agent that handles tickets better than yours, and they can get past security review somehow, you'll lose deals. But the first few security incidents involving self-learning agents are going to reshape the market. The question is whether you want to be the vendor who has to defend the first incident, or the vendor who waited six months and learned from someone else's."

Elena noted this down. Then she changed the subject. They talked about the fact that her company had just closed its first renewal with an enterprise customer since the November incident. The customer had asked seventy-three security questions. Elena's team had answered all seventy-three. The contract had closed on a Friday, for a smaller amount than Elena had hoped, with aggressive service-level commitments that would stretch her team.

"The recovery," she told Sam, "is going to take longer than the incident did. By a factor of about ten."

"That sounds right."

"Worth it?"

"It's the only path. The alternative was to not have a company."

---

## Act 2: The Architecture

### The SKILL.md Anatomy

A skill, in essentially every current implementation, is a directory containing:

**A SKILL.md file.** This file has three responsibilities. It declares the skill's name. It describes, in a few sentences, when the agent should reach for this skill — the *activation criterion*. And it provides the instructions the agent follows when the skill is active. The description is what the agent reads to decide whether to open the skill; the instructions are what the agent reads after deciding.

**Supporting documents.** Markdown files, CSVs, JSON references, and occasionally scripts, linked from the SKILL.md and loaded when the skill is active. They contain the procedural detail that would be prohibitively expensive to carry in the main context.

**Optional scripts.** Some skills include executable code — typically Python or shell — that the agent can call as part of the skill's procedure. Scripts let a skill perform deterministic work (validating a format, parsing a file, calling a specific API) without asking the model to do it in natural language.

**Optional metadata.** Some implementations add a manifest file declaring the skill's version, author, dependencies, and declared side effects.

### Progressive Disclosure

The architectural innovation is progressive disclosure: the system prompt contains only the skill's name and description, not its full contents. The agent reads the description, decides whether to open the skill, and only then loads the full body into context.

The token-cost implications are significant. An agent with two hundred skills, where each skill's full body is ten thousand tokens, would need two million tokens of context to carry all skills at once. With progressive disclosure, the same agent carries roughly two hundred short descriptions — perhaps twenty thousand tokens — and loads full skill bodies only for skills it actually uses.

### Skills Versus MCP

Skills and MCP are sometimes framed as competitors. They are not. They operate at different layers.

MCP is a protocol for live tool invocation: the agent calls a tool, the tool executes, the result returns. The tool is active — it does something in the world at the moment of invocation. A skill is a document that tells the agent how to do something, possibly including instructions to call specific tools in a specific order. The skill is passive — it is read by the agent, not executed by a server.

The practical rule: **skills tell the agent what to do; MCP servers are what the agent calls to do it.**

### From Static to Dynamic to Autonomous

**Static skills** are human-authored. A developer writes a SKILL.md, tests it, versions it, ships it. This is Anthropic Agent Skills, OpenAI Codex Skills, Elena's refund-workflow skill. Most of the production agent ecosystem in early 2026 is at this stage.

**Dynamic skills** are still human-authored, but the authoring is responsive. A team observes an agent failing on a class of task, writes a skill to address the gap, and ships it into the running system. This is the pattern Sam's team adopted: skills added to fill gaps the team discovered through production observation.

**Autonomous skills** are agent-authored. The agent, after completing a task, writes a skill describing what it learned. This is Hermes Agent. The abstraction is the same — a SKILL.md folder — but the authoring loop no longer passes through a human reviewer.

Each step grants the agent more adaptive capability. Each step removes a point where human judgment could catch a problem before the skill enters production use.

### The Audit Problem

The question David named — how do you audit an agent whose behavior is determined, in part, by skills it wrote itself — does not yet have a good answer.

The available partial answers include **skill provenance tracking** (every skill file records who wrote it — user, agent, or other), **skill review workflows** (agent-authored skills require human review before being made available on production tasks), **sandboxed self-authoring** (agent-authored skills run in restricted contexts), and **skill rollback** (the skills directory is version-controlled, and problematic skills can be reverted).

None of these are solutions. Provenance tracking does not prevent misuse; review workflows reintroduce human bottlenecks; sandboxing limits the skills the agent can usefully learn; rollback is reactive. The deeper problem is that the audit model the enterprise ecosystem has built over the past eighteen months assumes skills are artifacts produced by humans through a reviewable process. Self-authoring breaks this assumption.

Sam added a line to his migration document, under the section on skills:

*All skills in production must be human-authored, human-reviewed, and version-controlled. Agent-generated skills are a research direction, not a deployment direction.*

### What the Convergence Reveals

The fact that five independent teams arrived at the same abstraction, in the same six-month window, is the strongest signal that the skills pattern is structural rather than stylistic. It was not copied. It was discovered, repeatedly, by teams solving the same problem under similar constraints.

The constraints were: context windows are finite, capabilities are not, and the cost of carrying a capability in the context is real money. The natural solution was dynamic loading. The natural format was markdown, because models read markdown fluently. The natural unit was the folder, because capabilities rarely fit in one file.

Convergent evolution in software is rare and informative. When it happens, it suggests the designers have found a point in the design space that the underlying constraints force. Skills appear to be such a point.

What they do not settle is the question of authority. A human-authored skill carries the authority of the person who wrote it. An agent-authored skill carries — what? The agent's judgment? The model underneath the agent? The implicit authority of the environment that allowed the authoring? The category does not yet have an answer.

---

*Next: [Chapter 10 — The Third Layer](/book/chapter-10-the-third-layer/)*
