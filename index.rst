:tocdepth: 1

.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. sectnum::

Introduction
============

   There are only two hard things in Computer Science: cache invalidation and
   naming things.

   — Phil Karlton

We assert that hostnames are primarily for **human** convenience and should be
easy to remember, easy to predict, and concise.  This should be the primary
guiding principle when developing naming heuristic(s).

The current naming scheme being implemented at the summit was widely disliked
during discussion at the `Puppeton Tucson July 2019 <https://confluence.lsstcorp.org/display/LSSTCOM/Agenda+-+Puppeton+Tucson+July+2019>`_
as being overly complex and difficult to understand.  During that event, it was
agreed upon to `decouple the puppet role from the hostname
<https://ittn-001.lsst.io/#decouple-node-fqdn-from-hierarchy-layers>`_, this
change opens the door to revising the hos tame and domain name strategy.

Implementing a reformed DNS strategy is on the critical path for rolling out a
`Deployment Platform <https://ittn-002.lsst.io/>`_ at the Summit and Base to
support commissioning activities.  A practical and rapidly implementable
solution is urgently needed, even if it is not the complete solution that will
be used during observatory operations.

Goals
=====

* Support immediate development and commissioning needs
* Rapid implementation requiring low effort
* Avoid disrupting existing desktop/AD environment
* Low implementation complexity / low maintenance burden
* Avoid pain for multi-VPN users
* Support automated deployment via `foreman
  <https://ittn-002.lsst.io/#bare-metal-provisioning>`_; DNS service must have
  an API that is supported by an existing ``foreman`` plugin.

Scope
=====

Environment
-----------

There is an existing and mature Windows/Active Directory management environment
both in Tucson and at the Base.  This includes functionality such as email and
other desktop support related services. We propose a logical separation of the
desktop/AD environment and Tucson Lab/Summit & Base data centers.

Desktop/AD
^^^^^^^^^^

The windows Desktops/Desktop Support infrastructure/Active Directory
environments is well established and any change to name resolution would be
highly disruptive to desk users.  It would also require time consuming
migration of the existing AD data.

**The existing Desktop/AD environment(s) are explicitly out of scope of this
document.**  Other than adjusting the domain search list, it is recommended that
no refactoring of this environment is undertaken in the immediate future.

Datacenter(s)/Lab(s)
^^^^^^^^^^^^^^^^^^^^

The Tucson Test/Lab environment and the Summit/Base data centers are less
developed and somewhat less disruptive to refactor.

**This document only applies to the Datacenter(s) and Lab(s) environments.**

Sites
-----

*These Datacenter(s)/Lab(s) environments are considered in scope:*

* Lab/Tucson
* Summit/Cerro Pachón
* Base/La Serena

**All other sites/envirnoments that may be hosting lsst equipment or services
are explicitly out of scope.** E.g.: the names of camera development servers at
SLAC

Discussion
==========

Prior Art
---------

There is an existing proposal for LSST DNS organization, which is partially
implemented at the Summit:
`LSST ITC DNS Infrastructure <https://confluence.lsstcorp.org/display/SYSENG/LSST+ITC+DNS+Infrastructure>`_.

There are a number of criticisms of this proposal, including:

* It results in large and difficult to remember hostnames. E.g.,
  ``comp-a4-hyp-gs-02.cp.cl.lsst.org``
* Heavy partitioning / high complexity is not necessary for the modest scale
  of the in-scope environments.
* Country level sub-domains are unnecessary partitioning for a small number of
  sites.
* Hostnames for servers and services that encoding details for the physical
  location (room, rack, etc.) and/or network structure will result in brittle
  configuration of dependent services.  Re-provisioning or relocating a server
  to a different location should not generally require client configuration
  changes.  Similarly, changes to the logical network should not require
  modifying the hostname.

Domains
-------

LSST as a project, and within the IT organization, is already using a number of
different domains.  We do not believe there is a top level requirement or
constraint that all or even most of LSST's computing infrastructure needs to be
arranged in a hierarchy underneath a common domain.

A non-exhaustive list of domains known to be currently in use by LSST:

* ``lsst.org`` (IT)
* ``lsstcorp.org`` (IT)
* ``ls.st`` (IT)
* ``lsst.local`` (IT)
* ``lsst.codes`` (DM/SQRE)
* ``lsst.io`` (DM/SQRE)
* ``lsst.cloud`` (DM/SQRE)
* ``ncsa.illinois.edu`` (DM/\*)
* ``lsst.rocks`` (DM/DAX)

Split-View DNS and VPNs
-----------------------

`Split-view or split-horzon DNS
<https://en.wikipedia.org/wiki/Split-horizon_DNS>`_ is simply providing
different name resolution information to internal clients and the public DNS
hierarchy.  The implication of this is that "internal" name servers are the only
source of "internal" name information.  The motivation for a split view is
often to avoid providing network information to an adversary.  However, the
obscuration is by no measure absolute. For example, it does not mitigate
traffic observation.  If any host with access to "internal" DNS is compromised,
the hidden name information is exposed.  We also note that no LSST threat model
currently exists which requires obscuring internal network information.

Typically, access to "internal" nameservers is provided to and automatically
configured for VPN clients upon connection.  This model works for VPN users
that will only ever be connected to a single VPN at a time.  However, it
completely falls apart if multiple VPNs are in use, with each trying to
configure its own nameservers to provide "internal" name resolution.  This
problem is already being encountered within LSST Data Management with remote
users needing to connect via VPN to both NCSA and NOAO in Tucson.

Summit Degraded Operations
--------------------------

Unlike all other LSST sites, the summit network has a requirement to function
with a complete loss of external connectivity.  We acknowledge that this means
that a hosted service such as `route53 <https://aws.amazon.com/route53/>`_
alone is not sufficient for the operational needs of the summit network.
However, a hosted DNS service should be sufficient for all other sites.

Domain name(s)
==============

A per site sub-domain is to be used to provide isolated administrative domains
which do not need to coordinate changes and to allow the same hostname to be
used currently at multiple sites.

option a
---------

+---------------------+--------------------+
| Site                | Domain name        |
+=====================+====================+
| Tucson              | ``tuc.lsst.cloud`` |
+---------------------+--------------------+
| Summit/Cerro Pachón | ``cp.lsst.cloud``  |
+---------------------+--------------------+
| Base/La Serena      | ``ls.lsst.cloud``  |
+---------------------+--------------------+

option b
--------

+---------------------+----------------+
| Site                | Domain Name    |
+=====================+================+
| Tucson              | ``tu.lsst.st`` |
+---------------------+----------------+
| Summit/Cerro Pachón | ``cp.ls.st``   |
+---------------------+----------------+
| Base/La Serena      | ``ls.ls.st``   |
+---------------------+----------------+

Infrastructure
==============

.. figure:: /_static/resolvers.png
   :name: fig-name-forwarders
   :alt: obligatory diagram

Forward and reverse DNS for all sites is managed via public route53 zones.
`route53 <https://aws.amazon.com/route53/>`_ is considered the canonical and
sole "source of truth".  Two forwarding only / caching name servers shall be
maintained per site.  DNS clients are configured to use the per site local
resolvers to simplify any future refactoring.

The local site domain shall be added to the domain search list of any desktop
environments at that site.  E.g., ``tuc.lsst.cloud`` would be added to the
domain search list for desktop clients in Tucson.

Local Resolvers
---------------

Due to existing staff familiarity, at least initially, `ISC BIND
<https://www.isc.org/bind/>`_ shall be use as local site forward resolver.
`Unbound <https://www.nlnetlabs.nl/projects/unbound/about/>`_ or other "more
modern" DNS services shall be evaluated in the future.

Summit/Cerro Pachón
-------------------

Prior to the start of operations, the summit nameservers shall be reconfigured
to be "authoritative" for all local zones.  `route53
<https://aws.amazon.com/route53/>`_ will continue to be the definitive source
of truth.  The zone configuration for local nameservers shall be machine
generated from route53 information.

`ruby_route_53 <https://github.com/pcorliss/ruby_route_53>`_ is an example of
an existing tool that is able to generate `ISC BIND
<https://www.isc.org/bind/>`_ compatible zone files from route53 zones.

TTLs
----

As local forwarding only resolvers are inherently non-authoritative, in order
for changes to an existing record to become effective, the time to live (TTL)
on that record will first have to expire.  For this reason, relatively short
TTLs shall be used.

+------------+-------------------+
| Host Type  | max TTL (seconds) |
+============+===================+
| service    | 60                |
+------------+-------------------+
| all others | 300               |
+------------+-------------------+

Hostnames
=========

Servers/Hosts
-------------

Use "conversational" names -- the name that you would use to refer to a host
in conversation with a co-worker.

Highly generic terms such as ``node`` or ``server`` are often ambiguous.
Using a functional description such as ``www``, ``webserver``,
``kubernetes<node>``, ``hypervisor<node>`` convey more useful information and
are more natural to use in human conversation.

Cluster names, if applicable, are often the least ambiguous way to refer to a
host.  As an example, if the cluster name was ``larry``, nodes of that cluster
may be named ``larry<node>``.

Example hostnames:

+-----------+----------+------------+
| poor      | better   | preferred  |
+===========+==========+============+
| ``node1`` | ``k8s1`` | ``larry1`` |
+-----------+----------+------------+

Cluster Roles
^^^^^^^^^^^^^

If a cluster or logical grouping of machines has more than one role, this may
be expressed in the hostname.  The format is ``<cluster>-<role><node>``.

Example hostnames:

+---------+------+-------------------------+-------------+
| cluster | role | instance                | hostname    |
+=========+======+=========================+=============+
| comcam  | fp   | 1                       | comcam-fp1  |
+---------+------+-------------------------+-------------+
| comcam  | mcm  | (there can be only one) | comcam-mcm  |
+---------+------+-------------------------+-------------+
| comcam  | dc   | 1                       | comcam-dc1  |
+---------+------+-------------------------+-------------+
| comcam  | hcu  | 1                       | comcam-hcu1 |
+---------+------+-------------------------+-------------+
| comcam  | vw   | 1                       | comcam-vw1  |
+---------+------+-------------------------+-------------+
| comcam  | db   | 1                       | comcam-db1  |
+---------+------+-------------------------+-------------+

BMCs
----

A baseboard management controller (BMC) is tied to a specific physical server,
therefore it makes sense for the hostname to reflect this relationship. The
format is ``<phy hostname>-bmc``.

Example hostnames:

+-------------------+---------------------+
| physical hostname | BMC hostname        |
+===================+=====================+
| ``larry1``        | ``larry1-bmc``      |
+-------------------+---------------------+
| ``core1``         | ``core1-bmc``       |
+-------------------+---------------------+
| ``comcam-db01``   | ``comcam-db01-bmc`` |
+-------------------+---------------------+

IP Phones
---------

IP Phones have a tendency to be migrated between physical rooms.  An example
of this would be a desktop user migrating between offices and taking the phone
with them.  Due to this mobility, phones *shall not* be named by physical
location.  While the extensions bound to a phone may change over time, it is a
highly useful handle for identifying a device.  The suggested format is
``ph-x<primary extension>```.  If a phone does not have an internal extension
in the dial plane and only uses a DID, the format is ``ph-<DID>``.

Example hostnames:

+--------------------------+-------------------+
| primary extension or DID | phone hostname    |
+==========================+===================+
| ``x1234``                | ``ph-x1234``      |
+--------------------------+-------------------+
| ``x56``                  | ``ph-x56``        |
+--------------------------+-------------------+
| ``555-857-6309``         | ``ph-5555876309`` |
+--------------------------+-------------------+

Fixed Location Devices
----------------------

It is common for network devices to have a role or function which is dependent
upon its location within a data center. For example, a "top of rack" (TOR)
switch is inherently associated with the rack/cabinet in which it is installed.
If a physical TOR switch is relocated to another cabinet, its existing
configuration would be invalidated, and it would be serving a new role as the
TOR for the new cabinet.  Conversely, if a core/spine switch is relocated
between cabinets, as long as it is serving the same logical function within the
network, its name should not change.  For this reason, it is appropriate for
the location to be part of the hostname for certain classes of devices, such as
a TOR.

**Per site location names are out of the scope of this document.**

TOR Switches
^^^^^^^^^^^^

Consider a cabinet configured with two TOR switches.  One is a 10G access
switch configured as a leaf in the main leaf/spine topology and the other is a
lower cost 1G switch connected to an independent "management" topology that
provide redundant uplinks for reliability but is not designed for high
performance east<->west traffic.  The management switch would be used to
connect PDUs, BMCs, IP cameras, etc.

While both switches are l2/l3 Ethernet switches associated with the same
physical cabinet, their function may be different enough to warrant a distinct
naming convention.

TOR switch hostname format:

+----------------+-----------------------------+
| switch type    | format                      |
+================+=============================+
| regular access | ``<location>-sw<index>``    |
+----------------+-----------------------------+
| management     | ``<location>-mgtsw<index>`` |
+----------------+-----------------------------+

Example hostnames if the location / name of cabinet was ``dc-a1``:

+----------------+------------------+
| switch type    | hostname         |
+================+==================+
| regular access | ``dc-a1-sw1``    |
+----------------+------------------+
| management     | ``dc-aw-mgtsw1`` |
+----------------+------------------+

IP Cameras
^^^^^^^^^^

Format ``<location>-cam<index>``.

Wifi Access Points
^^^^^^^^^^^^^^^^^^

Format ``<location>-ap<index>``.
