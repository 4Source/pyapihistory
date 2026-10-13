# API History

**API History** is a planned tool for comparing the public API of two versions of a codebase and producing structured, reusable descriptions of the changes. It is intended for maintainers of libraries and developer tools who need to understand how an API evolved between source directories or Git revisions.

The project is in early development. The capabilities below describe the intended design and roadmap; they should not be read as a claim that all comparison functionality is available in the current package.

## Goals and planned functionality

- **Compare source snapshots.** Compare two local source directories or two revisions in a local Git repository. The Git adapter is intended to read tracked files directly from a revision, without checking it out, modifying the working tree, or fetching remote content.
- **Analyze API symbols.** Use language-specific analyzers to discover symbols and capture relevant identity, interface, implementation, documentation, and metadata information. Source files are analyzed statically and are not imported or executed.
- **Classify changes.** Report added and removed symbols, and distinguish relationships between symbols from modifications to a symbol and lifecycle transitions. Planned classifications include renames and moves (including a symbol that is both renamed and moved), interface/signature changes, implementation changes, documentation and metadata changes, and deprecation transitions.
- **Control the API surface.** Configure which discovered symbol categories participate in comparison, including whether private or internal symbols are included. Configure which modification and lifecycle categories are reported independently of the underlying detection.
- **Explain uncertain results.** Use matching evidence and configurable thresholds to help identify likely renames and moves. Results are intended to retain confidence and relevant evidence rather than presenting uncertain matches as facts.
- **Report diagnostics and partial results.** Return structured diagnostics for problems such as invalid input, unreadable files, syntax errors, unsupported syntax, or invalid Git revisions. Where possible, continue analyzing independent inputs and distinguish successful, partial, and failed comparisons.
- **Support automation and integrations.** Expose a public Python API and a command-line interface (CLI), along with a stable, deterministic, versioned JSON representation suitable for CI pipelines and other tools.

## Architecture

API History is intended to be **one package with language-specific analyzers**, not a separate distribution for each programming language. Language-specific parsing and semantic analysis are kept behind an analyzer interface; the comparison engine and result model remain language-independent.

The planned processing flow is:

```text
source adapter (directory or Git revision)
        ↓
language analyzer
        ↓
language-neutral API symbol model
        ↓
comparison engine
        ↓
language-neutral ComparisonResult
        ↓
CLI / JSON / external integrations
```

The public result model is intended to describe each change through separate relationship, modification, and lifecycle information. This allows one symbol to be, for example, renamed and moved while also having an interface change and a deprecation transition. Confidence and diagnostics are additional result information, not part of a symbol's identity.

Changelog generation is deliberately outside the core package. Separate adapter projects (for example, `apihistory-<changelog-tool>`) can consume the public comparison result or its JSON representation and translate it into the format expected by a particular changelog tool.

## Planned features

The initial implementation focuses on Python analysis and on establishing the language-independent comparison model.

- **Python analysis first.** The initial analyzer is planned to cover common Python API constructs such as functions, asynchronous functions, classes, methods, properties and their accessors, signatures, annotations, defaults, decorators, docstrings, and deprecation metadata, subject to documented syntax/runtime compatibility.
- **Directory and Git comparisons.** Use equivalent source-selection semantics for directory snapshots and Git revisions so equivalent source content can yield equivalent results.
- **Public API and CLI.** Obtain an analyzer through a language identifier (for example, `get_analyzer("python", ...)`), configure a comparison, and reuse the analyzer for multiple comparisons. The CLI is intended to select the language through subcommands.
- **Versioned JSON output.** Export changes, diagnostics, status, language and (when available) language-version information in a deterministic format. The JSON format version will be independent of the analyzed language version.
- **Additional programming languages.** Add language subcommands and analyzers within the same package as the architecture matures. C++ is one example discussed for future support; it is not represented here as currently supported.
- **Quality and compatibility.** Build out regression and end-to-end tests, define the tested Python runtime/syntax matrix, and document static-analysis limitations and confidence semantics.

## Current status

This repository is at an early stage. The full comparison pipeline, public comparison API, language-selecting CLI, and JSON output are planned work. The command shown below is the package's current entry point, not yet an example of comparing APIs.

## Usage

### Current entry point

After installing the package, the current console entry point can be invoked with:

```bash
pyapihistory
```

At this stage, it is a minimal scaffold; the planned comparison options described below are not yet available.

### Planned usage

The following examples illustrate the intended interface and are **design examples, not commands guaranteed to work in the current release**.

Compare two source directories:

```bash
apihistory python <old-source-directory> <new-source-directory>
```

Compare two Git revisions:

```bash
apihistory python <old-revision> <new-revision>
```

The final CLI syntax and option names will be documented when implemented. Planned options include API-surface selection, reported modification/lifecycle categories, matching thresholds, and human-readable or JSON output.

The equivalent planned Python API is expected to follow this general pattern:

```python
from apihistory import get_analyzer

analyzer = get_analyzer("python")
result = analyzer.compare(before, after)
```

The exact import path, source argument types, configuration classes, and result fields remain subject to the public API implementation.

## Build and install

Python 3.9 or newer is the initial minimum runtime target. Analyzer compatibility will be claimed only for Python versions and syntax combinations that are tested.

### Install for local development

From a checkout of the repository:

```bash
python -m pip install -e .
```

### Build distributions

Install the build frontend if needed, then build source and wheel distributions:

```bash
python -m pip install build
python -m build
```

The generated distributions are written to `dist/`.

### Optional: upload to TestPyPI

Publishing is a maintainer/release task. After verifying the distributions and configuring credentials, the intended TestPyPI workflow is:

```bash
python -m pip install twine
python -m twine upload --repository testpypi dist/*
```

Install a test release from TestPyPI with:

```bash
python -m pip install --index-url https://test.pypi.org/simple/ --no-deps pyapihistory
```

These commands describe packaging workflows; they do not imply that a feature-complete release has been published.

## Development and testing

Testing, formatting, linting, type checking, and performance tooling are part of the development plan. The exact development commands should be added here once the repository's tooling and configuration are in place. Benchmarking and profiling tools are intended for development use, not as runtime dependencies.

## Limitations

API comparison is intended to be static analysis, not a guarantee of behavioral equivalence. Results depend on what the language analyzer can infer from source code. Dynamic behavior, generated APIs, runtime monkey-patching, and other constructs not represented in analyzed source may not be fully captured. Rename and move detection can be ambiguous; confidence and evidence should be considered when interpreting such results. Unsupported syntax and analysis problems should be reported through diagnostics rather than silently treated as API changes.

## Contributing and tracking work

The [issue tracker](https://github.com/4Source/pyapihistory/issues) records planned functionality, architecture decisions, and implementation work. Contributions should preserve the separation between language-specific analysis, the language-neutral comparison engine, and the public result/serialization interfaces.
