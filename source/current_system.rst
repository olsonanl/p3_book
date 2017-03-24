==============================
 Current System Configuration
==============================

Part of the purpose of this document is to define a road forward for a
coherently-managed, replicatable, and monitorable system, but we are
not there yet. Until we get there we need to be able to monitor and
diagnose problems with the existing system. Thus this chapter.

We discuss here the PATRIC systems that are operated in the MCS/SEED
environment at Argonne. The PATRIC web frontend systems, Data API
service, and Solr service are operated and managed by BI staff at
Virginia Tech.

System Overview
===============

Following is an overview diagram of the PATRIC systems at Argonne:

.. image:: images/anl-system-diagram.pdf

The general architecture is that the primary entrypoint for service
requests to the PATRIC infrastructure is the HTTPS address
https://p3.theseed.org. Services are hosted in the /services address
space:

+---------------------------+--------------------+--------------------+
| Endpoint                  | Service            |Backend Service     |
|                           |                    |Location            |
+===========================+====================+====================+
|/services/app_service      |Application Service |beech:7124          |
+---------------------------+--------------------+--------------------+
|/services/awe_api          |AWE Server          |redwood:7080        |
+---------------------------+--------------------+--------------------+
|/services/handle_service   |Handle Service      |beech:7109          |
+---------------------------+--------------------+--------------------+
|/services/homology_service |Homology Service    |gum:7134            |
+---------------------------+--------------------+--------------------+
|Non-public                 |Kmer2/Pattyfam      |spruce:6100         |
|                           |SErvice             |                    |
+---------------------------+--------------------+--------------------+
|/services/minhash_services |Minhash Service     |gum:7138            |
+---------------------------+--------------------+--------------------+
|/services/ProbModelSeed    |Prob ModelSEED      |beech:7130          |
+---------------------------+--------------------+--------------------+
|/services/shock_api        |Shock server        |hemlock:7078        |
+---------------------------+--------------------+--------------------+
|/services/Workspace        |Workspace Service   |beech:7124          |
+---------------------------+--------------------+--------------------+
|/services/WorkspaceDownload|Workspace Download  |beech:7129          |
|                           |Service             |                    |
+---------------------------+--------------------+--------------------+

Application Service
-------------------

The Application Service runs on beech in
``/disks/p3/deployment/services/app_service``.  The source originates
from https://github.com/theseed/app_service. 

The Application Service handles request form the PATRIC website to
start execution of a potentially long-running computation. Each of
these computations is described by an application specfication
file. The work done by the application is implemented in a script
named App-ServiceName. The management of invocation of these scripts
is handled using the `AWE workflow engine
<https://github.com/MG-RAST/AWE/wiki>`_.

The AWE workers for most of these scripts are running on redwood. For large genome
annotation runs we will start annotation workers on several other SEED
systems (gum, holly, larch).

The Application Service accesses the mongo database hosted on redwood on port 27017
named AWEDB for low-level access to AWE data.

This service is responsible for the following PATRIC website services:

* Assembly (invokes ARAST). ``App-GenomeAssembly``

* Annotation. ``App-GenomeAnnotation``

* Variation Analysis. ``App-Variation``

* Expression Import. ``App-DifferentialExpression``

* RNA-Seq Analysis. ``App-RNASeq``

* Proteome Comparison. ``App-GenomeComparison``

* Model Reconstruction. ``App-ModelReconstruction``

AWE Server
----------

The AWE server responsible for running Application Service jobs is
hosted on redwood in
``/disks/kb/deployment/services/awe_service``. Unlike most of the rest
of the PATRIC backend systems, the files in the ``/disks/kb``
hieararchy are somewhat a legacy install and are thus owned by user
``iris`` instead of user ``p3``.

The AWE server uses a mongo database hosted on redwood on port 27017
named AWEDB.

Documentation for AWE is available at `the AWE wiki
<https://github.com/MG-RAST/AWE/wiki>`_.

Handle Service
--------------

The Handle Service is a wrapper around Shock and a database holding
information about the hosted files. It has limited use in PATRIC but
there is an instance running on beech.

Homology Service
----------------

The Homology Service runs on gum in
``/disks/p3/deployment/services/homology_service``. It provides the
backend computation for the PATRIC BLAST service. 

Kmer2/Pattyfam Service
----------------------

The Kmer2/Pattyfam service is an internal-only service used by the
Genome Annotation code to call kmer functions and pattyfam
membership. It runs on spruce and is started using the procedure at
https://github.com/PATRIC3/procedures/blob/master/system_startup/spruce.mcs.anl.gov.

Minhash Service
---------------

The Minhash Service runs on gum in
``/disks/p3/deployment/services/Minhash``. It provides the backend
computation for the PATRIC Similar Genome Finder service.

ProbModelSEED Service
---------------------

The ProbModelSEED Service hosts a service used by the ModelSEED
system. For more details contract Chris Henry.

Shock Server
------------

The Shock Server provides file hosting for the PATRIC Workspace. It is
hosted on hemlock in ``/disks/kb/deployment/services/shock_service``.
Documentation for Shock is available at `the Shock
Wiki <https://github.com/MG-RAST/Shock/wiki>`_. Dan Olson is familiar
with the operational details of Shock.

The Shock service uses a mongo database hosted on redwood named HemlockShock.

Workspace Service
-----------------

The Workspace Service implements the user workspace as seen in the
PATRIC website. It runs on beech in
``/disks/p3/deployment/services/Workspace``. It uses the mongo
database also hosted on beech (database name WorkspaceBuild).

Workspace Download Service
--------------------------

The Workspace Download Service is hosted aside the Workspace Service
and provides simple download services for workspace files.

Starting and Stopping Services
==============================

Most of the services listed above are KBase-style services. That is,
they are implemented in Perl and conform to a standard release
engineering protocol and general service code structure originally
defined for the KBase project. They implement a JSONRPC protocol for
exposing the API, and use the Perl `Plack <http://plackperl.org>`_
infrastructure for hosting web services, and in particular the
`starman <http://search.cpan.org/dist/Starman/>`_ preforking web
server. 

These services expose ``start_service`` and ``stop_service`` scripts
that are installed into the service directory, typically located in
the ``/disks/p3/deployment/services/<service-name>`` directory on the
local disk on the machine hosting the service. The ``start_service``
script writes the process identifier for the service into a file named
``<service-name>.pid`` in the service direcotry. The ``stop_service``
script will use that file to locate the service to be stopped. 

The PATRIC release engineering infrastructure will also expose
``start_service`` and ``stop_service`` scripts with the same
operational semantics for the other styles of services; in particular
the newer release engineering wrappers for the Solr service and the
node.js-based front-end and API services. 

AWE Workers
===========

The AWE service model is somewhat different than the other
servers. The AWE server provides job queueing and worker coordination;
the actual workers are started independently. The code for starting
these workers is located on redwood in
/disks/kb/deployment/services/awe_service. 
