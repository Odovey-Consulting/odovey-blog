---
title: "Harness Engineering: Building the Body of an AI Agent"
date: "2026-10-14"
excerpt: "Every AI agent is a model inside a harness. The model thinks; the harness observes, acts, and decides when to stop. Here's what a harness is made of, how to build one primitive at a time, and what it takes to get it ready for production."
author: "Chase Dovey"
tags:
  - Harness Engineering
  - AI Agents
  - Agentic AI
draft: false
---

Most conversations about AI agents start with the model: which one is smartest, which one is cheapest, which one tops this month's leaderboard. But the model is only half of an agent, and it's rarely the half that decides whether an agent works in production.

The other half is the **harness**. Harness engineering is the practice of building it, and it's the foundation of how we build and deploy agents at O'Dovey AI. We've published an open curriculum, [Harness Engineering](https://github.com/averagejoeslab/harness-engineering), that builds an agent from scratch, one piece at a time. This post walks through the ideas behind it.

## What is an agent?

An agent is a system that loops over three steps: it **observes** its inputs, **thinks** about them, and **acts**. Then it repeats, with each turn starting from what the last one taught it.

You run this loop yourself all the time. Imagine waking up somewhere completely dark. You notice a floor under your feet (observe), wonder whether you're in a room (think), and reach out (act). Your hand meets a wall, which suggests a room, which suggests a light switch. You slide your hand along the wall, find the switch, flip it, and the loop starts again with whatever you see next.

In that story, your mind did the thinking. Your body did the observing and acting. Agents have the same two parts.

## The model is the mind

A model does one thing: tokens go in, and it predicts the next one. That token is appended to what it reads, and it predicts again, one token at a time, until it predicts that it should stop.

```
TokensOut = Model(TokensIn)
```

That's all a model does. It can't capture its own inputs, assemble what it's shown, act on what it writes, or decide when the work is done. A model without a harness is a brain in a jar.

## The harness is the body

The harness does everything else. It captures inputs, assembles context, requests a response from the model, handles the model's outputs, and defines the shape of the loop:

```
┌──────────────────────────── harness ────────────────────────────┐
│                                                                 │
│  input ─► context ─► ┌───────── model ─────────┐ ─► output ─┬───┼─► world
│    ▲                 │ tokens in ─► next token │            │   │
│    │                 │      ▲            │     │            │   │
│    │                 │      └── append ◄─┘     │            │   │
│    │                 └─────────────────────────┘            │   │
│    └─────────────────────── what happened ◄─────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

Every harness, however sophisticated, reduces to five primitives:

| Primitive | What it does |
|---|---|
| **Input** | Captures what comes in: a message, an event, the result of an action |
| **Context** | Assembles what the model is shown: instructions, memory, the task |
| **Model interface** | Requests a response from the model: a hosted API, a local model, a router |
| **Output** | Handles the model's outputs: a reply, a tool call, a change to a real system |
| **Control flow** | Defines the shape of the loop: go again, hand back to a human, stop |

Every agent you've used, from a coding assistant to a chatbot that books flights, is a model inside these five primitives. They differ in how each one is built.

## Building a harness, one primitive at a time

The curriculum builds a harness around a single, concrete task. A tiny project has a bug: a function that should apply a percentage discount to a basket gets it wrong, and its test fails. The request is always the same: *"The tests are failing. Find out why and fix it."* Each step adds one primitive and shows exactly how far the agent gets, and what stops it.

**1. Reach the model.** The model interface sends the task and prints the response. The model answers, but it can't see the code, so it politely asks you to paste it. It can think, but it can't act.

**2. Let someone ask.** Input replaces the hardcoded request with whatever a person types. Now anyone can ask, but the model can still only reply in words.

**3. Let it act.** Output gives the model a single tool, a shell, and runs the commands it asks for. It runs a command, but what the command prints goes nowhere, so it can't take a second step.

**4. Let it keep going.** Control flow sends each result back to the model and goes again until the model says it's done. The agent reads the code, runs the tests, finds the bug, and fixes it. **This is the moment it becomes an agent.**

**5. Give it context.** The agent works, but it doesn't know what it is or how best to use its body. Context fixes that. It's the agent's working memory: whatever isn't in context, the agent doesn't have in mind. So deciding what goes there, which role each piece sits in, and how to make room when the window fills shapes everything the agent does. One rule matters more than the rest: a piece of context should never carry more authority than its source.

With all five primitives in place, the harness is complete, and it fits in a single readable file.

## Proving it works

An agent that fixed a bug once has proven very little. Any change to the instructions, the model, or the tools can change what the agent does. So the curriculum adds an evaluation: a set of tasks with known right answers, each run in a fresh copy of the project, **checked by what the agent actually did, not by what it says it did**. Run it after every change, and a failing task tells you where to look.

## Getting it ready for production

The agent now works and passes its evaluation. But would you leave it alone? It forgets everything when a session ends. It runs any command on your machine, unasked. You can't see what it did. One stuck command hangs it forever. Every request costs more than it needs to. And after a change, you can't tell whether it still works.

Getting a harness ready for production means asking six questions of every primitive:

| Question | Concern |
|---|---|
| What should outlast the session? | Persistence |
| What could do harm, and who could make it? | Safety & Security |
| What did it actually do? | Observability |
| What if something breaks? | Resilience |
| Why is it slow or costly? | Performance |
| How do we know it still works? | Evaluation |

Each answer adds a few lines to the harness. A few examples from the curriculum:

- **Persistence:** the agent keeps its memories as plain files and has to recall them before it can use them, the same way a person does. If a session ends mid-task, the harness offers to pick up where it left off.
- **Safety:** commands run in a sandboxed container that sees only the project folder and has no network, and nothing runs without a human's yes.
- **Security:** everything an agent reads becomes tokens in its context, and a model can't reliably tell an instruction from text that only looks like one. So secrets are hidden from the sandbox entirely, and the agent is told whose instructions to trust. A dedicated set of attack tasks checks that a planted instruction can't turn the agent against you.
- **Observability:** every request and command is written to a record the agent can't modify.
- **Resilience:** failed requests are retried, and runaway commands are stopped.
- **Performance:** prompt caching means each request pays full price only for what's new, responses stream so you see the first words immediately, and independent commands run in parallel.

None of these answers is exotic. But together, they're the difference between an agent that works in a demo and one you can trust with real systems.

## Why this matters

Models will keep changing every few months. The five primitives won't. When you understand an agent as a model inside a harness, a lot of decisions get clearer:

- **Model choice becomes swappable.** The model interface is one primitive. Design it well, and changing models is a contained change, not a rewrite.
- **Reliability becomes an engineering problem.** Most production failures, like lost state, runaway commands, prompt injection, and spiraling cost, live in the harness, not the model.
- **Your harness becomes the asset.** Your tools, context, guardrails, and evaluations encode how your organization works. That's the part worth owning.

Open up any agent, from this one to the coding assistants you use every day, and you'll find the same five primitives, built with more or less care.

## Try it yourself

The full curriculum is open source: [github.com/averagejoeslab/harness-engineering](https://github.com/averagejoeslab/harness-engineering). It includes the code for every step, the evaluation and security tasks, the production harness, and slides for teaching it. The code is MIT-licensed, and the written course is CC BY 4.0.

If you'd rather have help building a harness for your organization, or want your engineering team to learn harness engineering hands-on, [get in touch](https://odovey.com/contact/). That's exactly what we do.
