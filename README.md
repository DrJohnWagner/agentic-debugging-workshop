# Building an AI Agent from Scratch

A short workshop in which you write an AI agent — the whole thing, about fifteen
lines — and watch it find and fix a bug in a real file on your disk.

Part 1 uses no frameworks at all. The point is to see the mechanism before anything
hides it from you. Parts 2 to 5 then rebuild the identical agent four more times, on
Microsoft Agent Framework, CrewAI, LangGraph and smolagents — so you can watch your
loop get hidden, drawn as a graph, and finally replaced by a Python interpreter, while
the bug it fixes never changes.

## The idea

A language model cannot do anything. It can only read the descriptions of the tools
you give it and reply *"call this function with these arguments"*. Your code reads
that reply, runs the function, and hands the result back.

That loop is the agent. Everything marketed as an "AI agent" is this, with more
functions in the dispatch table.

## Setup

You need Python 3.10+, VS Code with the Jupyter extension, and an
[OpenRouter](https://openrouter.ai/) API key.

```bash
git clone https://github.com/DrJohnWagner/agentic-debugging-workshop.git
cd agentic-debugging-workshop
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env               # then edit .env and paste your key
```

Then open `notebooks/agent_workshop_student.ipynb` and select the `.venv` kernel.

Put your key in `.env` as `OPENROUTER_API_KEY`. The notebooks load it with
`python-dotenv`, `.env` is git-ignored, and the key never lands in the notebook
file or your shell history. Do not paste your key into a cell.

### Check your setup before the session

Run the first two cells. If a haiku prints, you are ready. If it doesn't, sort it out
before the workshop rather than during it.

## What's in here

```
notebooks/
    agent_workshop_student.ipynb       # part 1 — the loop is left as TODOs, work in this one
    agent_workshop_instructor.ipynb    # part 1 complete, plus teaching notes and variations
    microsoft_student.ipynb            # part 2 — the same agent on Microsoft Agent Framework
    microsoft_instructor.ipynb         # part 2 complete, plus teaching notes
    crewai_student.ipynb               # part 3 — the same agent again, on CrewAI
    crewai_instructor.ipynb            # part 3 complete, plus a multi-agent extension
    langgraph_student.ipynb            # part 4 — the loop rebuilt as an explicit graph
    langgraph_instructor.ipynb         # part 4 complete, plus teaching notes
    smolagents_student.ipynb           # part 5 — the outlier, where the action is Python
    smolagents_instructor.ipynb        # part 5 complete, plus teaching notes
```

**Part 1 first, always.** The value of everything after it is recognising that
`agent.run()`, `crew.kickoff()` and the rest do what you wrote by hand an hour earlier.
Taught first, a framework produces exactly the mental model this workshop exists to
prevent — that the library is doing something clever on the model's behalf.

After that, parts 2 to 5 stand alone and can be run in any order or in any subset. What
each one adds:

| | what it varies |
|---|---|
| 2 · Agent Framework | the loop disappears into one object; observability becomes middleware |
| 3 · CrewAI | agents get personas, tasks get rubrics, orchestration gets a crew |
| 4 · LangGraph | the loop comes back as a graph you can draw, branch and checkpoint |
| 5 · smolagents | the action stops being JSON and becomes Python |

**If you only have time for three notebooks, run 1 → 2 → 5.** That sequence gives the
sharpest contrast: build the mechanism, watch a framework hide it, then meet a framework
that changes what the mechanism is. Parts 3 and 4 are the ones to add when you have a
full day rather than a session.

The Python files the agent works on (`buggy.py`, `test_buggy.py`) are not in the repo.
Every notebook writes them itself with `%%writefile`, which also gives you a reset
button: re-run that cell to restore the bug and try the agent again from scratch.

## The shape of the session

1. One plain API call, no tools. Prove the connection works.
2. Meet the three tools: `read_file`, `edit_file`, `run_tests`. Each is a few lines
   of ordinary Python plus a JSON description.
3. **Write the loop.** This is the exercise.
4. Run it against a failing test suite and read the transcript.
5. Break it on purpose and find out what the loop was actually buying you.

Then, in the framework parts:

6. Rebuild the same agent with Microsoft Agent Framework, where the schemas are
   generated for you and the loop lives inside `run()`.
7. Recover the transcript with function middleware, and work out where your turn cap
   went.
8. Rebuild it on CrewAI, which wants an Agent, a Task and a Crew where Agent Framework
   wanted a single object.
9. Rebuild it on LangGraph as an explicit graph of nodes and edges — then have it draw
   itself, and compare the picture with the pseudocode from part 1.
10. Rebuild it once more on smolagents, where the agent writes Python instead of JSON,
    and read the code it actually wrote.
11. Compare all five side by side. Notice how little they agree on, and that the loop
    is one of the few things they do.

## The bug

`buggy.py` contains a mutable default argument. It reads as completely correct and
fails only on the *second* call, so it cannot be found by staring at it — the agent
has to run the tests and reason from the evidence.

That is the whole argument for the loop, demonstrated rather than asserted.

The same bug, the same three tools and the same task run through all five parts, so
the framework is the only variable across the notebooks.

## Cost

The notebooks default to `nvidia/nemotron-3.5-lightning:free` via OpenRouter: an
open mixture-of-experts model from NVIDIA, 3B active parameters out of 30B total,
built for tool calling and JSON-schema structured output. The `:free` endpoint
costs nothing; OpenRouter rate-limits it per account per day, which is enough for a
handful of runs.

If the rate limit gets in your way, drop the `:free` suffix for the paid endpoint:
$0.08 / $0.20 per million input / output tokens ($0.04/M on cache reads), a fraction
of a cent per run. Set a spend limit on any paid key.

Swapping models is one string in the setup cell:

```python
MODEL = "nvidia/nemotron-3.5-lightning:free"
```

Two notebooks ask more of the model than the others. CrewAI's ReAct-style prompting
uses noticeably more tokens per turn, and smolagents asks the model to write working
Python rather than emit a tool call. If either misbehaves, a stronger model is a
one-line change.

## Safety notes

`edit_file` writes to your disk and `run_tests` starts a subprocess. Both are
deliberately narrow, and the `DISPATCH` dictionary is the agent's entire set of
powers — if a function isn't in that dict, the model cannot invoke it. That dict is a
permission boundary you control, which is worth remembering the next time you are
tempted to add a `run_shell` tool.

Parts 2 to 4 rename that boundary but do not remove it: it becomes the `tools=[...]`
list you hand to the agent, or the `ToolNode` in the graph. Same rule, different
spelling.

Part 5 is the exception and deserves a closer read. smolagents executes model-written
Python, so the boundary has to become an interpreter with an import allowlist rather
than a list of functions. That is real protection and it is still a larger surface
than JSON tool calls — smolagents supports Docker and remote sandbox executors for
when the default is not enough.

Run this in a directory you don't mind it editing.
