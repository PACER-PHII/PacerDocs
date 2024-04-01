###################################
PACER Client
###################################

.. _client overview:

Client Workflow
===============
PACER workflow starts as the **ELR receiver** receives a positive STD electronic lab result (ELR) message in HL7 v2.5.1. Once 
the ELR message is received, an initial ECR is created from the ELR message. The ECR is then sent to **ECR manager** to be saved 
as a case in the database.

**ECR manager** constructs a PACER request for the ECR case and submit to the the PACER-server to query EHR FHIR server for the 
patient's clincal data. The query criteria the STD condition are defined with clincal query langurage (CQL). Figure 1 depicts the 
overall architecture of PACER.

.. image:: client_fig/PACER_Architecture.png
    :width: 700
    :alt: Overall PACER Architecture

.. _client installation:

Installation
============
PACKER client supports both Linux and Windows platform. Mac is considered as a Linux environment. Please download appropriate
package from GitHub repositories.

Windows: https://github.com/PACER-PHII/PACER-client-win
Linux: https://github.com/PACER-PHII/PACER-client

For the latest version, git clone from the repository. However, they are not fully tested and could be unstable. Please use 
the release link to download the released version.

Please note that this is a pilot project. Even the released version may have unseen issues. Contact the developers if any
issues occurred. 

Windows Server 2019 Installation
--------------------------------
PACER-client uses a wrapper to run the Java application as a window service. Windows Service Wrapper (WinSW) is used for the 
wrapper.exe. All PACER-client components in this repository already have this wrapper application. Thus, nothing needs to be 
done for this wrapper. If you want to learn about the WinSW, please refer to https://github.com/winsw/winsw

Prerequisites
^^^^^^^^^^^^^
**OpenJDK installation**

Java is required to run this service. Thus, either JRE or JDK needs to be installed. Go to https://docs.microsoft.com/en-us/java/openjdk/download 
and download OpenJDK17 msi file (microsoft-jdk-17.0.2.8.1-windows-x64.msi) to install OpenJDK. After installation, 
type "java --version" at the command line (or powershell) to verify its installation

**Authorization library installation**

After installation, authorization library must be downloaded and installed in JDK bin folder. download the following dll file.

mssql-jdbc_auth-10.2.0.x64.dll
from https://docs.microsoft.com/en-us/sql/connect/jdbc/download-microsoft-jdbc-driver-for-sql-server?view=sql-server-ver15

zip or tar.gz file is available from the link above. Once uncompressed, go to auth/ folder. and choose the one meets your VM 
configuration.

Then, copy the file, mssql-jdbc_auth-10.2.0.x64.dll, to JDK's bin folder. If installation msi file is used for the Java 
installation, then the JDK bin folder should be ```C:\Program Files\Microsoft\jdk-17.0.2.8-hotspot\bin```

**MS Sql Server Database**

Any relational database can be used. If you prefer another database such as PostgreSQL, refer to https://www.postgresql.org/download/windows/ 
and download the installer to install PostgreSQL database.

ECR Manager needs to have a database to persist ECR data from electronic lab reports and EHR. Whether you use MS SQL or PostgreSQL, 
"ecr" must be used for the name of database and schema. Tables will automatically be created.

In order to use Windows Authentication for MS SQL, make sure "ecr" database is owned by the account that will run PACER-client or 
the account has a db writer/reader permission. Please note that the schema name also needs to be "ecr"

**CERTIFICATE FOR SSL**

All the traffic from client to external need to be on the secure socker layer. SSL transaction requires a publically signed 
certificate. This needs to be done by server side. However, if client network (or client firewall) manipulates the incoming 
certificate(s) and modify the chain of trust, then the PACER client applications won't be able to successfully establish the 
secure connections because the applications cannot validate the modified certificate(s).

In this case, the certificate must be trusted and added to the keystore. Here is the proecure to trust the certificate in the 
trust store in Java.

Export the server's certificate to file. This can be done by copying the PACER server's endpoint URL to Browser's address bar. 
Then, click on the lock icon to export the certificate to file.

Copy the exported certified file to C:\Program Files\Microsoft\jdk-17.0.2.8-hotspot\lib\security folder.

Open Powershell, and go to C:\Program Files\Microsoft\jdk-17.0.2.8-hotspot\lib\security

Run this command,

```keytool -import -alias <alias_name> -file <exported cert filename> -keystore cacerts```

If you are asked for a password but haven't set it before, then the default password is "changeit". In production environment, 
you need to change this password. Certificate will be added to the trust store.



.. _client deployment:

PACER-client deployment
=======================
Database must be installed and properly configured before you proceed to deploy PACER-client service components. Tables will
be automatically created by ECR Manager. However, the schema must exist. In the database (MS-SQL or PostgreSQL), create a schema
called "ecr". And, setup a user for PACER-client. This user credentials are required for the service components to access the database,

There are three folders in the PACER-client-win repository. It is recommeded to create a separate folder to copy the following 
three folders. In this way, when updates are made, the original folder can be kept as a backup folder.

The applications must be deployed or started in the following order.

* pacer-index-api
* ecr-manager
* elr-receiver

In each foler, there is an xml file. Open the XML file and make necessary changes for the environment variables. After all the 
environment variables are set correctly, run the executable (exe) file. This will create a service for the application. The 
account information should be correctly entered as well.

.. _client database:

Database
========
<Database Information Here>

.. _client ecr manager:

ECR Manager
===========

ECR Manager: Overview
---------------------
<ECR Manager Information here>

ECR Manager: API Documentation
------------------------------
<ECR Manager API Review here>

.. _client elr receiver:

ELR Reciever
============
<ELR_Receiver Overview here>

.. _client index service:

PACER Index API
===============

PACER Index API: Overview
-------------------------
<PACER Index API Overview here>

PACER Index API: API Documentation
----------------------------------
<PACER Index API API Documentation here>

.. _client ui:

PACER UI
========

PACER UI: Overview
------------------
<PACER UI: Overview>

PACER UI: Walkthrough
---------------------
<PACER UI User Walkthrough with pictures here>