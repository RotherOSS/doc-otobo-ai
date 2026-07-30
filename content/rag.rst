Retrieval Augmented Generation (RAG)
====================================

The RAG system allows you to automatically generate answers to customer requests based on the information available in your OTOBO.
This document details the configuration and usage of the RAG system.


Workflow
--------

At first, all tickets, FAQ and external documentation are **imported** in the RAG system.
During this step, the information is scrubbed, chunked, and sent to a small LLM to obtain a numeric vector that describes the presented information.
This procedure is called *embedding*.
Every vector describes a point in high dimensional space.
Similar information will be close to each other, while unrelated information will be far apart.
All the vectors and documents are stored in the ``otobo-ai-services``.
This allows for fast selection of documents that address the same topic.

If a new article is created, **generate an answer**:

    #. An embedding is requested for the new article.
       The resulting vector is used to identify related documents.
    #. The related documents are injected into the prompt for a larger LLM to generate an answer based on the provided information.
    #. The generated answer is stored in a Dynamic Field at the article.
    #. The dynamic field can be referenced in an Answer Template.
    #. If the user chooses this Answer Template, the generated answer of the LLM will be visible in the compose window.


Setup
-----

OTOBO-AI supports multiple RAG definitions, each with its own input data selection, prompt and output queue.
While the RAG definition is stored in the ``otobo-ai-services`` docker compose project, input and output is configured in the OTOBO package.
These setups are called *pipelines*.
You may have multiple separate pipelines, for example one for each department or product you are supporting.
Pipelines allow for complete separation of concerns.

::

  ┌───────────────┐                          ┌─────────┐
  │ ticket search ┼───┐                  ┌───► queue 1 │
  └───────────────┘   │                  │   └─────────┘
     ┌────────────┐   │    ┌────────┐    │   ┌─────────┐
     │ faq search ┼───┼────►  RAG   ┼────┼───► queue 2 │
     └────────────┘   │    └───▲────┘    │   └─────────┘
     ┌────────────┐   │    ┌───┼────┐    │   ┌─────────┐
     │ doc folder ┼───┘    │ prompt │    └───►    ...  │
     └────────────┘        └────────┘        └─────────┘
        INPUT                                  OUTPUT


To set up such a pipeline, you need to create a RAG definition.

RAG Definition
~~~~~~~~~~~~~~

Under ``/opt/otobo-ai-services/rag_examples``, example RAG definitions are provided.
As a basic template the ``default`` RAG definition is provided.
This RAG definition supports Tickets (t), FAQ (f) and Documentation (d).
Copy the RAG description to your RAG definition folder:

.. code-block:: bash

   cp -r rags_examples/default rags


All RAG definitions placed here are exposed at the web service.
You may tune it to your liking (see :doc:`tuning`), or create a new one!
You *should* at least tailor the ``prompts/rag_prompt.txt`` to your use case.
Template text is provided in this file.

.. note::

   Docker mounts the local ``./rags`` directory into the container as ``src/rags``, enabling external customization.
   However, a restart of the container is required for the changes to take effect.


Input & Output Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Next, you need to configure the input and output of the RAG definition in the OTOBO package.
Navigate to the system configuration.
For the first RAG definition, the following settings need to be configured:

``OTOBOAI::Document::SearchRestrictions###0100-Default``
   Location of the documentation to be imported into the RAG system.

``OTOBOAI::FAQ::SearchRestrictions###0100-Default``
   FAQ search restrictions to select the FAQ articles to be imported into the RAG system.

``OTOBOAI::Ticket::SearchRestrictions###0100-Default``
   Ticket search restrictions to select the tickets to be imported into the RAG system.

Search restrictions are able to use any field and restriction from the classic search function (refer to the User Guide for details).
You may use the same configuration suffixed ``...###0200-Custom1`` for a second RAG definition, and so on.

To configure which pipeline generate answers for a queue, configure the system setting ``OTOBOAI::QueueToRAGMapping``.

.. note::

   Only text information is indexed by the RAG system and used for answer generation.
   No documents, pictures, or attachments will be processed.


Initial Data Import
~~~~~~~~~~~~~~~~~~~

Place documentation under the path configured in the system setting ``OTOBOAI::Document::SearchRestrictions`` relative to your OTOBO installation.

.. note::

   `Pandoc <https://pandoc.org>`_ may be helpful with data conversion.
   Beware of index files, they match on every topic and are relatively dense.
   Beware of files like a license, they are never a match.

And import all the data from command line:

.. code-block:: bash

   # run --help for an overview
   # importing may take some time depending on the data size and LLM throughput
   bin/otobo.Console.pl Maint::AI::Import


It is advisable to exclude a portion of the data from import.
The import command supports the parameters ``--sample-nth`` and ``--sample``.
A generated sample may serve as evaluation set for tuning (see :doc:`tuning`).

You may verify the import using the command line from the ``rag`` container:

.. code-block:: bash

    docker compose exec rag python src/cli.py


Incremental Data Ingestion
~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``OTOBO-AI`` package supports the automatic synchronization of newly added or updated tickets and FAQ items via incremental ingestion.
More precisely, OTOBO listens to a pre-defined set of Ticket and FAQ related *Events*, which in turn re-query the ``SearchRestrictions`` when triggered.

This means that the configuration of ``OTOBOAI::Document::SearchRestrictions`` and ``Ticket::EventModulePost###8300-OTOBOAIIncrementalSynchronization`` are closely coupled.
For correct incremental ingestion, please make sure that your specified *Events* fully cover your ``SearchRestrictions``, per data source.

For a list of ticket related events, refer to the Developer Manual.
The event configuration supports regular expressions, as seen in the default setting:

::

    TicketCreate|Ticket.*Update|TicketMerge|TicketDelete|ArticleCreate|ArticleUpdate

The FAQ event configuration is done by ``FAQ::EventModulePost###8300-OTOBOAIIncrementalSynchronization``.
Supported events are covered by the default setting:

::

    FAQCreate|FAQUpdate|FAQDelete

.. attention::

   On OTOBO 11.0 the FAQ event configuration relies on the optional ``Elasticsearch-FAQ`` package to provide event handling.
   For 11.1 and later, the FAQ event configuration is supported by default.



Answer Generation
-----------------

You may generate answers either prompted by a customer request or manually on-demand.


On-Demand Answers
~~~~~~~~~~~~~~~~~

There are two options to obtain on-demand answers, either the command line or a new widget in the agent overview.
The widget is served by the default RAG.
If you handle sensitive information with the default RAG, add a ``Group`` to the setting ``DashboardBackend###0160-OTOBOAIDashboard`` to restrict access to the widget.

The command line may be reached from the command line interface of OTOBO:

.. code-block:: bash

    docker compose exec web ./bin/otobo.Console.pl Maint::AI::GetAnswer "Hello Roboto!"



Automatic Answer Suggestions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

However, OTOBO may query the RAG for an answer suggestion whenever a new article is created.
Upon retrieval, the answer is stored in a dynamic field, configured in ``OTOBOAI::DynamicFieldOTOBOAI``.
It may be referenced from an answer template.
The pre-configured answer template is ``OTOBOAI Answer Template``.
Assign it to the queues you want to offer RAG answer suggestions to the agents.
Refer to the `Admin Manual <https://doc.otobo.org/manual/admin/11.1/en/content/administration-area/ticket-settings/templates-queues.html>`_ for more information.
If an agent chooses this answer template, the AI answer is revealed, may be edited by the agent and sent out to the customer.

.. note::

   You may also assign the dynamic field to the ``AgentTicketZoom`` to display the answer at the article and have it accessible without the need to lock the ticket.








