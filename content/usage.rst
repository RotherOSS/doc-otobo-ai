Usage
=====

In order for Roboto to generate answers, you need to import the relevant data.
After import, answers to articles may be generated and the database may be queried individually.
With the ``OTOBO-AI`` package installed, three data sources may be configured: Tickets, FAQs and documents.


Search Restictions
------------------

Tickets and FAQs import selection is guided by their search functions, i.e. you may use any field and restriction from the `classic search function <https://doc.otobo.org/manual/user/11.0/en/content/agent/search/search.html>`_.
Add the desired configurations to the system settings ``OTOBOAI::Ticket::SearchRestrictions`` and ``OTOBOAI::FAQ::SearchRestrictions``, respectively, as well as one or more labels under which the items should be imported into the database.

Feel free to configure multiple ``SearchRestrictions`` per data source in case different restrictions should map to different labels.  
Place documentation under the path configured in the system setting ``OTOBOAI::Document::SearchRestrictions``, relative to your OTOBO installation.


Data Import
-----------

Now you are ready to import all data sources from command line:

.. code-block:: bash

   # run --help for an overview
   bin/otobo.Console.pl Maint::AI::Import


.. note::

   Currently plain text documentation is preferred.
   `Pandoc <https://pandoc.org>`_ may be helpful with data conversion.
   Beware of index files, they match on every topic and are relatively dense.
   Beware of files like a license, they are never a match.


It is advisable to exclude a portion of the data from import.
The import command supports the parameters ``--sample-nth`` and ``--sample``.
A generated sample may serve as evaluation set for tuning (see :doc:`tuning`).

You may verify the import using the command line from the ``otobo-ai`` container:

.. code-block:: bash

    docker compose exec otobo-ai python src/cli.py

Please note that any change to the ``SearchRestrictions`` of a data source may warrant a full reimport of that source to guarantee a valid database state. The import command supports a ``‑‑reimport`` parameter for this situation.


Incremental Ingestion
---------------------

The ``OTOBO-AI`` package supports the automatic synchronization of newly added or updated tickets and FAQ items via incremental ingestion. More precisely, OTOBO listens to a pre-defined set of Ticket and FAQ related ``Events``, which in turn re-query the ``SearchRestrictions`` when triggered.

This means that e.g. the configuration of ``OTOBOAI::Document::SearchRestrictions`` and ``Ticket::EventModulePost###8300-OTOBOAIIncrementalSynchronization`` are, in this regard, closely coupled. For correct incremental ingestion, please make sure that your specified ``Events`` fully cover your ``SearchRestrictions``, per data source.

For a list of ticket related events, please see the `administrator documentation <>`_. The ``Event`` configuration supports regular expressions, as seen in the default setting:

::

    TicketCreate|Ticket.*Update|TicketMerge|TicketDelete|ArticleCreate|ArticleUpdate

The FAQ ``Event`` configuration, which can be found under ``FAQ::EventModulePost###8300-OTOBOAIIncrementalSynchronization``, currently relies on the optional ``Elasticsearch-FAQ`` package to provide event handling. The only to-date supported events are covered by the default setting:

::

    FAQCreate|FAQUpdate|FAQDelete


Answer Generation
-----------------

You may generate answers either prompted by a customer request or manually on-demand.

On-Demand Answers
^^^^^^^^^^^^^^^^^

There are two options to obtain on-demand answers, either the command line or a new widget in the agent overview.
The command line may be reached from the CLI of OTOBO:

.. code-block:: bash

    docker compose exec web ./bin/otobo.Console.pl Maint::AI::GetAnswer "Hello Roboto!"

This option is especially useful if service management requires deep domain knowledge that is not externalized or easily accessible.

Automatic Answer Suggestions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

However, OTOBO may query the LLM for an answer suggestion whenever it creates a new article.
Upon retrieval, the answer is stored in a dynamic field, configured in ``OTOBOAI::DynamicFieldOTOBOAI``.
It may be referenced from an answer template.
The pre-configured answer template is ``OTOBOAI Answer Template``, and might need to be tweaked to adhere to your use-case.
If an agent chooses this answer template, the LLM answer is revealed, may be edited by the agent and sent out to the customer.


