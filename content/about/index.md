---
title: "About This Book"
description: "About The Harness: The Trust Boundary Between AI and the Real World"
date: 2026-04-13
layout: "about"
---

## The Harness

*The Trust Boundary Between AI and the Real World* is a practitioner's guide to building AI agents that act in the world safely and accountably.

The book follows a team of engineers — Sam, Elena, Marcus, David, and Nadia — as they build, break, and rebuild agent infrastructure across two years of rapid industry change. It is structured around a single thesis: the agent infrastructure stack has three layers, and most teams build two of them.

The orchestration layer (LangGraph, OpenAI Agents SDK, Claude Agent SDK, Google ADK) gets the attention. The execution layer (sandboxing, tool management, MCP) gets the budget. The trust layer — identity binding, policy enforcement, human oversight, and accountability — gets the incident postmortem.

This book is for the engineers and architects deciding what to build, how to buy, and what to hold off on. It covers:

- Why the industry converged from chains to graphs to loops, and what was lost in each migration
- How MCP became the tool connectivity standard, and what its security model requires
- How the major frameworks differ by binding constraint, not just by capability
- What sandboxing costs and what it protects against
- How the trust layer works: identity, policy, HITL, and audit
- What went wrong in the first generation of personal agents
- How skills and learning loops change the accountability picture
- What questions to ask before going to production

---

## The Characters

The book's narrative follows five people who are composites of real practitioners. None of them are real individuals; all of their problems are.

**Sam** is a founding engineer at a growth-stage startup building an AI-powered workflow platform. He is responsible for the agent infrastructure choices and lives with their consequences.

**Elena** is a senior engineer at an enterprise software company retrofitting AI agents into a product that was not designed for them. She deals with the gap between what marketing promised and what the infrastructure can safely deliver.

**Marcus** is a security engineer who got pulled into AI infrastructure after it became clear the standard application security model did not transfer cleanly.

**David** is a protocol designer who thinks in standards and interoperability. He is the one who spots the convergence patterns early and the one nobody listens to until after the incident.

**Nadia** is a staff engineer at a large financial institution navigating the intersection of agent capabilities and regulatory requirements. She is the one who asks where the policy enforcement actually lives.

---

## A Note on Timing

The events in this book span October 2022 to April 2026. Several technologies described here were new or experimental at the time they appear in the narrative; some have since become standard. The appendices (especially the timeline and architecture diagrams) reflect the state of the industry as of early 2026.

The infrastructure is still moving. The trust problems are not.

---

## This Site

Built with [Hugo](https://gohugo.io/) and the [Stack theme](https://github.com/CaiJimmy/hugo-theme-stack). Hosted on GitHub Pages.

Source: [igniting/the-invisible-stack](https://github.com/igniting/the-invisible-stack)
