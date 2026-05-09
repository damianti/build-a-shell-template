# C++ Coding Practices — Build a Shell

This document defines the coding standards for this exercise.
Deviations from these practices are valid grounds for grade reduction.

## Useful Links

- [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html)
- [C++ Best Practices (Lefticus)](https://lefticus.gitbooks.io/cpp-best-practices/content/03-Style.html)

---

## Modular Design

Shell programs have multiple independent responsibilities: reading input, parsing commands, searching PATH, executing child processes, managing built-ins, tracking jobs. Each responsibility belongs in its own function — not all in `main`.

✅ **Each feature is a focused function:**
```cpp
std::vector<std::string> parseArgs(const std::string& line);
std::string findExecutable(const std::string& cmd);
void runForeground(const std::vector<std::string>& args);
void runBackground(const std::vector<std::string>& args);
bool handleBuiltin(const std::vector<std::string>& args);
```

❌ **Everything in one long `main`:**
```cpp
int main() {
    // 200 lines: reading, parsing, forking, searching PATH, cd, myjobs...
}
```

🔴 **Why it matters:** long functions are hard to test, debug, and grade. A grader reading your code should be able to find the fork/exec logic in under 10 seconds.

---

## Syscall Error Handling

Every POSIX syscall can fail. Always check return values and report errors to `stderr`.

✅ **Check every syscall:**
```cpp
pid_t pid = fork();
if (pid < 0) {
    std::cerr << "fork: " << strerror(errno) << '\n';
    return;
}
```

```cpp
if (execv(path.c_str(), argv.data()) == -1) {
    std::cerr << "exec: " << strerror(errno) << '\n';
    _exit(1);
}
```

❌ **Ignoring return values:**
```cpp
fork();         // no check
execv(path, argv); // if this fails, the child keeps running as a shell
```

🔴 **Why it matters:** unchecked `execv` failure spawns a duplicate shell process. Unchecked `fork` failure silently drops commands.

---

## Naming Conventions

### Constants — `UPPER_CASE_WITH_UNDERSCORES`
```cpp
const int MAX_JOBS = 64;
constexpr int STDIN_FD  = STDIN_FILENO;   // prefer the POSIX macro
constexpr int STDOUT_FD = STDOUT_FILENO;
```

### Classes — `PascalCase`
```cpp
class JobList { ... };
class ShellPrompt { ... };
```

### Functions and variables — `camelCase`
```cpp
std::string findExecutable(const std::string& cmdName);
bool isBuiltin(const std::string& cmd);
```

### Private members — underscore prefix
```cpp
class JobList {
    std::vector<Job> _jobs;
};
```

### Source files — `snake_case`
```
path_search.cpp
job_list.cpp
builtin_commands.cpp
```

---

## Error Handling in the REPL

Errors inside the shell loop must be caught and reported — never let them crash the shell. The pattern: detect the error, print to `stderr`, and continue to the next command.

✅ **Catch, report, and continue:**
```cpp
try {
    executeCommand(args);
} catch (const std::exception& e) {
    std::cerr << e.what() << '\n';
}
```

✅ **Throw with a clear message so the catch can print it:**
```cpp
if (chdir(path.c_str()) != 0) {
    throw std::runtime_error("cd: " + path + ": " + strerror(errno));
}
```

❌ **Swallowing errors silently:**
```cpp
try {
    chdir(path.c_str());
} catch (...) { }  // silent — user never knows cd failed
```

---

## No Magic Numbers

✅ Use named constants or POSIX macros:
```cpp
if (fd == STDIN_FILENO)  { ... }
if (exitCode == EXIT_SUCCESS) { ... }
```

❌ Avoid unexplained literals:
```cpp
if (fd == 0) { ... }
if (status == 0) { ... }
```

---

## Input Validation at Boundaries

Validate at the point where input enters the program (reading the command line), not inside every downstream function.

✅ **Validate once, at the top:**
```cpp
std::string line = readLine();
if (line.empty()) continue;
auto args = parseArgs(line);
if (args.empty()) continue;
// downstream functions trust args is non-empty
```

❌ **Checking emptiness everywhere:**
```cpp
void runForeground(const std::vector<std::string>& args) {
    if (args.empty()) return;  // redundant if caller already validated
    ...
}
```

---

## Documentation

Public functions should have a one-line doc comment explaining what they do — not how.

✅
```cpp
// Returns the full path of `cmd` by searching PATH, or "" if not found.
std::string findExecutable(const std::string& cmd);
```

❌
```cpp
// This function loops over PATH entries and checks if the file exists
std::string findExecutable(const std::string& cmd);
```

---

## No `using namespace std`

Never write `using namespace std;` — it pollutes the global namespace and causes silent name collisions.

✅ Always qualify:
```cpp
std::string line;
std::vector<std::string> args;
std::cerr << "error\n";
```

❌ Never:
```cpp
using namespace std;   // forbidden
string line;           // looks fine, breaks in large projects
```

---

## `nullptr`, not `NULL`

C++17 code must use `nullptr` for null pointers. `NULL` is a C macro that expands to `0` — it has no type and causes ambiguous overload resolution.

✅
```cpp
char* argv[] = { ... , nullptr };   // execv requires null terminator
pid_t pid = fork();
if (pid == 0) { /* child */ }
```

❌
```cpp
char* argv[] = { ... , NULL };      // compiles, but avoid in C++17
```

🔴 **Why it matters for this exercise:** `execv` requires the `argv` array to end with a null pointer. Forgetting `nullptr` as the last element is the most common crash in this assignment.

---

## `const` Correctness

Mark every parameter and function that does not modify its input as `const`. This prevents bugs and communicates intent to the reader.

✅
```cpp
std::string findExecutable(const std::string& cmd);
bool isBuiltin(const std::string& cmd);
void printJobs(const std::vector<Job>& jobs);
```

❌
```cpp
std::string findExecutable(std::string cmd);   // unnecessary copy
bool isBuiltin(std::string& cmd);              // implies mutation, confusing
```

---

## Multi-File Project Structure

Splitting the shell into multiple files is optional but encouraged for larger implementations. If you split:

**File layout:**
```
myshell/
├── CMakeLists.txt
├── src/
│   ├── main.cpp
│   ├── path_search.h
│   ├── path_search.cpp
│   ├── job_list.h
│   ├── job_list.cpp
│   ├── builtins.h
│   └── builtins.cpp
└── README.md
```

**CMakeLists.txt for multiple source files:**
```cmake
cmake_minimum_required(VERSION 3.16)
project(myshell)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_executable(myshell
    src/main.cpp
    src/path_search.cpp
    src/job_list.cpp
    src/builtins.cpp
)

target_include_directories(myshell PRIVATE src)
```

**Every header file must start with `#pragma once`:**
```cpp
// path_search.h
#pragma once
#include <string>

// Returns the full path of cmd by searching PATH, or "" if not found.
std::string findExecutable(const std::string& cmd);
```

**Rule:** each `.cpp` file has exactly one matching `.h` file with the same base name. `main.cpp` has no header.

---

## What Not to Commit

Your repository should contain only source files. Before submitting, make sure you have not committed:

- Build output directories and generated files (CMake creates these when you build)
- Compiled binaries and object files
- Any file your editor or IDE generates automatically

Use a `.gitignore` file to prevent these from being tracked. Figuring out which patterns to ignore is part of the exercise.

---

## Final Notes

- Keep functions short and focused on one task.
- Prefer `std::string` and `std::vector` over raw C arrays.
- For path operations, use `std::filesystem`, `access(2)`, or `stat(2)` — any is acceptable.
- Always `_exit(1)` in the child after a failed `execv` — never `return` or `exit`.
- Print user-facing errors to `stderr`, not `stdout`.
