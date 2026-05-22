# Shell Scripting

## Template

```bash
#!/bin/bash
set -euo pipefail

APP_NAME="my-app"
log() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"; }
log "Starting $APP_NAME"
```

## Control Structures

```bash
# If/else
if [ -f "/path/to/file" ]; then
  echo "File exists"
fi

# For loop
for server in web-{1..5}; do
  ssh "$server" 'sudo systemctl restart nginx'
done

# While loop
while read -r line; do
  echo "Processing: $line"
done < input.txt
```

## Useful Patterns

```bash
# Check if command exists
command -v docker >/dev/null 2>&1 || { echo "Docker required"; exit 1; }

# Trap for cleanup
trap 'rm -f /tmp/lockfile' EXIT
```
