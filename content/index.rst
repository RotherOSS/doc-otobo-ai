.. image:: ../images/otobo-logo.png
   :align: center

===============
OTOBO AI Manual
===============

This is the OTOBO AI Manual.
It serves as a reference to help administrators manage and configure the OTOBO AI capabilities effectively.

.. hint::

   OTOBO AI is currently in closed beta.
   If interested, feel free to `contact us <mailto:hallo@otobo.io>`_.

.. warning::
   This documentation is for the **11.1 beta** version of OTOBO.
   It may contain incomplete or inaccurate information, and some features may not be fully functional.
   Please use this documentation with caution and report any issues to the OTOBO development team.
   You may find the **documentation for the latest stable release** at https://doc.otobo.org/manual/ai/11.0/en/content/index.html.

Overview
--------

OTOBO AI implements two main features:

#. A RAG (Retrieval-Augmented Generation) system which is able to generate response suggestions for tickets based on the data from within your OTOBO.
#. A chat system that allows you to operate your OTOBO in natural language.

There are two components that interface OTOBO with an LLM:

#. The ``otobo-ai-services``.
#. The ``otobo-ai`` OTOBO package.

The OTOBO package is the interface between OTOBO and the services.
It also integrates the required configuration to start a chat session with the LLM (Large Language Model).
The services hold the data in a format that allows finding similar data related to a request efficiently, and interface with the LLM.
The chat is accompanied by a MCP (Model Context Protocol) server that allows a LLM to access your data in OTOBO.
Both features may be used independently, but they can also be combined to provide a more powerful experience.
Langfuse may be used to monitor the requests and responses of the LLM, and to provide insights into the performance of the system.
The following diagram illustrates the architecture of OTOBO AI:

.. raw:: html

   <style>
  .centered-code {
       display: flex;
       justify-content: center;
   }
   .centered-code pre {
       display: table;
       margin: 0 auto;
   }
   </style>

.. rst-class:: centered-code

.. code-block:: none

                                 ┌────┐    ┌╌╌╌╌╌╌╌╌╌╌┐
              ┌───────────────┐  │ DB ├─┐ ┌┤ Langfuse ┆
              │ OTOBO         │  └────┘ │ │└╌╌╌╌╌╌╌╌╌╌┘
              │   ┌──────────┐│     ┌───┴─┴───┐   ┌───────┐
              │   │ otobo-ai ├┼─────┤   RAG   ├───┤  LLM  │
              │   └──────────┘│     └─────────┘   └───┬───┘
              └───────┬───────┘ ┌─────────┐ ┌──────┐  │
                      └─────────┤   MCP   ├─┤ CHAT ├──┘
                                └─────────┘ └──────┘




.. toctree::
   :maxdepth: 2
   :caption: Contents

   installation
   rag
   chat
   tuning



License
-------

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

