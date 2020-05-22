# xOpera orchestration tool documentation

This repository introduces initial xOpera orchestrator documentation.

## Table of Contents
  - [Purpose and description](#purpose-and-description)
    - [Documentation parts](#documentation-parts)
  - [Features and tools](#features-and-tools)
    - [Local building and testing](#local-building-and-testing)

## Purpose and description
This is the very first documentation for [xOpera TOSCA orchestration tool](https://github.com/xlab-si/xopera-opera).
The documentation is originally prepared as a microsite for [RADON Horizon 2020 project](https://radon-h2020.eu/).

### Documentation parts
The documentation RST files can be found in [docs/source](./docs/source) directory. There are 5 main parts of this docs 
that are:

- [Introduction](./source/introduction.rst)
- [Business value](./source/business-value.rst)
- [Installation](./source/installation.rst)
- [Examples](./source/examples.rst)
- [Documentation](./source/documentation.rst)

## Features and tools
For documenting xOpera orchestrator we use [Sphinx documentation tool](https://www.sphinx-doc.org/en/master/).
Here we can render Sphinx Documentation from RST files and we use [Read the Docs](https://readthedocs.org/) theme.

### Local building and testing
To test the documentation locally run the commands below:

```bash
# create and activate a new virualenv
python3 -m venv .venv && . .venv/bin/activate

# install Sphinx and RTD theme
pip install sphinx==3.0.3 sphinx-rtd-theme==0.4.3

# build the documentation files
make html

# create direcory for documentation build
# build without the prepared Makefile
mkdir build
sphinx-build source build
```

After that you will found rendered documentation HTML files in `build` folder and you can open and view them inside 
your browser. 
