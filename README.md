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

## Quick start (5 minutes from zero)

You need [Claude Code](https://docs.claude.com/en/docs/agents-and-tools/claude-code/overview)
installed and Python 3 on `PATH`.

### 1. Get the level editor

```bash
git clone https://github.com/Citronetic/level-editor.git
cd level-editor
```

The editor is plain HTML/JS — no build step. Serve it locally:

```bash
./serve.sh 8081           # python3 -m http.server 8081 under the hood
# open http://localhost:8081 — you should see the editor with 2670+ levels
```

(Port 8080 is intentionally avoided — it collides with local Java tools
on some setups. Pick another free port if 8081 is taken.)

### 2. Install this skill into the checkout

From the `level-editor` directory you just `cd`'d into:

```bash
mkdir -p .claude/skills
git clone https://github.com/Citronetic/level-designer-skill.git \
  .claude/skills/level-designer
```

After this, the structure looks like:

```
level-editor/
├── index.html
├── js/                                # editor source (untouched)
├── data/levels/                       # 2670+ levels Claude will read & remix
├── .claude/
│   └── skills/
│       └── level-designer/            # ← this repo, cloned in
│           ├── SKILL.md
│           ├── scripts/
│           └── references/
```

### 3. Use the skill

Open Claude Code in the `level-editor` directory:

```bash
claude
```

Then invoke the skill with a natural-language brief:

> /level-designer please give me a hard variant of t76-level-14 with all
> warm colors and a 2-layer ice door on the main exit

Claude will:

1. Pick the base level from `data/levels/*.json`.
2. Apply named mutations (recolor / mirror / rotate / add ice / etc.).
3. Run [`scripts/validate.py`](scripts/validate.py) — hard rules.
4. Run [`scripts/solvability.py`](scripts/solvability.py) — slide-graph BFS.
5. Write `data/levels/custom-<your-name>.json` only if everything passes.
6. Report every mutation it applied + the validation summary.

To load the new level in the editor, refresh the page; the sidebar
auto-discovers anything in `data/level-manifest.json`. The skill adds
the new entry to the manifest as part of step 5.

## What's in this repo

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

The scripts work as plain Python tools when you want to mutate or
validate a level by hand. Run them from the level-editor root:

```bash
S=.claude/skills/level-designer/scripts

# Check that a level satisfies every hard rule
python3 $S/validate.py data/levels/t76-level-32.json

# Per-block reachability via slide-graph BFS
python3 $S/solvability.py data/levels/t76-level-32.json

# Recolor BCT 0 (Red) ↔ BCT 3 (Green), preserving block/door pairing
python3 $S/mutate.py \
  --in data/levels/t76-level-32.json \
  --out data/levels/custom-t76-32-green.json \
  --mutation recolor --params 0,3

# Validate the result
python3 $S/validate.py data/levels/custom-t76-32-green.json
```

See [references/mutations-cookbook.md](references/mutations-cookbook.md)
for every mutation with worked snippets, and
[references/worked-examples.md](references/worked-examples.md) for
chained end-to-end recipes.

## Updating

When the upstream skill changes, pull from inside the level-editor
directory:

```bash
git -C .claude/skills/level-designer pull
```

## License

MIT — same as the parent level-editor project.
