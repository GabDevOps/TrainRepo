# Basic Code Management – Pull – Commit – PR

A cheat sheet for the everyday Git + GitHub workflow.

## The full cycle

```
┌──────────────────────────────────────────────────────────────────────────┐
│ STEP          COMMAND                         WHERE IT HAPPENS           │
├──────────────────────────────────────────────────────────────────────────┤
│ 1. Sync       git checkout main               Local                      │
│               git pull                        GitHub → Local             │
│ 2. Branch     git checkout -b feature/name    Local only!                │
│ 3. Code       (write + test)                  Local                      │
│ 4. Stage      git add file1 file2             Local                      │
│ 5. Commit     git commit -m "What & why"      Local                      │
│               ↺ repeat 3–5 per logical change                            │
│ 6. Push       git push -u origin feature/name Local → GitHub             │
│ 7. PR         "Compare & pull request"        GitHub (coder opens it)    │
│ 8. Review     Comments / approve              GitHub (reviewers)         │
│ 9. Merge      "Merge pull request"            GitHub → main              │
│ 10. Clean up  "Delete branch" on GitHub       GitHub                     │
│               git checkout main               Local                      │
│               git pull                        get the merged code        │
│               git branch -d feature/name      delete local label         │
└──────────────────────────────────────────────────────────────────────────┘
```

## Each step explained

1. **Pull first.** Switch to `main` and `git pull`, so your new branch starts from the latest code.
2. **Create a branch** with `git checkout -b feature/name`. This creates it **only locally** –
   GitHub does not know about it until you push.
3. **Write code and test**, one logical change at a time (a change may span several files).
4. **Stage with `git add`.** This picks the *changes* that go into the next commit:
   - It records the file's content **as it is at that moment** – edit it again and you must `git add` again.
   - New files are invisible to Git until you `git add` them once.
5. **Commit** with `git commit -m "message"`. This saves a snapshot of the whole project.
   The commit ID (e.g. `1a9d317`) is a fingerprint (hash) of the snapshot + parent commit +
   author + date + message – not a counter.
6. **Repeat 3–5** as often as needed, then **push**. Push uploads the *commits* (snapshots + history)
   to the branch of the same name on GitHub. It does **not** touch `main`.
7. **Pull Request (PR).** Usually the coder opens it; other people (reviewers) review it.
   Automatic tests often run on it too.
8. **Merge.** If approved, the branch's commits are brought into `main`.
9. **Clean up.** Delete the branch. Its commits are now part of `main`, so nothing is lost –
   only the label goes away. (Tip: *Settings → General → Automatically delete head branches*.)

## Commands for looking around (they never change anything)

| Command                           | Shows                                              |
|-----------------------------------|----------------------------------------------------|
| `git status`                      | Current branch, and which files are changed/staged |
| `git branch`                      | Local branches (`*` = the current one)             |
| `git log --oneline --graph --all` | Commit history as a tree                           |
| `git diff`                        | Exact changed lines not yet staged                 |

## Key ideas

- **Commit** = a snapshot of the whole project + a link to its parent commit.
- **Branch** = a label pointing at one commit. It moves forward with each new commit.
- **Push** = the only way anything reaches GitHub.
- **Merge** = brings a branch's commits into `main`. The branch label stays until you delete it.

## Golden rules

- Never work directly on `main`.
- Pull before you start.
- Test before you commit.
- One commit per logical change, with a clear message.
