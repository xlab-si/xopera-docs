.. _Introduction:

************
Introduction
************

**xOpera project** includes a set of tools for advanced orchestration with an orchestration tool
**xOpera orchestrator** (or shorter **opera**).

.. _xopera_side_logo:

.. figure:: /images/xopera-black-text-side-mid.png
    :target: _images/xopera-black-text-side-mid.png
    :align: center

``opera`` aims to be a lightweight orchestrator compliant with `OASIS TOSCA`_ and the current compliance is with the
`TOSCA Simple Profile in YAML v1.3`_.
``opera`` is by following TOSCA primarily a (TOSCA) cloud orchestrator which enables orchestration of automated tasks
within cloud applications for different cloud providers such as Amazon Web Services (AWS), Microsoft Azure, Google
Cloud Platform (GCP), OpenFaaS, OpenStack and so on.
Apart from that this tool can be used and integrated to other infrastructures in order to orchestrate services or
applications and therefore reduce human factor.

xOpera orchestrator engine - called xOpera library - xOpera CLI and xOpera API are an open-source project which
currently reside on GitHub inside `xopera-opera`_ and `xopera-api`_ repositories.
As an orchestration tool xOpera uses `Ansible`_ automation tool to implement the the TOSCA standard and to run its
interface operations via Ansible playbook actuators which again opens a lot of new possibilities.

.. _xopera_architecture:

.. figure:: /images/xopera-architecture.png
    :target: _images/xopera-architecture.png
    :align: center

    The xOpera components.

Currently a set of components is presented in figure :numref:`xopera_architecture`.
This documentation covers all the xOpera tools.

We can point out the following tools:

- :ref:`xOpera CLI (opera)` is a command line interface to the **xOpera library** for deploying TOSCA templates and
  CSARs.
- :ref:`xOpera API` API allows integration of **xOpera library**.
- :ref:`xOpera SaaS` is a standalone service for application lifecycle management with xOpera orchestrator
  through GUI and API.
- :ref:`xOpera TPS (Template Library)` or Template Publishing Service is a library of published TOSCA templates and
  CSARs.
- :ref:`IaC Scanner` is a service that check your IaC for known vulnerabilities.

The xOpera project was also presented within the `TOSCA Implementation Stories`_.
The following video displays how users can benefit from xOpera project.

.. raw:: html

    <div style="text-align: center; margin-bottom: 2em;">
    <iframe width="100%" height="350" src="https://www.youtube.com/embed/NZLYWB9uxjk" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
    </div>

.. _Background:

==========
Background
==========

.. _xopera_core_logo:

.. figure:: /images/xopera-core-mark-black-text-mid.png
    :target: _images/xopera-core-mark-black-text-mid.png
    :width: 40%
    :align: center

xOpera is a TOSCA standard compliant orchestrator that is following the paradigm of having a minimal set of
features and is currently focusing on Ansible.
xOpera is following the traditional UNIX philosophy of having a tool that does one thing, and does it right.
So, with a minimal set of features xOpera will do just the orchestration, and do it well.

xOpera is available on GitHub under Apache License 2.0.

TOSCA stands for the OASIS Topology and Orchestration Specification for Cloud Applications (TOSCA) standard.
It's an industry-developed and supported standard, still lively and fast to adopt new technologies, approaches and
paradigms.
It's however mostly backwards compatible, so staying within the realm of TOSCA is currently a sound and, from the
longevity perspective, a wise decision.

Using the TOSCA as the system-defining language for the xOpera means that we have an overarching declarative way that
manages the actual deployment.
The Ansible playbooks are now in the role of the actuators, tools that concretise the declared system, its topology and
contextualisation of the components and networking.

This design takes the best of both worlds. TOSCA service template is a system definition, written in proverbial stone,
while the qualities of the individual Ansible playbooks are now shining.
Within the playbooks, we can now entirely focus on particular elements of the overall system, such as provisioning
virtual machines at the cloud provider, installing and configuring a service on a target node, etc.
xOpera, in its capacity, takes care of all the untidy inter-playbook coordination, state of the deployment and so on.

.. note::

    More about xOpera's background, its origins and goals can be found here:

    - `xOpera - Get your orchestra(tor) pitch perfect`_
    - `xOpera - an agile orchestrator`_

.. _Parser:

======
Parser
======

.. note::

   *TBD*: This part of the documentation will be improved in the future.

xOpera orchestrator has its own YAML and TOSCA parser which is shown on the image below
(:numref:`opera_parser_structure`.)

.. _opera_parser_structure:

.. figure:: /images/opera-parser-structure.png
    :target: _images/opera-parser-structure.png
    :width: 40%
    :align: center

    xOpera parser and executor

.. _Business value:

==============
Business value
==============

Orchestrator xOpera is a tool that follows OASIS TOSCA standard and unlocks its benefits of providing multi-cloud
orchestration support.
According to this business value of this tool can be seen through different perspectives.
Firstly, the aim is to have an orchestration tool which is easy to install and easy to use.
Installation process is very simple and requires only python and possibly a virtual environment.
Opera package is visible on PyPI so even older versions can be installed.
Secondly, opera is a very dynamic tool which can be used for simple or complex orchestration of applications because
one can prepare his own TOSCA templates that are unique and based on his use case.
All of the templates can also be modified and reused with xOpera.
And lastly, as opera is evolving, there are many different CLI commands that can be used with it.
Besides deploying and undeploying a solution, opera can be used for TOSCA CSAR service template validation or for
printing out outputs of the orchestration.

.. _xOpera SaaS and Template Library overview:

=========================================
xOpera SaaS and Template Library overview
=========================================

The `xOpera`_ ecosystem includes tools that target optimizing deployment processes and reducing the human factor along
with a faster preparation of deployment scripts.
The video points out the most crucial functionalities of xOpera SaaS and TPS:

- Template Library Publishing Service (TPS) opens up a place for publishing, storing, managing, downloading and
  versioning of OASIS TOSCA modules and blueprints (i.e., TOSCA CSARs).
- Similar templates can be grouped together to form a FaaS abstraction layer such as a bundle of ready to use templates
  for deployment to cloud providers (e.g., AWS, Azure, GCP, OpenFaaS, etc.).
- Template groups in TPS can be used for connecting to corresponding groups of users and therefore enable working on
  different templates in a team and sharing them with other teams later.
- TPS brings different modes of interaction such as REST API, CLI client, browser-based GUI and Eclipse Che/VS Code
  plugin.
- Published deployment scripts in TPS can orchestrate the deployment with xOpera SaaS, which introduces a browser
  service for orchestration with a lightweight opera orchestrator compliant with OASIS TOSCA standard and powered by
  Ansible automation engine.
- Users can choose the corresponding templates and create a new project, secrets and credentials for deployment. Then
  they can deploy the application and observe the progress and status of the deployment.
- It is possible to organize multiple projects in multiple workspaces, manage provider credentials and assign them
  directly to workspaces. They can all run concurrently and users can even share the workspaces with other members.
- Apart from standard validation, deployment and un-deployment, xOpera SaaS also offers more complex orchestration
  actions such as redeployment, discovering template differences or invoking TOSCA policy triggers to enable vertical
  or horizontal scaling.
- The SaaS component is available through an API, GUI or Eclipse Che/VS Code plugin. The core part of the SaaS is the
  `opera`_ orchestrator, which is CLI and can be installed as a Python package from PyPI.

.. raw:: html

    <div style="text-align: center; margin-bottom: 2em;">
    <iframe width="100%" height="350" src="https://www.youtube.com/embed/0hpKJ_LBlk8" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
    </div>

The following videos show how xOpera SaaS and Template Library work in action:

- `TPS with CLI`_
- `TPS with Eclipse Che`_
- `xOpera SaaS with GUI`_
- `xOpera SaaS with Eclipse Che`_

.. _OASIS TOSCA: https://www.oasis-open.org/committees/tc_home.php?wg_abbrev=tosca
.. _TOSCA Simple Profile in YAML v1.3: https://docs.oasis-open.org/tosca/TOSCA-Simple-Profile-YAML/v1.3/TOSCA-Simple-Profile-YAML-v1.3.html
.. _xopera-opera: https://github.com/xlab-si/xopera-opera
.. _xopera-api: https://github.com/xlab-si/xopera-api
.. _Ansible: https://www.ansible.com/
.. _TOSCA Implementation Stories: https://www.oasis-open.org/tosca-implementation-stories/
.. _xOpera - an agile orchestrator: https://www.sodalite.eu/content/xopera-agile-orchestrator
.. _xOpera - Get your orchestra(tor) pitch perfect: https://www.xlab.si/sl/blog/xopera-get-your-orchestrator-pitch-perfect/
.. _opera: https://pypi.org/project/opera/
.. _xOpera: https://xlab-si.github.io/xopera-docs/
.. _TPS with CLI: https://youtu.be/28eTwojw5ac
.. _TPS with Eclipse Che: https://youtu.be/vCjfZ4Iue0E
.. _xOpera SaaS with GUI: https://youtu.be/T4XviKWLc-A
.. _xOpera SaaS with Eclipse Che: https://youtu.be/SIiLOe5dSqc
