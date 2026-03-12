---
name: cli-anything
description: Use when the user wants to turn any Python or Node.js program into a standalone, publishable CLI binary. Packages the program as an installable bin (PyPI console_scripts or npm bin) so it can be run from anywhere without manual setup.
---

# CLI-Anything Skill

Use this skill when the user wants to take an existing Python or Node.js program and:
- Make it runnable as a standalone command (`myapp --help`)
- Publish it to PyPI (`pip install myapp`) or npm (`npm install -g myapp`)
- Make it discoverable and usable from any directory

This skill does **not** wrap GUI applications. It packages programs that already have logic — scripts, tools, utilities — as proper distributable CLI binaries.

## Inputs

Accept one of:

- A local path to a Python script or project: `./my_tool.py`, `/path/to/project/`
- A local path to a Node.js script or project: `./my_tool.js`, `/path/to/project/`
- A GitHub repository URL (agent clones it first)

Derive the package name from the directory or script name. Normalize to lowercase with hyphens (e.g., `MyTool.py` → `my-tool`).

## Detect Project Type

Before packaging, identify which runtime the project uses:

- **Python**: presence of `*.py`, `pyproject.toml`, `setup.py`, `requirements.txt`, or `Pipfile`
- **Node.js**: presence of `*.js`, `*.ts`, `package.json`, or `node_modules/`

If both exist, ask the user which to use as the primary entry point.

## Modes

### Build (default)

Package the program as a distributable CLI binary.

**For Python projects:**

Produce this structure:

```text
<package-name>/
├── <package_name>/
│   ├── __init__.py        # version = "0.1.0"
│   ├── __main__.py        # python -m <package_name> support
│   └── cli.py             # main() entry point (Click or argparse)
├── pyproject.toml         # [project.scripts] entry point
├── README.md
└── tests/
    └── test_cli.py
```

Rules:
- Use `pyproject.toml` (not `setup.py`) with `[build-system]` using `hatchling` or `setuptools>=61`
- Define the bin under `[project.scripts]`: `<package-name> = "<package_name>.cli:main"`
- Python package name uses underscores; bin name uses hyphens
- If the original program uses `argparse`, keep it; if it has no CLI yet, add one with `click`
- The `main()` function must be importable and callable with no arguments
- Verify with `pip install -e . && <package-name> --help`

**For Node.js projects:**

Produce this structure:

```text
<package-name>/
├── bin/
│   └── <package-name>.js  # #!/usr/bin/env node shebang + entry
├── lib/
│   └── index.js           # core logic
├── package.json           # "bin" field pointing to bin/<package-name>.js
├── README.md
└── tests/
    └── cli.test.js
```

Rules:
- `package.json` must have `"bin": { "<package-name>": "bin/<package-name>.js" }`
- The bin file must start with `#!/usr/bin/env node`
- Use `commander` or `minimist` for argument parsing if none exists
- Verify with `npm link && <package-name> --help`

### Test

Write and run tests for the packaged CLI:

- Test that the binary is accessible via `subprocess` / `child_process`
- Test `--help` output
- Test core commands with realistic inputs
- All tests must pass before marking the build complete

### Publish

Guide the user through publishing to the registry.

**PyPI:**
```bash
pip install build twine
python -m build
twine upload dist/*
# Users then: pip install <package-name>
```

**npm:**
```bash
npm publish
# Users then: npm install -g <package-name>
```

Remind the user to bump the version in `pyproject.toml` / `package.json` before each publish.

## Packaging Rules

### Python

- Use `pyproject.toml`, not `setup.py`
- Package name in `[project]`: lowercase hyphens (e.g., `my-tool`)
- Module name in `[project.scripts]`: lowercase underscores (e.g., `my_tool`)
- Set `requires-python = ">=3.10"`
- List all third-party deps in `[project.dependencies]`
- Do not vendor dependencies — use `pip install`

### Node.js

- `"main"` points to `lib/index.js`
- `"bin"` field is required for global install
- `"engines": { "node": ">=18" }` recommended
- List all deps in `dependencies`, not `devDependencies`, for runtime use
- Do not bundle — publish source and let npm handle installs

## Workflow

1. Receive path or URL.
2. Clone if URL; derive package name.
3. Detect Python or Node.js.
4. Read the existing entry point (main script, `__main__`, existing CLI).
5. Create packaging scaffold around existing logic — do not rewrite the core.
6. Add or wire up a `main()` / bin entry that calls the existing logic.
7. Write minimal tests.
8. Install locally and verify the bin works from a different directory.
9. Report files added/changed and the install command.

## Output Expectations

When done, report:

- package name and version
- runtime (Python / Node.js)
- bin command name
- local install command
- publish command
- files created or modified
- test results summary
