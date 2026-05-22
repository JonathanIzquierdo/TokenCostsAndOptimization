How to Accelerate Your Development with the Right Claude Code Setup
You've probably tried AI coding tools. Maybe you installed Copilot or Claude Code, used it for a bit, got some good suggestions, and moved on. It helps, nobody's arguing that. But after a while it still feels like you're doing most of the work yourself. The AI completes your lines, maybe generates a function, but the core workflow (the debugging, the context-switching, the testing, the deployments) is still on you.

If this sounds familiar: you're copying stack traces into a chat, re-explaining your project architecture for the third time, manually running tests after every change, switching between your ticket system and your editor dozens of times a day. That's not really a tool problem. That's a setup problem. The AI is there, it's just not in the loop.

The difference between AI-as-autocomplete and AI-as-teammate comes down to three things: giving it your project context so it understands the whole picture, setting up feedback loops so it self-corrects instead of you reviewing everything manually, and connecting it to the tools you actually work with instead of keeping it isolated in a chat window.

I'm Otto. I recently joined the AI Engineering & Technology team as AI Tech Lead. Before Visma, I spent the last few years building AI-powered products at startups. I've been coding exclusively with AI tools since early 2024, starting with GitHub Copilot, then Cursor where I pretty much stopped writing code manually, and Claude Code since it launched in February 2025. I've built web apps with Next.js, React, and Node, mobile apps with Expo and Flutter, a few games. After almost a year of daily use, this is the setup that stuck. It works the same for both greenfield and brownfield, just with slightly different starting points.







Give the AI your project context — CLAUDE.md
Claude Code reads a CLAUDE.md file at the root of your repo when a session starts. I think of it as onboarding notes — what would you tell a good developer on their first day before they start touching code?

Things like build/test/lint commands, how the architecture is laid out, coding conventions, behavioral rules ("always write tests", "don't touch the auth module without asking"). Keep it focused though — skip formatting rules (your linter handles those), skip stuff Claude already does correctly, and try to stay under ~80 lines.

You don't have to write it from scratch either. Just ask Claude Code to explore the repo and draft a first version. It can figure out what's worth documenting — the architecture, the commands, the conventions already in use. You review it, tweak what doesn't fit, done.

Check it into git so the team can iterate on it. It gets better over time.

For larger projects: You can also use nested CLAUDE.md files in subdirectories. Your root CLAUDE.md covers the project-wide stuff, and then /api/CLAUDE.md has API-specific conventions, /packages/ui/CLAUDE.md has component rules, and so on. Claude loads the relevant ones based on where it's working. Useful for monorepos or projects with distinct areas that have their own patterns.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AccountingAI is an intelligent accounting platform that automates invoice processing, reconciliation, and financial reporting for Nordic SMEs. Users upload invoices (PDF/e-invoice), the system extracts structured data via OCR + LLM, matches against bank transactions, and generates compliant financial reports.

## Monorepo Structure

pnpm workspace monorepo with three packages:

- apps/web/ — Next.js App Router frontend + API routes
- apps/worker/ — Bull queue workers for async invoice processing
- packages/shared/ — Shared types, validation schemas, constants

Key directories within apps/web/src/:
- app/(dashboard)/ — Authenticated pages (invoices, reports, settings)
- app/api/ — REST API routes
- lib/ — Core business logic (extraction, matching, reporting)
- lib/ai/ — LLM integration (extraction prompts, classification)
- types/ — Domain types + auto-generated DB types

## Commands

# Development (from repo root)
pnpm dev                    # Start all apps concurrently
pnpm build                  # Production build
pnpm lint                   # ESLint across all packages
pnpm typecheck              # tsc --noEmit

# Testing (from apps/web/)
pnpm test                   # Unit + integration (Vitest)
pnpm test:unit              # Unit tests with coverage
pnpm test -- src/tests/unit/foo.test.ts  # Single test file
pnpm test:e2e               # Playwright headless

# Database (from apps/web/)
pnpm db:migrate             # Apply pending migrations
pnpm db:reset               # Reset + seed local database
pnpm db:types               # Regenerate TypeScript types from schema
## Architecture

Invoice processing pipeline (`lib/extraction/pipeline.ts` → `processInvoice()`):
1. File intake: PDF parsing or e-invoice (Finvoice XML) deserialization
2. Data extraction: LLM extracts vendor, line items, amounts, dates
3. Validation: Cross-checks totals, VAT calculations, schema conformance
4. Matching: Finds corresponding bank transactions via amount + date heuristics
5. Storage: Persists invoice, line items, match candidates to Postgres

Reporting engine (`lib/reporting/`):
- Generates periodic financial summaries from matched transactions
- Outputs in Finnish SIE format and PDF

Auth: Supabase Auth with org-based multi-tenancy. Row-level security on all tables.

Path alias: @/* maps to apps/web/src/*

## Key Conventions

- TypeScript strict mode. Biome formatter (2-space indent, double quotes).
- Environment config centralized in lib/env.ts with runtime validation.
- Coverage thresholds: 90% lines. Tests fail below this.

## Gotchas

- IMPORTANT: All monetary values stored as integers (cents). Never use floats for money.
- Database types are auto-generated — edit migrations, not database.generated.ts.
- Finnish locale assumed for date/number formatting unless explicitly overridden.
- Worker jobs are idempotent — re-processing the same invoice must not create duplicates.

## Git Workflow

- Branches: feature/description, fix/description
- Commit messages: conventional commits (`feat:`, fix:, `chore:`)
- PRs require passing CI (typecheck + lint + tests) before merge

## Hooks

Post-edit: Biome auto-fix runs after every Edit/Write on ts/tsx/js/jsx/json files.
Stop: Knip checks for unused exports when conversation ends.

## Further Reading

@docs/architecture.md
@docs/local-setup.md





Let the AI see your errors
Run your dev build inside a Claude Code terminal. Your npm run dev, Docker compose, whatever — just run it where Claude can see the output.

It sounds simple, but it changes a lot. When something breaks, Claude reads the error log directly instead of you having to copy-paste stack traces or explain what went wrong. It sees the error, it already has the codebase context, and it starts working on a fix. That whole loop of copying errors back and forth mostly goes away.

Same thing applies to tests. If you run your test suite in a Claude Code terminal, it sees exactly what failed and why, and can go fix it without you having to relay anything. Write code, tests fail, Claude reads the output, fixes, tests pass.

As an example — I once wanted to get a Flutter game running on a laptop that had nothing installed. No Flutter, no Xcode, no simulators. I told Claude Code to figure out what was needed and not to stop until the game was running, and then went to the gym. When I came back an hour later it was done. Claude had been at it for 53 minutes, installing packages, launching the simulator, hitting errors, fixing them, trying again — something like 15 attempts until everything worked. That only happened because it could see every error as it came up. Without that visibility it would've been stuck after the first failure.





Automate quality feedback with hooks
Without some kind of guardrail, AI-generated code tends to drift. Duplicated functions, forgotten conventions, code that works but isn't clean. Hooks help with this.

They're shell commands that run automatically at certain points in the workflow. I use two hook types:

My PostToolUse hook runs after every file edit and handles linting and formatting through Biome with the Ultracite preset (like ESLint, but way faster, it's written in Rust). It auto-fixes what it can and blocks on remaining issues. Claude gets that feedback right away, corrects itself, and the code comes out clean on the first pass.

My Stop hook runs once when Claude finishes responding and does two heavier checks: TypeScript compiler for full type checking and Knip for catching unused exports, dependencies, and files. These are too slow to run after every edit, but running them at the end of each response catches issues before they pile up.

Before I had this set up, I'd spend time going back and forth on linting issues and small quality things. Now that mostly takes care of itself.

# .claude/settings.json

{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/post-edit-lint.sh",
            "timeout": 30
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/stop-knip.sh",
            "timeout": 30
          }
        ]
      }
    ]
  }
}

#!/bin/bash
# Hook: post-edit-lint.sh
# Type: PostToolUse (Edit, Write)
# Purpose: Auto-fix and lint edited files with Biome (using Ultracite preset)

FILE=$(cat | jq -r '.tool_input.file_path // empty')

[[ -z "$FILE" || ! -f "$FILE" ]] && exit 0
[[ "$FILE" =~ \.(ts|tsx|js|jsx|json|css)$ ]] || exit 0

# Auto-fix with Biome (biome.json extends ultracite/biome/core + next)
npx biome check --write "$FILE" >/dev/null 2>&1

# Check for remaining issues
LINT_OUT=$(npx biome check "$FILE" 2>&1)
if [[ $? -ne 0 ]]; then
  echo "$LINT_OUT" >&2
  exit 2
fi

exit 0

#!/bin/bash
# Hook: stop-knip.sh
# Type: Stop
# Purpose: Run tsc and Knip when Claude finishes responding

PROJECT_DIR="${CLAUDE_PROJECT_DIR:-$(git rev-parse --show-toplevel 2>/dev/null || pwd)}"
ERRORS=""

# Type check
TSC_OUT=$(cd "$PROJECT_DIR/apps/web" && npx tsc --noEmit --incremental --tsBuildInfoFile /tmp/project-tsbuildinfo 2>&1)
if [[ $? -ne 0 ]]; then
  ERRORS+="$TSC_OUT"$'\n'
fi

# Unused exports/deps (run from project root so knip.json is found)
KNIP_OUT=$(cd "$PROJECT_DIR" && npx knip --no-progress --cache 2>&1)
if [[ $? -ne 0 ]]; then
  ERRORS+="$KNIP_OUT"$'\n'
fi

if [[ -n "$ERRORS" ]]; then
  echo "$ERRORS" >&2
  exit 2
fi

exit 0

This works with whatever linter your project uses. ESLint, Prettier, Ruff, doesn't matter. And you can ask Claude Code to set up the hook config for you too, just tell it what linter you're using and what kind of workflow you want. Hooks can do a lot more than linting though, they're one of the most powerful and underutilized features in Claude Code. Worth exploring the hooks guide to see what's possible.





Work in multiple sessions
I usually have several Claude Code sessions running at the same time, each with a different job. One runs the dev build so it can monitor errors, one handles whatever feature I'm working on, one writes tests, one does code review and refactoring, and then I'll have a few more for ticket work or whatever comes up.

The reason for splitting things up is that a single session starts losing quality when it's trying to do too many things at once. The context fills up and it loses track of what it was doing. Separate sessions keep each task focused, and they run in parallel naturally — while one is writing tests, another is building a feature.

One thing I do pretty regularly is dedicate a session to just proactive refactoring. A couple times during the day I'll give it a prompt like "look through this module, find anything worth improving." It's the kind of thing that's easy to skip when you're focused on features, but having a separate session for it means it actually happens.

You don't need to start with five sessions though. Even two makes a difference — one running your app, one doing work. You can scale up from there once you get the feel for it.





Connect to your tools with MCPs
MCPs (Model Context Protocol) give Claude Code direct access to external tools — your ticket system, database, browser, whatever you connect.

The most useful one for me has been connecting to my project management tool. Most of the common ones have MCP servers available already, Jira, Linear, Asana, and others. You reference a ticket, Claude fetches the description, implements the fix, writes tests, opens a PR. No copy-pasting ticket details back and forth.

Another big one is browser control. Claude Code can connect to your browser through the Claude in Chrome extension or through Playwright. I use this a lot for frontend work and testing. Claude can navigate to your localhost, take screenshots, record video, interact with elements on the page, all from within your terminal session. So if you're building a feature and want to verify it looks right, or you need to do some visual regression testing, Claude can just go look at it itself instead of you describing what's on the screen. I've found the Claude in Chrome extension to be more accurate and easier for Claude to control compared to Playwright, but Playwright is better if you want a more integrated end-to-end testing setup that lives in your codebase. Either way, it makes testing a lot more hands-off.





Encode your workflows with skills
Skills are reusable instruction sets — markdown files that load on demand when you trigger them. They're where you encode specific workflows, processes, or multi-step tasks that you want done a certain way.

I build them reactively. When I notice I'm giving Claude the same instructions over and over, that's the signal to extract it into a skill. A couple of concrete examples:

Versioning and changelog: Checks what changed, updates changelog.md, decides if it's a minor or major bump, updates the version, pushes or opens a PR. Automatic version tracking without thinking about it.

TestFlight deploy: Building and deploying a Flutter app to TestFlight has a lot of moving parts — signing profiles, provisioning, Apple config. Before the skill, Claude got through the full process maybe 30% of the time. After putting the exact steps into a skill, it hasn't failed once. Five minutes, I just say "push to TestFlight."

Both came from the same thing — I was doing something manually, it was error-prone, so I made a skill for it. I wouldn't bother building skills upfront. Wait until you feel the friction, then extract. Claude Code can write the skill file for you too.

Skills are also a good place to put workflow definitions that you might otherwise put in your CLAUDE.md — things like PR conventions, deployment steps, or ticket workflows. The difference is that skills only load when you need them, so they don't take up context space all the time.





Use speech-to-text for prompting
When you're prompting multiple sessions all day, typing becomes the bottleneck. Speech-to-text helped a lot with that.

Speed is part of it — you talk about 3-4x faster than you type. But the bigger thing is prompt quality. When you're speaking you naturally explain things more — "so this function handles the case where a user hasn't verified their email yet, and right now it just crashes, can you add a proper check and maybe redirect to the verification page?" You just give more context when talking than when typing, and that makes for better results.

I use Superwhisper. Most tools in this space run on Whisper 3 and they're all fine for engineering work — you can run it locally too. No clear winner, just pick one.





Parallelization — sub-agents and agent teams
When tasks don't depend on each other, you can run them in parallel. Two ways to do this:

Sub-agents — one session spawns focused workers that each handle a task and report back. Lightweight, built-in, good for things like writing tests for different modules or updating several endpoints. The parent session coordinates, sub-agents don't talk to each other.

Agent teams — multiple independent sessions that actually coordinate. Shared task list, messaging between agents, autonomous work pickup. One leads, the others grab tasks and work through them. More like how a real team works.

Trade-off is cost. Sub-agents are cheap. Agent teams use more resources since each member is a full session with its own context. Worth it for complex coordinated work, overkill for routine stuff.

Running separate terminals on different tasks like I described earlier is already parallelization too, just manual.





Greenfield vs. brownfield
Everything above works for both new and existing projects. The difference is mostly how you start.

New projects — set up CLAUDE.md from day one. Define conventions early, because the AI will follow whatever patterns you set.

Existing projects — have Claude Code explore the repo and draft a CLAUDE.md for you. It's also good for getting up to speed on unfamiliar code. Ask it to explain flows, trace dependencies, walk you through how things connect. Start small and scale up.





Where to start
If you want to try any of this:

CLAUDE.md — ask Claude Code to explore your repo and draft one. Review, adjust, commit. Claude Code also has a built-in /init command you can run to do this.
Dev server in Claude Code — run it where Claude can see your errors.
Post-edit hook — connect your linter so Claude gets feedback on every edit.
Give it a week and see what sticks.
