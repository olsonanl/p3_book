=============================
System Configuration
=============================

The PATRIC system is composed of a large number of interconnected
components which fall into two major categories: the frontend
services and the backend services.

Frontend services provide the primary website interface for the PATRIC
website at http://www.patricbrc.org. The frontend services are
implemented in Node.js using the Express toolkit. 

The backend services comprise the bulk of services in the system. These
include the Solr and Data API services that feed both the frontend and
some of the computational backends, as well as the purely backend
services that implement the bioinformatics applications provided by
PATRIC.

The services describe here fall into several different categories; the
distinction between these categories becomes important to understand
when we define the master configuration file and the monitoring tools
which we may derive from it.

Physical Organization
---------------------

Before I delve into the details of the service types I wish to discuss
the concrete implementation details of the PATRIC services.

When we use the term *service* in this document it has a very specific
meaning:

  A *service* is a process (or set of related processes) which present a
  network API to its clients via well-defined protocol over one or
  more TCP sockets.

We can thus identify a service by the name of the host on which it
executes and the TCP port(s) on which it listens.

The implementation of each service provides a mechanism to start and
stop the service. The stop-service mechanism must support a way to
stop precisely the service that is meant to be stopped; that is,
``pkill service-name`` is not an acceptable mechanism.

The services in PATRIC are implemented using one of two primary
mechanisms: Node.js services and KBase-style JSONRPC services.

Node.js Services
~~~~~~~~~~~~~~~~

KBase-style Services
~~~~~~~~~~~~~~~~~~~~




There are three primary Node.js services. 

  Web service
    The web service hosts the webpages and frontend Javascript that
    implements the PATRIC web interface.

  User service
    The user service hosts the PATRIC authorization and user
    management mechanisms. 

  API service
    The API service provides a wrapper around the underlying Solr
    database that holds the PATRIC genomic data. This wrapper enforces
    protection of user data as well as providing convenience methods
    for accessing complex data.

