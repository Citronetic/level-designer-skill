# level-designer — a Claude Code skill for the Citronetic level editor

A [Claude Code](https://docs.claude.com/en/docs/agents-and-tools/claude-code/overview) skill that
lets Claude design, mutate, and validate levels for the
[Citronetic/level-editor](https://github.com/Citronetic/level-editor)
puzzle authoring tool.

Levels are not free-form sketches — they are small mathematical
puzzles with strict invariants (wall continuity, color throughput,
exit reachability, slide-graph solvability). This skill encodes the
invariants so a remix either preserves them by construction OR is
flagged as broken before the file is written.

## What it does

Given an existing level (or a natural-language brief), the skill:

1. Picks a base level from `data/levels/*.json` whose structure best
   matches the user's intent (size, block count, color palette).
2. Applies one or more **safe, named mutations** — recolor, mirror,
   rotate, translate, add ice, add curtain — composing them step by
   step.
3. **Validates** the result against every hard rule (G1..G6 +
   CB1..CB4 + element-limit ranges) using [`scripts/validate.py`](scripts/validate.py).
4. **Checks solvability** via a slide-graph BFS using
   [`scripts/solvability.py`](scripts/solvability.py).
5. Writes the new file to `data/levels/custom-<name>.json` only after
   both pass.

## Install into a level-editor checkout

```bash
cd path/to/level-editor
mkdir -p .claude/skills
git clone https://github.com/Citronetic/level-designer-skill.git \
  .claude/skills/level-designer
```

Then from inside a Claude Code session in that directory:

> /level-designer remix t76-level-14 with all warm colors and a 2-layer
> ice door on the main exit

## Repository layout

```
SKILL.md                       Main entry point — Claude reads this first.
                               Contains the rule taxonomy, mutation
                               playbook, and difficulty heuristics.

scripts/
  validate.py                  All hard rules in one script.
                               Exit 0 = valid level.
  mutate.py                    Apply a named, rule-preserving mutation:
                               recolor / mirror / rotate / translate /
                               add_ice / add_curtain. Chainable.
  solvability.py               Per-block slide-graph BFS to a matching
                               door. Necessary-but-not-sufficient check
                               (treats blocks in isolation).

references/
  solvability.md               Deep dive on the slide model + BFS
                               algorithm + curtain/ice door handling.
  mutations-cookbook.md        Before/after code snippets for every
                               mutation + a safety matrix.
  worked-examples.md           Three end-to-end remix sessions: a
                               recolor+mirror variant, a 7-block hard
                               puzzle from a similar base, and a
                               from-scratch tutorial.
```

## Standalone CLI usage (no Claude needed)

The scripts work as standard CLI tools when you want to mutate or
validate a level by hand:

```bash
python3 scripts/validate.py data/levels/t76-level-32.json
python3 scripts/solvability.py data/levels/t76-level-32.json

# Recolor BCT 0 (Red) → BCT 3 (Green), preserving block/door pairing
python3 scripts/mutate.py \
  --in data/levels/t76-level-32.json \
  --out data/levels/custom-t76-32-green.json \
  --mutation recolor --params 0,3

# Validate the result
python3 scripts/validate.py data/levels/custom-t76-32-green.json
```

See [references/mutations-cookbook.md](references/mutations-cookbook.md)
for every mutation with worked snippets, and
[references/worked-examples.md](references/worked-examples.md) for
chained end-to-end recipes.

## License

MIT — same as the parent level-editor project.
