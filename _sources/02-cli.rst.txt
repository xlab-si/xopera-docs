.. _xOpera CLI (opera):

******************
xOpera CLI (opera)
******************

.. _xopera_cli_logo:

.. figure:: /images/xopera-cli-mark-black-text-mid.png
    :target: _images/xopera-cli-mark-black-text-mid.png
    :width: 40%
    :align: center

.. _CLI installation:

============
Installation
============

This section explains the installation of xOpera orchestrator.

**Prerequisites**

``opera`` requires python 3 and a virtual environment.
In a typical modern Linux environment, we should already be set.
In Ubuntu, however, we might need to run the following commands:

.. code-block:: console

    $ sudo apt update
    $ sudo apt install -y python3-venv python3-wheel python-wheel-common

**Installation**

xOpera is distributed as Python package that is regularly published on `PyPI`_.
So the simplest way to test ``opera`` is to install it into virtual environment:

.. code-block:: console

    $ mkdir ~/opera && cd ~/opera
    $ python3 -m venv .venv && . .venv/bin/activate
    (.venv) $ pip install --upgrade pip
    (.venv) $ pip install opera

The development version of the package is available on `Test PyPI`_ and the installation goes as follows.

.. code-block:: console

    (.venv) $ pip install --index-url https://test.pypi.org/simple/ --extra-index-url https://pypi.org/simple/ opera

.. _CLI Quickstart:

==========
Quickstart
==========

After you `installed xOpera CLI <CLI installation>`_ into virtual environment you can test if everything is working
as expected.
We can now explain how to deploy the `hello-world`_ example.

The hello world TOSCA service template is below.

.. code-block:: yaml

    tosca_definitions_version: tosca_simple_yaml_1_3

    metadata:
      template_name: "hello-world"
      template_author: "XLAB"
      template_version: "1.0"

    node_types:
      hello_type:
        derived_from: tosca.nodes.SoftwareComponent
        interfaces:
          Standard:
            inputs:
              marker:
                value: { get_input: marker }
                type: string
            operations:
              create: playbooks/create.yaml
              delete: playbooks/delete.yaml

    topology_template:
      inputs:
        marker:
          type: string
          default: default-marker

      node_templates:
        my-workstation:
          type: tosca.nodes.Compute
          attributes:
            private_address: localhost
            public_address: localhost

        hello:
          type: hello_type
          requirements:
            - host: my-workstation

As you can see it is has only one node type defined. This `hello_type` here has two linked implementations that are
actually two TOSCA operations (`create` and `delete`) that are implemented in a form of Ansible playbooks. The Ansible
playbook for creation is shown below and it is used to create a new folder and hello world file in `/tmp` directory.

The Ansible playbook that implements the `create` TOSCA operation looks like this:

.. code-block:: yaml

    ---
    - hosts: all
      gather_facts: false

      tasks:
        - name: Make the location
          file:
            path: /tmp/playing-opera/hello
            recurse: true
            state: directory

        - name: Ansible was here
          copy:
            dest: /tmp/playing-opera/hello/hello.txt
            content: "{{ marker }}"

And the playbook for destroying the service is below.

.. code-block:: yaml

    ---
    - hosts: all
      gather_facts: false

      tasks:
        - name: Remove the location
          file:
            path: /tmp/playing-opera
            state: absent

The deployment operation returns the following output:

.. code-block:: console

   (.venv) $ git clone git@github.com:xlab-si/xopera-opera.git
   (.venv) $ cd examples/hello
   (.venv) examples/hello$ opera deploy service.yaml
   [Worker_0]   Deploying my-workstation_0
   [Worker_0]   Deployment of my-workstation_0 complete
   [Worker_0]   Deploying hello_0
   [Worker_0]     Executing create on hello_0
   [Worker_0]   Deployment of hello_0 complete

If nothing went wrong, new empty file has been created at ``/tmp/playing-opera/hello/hello.txt``.

.. code-block:: console

   (venv) examples/hello$ ls -lh /tmp/playing-opera/hello/
   total 0
   -rw-rw-rw- 1 user user 0 Feb 20 16:02 hello.txt

To delete the created directory, we can undeploy our stuff by running:

.. code-block:: console

   (venv) examples/hello$ opera undeploy
   [Worker_0]   Undeploying hello_0
   [Worker_0]     Executing delete on hello_0
   [Worker_0]   Undeployment of hello_0 complete
   [Worker_0]   Undeploying my-workstation_0
   [Worker_0]   Undeployment of my-workstation_0 complete

After that the created directory and file are deleted:

.. code-block:: console

   (venv) examples/hello$ ls -lh /tmp/playing-opera/hello/
   ls: cannot access '/tmp/playing-opera/hello/': No such file or directory

.. tip::

    You can retrieve currently installed version with ``opera --version/-v``.

.. _CLI commands reference:

======================
CLI commands reference
======================

``opera`` was originally meant to be used in a terminal as a client and it currently allows users to execute the
following CLI commands:

+---------------------+--------------------------------------------------------+
| CLI command         | Purpose and description                                |
+=====================+========================================================+
| `opera deploy`_     | deploy TOSCA service template or CSAR                  |
+---------------------+--------------------------------------------------------+
| `opera undeploy`_   | undeploy TOSCA service template or CSAR                |
+---------------------+--------------------------------------------------------+
| `opera validate`_   | validate TOSCA service template or CSAR                |
+---------------------+--------------------------------------------------------+
| `opera outputs`_    | retrieve outputs from service template                 |
+---------------------+--------------------------------------------------------+
| `opera info`_       | show information about the current project             |
+---------------------+--------------------------------------------------------+
| `opera package`_    | package templates and accompanying files into CSAR     |
+---------------------+--------------------------------------------------------+
| `opera unpackage`_  | unpackage CSAR into directory                          |
+---------------------+--------------------------------------------------------+
| `opera diff`_       | compare service templates and instances                |
+---------------------+--------------------------------------------------------+
| `opera update`_     | update/redeploy template and instances                 |
+---------------------+--------------------------------------------------------+
|| `opera notify`_    || notify the orchestrator about changes after the       |
||                    || deployment and run triggers defined in TOSCA policies |
+---------------------+--------------------------------------------------------+

The commands can be executed in a random order and the orchestrator will warn you in case of any problems.
Each CLI command is described more in detail in the following sections.

------------------------------------------------------------------------------------------------------------------------

.. _opera deploy:

deploy
######

``opera deploy`` - used to deploy and control deployment of the TOSCA application described in YAML or CSAR.

.. tabs::

    .. tab:: Usage

        .. argparse::
            :module: opera.cli
            :func: create_parser
            :prog: opera
            :path: deploy

            The ``--resume/-r`` and ``--clean-state/-c`` options are mutually exclusive.

    .. tab:: Description

        The ``opera deploy`` command is used to initiate the deployment orchestration process using the supplied TOSCA
        service template or the compressed (or uncompressed) TOSCA CSAR.
        Within this CLI command the xOpera orchestrator invokes multiple `TOSCA interface operations`_ (TOSCA
        `Standard interface` node operations and also TOSCA `Configure interface` relationship operations).
        The operations are executed in the following order:

        1. ``create``
        2. ``pre_configure_source``
        3. ``pre_configure_target``
        4. ``configure``
        5. ``post_configure_source``
        6. ``post_configure_target``
        7. ``start``

        The operation gets executed if it is defined within the TOSCA service template and has a link to the
        corresponding Ansible playbook.

        After the deployment the following files and folders will be created in your opera storage directory (by
        default that is ``.opera`` and can be changed using the ``--instance-path/-p`` flag):

        - ``root_file`` file - contains the path to the service template or CSAR
        - ``inputs`` file - JSON file with the supplied inputs
        - ``instances`` folder - includes JSON files that carry the information about the status of TOSCA node and
          relationship instances
        - ``csars`` folder contains the extracted copy of your CSAR (created only if you deployed the compressed TOSCA
          CSAR)

        The ``clean-state/-c`` option can be used if you want to clean the current deployment data and status from
        opera storage (`.opera` by default) and redeploy again from the start.
        And ``--resume/-r`` can be used to resume the deployment from where it was interrupted (due to some error for
        instance) and this will deploy only the the nodes that have not been deployed yet.
        Use ``--force/-f`` option to force the action and skip any possible yes/no prompts.
        Use ``--verbose/-v`` option to see the outputs from the TOSCA executors.

        The ``opera deploy`` CLI command also includes the ``--workers/-w`` switch, which allows users to specify the
        amount of orchestration workers (i.e., concurrent threads).
        This enables simultaneous deployment of multiple independent nodes.
        By default (if not specified) the number of workers is set to 1 (i.e., only one node is deployed at once).
        If the number of specified workers is higher than the number of independent nodes, the orchestrator will take
        care of this and will decrease the amount of workers if needed.

    .. tab:: Example

        Follow the next CLI instructions and results for the `hello-world`_ and `concurrency`_ examples.

        .. code-block:: console
            :emphasize-lines: 2, 9, 171

            (venv) $ cd misc/hello-world
            (venv) misc/hello-world$ opera deploy service.yaml
            [Worker_0]   Deploying my-workstation_0
            [Worker_0]   Deployment of my-workstation_0 complete
            [Worker_0]   Deploying hello_0
            [Worker_0]     Executing create on hello_0
            [Worker_0]   Deployment of hello_0 complete

            (venv) csars/misc-tosca-types$ opera deploy -vcf service.yaml
            [Worker_0]   Deploying my-workstation_0
            [Worker_0]   Deployment of my-workstation_0 complete
            [Worker_0]   Deploying hello_0
            [Worker_0]     Executing create on hello_0
            ***inputs***
            ['marker: default-marker']
            ***inputs***
            [Worker_0] ------------
            {
                "custom_stats": {},
                "global_custom_stats": {},
                "plays": [
                    {
                        "play": {
                            "duration": {
                                "end": "2021-11-18T09:39:50.479629Z",
                                "start": "2021-11-18T09:39:49.775013Z"
                            },
                            "id": "33b7ca13-44dc-e7a3-12b1-000000000006",
                            "name": "all"
                        },
                        "tasks": [
                            {
                                "hosts": {
                                    "opera": {
                                        "_ansible_no_log": false,
                                        "action": "file",
                                        "changed": false,
                                        "diff": {
                                            "after": {
                                                "path": "/tmp/playing-opera/hello"
                                            },
                                            "before": {
                                                "path": "/tmp/playing-opera/hello"
                                            }
                                        },
                                        "gid": 1000,
                                        "group": "anzoman",
                                        "invocation": {
                                            "module_args": {
                                                "_diff_peek": null,
                                                "_original_basename": null,
                                                "access_time": null,
                                                "access_time_format": "%Y%m%d%H%M.%S",
                                                "attributes": null,
                                                "follow": true,
                                                "force": false,
                                                "group": null,
                                                "mode": null,
                                                "modification_time": null,
                                                "modification_time_format": "%Y%m%d%H%M.%S",
                                                "owner": null,
                                                "path": "/tmp/playing-opera/hello",
                                                "recurse": true,
                                                "selevel": null,
                                                "serole": null,
                                                "setype": null,
                                                "seuser": null,
                                                "src": null,
                                                "state": "directory",
                                                "unsafe_writes": false
                                            }
                                        },
                                        "mode": "0775",
                                        "owner": "anzoman",
                                        "path": "/tmp/playing-opera/hello",
                                        "size": 4096,
                                        "state": "directory",
                                        "uid": 1000
                                    }
                                },
                                "task": {
                                    "duration": {
                                        "end": "2021-11-18T09:39:50.025377Z",
                                        "start": "2021-11-18T09:39:49.781956Z"
                                    },
                                    "id": "33b7ca13-44dc-e7a3-12b1-000000000008",
                                    "name": "Make the location"
                                }
                            },
                            {
                                "hosts": {
                                    "opera": {
                                        "_ansible_no_log": false,
                                        "action": "copy",
                                        "changed": false,
                                        "checksum": "0bcf4d22cec95aaf8b3814d5cf6a208fa119c731",
                                        "dest": "/tmp/playing-opera/hello/hello.txt",
                                        "diff": {
                                            "after": {
                                                "path": "/tmp/playing-opera/hello/hello.txt"
                                            },
                                            "before": {
                                                "path": "/tmp/playing-opera/hello/hello.txt"
                                            }
                                        },
                                        "gid": 1000,
                                        "group": "anzoman",
                                        "invocation": {
                                            "module_args": {
                                                "_diff_peek": null,
                                                "_original_basename": "tmpexrgiciv",
                                                "access_time": null,
                                                "access_time_format": "%Y%m%d%H%M.%S",
                                                "attributes": null,
                                                "dest": "/tmp/playing-opera/hello/hello.txt",
                                                "follow": true,
                                                "force": false,
                                                "group": null,
                                                "mode": null,
                                                "modification_time": null,
                                                "modification_time_format": "%Y%m%d%H%M.%S",
                                                "owner": null,
                                                "path": "/tmp/playing-opera/hello/hello.txt",
                                                "recurse": false,
                                                "selevel": null,
                                                "serole": null,
                                                "setype": null,
                                                "seuser": null,
                                                "src": null,
                                                "state": "file",
                                                "unsafe_writes": false
                                            }
                                        },
                                        "mode": "0664",
                                        "owner": "anzoman",
                                        "path": "/tmp/playing-opera/hello/hello.txt",
                                        "size": 14,
                                        "state": "file",
                                        "uid": 1000
                                    }
                                },
                                "task": {
                                    "duration": {
                                        "end": "2021-11-18T09:39:50.479629Z",
                                        "start": "2021-11-18T09:39:50.027874Z"
                                    },
                                    "id": "33b7ca13-44dc-e7a3-12b1-000000000009",
                                    "name": "Ansible was here"
                                }
                            }
                        ]
                    }
                ],
                "stats": {
                    "opera": {
                        "changed": 0,
                        "failures": 0,
                        "ignored": 0,
                        "ok": 2,
                        "rescued": 0,
                        "skipped": 0,
                        "unreachable": 0
                    }
                }
            }
            [Worker_0] ------------
            [Worker_0] ============
            [Worker_0]   Deployment of hello_0 complete

            (venv) misc/hello-world$ ../concurrency
            (venv) misc/concurrency$ opera deploy -w 10 service.yaml
            [Worker_0]   Deploying my-workstation_0
            [Worker_0]   Deployment of my-workstation_0 complete
            [Worker_0]   Deploying hello-1_0
            [Worker_2]   Deploying hello-2_0
            [Worker_3]   Deploying hello-3_0
            [Worker_0]     Executing create on hello-1_0
            [Worker_4]   Deploying hello-4_0
            [Worker_5]   Deploying hello-8_0
            [Worker_3]     Executing create on hello-3_0
            [Worker_4]     Executing create on hello-4_0
            [Worker_2]     Executing create on hello-2_0
            [Worker_1]   Deploying hello-9_0
            [Worker_7]   Deploying hello-10_0
            [Worker_5]     Executing create on hello-8_0
            [Worker_7]     Executing create on hello-10_0
            [Worker_1]     Executing create on hello-9_0
            [Worker_1]     Executing start on hello-9_0
            [Worker_4]     Executing start on hello-4_0
            [Worker_3]     Executing start on hello-3_0
            [Worker_7]     Executing start on hello-10_0
            [Worker_1]   Deployment of hello-9_0 complete
            [Worker_3]   Deployment of hello-3_0 complete
            [Worker_6]   Deploying hello-12_0
            [Worker_6]     Executing create on hello-12_0
            [Worker_4]   Deployment of hello-4_0 complete
            [Worker_1]   Deploying hello-13_0
            [Worker_1]     Executing create on hello-13_0
            [Worker_5]     Executing start on hello-8_0
            [Worker_0]     Executing start on hello-1_0
            [Worker_2]     Executing start on hello-2_0
            [Worker_6]     Executing start on hello-12_0
            [Worker_7]   Deployment of hello-10_0 complete
            [Worker_1]     Executing start on hello-13_0
            [Worker_6]   Deployment of hello-12_0 complete
            [Worker_5]   Deployment of hello-8_0 complete
            [Worker_0]   Deployment of hello-1_0 complete
            [Worker_3]   Deploying hello-5_0
            [Worker_3]     Executing create on hello-5_0
            [Worker_8]   Deploying hello-11_0
            [Worker_8]     Executing create on hello-11_0
            [Worker_1]   Deployment of hello-13_0 complete
            [Worker_2]   Deployment of hello-2_0 complete
            [Worker_8]     Executing start on hello-11_0
            [Worker_3]     Executing start on hello-5_0
            [Worker_8]   Deployment of hello-11_0 complete
            [Worker_3]   Deployment of hello-5_0 complete
            [Worker_4]   Deploying hello-6_0
            [Worker_4]     Executing create on hello-6_0
            [Worker_4]     Executing start on hello-6_0
            [Worker_4]   Deployment of hello-6_0 complete
            [Worker_9]   Deploying hello-7_0
            [Worker_9]     Executing create on hello-7_0
            [Worker_9]     Executing start on hello-7_0
            [Worker_9]   Deployment of hello-7_0 complete
            [Worker_7]   Deploying hello-14_0
            [Worker_7]     Executing create on hello-14_0
            [Worker_7]     Executing start on hello-14_0
            [Worker_7]   Deployment of hello-14_0 complete

------------------------------------------------------------------------------------------------------------------------

.. _opera undeploy:

undeploy
#########

``opera undeploy`` - undeploys application; removes all application instances and components.

.. tabs::

    .. tab:: Usage

        .. argparse::
            :module: opera.cli
            :func: create_parser
            :prog: opera
            :path: undeploy

            The ``opera undeploy`` command does not take any positional arguments.

    .. tab:: Description

        The ``opera undeploy`` command is used to tear down the deployed blueprint. Within the undeployment process the
        xOpera orchestrator invokes two TOSCA Standard interface node operations in the following order:

        1. ``stop``
        2. ``delete``

        The operation gets executed if it is defined within the TOSCA service template and has a link to the
        corresponding implementation (e.g., Ansible playbook).

        The undeploy CLI command also accepts flags similar to deploy command: ``--resume/-r``, ``--force/-f`` and
        ``--workers/-w``.

    .. tab:: Example

        Follow the next CLI instructions and results for the `hello-world`_ example.

        .. code-block:: console
            :emphasize-lines: 9

            (venv) $ cd misc/hello-world
            (venv) misc/hello-world$ opera deploy service.yaml
            [Worker_0]   Deploying my-workstation_0
            [Worker_0]   Deployment of my-workstation_0 complete
            [Worker_0]   Deploying hello_0
            [Worker_0]     Executing create on hello_0
            [Worker_0]   Deployment of hello_0 complete

            (venv) misc/hello-world$ opera undeploy
            [Worker_0]   Undeploying hello_0
            [Worker_0]     Executing delete on hello_0
            [Worker_0]   Undeployment of hello_0 complete
            [Worker_0]   Undeploying my-workstation_0
            [Worker_0]   Undeployment of my-workstation_0 complete

------------------------------------------------------------------------------------------------------------------------

.. _opera validate:

validate
########

``opera validate`` - validates the structure of TOSCA template or CSAR.

.. tabs::

    .. tab:: Usage

        .. argparse::
            :module: opera.cli
            :func: create_parser
            :prog: opera
            :path: validate

    .. tab:: Description

        With ``opera validate`` you can validate any TOSCA template or CSAR (including its inputs) and find out whether
        it's properly structured and deployable by opera.
        At the end of this operation you will receive the validation result where opera will warn you about TOSCA
        template inconsistencies if there was any.
        Since validation can be successful or unsuccessful the ``opera validate`` command has corresponding return
        codes - 0 for success and 1 for failure.
        If the validation succeeds this means that your TOSCA templates are valid and that all their implementations
        (e.g., Ansible playbooks) can be invoked.
        However, this doesn't mean that these operations will succeed.
        To also check the executors behind the TOSCA templates, you can use the ``--executors/-e`` options that will
        not deploy anything, but will just try to run the executors in test mode (e.g., Ansible check mode).

    .. tab:: Example

        Follow the next CLI instructions and results for the `misc-tosca-types-csar`_ and `hello-world`_ examples.

        .. code-block:: console
            :emphasize-lines: 2, 7

            (venv) $ cd csars/misc-tosca-types
            (venv) csars/misc-tosca-types$ opera validate -i inputs.yaml service.yaml
            Validating service template...
            Done.

            (venv) $ cd ../../hello-world
            (venv) misc/hello-world$ opera validate --executors service.yaml
            Validating service template...
            [Worker_0]   Validating my-workstation_0
            [Worker_0]   Validation of my-workstation_0 complete
            [Worker_0]   Validating hello_0
            [Worker_0]     Executing create on hello_0
            [Worker_0]     Executing delete on hello_0
            [Worker_0]   Validation of hello_0 complete
            Done.

------------------------------------------------------------------------------------------------------------------------

.. _opera outputs:

outputs
#######

``opera outputs`` - print the outputs from service template (after deployment/undeployment).

.. tabs::

    .. tab:: Usage

        .. argparse::
            :module: opera.cli
            :func: create_parser
            :prog: opera
            :path: outputs

    .. tab:: Description

        The ``opera outputs`` command lets you access the orchestration outputs defined in the TOSCA service template
        and print them out to the console in JSON or YAML format (used by default).

    .. tab:: Example

        Follow the next CLI instructions and results for the `outputs`_ example.

        .. code-block:: console
            :emphasize-lines: 7, 15

            (venv) $ cd tosca/outputs
            (venv) tosca/outputs$ opera deploy service.yaml
            [Worker_0]   Deploying my_node_0
            [Worker_0]     Executing create on my_node_0
            [Worker_0]   Deployment of my_node_0 complete

            (venv) tosca/outputs$ opera outputs
            output_prop:
              description: Example of property output
              value: 123
            output_attr:
              description: Example of attribute output
              value: my_custom_attribute_value

            (venv) tosca/outputs$ opera outputs --format json
            {
              "output_prop": {
                "description": "Example of property output",
                "value": 123
              },
              "output_attr": {
                "description": "Example of attribute output",
                "value": "my_custom_attribute_value"
              }
            }

------------------------------------------------------------------------------------------------------------------------

.. _opera info:

info
#######

``opera info`` - print the details of current deployment project.

.. tabs::

    .. tab:: Usage

        .. argparse::
            :module: opera.cli
            :func: create_parser
            :prog: opera
            :path: info

    .. tab:: Description

        With ``opera info`` user can get the information about the current opera project and can access its storage and
        state.
        This includes printing out the path to TOSCA service template entrypoint, extracted CSAR location, path to the
        storage inputs, status/state of the deployment, TOSCA CSAR/service template metadata and CSAR validation status.
        The output can be formatted in YAML or JSON. The created JSON object looks like this:

        .. code-block:: json

            {
                "service_template":          "string | null",
                "content_root":              "string | null",
                "inputs":                    "string | null",
                "status":                    "deploying | deployed | undeploying | undeployed | error | null",
                "csar_metadata":             "YAML map | null",
                "service_template_metadata": "YAML map | null",
                "csar_valid":                "true | false | null"
            }

        xOpera TOSCA orchestration tool now provides the following orchestration states (the are also CLI commands
        that trigger changing the state in the brackets):

        - `deploying` (``opera deploy``)
        - `deployed` (``opera deploy``)
        - `undeploying` (``opera undeploy``)
        - `undeployed` (``opera undeploy``)
        - `error` (``opera deploy/undeploy/update/notify``)
        - `unknown` (for special cases which might happen)

        These state are changed during the execution of TOSCA interface operations (e.g., Ansible playbooks).

    .. tab:: Example

        Follow the next CLI instructions and results for the `misc-tosca-types-csar`_ example.

        .. code-block:: console
            :emphasize-lines: 2, 33, 69, 107

            (venv) $ cd csars/misc-tosca-types
            (venv) csars/misc-tosca-types$ opera info
            service_template: service.yaml
            content_root: .
            inputs: null
            status: null
            csar_metadata:
              Name: null
              Content-Type: null
              TOSCA-Meta-File-Version: '1.1'
              CSAR-Version: '1.1'
              Created-By: XLAB
              Entry-Definitions: service.yaml
            service_template_metadata: null
            csar_valid: true

            (venv) csars/misc-tosca-types$ opera deploy -i inputs.yaml service.yaml
            [Worker_0]   Deploying my-workstation1_0
            [Worker_0]   Deployment of my-workstation1_0 complete
            [Worker_0]   Deploying my-workstation2_0
            [Worker_0]   Deployment of my-workstation2_0 complete
            [Worker_0]   Deploying file_0
            [Worker_0]     Executing create on file_0
            [Worker_0]   Deployment of file_0 complete
            [Worker_0]   Deploying hello_0
            [Worker_0]     Executing create on hello_0
            [Worker_0]   Deployment of hello_0 complete
            [Worker_0]   Deploying interfaces_0
            [Worker_0]     Executing create on interfaces_0
            ^C[Worker_0] ------------
            KeyboardInterrupt

            (venv) csars/misc-tosca-types$ opera info -f json
            {
              "service_template": "service.yaml",
              "content_root": ".",
              "inputs": {
                "host_ip": "localhost",
                "slovenian_greeting": "Hej prijatelj, pojdi z nami!"
              },
              "status": "error",
              "csar_metadata": {
                "Name": null,
                "Content-Type": null,
                "TOSCA-Meta-File-Version": "1.1",
                "CSAR-Version": "1.1",
                "Created-By": "XLAB",
                "Entry-Definitions": "service.yaml"
              },
              "service_template_metadata": null,
              "csar_valid": true
            }

            (venv) csars/misc-tosca-types$ opera deploy --resume --force
            [Worker_0]   Deploying interfaces_0
            [Worker_0]     Executing create on interfaces_0
            [Worker_0]     Executing configure on interfaces_0
            [Worker_0]     Executing start on interfaces_0
            [Worker_0]   Deployment of interfaces_0 complete
            [Worker_0]   Deploying noimpl_0
            [Worker_0]   Deployment of noimpl_0 complete
            [Worker_0]   Deploying setter_0
            [Worker_0]     Executing create on setter_0
            [Worker_0]   Deployment of setter_0 complete
            [Worker_0]   Deploying test_0
            [Worker_0]     Executing create on test_0
            [Worker_0]   Deployment of test_0 complete

            (venv) csars/misc-tosca-types$ opera info -f yaml
            service_template: service.yaml
            content_root: .
            inputs:
              host_ip: localhost
              slovenian_greeting: Hej prijatelj, pojdi z nami!
            status: deployed
            csar_metadata:
              Name: null
              Content-Type: null
              TOSCA-Meta-File-Version: '1.1'
              CSAR-Version: '1.1'
              Created-By: XLAB
              Entry-Definitions: service.yaml
            service_template_metadata: null
            csar_valid: true

            (venv) csars/misc-tosca-types$ opera undeploy
            [Worker_0]   Undeploying my-workstation2_0
            [Worker_0]   Undeployment of my-workstation2_0 complete
            [Worker_0]   Undeploying file_0
            [Worker_0]     Executing delete on file_0
            [Worker_0]   Undeployment of file_0 complete
            [Worker_0]   Undeploying interfaces_0
            [Worker_0]     Executing stop on interfaces_0
            [Worker_0]     Executing delete on interfaces_0
            [Worker_0]   Undeployment of interfaces_0 complete
            [Worker_0]   Undeploying noimpl_0
            [Worker_0]   Undeployment of noimpl_0 complete
            [Worker_0]   Undeploying setter_0
            [Worker_0]   Undeployment of setter_0 complete
            [Worker_0]   Undeploying hello_0
            [Worker_0]   Undeployment of hello_0 complete
            [Worker_0]   Undeploying my-workstation1_0
            [Worker_0]   Undeployment of my-workstation1_0 complete
            [Worker_0]   Undeploying test_0
            [Worker_0]   Undeployment of test_0 complete

            (venv) csars/misc-tosca-types$ opera info
            service_template: service.yaml
            content_root: .
            inputs:
              host_ip: localhost
              slovenian_greeting: Hej prijatelj, pojdi z nami!
            status: undeployed
            csar_metadata:
              Name: null
              Content-Type: null
              TOSCA-Meta-File-Version: '1.1'
              CSAR-Version: '1.1'
              Created-By: XLAB
              Entry-Definitions: service.yaml
            service_template_metadata: null
            csar_valid: true

------------------------------------------------------------------------------------------------------------------------

.. _opera package:

package
#######

``opera package`` - package TOSCA YAML templates and their accompanying files to a compressed (zipped) TOSCA CSAR.

.. tabs::

    .. tab:: Usage

        .. argparse::
            :module: opera.cli
            :func: create_parser
            :prog: opera
            :path: package

    .. tab:: Description

        The ``opera package`` command is used to create a valid TOSCA CSAR - a deployable zip compressed archive file
        (CSAR = Cloud Service Archive).
        TOSCA CSARs contain the blueprint of the application that we want to deploy.
        The process includes packaging together the TOSCA service template and all the accompanying files.

        In general, ``opera package`` receives a directory (where user's TOSCA templates and other files are located)
        and produces a compressed CSAR file.
        The command can create the CSAR if there is at least one TOSCA YAML file in the input folder.
        If the CSAR structure is already present (if `TOSCA-Metadata/TOSCA.meta` or exists or ``metadata`` is present
        in the TOSCA service template and if all other TOSCA CSAR constraints are satisfied) the CSAR is created
        without an additional temporary directory.
        And if not, the files are copied to the tempdir, where the CSAR structure is created and at the end the tempdir
        is compressed.
        The input folder is the mandatory positional argument, but there are also other command flags that can be used.

    .. tab:: Example

        Follow the next CLI instructions and results for the `hello-world`_ and `misc-tosca-types-csar`_ examples.

        .. code-block:: console
            :emphasize-lines: 2, 6

            (venv) $ cd misc/hello-world
            (venv) misc/hello-world$ opera package .
            CSAR was created and packed to '/home/user/Desktop/xopera-examples/misc/hello-world/opera-package-45045f.zip'.

            (venv) misc/hello-world$ cd ../../csars
            (venv) csars$ opera package -t service.yaml -o misc-tosca-types  misc-tosca-types/
            CSAR was created and packed to '/home/user/Desktop/xopera-examples/csars/misc-tosca-types.zip'.

------------------------------------------------------------------------------------------------------------------------

.. _opera unpackage:

unpackage
##########

``opera unpackage`` - uncompress TOSCA CSAR (to a specified directory).

.. tabs::

    .. tab:: Usage

        .. argparse::
            :module: opera.cli
            :func: create_parser
            :prog: opera
            :path: unpackage

    .. tab:: Description

        The ``opera unpackage`` has the opposite function of the ``opera package`` command.
        It serves for unpacking (i.e. validating and extracting) the compressed TOSCA CSAR files.
        The opera unpackage command receives a compressed CSAR as a positional argument.
        It then validates and extracts the CSAR to a given location.

        Currently, the compressed CSARs can be supplied in two different compression formats - `zip` or `tar`, but
        `zip` is prefferedf because TOSCA CSARs are supposed to be zipped archives.

    .. tab:: Example

        Follow the next CLI instructions and results for the `misc-tosca-types-csar`_ and `small-csar`_ examples.

        .. code-block:: console
            :emphasize-lines: 5, 11

            (venv) $ cd csars
            (venv) csars$ opera package -t service.yaml -o misc-tosca-types misc-tosca-types/
            CSAR was created and packed to '/home/user/Desktop/xopera-examples/csars/misc-tosca-types.zip'.

            (venv) csars$ opera unpackage misc-tosca-types.zip
            The CSAR was unpackaged to '/home/user/Desktop/xopera-examples/csars/opera-unpackage-1cabf6'.

            (venv) csars$ opera package -t service.yaml -o small small/
            CSAR was created and packed to '/home/user/Desktop/xopera-examples/csars/small.zip'.

            (venv) csars$ opera unpackage -d small-extracted small.zip
            The CSAR was unpackaged to '/home/user/Desktop/xopera-examples/csars/small-extracted'.

------------------------------------------------------------------------------------------------------------------------

.. _opera diff:

diff
####

``opera diff`` - compare TOSCA service templates and instances.

.. tabs::

    .. tab:: Usage

        .. argparse::
            :module: opera.cli
            :func: create_parser
            :prog: opera
            :path: diff

    .. tab:: Description

        The ``opera diff`` CLI command holds the functionality to find the differences between the deployed TOSCA
        service template and the updated TOSCA service template that you wish to redeploy.
        Moreover, this operation compares the desired TOSCA service template to the one from the opera project storage
        (by default this one is located in ``.opera``) and print out their differences.

        The command includes two sub-operations that invoke template and instance comparers.
        The template comparer allows the comparison of changed blueprint (and changed inputs) in a folder containing
        the existing TOSCA service template that was deployed before.
        The instance comparer looks for changes in instance states and also traverses the dependency graph in order to
        propagate changes from parent to child nodes.
        If a parent node is marked as changed, then child node is also considered changed.

        The output of ``opera diff`` is a human readable representation of templates differences, is formatted either
        as JSON or YAML (default) and can be optionally saved in a file.

    .. tab:: Example

        Follow the next CLI instructions and results for the `compare-templates`_ example.

        .. code-block:: console
            :emphasize-lines: 21

            (venv) $ cd misc/compare-templates
            (venv) misc/compare-templates$ opera deploy -i inputs1.yaml service1.yaml
            [Worker_0]   Deploying my-workstation_0
            [Worker_0]   Deployment of my-workstation_0 complete
            [Worker_0]   Deploying hello-1_0
            [Worker_0]     Executing create on hello-1_0
            [Worker_0]   Deployment of hello-1_0 complete
            [Worker_0]   Deploying hello-2_0
            [Worker_0]     Executing create on hello-2_0
            [Worker_0]   Deployment of hello-2_0 complete
            [Worker_0]   Deploying hello-3_0
            [Worker_0]     Executing create on hello-3_0
            [Worker_0]   Deployment of hello-3_0 complete
            [Worker_0]   Deploying hello-4_0
            [Worker_0]     Executing create on hello-4_0
            [Worker_0]   Deployment of hello-4_0 complete
            [Worker_0]   Deploying hello-6_0
            [Worker_0]     Executing create on hello-6_0
            [Worker_0]   Deployment of hello-6_0 complete

            (venv) misc/compare-templates$ opera diff -i inputs2.yaml service2.yaml
            nodes:
            added:
            - hello-5
            changed:
             hello-1:
               capabilities:
                 deleted:
                 - test
               interfaces:
                 Standard:
                   operations:
                     create:
                       artifacts:
                         added:
                         - lib/files/file1_2.yaml
                         deleted:
                         - lib/files/file1_1.yaml
                       inputs:
                         marker:
                         - marker1
                         - marker2
                         time:
                         - '0'
                         - '1'
                     delete:
                       artifacts:
                         added:
                         - lib/files/file1_2.yaml
                         deleted:
                         - lib/files/file1_1.yaml
                       inputs:
                         marker:
                         - marker1
                         - marker2
                         time:
                         - '0'
                         - '1'
               properties:
                 time:
                 - '0'
                 - '1'
             hello-2:
               capabilities:
                 test:
                   properties:
                     test1:
                     - '2'
                     - '3'
                     test2:
                     - '2'
                     - '3'
               dependencies:
               - hello-2
               interfaces:
                 Standard:
                   operations:
                     create:
                       artifacts:
                         added:
                         - lib/files/file2.yaml
                         deleted:
                         - lib/files/file1_1.yaml
                       inputs:
                         marker:
                         - marker1
                         - marker2
                     delete:
                       artifacts:
                         added:
                         - lib/files/file2.yaml
                         deleted:
                         - lib/files/file1_1.yaml
                       inputs:
                         marker:
                         - marker1
                         - marker2
               properties:
                 day:
                 - '1'
                 - '2'
               requirements:
                 added:
                 - dependency
               types:
               - hello_type_old
               - hello_type_new
             hello-3:
               interfaces:
                 Standard:
                   operations:
                     create:
                       inputs:
                         marker:
                         - marker1
                         - marker2
                     delete:
                       inputs:
                         marker:
                         - marker1
                         - marker2
             hello-6:
               dependencies:
               - hello-6
               interfaces:
                 Standard:
                   operations:
                     create:
                       inputs:
                         marker:
                         - marker1
                         - marker2
                     delete:
                       inputs:
                         marker:
                         - marker1
                         - marker2
               requirements:
                 dependency:
                   target:
                   - hello-1
                   - hello-2
            deleted:
            - hello-4

------------------------------------------------------------------------------------------------------------------------

.. _opera update:

update
######

``opera update`` - update the deployed TOSCA service template and redeploy it according to the discovered template diff.

.. tabs::

    .. tab:: Usage

        .. argparse::
            :module: opera.cli
            :func: create_parser
            :prog: opera
            :path: update

    .. tab:: Description

        The ``opera update`` command extends the usage of ``opera diff`` and is able to redeploy the update TOSCA
        service template according to the changes that were made to the previously deployed template.
        This means that ``opera update`` will first compare the two templates and instances with and then redeploy.

        The user is able to run update command providing a changed blueprint and inputs in a folder containing existing
        service template that was deployed before.
        The result of the execution would be undeployment of the nodes that were removed from the service template,
        deployment of the nodes that were added to the service template and consequential undeployment/deployment of
        changed nodes.

    .. tab:: Example

        Follow the next CLI instructions and results for the `compare-templates`_ example.

        .. code-block:: console
            :emphasize-lines: 21

            (venv) $ cd misc/compare-templates
            (venv) misc/compare-templates$ opera deploy -i inputs1.yaml service1.yaml
            [Worker_0]   Deploying my-workstation_0
            [Worker_0]   Deployment of my-workstation_0 complete
            [Worker_0]   Deploying hello-1_0
            [Worker_0]     Executing create on hello-1_0
            [Worker_0]   Deployment of hello-1_0 complete
            [Worker_0]   Deploying hello-2_0
            [Worker_0]     Executing create on hello-2_0
            [Worker_0]   Deployment of hello-2_0 complete
            [Worker_0]   Deploying hello-3_0
            [Worker_0]     Executing create on hello-3_0
            [Worker_0]   Deployment of hello-3_0 complete
            [Worker_0]   Deploying hello-4_0
            [Worker_0]     Executing create on hello-4_0
            [Worker_0]   Deployment of hello-4_0 complete
            [Worker_0]   Deploying hello-6_0
            [Worker_0]     Executing create on hello-6_0
            [Worker_0]   Deployment of hello-6_0 complete

            (venv) misc/compare-templates$ opera update -i inputs2.yaml service2.yaml
            [Worker_0]   Undeploying hello-2_0
            [Worker_0]     Executing delete on hello-2_0
            [Worker_0]   Undeployment of hello-2_0 complete
            [Worker_0]   Undeploying hello-3_0
            [Worker_0]     Executing delete on hello-3_0
            [Worker_0]   Undeployment of hello-3_0 complete
            [Worker_0]   Undeploying hello-4_0
            [Worker_0]     Executing delete on hello-4_0
            [Worker_0]   Undeployment of hello-4_0 complete
            [Worker_0]   Undeploying hello-6_0
            [Worker_0]     Executing delete on hello-6_0
            [Worker_0]   Undeployment of hello-6_0 complete
            [Worker_0]   Undeploying hello-1_0
            [Worker_0]     Executing delete on hello-1_0
            [Worker_0]   Undeployment of hello-1_0 complete
            [Worker_0]   Deploying hello-1_0
            [Worker_0]     Executing create on hello-1_0
            [Worker_0]   Deployment of hello-1_0 complete
            [Worker_0]   Deploying hello-2_0
            [Worker_0]     Executing create on hello-2_0
            [Worker_0]   Deployment of hello-2_0 complete
            [Worker_0]   Deploying hello-3_0
            [Worker_0]     Executing create on hello-3_0
            [Worker_0]   Deployment of hello-3_0 complete
            [Worker_0]   Deploying hello-5_0
            [Worker_0]     Executing create on hello-5_0
            [Worker_0]   Deployment of hello-5_0 complete
            [Worker_0]   Deploying hello-6_0
            [Worker_0]     Executing create on hello-6_0
            [Worker_0]   Deployment of hello-6_0 complete

------------------------------------------------------------------------------------------------------------------------

.. _opera notify:

notify
######

``opera notify`` - notify the orchestrator about changes after deployment and run triggers defined in TOSCA policies.

.. tabs::

    .. tab:: Usage

        .. argparse::
            :module: opera.cli
            :func: create_parser
            :prog: opera
            :path: notify

    .. tab:: Description

        There are cases when the user would want to execute some tasks after the deployment based on the changes that
        occur on already deployed instances at runtime.
        With ``opera notify`` command, the user can inform the orchestrator about the changes (e.g., CPU load has
        increased) and the orchestrator will invoke the operations that are needed to make necessary actions (e.g.,
        horizontal or vertical scaling of the instances).

        In general ``opera notify`` is meant to be used after the deployment (after running ``opera deploy``) to notify
        the orchestrator about some changes after the deployment.
        According to these changes (metrics) that can be specified in the notification file, the orchestrator can the
        execute the desired actions.
        In other words, ``opera notify`` introduces a use case for TOSCA policies and their TOSCA triggers as it
        enables running TOSCA policy trigger actions (these are basically just pointing to TOSCA interface operations
        from TOSCA nodes).
        Notification process is invoked on every node similar to deploy or undeploy workflows.

        As mentioned above the commands should be used after the deployment but this is not the limit as it can also be
        used during other stages of orchestration (at the beginning, before deployment, after undeployment and so on).
        The orchestrator will warn users in these non-standard scenarios because the consequences of notify can be
        crucial.

        For the CLI command, there is one mandatory positional argument called ``--trigger/-t`` (you can also use the
        ``--event/-e`` alias for this option), which stands for trigger or event name.
        So, the CLI command cannot be invoked just with ``opera notify`` and this is because you probably won't need to
        use all policy triggers, but just one or two, which you can specify with by trigger's full name or its event
        using ``--trigger/-t`` option.
        It is also recommended that you use the ``--notification/-n`` switch for the path to the notification file
        (usually a JSON file) that includes changes (e.g., metrics from monitoring tool) that will be exposed to TOSCA
        interfaces as ``notification`` variable (for example in Ansible playbooks you can use Jinja2
        ``{{ notification }}`` template to retrieve and parse the notification file contents).

    .. tab:: Example

        With ``opera notify`` and by empowering the orchestrator with the practical usage of TOSCA policies and
        triggers we wanted to enable scaling and other similar use cases that are based on policies and triggers.
        Many applications and services (e.g., AWS Lambda, Docker containers, Kubernetes solutions etc.) that are
        deployed with xOpera orchestrator often include the configuration of monitoring tool (e.g., Prometheus) that is
        able to collect certain metrics like CPU load or memory usage.
        We wanted to ensure scaling of the solutions when certain limits (from TOSCA policies) are reached (like too
        high CPU usage).
        By running opera notify the scaling scripts (e.g Ansible playbooks) are invoked and scaling can be performed
        (the metrics from monitoring tool can also be provided as a notification file).

        Follow the next CLI instructions and results for the `scaling`_ example.

        .. code-block:: console
            :emphasize-lines: 11, 21

            (venv) $ cd misc/scaling
            (venv) misc/scaling$ opera deploy service.yaml
            [Worker_0]   Deploying aws_lambda_0
            [Worker_0]     Executing create on aws_lambda_0
            [Worker_0]   Deployment of aws_lambda_0 complete
            [Worker_0]   Deploying configure_monitoring_0
            [Worker_0]     Executing configure on configure_monitoring_0
            [Worker_0]   Deployment of configure_monitoring_0 complete

            # scale down by calling scale_down_trigger event with notification_scale_down.json notification file
            (venv) misc/scaling$ opera notify -e scale_down_trigger -n files/notification_scale_down.json
            [Worker_0]   Notifying aws_lambda_0
            [Worker_0]    Calling trigger radon.triggers.scaling.ScaleDown with event scale_down_trigger
            [Worker_0]     Executing scale_down on aws_lambda_0
            [Worker_0]    Calling trigger actions with event scale_down_trigger complete
            [Worker_0]   Notification on aws_lambda_0 complete
            [Worker_0]   Notifying configure_monitoring_0
            [Worker_0]   Notification on configure_monitoring_0 complete

            # scale up by calling scale_up_trigger event with notification_scale_up.json notification file
            (venv) misc/scaling$ opera notify -e scale_up_trigger -n files/notification_scale_up.json
            [Worker_0]   Notifying aws_lambda_0
            [Worker_0]    Calling trigger radon.triggers.scaling.ScaleUp with event scale_up_trigger
            [Worker_0]     Executing scale_up on aws_lambda_0
            [Worker_0]    Calling trigger actions with event scale_up_trigger complete
            [Worker_0]   Notification on aws_lambda_0 complete
            [Worker_0]   Notifying configure_monitoring_0
            [Worker_0]   Notification on configure_monitoring_0 complete

        You can also try to deploy the `policy-triggers`_ example with the CLI instructions below.

        .. code-block:: console
            :emphasize-lines: 10, 20, 30

            (venv) $ cd tosca/policy-triggers
            (venv) tosca/policy-triggers$ opera deploy service.yaml
            [Worker_0]   Deploying workstation_0
            [Worker_0]   Deployment of workstation_0 complete
            [Worker_0]   Deploying openstack_vm_0
            [Worker_0]     Executing create on openstack_vm_0
            [Worker_0]   Deployment of openstack_vm_0 complete

            # invoke TOSCA policy scale down trigger interface operations with opera notify
            (venv) tosca/policy-triggers$ opera notify -t radon.triggers.scaling.ScaleDown
            [Worker_0]   Notifying workstation_0
            [Worker_0]   Notification on workstation_0 complete
            [Worker_0]   Notifying openstack_vm_0
            [Worker_0]    Calling trigger radon.triggers.scaling.ScaleDown with event scale_down_trigger
            [Worker_0]     Executing scale_down on openstack_vm_0
            [Worker_0]    Calling trigger actions with event scale_down_trigger complete
            [Worker_0]   Notification on openstack_vm_0 complete

            # invoke TOSCA policy scale up trigger interface operations with opera notify
            (venv) tosca/policy-triggers$ opera notify -t radon.triggers.scaling.ScaleUp
            [Worker_0]   Notifying workstation_0
            [Worker_0]   Notification on workstation_0 complete
            [Worker_0]   Notifying openstack_vm_0
            [Worker_0]    Calling trigger radon.triggers.scaling.ScaleUp with event scale_up_trigger
            [Worker_0]     Executing scale_up on openstack_vm_0
            [Worker_0]    Calling trigger actions with event scale_up_trigger complete
            [Worker_0]   Notification on openstack_vm_0 complete

            # invoke TOSCA policy auto-scale trigger interface operations with opera notify
            (venv) tosca/policy-triggers$ opera notify -t radon.triggers.scaling.AutoScale
            [Worker_0]   Notifying workstation_0
            [Worker_0]   Notification on workstation_0 complete
            [Worker_0]   Notifying openstack_vm_0
            [Worker_0]    Calling trigger radon.triggers.scaling.AutoScale with event auto_scale_trigger
            [Worker_0]     Executing retrieve_info on openstack_vm_0
            [Worker_0]     Executing autoscale on openstack_vm_0
            [Worker_0]    Calling trigger actions with event auto_scale_trigger complete
            [Worker_0]   Notification on openstack_vm_0 complete

------------------------------------------------------------------------------------------------------------------------

.. _CLI secrets and Environment variables:

=================================
Secrets and Environment variables
=================================

You can use the following environment variables:

+-----------------------------------+--------------------------------+---------------------------+
| Environment variable              | Description                    | Example value             |
+===================================+================================+===========================+
| | ``OPERA_SSH_USER``              | | Username for the Ansible ssh | | ``ubuntu``              |
| |                                 | | connection to a remote VM    | | (default is ``centos``) |
+-----------------------------------+--------------------------------+---------------------------+
| | ``OPERA_SSH_IDENTITY_FILE``     | | Path to the file containing  | | ``~/.ssh/id_ed25519``   |
| |                                 | | your private ssh key that    | |                         |
| |                                 | | will be used for a           | |                         |
| |                                 | | connection to a remote VM    | |                         |
+-----------------------------------+--------------------------------+---------------------------+
| | ``OPERA_SSH_HOST_KEY_CHECKING`` | | Disable Ansible host key     | | ``false`` or ``f``      |
| |                                 | | checking (not recommended)   | | (not case sensitive)    |
+-----------------------------------+--------------------------------+---------------------------+

.. danger::

    Be very careful with your orchestration secrets (such as SSH private keys, cloud credentials, passwords and so on)
    that are stored as opera inputs.
    To avoid exposing them don't share the inputs file and the created opera storage folder with anyone.

.. _CLI shell completion:

================
Shell completion
================

For easier usage of the CLI tool ``opera`` enables tab completion for all CLI commands and arguments.
We use `shtab`_ in our code to generate a shell completion script.
We don't have a separate command to do that since but rather a global optional argument that will print out the
completion script for the main parser.
This flag is called ``--shell-completion/-s`` and it receives a shell type to generate completion for.
Shtab currently supports `bash` and `zsh` so those are the options.
So, after running ``opera -s bash|zsh`` the generated tab completion script will be printed out.
To activate it you must source the contents which can be done with ``eval "$(opera -s bash)"`` or you can save it to a
file and then source it.

.. code-block:: console

    # print out completion script for bash shell
    (venv) $ opera -s bash
    #!/usr/bin/env bash
    # AUTOMATCALLY GENERATED by `shtab`

    _shtab_opera_subparsers=('deploy' 'diff' 'info' 'notify' 'outputs' 'package' 'undeploy' 'unpackage' 'update' 'validate')

    _shtab_opera_option_strings=('-h' '--help' '-s' '--shell-completion' '--version' '-v')
    _shtab_opera_deploy_option_strings=('-h' '--help' '--instance-path' '-p' '--inputs' '-i' '--workers' '-w' '--resume' '-r' '--clean-state' '-c' '--force' '-f' '--verbose' '-v')
    ...

    # print out completion script for zsh shell
    (venv) $ opera -s zsh
    #compdef opera

    # AUTOMATCALLY GENERATED by `shtab`

    _shtab_opera_options_=(
      "(- :)"{-h,--help}"[show this help message and exit]"
      {-s,--shell-completion}"[Generate tab completion script for your shell]:shell_completion:(bash zsh)"
      {--version,-v}"[Get current opera package version]:version:"
    )

    _shtab_opera_commands_() {
      local _commands=(
        "deploy:"
        "diff:"
        "info:"
        "notify:"
        "outputs:"
        "package:"
        "undeploy:"
        "unpackage:"
        "update:"
        "validate:"
      )
    ...

    # activate completion for bash directly
    (venv) $ eval "$(opera -s bash)"

    # activate completion for zsh directly
    (venv) $ eval "$(opera -s zsh)"

.. _CLI troubleshooting:

===============
Troubleshooting
===============

Every CLI command is equipped with ``--help/-h`` switch that displays the information about it and its arguments, and
with ``--verbose/-v`` switch which turns on debug mode and prints out the orchestration parameters and the results from
the executed Ansible playbooks.
Consider using the two switches if you face any problems.
If the issue persists please have a look at the existing `opera issues`_ or open a new one yourself.

.. _PyPI: https://pypi.org/project/opera/
.. _Test PyPI: https://test.pypi.org/project/opera/
.. _opera issues: https://github.com/xlab-si/xopera-opera/issues
.. _TOSCA interface operations: https://docs.oasis-open.org/tosca/TOSCA-Simple-Profile-YAML/v1.3/cos01/TOSCA-Simple-Profile-YAML-v1.3-cos01.html#_Toc26969470
.. _misc-tosca-types-csar: https://github.com/xlab-si/xopera-examples/tree/master/csars/misc-tosca-types
.. _small-csar: https://github.com/xlab-si/xopera-examples/tree/master/csars/small
.. _hello-world: https://github.com/xlab-si/xopera-examples/tree/master/misc/hello-world
.. _concurrency: https://github.com/xlab-si/xopera-examples/tree/master/misc/concurrency
.. _outputs: https://github.com/xlab-si/xopera-examples/tree/master/tosca/outputs
.. _attribute-mapping: https://github.com/xlab-si/xopera-examples/tree/master/tosca/attribute-mapping
.. _capability-attributes-properties: https://github.com/xlab-si/xopera-examples/tree/master/tosca/capability-attributes-properties
.. _intrinsic-functions: https://github.com/xlab-si/xopera-examples/tree/master/tosca/intrinsic-functions
.. _policy-triggers: https://github.com/xlab-si/xopera-examples/tree/master/tosca/policy-triggers
.. _opera integration tests CSAR examples: https://github.com/xlab-si/xopera-opera/tree/master/tests/integration
.. _artifacts: https://github.com/xlab-si/xopera-examples/tree/master/tosca/artifacts
.. _compare-templates: https://github.com/xlab-si/xopera-examples/tree/master/misc/compare-templates
.. _scaling: https://github.com/xlab-si/xopera-examples/tree/master/misc/scaling
.. _shtab: https://github.com/iterative/shtab
