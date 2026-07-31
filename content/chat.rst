Chat
====

OTOBO may interface with an external chat system.
The connection is established via the OTOBO webservices and an MCP (Model Context Protocol) server.
The following diagram illustrates the connection between OTOBO, the external chat system, and the MCP server:

::

                       ┌───────┐
  link:         ┌──────┼ OTOBO ◄──────┐
   - session_id │      └───────┘      │  webservices
   - ...        │                     │
             ┌──▼───┐              ┌──┼──┐
             │ CHAT ┼──────────────► MCP │
             └──────┘   session_id └─────┘
                        [...]


The OTOBO-AI package generates a ``session_id`` for the current user and the IP if the MCP and allows to pass it as a link parameter.
OTOBO keeps ``session_ids`` bound to IP addresses as per system configuration ``SessionCheckRemoteIP``.
Further, it allows to create a link to an external chat service and displays it beside every article.
It may pass the ``ticket_id`` and the ``article_id`` as link parameters to the external chat service.
This allows an agent to jump into an AI chat out of the ticket context and to pick up the context within the chat.
The chat calls the MCP server to interface with the OTOBO webservices, pre-configured by the OTOBO-AI package.


MCP Server
----------

The OTOBO MCP server serves as a proxy to the OTOBO webservices.
It relies on the ``session_id`` passed by the chat service to authenticate as the user at the OTOBO webservices.
This allows to keep the access management consistent with the configuration in OTOBO, i.e., a user may only access resources (e.g., tickets) which are accessible to them in OTOBO.

The MCP server is developed in its own repository `otobo-ai-mcp <https://github.com/RotherOSS/otobo-ai-mcp>`_.
However, its images are published to Docker Hub and may be used in the OTOBO Docker Compose setup.
Example configuration is provided in the ``docker-compose/otobo-ai_mcp.yml`` file of the ``otobo-ai-services``.
To activate the MCP server, you need to add the file to the ``COMPOSE_FILE`` variable in the ``.env`` file and fill out the required variables in the MCP labeled section.

The MCP server is not exposed to the outside by default.
It is meant to serve as an internal interface between the chat service and the OTOBO webservices.
If you want to expose the MCP server to the outside, you need to configure a reverse proxy in your OTOBO Docker Compose setup to handle SSL and strongly control access by a firewall.

.. note::

   If you use the MCP server inside a chat service, simply ask the LLM about the capabilities of the MCP server.


Chat Service
------------

The chat service is interchangeable.
However, it needs to be able to pick up the GET parameters, ``session_id`` and the different context ids provided by the OTOBO-AI package and place them to the chat context.
It is recommended to separate the chat sessions of different users.
Hence, the chat service should be able to handle user authentication.
Users may be managed by the chat service itself or by an external identity provider like LDAP or OIDC.

Within the ``otobo-ai-services`` `OpenWebUI <https://openwebui.com>`_ is provided as a reference implementation of a chat service.
You may find the Docker Compose file in the repository at ``docker-compose/otobo-ai_openwebui.yml``.
The service is pre-configured to share most of the variables declared in the ``.env`` (see: :doc:`installation`).
To activate the service you need to add the file to the ``COMPOSE_FILE`` variable in the ``.env`` file and fill out the required variables in the chat labeled section.

.. code-block:: bash

   # after changes to the .env file, reinitialize the Docker Compose setup:
   docker compose down && docker compose up --detach


We may use NGINX of your OTOBO Docker Compose setup to handle SSL and pose as a reverse proxy to the OpenWebUI service on port ``9000``.
To do so, you need to configure a custom override.
Navigate to your OTOBO Docker Compose project, by default in ``/opt/otobo-docker``.
Place a file named ``docker-compose/otobo-override-nginx-openwebui.yml`` in this folder:

.. code-block:: bash

      touch /opt/otobo-docker/docker-compose/otobo-override-nginx-openwebui.yml

Place the following content in the file:

.. code-block:: yaml

   services:
     nginx:
       ports:
         - "9000:9000"


This will expose the port ``9000`` of the NGINX container to the outside.
Further, you need to override the NGINX configuration template used to generate the configuration to serve OTOBO.
Please refer to the `OTOBO Installation Guide <https://doc.otobo.org/manual/installation/11.0/en/content/installation/installation-docker.html#custom-configuration-of-the-nginx-webproxy>`_ for details.
Add this block to the end of the exposed template:


.. code-block:: nginx

   server {
       listen 9000 ssl;
       listen [::]:9000 ssl;
       http2 on;

       include snippets/ssl-params.conf;
       ssl_certificate     ${OTOBO_NGINX_SSL_CERTIFICATE};
       ssl_certificate_key ${OTOBO_NGINX_SSL_CERTIFICATE_KEY};

       server_name  localhost;

       client_max_body_size 1G;

       location / {
           proxy_http_version 1.1;

           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection "upgrade";
           proxy_set_header Host $host;

           proxy_set_header X-Forwarded-Host $host:$server_port;
           proxy_set_header X-Forwarded-Server $host;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
           proxy_pass       http://chat:8080/;

           proxy_read_timeout 300s;
           proxy_send_timeout 300s;
       }
   }

Now we need to add the file ``otobo-override-nginx-openwebui.yml`` to the end of the ``COMPOSE_FILE`` variable in the ``.env`` file.
Then we may reinitialize the NGINX container to apply the changes:

.. code-block:: bash

   docker compose down nginx && docker compose up --detach


After start, you may access the OpenWebUI service at ``https://<your-otobo-host>:9000``.
For user management setup, see the `documentation of OpenWebUI <https://docs.openwebui.com/features/authentication-access/>`_.
Using an OIDC service that is able to handle Single Sign-On (SSO) provides the most seamless experience for the user.

Model configuration heavily depends on the LLM API used (see also the `OpenWebUI documentation <https://docs.openwebui.com/features/workspace/models>`_).
At the time of writing, the best result is achieved, by:

#. Setting the model to be used to ``public`` so it can be freely accessed.
#. Set a system prompt that refers to the MCP tool but does not respond with a welcome message in the first response.
   Even in multiple scenarios we were unable to get the LLM to use the MCP from the system prompt.
#. You should uncheck everything besides ``file context``, ``status updates``, the built-in tool ``memory``, and the OTOBO MCP.
   This will provide you with a simple setup; you can expand it as needed.


.. note::

   `LibreChat <https://www.librechat.ai>`_ may also be used as a chat service.
   While it is more capable out-of-the-box, to handle a larger amount of users, it is more complex to maintain.






