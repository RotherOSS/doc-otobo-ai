Tuning
======

Every OTOBO is different.
And so is the information processed by it.
To account for this fact, this setup is as flexible as possible.
There are a `lot` of options when tuning this setup.
If you would like to dig into the answer generation process, it is recommended to hook a `Langfuse <https://www.langfuse.com/>`_ to your RAG setup.
This chapter addresses the various tuning options available.

Prompt
------

Prompt templates can be adjusted very freely.
They are placed at the rags folder in the folders of the ``otobo-ai-services``.
There are two prompts, one for answer generation, which is discussed here, and one for answer evaluation, which is discussed below.
They have significant impact on the form of the generated answer.

A prompt is composed of a *system* part.
It allows you to define a general setting and role for the LLM to act in.
It is introduced by:

::

    <|begin_of_text|><|start_header_id|>system<|end_header_id|>

It is followed by the *user* part.
This part is composed of the *context*, a specific task and the *question*, injected to the prompt by ``{question}``, to answer.
It is introduced by:

::

    <|eot_id|><|start_header_id|>user<|end_header_id|>

The context is composed by these tags: ``{faqs}``, ``{docs}``, ``{ticket_data}``, and ``{ticket_chunks}``.
Tickets are presented by question-answer-pairs (first customer article in and last agent article out) as well as chunks.
During import, data is cut into overlapping chunks.
This facilitates topical specificity within the chunks.
However, on close inspection of the prompts, one may find the chunks too small or too large.
**Chunk size** might be fit by setting the variables ``OTOBO_AI_EMBEDDING_CHUNK_SIZE`` and ``OTOBO_AI_EMBEDDING_CHUNK_OVERLAP`` in your ``.env``.

Given a question, the most relevant pieces by type are selected.
However, this list has to be cut off.
Only the most ``n`` relevant hits are considered.
Please refer to the ``graph.py`` of your rag definition.

The format of the external documentation may also be considered.
Different data format might be processed differently.

The prompt closes with a tag for the answer:

::

    <|eot_id|><|start_header_id|>assistant<|end_header_id|>


Models
------

There are two models at play, one that provides the embeddings and hence is responsible for the decision on what information is deemed most relevant to the question at hand.
The other one is for answer generation and extracts an answer from the presented documents and presents it in the form requested in the prompt.
Both models have to be capable of handling the language(s) you want to process.
They may be configured in the ``.env`` file of the ``otobo-ai-services``.

The main characteristics of the embedding models, aside from benchmarks, are *context size* and *dimensions*.
The bigger the information pieces to process are, the bigger the *context size* needs to be.
With an increase in nuances in the whole information pool, the *dimensions* need to increase to account for the differences.

At the time of writing, it is generally recommended to use an instruction-tuned model in order to obtain answers faithful to the prompt given.
Furthermore, a multi-modal model should be considered.
Please keep in mind, that this model has to support a quite large context window because it needs to consume all relevant documents at once.

The model itself may be tuned by *temperature*, which is the models risk affinity to the possibility to hallucinate.
It may be specified in the ``.env``.
In order to obtain faithful answers, lower values are recommended.

.. note::
   At the time of writing, images are not considered for processing.



Evaluation
----------

In order to evaluate the impact of changes to the quality of system responses, Roboto facilitates scoring of answers generated for an *evaluation set*.
An evaluation set is a portion of the ticket data, randomly chosen and deliberately not imported (see :doc:`rag`).
You may generate a *sample* during import like this:

.. code-block:: bash

    bin/otobo.console.pl Maint::AI::Import --sample-nth 20 --sample ~/var/ai/sample.json

Sample size should depend on the amount of tickets to import, but should range from 20% (hard validation) to 5% (plausibility check).
This evaluation set might even be excluded from reimport by passing the ``--reference`` argument.
It seems reasonable to keep a sample file at ``~/var/ai/sample.json``.
If the evaluation set is ever imported, evaluation will be structurally biased and hence useless.

In order to evaluate Roboto's performance on an evaluation set, answers are generated for the tickets in the evaluation set and these answers are evaluated with respect to a set of freely chosen criteria.
These evaluation criteria are defined in the ``eval_prompt.txt`` of your RAG definition.
The difference to the RAG prompt is the data description and format definition as the task.

.. note::

   The model chosen for evaluation needs to follow prompt instructions very closely.

Run the evaluation like this:

.. code-block:: bash

    bin/otobo.console.pl Maint::AI::Scoring \
      --sample /opt/otobo/var/ai/sample.json \
      --result /opt/otobo/var/ai/score-baseline.csv \
      --json /opt/otobo/var/ai/score-baseline.json

You may use ``jq`` and a spreadsheet application for analysis.
