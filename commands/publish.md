# cli-anything:publish Command

Publish a packaged CLI binary to PyPI or npm.

## Usage

```bash
/cli-anything:publish <package-path>
```

## Arguments

- `<package-path>` - **Required.** Local path to the package directory containing `pyproject.toml` (Python) or `package.json` (Node.js).

## What This Command Does

### Step 1: Pre-publish Checks

- Verify `pyproject.toml` or `package.json` exists and has required fields
- Verify version is set (not `0.0.0` or placeholder)
- Run tests and confirm all pass
- Check the bin command works locally

### Step 2: Build

**Python:**
```bash
pip install build
python -m build
# Produces dist/<package-name>-<version>.tar.gz and .whl
```

**Node.js:**
```bash
npm pack
# Produces <package-name>-<version>.tgz for inspection
```

### Step 3: Publish

**Python (PyPI):**
```bash
pip install twine
twine upload dist/*
```

Requires `~/.pypirc` with credentials or `TWINE_USERNAME` / `TWINE_PASSWORD` env vars.

**Node.js (npm):**
```bash
npm publish
```

Requires `npm login` to be completed beforehand.

### Step 4: Verify

After publishing, verify the package is available:

**Python:**
```bash
pip install <package-name>
<package-name> --help
```

**Node.js:**
```bash
npm install -g <package-name>
<package-name> --help
```

## Notes

- Bump version before each publish — PyPI and npm reject duplicate versions
- Use `twine check dist/*` to validate the Python package before upload
- Use `npm pack` to preview what files will be published before `npm publish`
- Test publish to TestPyPI first: `twine upload --repository testpypi dist/*`

## Example

```bash
# Publish a Python package
/cli-anything:publish ./my-tool

# Publish a Node.js package
/cli-anything:publish ./my-node-tool
```
