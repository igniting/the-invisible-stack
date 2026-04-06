---
title: "Chapter 2: The Chain, the Graph, and the Loop"
description: "Three metaphors for how agents think — and why each one broke"
date: 2026-04-02
weight: 2
linkTitle: "2. The Chain, the Graph, and the Loop"
tags: ["orchestration", "LangGraph", "frameworks"]
---

## Three metaphors for how agents think — and why each one broke

---

## Act 1: The Story

Six months into building his company's agent platform, Sam Kovic hit the wall.

It was a Thursday afternoon in August 2023. He was in a conference room with two of his engineers, staring at a whiteboard covered in arrows, and they were trying to debug an agent that processed customer support tickets. The agent was supposed to read the ticket, search the knowledge base for relevant documentation, draft a response, check the draft against company policy, and send it. Five steps. A chain.

The problem was that tickets didn't fit neatly into five steps. Some tickets required the agent to ask the customer a clarifying question before searching the knowledge base. Some required checking the customer's account status before drafting a response, which meant calling an API that sometimes timed out, which meant the agent needed retry logic, which meant it needed to remember where it was when the retry succeeded. Some tickets mentioned multiple issues — a billing question and a bug report in the same message — and the agent needed to split them, route each part to the right handler, and merge the responses back into a single reply.

Each of these variations was, in LangChain's chain abstraction, a separate chain. A chain for tickets with clarifying questions. A chain for tickets that needed account lookups. A chain for tickets with multiple issues. Sam's team had written fourteen different chains for what was, from the customer's perspective, a single interaction: "I sent a support ticket and got a reply."

The chains shared code, sort of. They diverged in places that were hard to predict. When a bug appeared — and bugs appeared constantly, because language models are non-deterministic and fourteen code paths are fourteen opportunities for things to go wrong — debugging meant first figuring out which of the fourteen chains the ticket had been routed to, then finding the specific step where the model had made a wrong turn, then understanding what prompt had been constructed at that step, which required reading LangChain's source code because the prompts were embedded inside the framework's abstractions, not in Sam's code.

"I'm spending more time reading LangChain's internals than writing our own product," said one of his engineers, a statement that would have been funny if it weren't also true.

Sam walked to the whiteboard. He erased the arrows and drew a picture that was not a chain. It was a graph — nodes for each operation (read ticket, search knowledge base, check account, draft response, check policy, send), with edges between them that could go forward, backward, or sideways depending on the situation. Some edges were conditional: if the ticket mentions billing, go to the billing handler; if it mentions a bug, go to the technical handler. One edge looped back: if the policy check fails, go back to draft response and try again. One edge went off the board entirely: if the agent has been looping for more than three attempts, escalate to a human.

He stepped back and looked at what he'd drawn. It was not a chain. It was a state machine — a system with defined states and explicit transitions between them. The ticket moved through the states based on its content and the results of each step, not based on a fixed sequence that some developer had prescribed in advance.

"This is what the agent should look like," Sam said. "But LangChain can't express this. At least not without building a state machine on top of it by hand."

The irony was that Harrison Chase — LangChain's creator — had already arrived at the same conclusion.

---

Chase had been hearing the same complaints from thousands of developers, and the complaints converged on the same theme Sam had identified: the chain abstraction assumed that agent work was linear, and real agent work was almost never linear.

The criticism came from multiple directions. Some developers complained about opacity — the framework's high-level abstractions hid the prompts being sent to the model, making debugging a forensic exercise. Some complained about control — they wanted to add custom logic between steps, inject human approval before certain actions, implement their own error handling, and the chain abstraction made none of this easy. Some complained about bloat — LangChain had grown into a massive package with hundreds of integrations, each adding dependencies and surface area for bugs.

But the deeper issue, the one that no amount of engineering polish could fix, was structural. A chain could only go forward. Step one to step two to step three. If step three needed to loop back to step one, or branch to step four-A or step four-B depending on a condition, or pause for human input and resume later, the developer had to build that logic on top of the chain — effectively building a state machine using a tool designed for pipelines. It was possible but perverse, like driving screws with a hammer.

In the summer of 2023, Chase made a decision that Sam would later describe to Elena as "the bravest thing I've seen a founder do." He began building a replacement for his own creation. Not a patch, not a version 2.0 — a genuinely different framework with a genuinely different abstraction. He called it LangGraph.

LangGraph launched in January 2024. Sam adopted it within weeks.

---

The difference was immediate. Where LangChain had chains, LangGraph had a **StateGraph** — a directed graph where nodes were functions and edges defined control flow. But the real innovation was not the graph. It was the explicit treatment of **state**.

In a chain, state was implicit. The output of one step became the input to the next, and that was the extent of the system's memory. If step three needed information from step one that wasn't in step two's output, you had to thread it through by hand — passing data along like a bucket brigade, each link responsible for carrying information it didn't need for the sole purpose of handing it to a later link.

In LangGraph, state was a first-class object. You defined a typed data structure — Sam used a Python TypedDict containing the ticket text, the customer ID, the conversation history, the retrieved documents, and a retry counter — and every node received the current state, did its work, and returned a partial update. The framework merged the update into the existing state. The state was the single source of truth for everything the agent knew and everything it had done.

This made three things possible that changed how Sam thought about agent architecture.

The first was **checkpointing**. Because the state was an explicit, serializable object, the framework could save it to a database after each node. If the process crashed — due to an error, a timeout, a deployment, or an infrastructure outage — a new process could load the most recent checkpoint and resume from that point. The agent didn't re-execute the nodes that had already completed. The tool calls that had already been made were not repeated. The emails that had already been sent were not sent again.

Sam had been losing sleep over this problem. His chain-based agents ran in a single process, and if that process died, the entire interaction had to be re-run from the beginning. For a quick two-step lookup, restarting was fine. For a ten-minute agent session that had already modified customer records, sent notifications, and filed internal reports, restarting was somewhere between wasteful and catastrophic. Checkpointing turned agent execution from a fragile, all-or-nothing gamble into a durable, resumable process.

The second was **human-in-the-loop**. Sam could now define certain nodes in his graph as **interrupt points**. When the agent reached an interrupt node — say, before sending a response to a customer, or before modifying a billing record — the framework paused execution, serialized the state, and emitted it to whatever system was responsible for collecting human input. A Slack notification, an email, a dashboard widget. The human reviewed the agent's proposed action, approved or modified it, and the framework resumed the graph from the interrupt point with the human's input merged into the state.

This was not a feature Sam bolted onto the side of his system. It was a natural consequence of the graph-and-state architecture. The framework didn't distinguish between "the model decided what to do next" and "the human decided what to do next." Both were state transitions. Both were checkpointed. Both were logged. The uniformity was elegant, and it solved a problem that had been nagging Sam since his conversation with Marcus Chen at the security conference.

"How do you decide where to put the interrupts?" Marcus asked when Sam showed him the architecture.

"Where the cost of being wrong is highest," Sam said. "Before sending external communications. Before modifying financial records. Before any action that can't be undone."

"And the rest of the time, the agent just runs?"

"The rest of the time, the agent just runs."

Marcus considered this. "That's the right design. Most people either approve everything — which makes the agent useless — or approve nothing, which makes it dangerous. You're approving the things that matter."

The third was **streaming**. Because the graph's execution was a series of discrete, observable node transitions, each transition could be emitted to the client in real time. Sam built a dashboard that showed the agent's progress as it moved through the graph: "Reading ticket… Searching knowledge base… Found 3 relevant documents… Drafting response… Checking against policy… Policy check passed… Awaiting human approval." The customer support team could watch the agent work, understand what it was doing and why, and catch problems before they reached the customer.

This transparency turned out to be as important for internal trust as it was for debugging. Sam's CEO, Leo, had been skeptical of the agent's judgment. The streaming dashboard changed his mind. Not because the agent stopped hallucinating, but because when it did hallucinate, the dashboard made the failure visible and understandable. Leo could see which step had gone wrong and why. The agent was no longer a black box. It was a process with visible, auditable steps.

"I trust this more," Leo told Sam after watching the dashboard for a week. "Not because it's perfect. Because I can see when it's not."

---

While Sam was rebuilding his system on LangGraph, Elena Voss was having a very different experience.

Elena's startup had chosen CrewAI — a multi-agent framework that had launched in late 2023 with a compelling pitch: define your agents as characters with roles, goals, and backstories, and let them collaborate on tasks like a human team. A "researcher" agent that gathered information. A "writer" agent that drafted responses. A "reviewer" agent that checked quality. The metaphor was intuitive and the getting-started experience was superb. Elena had a working prototype in four days.

CrewAI was not a chain or a graph. It was closer to the **loop** metaphor — each agent ran in a cycle of reasoning and action — but with a multi-agent layer on top: the agents communicated with each other, handed off tasks, and coordinated their work through a shared context. Elena loved it. Her team loved it. The investors loved the demo. The first customers loved the results.

The problems started six months in, when Elena tried to take the prototype to production.

The first problem was customization. CrewAI's high-level abstractions — roles, goals, backstories — made it easy to define agents but hard to control their behavior at a fine-grained level. Elena wanted to add a step between the researcher and the writer: a check against the customer's account status that determined which response template to use. In LangGraph, this would have been a node in the graph with conditional edges. In CrewAI, it required reaching inside the framework's agent-communication layer and injecting custom logic in a place the framework hadn't designed for customization. The workaround was brittle and broke when CrewAI released a new version.

The second problem was observability. When a customer complained about a wrong answer, Elena's team needed to understand what had happened: which agent had produced the wrong information, what documents it had consulted, what reasoning had led to the error. CrewAI's multi-agent communication was largely opaque. Elena's engineers wrote custom logging to capture the inter-agent messages, but the logging was incomplete and inconsistent.

The third problem was durability. CrewAI had no checkpointing. If a multi-agent session failed midway through — and with multiple agents making multiple model calls, failures were common — the entire session had to restart from the beginning. Elena's agents were processing customer support tickets for e-commerce companies, and each ticket took roughly 30-90 seconds to process. A failure rate of 15% meant that 15% of tickets were processed twice, at twice the cost, with twice the latency. The economics were painful.

Sam and Elena talked about this on a call in the spring of 2024.

"You should look at LangGraph," Sam said.

"I know," Elena said. "But migrating would take three months minimum. We'd have to rewrite the agent logic, the inter-agent communication, the prompt templates, the evaluation suite. And we'd ship nothing new during those three months. The board would not be thrilled."

"What's the alternative?"

"Keep patching CrewAI. Add our own checkpointing on top. Build better logging."

"That's building a state machine on top of a framework that wasn't designed for state machines."

"Yes, Sam, I know what it is. I just don't have a better option right now."

Sam recognized the trap. Elena was stuck on a framework whose low floor (easy to start) came with a low ceiling (hard to scale), and the cost of migrating to a higher-ceiling framework was prohibitive because she'd already built six months of product on the lower one. This was the framework selection problem in its purest form: the framework that was easiest to start with was rarely the one that was easiest to operate.

"For what it's worth," Sam said, "I think you'll end up migrating anyway. The question is whether you do it now, when it's expensive, or later, when it's more expensive."

Elena didn't argue. She knew he was right. She also knew that "later" was the only option her board would accept.

---

While Sam was migrating to LangGraph and Elena was patching CrewAI, a team at OpenAI was approaching the entire problem from the opposite direction.

Where LangGraph gave developers maximal control — every node, every edge, every state transition defined in code — the OpenAI team was drawn to maximal simplicity. They had observed that the most impressive agent behaviors often emerged not from elaborate orchestration but from a tight loop: give the model a task and some tools, let it decide what to do, observe the result, repeat until done. No graph. No state machine. Just a **loop**.

In October 2024, they published this idea as **Swarm** — a framework so minimal that they labeled it "educational" and explicitly said it was not production-ready. Swarm had two abstractions: an **Agent** (a model with instructions and tools) and a **Handoff** (the ability for one agent to transfer control to another). No checkpointing. No durable execution. No streaming infrastructure. No human-in-the-loop.

The loop metaphor assumed something radical about the model: that it was smart enough to manage its own execution. Rather than the developer prescribing a graph of possible paths, the model would reason about the task, decide what tool to use, evaluate the result, and determine the next step. The developer's job was not to design the agent's logic but to provide the right tools and get out of the way.

Sam read the Swarm source code the weekend it was released. It was elegant and tiny — the core loop fit on a single screen. He showed it to Marcus on a call.

"It's beautiful," Marcus said. "It's also terrifying."

"Why terrifying?"

"Because there are no guardrails. The model decides everything. If the model decides wrong — calls the wrong tool, accesses the wrong data, enters an infinite loop — there's nothing to stop it. No checkpoints to roll back to. No approval gates. No audit trail. Just a model with tools, running unsupervised."

"It's educational," Sam said. "They said it's not for production."

"People will use it for production anyway. They always do."

Marcus was right. Within weeks, developers were deploying Swarm-based agents in production environments, drawn by its simplicity and undeterred by the warnings. Some of them got burned. Others got lucky. The line between the two was thinner than anyone was comfortable with.

Five months later, in March 2025, OpenAI released the **Agents SDK** — the production-ready evolution of Swarm. It kept the minimalist philosophy (four primitives: Agents, Handoffs, Guardrails, Tracing) while adding the bare minimum of production infrastructure. Guardrails could validate inputs and outputs. Tracing captured the execution trajectory for debugging. But the SDK still had no checkpointing, no durable state, no human-in-the-loop. Persistence was the developer's problem.

The Agents SDK embodied a belief that Sam found both appealing and disquieting: as models got smarter, the harness could get thinner. A sufficiently capable model didn't need a complex graph to guide its reasoning. It needed tools, constraints, and the freedom to work.

Sam discussed this with David Almeida over video call.

"The OpenAI position is that the model is smart enough to be trusted with the loop," Sam said. "The LangGraph position is that no model is smart enough to be trusted without structure. Which is right?"

David gave an answer that stuck with Sam for months: "They're both right, for different tasks. The question isn't 'how smart is the model?' The question is 'what happens if the model is wrong?' If the cost of being wrong is a bad draft that gets revised, use a loop. If the cost of being wrong is a financial transaction that can't be reversed, use a graph with approval gates. The architecture should match the stakes, not the model's capability."

Sam wrote this down. It would become one of the core principles of his team's agent design: **match the autonomy to the stakes**.

The three metaphors — chain, graph, loop — were not competing answers to the same question. They were different answers to different questions, and the art of agent architecture was knowing which question you were asking.

---

There is one more development from this period that deserves attention. In November 2025, LangChain released version 1.0 alongside LangGraph 1.0. The release included a new `create_agent` function — a high-level abstraction for building agents that was built, under the hood, on LangGraph. The old chains, agents, and memory modules were moved to a separate package called `langchain-classic`. The message was unmistakable: the chain era was over.

More striking was a project that appeared shortly afterward: **DeepAgents**, an agent harness built with LangChain and LangGraph that the team described as having been "primarily inspired by Claude Code." It followed what the creators called a "trust the LLM" philosophy: give the agent tools, enforce boundaries at the tool and sandbox level, and let the model drive. This was the loop philosophy — but implemented on top of graph infrastructure that provided the checkpointing and observability the pure loop lacked.

Sam noticed the convergence. The team that built the chain had replaced it with the graph, then shipped a high-level agent that looked like a loop with guardrails. The team that started with a pure loop (OpenAI's Swarm) was gradually adding structure. And the team that started with a fully equipped loop (Anthropic's Claude Code) was exposing its architecture as a library. Three starting points. Three philosophies. And a gradual convergence toward a common architecture: a model-driven loop, running inside a graph that provided structure and checkpoints, with tool-level and sandbox-level enforcement replacing prescriptive orchestration.

"Everyone's building the same thing," Sam told Elena on a call. "They just started from different corners of the design space."

"Great," Elena said. "So which corner should I migrate to?"

"The one that gives you checkpointing, human-in-the-loop, and observability. Because those are the things you're going to wish you had when something goes wrong."

"You keep saying 'when,' not 'if.'"

"I do," Sam said. "Because I've talked to Marcus."

---

## Act 2: The Architecture

The three metaphors correspond to distinct execution models, and understanding them at a technical level clarifies both their strengths and their failure modes.

### The Chain Execution Model

A chain is a directed acyclic sequence of callable units. Each unit receives an input dictionary, performs its work, and returns an output dictionary that becomes the input to the next unit. The execution model is linear: call unit one, wait, pass to unit two, wait, pass to unit three.

There is no branching in the execution engine. Branching must be implemented by the developer inside a unit's code, which means the framework has no visibility into conditional paths. If a unit needs to retry on failure, the unit must implement its own retry logic.

**State management** is the chain's most significant limitation. Input-output dictionaries are the only mechanism for passing data between steps. If step three needs information from step one that isn't in step two's output, the developer must ensure step two passes it through — threading state by hand. For simple chains, this is trivial. For chains with ten or twenty steps, it's an exercise in defensive plumbing.

**Error handling** is manual. If step five fails, the entire chain fails. There is no built-in mechanism for catching the error, reverting to a prior state, and trying an alternative path. The developer can wrap the chain in try-catch and restart, but restarting a chain that has already sent emails, modified records, or called external APIs creates obvious problems.

Chains remain the right abstraction for well-defined workflows where the sequence is fixed and each step is idempotent. The mistake is reaching for a chain-like abstraction when the task requires branching, looping, or recovery.

### The Graph Execution Model

LangGraph replaced the chain with a **StateGraph**: a directed graph where nodes are functions and edges define control flow.

A LangGraph agent begins with a **state definition** — a typed data structure representing everything the agent needs to know. Every node received the current state, did its work, and returned a partial update. The framework merged the update into the state using configurable reducers.

**Edges** can be unconditional (always go from A to B) or **conditional** (examine the state and route to different nodes). The typical agent pattern is: an "agent node" calls the model, a conditional edge routes to a "tool execution node" if the model produced tool calls or to the END node if it produced a final response, and the tool node routes back to the agent node. This cycle is the agent loop expressed as a graph.

**Checkpointing** is implemented through a pluggable checkpointer interface. Before or after each node execution, the framework serializes the state and saves it to a backing store — SQLite for development, PostgreSQL or Redis for production. If the process fails, a new process loads the latest checkpoint and resumes.

**Human-in-the-loop** is implemented as an **interrupt**. The developer marks certain nodes as interrupt points. When the graph reaches one, it serializes the state, halts, and emits the state to the caller. From the graph's perspective, the human's input is just another state transition. The architecture doesn't distinguish between model decisions and human decisions — both are state updates.

The graph's deepest insight is that checkpointing, HITL, error recovery, and streaming are all consequences of the same design: explicit state, deterministic transitions, and serializable execution. The graph is not just an orchestration pattern. It is a runtime.

### The Loop Execution Model

The OpenAI Agents SDK implements the loop through a **Runner** that executes a tight cycle: send messages to the model, check the response, either return (if the model produced a final output) or execute tool calls and loop.

The Runner maintains a list of messages. At each iteration, it sends the messages plus the agent's instructions and tool definitions to the model. The model responds with text (final answer) or tool calls. If tool calls, the Runner executes each one, appends results to the messages, and loops.

**Handoffs** are special tool calls. When the model invokes a handoff, the Runner replaces the active agent with the target agent and continues. The new agent inherits the message history but has its own instructions and tools.

What the SDK notably lacks — by design — is persistent state. The Runner doesn't checkpoint. It doesn't save to a database. The message list exists in memory and is lost when the process ends. This is a deliberate bet that most interactions are short enough for in-memory state.

### The Convergence Pattern

By early 2026, the boundaries between these models were blurring. LangChain 1.0 shipped `create_agent` on top of LangGraph. DeepAgents built a Claude Code-inspired loop on LangGraph infrastructure. The OpenAI Agents SDK added session management and more sophisticated guardrails. Anthropic's Claude Agent SDK exposed a fully-equipped model-driven loop with built-in tools.

The converged architecture: a **model-driven loop** running inside a **stateful execution engine** (with checkpointing, streaming, and interrupt support) with **tool-level and sandbox-level enforcement**. Loop provides autonomy. Graph provides durability. Sandbox provides safety.

### The Cost Architecture

The choice of execution model has direct, often unintuitive cost implications.

A **chain** costs a fixed number of model calls — one per step, determined at design time. Predictable and easy to budget.

A **graph** costs a variable number — depending on the path taken. A graph with retry loops ranges from one call (no retries) to the maximum retry count. Bounded but variable.

A **loop** is unbounded by default. Without a maximum iteration count or token budget, the cost is unpredictable.

The most significant cost factor is **context growth**. Each iteration sends the accumulated state as input. A ten-turn interaction where each turn adds 500 tokens sends 500 input tokens on turn one, 1,000 on turn two, 1,500 on turn three. Total input cost: not 10x but approximately 55x the first turn. This quadratic growth is the hidden tax of long-running agents.

**Checkpointing** provides economic benefit beyond durability. Without it, a failed ten-step loop wastes the cost of all ten steps. With it, recovery costs only the steps after the last checkpoint. At Elena's 15% failure rate and $14,000 monthly model spend, the waste was roughly $5,000 per month — more than enough to justify the engineering investment in checkpointing infrastructure.

The broader principle: the infrastructure that feels like overhead — checkpointing, observability, durable execution — is not cost added on top of the agent. It is cost subtracted from the agent's failure modes. In a field where the most expensive thing is not running an agent but re-running a failed one, the harness is not overhead. It is savings.

Sam understood this. Elena would learn it the hard way. The lesson, when it came, would cost more than $5,000 a month.

---

*Next: [Chapter 3 — The Tool Problem](/book/chapter-03-the-tool-problem/)*
