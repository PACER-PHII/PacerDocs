###################################
Welcome to PACER's documentation!
###################################

This document exists as a guide to help users understand the **P** ublic **H** ealth Automated **C** ase **E** lectronic **R** eporting tool, providing user and developer documentation.

What is PACER?
==============

**PACER** is a client-server architechture for collecting supplemental health care data for health department and public health organizations.

PACER describes both a client component, as well as a server component.
PACER client consists of a local database, an ELR receiver, a manager api, and a ui frontend.
The client program can be configured to connect to one or many seperate PACER Server
PACER server is a hosted supplemental service for FHIR EHR servers, designed to retrieve supplimental information for relevant cases data


For users, You can review the :doc:`client` section and :doc: `server` section for a detailed breakdown of services within each components
For developers, direct installation instructions can be found in the :ref:`client installation` and :ref:`server implementation` page.

.. note::

   This project is under active development.

Contents
========

.. toctree::

   client
   server