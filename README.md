# Python API History

A Python package that compares two versions of a Python codebase, identifies API changes, classifies relationships and exposes the results in a reusable format.

# Help
## Build and upload the package
```bash
py -m build
py -m twine upload --repository testpypi dist/*
```
### Install the uploaded package 
```bash
pip install --index-url https://test.pypi.org/simple/ --no-deps pyapihistory
```
or if already installed and want the latest
```bash
pip install -i https://test.pypi.org/simple/ --no-deps --upgrade pyapihistory
```

## Build and Install the package locally
```bash
pip install -e .
```

## Execute
The local as well as the uploaded package (after installation) can be executed using following command. 
```bash
pyapihistory
```