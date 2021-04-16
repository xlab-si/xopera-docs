# xOpera documentation
This repository holds the documentation for xOpera and is published on https://xlab-si.github.io/xopera-docs.

[![GitHub Workflow Status](https://img.shields.io/github/workflow/status/xlab-si/xopera-docs/xOpera%20docs%20workflow?label=ci/cd)](https://github.com/xlab-si/xopera-docs/actions/workflows/docs.yaml)
[![GitHub deployments](https://img.shields.io/github/deployments/xlab-si/xopera-docs/github-pages?label=gh-pages)](https://github.com/xlab-si/xopera-docs/deployments)
[![GitHub contributors](https://img.shields.io/github/contributors/xlab-si/xopera-docs)](https://github.com/xlab-si/xopera-docs/graphs/contributors)

## Table of Contents
  - [Description](#purpose-and-description)
  - [Local building and testing](#local-building-and-testing)

## Purpose and description
This was initially the documentation for [xOpera TOSCA orchestration tool](https://github.com/xlab-si/xopera-opera) and
has been later extended to document all the related xOpera tools and services.

## Local building and testing
For documenting xOpera orchestrator we use [Sphinx documentation tool](https://www.sphinx-doc.org/en/master/).
Here we can render Sphinx Documentation from RST files and we use [Read the Docs](https://readthedocs.org/) theme.

To test the documentation locally run the commands below:

```console
# create and activate a new virualenv
python3 -m venv .venv && . .venv/bin/activate

# update pip and install Sphinx requirements
pip install --upgrade pip
pip install -r requirements.txt

# build the HTML documentation
sphinx-build -M html docs build

# build the Latex and PDF documentation
sphinx-build -M latexpdf docs build
```

After that you will found rendered documentation HTML files in `build` folder and you can open and view them inside 
your browser. 
