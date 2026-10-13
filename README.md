# API History

**API History** compares the public API of two versions of a codebase and produces structured descriptions of what changed. It helps library maintainers and integrators understand API evolution across source directories and Git revisions, assess potentially breaking changes, and use comparison results in automated release workflows.

The comparison model is language-independent. **Python is the only currently supported language; C++ support is planned.**

## Features

- **Compare source snapshots.** Compare two local source directories or two revisions in a local Git repository. Git revisions can be analyzed without checking them out or modifying the working tree.
- **Discover API symbols.** Analyze source statically, without importing or executing the code, and capture relevant symbol identity and interface information.
- **Classify changes.** Report added and removed symbols and identify modifications to signatures, implementations, documentation, and metadata, as well as deprecation transitions.
- **Identify likely renames and moves.** Use matching evidence and configurable thresholds to surface likely symbol renames or moves. Results include confidence information so uncertain matches are not presented as certain facts.
- **Configure the API surface.** Choose which symbol categories are included in a comparison, including whether private or internal symbols are considered, and select which change categories are reported.
- **Produce structured results.** Return language-neutral comparison results and deterministic, versioned JSON suitable for scripts, CI pipelines, and downstream tools.
- **Report diagnostics.** Surface issues such as invalid inputs, unreadable files, syntax errors, unsupported syntax, or invalid Git revisions. Where possible, continue analyzing independent inputs and indicate whether results are complete, partial, or failed.
- **Integrate with release tooling.** Use comparison results in CI and release workflows, or pass them to separate changelog adapters to generate release notes in a specific tool's format.

## Use cases

### 1. Review API changes between releases

Compare the source snapshots associated with two releases to get a structured view of the API additions, removals, and modifications between them. Use the result during release review to understand how the public interface has evolved.

### 2. Identify potentially breaking changes

Inspect removed symbols and interface changes—such as changed signatures—before publishing a new version. Use change classifications and confidence information to prioritize human review; API History reports evidence of changes rather than making a universal guarantee of backward compatibility.

### 3. Track API evolution across Git revisions

Compare two Git revisions without switching branches or changing the working tree. This makes it possible to review API changes between tags, commits, or other revisions as part of a development or release process.

### 4. Automate release checks and changelog preparation

Run API comparisons in a CI pipeline or release workflow and consume the structured JSON output to enforce project-specific checks. A separate changelog adapter can transform the same language-neutral results into the format required by a chosen changelog tool; changelog generation itself is kept outside the core package.

## Usage

The command-line interface selects the programming language explicitly. Python is currently supported.

### Compare source directories

```bash
pyapihistory python <old-source-directory> <new-source-directory>
```

### Compare Git revisions

```bash
pyapihistory python <old-revision> <new-revision>
```

Use the CLI's options to configure the API surface, change categories, matching thresholds, and output format. JSON output is designed for deterministic consumption by automation and other tools.

### Use the Python API

The Python API exposes the same comparison capabilities for integration into scripts and applications. A typical workflow selects the Python analyzer, configures the comparison, and consumes the result:

```python
from apihistory import get_analyzer

analyzer = get_analyzer("python")
result = analyzer.compare(before, after)
```

The result model separates relationships between symbols (such as a rename or move) from modifications and lifecycle changes. This allows a single symbol to have multiple relevant change classifications in one comparison.

## Supported languages

- **Python:** supported.
- **C++:** planned; not currently supported.

Additional language analyzers are intended to use the same language-neutral comparison model and result format, rather than requiring a separate core package for each language.

## Build and install

Python 3.9 or newer is the minimum runtime target.

### Install from a repository checkout

```bash
python -m pip install .
```

For an editable installation during local work:

```bash
python -m pip install -e .
```

### Build distributions

Install the build frontend, then build source and wheel distributions:

```bash
python -m pip install build
python -m build
```

The generated distributions are written to `dist/`.

## Limitations

API History performs static analysis; it does not guarantee behavioral equivalence or prove that a release is backward-compatible. Results depend on what the language analyzer can infer from the source. Dynamic behavior, generated APIs, runtime monkey-patching, and constructs not represented in analyzed source may not be fully captured. Rename and move detection can be ambiguous, so confidence and supporting evidence should be considered when interpreting results. Unsupported syntax and analysis problems are reported through diagnostics rather than silently treated as API changes.
