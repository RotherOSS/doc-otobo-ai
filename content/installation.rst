Installation
============

This chapter describes the installation of OTOBO AI.
It guides you through the setup process for both the services and the OTOBO package.
For further configuration, refer to the specific use case.

There are two components to OTOBO AI:

#. The services ``otobo-ai-services``.
#. The ``otobo-ai`` OTOBO package.

These installation instructions illustrate their installation.


Requirements
------------

In order to setup the OTOBO AI solution, you need to meet the following requirements:

#. The host you run the OTOBO AI services on needs 20GB free disk space for tooling itself, as well as free disk space to hold your OTOBO plain text data.
   Further, you need at least 2GB RAM and 2 spare CPU cores.
   Requirements grow with the number of active Agents and Article creation volume.
#. You need connection to at least two LLMs by OpenAI compatible API.
   For the RAG you need an embedding model and a generation model.
   For the chat feature you need a generation model with tool calling capability.



``otobo-ai-services`` Docker Compose Stack
------------------------------------------

The ``otobo-ai-services`` are provided as self-contained ``docker compose`` stack.
There are two docker containers:

``rag``
    A Chroma database for vector data and webservice to interface with OTOBO
``postgres``
    A database holding prepared data


The repository may be found here: https://github.com/RotherOSS/otobo-ai-services

First, set up the docker compose project:

.. code-block:: bash

    cd /opt
    git clone https://github.com/RotherOSS/otobo-ai-services.git
    cd otobo-ai-services

Create a ``.env`` file in the root directory to configure environment variables:

.. code-block:: bash

    cp .docker_compose_env_ai .env

Edit the ``.env`` file to set your desired configuration options.

As a minimum, configure these settings:
   ``OTOBO_AI_API_KEY`` = *your API key*

   ``OTOBO_AI_LLM_API_KEY`` = *your API key*

   ``OTOBO_AI_LLM_HOST`` = *https://roboto.otobo.io:4000*

   ``OTOBO_MCP_PUBLIC_URL`` = *https://your-otobo-host>/otobo/index.pl*

Use Docker Compose to run the configured services:

.. code-block:: bash

   docker compose up --detach



``OTOBO-AI`` Package
--------------------

Refer here to the the `Retrieval Augmented Generation (RAG) <https://doc.otobo.org/manual/ai/11.1/en/content/rag.html#retrieval-augmented-generation-rag>`__ documentation.

The package can simply be installed via the OTOBO Package Manager on OTOBO version ``>= 11.0``.
See the `OTOBO Admin Guide: Package Manager <https://doc.otobo.org/manual/admin/11.0/en/content/administration-area/administration/package-manager.html>`_ for details on package management in OTOBO.

Configure the package settings in the system configuration.
See the `OTOBO Admin Guide: System Configuration <https://doc.otobo.org/manual/admin/11.0/en/content/administration-area/administration/system-configuration.html>`_ for general instructions.
The package manual includes all relevant information.
You may just search ``OTOBOAI::`` to find all relevant settings.
These are the basic steps:

#. Select data for import into the RAG stack (what your answers are based on).
#. Configure the auth token into the webservice.
#. Setup the answer template.

Continue depending on your use case with :doc:`chat` or :doc:`rag`.
