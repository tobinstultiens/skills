---
name: build-analyzer
description: "DIAGNOSTIC-ONLY Use when a user asks anything about a build's status, failures, or performance, e.g. \"why does the build fail\", \"why is the build so slow\", \"what broke the build\", \"is CI green\", \"which step is taking so long\", or any question about compile errors, failing pipelines, or build timings. Covers investigating builds starting from local output, then the Bamboo CI server (atlassian-bamboo-mcp), and proposing (never silently running) deeper actions like checking out a commit via atlassian-bitbucket-mcp or rebuilding locally. The agent must inspect read-only evidence first, report the diagnosis, and stop before any workspace edits, local builds, test runs, checkouts, remote build triggers, or fixes unless the user explicitly approves that exact next action."
---

# Build Analyzer

Diagnose build failures and slowness. The guiding principle is **investigate cheaply first, escalate only as needed, and never mutate the workspace or trigger builds without explicit user approval.**

## Mandatory diagnosis checkpoint

This skill is diagnostic-only by default.

When the user asks about a build failure/status/performance, the agent must:
1. Use read-only evidence to identify the first/root cause.
2. Report the diagnosis, evidence source, and proposed next action.
3. Stop.

The agent must not edit files, run local builds/tests/lints/formatters, check out commits, trigger remote builds, or apply fixes unless the user explicitly approves that exact action after the diagnosis is reported.

Phrases like "can you help?", "why did it fail?", "what broke?", or "my build failed" are not approval to fix, edit, build, test, lint, format, or validate locally.

## Escalation ladder

Work through these tiers in order. Stop as soon as you can answer the question — don't climb further than needed.

### Tier 1 — Local build output (always start here)

Look at what's already on disk before touching any remote service. No user permission needed; this is read-only.

- Read the terminal/build panel output if it's in context.
- Otherwise inspect local artifacts: log files, `build/`, `dist`, `target/`, `out/`, `bin/obj`, `*.binlog`, MSBuild/`dotnet build` output, test reports, and any `*.log` in the project root.
- For **failures**: find the first real error (compiler error, failed test, missing dependency, non-zero exit). Later errors are often cascades — trace to the root cause.
- For **slowness**: look for per-step or per-target timings, repeated work, cache misses, restore/download phases, or a single dominating step.

If the local output fully answers the question, report the diagnosis and stop.

### Tier 2 — Bamboo CI (`atlassian-bamboo-mcp`)

If local output is missing, inconclusive, or the question is about CI specifically ("is the build green", "why did the pipeline fail"), check Bamboo.

**Figuring out which build to check** — do not ask the user for the plan key if you can derive it:
1. Determine the **repository**. Read the git remote (`git remote -v`) / the linked repo in the current workspace.
2. Determine the **branch**. If the user named one, use it. Otherwise use the currently checked-out branch (`git rev-parse --abbrev-ref HEAD`).
3. If the local repository has a `bamboo-specs/` folder, inspect it first for plan definitions, project keys, plan keys, branch settings, linked repository names, or specs-generated naming conventions that identify the relevant Bamboo plan.
4. Find or confirm the Bamboo plan whose **linked repository** matches, then the **plan branch** matching the git branch. Use any plan key or naming clues from `bamboo-specs/` where available, and use the Bamboo MCP tools to list/search plans and their branches, then pull the latest build result for that plan branch.
5. Only ask the user if the repo→plan mapping is genuinely ambiguous (multiple plans on the same repo, no matching plan branch, or local specs pointing to more than one plausible plan).

Once the right build result / plan branch is identified:
- **Failures**: pull the build result and its logs; find the first real failure the same way as Tier 1.
- **Slowness**: compare stage/job/task durations, look at trends across recent builds, and identify the dominating stage or a regression versus prior builds.

If Bamboo gives you enough, report and stop.

### Tier 3 — Propose deeper actions (ASK FIRST — never execute automatically)

If Tiers 1–2 still don't answer it, do **not** change the workspace or kick off builds on your own. Instead, present the user with concrete next-step options and wait for their choice. Options typically include:

- **Check out a specific commit** (via `atlassian-bitbucket-mcp`) — e.g. to reproduce a failure that only appears at the commit Bamboo built, or to bisect a regression. Name the commit/branch you'd check out and why.
- **Build the local project** — run the build locally to reproduce and get fresh output. State the exact command you'd run.
- **Both** — check out the relevant commit, then build it locally.

Frame each option with what it would tell you, so the user can pick. Use the questioning/elicitation tool to offer these as selectable options where possible. **Wait for explicit approval before running any checkout, build, or other state-changing command.**

## Hard rules

- Read-only investigation (Tier 1, and read-only Bamboo/Bitbucket lookups in Tier 2) needs no permission — just do it.
- **Never** run `git checkout`/`switch`, modify files, or start a local or remote build without the user first agreeing to that specific action. Proposing is fine; executing is not.
- Always trace to the **first / root** cause rather than reporting the last error line.
- When you give a diagnosis, say which tier the evidence came from (local output vs. Bamboo) so the user knows how authoritative it is.

## Reporting format

Keep it tight:

1. **Answer** — the direct diagnosis (why it fails / why it's slow).
2. **Evidence** — the specific error, failing step, or timing, and where it came from.
3. **Next step** — a fix if the cause is clear, or the proposed Tier-3 options if more investigation is needed (asked, not executed).
