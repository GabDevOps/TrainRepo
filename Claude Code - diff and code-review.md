# Recap: `/diff` and `/code-review` in Claude Code

Notes from trying both commands on the [ClaudeRepo](https://github.com/GabDevOps/ClaudeRepo) project (a cloud session). The examples come from there.

## 1. First, where does the code live?

Both commands depend on **where** a change is. There are three places:

| Place | What it holds | How a change gets there |
|---|---|---|
| **Working tree** (files in the session's folder, `/home/user/ClaudeRepo`) | Edits that are not committed yet | Editing a file |
| **Local commits** (the git history of that same clone) | Committed changes | `git commit` |
| **GitHub** (`origin`) | Pushed commits, branches, pull requests | `git push`, opening a PR |

> ⚠️ In a **cloud session** the "local" folder is inside a temporary container,
> **not your laptop**. The container is deleted after a period of inactivity, so
> anything not committed **and** pushed is lost.

## 2. `/diff`: see what you changed but haven't committed

**What it is:** a built-in Claude Code viewer that shows the uncommitted changes
in the session's working directory, and what changed on each turn.

**How to read it:**
- 🟥 **Red** lines (`-`): old code that the change removes
- 🟩 **Green** lines (`+`): new code that the change adds
- Plain lines: unchanged code, shown for context
- An edited line shows up as a red line followed by a green line.

**Where it CAN look**
- Uncommitted edits in the repo in the session's working directory
  (`/home/user/ClaudeRepo`).

**Where it CANNOT look**
- ❌ Changes that are already **committed**. Once you commit, they disappear from `/diff`.
- ❌ **GitHub**: remote branches and pull requests.
- ❌ **Other branches** you aren't on.
- ❌ **Your laptop's files**. A cloud session sees only its own container.

**Example from this session:** the XSS fix had already been committed and pushed,
so `/diff` showed nothing. Running `git reset --mixed HEAD~1` undid the commit but kept
the edit, which made the change uncommitted again and visible in `/diff`.

## 3. `/code-review`: have Claude look for bugs in a change

**What it is:** a skill that reviews a change for bugs and lists what it finds.
**By default it only reports and changes nothing.**

**What it can review (the "target"):**

| You type | It reviews | Where the code comes from |
|---|---|---|
| `/code-review` | The current diff | **Local** working tree (default) |
| `/code-review 3` | Pull request #3 | **GitHub** |
| `/code-review feature/mortgage-estimate` | A branch | Branches available in the clone |
| `/code-review frontend/app.js` | Specific files or folders | Local files |

**Where it CAN look**
- Local uncommitted changes (default)
- Branches the clone knows about
- Pull requests on GitHub, but **only in repos this session has access to**
  (currently `gabdevops/clauderepo`)

**Where it CANNOT look**
- ❌ Repos outside the session's GitHub access
- ❌ Your laptop's files (in a cloud session)

So `/code-review` does **not** only work on GitHub. With no target it reviews the
**local** uncommitted diff, the same changes `/diff` shows.

**Options:**
- **Effort level** first: `low`, `medium`, `high` or `xhigh` (e.g. `/code-review medium`).
  Higher levels find more, including findings it is less sure about. `max` is
  the most expensive, so use it only on purpose.
- `--comment`: posts the findings as comments on the GitHub PR (**public, so others can see them**)
- `--fix`: **edits your files** to apply the fixes
- `--max-findings <n|all|default>`: how many findings to report

**Example from this session:** run on the local XSS fix in `frontend/app.js`, it
reported 8 findings:
1. The code doesn't check `response.ok`, so backend errors show as "Could not reach the backend".
2. The error is swallowed and never logged.
3. `Number(house.price)` turns bad prices into `€0` or `€NaN`.
4. Two quick searches can race and mix their results.
5. Missing `rooms` or `size_m2` fields show as the text "undefined".
6. The local diff duplicates the commit that is still on the remote branch.
7. The price format depends on the browser's language settings (`Intl.NumberFormat` would fix it).
8. There is a leftover `innerHTML = ""` (`replaceChildren()` is cleaner).

## 4. Side by side

| | `/diff` | `/code-review` |
|---|---|---|
| Default target | Local uncommitted changes | Local uncommitted changes |
| Can read GitHub (PRs)? | ❌ No | ✅ Yes, with a PR number (repos in scope only) |
| Sees committed changes? | ❌ No | ✅ Yes, via a branch or PR target |
| Can see other branches? | ❌ No | ✅ Yes, via a branch target |
| Changes files? | ❌ Never (view only) | Only with `--fix` |
| Posts to GitHub? | ❌ Never | Only with `--comment` |

## 5. Suggested workflow

1. Edit the code.
2. `/diff` to look at the change.
3. `/code-review` to check it for bugs.
4. Approve, then commit.
5. Push the branch.
6. Open a pull request.
7. `/code-review <PR#>` for a last check on GitHub (add `--comment` to leave the notes on the PR).
