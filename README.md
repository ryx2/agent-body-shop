# Agent Body Shop

```
claude "You are the Agent Body Shop — an agent that improves other agents. Ask me these questions one at a time: 1) Which files contain my agent's prompts and logic? 2) How do I run my evals? (give me the command) 3) What metric matters most — accuracy, cost, or speed? Then enter an improvement loop. Each iteration: run evals for a baseline, analyze the results to find failure patterns, pick ONE improvement from this list (replace vague attitude instructions with structured processes and explicit checklists; add domain-specific knowledge and concrete examples to the prompt; build decision trees with clear criteria instead of open-ended instructions; detect and fix output bias or degenerate behavior; track what works across iterations and double down on it), make the change, re-run evals, show me before/after numbers, and ask to keep or revert. Repeat until I say stop."
```

Copy that line. Run it from your agent's project directory. It asks you three questions, then starts improving.

---

## What this does

You point it at your agent and your evals. It runs an improvement loop:

1. **Eval** — runs your evals, gets a baseline score
2. **Analyze** — looks at *what failed and why*, not just the number
3. **Change** — makes one targeted change based on the failure analysis
4. **Re-eval** — runs evals again, shows you the before/after delta
5. **Decide** — you keep or revert
6. **Repeat** — each pass uses what it learned from previous passes

## What it actually changes

These are the optimization strategies that work, based on [HyperAgents](https://arxiv.org/abs/2603.19461) (Zhang et al., 2026) — a paper that ran self-improving agents across coding, paper review, robotics, and math grading. They found that the same categories of improvement show up regardless of domain:

### 1. Replace attitude instructions with structured processes

The single biggest win. Telling an agent to "be rigorous and critical" doesn't work. Replacing that with a two-stage process (first identify problems, then make a decision) does. Vague behavioral prompts produce unreliable results — explicit step-by-step procedures produce consistent ones.

**Before:** `"You are a rigorous and critical reviewer. Be thorough."`

**After:** `"Stage 1: List all weaknesses. Stage 2: Check each against these rejection criteria. Stage 3: Make your decision based on the checklist."`

### 2. Add domain knowledge and concrete examples

Agents fail when they're guessing about the domain. Grounding them with real constraints, valid parameters, and concrete examples eliminates hallucination-driven failures. Rather than isolated prompt edits, build up a shared knowledge base in the prompt.

**Before:** `"Design a reward function for this robotics task."`

**After:** `"Here are the valid environment attributes: [list]. Use ONLY these. Here is the reward structure: primary tracking (0.5–1.0 scale), penalties (−0.01 to −0.001). Example: [concrete code]."`

### 3. Build decision trees with explicit criteria

Open-ended instructions let the agent make up its own criteria each time. Explicit decision trees with calibrated thresholds produce consistent results. Replace "use your judgment" with "if X then Y."

**Before:** `"Decide if this is good enough."`

**After:** `"Apply this decision tree: (1) Does it meet ALL acceptance criteria? → accept. (2) Does it fail ANY rejection criteria? → reject. (3) Borderline → default to reject."`

### 4. Detect and fix degenerate behavior

Agents drift. A paper review agent might start accepting everything. A classifier might collapse to one label. Track output distributions and add correction when bias is detected — this is something the agent can do automatically by analyzing its own eval results.

### 5. Track what works across iterations

Don't treat each improvement as independent. Keep a log of what you tried, what worked, and what regressed. Use insights from failed attempts to inform the next change. The best improvements are often combinations of ideas from multiple earlier attempts.

### 6. Improve the improvement process itself

The most powerful finding from the paper: the agent doesn't just get better at its task — it gets better at *improving itself*. It learns to analyze evals before making changes, allocate effort based on remaining budget, and build reusable infrastructure (prompt templates, performance trackers, memory systems) that make future improvements easier.

## What you need

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed (`npm i -g @anthropic-ai/claude-code`)
- Your agent's source code in a local directory
- An eval script, test suite, or webhook that returns pass/fail + metrics

If you don't have evals yet, the Body Shop will help you write them first.

## How evals should work

Anything that returns a score. Some examples:

```bash
# A script that returns pass rate
python run_tests.py --format json | jq '{pass_rate: .passed/.total}'

# A pytest suite
pytest tests/agent/ -v --tb=short

# A webhook
curl -X POST https://your-eval-endpoint.com/run -d '{"agent_version": "current"}'
```

## Tips

- **Start with evals.** If you don't measure it, you can't improve it. Even 5 manual test cases is enough to start.
- **One change at a time.** The loop is intentionally atomic — you need to know what helped and what didn't.
- **Analyze failures, not just scores.** The score tells you *how much* is broken. The failure cases tell you *what* to fix.
- **Watch for regressions.** Improving one metric can break another. Always check the full eval suite.
- **Run it again after a week.** As you add features, bloat and drift creep back in.

## Examples

See **[EXAMPLES.md](EXAMPLES.md)** for before/after optimizations of 15 real open-source agents, including OpenAI Swarm, Aider, SWE-agent, OpenHands, and MetaGPT.

**Live demo result:** We ran the Body Shop on a support ticket classifier with a vague prompt ("You are a helpful customer support assistant. Be very accurate and careful."). One pass — replacing the attitude instruction with explicit category definitions and a decision tree — took accuracy from **85% to 100%** and reduced latency by 25%.

## Background

Based on [HyperAgents](https://arxiv.org/abs/2603.19461) (Zhang et al., 2026). They showed that agents which can modify their own code and improvement process achieve sustained self-improvement across coding (Polyglot), paper review, robotics reward design, and Olympiad math grading — outperforming both static baselines and systems with fixed improvement mechanisms. The key insight: the improvement mechanism itself should be editable, not just the task-solving logic. Agent Body Shop is the stripped-down, practical version of that loop.

## License

MIT
