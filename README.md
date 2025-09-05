# Audit Process

## Intro
When I was first getting into smart contract auditing, [watching Miletruck’s process](https://youtu.be/DySpPB3079k?si=2PON6jPrIIY_2pxa) was a game-changer. It gave me some structure around something that initially felt pretty opaque. I started by copying his approach, then tweaked it over time to fit my own style.

Here’s what my audit process looks like now. I’m sharing it in case it helps other auditors who are learning. Whenever people drop tips about their process on Twitter, it’s been super helpful for me—so I figured I’d put mine out there too.

## Mindsets
1. "Slow is smooth, smooth is fast."
2. There's a difference between "knowing of" something versus understanding it.
3. A deep understanding of the protocol is what allows you to quickly understand if your ideas are any good or not.
4. I feel slow at understanding a codebase. Because contests are timeboxed, I start getting stressed I'm not going to finish. To avoid this, start contests early if possible and only start a contest if I at least start the day it drops. Otherwise, wait for a new one to start. it's not worth it.

## Setup
When I spend the time at the start of an audit to get organized, it saves me hours of time in the future.

Setup steps:
- [ ] Create "Notes" document
- [ ] Create "Core Flows" document
- [ ] Create "Project Tracker" document
- [ ] Create "State Transitions" Google Sheet
- [ ] Create a new blank project in Whimsical
- [ ] Clone the repo and compile

## Entry Point Identification
1. Open the codebase and the "Core Flows" document
2. Write each contract that's in scope in the "Core Flows" document
3. Run `forge inspect abi ContractName` on each contract and write down each `external` or `public` function.
4. Organize contracts and functions into an order that makes intuitive sense (e.g. `Factory.sol` before `Pool.sol`; `deposit()` before `withdraw`). Doesn't need to be perfect. Easy to reorder as you learn the codebase.
5. Paste the contracts and external / public functions into the "Project Tracker" document

## State Transition Document Setup
1. Run `forge inspect storageLayout ContractName` for each contract
2. Add each storage variable to the "State Transitions" Google Sheet

## High-Level Mental Model
The goal here is to understand "what" the system is doing, but any "how" related details.

Have open the codebase, the protocol's documentation, the Core Flows document, and Whimsical.

Using the "Core Flows" document:
1. Go through each contract and write 1-2 sentences explaining its purpose in the system.
2. Go through each external / public function and write 1-2 sentences explaining it's purpose in the system.

Important: This is NOT a line-by-line pass through the code. I'm scanning for things like fund transfers and how the different contracts interact with the goal of understanding what the system is designed to do from a high-level.

In Whimsical, draw simple diagrams showing:
1. How the different contracts interact
2. The actors
3. How funds move between actors and contract

Important: Only draw diagrams in situations where there are many different contracts interacting with each other. 

## Learn
Understanding a codebase for me needs to happen top down. If I don't understand "what" a codebase is supposed to do, or "why" it's doing it, it's difficult for me to understand the code. 

During the "High-Level Mental Model" phase, I'll usually identify any concepts I'm not familiar with. Now is the time to learn about those topics.

For example, if the protocol is implementing call and put options, but I don't know what options are, I'll fill in that gap of understanding.

Whenever I learn something new for an audit, it's really important that the new knowledge is accessible to me in memory (i.e. without me needing to look it up). The reason for this is that when you need to look something up / remember something, it breaks your flow, which I've found is important while manually reviewing code. 

To make sure all this new knowledge is accessible in my memory, I create flashcard using a spaced repetition software called Mochi. Each morning of an audit, I study all these flashcards which loads the information I'll need into memory.

Note: Sometimes I completely skip this step if I'm auditing a protocol without any new concepts. Other times, I spend a whole day or two if I'm auditing something with a lot of new, novel concepts. 

## Setup a Testing Environment
I learn best by doing. When I'm manually reviewing code, it's very beneficial if I have a Foundry test file ready for me to play with different pieces of the code in. 
- [ ] Create a Foundry test file called `wellbyt3-playground.t.sol` and import the Forge library

Next, I look at the protocol's tests. Depending on what I see, I'll either:
1. Plan on using the existing test suite for my own future tests, OR
2. Setup my own testing environment.

A lot of this decision depends how much I trust the protocol's initialization of their testing environment and how complex the deployment and state initialization of the protocol is. For example, if they're using a lot of mocks that remove a lot of complexity, I'll often create my own envirnoment forking the actual contracts their contracts will be interacting with. [This bug from the Plaza Finance contest on Sherlock](https://github.com/sherlock-audit/2024-12-plaza-finance-judging/issues/835) was a unique and could have bveen found by simply setting up a forked test.

Time is the other deciding factor. Seting up my own testing environment is great to build understanding of how state is initialized, but I don't want it to take too long. Sometimes, the deployment of the protocol is too complex and would take too much time to setup my own testing environment.

## Manual Review (Happy Pass)
Now that I've got a high-level overview of the system, I know how I'm going to want to approach the first pass of my manual review.

Open the "Project Tracker" document and being manually reviewing each contract and entry point. 

Notes:
-Always review the constructor / initializers first so I know how state is initialized
- For each entry point, make sure to define what input parameter looks like. Use the protocol's test to find the "happy path" input values.
- Audit in layers, making sure to write each internal / external call in the project tracker to avoid distraction (create a document showing how to actual do this with some examples).
- Use the following tags:
  - @audit-issue when I find a bug
  - @audit-check when there's some action item / task I'm assigning myself (e.g. attack ideas, questions I need to answer, etc.)
  - @audit-info when there's an important piece of information about the code that I need to remember.
  - @test when I want to test something in the playground or testing environment that will take longer than a minute or two to test.
  - @complex when I notice an area where there's a lot of complexity that is likely to contain bugs. I'll usually tag external integrations as @complex as well.
- When there are many state variables tracking lots of different things or state variables that are unintuitive, memorizing what they represent really helps as you get deeper into a codebase. Once I learn what a state variables represents, add the definition to the "Notes" document and then at the end of the day, create spaced repetition flashcards.
- When a function has logic to handle many different inputs, pick an input, then go through the function only thinking about that input. Repeat this for every input. Notate each variation of input in the "Project Tracker" first, then go through each one by one.
- Once I finish going through an entry point, go back through, but only identifying when state is updated. Log these state updates in the state tracker so I can visually see how state is changing.

Some situational approaches I've found helpful:
- When there's a lot of math, convert the code to formulas on paper.
- When timelines exists, draw them on paper.

IMPORTANT: Focus on keeping a steady pace during this first pass. Don't allow complexity / details to disrupt the rhythm. Whenever something is confusing and I don't understand it within a reasonable amount of time, tag it with @audit-check to come back to on the next pass.

## Manual Review (Attacker Pass)
Now that I've been through every line of code once, consciously make the shift from "happy path" mindset to an "attacker's mindset."

During this pass, approach it in the same order as the first pass, but this time stopping at anything that is tagged and following those threads to their completion. Going in the same order helps build deeper and deeper context into the codebase I've found versus just randomly going back to your tags and reviewing the ones you feel like reviewing. 

I'll usually come up with more questions and ideas during this pass. Instead of tagging for later, I'll follow the thread to its completion.

Now is also the time to test anything tagged with @test.

When I come across a bug in this phase, I'll almost always write it up immediately.

## Bug Hunting
