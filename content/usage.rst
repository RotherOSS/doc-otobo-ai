Usage
=====

In order for Roboto to generate answers, you need to import the relevant data.
After import answers to articles may be generated and the database may be queried individually.


Data Import
-----------

With the ``OTOBO-AI`` package installed, three data sources may be configured: tickets, FAQ and documents.
Tickets and FAQs import selection is guided by their search functions, i.e., you may use any field and restriction from the `classic search function <https://doc.otobo.org/manual/user/11.0/en/content/agent/search/search.html>`_.
Place documentation under the path configured in the system setting ``OTOBOAI::Document::SearchRestrictions`` relative to your OTOBO installation.
And import all the data from command line:


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


