# START HERE — two-session setup (verified 2026-10-07)

This bundle sets up two Claude Code sessions that work on this project together:

| Session | Where it runs | Job |
|---|---|---|
| **mentor-local** | Claude Desktop app → Code tab → **Local**, on this folder | Teaches you, builds the code with you, runs it on your data |
| **steward-cloud** | Claude Code cloud session (claude.ai/code, or Desktop → **Cloud**) | GitHub only: PRs, CI, reviews, merges, tags, releases, issues. Never teaches |

They talk to each other directly via Claude Code's cross-session messaging, with two fallbacks.
All the detailed instructions live in files Claude Code reads automatically, so the prompts you
paste are short:

- `CLAUDE.md` — shared: project, architecture, ownership, comms protocol, verified versions
- `.claude/roles/mentor.md` — local session: your profile, teaching rules, the full 8-stage curriculum
- `.claude/roles/repo-steward.md` — cloud session: GitHub duties and limits
- `.claude/settings.json` — guard rails: blocks reading `.env`, blocks force-push, asks before merges/pushes to main

---

## Step 1 — 🔧 One-time setup outside Claude (about 30 minutes)

1. **Claude plan:** Pro, Max, or Team. Cloud sessions and Remote Control need a claude.ai
   subscription; they do not work with an API key.
2. **Claude Desktop app**, updated to the latest version.
3. **Claude Code CLI.** It's used for the fallback message channel and for `--teleport`.
   - Windows PowerShell: `irm https://claude.ai/install.ps1 | iex`
   - macOS/Linux/WSL: `curl -fsSL https://claude.ai/install.sh | bash`
   - Then run `claude auth login` and choose your **claude.ai** account.
   - Check `claude --version`: cross-session messaging needs v2.1.234+ on native Windows,
     or v2.1.224+ on macOS/Linux/WSL.
4. **Git for Windows** (recommended, so Claude Code gets a Bash tool) and the **GitHub CLI**:
   `winget install --id GitHub.cli`, then `gh auth login`.
5. **GitHub repo:** create an EMPTY repo named `ai-data-analyst` (no README, .gitignore, or
   license). Public is recommended: free secret-scanning push protection, and a visible
   history for recruiters. The data and `.env` are gitignored either way.
6. **Claude GitHub App:** install it on that repo, via claude.ai/code onboarding or
   github.com/apps/claude.
7. **Anthropic Console API key** for the *project's own code*, created at platform.claude.com
   with ~$5 credit. This is separate billing from your Claude subscription. You'll paste it
   into `.env` as `APP_ANTHROPIC_API_KEY` during Stage 0.
8. ⚠️ Make sure **no system-wide `ANTHROPIC_API_KEY` environment variable** exists. If one
   does, Claude Code silently switches to API-key auth, and Remote Control, messaging, and
   cloud sessions stop working.
9. **Kaggle account.** You'll download the Olist dataset during Stage 0.

## Step 2 — Put this bundle in your project folder

Create e.g. `C:\Projects\ai-data-analyst` and extract the bundle into it. Check that the
dot-entries survived: `.claude\`, `.gitignore`, `.env.example`, `.coord\`.

## Step 3 — Start the LOCAL session

1. Open Claude Desktop → **Code** tab → **Local** → select the project folder.
2. Set the model to Sonnet and the effort to Medium (`/model`, `/effort`).
3. Run `/rename mentor-local`.
4. Turn on Remote Control with `/remote-control`, or permanently via *Settings → Claude Code →
   Connect new sessions to Remote Control*. Without it, the cloud session can't be reached.
5. Paste:

```
You are mentor-local for this project.
1. Read CLAUDE.md, then .claude/roles/mentor.md, completely, before doing anything else.
2. Follow the "First run" steps in mentor.md §9, then begin Stage 0.
3. I'm Mohit: experienced .NET + data developer, complete beginner in GenAI.
   Fast pace: theory from external resources I complete myself; chat time for building,
   decisions, and checking my understanding.
```

The mentor walks you through Stage 0. It ends with the first commit pushed to GitHub, which
the cloud session needs before it can start.

## Step 4 — Start the CLOUD session (after the first push)

Either:

- **A.** Go to claude.ai/code (or Desktop → Code → **Cloud**), pick the `ai-data-analyst`
  repo, use the Default environment, and paste the prompt below; or
- **B.** Ask the mentor to launch it. It runs
  `claude --cloud "<the prompt below>"` and asks your permission first.

```
You are steward-cloud for this repository.
1. Read CLAUDE.md, then .claude/roles/repo-steward.md, completely, before doing anything else.
2. Run the startup sequence in repo-steward.md §1 (including /rename steward-cloud and the hello message to mentor-local).
3. GitHub and repo work only. Never teach. Never edit learner-owned code.
```

If it reports that `uv` is missing, add a setup script to the cloud environment
(claude.ai/code → environment settings):
`curl -LsSf https://astral.sh/uv/install.sh | sh`

## Step 5 — Verify they can talk

In the local session, run `/list-agents`. You should see `steward-cloud`. Then ask:
*"Ping steward-cloud and confirm the channel works."* A hello/ack pair is the proof.

---

## Day-to-day

- Work only in the local session. The mentor messages the steward when a stage is ready,
  relays the review back to you, and asks you before anything gets merged.
- Type **Status Summary** any time for stage-by-stage progress with %.
- You'll get **⚠️ Fable recommended from next response** one message ahead of Stage 5, a
  stubborn bug, or the final narrative. Switch with `/model`.
- **Costs:**
  - Both sessions draw from your subscription's shared usage limits (no separate VM charge).
  - The project's code uses your ~$5 API credit.
  - Stage 8 Azure work gets a budget alert first.

## Troubleshooting

| Symptom | Likely cause → fix |
|---|---|
| `steward-cloud` not in `/list-agents` | Local Remote Control is off (`/remote-control`), the sessions use different accounts, or the cloud session expired (reopen it at claude.ai/code) |
| "Remote Control requires claude.ai subscription auth" | `ANTHROPIC_API_KEY` is set somewhere → remove it, then `claude auth login` |
| Messages arrive "held" / never delivered | One session is in bypass-permissions mode → keep both in Manual/Auto mode |
| Native channel keeps failing | Mentor falls back to `claude -p "<msg>" --cloud <session-id>`; steward falls back to PR/issue comments (CLAUDE.md §6) |
| Cloud session can't push or open PRs | Claude GitHub App isn't installed on the repo |

## What changed from the earlier chat files

- `requirements.txt` is replaced by **uv** (`pyproject.toml` + `uv.lock`). The mentor sets it up in Stage 0.
- `stage0_check.py` is regenerated in Stage 0. The env var is now `APP_ANTHROPIC_API_KEY`
  (see step 1.8 for why).
- The corpus docs moved to `docs/corpus/`. Their content is unchanged.
