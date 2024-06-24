###################################
PACER Server
###################################

.. _server overview:

This is the documentation for the server components of PACER. The server is responsible for managing a request from the PACER client, maintaining the query strategy and logic for interfacing to the local FHIR server, and responding with the collected ECR format. On installation; the server is pointed towards a singular endpoint for a FHIR interface. Using Clinical Quality Language(CQL) scripts and some custom handling, PACER retrieves related FHIR resources, and maps to the local ECR json format.

PACER consists of: a JobManagementSystem for handling requests, a ResultsManager for handling FHIR results, a CQL Execution Service for executing CQL, a CQLStorage service for maintaining CQL scripts, a "FHIR Filter" service for redacting senstive data, and a Translation Service for translating local codesets into standardized codes.

.. image:: server_fig/PACER_Architecture.png


.. _server installation:

Server Installation And Deploy Instructions
===========================================
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
    docker-compose -f docker-compose-apps.yml build

This will build deployable images for 6 different containers.

* ``cql-storage``
* ``cql-execution``
* ``db``
* ``fhir-filter``
* ``job-management-system``
* ``results-manager``

3. Deploy dockerfiles in stages using compose.

.. note::
    Again before deploying, it is recommended to refer to :ref:`server configruation below <server configuration>` to ensure connection to the FHIR server is correct

.. code-block:: console

    docker-compose -f docker-compose-db.yml up -d #wait for the database service to complete it's initdb.script
    docker-compose -f docker-compose-apps.yml up -d

This will turn on all application components and enable networking for the PACER application to recieve requests.

.. _server configuration:

Configuration
=============

Before the server can be deployed, configuration within the ``docker-compose-apps.yml`` file must be reviewed
The yml file describes confirugation for each of the above components. Confirugation is passed as envrionment variables into each container.

Configuration Requirements
--------------------------

    +  ``results-manager/CQL_EXECUTION_DATA_SERVICE`` This is a defined baseurl to the hosted fhir server to be used. FHIR url paths should end in ``/fhir/`` as the baseurl.
    +  ``results-manager/CQL_EXECUTION_DATA_USER`` If the FHIR server uses  `Basic Authentication <https://www.twilio.com/docs/glossary/what-is-basic-authentication>`_ for authentication; provide a username here.
    +  ``results-manager/CQL_EXECUTION_DATA_PASS`` If the FHIR server uses  `Basic Authentication <https://www.twilio.com/docs/glossary/what-is-basic-authentication>`_ for authentication; provide a password here.
    +  ``results-manager/CQL_EXECUTION_EPIC_CLIENT_ID`` If the FHIR server uses  `Bearer Token Authentication <https://swagger.io/docs/specification/authentication/bearer-authentication/>`_  generate a service token for PACER and apply the plaintext token string here.
    +  ``results-manager/CQL_EXECUTION_TERMINOLOGY_SERVICE`` If a terminology service is provided via FHIR, provide the service url. FHIR url paths should end in ``/fhir/`` as the baseurl.
    +  ``results-manager/CQL_EXECUTION_TERMINOLOGY_USER`` If the terminology server uses  `Basic Authentication <https://www.twilio.com/docs/glossary/what-is-basic-authentication>`_ for authentication; provide a username here.
    +  ``results-manager/CQL_EXECUTION_TERMINOLOGY_PASS`` If the terminology service uses  `Basic Authentication <https://www.twilio.com/docs/glossary/what-is-basic-authentication>`_ for authentication; provide a password here.
    + ``results-manager/CQL_EXECUTION_CODEMAPPER_SYSTEMS_MAP`` If the FHIR server uses local system definitions for common terminology systems (LOINC, Snomed, RxNorm), provide a key:value pair for the general url to the local system url provided. An example is provided in the comments for creating this key:value pair.

.. _server FHIR Server:

External FHIR Server
====================

The external FHIR server is the system managed FHIR server. In addition to read capabilities in each resource, It is expected to support these specific search capabilities

FHIR Server Requirements
------------------------

    + ``Patient``
        + ``?identifier`` Identifier search parameter is used to identify the patient.

    + ``Condition``
        + ``?patient`` Patient id related to the Condition
        + ``?category`` Code search with supported condition category types
        + ``?code:in`` Code search param used with :in modifier to provide large concept sets to be searched.

    + ``Encounter``
        + ``?patient`` Patient id related to the Encounter

    + ``Immmunization``
        + ``?patient`` Patient id related to the Immmunization

    + ``Medication``
        + ``?code:in`` Code search param used with :in modifier to provide large concept sets to be searched.

    + ``MedicationRequest``
        + ``?patient`` Patient id related to the MedicationRequest
        + ``code:in`` Code search param used with :in modifier to provide large concept sets to be searched.

    + ``Observation``
        + ``?patient`` Patient id related to the Observation
        + ``?category`` Code search with supported observation category types
        + ``code:in`` Code search param used with :in modifier to provide large concept sets to be searched.

.. warning::
    In cases where MedicationRequest search code is not available, a Medication Reference must be provided within the response json; in order to retrieved related medication data.

.. _server Job Management System:

Job Management System
=====================

Job Management System: Overview
-------------------------------
The JobManagementSystem component is the entry-point for the PACER-server. Responsible for negotiating the 
workflow for PACER-server, JMS manages the request to results amnager as well as an additional workflow
considerations. While the resultsmanager does most of the orchestration per patient, The Job Manager
System oversees the cohort of patients if a list of patients is submitted.

Job Management System: API Documentation
----------------------------------------
.. http:POST:: /JobManagementSystem/List

    **Example JMS Request**

    .. sourcecode:: http

        POST /JobManagementSystem/List/ HTTP/1.1
        Host: example.org
        Accept: */*
        Content-Type: application/json

    **Example JMS Response**

    .. sourcecode:: http

        HTTP/1.1 200 
        Vary: Origin
        Vary: Access-Control-Request-Method
        Vary: Access-Control-Request-Headers
        Content-Type: application/json
        Transfer-Encoding: chunked
        Date: Tue, 07 May 2024 14:47:26 GMT

        [
            {
                "Id": "4602",
                "Status": "A",
                "StatusLog": null,
                "Provider": [
                    {
                        "ID": {
                            "value": " GT|Reliable",
                            "type": "appfac"
                        },
                        "Name": "",
                        "Phone": "",
                        "Fax": "",
                        "Email": "",
                        "Facility": "",
                        "Address": "",
                        "Country": ""
                    },
                    {
                        "ID": {
                            "value": "P49430",
                            "type": "ORDPROVIDER"
                        },
                        "Name": "D ATKINSON",
                        "Phone": "",
                        "Fax": "",
                        "Email": "",
                        "Facility": "",
                        "Address": "",
                        "Country": ""
                    },
                    {
                        "ID": {
                            "value": "P49430",
                            "type": "ORDPROVIDER"
                        },
                        "Name": "John Duke",
                        "Phone": "",
                        "Fax": "",
                        "Email": "",
                        "Facility": "",
                        "Address": "",
                        "Country": ""
                    }
                ],
                "Facility": {
                    "ID": null,
                    "Name": "",
                    "Phone": "",
                    "Address": "",
                    "Fax": "",
                    "Hospital_Unit": ""
                },
                "Patient": {
                    "ID": [
                        {
                            "value": "2000",
                            "type": "urn:local:gtritest"
                        },
                        {
                            "value": "500000000",
                            "type": "SS"
                        },
                        {
                            "value": "82713",
                            "type": "urn:local:gtritest"
                        }
                    ],
                    "Name": {
                        "given": "SOPHIE82713",
                        "family": "STONE"
                    },
                    "Parents_Guardians": [],
                    "Street_Address": "2222 Home Street, Ann Arbor MI 99999",
                    "Birth_Date": "19750602",
                    "Sex": "M",
                    "PatientClass": "",
                    "Race": {
                        "Code": "",
                        "System": "",
                        "Display": ""
                    },
                    "Ethnicity": {
                        "Code": "",
                        "System": "",
                        "Display": ""
                    },
                    "Preferred_Language": {
                        "Code": "",
                        "System": "",
                        "Display": ""
                    },
                    "Occupation": "",
                    "Pregnant": false,
                    "Travel_History": [],
                    "Insurance_Type": {
                        "Code": "",
                        "System": "",
                        "Display": ""
                    },
                    "Immunization_History": [],
                    "Visit_DateTime": "",
                    "Admission_DateTime": "",
                    "Date_Of_Onset": "",
                    "Symptoms": [],
                    "Lab_Order_Code": [
                        {
                            "Code": "164200",
                            "System": "L",
                            "Display": "C. trachomatis - PCA",
                            "Date": "Fri Apr 29 17:01:00 EDT 2005",
                            "Laboratory_Results": [
                                {
                                    "Code": "164200",
                                    "System": "L",
                                    "Display": "C. trachomatis - PCA",
                                    "Date": "Tue May 03 15:32:00 EDT 2005",
                                    "Value": "Positive",
                                    "Unit": {
                                        "Code": "",
                                        "System": "",
                                        "Display": ""
                                    }
                                }
                            ],
                            "Facility": {
                                "ID": null,
                                "Name": "",
                                "Phone": "",
                                "Address": "",
                                "Fax": "",
                                "Hospital_Unit": ""
                            },
                            "Provider": {
                                "ID": {
                                    "value": "P49430",
                                    "type": "ORDPROVIDER"
                                },
                                "Name": "D ATKINSON",
                                "Phone": "",
                                "Fax": "",
                                "Email": "",
                                "Facility": "",
                                "Address": "",
                                "Country": ""
                            }
                        },
                        {
                            "Code": "164205",
                            "System": "L",
                            "Display": "N gonorrhoeae Competition Rflx",
                            "Date": "Fri Apr 29 17:01:00 EDT 2005",
                            "Laboratory_Results": [
                                {
                                    "Code": "164205",
                                    "System": "L",
                                    "Display": "N gonorrhoeae Competition Rflx",
                                    "Date": "Fri Apr 29 17:01:00 EDT 2005",
                                    "Value": "Negative",
                                    "Unit": {
                                        "Code": "",
                                        "System": "",
                                        "Display": ""
                                    }
                                },
                                {
                                    "Code": "164212",
                                    "System": "L",
                                    "Display": "N gonorrhoeae DNA Probe w/Rflx",
                                    "Date": "Fri Apr 29 17:01:00 EDT 2005",
                                    "Value": "See Reflex",
                                    "Unit": {
                                        "Code": "",
                                        "System": "",
                                        "Display": ""
                                    }
                                }
                            ],
                            "Facility": {
                                "ID": null,
                                "Name": "",
                                "Phone": "",
                                "Address": "",
                                "Fax": "",
                                "Hospital_Unit": ""
                            },
                            "Provider": {
                                "ID": {
                                    "value": "P49430",
                                    "type": "ORDPROVIDER"
                                },
                                "Name": "John Duke",
                                "Phone": "",
                                "Fax": "",
                                "Email": "",
                                "Facility": "",
                                "Address": "",
                                "Country": ""
                            }
                        }
                    ],
                    "Placer_Order_Code": "",
                    "Diagnosis": [],
                    "Medication Provided": [],
                    "Death_Date": "",
                    "Date_Discharged": "",
                    "Laboratory_Results": [],
                    "Trigger_Code": [],
                    "Lab_Tests_Performed": []
                },
                "Sending Application": "",
                "Notes": []
            }
        ]

    :<json string name: Assigned name of the job if client wishes to track job status asynchronously.
        Optional and will be autogenerate if not provided.
    :<json string jobType: Enumerated value describing the type of job and return to be completed.
        Currently, only one jobType is accepted: 'ECR'
    :<json string listType: Enumerated value describing the type of list reoccurence to be used.
        Currently supports 'SINGLE_USE' and 'PERIODIC'. SINGLE_USE for synchronous requests and PERIODIC
        for asynchronous and repeated work. 'SINGLE_USE' is default and recommended for most applications.
    :<jsonarr listElements: An array of patient elements which constitute the patients to be used in this job.
    :<json string listElements[x].referenceId: A pipe delimited (\|) set of system\|value identifiers which contains
        the patient identifier. This is an identifier which can be used directly upon the FHIR server to help identify
        the patient.
    :<json string listElements[x].name: Common name provided for the patient; helps to supplement the patient
        identification procedure.
    :<json string listElements[x].labOrderDate: A common string structured Date used to support the CQL process by
        determining relevant conditions, symptoms, and observations based upon the initial labOrderDtae provided
        by the Health Department.
    :resheader Content-Type: application/json
    :statuscode 200: no error


.. _server Results Manager:

Results Manager
===============

Results Manager: Overview
-------------------------
The ResultsManager oversees orchestration of the other components to complete an final ECR for the patient.
It manages interfacing to the CQL service, access of the main ECR.cql body, concept translations to local systems if provided,
and mapping CQL results to ECR fields.

Results Manager: API Documentation
----------------------------------

.. http:POST:: /ResultsManager/Case

    **Example Results Manager Request**

    .. sourcecode:: http

        POST /JobManagementSystem/List/ HTTP/1.1
        Host: example.org
        Accept: */*
        Content-Type: application/json

    **Example Response**

    .. sourcecode:: http

        HTTP/1.1 200 
        Vary: Origin
        Vary: Access-Control-Request-Method
        Vary: Access-Control-Request-Headers
        Content-Type: application/json
        Transfer-Encoding: chunked
        Date: Tue, 07 May 2024 14:47:26 GMT

        {
            "CQLs": "4602",
            "Status": "A",
            "StatusLog": null,
            "Provider": [
                {
                    "ID": {
                        "value": " GT|Reliable",
                        "type": "appfac"
                    },
                    "Name": "",
                    "Phone": "",
                    "Fax": "",
                    "Email": "",
                    "Facility": "",
                    "Address": "",
                    "Country": ""
                },
                {
                    "ID": {
                        "value": "P49430",
                        "type": "ORDPROVIDER"
                    },
                    "Name": "D ATKINSON",
                    "Phone": "",
                    "Fax": "",
                    "Email": "",
                    "Facility": "",
                    "Address": "",
                    "Country": ""
                },
                {
                    "ID": {
                        "value": "P49430",
                        "type": "ORDPROVIDER"
                    },
                    "Name": "John Duke",
                    "Phone": "",
                    "Fax": "",
                    "Email": "",
                    "Facility": "",
                    "Address": "",
                    "Country": ""
                }
            ],
            "Facility": {
                "ID": null,
                "Name": "",
                "Phone": "",
                "Address": "",
                "Fax": "",
                "Hospital_Unit": ""
            },
            "Patient": {
                "ID": [
                    {
                        "value": "2000",
                        "type": "urn:local:gtritest"
                    },
                    {
                        "value": "500000000",
                        "type": "SS"
                    },
                    {
                        "value": "82713",
                        "type": "urn:local:gtritest"
                    }
                ],
                "Name": {
                    "given": "SOPHIE82713",
                    "family": "STONE"
                },
                "Parents_Guardians": [],
                "Street_Address": "2222 Home Street, Ann Arbor MI 99999",
                "Birth_Date": "19750602",
                "Sex": "M",
                "PatientClass": "",
                "Race": {
                    "Code": "",
                    "System": "",
                    "Display": ""
                },
                "Ethnicity": {
                    "Code": "",
                    "System": "",
                    "Display": ""
                },
                "Preferred_Language": {
                    "Code": "",
                    "System": "",
                    "Display": ""
                },
                "Occupation": "",
                "Pregnant": false,
                "Travel_History": [],
                "Insurance_Type": {
                    "Code": "",
                    "System": "",
                    "Display": ""
                },
                "Immunization_History": [],
                "Visit_DateTime": "",
                "Admission_DateTime": "",
                "Date_Of_Onset": "",
                "Symptoms": [],
                "Lab_Order_Code": [
                    {
                        "Code": "164200",
                        "System": "L",
                        "Display": "C. trachomatis - PCA",
                        "Date": "Fri Apr 29 17:01:00 EDT 2005",
                        "Laboratory_Results": [
                            {
                                "Code": "164200",
                                "System": "L",
                                "Display": "C. trachomatis - PCA",
                                "Date": "Tue May 03 15:32:00 EDT 2005",
                                "Value": "Positive",
                                "Unit": {
                                    "Code": "",
                                    "System": "",
                                    "Display": ""
                                }
                            }
                        ],
                        "Facility": {
                            "ID": null,
                            "Name": "",
                            "Phone": "",
                            "Address": "",
                            "Fax": "",
                            "Hospital_Unit": ""
                        },
                        "Provider": {
                            "ID": {
                                "value": "P49430",
                                "type": "ORDPROVIDER"
                            },
                            "Name": "D ATKINSON",
                            "Phone": "",
                            "Fax": "",
                            "Email": "",
                            "Facility": "",
                            "Address": "",
                            "Country": ""
                        }
                    },
                    {
                        "Code": "164205",
                        "System": "L",
                        "Display": "N gonorrhoeae Competition Rflx",
                        "Date": "Fri Apr 29 17:01:00 EDT 2005",
                        "Laboratory_Results": [
                            {
                                "Code": "164205",
                                "System": "L",
                                "Display": "N gonorrhoeae Competition Rflx",
                                "Date": "Fri Apr 29 17:01:00 EDT 2005",
                                "Value": "Negative",
                                "Unit": {
                                    "Code": "",
                                    "System": "",
                                    "Display": ""
                                }
                            },
                            {
                                "Code": "164212",
                                "System": "L",
                                "Display": "N gonorrhoeae DNA Probe w/Rflx",
                                "Date": "Fri Apr 29 17:01:00 EDT 2005",
                                "Value": "See Reflex",
                                "Unit": {
                                    "Code": "",
                                    "System": "",
                                    "Display": ""
                                }
                            }
                        ],
                        "Facility": {
                            "ID": null,
                            "Name": "",
                            "Phone": "",
                            "Address": "",
                            "Fax": "",
                            "Hospital_Unit": ""
                        },
                        "Provider": {
                            "ID": {
                                "value": "P49430",
                                "type": "ORDPROVIDER"
                            },
                            "Name": "John Duke",
                            "Phone": "",
                            "Fax": "",
                            "Email": "",
                            "Facility": "",
                            "Address": "",
                            "Country": ""
                        }
                    }
                ],
                "Placer_Order_Code": "",
                "Diagnosis": [],
                "Medication Provided": [],
                "Death_Date": "",
                "Date_Discharged": "",
                "Laboratory_Results": [],
                "Trigger_Code": [],
                "Lab_Tests_Performed": []
            },
            "Sending Application": "",
            "Notes": []
        }

    :<query string firstName: first name of patient. Used as supporting information for identifying the patient.
    :<query string lastName: last name of patient. Used as supporting information for identifying the patient.
    :<query string identifier: A pipe delimited (\|) set of system\|value identifiers which contains
        the patient identifier. This is an identifier which can be used directly upon the FHIR server to help identify
        the patient.
    :<ecrId: Optional parameter for setting an id for the ECR record itself. No id will be returned if not provided.
    :<json string listElements[x].labOrderDate: A common string structured Date used to support the CQL process by
        determining relevant conditions, symptoms, and observations based upon the initial labOrderDtae provided
        by the Health Department.
    :resheader Content-Type: application/json
    :statuscode 200: no error

CQL Storage
============

CQL Storage: Overview
---------------------
CQLStorage service is responsible for hosting CQL files for retrieval during the PACER workflow. It is a simple storage service
using a simple blob schema within the PACER server database for holding files. Files are accessed by their title.

CQL Storage: API Documentation
------------------------------
.. http:GET:: /CQLStorage/CQL

    **Example CQL Storage Request**

    .. sourcecode:: http

        GET /CQLStorage/CQL/ HTTP/1.1
        Host: example.org
        Accept: */*
        Content-Type: application/json

    **Example Response**

    .. sourcecode:: http

        HTTP/1.1 200 
        Vary: Origin
        Vary: Access-Control-Request-Method
        Vary: Access-Control-Request-Headers
        Content-Type: application/json
        Transfer-Encoding: chunked
        Date: Tue, 07 May 2024 14:47:26 GMT

        {
            "CQLs": [
                "name": "ECR",
                "body": "library ECR version '1.1'\r\nusing FHIR version '3.0.0'\r\ninclude FHIRHelpers version '3.0.0' called FHIRHelpers\r\ncodesystem \"ICD9PROC\": 'http://hl7.org/fhir/sid/icd-9-proc'\r\ncodesystem \"LOINC\": 'http://loinc.org'\r\ncodesystem \"CPT\": 'http://www.ama-assn.org/go/cpt'\r\ncodesystem \"icd10cm\": 'http://hl7.org/fhir/sid/icd-10-cm'\r\ncodesystem \"sct\": 'http://snomed.info/sct'\r\ncodesystem \"rxnorm\": 'http://www.nlm.nih.gov/research/umls/rxnorm'\r\n\r\ndefine \"Chlamydia_Codes\": Concept {Code 'A56' from icd10cm, Code 'A56.0' from icd10cm, Code 'A56.00' from icd10cm, Code 'A56.01' from icd10cm, Code 'A56.02' from icd10cm, $\<Encoded CQL Script Continues\>"
            ]
        }

    :<query string name: identifiable name of the CQL body. Shoudlmatch the library name within the CQL content itself.
    :resheader Content-Type: application/json
    :statuscode 200: no error

.. _server CQL Execution Service:

CQL Execution Service
=====================

CQL Execution Service: Overview
-------------------------------
The CQL Execution Service is a derived cql-execution-service as developed by the DBCG group.
The service is responsible for actually executing relevant CQL against a patient, and creating
a set of CQL results which will be parsed by the results manager for the resultant ECR

CQL Execution Service: API Documentation
----------------------------------------

.. http:POST:: /cql-execution-service/cql/evaluate

    **Example CQL Storage Request**

    .. sourcecode:: http

        GET /cql-execution-service/cql/evaluate HTTP/1.1
        Host: example.org
        Accept: */*
        Content-Type: application/json
        Epic-Client-id; $\<Stored Epic Client Id if OAuth workflow is enabled>

        {
            "code": "Your CQL code",
            "terminologyServiceUri": "Terminology Service Endpoint",
            "terminologyUser": "Username for authentication",
            "terminologyPass": "Password for authentication",
            "dataServiceUri": "Fhir Data Provider Endpoint",
            "dataUser": "Username for authentication",
            "dataPass": "Password for authentication",
            "patientId": "The patient you want to run the library against"
            "parameters": [
                {
                    "name": "Name of the parameter as specified in the CQL",
                    "type": "Name of the type (currently only singleton CQL types are supported)",
                    "value": String (String, DateTime, and Time) | Integer | Decimal | Object (Code, Concept, Quantity, Interval)
                }
            ]
        }

    **Example Response**

    .. sourcecode:: http

        HTTP/1.1 200 
        Vary: Origin
        Vary: Access-Control-Request-Method
        Vary: Access-Control-Request-Headers
        Content-Type: application/json
        Transfer-Encoding: chunked
        Date: Tue, 07 May 2024 14:47:26 GMT

        [
            {
                "name": "Pt (Patient)",
                "location": "[1:1]",
                "resultType": "FHIR.Patient",
                "error": "",
                "result": "$\<FHIR Patient json here\>"
            },
            {
                "name": "Chalmydia Diagnosis",
                "location": "[15:1]",
                "resultType": "FHIR.Condition",
                "error": "",
                "result": "$\<FHIR Condition json here\>"
            }
        ]

    :<json string code: UTF-8 Encoded string body of CQL script definition
    :<json string terminologyServiceUri: If a translations service for converting valuesets is provided, CQL-execution-service will use
        this terminology service. Refer to https://build.fhir.org/terminology-service.html for specification on terminology interfaces
        using fhir
    :<json string terminologyUser: A basic username for authentication to the terminologyService, if applicable
    :<json string terminologyPass: A basic password for authentication to the terminologyService, if applicable
    :<json string dataServiceUri: The FHIR server the CQL script will be executed against
    :<json string dataUser: A basic username for authentication to the dataService, if applicable
    :<json string dataPass: A basic password for authentication to the dataService, if applicable
    :<json string patientId: A FHIR id which identifies the FHIR patient to be used in the CQL script.
    :resheader Content-Type: application/json
    :statuscode 200: no error