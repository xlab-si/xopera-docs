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


.. _CLI Quickstart:

==========
Quickstart
==========

After you `installed xOpera CLI <CLI installation>`_ into virtual environment you can test if everything is working
as expected. We can now explain how to deploy the `hello-world`_ example.

The hello world TOSCA service template is below.

.. code-block:: yaml

    ---
    tosca_definitions_version: tosca_simple_yaml_1_3

    node_types:
      hello_type:
        derived_from: tosca.nodes.SoftwareComponent
        interfaces:
          Standard:
            inputs:
              content:
                default: { get_input: content }
                type: string
            operations:
              create: playbooks/create.yml
              delete: playbooks/delete.yml

    topology_template:
      inputs:
        content:
          type: string
          default: "Hello from Ansible and xOpera!\n"

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
    ...

As you can see it is has only one node type defined. This `hello_type` here has two linked implementations that are
actually two TOSCA operations (create and delete) that are implemented in a form of Ansible playbooks. The Ansible
playbook for creation is shown below and it is used to create a new folder and hello world file in `/tmp` directory.

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

And the playbook for destroying the service is below.

.. code-block:: yaml

    ---
    - hosts: all
      gather_facts: false

      tasks:
        - name: Remove the location
          file:
            path: /tmp/opera-test
            state: absent
    ...

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

.. _CLI commands reference:

======================
CLI commands reference
======================

``opera`` was originally meant to be used in a terminal as a client and it currently allows users to execute the
following commands:

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
| `opera package`_    | retrieve outputs from service template                 |
+---------------------+--------------------------------------------------------+
| `opera unpackage`_  | retrieve outputs from service template                 |
+---------------------+--------------------------------------------------------+
| `opera diff`_       | compare service templates and instances                |
+---------------------+--------------------------------------------------------+
| `opera update`_     | update/redeploy template and instances                 |
+---------------------+--------------------------------------------------------+
|| `opera notify`_    || notify the orchestrator about changes after the       |
||                    || deployment and run triggers defined in TOSCA policies |
+---------------------+--------------------------------------------------------+
| `opera init`_       | initialize the service template or CSAR (*deprecated*) |
+---------------------+--------------------------------------------------------+

The commands can be executed in a random order and the orchestrator will warn and the orchestrator will warn you in
case if any problems.
Each CLI command is described more in detail in the following sections.

------------------------------------------------------------------------------------------------------------------------

.. _opera deploy:

deploy
######

``opera deploy`` - used to deploy and control deployment of the application described in YAML or CSAR.

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
        service template or the compressed TOSCA CSAR.
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
        default that is ``.opera`` and can be changed using the ``--instance-path`` flag):

        - ``root_file`` file - contains the path to the service template or CSAR
        - ``inputs`` file - JSON file with the supplied inputs
        - ``instances`` folder - includes JSON files that carry the information about the status of TOSCA node and
          relationship instances
        - ``csars`` folder contains the extracted copy of your CSAR (created only if you deployed the compressed TOSCA
          CSAR)

    .. tab:: Example

        Follow the next CLI instructions and results for the `hello-world`_ example.

        .. code-block:: console
            :emphasize-lines: 2

            (venv) $ cd misc/hello-world
            (venv) misc/hello-world$ opera deploy service.yaml
            [Worker_0]   Deploying my-workstation_0
            [Worker_0]   Deployment of my-workstation_0 complete
            [Worker_0]   Deploying hello_0
            [Worker_0]     Executing create on hello_0
            [Worker_0]   Deployment of hello_0 complete

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
        corresponding implementation (e.g. Ansible playbook).

    .. tab:: Example

        Follow the next CLI instructions and results for the `hello-world`_ example.

        .. code-block:: console
            :emphasize-lines: 8

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
        Since validation can be successful or unsuccessful the `opera validate` commands has corresponding return
        codes - 0 for success and 1 for failure.
        If the validation succeeds this means that your TOSCA templates are valid and that all their implementations
        (e.g. Ansible playbooks) can be invoked. However, this doesn't mean that these operations will succeed.

    .. tab:: Example

        Follow the next CLI instructions and results for the `misc-tosca-types-csar`_ example.

        .. code-block:: console
            :emphasize-lines: 2

            (venv) $ cd misc/hello-world
            (venv) csars/misc-tosca-types$ opera validate -i inputs.yaml service.yaml
            Validating service template...
            Done.

------------------------------------------------------------------------------------------------------------------------

.. _opera outputs:

outputs
#######

``opera outputs`` - print the outputs of the deploy/undeploy.

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
            :emphasize-lines: 7

            (venv) $ cd tosca/outputs
            (venv) tosca/outputs$ opera deploy service.yaml
            [Worker_0]   Deploying my_node_0
            [Worker_0]     Executing create on my_node_0
            [Worker_0]   Deployment of my_node_0 complete

            (venv) tosca/outputs$ opera outputs
            output_attr:
            description: Example of attribute output
            value: my_custom_attribute_value
            output_prop:
            description: Example of property output
            value: 123

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
        This included printing out the path to TOSCA service template entrypoint, extracted CSAR location, path to the
        storage inputs and status/state of the deployment.
        The output can be formatted in YAML or JSON. The created json object looks like this:

        .. code-block:: json

            {
                "service_template":  "string | null",
                "content_root":      "string | null",
                "inputs":            "string | null",
                "status":            "initialized | deployed | undeployed | interrupted | null"
            }

    .. tab:: Example

        Follow the next CLI instructions and results for the `misc-tosca-types-csar`_ example.

        .. code-block:: console
            :emphasize-lines: 2, 12, 34, 56, 84

            (venv) $ cd csars/misc-tosca-types
            (venv) csars/misc-tosca-types$ opera info
            content_root: null
            inputs: null
            service_template: null
            status: null

            (venv) csars/misc-tosca-types$ opera init -i inputs.yaml service.yaml
            Warning: 'opera init' command is deprecated and will probably be removed within one of the next releases. Please use 'opera deploy' to initialize and deploy service templates or compressed CSARs.
            Service template was initialized

            (venv) csars/misc-tosca-types$ opera info
            content_root: null
            inputs: /home/user/Desktop/xopera-examples/csars/misc-tosca-types/.opera/inputs
            service_template: service.yaml
            status: initialized

            (venv) csars/misc-tosca-types$ opera deploy
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

            (venv) csars/misc-tosca-types$ opera info

            content_root: null
            inputs: /home/user/Desktop/xopera-examples/csars/misc-tosca-types/.opera/inputs
            service_template: service.yaml
            status: interrupted

            (venv) csars/misc-tosca-types$ opera deploy -r -f
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

            (venv) csars/misc-tosca-types$ opera info

            content_root: null
            inputs: /home/user/Desktop/xopera-examples/csars/misc-tosca-types/.opera/inputs
            service_template: service.yaml
            status: deployed

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

            content_root: null
            inputs: /home/user/Desktop/xopera-examples/csars/misc-tosca-types/.opera/inputs
            service_template: service.yaml
            status: undeployed

------------------------------------------------------------------------------------------------------------------------

.. _opera package:

package
#######

``opera package`` - package TOSCA YAML templates and their accompanying files to a compressed TOSCA CSAR.

.. tabs::

    .. tab:: Usage

        .. argparse::
            :module: opera.cli
            :func: create_parser
            :prog: opera
            :path: package

    .. tab:: Description

        The ``opera package`` command is used to create a valid TOSCA CSAR - a deployable zip (or tar) compressed
        archive file.
        TOSCA CSARs contain the blueprint of the application that we want to deploy.
        The process includes packaging together the TOSCA service template and all the accompanying files.

        In general, ``opera package`` receives a directory (where user's TOSCA templates and other files are located)
        and produces a compressed CSAR file.
        The command can create the CSAR if there is at least one TOSCA YAML file in the input folder.
        If the CSAR structure is already present (if `TOSCA-Metadata/TOSCA.meta` exists and all other TOSCA CSAR
        constraints are satisfied) the CSAR is created without an additional temporary directory.
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

``opera unpackage`` - uncompress TOSCA CSAR.

.. tabs::

    .. tab:: Usage

        .. argparse::
            :module: opera.cli
            :func: create_parser
            :prog: opera
            :path: unpackage

    .. tab:: Description

        The ``opera unpackage`` has the opposite function of the ``opera package`` command.
        It  serves for unpacking (i.e. validating and extracting) the compressed TOSCA CSAR files.
        The opera unpackage command receives a compressed CSAR as a positional argument.
        It then validates and extracts the CSAR to a given location.

        There's no ``--format/-f`` option. Rather than that, the compressed file format (that will be used to extract
        the CSAR) is determined automatically.
        Currently, the compressed CSARs can be supplied in two different compression formats - `zip` or `tar`.

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
        With ``opera notify`` command, the user can inform the orchestrator about the changes (e.g. CPU load has
        increased) and the orchestrator will invoke the operations that are needed to make necessary actions (e.g.
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
        (usually a JSON file) that includes changes (e.g. metrics from monitoring tool) that will be exposed to TOSCA
        interfaces as ``notification`` variable (for example in Ansible playbooks you can use Jinja2
        ``{{ notification }}`` template to retrieve and parse the notification file contents).

    .. tab:: Example

        With ``opera notify`` and by empowering the orchestrator with the practical usage of TOSCA policies and
        triggers we wanted to enable scaling and other similar use cases that are based on policies and triggers.
        Many applications and services (e.g. AWS Lambda, Docker containers, Kubernetes solutions etc.) that are
        deployed with xOpera orchestrator often include the configuration of monitoring tool (e.g. Prometheus) that is
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

.. _opera init:

init (deprecated)
#################

``opera init`` - initialize TOSCA CSAR or service template.

.. tabs::

    .. tab:: Usage

        .. argparse::
            :module: opera.cli
            :func: create_parser
            :prog: opera
            :path: init

    .. tab:: Description

        The deprecated ``opera init`` command is used to initialize the deployment.
        It either takes a TOSCA template file or a compressed (zipped CSAR) file (and an optional YAML file with
        inputs).

        When the compressed CSAR is provided to the ``opera init`` command it is then validated to be sure that the
        CSAR is compliant with TOSCA.

        After the initialization the following files and folders will be created in your opera storage directory (by
        default that is ``.opera`` and can be changed using the ``--instance-path`` flag):

        - ``root_file`` file - contains the path to the service template or CSAR
        - ``inputs`` file - JSON file with the supplied inputs
        - ``csars`` folder contains the extracted copy of your CSAR (created only if you deployed the compressed TOSCA
          CSAR)

        After running ``opera init`` you will be able to initiate the deployment process using just the
        ``opera deploy`` command without any positional arguments (however, you can still supply inputs or override
        TOSCA service template/CSAR).

        .. deprecated:: 0.6.1

    .. tab:: Example

        Follow the next CLI instructions and results for the `misc-tosca-types-csar`_ example.

        .. code-block:: console
            :emphasize-lines: 2

            (venv) $ cd csars/misc-tosca-types
            (venv) csars/misc-tosca-types$ opera init -i inputs.yaml service.yaml
            Warning: 'opera init' command is deprecated and will probably be removed within one of the next releases. Please use 'opera deploy' to initialize and deploy service templates or compressed CSARs.
            Service template was initialized

.. note::

    The ``opera init`` command is deprecated (since version *0.6.1*) and will probably be removed within one of the
    next releases.
    Please use ``opera deploy`` to initialize and deploy service templates or compressed CSARs.

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

    Be very careful with your orchestration secrets (such as SSH private keys, cloud credentials, passwords ans so on)
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

    _shtab_opera_options_='-h --help -s --shell-completion'
    _shtab_opera_commands_='deploy diff info init outputs package undeploy unpackage update validate'

    _shtab_opera_deploy='-h --help --instance-path -p --inputs -i --workers -w --resume -r --clean-state -c --force -f --verbose -v'
    _shtab_opera_deploy_COMPGEN=_shtab_compgen_files
    ...

    # print out completion script for zsh shell
    (venv) $ opera -s zsh
    #compdef opera

    # AUTOMATCALLY GENERATED by `shtab`

    _shtab_opera_options_=(
    "(- :)"{-h,--help}"[show this help message and exit]"
    {-s,--shell-completion}"[Generate tab completion script for your shell]:shell_completion:(bash zsh)"
    )

    _shtab_opera_commands_() {
    local _commands=(
    "deploy:"
    "diff:"
    "info:"
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

.. _CLI video:

=====
Video
=====

This video will help you to get started with xOpera.
It also shows an example of deploying a simple image resize solution to AWS Lambda:

.. raw:: html

    <div style="text-align: center; margin-bottom: 2em;">
    <iframe width="100%" height="350" src="https://www.youtube.com/embed/cb1efi3wnpw" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
    </div>


.. _PyPI: https://pypi.org/project/opera/
.. _opera issues: https://github.com/xlab-si/xopera-opera/issues
.. _TOSCA interface operations: https://docs.oasis-open.org/tosca/TOSCA-Simple-Profile-YAML/v1.3/cos01/TOSCA-Simple-Profile-YAML-v1.3-cos01.html#_Toc26969470
.. _misc-tosca-types-csar: https://github.com/xlab-si/xopera-examples/tree/master/csars/misc-tosca-types
.. _small-csar: https://github.com/xlab-si/xopera-examples/tree/master/csars/small
.. _hello-world: https://github.com/xlab-si/xopera-examples/tree/master/misc/hello-world
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
