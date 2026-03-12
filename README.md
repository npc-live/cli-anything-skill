# cli-anything Skill

Turn any Python or Node.js program into a standalone CLI binary, ready to install and publish.

## What It Does

Given a Python script or Node.js program, this skill:

1. Detects the entry point and dependencies
2. Scaffolds a proper package structure (`pyproject.toml` or `package.json` with `bin`)
3. Wires the existing logic to a `main()` / bin entry — no rewrites
4. Writes and runs basic CLI tests
5. Verifies the binary works globally after `pip install -e .` or `npm link`

The result is a package ready to publish to PyPI (`pip install <name>`) or npm (`npm install -g <name>`).

## Installation

```bash
# Claude Code
bash cli-anything-skill/scripts/install.sh

# Or manually copy to your skills directory
cp -r cli-anything-skill ~/.claude/skills/cli-anything
```

## Commands

| Command | What It Does |
|---------|-------------|
| `/cli-anything <path-or-repo>` | Package a Python or Node.js program as a CLI binary |
| `/cli-anything:publish <path>` | Publish the packaged binary to PyPI or npm |

## Example

```bash
# Package a local Python script
/cli-anything ./resize_images.py

# Package a Node.js project from GitHub
/cli-anything https://github.com/user/my-node-tool

# Publish to PyPI
/cli-anything:publish ./resize-images
```

## Scope

This skill handles **pure Python and Node.js programs**. It does not wrap GUI applications or generate scripting layers for desktop software — for that, see the `cli-anything-plugin`.
