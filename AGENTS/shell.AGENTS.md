# Shell Scripts (Bash/Zsh) — AGENTS Protocol

## Shell Scripts (Bash/Zsh)

### Minimum Required Linters

- **shellcheck** — the only shell linter you need, and you need it badly

Install if missing:
```bash
apt install shellcheck  # or brew install shellcheck
```

Run:
```bash
shellcheck -x script.sh
```

### Always Start with Strict Mode

```bash
#!/usr/bin/env bash
set -euo pipefail
# -e: exit on error
# -u: treat unset variables as errors
# -o pipefail: catch errors in piped commands

# GOOD: Script fails immediately on any error

# BAD: No set flags, errors silently ignored
#!/bin/bash
cd /some/path    # might fail
rm -rf ./build   # deletes from wrong directory
```

### Quote All Variables

```bash
# GOOD
rm -rf "${BUILD_DIR}/output"
cp "${source_file}" "${dest_dir}/"

# BAD: Word splitting and glob expansion
rm -rf $BUILD_DIR/output    # if BUILD_DIR is empty, this is rm -rf /output
cp $source_file $dest_dir/  # breaks on spaces in paths
```

### Check Command Existence

```bash
# GOOD
if ! command -v docker &>/dev/null; then
    echo "ERROR: docker is not installed" >&2
    exit 1
fi

# BAD: Assumes tools exist
docker build .  # cryptic error if docker isn't installed
```

---

---

> **This file is part of the AGENTS protocol.** For general rules, workflow, and the full table of contents, see [AGENTS.md](./AGENTS.md).
