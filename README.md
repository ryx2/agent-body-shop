# Agent Body Shop

```
claude "You are the Agent Body Shop. Your job is to make the AI agent in this repo faster, cheaper, and more accurate. Ask me these questions one at a time: 1) Which files contain my agent's prompts and logic? 2) How do I run my evals? (give me the command) 3) What metric matters most — tokens, accuracy, or speed? Then enter a loop: run evals to get a baseline, find the single highest-impact optimization, make the change, re-run evals, show me before/after numbers, and ask to keep or revert. Repeat until I say stop."
```

Copy that. Paste it in your terminal from your agent's project directory. That's it.

---

## What this does

Most agent prompts are 2-3x longer than they need to be. Most agents make more tool calls than necessary. Most agent systems have never been systematically optimized.

Agent Body Shop fixes that. You point it at your agent's code and your evals, and it runs an improvement loop:

1. **Baseline** — runs your evals, measures tokens, accuracy, and latency
2. **Diagnose** — reads your agent's code and finds the highest-impact optimization
3. **Change** — makes one atomic change (prompt trim, tool consolidation, routing fix, etc.)
4. **Measure** — re-runs evals, shows you the before/after delta
5. **Decide** — you keep or revert
6. **Repeat**

Each pass through the loop should produce a measurable improvement. Common wins on the first pass:

- **30-50% token reduction** from prompt compression alone
- **Fewer tool calls** by combining related operations
- **Higher accuracy** from clearer, more specific instructions
- **Lower latency** from parallel tool calls and fewer round-trips

## What you need

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed (`npm i -g @anthropic-ai/claude-code`)
- Your agent's source code in a local directory
- An eval script, test suite, or webhook that returns pass/fail + metrics

## Eval format

Agent Body Shop works with whatever you have. Some examples:

**A shell script that returns a score:**
```bash
#!/bin/bash
# eval.sh — runs 10 test cases, prints pass rate and avg tokens
python run_tests.py --count 10 --format json | jq '{pass_rate: .passed/.total, avg_tokens: .avg_tokens}'
```

**A pytest suite:**
```bash
pytest tests/agent/ -v --tb=short
```

**A webhook that accepts a prompt and returns metrics:**
```bash
curl -X POST https://your-eval-endpoint.com/run \
  -H "Content-Type: application/json" \
  -d '{"agent_version": "current"}'
```

If you don't have evals yet, Agent Body Shop will help you write them first.

## What it optimizes

In priority order:

| Target | How |
|--------|-----|
| **Token usage** | Compress prompts, remove redundant instructions, tighten tool descriptions, reduce few-shot examples |
| **Accuracy** | Clarify ambiguous instructions, add guardrails for common failure modes, improve tool selection logic |
| **Speed** | Parallelize tool calls, reduce round-trips, remove unnecessary confirmation steps, optimize routing |

## Tips

- **Start with evals.** If you don't measure it, you can't improve it. Even 5-10 manual test cases is enough to start.
- **One change at a time.** The loop is intentionally atomic. Don't let it batch changes — you'll lose track of what helped.
- **Prompt compression is usually the biggest win.** Most agents ship with prompts written for human readability, not token efficiency.
- **Watch for accuracy regressions.** Token savings mean nothing if your agent breaks. Always check accuracy after compression.
- **Run it again after a week.** As you add features to your agent, bloat creeps back in.

## Inspired by

[HyperAgents](https://arxiv.org/abs/2603.19461) (Zhang et al., 2026) — the academic version of this idea. They showed that self-referential agents that modify their own improvement mechanism can achieve unbounded self-improvement on coding, paper review, and math tasks. Agent Body Shop is the practical, stripped-down version: you bring the agent, you bring the evals, and Claude Code does the optimization loop.

## License

MIT
