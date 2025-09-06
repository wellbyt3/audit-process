# Audit Process

When I was first learning smart contract auditing, [watching Miletruck’s process](https://youtu.be/DySpPB3079k?si=2PON6jPrIIY_2pxa) was a game-changer. It provided structure to something that initially felt opaque. I started by copying his approach, then tweaked it over time to fit my own style.

Here’s what my audit process looks like now. I’m sharing publicly in case it helps other auditors. Enjoy!

## Table of Contents 📜
- [Part 1: Preparation & Setup](#mindsets)
    - [Setup](#setup)
    - [Entry Points](#entry-points)
    - [State Transition Doc](#state-transition-doc)
- .....work in progress

## Part 1: Preparation & Setup ⚒️

### 1. Reminders 🙌
Before I start an audit, I like to remind myself of what I've learned from previous audits. 

**Here are those reminders**:
1. "Slow is smooth, smooth is fast." Don't ever rush.
2. There's a difference between **_knowing of_** something versus **_understanding_** it. Be intellectually honest with yourself.
3. Do NOT start a contest midway through unless you're 100% certain you'll be able to finish early.
6. Treat yourself like a professional athlete during contests. Consistent bedtime, eat clean, workout, etc.
7. Too much caffeine in the AM hurts producitvity in the mid term. It's not worth it.
8. Creativity usually comes during idle time. Build idle time into each audit day.

### 2. Get Organized 📁
Spending 10 minutes to get organized at the start of an audit, saves many future hours.

Setup steps:
- [ ] Create "Notes" document
- [ ] Create "Core Flows" document
- [ ] Create "Audit Tracker" document
- [ ] Create "State Transitions" Google Sheet
- [ ] Create a new blank project in Whimsical
- [ ] Clone the repo and compile

### 3. Scan the Docs and README 👀
- [ ] Quickly scan the documentation and README. The goal is NOT total understanding. Just get a lay of the land.

### 4. Entry Points ⬆️
- [ ] Open the codebase and the "Core Flows" document
- [ ] Write each in-scope contract into the "Core Flows" doc
- [ ] Run `forge inspect abi ContractName` and write down `external` and `public` state changing function.
- [ ] Organize contracts and functions into an order that makes intuitive sense (e.g. `Factory.sol` before `Pool.sol`; `deposit()` before `withdraw`). Doesn't need to be perfect. Easy to reorder as you learn more.
- [ ] Paste the contracts and functions into the "Audit Tracker" doc (for use later during manual review).

### 5. State Transition Doc ↪️
- [ ] Run `forge inspect storageLayout ContractName` for each contract.
- [ ] Add each storage variable to the "State Transitions" Google Sheet.

> Visualizing storage variable state in a spreadsheet helps me a lot. I thought I was the only one doing this, but then I learned [Phil usees spreadsheets](https://x.com/philbugcatcher/status/1909428628015788501) and Obront tracked the state of storage variables when he won the Story Protocol contest.

## Part 2: System Overview 💡

### 1. Get a "Birds Eye View" 🦉
The goal here is to understand _what_ the system is doing, but ignore any _how_ related details. 

Using the "Core Flows" document:
- [ ] For each contract, write 1-2 sentences explaining its purpose in the system.
- [ ] For each entry point (function), write 1-2 sentences explaining it's purpose in the system. 

> **IMPORTANT**: This is NOT a line-by-line pass through the code. I'm scanning for things like fund transfers and how the different contracts interact. The goal is to understand what the system is designed to do at a high-level.

If there are many contracts interacting, draw simple diagrams in Whimsical showing the interactions:
- [ ] How the different contracts interact
- [ ] The actors
- [ ] How funds move between actors and contract

> **IMPORTANT**: Diagraming can be a form of procrastination. Beware! 

### 2. Fill in Knowledge Gaps 📚
For me, understanding a codebase has to happen top-down. If I don’t know what it’s supposed to do or why it exists, the actual code won’t make much sense.

By this point, I'll have flagged any concepts from the documentation or code that I’m not familiar with. Now is the time to fill in any of those knowledge gaps I have. For example, if a protocol implements call and put options and I don’t know how options work, I’ll researcher options.

Whenever I learn something new for an audit, it's important that the new knowledge is accessible to me from memory. I don't want to have to look up concepts / definitions when I'm manually reviewing the code because it disrupts my flow, making me audit at a level lower than my potential.

To lock in new concepts, I make flashcards in a spaced-repetition app called Mochi. Each morning, I review those cards so the new concepts are fresh and accessible.

### 3. Prepare a Testing Environment 🧑‍💻
I learn best by doing. When I’m manually reviewing code, it really helps to have a Foundry test file ready so I can play around with different parts of the code:
- [ ] Create a Foundry test file called `wellbyt3-playground.t.sol` and setup a simple testing playground

Next, review the protocol's tests. Depending on what I see, either:
- [ ] Plan on using the existing test suite for my own future tests, OR
- [ ] Setup my own testing environment.

A lot of this decision comes down to how much I trust the protocol’s test setup and how complex the deployment and state initialization are. If they’re using a bunch of mocks that strip away real complexity, I’ll usually spin up my own environment by forking the actual contracts they interact with. [This unique Medium from the Plaza Finance contest on Sherlock](https://github.com/sherlock-audit/2024-12-plaza-finance-judging/issues/835) is a good example — it could've been caught by just running a forked test.

The other factor is time. Setting up my own environment is great for understanding how state gets initialized, but it can also eat up hours. If the deployment is too complex, it’s sometimes just not worth it.

## Part 3: Manual Review 🔎
With a high-level system overview loaded up in the 🧠 , it's time to start the line by line manual review.

**Here are some general tips for manually reviewing code:**
1. Audit in layers, top-down
2. When there's confusing math, convert the code to formulas on paper.
3. When timelines exists, draw them on paper.

### 1. Happy Pass 😃

Open the "Audit Tracker" doc and being the manual review:
- [ ] Review the constructor / initializers to understand state initialization
- [ ] Define all state variables. If any are non-intuitive, make flashcards for them.
- [ ] For each entry point, define input parameters.
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
- [ ] Identify state updated. Log them in the State Tracker Google Sheet.

> **IMPORTANT**: During the first pass, focus on keeping a steady pace. Don’t let details or complexity break your rhythm. If something doesn’t click after a reasonable amount of time, tag it and move on. Easy to revisit it on the next pass.

### 2. Attacker Pass 😉
Go through the code again, but shift mindsets from "happy path" to "attacker's mindest."

Go through the code in the same order as before, but this time stop at anything tagged and follow those threads to completion. Sticking to the same order builds deeper context than just jumping around tags. During this pass, I usually come up with new questions and ideas—and instead of tagging them, I dig in until I’ve followed the thread all the way through.

Open the "Audit Tracker" doc and being the manual review:
- [ ] Review the constructor / initializers to understand state initialization
- [ ] For each entry point, define input parameters. Make sure to go through all possible codepath variations.
- [ ] Audit in layers
- [ ] When you come across a tag, take whatever action is required
    - Write up all bugs as I come across them.


## Part 4: Bug Hunting 🐛
By this point,

### Question Bank Review
