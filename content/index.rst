.. image:: ../images/otobo-logo.png
   :align: center

===============
OTOBO AI Manual
===============

This is the OTOBO AI (codename "Roboto") Manual.
It serves as a reference to help administrators manage and configure the OTOBO AI capabilities effectively.

.. hint::
   Roboto is currently in closed beta.
   If interested, feel free to `contact us <mailto:hallo@otobo.io>`_.


Overview
--------

*Roboto implements a RAG (Retrieval-Augmented Generation) system i.e.,
it is able to generate response suggestions for tickets based on the data from within your OTOBO.*

There are two components that interface OTOBO with an LLM:

#. The services ``otobo-ai-services``.
#. The ``otobo-ai`` OTOBO package.

The OTOBO package is the interface between OTOBO and the services.
The services hold the data in a format that allows finding similar data related to a request efficiently, and interface with the LLM.

::

                            ┌╌╌╌╌╌╌╌╌╌╌╌╌┐
      ┌───────────────┐     ┆  Langfuse  ┆
      │ OTOBO         │     └╌╌╌╌╌┬╌╌╌╌╌╌┘
      │   ┌──────────┐│  ┌────────┴──────────┐  ┌───────┐
      │   │ otobo-ai─┼│──┼ otobo-ai-services ┼──┼  LLM  │
      │   └──────────┘│  └───────────────────┘  └───────┘
      └───────────────┘

Workflow
^^^^^^^^

At first, all tickets, FAQ and external documentation are **imported**.
During this step, the information is scrubbed, chunked, and sent to a small LLM to obtain a numeric vector that describes the presented information.
This procedure is called embedding.
Every vector describes a point in high dimensional space.
Similar information will be close to each other.
All the vectors and documents are stored in the otobo-ai-services.
This allows for fast selection of documents that address the same topic.


If a new article is created, **generate an answer**:

    #. An embedding is requested for the new article.
       The resulting vector is used to identify related documents.
    #. The related documents are injected into the prompt for a larger LLM to generate an answer based on the provided information.
    #. The generated answer is stored in a Dynamic Field at the article.
    #. The dynamic field can be referenced in an Answer Template.
    #. If the user chooses this Answer Template, the generated answer of the LLM will be visible in the compose window.




.. toctree::
   :maxdepth: 2
   :caption: Contents

   installation
   usage
   tuning


This work is copyrighted by ROTHER OSS GmbH (https://otobo.io),
Oberwalting 31, 94339 Leiblfing, Germany

Terms and Conditions Rother OSS:
Permission is granted to copy, distribute and/or modify this document under the
terms of the GNU Free Documentation License, Version 1.3 or any later version
published by the Free Software Foundation; with no Invariant Sections, no
Front-Cover Texts, and no Back-Cover Texts.
A copy of the license is included in the section entitled "COPYING".

Published by: Rother OSS GmbH, (https://otobo.de),
Oberwalting 31, 94339 Leiblfing, Germany.

Authors: Rother OSS GmbH (https://otobo.de).

