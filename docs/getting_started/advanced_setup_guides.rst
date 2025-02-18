.. role:: raw-html-m2r(raw)
   :format: html

.. _advanced-setup-guides:

Advanced setup guides
=====================

Here we provide some advanced setup guides, in case you want to use docker, configure your own Elasticsearch instance or install the cutting-edge master version.

.. _using-docker:

Using docker
------------

You can use vanilla docker to run our image of the server.
First, pull the image from the `Docker Hub <https://hub.docker.com/>`_:

.. code-block:: shell

   docker pull recognai/rubrix

Then simply run it.
Keep in mind that you need a running Elasticsearch instance for Rubrix to work.
By default, the Rubrix server will look for your Elasticsearch endpoint at ``http://localhost:9200``.
But you can customize this by setting the ``ELASTICSEARCH`` environment variable.

.. code-block:: shell

   docker run -p 6900:6900 -e "ELASTICSEARCH=<your-elasticsearch-endpoint>" --name rubrix recognai/rubrix

To find running instances of the Rubrix server, you can list all the running containers on your machine:

.. code-block:: shell

   docker ps

To stop the Rubrix server, just stop the container:

.. code-block:: shell

   docker stop rubrix

If you want to deploy your own Elasticsearch cluster via docker, we refer you to the excellent guide on the `Elasticsearch homepage <https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html>`_

.. _configure-elasticsearch-role-users:

Configure elasticsearch role/users
----------------------------------

If you have an Elasticsearch instance and want to share resources with other applications, you can easily configure it for Rubrix.

All you need to take into account is:


* Rubrix will create its ES indices with the following pattern ``.rubrix*``. It's recommended to create a new role (e.g., rubrix) and provide it with all privileges for this index pattern.

* Rubrix creates an index template for these indices, so you may provide related template privileges to this ES role.

Rubrix uses the ``ELASTICSEARCH`` environment variable to set the ES connection. 

You can provide the credentials using the following scheme: 

.. code-block:: bash

      http(s)://user:passwd@elastichost

Below you can see a screenshot for setting up a new *rubrix* Role and its permissions:

:raw-html-m2r:`<img src="https://user-images.githubusercontent.com/2518789/142883104-f4f20cf0-34a0-47ff-8ee3-ab9f4644271c.png"/>`


Deploy to aws instance using docker-machine
-------------------------------------------

Setup an AWS profile
^^^^^^^^^^^^^^^^^^^^

The ``aws`` command cli must be installed. Then, type:

.. code-block:: shell

   aws configure --profile rubrix

and follow command instructions. For more details, visit `AWS official documentation <https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html>`_

Once the profile is created (a new entry should be appear in file ``~/.aws/config``\ ), you can activate it via setting environment variable:

.. code-block:: shell

   export AWS_PROFILE=rubrix

Create docker machine (aws)
^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: shell

   docker-machine create --driver amazonec2 \
   --amazonec2-root-size 60 \
   --amazonec2-instance-type t2.large \
   --amazonec2-open-port 80 \
   --amazonec2-ami ami-0b541372 \
   --amazonec2-region eu-west-1 \
   rubrix-aws

Available ami depends on region. The provided ami is available for eu-west regions

Verify machine creation
^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: shell

   $>docker-machine ls

   NAME                   ACTIVE   DRIVER      STATE     URL                        SWARM   DOCKER     ERRORS
   rubrix-aws             -        amazonec2   Running   tcp://52.213.178.33:2376           v20.10.7

Save asigned machine ip
^^^^^^^^^^^^^^^^^^^^^^^

In our case, the assigned ip is ``52.213.178.33``

Connect to remote docker machine
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To enable the connection between the local docker client and the remote daemon, we must type following command:

.. code-block:: shell

   eval $(docker-machine env rubrix-aws)

Define a docker-compose.yaml
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: yaml

   # docker-compose.yaml
   version: "3"

   services:
     rubrix:
       image: recognai/rubrix
       ports:
         - "80:80"
       environment:
         ELASTICSEARCH: <elasticsearch-host_and_port>
       restart: unless-stopped

Pull image
^^^^^^^^^^

.. code-block:: shell

   docker-compose pull

Launch docker container
^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: shell

   docker-compose up -d

Accessing Rubrix
^^^^^^^^^^^^^^^^

In our case http://52.213.178.33




.. _install-from-master:

Install from master
-------------------

If you want the cutting-edge version of *Rubrix* with the latest changes and experimental features, follow the steps below in your terminal.
**Be aware that this version might be unstable!**

First, you need to install the master version of our python client:

.. code-block:: shell

    pip install -U git+https://github.com/recognai/rubrix.git

Then, the easiest way to get the master version of our web app up and running is via docker-compose:

.. code-block:: shell

    # get the docker-compose yaml file
    mkdir rubrix && cd rubrix
    wget -O docker-compose.yml https://raw.githubusercontent.com/recognai/rubrix/master/docker-compose.yaml
    # use the master image of the rubrix container instead of the latest
    sed -i 's/rubrix:latest/rubrix:master/' docker-compose.yml
    # start all services
    docker-compose up

If you want to use vanilla docker (and have your own Elasticsearch instance running), you can just use our master image:

.. code-block:: shell

    docker run -p 6900:6900 -e "ELASTICSEARCH=<your-elasticsearch-endpoint>" --name rubrix recognai/rubrix:master

If you want to execute the server code of the master branch manually, we refer you to our :ref:`development-setup`.
