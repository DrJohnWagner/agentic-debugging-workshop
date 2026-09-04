# Building an AI Agent from Scratch

A short workshop in which you write an AI agent — the whole thing, about fifteen
lines — and watch it find and fix a bug in a real file on your disk.

Part 1 uses no frameworks at all. The point is to see the mechanism before anything
hides it from you. Parts 2 and 3 then rebuild the identical agent on Microsoft Agent
Framework and on CrewAI, so you can watch your loop collapse into a single method
call — twice — and see how thoroughly two frameworks disagree about everything
except the loop itself.

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
```

**Part 1 first, always.** The value of parts 2 and 3 is recognising that `agent.run()`
and `crew.kickoff()` do what you wrote by hand an hour earlier. Taught first, a
framework produces exactly the mental model this workshop exists to prevent — that the
library is doing something clever on the model's behalf.

Parts 2 and 3 are interchangeable in order, but do at least two of them. One framework
looks like *the* way to build agents; two frameworks disagreeing about what an agent
even is makes the point on their own.

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

Then in part 2:

6. Rebuild the same agent with Microsoft Agent Framework, where the schemas are
   generated for you and the loop lives inside `run()`.
7. Recover the transcript with function middleware, and work out where your turn cap
   went.

And in part 3:

8. Rebuild it once more on CrewAI, which wants an Agent, a Task and a Crew where
   Agent Framework wanted a single object.
9. Compare all three side by side, and notice that the only row they agree on is the
   loop.

## The bug

`buggy.py` contains a mutable default argument. It reads as completely correct and
fails only on the *second* call, so it cannot be found by staring at it — the agent
has to run the tests and reason from the evidence.

That is the whole argument for the loop, demonstrated rather than asserted.

The same bug, the same three tools and the same task run through all three parts, so
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

CrewAI's ReAct-style prompting uses noticeably more tokens per turn than the other two
notebooks, so part 3 is the one most likely to reach the free tier's daily limit.

## Safety notes

`edit_file` writes to your disk and `run_tests` starts a subprocess. Both are
deliberately narrow, and the `DISPATCH` dictionary is the agent's entire set of
powers — if a function isn't in that dict, the model cannot invoke it. That dict is a
permission boundary you control, which is worth remembering the next time you are
tempted to add a `run_shell` tool.

Parts 2 and 3 rename that boundary but do not remove it: it becomes the `tools=[...]`
list you hand to the agent. Same rule, different spelling.

Run this in a directory you don't mind it editing.
