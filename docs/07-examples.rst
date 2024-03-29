.. _Examples:

********
Examples
********

In this section we show some simple examples of orchestrating with xOpera.
All the examples are available in `opera-examples repository`_.

.. _Examples private cloud:

=============
Private cloud
=============

The xOpera orchestration tool enables setting up and deployment within private cloud providers such as OpenStack or
OpenFaaS.

.. _Examples IaaS:

IaaS
####

This part contains Infrastructure as a Service (IaaS) examples for private cloud providers.

.. _Examples OpenStack client setup:

OpenStack client setup
----------------------

This subsection describes `nginx-openstack`_ example, which shows how to deploy and set up an OpenStack VM and an nginx
site on top of it.

Because using OpenStack modules from Ansible playbooks is quite common, we can install ``opera`` with all required
OpenStack libraries by running:

.. code-block:: console

    (.venv) $ pip install -U opera[openstack]

Before we can actually use the OpenStack functionality, we also need to obtain the OpenStack credentials.
If we log into OpenStack and navigate to the :guilabel:`Access & Security` -> :guilabel:`API Access` page, we can
download the rc file with all required information.

At the start of each session (e.g., when we open a new command line console), we must source the rc file by running:

.. code-block:: console

    (venv) $ . openstack.rc

After we enter the password, we are ready to start using the OpenStack modules in playbooks that implement life cycle
operations.

After the deployment the sample HTML website will be available on `<YOUR_VM_IP>:80`, so make sure that you unlock the
(ingress) port ``80`` within the specified OpenStack security group.

To deploy and undeploy the example follow the steps below.

.. code-block:: console

    (venv) $ cd misc/nginx-openstack
    (venv) misc/nginx-openstack$ opera deploy -i inputs.yaml service.yaml
    [Worker_0]   Deploying vm_0
    [Worker_0]     Executing create on vm_0
    [Worker_0]   Deployment of vm_0 complete
    [Worker_0]   Deploying nginx_0
    [Worker_0]     Executing create on nginx_0
    [Worker_0]     Executing post_configure_target on site_0--nginx_0
    [Worker_0]   Deployment of nginx_0 complete
    [Worker_0]   Deploying site_0
    [Worker_0]     Executing create on site_0
    [Worker_0]   Deployment of site_0 complete

    (venv) misc/nginx-openstack$ opera undeploy
    [Worker_0]   Undeploying site_0
    [Worker_0]     Executing delete on site_0
    [Worker_0]   Undeployment of site_0 complete
    [Worker_0]   Undeploying nginx_0
    [Worker_0]     Executing delete on nginx_0
    [Worker_0]   Undeployment of nginx_0 complete
    [Worker_0]   Undeploying vm_0
    [Worker_0]     Executing delete on vm_0
    [Worker_0]   Undeployment of vm_0 complete

.. _Examples private cloud FaaS:

FaaS
####

This part contains Function as a Service (FaaS) and Serverless examples for private cloud providers.

To get you started with cloud deployment we prepared a simple thumbnail generator application for different platforms
which also supports the idea of Serverless computing in FaaS environments.
The main functionality of this solution is the image resize functionality.
To create thumbnails the source image must be uploaded into input bucket and then three thumbnails will be created and
saved to another output bucket.

.. _Examples OpenFaaS:

OpenFaaS
--------

The `openfaas-thumbnail-generator`_ solution is made of two parts. Since OpenFaaS can be treated as a sort of private
cloud you have to set it up on a remote VM first.
The second part with image-resize functionality makes use of OpenFaaS functions.

To deploy the application follow the steps below or navigate to `opera-examples repository`_.

.. code-block:: console

    # install necessary prerequisites
    python3 -m venv .venv
    source .venv/bin/activate
    pip install --upgrade pip
    pip install git+https://github.com/ansible/ansible.git@devel
    pip install opera

    # configure MinIO credentials (here we use default example access and secret key)
    echo '{ "minio_access_key": "AKIAIOSFODNN7EXAMPLE", "minio_secret_key": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY" }' > /tmp/credentials.json

    # modify vars in inputs.yaml and run ansible playbook build.yaml it to build and pack docker image with image-resize function (or use prepared tar in examples)
    cd docker-image-templates
    ansible-playbook build.yaml

    # run deployment with with xOpera (here you should prepare a new VM on OpenStack and configure it to use passwordless ssh)
    OPERA_SSH_USER=root opera deploy -i inputs.yaml service.yaml

.. warning::

    If you want to deploy on a remote VM you should use ``OPERA_SSH_USER`` env var to tell xOpera as which user you
    want to connect.

.. _Examples public cloud:

============
Public cloud
============

xOpera and Ansible open up a great possibility to deploy solutions on different public cloud providers such as
Amazon Web Services (AWS), Microsoft Azure (Azure) or Google Cloud Platform (GCP).

.. _Public cloud FaaS:

FaaS
####

This part contains Function as a Service (FaaS) and Serverless examples for public cloud providers.

To get you started with cloud deployment we prepared a simple thumbnail generator application for different cloud
platforms which also supports the idea of Serverless computing in FaaS environments.
The main functionality of this solution is the image resize functionality.
To create thumbnails the source image must be uploaded into input bucket and then three thumbnails will be created and
saved to another output bucket.

.. _Examples Amazon Web Services (AWS):

Amazon Web Services (AWS)
-------------------------

The `aws-thumbnail-generator`_ solution uses AWS role, AWS S3 bucket, AWS Lambda and AWS API Gateway cloud resources.
To deploy the application follow the steps below or navigate to `opera-examples repository`_.

.. code-block:: console

    # install AWS CLI v2
    curl "https://d1vvhvl2y92vvt.cloudfront.net/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    unzip awscliv2.zip
    sudo ./aws/install

    # configure your account with your aws credentials (access key, secret key, region)
    aws configure

    # initialize virtualenv with Python 3.6 and install prerequisites
    cd aws
    python3.6 -m venv .venv
    . .venv/bin/activate
    pip install --upgrade pip
    pip install ansible
    pip install opera

    # run xOpera service (make sure to setup inputs in inputs.yaml)
    opera deploy -i inputs.yaml resize_service_opera_v1_3.yaml

    # you can also run the following command to undeploy the solution:
    opera undeploy

    # if you want to test the API Gateway solution run (make sure to modify inputs in inputs-api.yaml):
    opera deploy -i inputs-api-gateway.yaml service-api-gateway.yaml

.. _Examples Microsoft Azure (Azure):

Microsoft Azure (Azure)
-----------------------

The `azure-thumbnail-generator`_ solution uses Azure Resource Group, Azure Storage Account, Azure Containers and Azure
Function App cloud resources.
To deploy the application follow the steps below or navigate to `opera-examples repository`_.

.. code-block:: console

    # install Azure CLI and try to login in your account
    curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
    az login

    # install prerequisites
    cd azure
    python3 -m venv .venv
    source .venv/bin/activate
    pip install --upgrade pip
    pip install ansible
    pip install opera

    # setup Azure Function Tools
    wget -q https://packages.microsoft.com/config/ubuntu/19.04/packages-microsoft-prod.deb
    sudo dpkg -i packages-microsoft-prod.deb
    sudo apt-get update
    sudo apt-get install azure-functions-core-tools
    rm packages-microsoft-prod.deb

    # run xOpera service (make sure to setup the right inputs in yaml file)
    opera deploy -i inputs.yaml service.yaml

.. _Examples Google Cloud Platform (GCP):

Google Cloud Platform (GCP)
---------------------------

The `gcp-thumbnail-generator`_ solution uses GCP Storage Buckets and GCP Functions cloud resources.
To deploy the application follow the steps below or navigate to `opera-examples repository`_.

.. code-block:: bash

    # install Google Cloud SDK from https://cloud.google.com/sdk/docs/downloads-apt-get with apt-get
    echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
    sudo apt-get install apt-transport-https ca-certificates gnupg
    curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg add -
    sudo apt-get update && sudo apt-get install google-cloud-sdk

    # create GCP service account, create a JSON file key and put to /tmp folder
    cat /tmp/service_account.json

    # install prerequisites
    cd gcp
    python3 -m venv .venv
    source .venv/bin/activate
    pip install --upgrade pip
    pip install opera

    # run xOpera service (don't forget to set the appropriate inputs in inputs.yaml)
    opera deploy -i inputs.yaml service.yaml

.. _Examples Connection of cloud platforms:

Connection of cloud platforms
-----------------------------

To show that opera can establish a connection between two cloud platforms, we created a simple
`aws-azure-platform-connection`_. example that connects Azure and AWS cloud providers.
Moreover, there are two examples that implement data flow from Azure to AWS and vice versa (shown in
figure :numref:`aws_azure_connection`).
The main functionality of this solution is to sequentially execute two operations on images which are image-watermark
and image-resize on two different platforms (Azure and AWS).
Orchestration creates two containers on Azure and two buckets on AWS.
Images are passed from container to bucket using AWS Lambda or Azure Function.

.. _aws_azure_connection:

.. figure:: /images/platform-connection.png
    :target: _images/platform-connection.png
    :width: 95%
    :align: center

    The two examples of AWS<->Azure connection.

.. hint::

    Within your Python functions you can use Python package module called `object-store`_ that generalizes manipulation
    with different object store types like AWS S3, MinIO or Azure Containers. Source code and usage is also explained
    in detail on GitHub in `python-object-store-library`_ repository.

.. _Examples screencast video:

Screencast video
################

This video will help you to get started with xOpera.
It also shows an example of deploying a simple image resize solution to AWS Lambda:

.. raw:: html

    <div style="text-align: center; margin-bottom: 2em;">
    <iframe width="100%" height="350" src="https://www.youtube.com/embed/cb1efi3wnpw" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
    </div>

.. _Examples Docker and Kubernetes:

=====================
Docker and Kubernetes
=====================

The ``opera`` orchestrator is capable of deploying blueprints that use Docker and Kubernetes.
The following examples show some deployments to get you started.

.. _Examples Docker and MinIO:

Docker and MinIO
################

xOpera orchestrator is able to run Docker containers by using the proper Ansible modules in the Ansible playbooks.
The `docker`_ example was made to show how the installation of the Docker service and running Docker containers can be
done with ``opera``, TOSCA and Ansible.

To deploy and undeploy the example follow the steps below.

.. code-block:: console

    (venv) $ cd kubernetes/docker
    (venv) kubernetes/docker$ opera deploy -i inputs.yaml service.yaml
    [Worker_0]   Deploying my-workstation_0
    [Worker_0]   Deployment of my-workstation_0 complete
    [Worker_0]   Deploying docker_0
    [Worker_0]     Executing create on docker_0
    [Worker_0]   Deployment of docker_0 complete
    [Worker_0]   Deploying minio_0
    [Worker_0]     Executing create on minio_0
    [Worker_0]   Deployment of minio_0 complete

    (venv) kubernetes/docker$ opera undeploy
    [Worker_0]   Undeploying docker_0
    [Worker_0]   Undeployment of docker_0 complete
    [Worker_0]     Executing delete on docker_0
    [Worker_0]   Undeploying minio_0
    [Worker_0]   Undeployment of minio_0 complete
    [Worker_0]     Executing delete on minio_0
    [Worker_0]   Undeploying my-workstation_0

The example will install Docker on a target machine and will run `MinIO object storage`_ that will be accessible on
``localhost:9000`` where you can login wit the credentials you specified in ``inputs.yaml``.

.. _Examples Kubernetes with Rancher:

Kubernetes with Rancher
#######################

Kubernetes can be deployed using `Rancher platform`_, which is the open-source multi-cluster orchestration platform.
The `rancher`_ example is can be used to deploy Rancher Docker container that will set up Kubernetes which can be
accessed through the Rancher dashboard, where you can create your account (see :numref:`rancher_kubernetes_login`) and
use cluster explorer (see :numref:`rancher_kubernetes_cluster_explorer`) to do manipulate with Kubernetes.

.. _rancher_kubernetes_login:

.. figure:: /images/rancher-kubernetes-login.png
    :target: _images/rancher-kubernetes-login.png
    :width: 80%
    :align: center

    Log in to Rancher Kubernetes dashboard.

To deploy the example follow the steps below.

.. code-block:: console

    (venv) $ cd kubernetes/rancher
    (venv) kubernetes/rancher opera deploy -i inputs.yaml service.yaml
    [Worker_0]   Deploying my-workstation_0
    [Worker_0]   Deployment of my-workstation_0 complete
    [Worker_0]   Deploying rancher_0
    [Worker_0]     Executing create on rancher_0
    [Worker_0]   Deployment of rancher_0 complete
    [Worker_0]   Deploying prometheus-helm-chart_0
    [Worker_0]     Executing create on prometheus-helm-chart_0
    [Worker_0]   Deployment of prometheus-helm-chart_0 complete

    (venv) kubernetes/rancher$ opera undeploy
    [Worker_0]   Undeploying prometheus-helm-chart_0
    [Worker_0]     Executing delete on prometheus-helm-chart_0
    [Worker_0]   Undeployment of prometheus-helm-chart_0 complete
    [Worker_0]   Undeploying rancher_0
    [Worker_0]     Executing delete on rancher_0
    [Worker_0]   Undeployment of rancher_0 complete
    [Worker_0]   Undeploying my-workstation_0
    [Worker_0]   Undeployment of my-workstation_0 complete

After running the example the Rancher dashboard will be accessible on ``localhost:80`` and ``localhost:443``.
Additionally we deploy the `Prometheus helm chart`_ to be able to monitor the Kubernetes cluster.

.. _rancher-kubernetes-dashboard:

.. figure:: /images/rancher-kubernetes-dashboard.png
    :target: _images/rancher-kubernetes-dashboard.png
    :width: 80%
    :align: center

    Rancher Kubernetes dashboard.

.. _rancher_kubernetes_cluster_explorer:

.. figure:: /images/rancher-kubernetes-cluster-explorer.png
    :target: _images/rancher-kubernetes-cluster-explorer.png
    :width: 90%
    :align: center

    Kubernetes cluster explorer in Rancher dashboard.

.. _Examples HPC:

===
HPC
===

.. _xopera_hpc_logo:

.. figure:: /images/xopera-hpc-mark-black-text-mid.png
    :target: _images/xopera-hpc-mark-black-text-mid.png
    :width: 40%
    :align: center

.. note::

   *TBD*: This part of the documentation will be improved in the future.

.. _Examples TOSCA CSARs:

===========
TOSCA CSARs
===========

xOpera orchestrator can effectively validate, initialize, deploy and undeploy compressed `TOSCA CSAR <https://www.oasis-open.org/committees/download.php/46057/CSAR%20V0-1.docx>`_
files which represent the main orchestration packages, containing TOSCA templates, their implementations
(e.g., Ansible playbooks) and all the other accompanying files that are needed for the deployment.

.. _CExamples SAR without TOSCA.meta file:

CSAR without TOSCA.meta file
############################

The following example shows a deployment of the compressed TOSCA CSAR containing different TOSCA entities
(extracted version is available here: `small-csar`_).
This is a type of a TOSCA CSAR that doesn't contain a separate ``TOSCA-Metadata/TOSCA-meta`` file for metadata but has
metadata specified within the TOSCA service template itself, which may be more convenient for a small TOSCA CSAR.
The special thing about this CSAR is also that it uses JSON inputs file instead of YAML inputs file which also makes it
smaller.

The result of the example consisting of deploy and outputs operations is shown below.

.. code-block:: console

    (venv) $ cd csars/small
    # you can also zip all files without inputs.json in csars/small to small.csar
    # compressed CSAR can be deployed with: opera deploy -i inputs.json small.csar
    (venv) csars/small$ opera deploy -i inputs.json service.yaml
    opera deploy -i inputs.json service.yaml
    [Worker_0]   Deploying my_workstation_0
    [Worker_0]     Executing pre_configure_target on test_node_0--my_workstation_0
    [Worker_0]     Executing post_configure_target on test_node_0--my_workstation_0
    [Worker_0]   Deployment of my_workstation_0 complete
    [Worker_0]   Deploying test_node_0
    [Worker_0]     Executing create on test_node_0
    [Worker_0]     Executing pre_configure_source on test_node_0--my_workstation_0
    [Worker_0]     Executing post_configure_source on test_node_0--my_workstation_0
    [Worker_0]   Deployment of test_node_0 complete

    (venv) csars/small$ opera outputs
    output_node_attribute:
      description: Node attribute output
      value: Node attribute
    output_post_configure_source_attribute:
      description: Relationship attribute output
      value: Relationship attribute
    output_post_configure_source_input_attribute:
      description: Relationship attribute output
      value: Hey, I am in relationship!
    output_post_configure_source_property_attribute:
      description: Relationship attribute output
      value: Relationship property
    output_post_configure_source_txt_file_attribute:
      description: Relationship attribute output
      value: This is an example file content.
    output_post_configure_target_attribute:
      description: Relationship attribute output
      value: This is post configure target attribute
    output_pre_configure_source_attribute:
      description: Relationship attribute output
      value: This is pre configure source attribute
    output_pre_configure_target_attribute:
      description: Relationship attribute output
      value: This is pre configure target attribute
    output_relationship_attribute:
      description: Relationship attribute output
      value: Relationship attribute
    output_relationship_input:
      description: Relationship input output
      value: Hey, I am in relationship!
    output_relationship_property:
      description: Relationship property output
      value: Relationship property

.. _Examples CSAR with TOSCA.meta file:

CSAR with TOSCA.meta file
#########################

The next example shows a deployment of the compressed TOSCA CSAR containing different TOSCA entities
(extracted version is available here: `misc-tosca-types-csar`_).
This TOSCA CSAR uses ``TOSCA-Metadata/TOSCA-meta`` file for specifying the orchestration metadata.

The result of the example consisting of deploy, outputs and undeploy operations is shown below.

.. code-block:: console

    (venv) $ cd csars/misc-tosca-types
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

    (venv) csars/misc-tosca-types$ opera outputs
    node_output_attr:
      description: Example of attribute output
      value: my_custom_attribute_value
    node_output_prop:
      description: Example of property output
      value: 123
    relationship_output_attr:
      description: Example of attribute output
      value: rel_attr_test123
    relationship_output_prop:
      description: Example of attribute output
      value: rel_prop_test123

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

.. hint::

    You don't need to initialize the CSAR with before the deployment anymore.
    The ``opera init`` command is deprecated since ``opera deploy`` can be used directly with both service templates
    and compressed CSARs.

.. _Examples more templates and blueprints:

=============================
More templates and blueprints
=============================

The following video shows how xOpera examples can be used with different xOpera tools.

.. raw:: html

    <div style="text-align: center; margin-bottom: 2em;">
    <iframe width="100%" height="350" src="https://www.youtube.com/embed/NZLYWB9uxjk" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
    </div>

More examples are available in `opera-examples repository`_.
Below is a table that lists all the currently available xOpera examples and their purpose.

.. _TOSCA examples:

TOSCA examples
##############

+--------------------------------------------+-----------------------------------------------------------------+
| Example name and link                      | Purpose                                                         |
+============================================+=================================================================+
| `artifacts`_                               | TOSCA artifacts                                                 |
+--------------------------------------------+-----------------------------------------------------------------+
| `attribute-mapping`_                       | TOSCA attribute mapping                                         |
+--------------------------------------------+-----------------------------------------------------------------+
| `capability-attributes-properties`_        | TOSCA attributes and properties for capabilities                |
+--------------------------------------------+-----------------------------------------------------------------+
| `intrinsic-functions`_                     | Intrinsic TOSCA functions                                       |
+--------------------------------------------+-----------------------------------------------------------------+
| `outputs`_                                 | TOSCA outputs                                                   |
+--------------------------------------------+-----------------------------------------------------------------+
| `policy-triggers`_                         | TOSCA policies and triggers                                     |
+--------------------------------------------+-----------------------------------------------------------------+
| `relationship-outputs`_                    | TOSCA outputs for relationships                                 |
+--------------------------------------------+-----------------------------------------------------------------+

.. _artifacts: https://github.com/xlab-si/xopera-examples/tree/master/tosca/artifacts
.. _attribute-mapping: https://github.com/xlab-si/xopera-examples/tree/master/tosca/attribute-mapping
.. _capability-attributes-properties: https://github.com/xlab-si/xopera-examples/tree/master/tosca/capability-attributes-properties
.. _intrinsic-functions: https://github.com/xlab-si/xopera-examples/tree/master/tosca/intrinsic-functions
.. _outputs: https://github.com/xlab-si/xopera-examples/tree/master/tosca/outputs
.. _policy-triggers: https://github.com/xlab-si/xopera-examples/tree/master/tosca/policy-triggers
.. _relationship-outputs: https://github.com/xlab-si/xopera-examples/tree/master/tosca/relationship-outputs

.. _TOSCA CSAR examples:

TOSCA CSAR examples
###################

+--------------------------------------------+-----------------------------------------------------------------+
| Example name and link                      | Purpose                                                         |
+============================================+=================================================================+
| `misc-tosca-types-csar`_                   | TOSCA CSAR containing a lot of TOSCA entities                   |
+--------------------------------------------+-----------------------------------------------------------------+
| `small-csar`_                              | A minimal TOSCA CSAR example                                    |
+--------------------------------------------+-----------------------------------------------------------------+

.. _misc-tosca-types-csar: https://github.com/xlab-si/xopera-examples/tree/master/csars/misc-tosca-types
.. _small-csar: https://github.com/xlab-si/xopera-examples/tree/master/csars/small

.. _Cloud examples:

Cloud examples
##############

+---------------------------------------------+-----------------------------------------------------------------+
| Example name and link                       | Purpose                                                         |
+=============================================+=================================================================+
| `aws-thumbnail-generator`_                  | FaaS thumbnail generator blueprint for AWS                      |
+---------------------------------------------+-----------------------------------------------------------------+
| `aws-thumbnail-generator-with-api-gateway`_ | FaaS thumbnail generator blueprint for AWS with API Gateway     |
+---------------------------------------------+-----------------------------------------------------------------+
| `aws-thumbnail-generator-with-vm`_          | FaaS thumbnail generator blueprint for AWS with EC2 VM          |
+---------------------------------------------+-----------------------------------------------------------------+
| `azure-thumbnail-generator`_                | FaaS thumbnail generator blueprint for Azure                    |
+---------------------------------------------+-----------------------------------------------------------------+
| `gcp-thumbnail-generator`_                  | FaaS thumbnail generator blueprint for GCP                      |
+---------------------------------------------+-----------------------------------------------------------------+
| `openfaas-thumbnail-generator`_             | FaaS thumbnail generator blueprint for OpenFaaS                 |
+---------------------------------------------+-----------------------------------------------------------------+
|| `aws-azure-platform-connection`_           || FaaS solution that connects AWS and Azure cloud providers      |
||                                            || with image-resize and image-watermark functionalities          |
+---------------------------------------------+-----------------------------------------------------------------+

.. _aws-thumbnail-generator: https://github.com/xlab-si/xopera-examples/tree/master/cloud/aws/thumbnail-generator
.. _aws-thumbnail-generator-with-api-gateway: https://github.com/xlab-si/xopera-examples/tree/master/cloud/aws/thumbnail-generator-with-api-gateway
.. _aws-thumbnail-generator-with-vm: https://github.com/xlab-si/xopera-examples/tree/master/cloud/aws/thumbnail-generator-with-vm
.. _azure-thumbnail-generator: https://github.com/xlab-si/xopera-examples/tree/master/cloud/azure/thumbnail-generator
.. _gcp-thumbnail-generator: https://github.com/xlab-si/xopera-examples/tree/master/cloud/gcp/thumbnail-generator
.. _openfaas-thumbnail-generator: https://github.com/xlab-si/xopera-examples/tree/master/cloud/openfaas/thumbnail-generator
.. _aws-azure-platform-connection: https://github.com/xlab-si/xopera-examples/tree/master/cloud/platform-connection/aws-azure-connection

.. _Kubernetes examples:

Kubernetes examples
###################

+--------------------------------------------+-----------------------------------------------------------------+
| Example name and link                      | Purpose                                                         |
+============================================+=================================================================+
| `docker`_                                  | Install Docker and run a Docker container on a target machine   |
+--------------------------------------------+-----------------------------------------------------------------+
| `rancher`_                                 | Run Kubernetes service using Rancher Docker container           |
+--------------------------------------------+-----------------------------------------------------------------+

.. _docker: https://github.com/xlab-si/xopera-examples/tree/master/kubernetes/docker
.. _rancher: https://github.com/xlab-si/xopera-examples/tree/master/kubernetes/rancher

.. _Other miscellaneous examples:

Other miscellaneous examples
############################

+--------------------------------------------+-----------------------------------------------------------------+
| Example name and link                      | Purpose                                                         |
+============================================+=================================================================+
| `compare-templates`_                       | Compare and redeploy/update TOSCA templates and instances       |
+--------------------------------------------+-----------------------------------------------------------------+
| `concurrency`_                             | Use workers for concurrent deployment of TOSCA nodes            |
+--------------------------------------------+-----------------------------------------------------------------+
| `hello-world`_                             | The very first and minimal hello world xOpera example           |
+--------------------------------------------+-----------------------------------------------------------------+
| `nginx-openstack`_                         | Deploy nginx site on top of OpenStack VM                        |
+--------------------------------------------+-----------------------------------------------------------------+
| `scaling`_                                 | An example of AWS Lambda scaling with TOSCA policies            |
+--------------------------------------------+-----------------------------------------------------------------+
| `server-client`_                           | Connect server and client nodes with TOSCA relationships        |
+--------------------------------------------+-----------------------------------------------------------------+

.. _compare-templates: https://github.com/xlab-si/xopera-examples/tree/master/misc/compare-templates
.. _concurrency: https://github.com/xlab-si/xopera-examples/tree/master/misc/concurrency
.. _hello-world: https://github.com/xlab-si/xopera-examples/tree/master/misc/hello-world
.. _nginx-openstack: https://github.com/xlab-si/xopera-examples/tree/master/misc/nginx-openstack
.. _scaling: https://github.com/xlab-si/xopera-examples/tree/master/misc/scaling
.. _server-client: https://github.com/xlab-si/xopera-examples/tree/master/misc/server-client

.. _opera-examples repository: https://github.com/xlab-si/xopera-examples
.. _object-store: https://pypi.org/project/object-store
.. _python-object-store-library: https://github.com/xlab-si/python-object-store-library
.. _MinIO object storage: https://min.io
.. _Rancher platform: https://rancher.com
.. _Prometheus helm chart: https://artifacthub.io/packages/helm/prometheus-community/prometheus
