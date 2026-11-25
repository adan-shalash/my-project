Degree level: Bachelor’s

# Project proposal

## 1. The problem and goal

(Explain the problem you want to solve and why it matters. Describe what you
want to improve, how you will measure improvement, and which limits or
requirements must be respected. A short example can help someone outside your
field understand the problem.)

**The problem and goal in my project:**

Before a chip design is manufactured, verification engineers must show that it was tested in every situation that matters. The list of those situations is the functional coverage model: a set of bins, each one a situation that must be seen at least once in simulation. Random test inputs fill the easy bins fast and then stop making progress, because the remaining bins need a specific sequence of events that random inputs almost never produce. Closing those last bins by hand is one of the most time-consuming parts of verification.

My design under test is a 4-channel arbiter: a circuit that decides which of four requesters gets a shared resource. It serves them in round-robin order, and a requester that has waited too long is promoted ahead of the others (priority aging). A hard bin is, for example: two requesters reach the aging limit in the same cycle, while the round-robin pointer points at a third, and a reset arrives one cycle later.

Goal: reach 100% functional coverage with as few simulated cycles as possible. Measure: coverage reached within a fixed total budget of simulated cycles, and the number of simulation runs needed to reach 100%. Limits: every run must pass the scoreboard (the design's outputs must match a reference model that I write); the inputs must be legal for the design's interface; the budget is enforced by the tool, not by the agent.

## 2. The simulator

(Describe a program or model that could simulate your system and calculate what
happens under different conditions. Explain what inputs it takes and what
results it produces. State whether it can run on your computer and accept inputs
in a text file, such as JSON or YAML. If you have not chosen a simulator yet,
name an option you want to explore and explain what you need to check.)

**The simulator in my project:**

Icarus Verilog, an open-source simulator for hardware designs written in Verilog, driven from Python by cocotb (an open-source library for writing testbenches in Python) with cocotb-coverage for the coverage model. All three are free and run on my computer under Ubuntu.

Inputs: the arbiter's Verilog source (fixed), and a JSON file with the input sequence: for each clock cycle, which requesters are asking and whether reset is active. Results: pass or fail from the scoreboard, a coverage report listing every bin and how many times it was hit, and a waveform file with the value of each signal in each cycle. The random seed is fixed, so the same JSON file always gives the same results.

What I still need to check: that the current versions of cocotb and cocotb-coverage work together; that one run takes about a second; that two runs of the same JSON file give identical coverage reports; and that I can start a run from an ordinary Python function, since cocotb normally runs inside the simulator.

## 3. The agent's role

(Explain what the language-model-based agent will decide or change, and how it
will use the simulator's results to choose its next action. If known, list the
variables it may change and their allowed ranges. Explain what you expect the
language model to contribute: could a simple program that tries different
options and picks the best result achieve the same goal? Include one question
you still need to answer.)

**The agent in my project:**

The agent decides the input sequence for the next run. It may read the arbiter's specification, its Verilog source and the coverage model. It has three tools: run a sequence (returns pass or fail and the updated coverage), list the coverage holes (the bins not yet hit, and the hit counts of nearby bins), and inspect the waveform over a window of cycles for chosen signals. What it may change: for each of up to 150 cycles per run, the four request bits and the reset bit.

What I expect the model to contribute is planning a path to a rare state. To hit the example bin above, it has to work out that two requesters must start waiting in the same cycle, that a third must keep the resource busy for exactly the aging limit, and that the pointer must be in the right position first. When a run misses, the waveform shows where the plan went wrong, and the agent corrects it.

The for-loop question: pure random inputs will lose, but that is too easy a comparison. A stronger simple program keeps every sequence that hit a new bin, changes it slightly and tries again (coverage-guided mutation, the method hardware fuzzers use). My evaluation will compare the agent against random inputs and against this program, all with the same total budget.

Question I still need to answer: does the agent beat coverage-guided mutation on the deep bins of this arbiter, or is the design too simple to need reasoning? If it is too simple, I will make the aging rules or the coverage model harder.

## Explain your project to someone outside your field

(Now describe your whole project again, as if you were explaining it to someone
from another field who has no background in your topic. For example, if you
study chemical engineering, imagine explaining it to a physicist; if you study
aeronautical engineering, imagine explaining it to someone in medicine; if you
study data science, imagine explaining it to a materials engineer. In one or two
short paragraphs, connect all three parts: the problem you want to solve, how
the simulator helps you test ideas, and what the agent will decide or change.
Use everyday language, explain any technical terms you need, and avoid
unexplained abbreviations. Do not simply copy your answers above.)

**My project in simple words:**

Every phone and computer runs on a chip: an electronic part the size of a fingernail that does all the work. Building a chip costs a great deal of money, and once it is built it cannot be repaired. So before building it, engineers test its design with a program that pretends to be the chip, the way a flight simulator pretends to be a plane. My project tests one small piece of a chip. Its job is like a teacher deciding which of four children gets the one swing next: the children take turns, and a child who has waited too long goes first. To trust this piece, I have a checklist of situations I must see it handle correctly. Most of them are easy to see. A few are rare, for example two children reaching their waiting limit at exactly the same moment. If the children ask for the swing at random, the rare situations almost never happen, and the checklist stays unfinished.

In my project, an AI program like the ones behind chat assistants makes the rare situations happen. It sees which checklist items are still missing, thinks about which order of requests would cause one of them, and tries that order on the pretend chip. If it does not work, the AI looks at what happened step by step, understands its mistake and tries again. It continues until the checklist is complete, using as few tries as possible.
