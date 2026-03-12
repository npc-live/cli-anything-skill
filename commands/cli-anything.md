# cli-anything Command

Package any Python or Node.js program as a standalone CLI binary and prepare it for publishing.

## Usage

```bash
/cli-anything <path-or-repo>
```

## Arguments

- `<path-or-repo>` - **Required.** Either:
  - A local path to a Python script, Python project directory, or Node.js project directory
  - A GitHub repository URL (the agent clones it first)

## What This Command Does

### Phase 0: Source Acquisition

- If given a GitHub URL, clone to a local working directory
- Verify the path exists and contains Python (`*.py`) or Node.js (`*.js` / `package.json`) source

### Phase 1: Analyze the Entry Point

- Find the existing entry point: `main()`, `if __name__ == "__main__"`, top-level script, or `package.json` `"main"`
- Identify the argument parsing in use: `argparse`, `click`, `commander`, `minimist`, or none
- List all third-party imports/requires to capture dependencies

### Phase 2: Scaffold the Package

**Python:**

```
<package-name>/
├── <package_name>/
│   ├── __init__.py        # version = "0.1.0"
│   ├── __main__.py        # calls main()
│   └── cli.py             # main() entry point
├── pyproject.toml
├── README.md
└── tests/
    └── test_cli.py
```

`pyproject.toml` minimal template:

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "<package-name>"
version = "0.1.0"
requires-python = ">=3.10"
dependencies = [
    # list deps here
]

[project.scripts]
<package-name> = "<package_name>.cli:main"
```

**Node.js:**

```
<package-name>/
├── bin/
│   └── <package-name>.js  # #!/usr/bin/env node
├── lib/
│   └── index.js
├── package.json
├── README.md
└── tests/
    └── cli.test.js
```

`package.json` minimal template:

```json
{
  "name": "<package-name>",
  "version": "0.1.0",
  "bin": {
    "<package-name>": "bin/<package-name>.js"
  },
  "main": "lib/index.js",
  "engines": { "node": ">=18" }
}
```

### Phase 3: Wire Up the Entry Point

- Move or import existing logic into `cli.py` (Python) or `lib/index.js` (Node.js)
- Do **not** rewrite business logic — only add the packaging wrapper
- Ensure `main()` (Python) or the bin script (Node) calls the existing code

### Phase 4: Test

Write `tests/test_cli.py` (Python) or `tests/cli.test.js` (Node.js) that:

- Runs the bin via subprocess / `child_process.exec`
- Tests `--help` prints usage
- Tests at least one real command with a known input

### Phase 5: Local Install and Verify

**Python:**
```bash
pip install -e .
<package-name> --help
```

**Node.js:**
```bash
npm link
<package-name> --help
```

Confirm the binary is found in PATH and works from a directory other than the project root.

## Success Criteria

1. Binary is accessible via `<package-name> --help` after local install
2. All tests pass
3. `pyproject.toml` or `package.json` is valid and complete
4. No hardcoded paths in the entry point
5. README documents install command and usage

## Example

```bash
# Package a local Python script
/cli-anything ./convert_images.py

# Package a Node.js project from GitHub
/cli-anything https://github.com/user/my-node-tool
```
