###################################
PACER Client
###################################

.. _client overview:

***************
Client Workflow
***************
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

==============================
Installation and Configuration
==============================
PACKER client supports both Linux and Windows platform. Mac is considered as a Linux environment. Please download appropriate
package from GitHub repositories. Repository URLs as follows.

* Windows: https://github.com/PACER-PHII/PACER-client-win.git
* Linux or Mac: https://github.com/PACER-PHII/PACER-client.git

For the latest development version, please clone the repository with ``git clone <repository_url>`` from the command line in Linux or GitHub Desktop in Windows or Mac. 
However, if more tested version is wanted, go to the reposiotry from browser and download either zip or gzip file from the released version. 
Link to the released version is available on the right middle side of the repository page.

Please note that this is a pilot project. The released version may have unseen issues. Contact the developers if any
issues occur. 

Database
********
Database must be created and properly configured before you proceed to deploy PACER-client service components. Any 
relational database can be used. PACER-client is tested with MS SQL and PostgreSQL server. 

.. note::
    
    * PostgreSQL can be downloaded from https://www.postgresql.org/download/
    * MS SQL is available at https://www.microsoft.com/en-us/sql-server/sql-server-downloads  


Database tables will
be automatically created by ECR Manager. However, the schema must exist. In the database (MS-SQL or PostgreSQL), create a schema
called "ecr" and setup a user for PACER-client.

Whether MS SQL or PostgreSQL is used, a schema needs to be named as "ecr". Tables will automatically be created by ECR manager
under the "ecr" schema.

For MS SQL server, in order to use Windows Authentication for MS SQL, make sure "ecr" database is owned by the account that will 
run PACER-client or the account has a db writer/reader permission. Please note that the schema name also needs to be "ecr"


Windows Server 2019
*******************
PACER-client uses a wrapper to run the Java application as a window service. Windows Service Wrapper (WinSW) is used for the 
wrapper.exe. All PACER-client components in this repository already have this wrapper application. Thus, nothing needs to be 
done for this wrapper. If you want to learn about the WinSW, please refer to https://github.com/winsw/winsw

Prerequisites
=============
**OpenJDK installation**

Java runtime environment is required to run this service. Either JRE or JDK needs to be installed.

* Go to https://docs.microsoft.com/en-us/java/openjdk/download 
* Download OpenJDK17 msi file (microsoft-jdk-17.0.2.8.1-windows-x64.msi) to install OpenJDK. 
* After installation, type ``java --version`` at the command line (or powershell) to verify its installation

**Authorization library installation**

After *OpenJDK* installation, authorization library must be downloaded and installed in JDK bin folder. download the following dll 
file.

* mssql-jdbc_auth-10.2.0.x64.dll from https://docs.microsoft.com/en-us/sql/connect/jdbc/download-microsoft-jdbc-driver-for-sql-server?view=sql-server-ver15 in zip or tar.gz file.
* Once the downloaded file is uncompressed, go to auth/ folder and choose the one that meets your host machine hardware configuration.
* Copy the file, *eg: mssql-jdbc_auth-10.2.0.x64.dll*, to JDK's bin folder. If installation msi file is used for the Java installation, then the JDK bin folder should be at ``C:\Program Files\Microsoft\jdk-17.0.2.8-hotspot\bin``

**Certificates for SSL**

All the traffic from client to external need to be on the secure socker layer. SSL requires a public signed 
certificate. This needs to be done by server side. However, if client network (or client firewall) manipulates the incoming 
certificate(s) and modify the chain of trust, then the PACER client applications won't be able to successfully establish the 
secure connections because the applications cannot validate the modified certificate(s).

In this case, the certificate must be trusted and added to the keystore. Here is the procedure to trust the certificate in the 
trust-store in Java.

* Export the server's certificate to file. This can be done by copying the server's endpoint URL and pasting the URL into the browser's address bar. Then, click on the lock icon to export the certificate to file.
* Copy the exported certified file to C:\Program Files\Microsoft\jdk-17.0.2.8-hotspot\lib\security folder.
* Open Powershell, and go to C:\Program Files\Microsoft\jdk-17.0.2.8-hotspot\lib\security
* Run the following command,

``keytool -import -alias <alias_name> -file <exported cert filename> -keystore cacerts``

* If you are asked for a password but haven't set it before, then the default password is "changeit". In production environment, this password needs to be updated. The certificate will be added to the trust-store.

Deployment
==========
There are three folders in the PACER-client-win repository. It is recommeded to create a separate folder to copy the following 
three folders. In this way, when updates are made, the original folder can be kept as a backup folder.

The applications must be deployed or started in the following order.

* pacer-index-api
* ecr-manager
* elr-receiver

In each foler, there is an xml file. Open the XML file and make necessary changes for the environment variables. After all the 
environment variables are set correctly, run the executable (exe) file. This will create a service for the application. The 
account information should be correctly entered as well.

.. warning::
    If any user access level is changed (for example, adding permission to the account used by PACER-client applications), 
    then service(s) MUST be restarted so that the new changes to the account can be affective.

Detail application installation instructions for each application are provided below.

.. _client win-pacer-index-api:

PACER-INDEX-API
~~~~~~~~~~~~~~~

At Powershell (in Admin mode), go to ``pacer-index-api/`` folder. And open ``pacer-index-api.xml`` file. Then, check the environment 
variables and change them as needed. JAVA_HOME should work as is if the same version of JDK in this document is used. If you are 
running this in the environment that security needs to be tightened, please change BASIC Auth parameters. SERVER_PORT can also 
be changed. Please note these variables are important as these will be used in another application. When everything is done, 
please run the following command at the Powershell.

``.\pacer-index-api.exe install``

This will install the pacer-index-api as a service. After the installation, open 'services' application (built-in app in Windows). 
From the list of services, locate the PACER Index API service. Right click on it and choose Properties. There, go to 'Log On' tab 
and choose 'this account' option. Then, add username and password. Please note that this account should have a permission to access 
local harddrive, otherwise the application will have an issue writing data to PIDB.db file.

*pacer-index-api service configuration*

pacer-index.api is used by ecr-manager. In order for ecr-manager to talk to PACER-server, we need to populate the pacer-index-api 
with PACER-server information. From a Chome browser, go to "http://localhost:8086/pacer-index-api/1.0.0/" 
And, use the 'manage-api-controller' option to add the index engry in the following format. Use POST option.

.. code-block:: JSON
    :linenos:

    {
       "providerName":"<provider name if available>",
       "identifier":"<provider identifier. ex, ORDPROVIDER|P49430>",
       "pacerSource":
       {
          "name":"<any name>",
          "serverUrl":"<PACER-server Job Maanagement System URL>",
          "security":
          {
             "type":"basic",
             "username":"<username of list manager in the PACER server>",
             "password":"<password of list manager in the PACER server>"
           },
              "version":"1.0.0",
              "type":"ECR"
           }
        }
    }

If provider information is not available, or facility information is preferred for the indexing, the following format can be used.
``providerName`` is left to blank. And, the ``identifier`` is used for the facility information.

.. code-block:: JSON
    :linenos:

    {
       "providerName": "",
       "identifier": "appfac|CYBERLAB|City of Houston",
       "pacerSource":
       {
          "name":"<any name>",
          "serverUrl":"<PACER-server Job Maanagement System URL>",
          "security":
          {
             "type":"basic",
             "username":"<username of list manager in the PACER server>",
             "password":"<password of list manager in the PACER server>"
           },
              "version":"1.0.0",
              "type":"ECR"
           }
        }
    }

The ``identifier`` value for both provider and facility should be identical to HLv2 ELR's provider and facility segment.

ECR manager uses the index API to get the PACER-server endpoint to send a query request. Thus, if there are multiple providers or facilities 
for the ELR messages, they all need to be added to this API service with their PACER-server endpoint.

ECR-MANAGER
~~~~~~~~~~~
In ``ecr-manager/`` folder. Open ``ecr-manager.xml`` file. Example xml file is shown below.

.. code-block:: XML
    :linenos:

    <service>
       <id>ecrmanager</id>
       <name>ECR Manager</name>
       <description>This manages ECR data from lab report and EHR data</description>
       <env name="JAVA_HOME" value="C:\Program Files\Microsoft\jdk-17.0.2.8-hotspot\"/>
       <env name="JDBC_DRIVER" value="com.microsoft.sqlserver.jdbc.SQLServerDriver"/>
       <env name="JDBC_URL" value="jdbc:sqlserver://<host>:1433;databaseName=ecr;integratedSecurity=true"/>
       <env name="LOCAL_BULKDATA_PATH" value="C:\workspace\PACER-client-win\ecr-manager\bulkdata"/>
       <env name="LOCAL_PACER_SECURITY" value="Basic username:password"/>
       <env name="LOCAL_PACER_URL" value="http://musctest.hdap.gatech.edu:8082/JobManagementSystem/List"/>
       <env name="PACER_INDEX_SERVICE" value="http://localhost:8086/pacer-index-api/1.0.0/search"/>
       <env name="TRUST_CERT" value="true"/>
       <env name="SERVER_PORT" value="8085"/>
       <executable>java</executable>
       <arguments>-jar "%BASE%\ecr-manager.jar"</arguments>
       <log mode="roll"></log>
    </service>

Please make sure the environment variables are accurate. 

In the ``ecr-manager.xml``, ``JDBC_URL`` must be set to the MS-SQL database where you will be storing the PACER data. 
LOCAL_* environment varialbles are mostly place holders. Even though they will not be used, please set them to correct values. 
LOCAL_BULKDATA_PATH needs to be pointing to existing folders. If not, path not available error message will be shown until 
the folder is creaed.

After configuring the XML file, save it and run the following command,

``.\ecr-manager.exe install``

This will install the ecr-manager as a service. After the installation, open 'services' application (built-in app in Windows). 
From the list of services, locate the ECR Manager service. Right click on it and choose Properties. There, go to 'Log On' 
tab and choose 'this account' option. Then, add username and password. Please note that this account should have a permission 
to access (read and write) the MS SQL server.

.. note::
    ***Exporting cases in CSV file***
    
    ECR-Manager has an API that will dump entire cases in csv file. The endpoint is http(s)://<yourhost>/ecr-manager/exportCSV. 
    If you run it from the browser, it will save the file in the download folder with name = csv_[datetime].csv.

ELR-RECEIVER
~~~~~~~~~~~~
In ``elr-receiver/`` folder, update ``elr-receiver.xml`` file. ``ECR_URL`` in the ``elr-receiver.xml`` is an environment variable 
that may need to be updated. However, if default values are used for ECR-MANAGER installation, and ECR-MANAGER and ELR-RECEIVER are 
running in the same machine, then the default configuraion may be used without modifications.

After the XML file is configured, please save it and run the follwoing command from the Powershell,

``.\elr-receiver.exe install``

This will install the elr-receiver as a service. After the installation, open 'services' application (built-in app in Windows). 
From the list of services, locate the ELR Receiver service. Right click on it and choose Properties. There, please go to 'Log On' tab 
and choose 'this account' option. Then, add username and password. Please note that this account should have a permission to access 
the local hard disk. ELR-RECEIVER needs to have read and write permission to the hard disk so that a queue file can be created and managed.

PACER-UI
~~~~~~~~
This is a user dashboard that shows the case reports in PACER. The dashboard is written in Angular, and the source codes are available in 
https://github.com/PACER-PHII/pacer-ui.git if you are interested in and willing to contribute in the development.

If you just want to deploy the dashboard, please follow the instruction below.

1. In the Server Manager, Enable IIS. You may also need to configure user so that the IIS server can access the folder
2. Download the zip file from the release tag (https://github.com/PACER-PHII/PACER-client-win/releases)
3. Unzip the downloaded file and copy the folder named, "pacer-ui" to the place where you want to run your IIS server on.
4. Locate config.json file located in ``/pacer-ui/config/config.json`` and edit the line "api": "http://yellowisland01.icl.gtri.org:8085" to your API URL. For example "api": "http://myapi.org:8080/" if you will be using the UI from the same host that ECR-MANAGER is deployed.
5. Add a new application in the IIS setup and name the alias as "pacer-ui" and set the path to the "pacer-ui" folder that you created in the step 3.

Use the web browser and go to http://localhost/pacer-ui if you are running from the same host.

If you want to have authentication on the UI, please follow the instruction at https://learn.microsoft.com/en-us/iis/configuration/system.webserver/security/authentication/windowsauthentication/
The IIS will ask the Windows Serverx user credential. Please note that the browser saves the credentials. So, if you are using the shared computer, please make sure you clean the web data.


Linux Server
************
Prerequisites
=============
PACER-client can be installed in the Linux environment in two ways. One is using Docker, and the other is using Java and runnit them at the command line. 

**Docker Installation**
For Docker installation, the following packages must be installed.
* Docker Engine : (Unbuntu) https://docs.docker.com/engine/install/ubuntu/ (Redhat) 
* Docker Compose : https://docs.docker.com/compose/compose-file/ 

Docker downloads base images from Docker server. Therefore, incoming HTTP traffic from outside must be allowed.

PACER-client Deployment for Docker
==================================
Once the docker is installed, run the following command to install PACER-client in Docker container.
1. Go to the folder where the PACER-client is cloned or downloaded.
2. Open ``docker-compose.yml`` and check the envrionment varialbes. In most cases, the variables can be used as is. However, if you wish to change, please do for your environment. Only *ecr-postgresql* is set to restart when host restarted. If the other components need to be reatarted, please put *restart: always* in each component you want to enable the restart.
3. Run the following command,

``sudo docker-compose up --build -d``

4. Please check the status of each component by running,

``sudo docker ps -a``

If all components are successfully installed and deployed, the following components must be in the running state.
* ecr-postgresql
* elr-receiver
* ecr-manager
* pacer-index-api

PACER-client Deployment for non-Docker
======================================








===========
ECR Manager
===========
Overview
********


API Documentation
*****************


.. _client elr receiver:

============
ELR Reciever
============
<ELR_Receiver Overview here>

.. _client index service:

===============
PACER Index API
===============

Overview
********
<PACER Index API Overview here>

API Documentation
*****************
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