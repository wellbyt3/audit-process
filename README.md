# Audit Process

When I was first learning smart contract auditing, [watching Miletruck’s process](https://youtu.be/DySpPB3079k?si=2PON6jPrIIY_2pxa) was a game-changer. It provided structure to something that felt opaque. I started by copying his approach, then tweaked it over time to fit my own style.

Here’s what my audit process looks like now. I’m sharing publicly in case it helps other auditors. Enjoy!

## Table of Contents 📜
- [Part 1: Preparation & Setup](#part-1-preparation--setup-%EF%B8%8F)
    - [Reminders](#1-reminders-)
    - [Get Organized](#2-get-organized-)
    - [Scan Documentation & README](#3-scan-documentation--readme-)
    - [Entry Points](#4-entry-points-%EF%B8%8F)
    - [State Transition Doc](#5-state-transition-doc-%EF%B8%8F)
- [Part 2: System Overview](#part-2-system-overview-)
    - [Get a "Birds Eye View"](#1-get-a-birds-eye-view-)
    - [Fill in Knowledge Gaps](#2-fill-in-knowledge-gaps-)
    - [Prepare a Testing Environment](#3-prepare-a-testing-environment-)
- [Part 3: Manual Review](#part-3-manual-review-)
    - [Happy Pass](#1-happy-pass-)
    - [Attacker Pass](#2-attacker-pass-)
- [Part 4: Bug Hunting](#part-4-bug-hunting-)


## Part 1: Preparation & Setup ⚒️

### 1. Reminders 🙌
Before I start an audit, I like to remind myself of what I've learned from previous audits. 

**Here are those reminders**:
1. "Slow is smooth, smooth is fast." Don't ever rush.
2. There's a difference between **_knowing of_** something versus **_understanding_** it. Be intellectually honest with yourself.
3. Do NOT start a contest midway through unless you're 100% certain you'll be able to finish early. You never want to feel rushed.
6. Treat yourself like a professional athlete during contests. Consistent bedtime, eat clean, workout, etc.
7. Too much caffeine in the AM hurts productivity. That second cup is not worth it.
8. Creativity usually comes during idle time. Build idle time into each audit day.
9. Don't do too much in your head. When things get complicated, rely on your tools (e.g. paper + pen, Foundry, spreadsheets, notes, etc.).
10. When getting comfortable with a codebase, remember all a function can do is update state or returning a value. Zoom out before zooming in.
11. Forked Contracts != Less Bugs
12. Clear writing = Clear thinking. Take notes often. Anything critical, add to spaced repetition practice.

### 2. Get Organized 📁
Spending 10 minutes to get organized at the start of an audit, saves many future hours.

Setup steps:
- [ ] Create `Notes` document
- [ ] Create `Core Flows` document
- [ ] Create `Audit Tracker` document
- [ ] Create `State Transitions` Google Sheet
- [ ] Create a new blank project in Whimsical
- [ ] Clone and build the repo

### 3. Scan Documentation & README 👀
- [ ] Quickly scan the documentation and README. The goal is NOT total understanding. Just get a lay of the land.

### 4. Entry Points ⬆️
- [ ] Open the codebase and `Core Flows` document
- [ ] Write each in-scope contract into `Core Flows`
- [ ] Run `forge inspect abi ContractName` and write down `external` and `public` state changing function.
- [ ] Organize contracts and functions into an order that makes intuitive sense (e.g. `Factory.sol` before `Pool.sol`; `deposit()` before `withdraw`). Doesn't need to be perfect. Easy to reorder as you learn more.
- [ ] Paste the contracts and functions into the `Audit Tracker` doc (for use later during manual review).

### 5. State Transition Doc ↪️
- [ ] Run `forge inspect storageLayout ContractName` for each contract.
- [ ] Add each storage variable to the `State Transitions` Google Sheet.

> Visualizing storage variable state in a spreadsheet helps me a lot. I thought I was the only one doing this, but then I learned [Phil uses spreadsheets](https://x.com/philbugcatcher/status/1909428628015788501) and many others.

## Part 2: System Overview 💡

### 1. Get a "Birds Eye View" 🦉
The goal here is to understand _what_ the system is doing, but ignore any _how_ related details. 

Using the "Core Flows" document:
- [ ] Go through each contract and determine how it fits into the system at a high-level
- [ ] Review entrypoint functions and determine how they fit into the system

> **IMPORTANT**: This is NOT a line-by-line pass through the code. Scan for things like fund transfers and how the different contracts interact. The goal is to understand what the system is designed to do at a high-level.

If there are lots of contracts interacting, draw diagrams in Whimsical showing the interactions:
- [ ] How the different contracts interact
- [ ] The actors
- [ ] How funds move between actors and contract

> **IMPORTANT**: Diagraming can be a form of procrastination. Beware! 

### 2. Fill in Knowledge Gaps 📚
For me, understanding a codebase has to happen top-down. If I don’t know what it’s supposed to do or why it exists, the actual code won’t make much sense.

By this point, I'll have flagged any concepts from the documentation or code that I’m not familiar with. Now is the time to fill in any of those knowledge gaps I have. For example, if a protocol implements call and put options, but I don’t know how options work, I’ll research options. While I'm learning, I'll take detailed notes on anything that's new to me.

### 3. Spaced Repetition

Whenever I learn something new for an audit, it's important that the new knowledge is accessible in memory. I don't want to have to look up concepts / definitions when I'm manually reviewing the code because it disrupts my flow, making me audit at a level lower than my potential.

To lock in new concepts, I make flashcards in a spaced-repetition app called [Mochi](https://mochi.cards/). Each morning, I review those cards so the new concepts are fresh and accessible.

> TIP: Ask your LLM of choice, “Teach me X using Socratic tutoring. Do not move on until I've answered the question to your satisfaction.“

### 4. Prepare a Testing Environment 🧑‍💻
I learn best by doing. When I’m manually reviewing code, it really helps to have a Foundry test file ready so I can play around with different parts of the code:
- [ ] Create a Foundry test file called `wellbyt3-playground.t.sol` and setup a simple testing playground

Next, review the protocol's tests. Depending on what I see, either:
- [ ] Plan on using the existing test suite for my own future tests, **OR**
- [ ] Setup my own testing environment.

This decision comes down to the protocol’s tests and how complex their deployment is:
- If the delpoyment is simple, I'll create my own testing environment because it helps me gain a deeper understanding of the system.
- If the protocol's tests use mocks that strip away complexity, I’ll spin up my own environment and replace the mocks with forked contracts. [This unique Medium from the Plaza Finance contest on Sherlock](https://github.com/sherlock-audit/2024-12-plaza-finance-judging/issues/835) could've been caught by just running a forked test.
- If the deployment is complex, it’s usually not worth setting up my own environment, so I’ll use the protocol’s.

## Part 3: Manual Review 🔎
With a high-level system overview loaded up in the 🧠 , it's time to start the line by line manual review.

**Here are some general tips for manually reviewing code:**
1. Audit in layers, top-down
2. When there's confusing math, convert the code to formulas on paper.
3. When timelines exists, draw them on paper.
4. If a function has logic to handle different inputs, pick an input, then go through the function thinking only about that input. Repeat this for every input.

### 1. Happy Pass 😃

Open the "Audit Tracker" doc and being the manual review:
- [ ] Review the constructor / initializers to understand state initialization
- [ ] Define all state variables. If any are non-intuitive, make flashcards for them.
- [ ] For each entry point, define input parameters. If any are non-intuitive, make flashcards for them.
    - Use the protocol's tests to find "happy path" input params.
    - When a function has many different code paths for different inputs, write them all down but only audit one path at a time.
- [ ] Audit in layers.
- [ ] Use the following tags:
    - `@audit-issue`: Use to tag bugs
    - `@audit-check`: Use to tag action items, tasks, attack ideas, or questions I need to answer.
    - `@audit-info`: Use to tag important context about the code that I want to remember for future passes.
    - `@test`: Use to tag portion of the code I want to test or fuzz later. If something takes less than 2 minutes to test, do it immediately.
    - `@complex`: Use to tag areas where bugs are likely more common. Examples include fees, thresholds, external integration, complex branch logic, etc.

When you finish an entry point, go back through and:
- [ ] Identify state updates. Log them in the `State Tracker` Google Sheet.
- [ ] Review state update and ask, "Should this state variable have been updated?"

> **IMPORTANT**: During the first pass, focus on keeping a steady pace. Don’t let details or complexity break your rhythm. If something doesn’t click after a reasonable amount of time, tag it and move on. Easy to revisit it on the next pass.

### 2. Attacker Pass 😉
Now that you've been through the code once, your mindset should naturally starts shifting toward an "attacker's mindest."

Go through the code in the same order as before, but this time stop at anything tagged and follow those threads to completion. Sticking to the same order builds deeper context than just jumping around tags. If you come up with new questions and ideas, instead of tagging, dig into them until each threads has been pulled on fully.

Open the `Audit Tracker` doc and begin the 2nd pass:
- [ ] Review the constructor / initializers to understand state initialization
- [ ] For each entry point, define input parameters. Make sure to go through all possible codepath variations.
- [ ] Audit in layers
- [ ] When you come across a tag, take whatever action is required (e.g. test, explore external integrations, etc.)
    - Write up all bugs as you come across them.

## Part 4: Bug Hunting 🐛
By this point, you should understand all the intimate details of the protocol you're auditing. Time to hunt for those non-obvious bugs!

- [ ] Anything tagged with @complex should really be scrutinized
- [ ] Go through `Question Bank` doc
- [ ] Try on [different Auditing Mindsets](https://www.youtube.com/watch?v=kORcOhftwVw)
    - "Singular mindset." Focus on one critical state variable and look where it's updated.
    - "It's be really bad if X happened." Can I prove it's impossible?

> **CONTEXT**: My question bank was [inspired by Zach's](https://youtu.be/57V-57ZXmfA?si=7ZhPp3tsGUfiQkFg&t=1371). Whenever I finish an audit and learn what I missed, I add to it.
