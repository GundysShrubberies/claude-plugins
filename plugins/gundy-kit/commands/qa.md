---
description: Run whatever local QA this repo has (lint, typecheck, tests, build) and report PASS or FAIL in plain English.
argument-hint: ""
---

Run the repo's local QA — build, lint, typecheck, tests, whatever it actually
has — and report a plain PASS/FAIL. There is no CI here (no GitHub Actions in
this plugin's model): this command **is** the QA gate, run locally, every
time. `/work-issue` runs it before opening a PR; `kit-reviewer` runs it again
independently on the PR branch before approving.

## Arguments

`$ARGUMENTS` — none expected. `/qa` always runs everything this repo has.

## Steps

1. **Detect what this project has**, in this order, running everything that
   applies (a repo can match more than one):
   - **Node** (`package.json` present): for each of `lint`, `typecheck`,
     `test`, `build` that exists under `.scripts`, run `npm run <script>`
     (use `pnpm`/`yarn` instead of `npm` if `pnpm-lock.yaml`/`yarn.lock` is
     present). Skip a script name that isn't defined — don't invent one.
   - **Python** (`pyproject.toml` or `requirements.txt` present): if
     `pyproject.toml` declares a `pytest` config or a `tests/` directory
     exists, run `pytest`. If `ruff`/`flake8`/`mypy` config files exist, run
     those too (`ruff check .`, etc.) — only the ones actually configured.
   - **Make** (`Makefile` present): if a `test` target exists (`make -n test`
     doesn't error), run `make test`. Same for a `build`/`lint` target if
     present.
   - **Go** (`go.mod` present): `go build ./...` and, if any `_test.go` files
     exist, `go test ./...`.
   - Anything else recognizable (Cargo.toml → `cargo test`, etc.) — use
     judgment, but only run commands that are genuinely defined by the
     project, never a guess at what "should" exist.

2. **Nothing detected at all** — say so plainly ("this project doesn't
   define any lint/test/build commands I can find") and **require at least a
   build or smoke run** before calling anything green: try the most basic
   thing that proves the code isn't obviously broken (the language's
   compile/build step, or — for a script with no build step — actually
   running it once with a no-op/help invocation). Report that you did this
   and why, rather than silently passing with nothing checked.

3. **Secrets scan, if available** — best-effort, never required:
   ```bash
   command -v gitleaks >/dev/null 2>&1 && gitleaks detect --no-banner
   ```
   A finding here is always a FAIL (a secret got written to a tracked file),
   regardless of what else passed. A missing `gitleaks` binary is not a
   FAIL — just skip this check and say so in the report.

4. **Report — plain PASS/FAIL, then stop:**
   ```
   QA: PASS
   Ran: <one line per thing you ran — command + result>
   ```
   or
   ```
   QA: FAIL
   Failed: <which command, and the one most relevant line of its output>
   Ran: <everything else, and its result>
   ```
   Keep it short — this report is read by both a human and another agent
   (`kit-reviewer`), never a full log dump. If nothing at all could be
   checked (step 2's build/smoke run also isn't possible), report `QA: FAIL`
   and say plainly that the project has no way to verify itself yet.
