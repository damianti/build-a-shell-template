# Exercise 2 — Build a Shell (Part 1)

## Overview

In this exercise you will implement a basic Unix shell in C++.
The shell reads commands from the user, searches for the executable manually, and launches child processes using `fork` and `exec`.

---

## Repository structure

```
myshell/
├── CMakeLists.txt
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

- The `main` branch of your repository must remain **empty** (no code, no commits beyond the initial empty state).
- Do all your work on one or more feature branches (e.g. `phase1`, `solution`).
- When the exercise is complete, open **one Pull Request** from your finished branch to `main`. That PR is what will be graded.

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

---

## AI and conversation log

You may use AI assistants subject to the rules in `AI_RULES.md`.
Give your AI the full text of `AI_RULES.md` as the **first message** before asking for any help.
Submit the full conversation as `ai_log.md` (or `ai_log.txt`) in the root of your repository.
If you did not use AI, include `ai_log.md` with the single line: `No AI assistance used.`
Submissions without an AI log will receive a **10-point deduction**.

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
