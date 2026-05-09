# Submission Workflow

## Rules

- The `main` branch must remain **empty** — no code, no commits beyond the initial state.
- All work goes on a feature branch (e.g. `solution`, `phase1`).
- When done, open **one** Pull Request from your branch to `main`. That PR is what will be graded.
- Do not open multiple PRs (one per feature, one per phase, etc.) — a single PR is required.

## Step by step

**1. Clone the template repository**
```bash
git clone <your-repo-url>
cd <repo-name>
```

**2. Create a feature branch**
```bash
git checkout -b solution
```

**3. Work on your branch — commit regularly**
```bash
git add src/main.cpp CMakeLists.txt
git commit -m "implement fork/exec loop"
```

**4. Push your branch**
```bash
git push origin solution
```

**5. Open a Pull Request**

On GitHub: open a PR from `solution` → `main`.
Leave it **open** — do not merge. The instructor will review and grade it there.

## Verify before submitting

- `main` branch has no code commits.
- Your PR is open (not merged, not draft).
- `ai_log.md` (or `ai_log.txt`) is present in the root of your branch.
- The project builds with:
  ```bash
  cmake -S . -B build && cmake --build build
  ```
