# Building an AI Agent from Scratch

A short workshop in which you write an AI agent — the whole thing, about fifteen
lines — and watch it find and fix a bug in a real file on your disk.

No frameworks. The point is to see the mechanism before anything hides it from you.

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
git clone <this repo>
cd agentic-debugging-workshop
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

Then open `notebooks/agent_workshop_student.ipynb` and select the `.venv` kernel.

The notebook asks for your API key with `getpass`, so it is never written to the
notebook file and never appears in your shell history. Do not paste your key into a
cell.

### Check your setup before the session

Run the first two cells. If a haiku prints, you are ready. If it doesn't, sort it out
before the workshop rather than during it.

## What's in here

```
notebooks/
    agent_workshop_student.ipynb       # the loop is left as TODOs — this is the one you work in
    agent_workshop_instructor.ipynb    # complete, plus teaching notes and variations
```

The Python files the agent works on (`buggy.py`, `test_buggy.py`) are not in the repo.
The notebook writes them itself with `%%writefile`, which also gives you a reset
button: re-run that cell to restore the bug and try the agent again from scratch.

## The shape of the session

1. One plain API call, no tools. Prove the connection works.
2. Meet the three tools: `read_file`, `edit_file`, `run_tests`. Each is a few lines
   of ordinary Python plus a JSON description.
3. **Write the loop.** This is the exercise.
4. Run it against a failing test suite and read the transcript.
5. Break it on purpose and find out what the loop was actually buying you.

## The bug

`buggy.py` contains a mutable default argument. It reads as completely correct and
fails only on the *second* call, so it cannot be found by staring at it — the agent
has to run the tests and reason from the evidence.

That is the whole argument for the loop, demonstrated rather than asserted.

## Cost

The default model is `nvidia/nemotron-3.5-lightning` via OpenRouter: a 30B
mixture-of-experts model with ~3B active parameters, trained for tool-calling
execution, at roughly $0.08 / $0.20 per million input / output tokens. A full agent
run costs a fraction of a cent.

Swapping models is one string in the setup cell:

```python
MODEL = "nvidia/nemotron-3.5-lightning"
```

## Safety notes

`edit_file` writes to your disk and `run_tests` starts a subprocess. Both are
deliberately narrow, and the `DISPATCH` dictionary is the agent's entire set of
powers — if a function isn't in that dict, the model cannot invoke it. That dict is a
permission boundary you control, which is worth remembering the next time you are
tempted to add a `run_shell` tool.

Run this in a directory you don't mind it editing.
