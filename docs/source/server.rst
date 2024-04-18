###################################
PACER Server
###################################

.. _server overview:

This is the documentation for the server components of PACER. The server is responsible for managing a request from the PACER client, maintaining the query strategy and logic for interfacing to the local FHIR server, and responding with the collected ECR format. On installation; the server is pointed towards a singular endpoint for a FHIR interface. Using Clinical Quality Language(CQL) scripts and some custom handling, PACER retrieves related FHIR resources, and maps to the local ECR json format.

PACER consists of: a JobManagementSystem for handling requests, a ResultsManager for handling FHIR results, a CQL Execution Service for executing CQL, a CQLStorage service for maintaining CQL scripts, a "FHIR Filter" service for redacting senstive data, and a Translation Service for translating local codesets into standardized codes.

.. image:: server_fig/PACER_Architecture.png


.. _server installation:

Server Installation Instructions
============
PACER is a dockerized-project. It is preferable to build each component as a docker image and deploy with the Docker-Compose tool. For more information refer to

* Docker-CE (Community Edition). Refer to https://docs.docker.com/install/ for installation instructions
* Docker-Compose, version 1.3 or later. Refer to https://docs.docker.com/compose/install/ for installation instructions.

Pacer consists of 2 seperate docker-compose files: ``docker-compose-db.yml`` and ``docker-compose-apps.yml``.
* ``docker-compose-db.yml`` contains the configuration information for the database, and should be built and ran first before the docker-compose-apps.yml script.
* ``docker-compose-apps.yml`` consist of the application components that manage the PACER workflow.

To build and deploy the compose project follow these steps from the command line.
1. Review and confirm configration of the yaml files before installation. Refer to :ref:`configruation below <server configuration>` for more detailed instructions
2. Build the database and application projects

.. code-block:: console
docker-compose -f docker-compose-db.yml build
docker-compose -f docker-compose-apps.yml

This will build deployable images for 6 different containers.

* ``cql-storage``
* ``cql-execution``
* ``db``
* ``fhir-filter``
* ``job-management-system``
* ``results-manager``

3. Deploy dockerfiles in stages using compose

.. code-block:: console
docker-compose -f docker-compose-db.yml up -d #wait for the database service to complete it's initdb.script
docker-compose -f docker-compose-apps.yml up -d

This will turn on all application components and enable networking for the PACER application to recieve requests.

.. _server configuration:

Configuration
=============
<Configuration After Installation Instructions>

.. _server FHIR Server:

External FHIR Server
====================
<Information regarding FHIR server>

.. _server Job Management System:

Job Management System
=====================

Job Management System: Overview
-------------------------------
<ECR Manager Information here>

Job Management System: API Documentation
----------------------------------------
<ECR Manager API Review here>

.. _server Results Manager:

Results Manager
===============

Results Manager: Overview
-------------------------
<Results Manager Information here>

Results Manager: API Documentation
----------------------------------
<Results Manager API Review here>

.. _server CQL Storage:

CQL Storage
============

CQL Storage: Overview
---------------------
<CQL Storage Information here>

CQL Storage: API Documentation
------------------------------
<CQL Storage API Review here>

.. _server CQL Execution Service:

CQL Execution Service
=====================

CQL Execution Service: Overview
-------------------------------
<CQL Execution Service here>

CQL Execution Service: API Documentation
----------------------------------------
<CQL Execution Service API Review here>

.. _server FHIR Filter:

FHIR Filter
===========

FHIR Filter: Overview
---------------------
<FHIR Filter here>

FHIR Filter: API Documentation
------------------------------
<FHIR Filter API Review here>

.. _server Translate Concept Service:

Translate Concept Service
=========================

Translate Concept Service: Overview
-----------------------------------
<Translate Concept Service here>

Translate Concept Service: API Documentation
--------------------------------------------
<Translate Concept Service API Review here>