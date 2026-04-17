Installation
============

There are two components that interface OTOBO with an LLM:

#. The services ``otobo-ai-services``.
#. The ``otobo-ai`` OTOBO package.

These installation instructions illustrate their setup.


``otobo-ai-services`` Docker Compose Stack
------------------------------------------

The ``otobo-ai-services`` are provided as self-contained ``docker compose`` stack.
There are two docker containers:

``otobo-ai``
    A Chroma database for vector data and webservice to interface with OTOBO
``postgres``
    A database holding prepared data


The repository may be found here: https://github.com/RotherOSS/otobo-ai-services

First, set up the docker compose project:

.. code-block:: bash

    cd /opt
    git clone git@github.com:RotherOSS/otobo-ai.git
    cd otobo-ai

Example RAG definitions are provided under ``rag_examples``.
The ``simple_rag`` is for standalone development.
If you use this setup with OTOBO, choose the ``tfd_rag1``.
It supports Tickets (t), FAQ (f) and Documentation (d).
Copy the RAG description to your RAG definition folder:

.. code-block:: bash

   cp -r rags_examples/tfd_rag1 rags


All RAG definitions placed here are exposed at the web service.

.. note::
   Docker mounts the local ``./rags`` directory into the container as ``src/rags``, enabling external customization.
   However, a restart of the container is required for the changes to take effect.

You may tune it to your liking (see :doc:`tuning`), or create a new one!
You *should* at least tailor the prompt to your use case.

Create a ``.env`` file in the root directory to configure environment variables:

.. code-block:: bash

    cp .docker_compose_env_ai .env

Edit the ``.env`` file to set your desired configuration options.

Use Docker Compose to build and run the server:

.. code-block:: bash

   docker compose up --build --detach



``OTOBO-AI`` Package
--------------------

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

Continue to :doc:`usage`.
