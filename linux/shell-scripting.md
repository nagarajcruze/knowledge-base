# Linux Shell Scripting

A structured reference guide to writing robust, safe, and maintainable Bash shell scripts.

---

## 1. Script Structure & "Strict Mode"

Every production-ready script should start with a Shebang and strict execution flags to catch errors early.

```bash
#!/bin/bash

# Enable Bash Strict Mode
set -euo pipefail

APP_NAME="backup-manager"
log() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"; }

log "Starting ${APP_NAME}..."
```

### Explaining `set -euo pipefail`
- **`-e` (Exit on Error)**: Instructs Bash to exit immediately if any command returns a non-zero exit status (error code). Prevents scripts from continuing to execute when a step fails.
- **`-u` (Unset Variables Error)**: Treats references to variables that have not been previously declared/defined as errors, halting execution. Prevents issues caused by typos in variable names.
- **`-o pipefail` (Pipeline Errors)**: Ensure that if a command inside a pipeline fails (e.g. `cat file | grep "match"`), the entire pipeline returns that error code. By default, Bash only reports the error code of the *last* command in a pipeline.

---

## 2. Variables, Parameters, & Functions

### Custom Variables & Parameter Fallbacks
- **Standard Variable**: `DB_NAME="production_db"`
- **Fallback Variable**: `DB_PORT="${ENV_DB_PORT:-5432}"`
  *If the environment variable `ENV_DB_PORT` is empty or unset, it defaults to `5432`.*

### Special Script Arguments
When invoking a script via CLI (`./script.sh arg1 arg2`), references are mapped automatically:
- **`$0`**: Name of the script currently executing (e.g. `./script.sh`).
- **`$1` - `$9`**: Positional arguments (first argument, second argument, etc.).
- **`$#`**: Number of positional arguments passed to the script.
- **`$@`**: All positional arguments representing a list. Useful for loops (`for arg in "$@"`).
- **`$$`**: Process ID (PID) of the current shell running the script.
- **`$?`**: Exit status of the most recently executed command ($0$ means success, non-zero means failure).

### Functions & Local Variables
Always declare variables within functions as `local` to prevent them from leaking into the global scope.
```bash
calculate_sum() {
  local num1="$1"
  local num2="$2"
  local sum=$((num1 + num2))
  echo "${sum}"
}

result=$(calculate_sum 10 20)
```

---

## 3. Control Structures & Operators

### File & Directory Checks
- **`-f`**: Check if path exists and is a regular file.
- **`-d`**: Check if path exists and is a directory.
- **`-s`**: Check if file exists and is not empty.
```bash
if [ -f "/etc/nginx/nginx.conf" ]; then
  log "Nginx configuration found."
fi
```

### String Comparison
- **`-z`**: True if string is empty (zero length).
- **`-n`**: True if string is not empty.
```bash
if [ -z "${USER_INPUT:-}" ]; then
  echo "Input cannot be empty."
  exit 1
fi
```

### Loop Workflows
- **Standard For Loop**:
  ```bash
  for server in web-{1..5}; do
    log "Restarting service on ${server}"
    ssh "${server}" "sudo systemctl restart nginx"
  done
  ```
- **While Loop (Streaming file contents line-by-line)**:
  ```bash
  while read -r line; do
    echo "Processing line: ${line}"
  done < servers.txt
  ```

---

## 4. Standard Streams & Redirection

Operating systems track three primary channels (File Descriptors) for command inputs and outputs:
- **`0` (stdin)**: Standard Input (keyboard or stream input).
- **`1` (stdout)**: Standard Output (console printouts).
- **`2` (stderr)**: Standard Error (diagnostic error logs printed on console).

```text
               ┌───────────┐ ── 1 (stdout) ──> [Console / file.txt]
[Input stream] │  Command  │
               └───────────┘ ── 2 (stderr) ──> [Console / error.log]
```

### Redirection Syntax
- **Overwrite File (`>`)**: Redirects stdout, wiping target file contents first.
  ```bash
  echo "text" > file.txt   # equivalent to echo "text" 1> file.txt
  ```
- **Append File (`>>`)**: Appends stdout to the bottom of the target file.
  ```bash
  echo "text" >> file.txt
  ```
- **Redirect Errors (`2>`)**: Redirects only stderr to a file, letting stdout print on console.
  ```bash
  ls /root 2> error.log
  ```
- **Merge Streams (`2>&1`)**: Redirects stderr (2) to the same destination as stdout (1).
  ```bash
  # Overwrites file.txt with both normal stdout outputs and error messages
  build_command > file.txt 2>&1
  ```
- **Silent execution (`>/dev/null 2>&1`)**: Discards both normal output and error logs entirely.
  ```bash
  command -v docker >/dev/null 2>&1
  ```

---

## 5. Robust Scripting Patterns

### Check Dependency Availability
Verify if required binaries are installed on the path before proceeding:
```bash
# Exit if git is not installed
command -v git >/dev/null 2>&1 || { echo "Error: git is required." >&2; exit 1; }
```
*Note: Writing error logs using `>&2` ensures they are sent to the standard error stream, not standard output.*

### Resource Cleanup using Traps (`trap`)
Ensure temporary files, configurations, or lock files are cleaned up, even if the script crashes or is terminated.
```bash
LOCKFILE="/tmp/backup.lock"

# Check if lock file exists to prevent concurrent script runs
if [ -f "${LOCKFILE}" ]; then
  echo "Script is already running."
  exit 1
fi

touch "${LOCKFILE}"

# Register exit trap to clean up the lock file when the shell exits
# (Catches normal exits, shell errors, and SIGINT/SIGTERM interruptions)
trap 'rm -f "${LOCKFILE}"' EXIT INT TERM

log "Processing backup operations..."
```

---

## 6. Bash Array Handling

Bash supports both **Indexed Arrays** (numeric indices) and **Associative Arrays** (key-value maps).

### A. Indexed Arrays

#### 1. Declaration & Assignment
```bash
# Explicit declaration (optional)
declare -a SERVERS

# Inline assignment
SERVERS=("web-01" "web-02" "db-01")

# Appending elements to an array
SERVERS+=("cache-01")
```

#### 2. Accessing Elements
- **Access single element** (0-indexed): `${SERVERS[0]}` (returns `web-01`).
- **Access all elements**: `${SERVERS[@]}`.
- **Get array size/length**: `${#SERVERS[@]}` (returns `4`).
- **Get list of array indices**: `${!SERVERS[@]}` (returns `0 1 2 3`).
- **Delete an element by index**: `unset SERVERS[1]` (removes `web-02` but leaves a gap in indices).

#### 3. Iteration Loops
- **Iterating over array elements (Values)**:
  ```bash
  for server in "${SERVERS[@]}"; do
    echo "Deploying to: ${server}"
  done
  ```
- **Iterating over array indices**:
  ```bash
  for i in "${!SERVERS[@]}"; do
    echo "Index ${i} maps to: ${SERVERS[$i]}"
  done
  ```

---

### B. Associative Arrays (Maps/Hashes)
Requires explicit declaration via `declare -A` (Bash 4.0+).

#### 1. Declaration & Assignment
```bash
# Explicit declaration (Mandatory)
declare -A PORT_MAP

# Adding key-value pairs
PORT_MAP=(
  ["http"]=80
  ["https"]=443
  ["ssh"]=22
)

# Adding/modifying single keys
PORT_MAP["db"]=5432
```

#### 2. Accessing & Iterating Keys
- **Access single key**: `${PORT_MAP["http"]}` (returns `80`).
- **Iterating over key-value pairs**:
  ```bash
  for service in "${!PORT_MAP[@]}"; do
    echo "Service: ${service} is bound to Port: ${PORT_MAP[$service]}"
  done
  ```