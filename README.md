# xOpera documentation
This repository holds the documentation for xOpera and is published on https://xlab-si.github.io/xopera-docs.

[![GitHub Workflow Status](https://img.shields.io/github/workflow/status/xlab-si/xopera-docs/xOpera%20docs%20workflow?label=ci/cd)](https://github.com/xlab-si/xopera-docs/actions/workflows/docs.yaml)
[![GitHub deployments](https://img.shields.io/github/deployments/xlab-si/xopera-docs/github-pages?label=gh-pages)](https://github.com/xlab-si/xopera-docs/deployments)
[![GitHub contributors](https://img.shields.io/github/contributors/xlab-si/xopera-docs)](https://github.com/xlab-si/xopera-docs/graphs/contributors)

## Table of Contents
  - [Description](#purpose-and-description)
  - [Local building and testing](#local-building-and-testing)
  - [License](#license)
  - [Contact](#contact)
  - [Acknowledgement](#acknowledgement)

## Purpose and description
This was initially the documentation for [xOpera TOSCA orchestration tool] and has been later extended to document all 
the related xOpera tools and services.

## Local building and testing
For documenting xOpera orchestrator we use [Sphinx documentation tool].
Here we can render Sphinx Documentation from RST files and we use [Read the Docs] theme.

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

## License
This work is licensed under the [Apache License 2.0].

## Contact
You can contact the xOpera team by sending an email to [xopera@xlab.si].

## Acknowledgement
This project has received funding from the European Union’s Horizon 2020 research and innovation programme under Grant 
Agreements No. 825040 ([RADON]), No. 825480 ([SODALITE]) and No. 101000162 ([PIACERE]).

[xOpera TOSCA orchestration tool]: https://github.com/xlab-si/xopera-opera
[Sphinx documentation tool]: https://www.sphinx-doc.org/en/master/
[Read the Docs]: https://readthedocs.org/
[Apache License 2.0]: https://www.apache.org/licenses/LICENSE-2.0
[xopera@xlab.si]: mailto:xopera@xlab.si
[RADON]: http://radon-h2020.eu
[SODALITE]: http://www.sodalite.eu/
[PIACERE]: https://www.piacere-project.eu/
