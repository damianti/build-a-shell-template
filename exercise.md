# Exercise 2 — Build a Shell (Part 1)

## Overview

In this exercise you will implement a basic Unix shell in C++.
The shell reads commands from the user, searches for the executable manually, and launches child processes using `fork` and `exec`.

---

## Files you received

| File | What it is |
|------|------------|
| `README.md` | Fill this in — it is part of your grade |
| `exercise.md` | This file — the full assignment; read it before writing any code |
| `AI_RULES.md` | Rules for your AI assistant; send it as your **first message** before asking for help — you do not need to read it yourself |
| `PRACTICES.md` | C++ coding standards for this exercise; keep it open as a reference while coding |

---

## Repository structure

```
myshell/
├── CMakeLists.txt        ← you create this
├── src/
│   └── main.cpp          ← your implementation
└── README.md
```

**Build commands:**
```bash
cmake -S . -B build
cmake --build build
./build/myshell
```

---

## Git Workflow

- The `main` branch must remain **empty** — no code, no commits beyond the initial state.
- All work goes on a feature branch (e.g. `solution`, `phase1`).
- When done, open **one** Pull Request from your branch to `main`. That PR is what will be graded.
- Do not open multiple PRs.

```bash
# 1. Clone and enter the repo
git clone <your-repo-url> && cd <repo-name>

# 2. Create a feature branch
git checkout -b solution

# 3. Work and commit regularly
git add src/main.cpp CMakeLists.txt
git commit -m "implement fork/exec loop"

# 4. Push your branch
git push origin solution

# 5. Open a PR on GitHub: solution → main. Leave it open — do not merge.
```

**Before submitting, verify:**
- `main` branch has no code commits
- Your PR is open (not merged, not draft)
- `ai_log.md` is present in the root of your branch
- Project builds: `cmake -S . -B build && cmake --build build`

---

## Requirements

### 1. Read-eval-print loop

Your shell must display a prompt, read a line of input, execute the command, and repeat until the user types `exit` or sends EOF (`Ctrl-D`).

### 2. Fork + exec

Use `fork(2)` and `execv(2)` to launch each command in a child process. Wait for the child to finish with `waitpid(2)`. If `fork` fails, print an error to stderr and continue.

### 3. Manual PATH search — `<filesystem>` required

**You must NOT use `execvp` or `execlp`** (functions that search PATH automatically).
Instead, implement the search manually:
- Split the `PATH` environment variable on `:`.
- For each directory, construct the full path and check if the executable exists and is executable.
- Use `std::filesystem` (`<filesystem>`, C++17) for path operations.
- Support absolute paths (e.g. `/bin/echo hello`) and relative paths (e.g. `./myscript.sh`) directly, without searching PATH.

### 4. Arguments

Parse the command line and pass arguments to the child process correctly.
Example: `ls -la /tmp` → `exec` is called with `["ls", "-la", "/tmp"]`.

### 5. Background execution with `&`

If the last token on the command line is `&`, run the child in the background (do not wait).
Print the PID of the background process: `[PID] <pid>`.

### 6. Built-in: `myjobs`

The `myjobs` command (internal — do not fork) lists all background processes started by the shell.
For each, display:
```
[<pid>] <command> <status>
```
Where status is `Running` or `Done` (check with `waitpid(..., WNOHANG)`).
Remove `Done` entries from the list after displaying them.

### 7. Built-in: `cd`

Implement `cd [path]` internally (do not fork). With no argument, change to `$HOME`. Print an error to stderr if the path does not exist.

Make sure `cd ..` and `cd .` work correctly as relative paths.

---

## Bonus (15 pts)

#### B1 — Environment variable expansion (10 pts)

Before executing a command, expand `$VAR` and `${VAR}` references in arguments using the current environment.
Example: `echo $HOME` should print the home directory.

#### B2 — Command history (5 pts)

- Maintain a history of executed commands.
- Save history to `~/.myshell_history` on exit; load it on startup.
- Implement built-in `myhistory` — prints a numbered list of previous commands.

---

## Constraints

- Language: **C++17**
- No external libraries beyond the C++ standard library and POSIX
- Build must succeed with `cmake -S . -B build && cmake --build build`
- **Do not** use `execvp` or `execlp` — manual PATH search is required
- **Do not** use `system(3)` — it uses `/bin/sh` internally
- Code must follow the standards in `PRACTICES.md`

---

## AI and conversation log

You may use AI assistants. Before asking for any help, send the full text of `AI_RULES.md` as your **first message** — the AI reads it, not you.

Submit the full conversation as `ai_log.md` (or `ai_log.txt`) in the root of your repository.
If you did not use AI, include `ai_log.md` with the single line: `No AI assistance used.`

**Submissions without an AI log receive a 10-point deduction.**

---

## Grading

| Criterion | Points |
|-----------|--------|
| CMakeLists.txt present, project builds cleanly | 6 |
| README.md (purpose, build/run instructions, file roles) | 4 |
| `fork` + `execv` + `waitpid` + fork failure handling | 20 |
| Manual PATH search using `std::filesystem` | 20 |
| Arguments passed correctly to child process | 5 |
| Background `&` + `myjobs` (Running/Done, cleanup) | 15 |
| `cd` built-in (no fork, `$HOME`, error to stderr) | 15 |
| Git workflow (branch, single PR, clean `main`) | 10 |
| No use of `system(3)` / `execvp` / `execlp` | 5 |
| **Total** | **100** |
| Bonus B1 — environment variable expansion | +2.5 |
| Bonus B2 — command history | +2.5 |

See `../grading/rubric.json` for detailed criteria.
