.. _installation:

========================
Installing Archivematica
========================

*On this page*

* :ref:`Overview <overview>`
* :ref:`Technical requirements <tech-requirements>`
* :ref:`Instructions for new installations <instructions>`
* :ref:`Advanced installation options <advanced>`

.. _overview:

Overview
========

Archivematica is not a single application - dozens of different components and
tools are required for a fully working installation. As a result, there are many
possible deployment configurations.

These instructions are designed to get you up and running as quickly as
possible, even if you are not familiar the various tools and applications that
are bundled into Archivematica. Experience with the Linux command line is
helpful, and to support a running production system it should be considered a
requirement.

If you need assistance or clarification regarding the installation instructions,
the `Archivematica google group`_ is a good place ask questions.

.. note:: For testing purposes, you may find it easier to install on a virtual
   machine using Vagrant. See the :ref:`Quick Start Guide <quick-start-install>`

.. _tech-requirements:

Technical requirements
======================

Operating system
----------------

Archivematica |release| installation instructions are provided here for the
following operating systems:

* Ubuntu 14.04.5 64-bit Server Edition
* Ubuntu 16.04.2 64-bit Server Edition (beta)
* CentOS 7.3.1611 64-bit

Archivematica |version| is the first release to be tested on Ubuntu 16.04. Support
for this OS is still considered beta; installation has been tested but production
deployments are limited.

Other Linux distributions should work, but will require customization of these
installation instructions.

Support for Mac OS X is possibly in theory, but is not being tested, and would
require more significant deviation from these instructions.

Archivematica is unlikely to ever run directly in a Windows environment.
Consider the use of a virtualization platform to run Linux VMs.

Dependencies
------------

Archivematica has a long list of software it depends on. All of these
dependencies are installed when following the instructions below.

In these instructions everything is installed into one machine. It is possible
to install some of the components on separate machines, to improve performance,
such as:

* MySQL
* gearman
* Elasticsearch (optional as of Archivematica 1.7, see below)

Archivematica has a long list of software it depends on. All of these
dependencies are installed when following the instructions below.

In these instructions everything is installed into one machine. It is possible
to install some of the components, such as MySQL, Elasticsearch, and Gearman,
on separate machines, to improve performance. Using additional machines will
require additional configuration. For more information, see :ref:`Advanced <advanced>`.

.. note::
   Archivematica |version| has been tested with MySQL 5.5, including
   the Percona and MariaDB alternatives. Archivematica uses MySQL 5.7 on
   Ubuntu 16.04.

   Some of the tools run by Archivematica require Java to be
   installed (primarily Elasticsearch and fits). On Ubuntu 14.04, Open JDK 7
   is used. On Ubuntu 16.04 Open JDK 8 is the default. It is possible to use
   Oracle Java 7 or 8 instead.

   The remaining dependencies should be kept at the versions installed
   by Archivematica.

Elasticsearch
^^^^^^^^^^^^^

Installing Elasticsearch to provide a search index is now optional as of
Archivematica version 1.7. Installing Archivematica without Elasticsearch means
reduced consumption of compute resources and lower operational complexity.
However, some functionality, such as the Backlog, Appraisal and Archival Storage
tabs, is not available.

When Elasticsearch is used, Archivematica |release| requires version 1.x (tested
with 1.7.6). Support for a more recent version of Elasticsearch is being
developed and is planned for a future release.


Hardware
--------

Archivematica is capable of running on almost any hardware supported by Linux;
however, processing large collections will require better hardware.

Minimum hardware requirements
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For small-scale functionality testing using small collections (transfers with 100
files or less, total file size 1 GB or smaller), we recommend the following minimum
hardware requirements:

* Processor: 2 CPU cores
* Memory: 2GB+
* Disk space (processing): 7GB plus two to three times the disk space required for the
  collection being processed (e.g., 3GB to process a 1GB transfer)

Recommended minimum production requirements
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For production processing, the hardware requirements depend almost entirely on
the size and number of files being processed. These recommendations should be
considered the minimum for a viable production system:

* Processor: 2 CPU cores
* Memory: 4GB
* Disk space (processing): 200GB

More commonly, we deploy the following:

* Processor: 8 CPU cores
* Memory: 16GB

For processing disk space, we recommend allocating 20GB plus four times
the disk space required for the largest transfer that you expect to process. If
your largest transfer is 50GB, allocation at least 220GBs of disk space.

The amount of transfer source disk space needed is subjective, and depends on
individual workflows.

The amount of storage disk space needed will depend on how much material you
intend to store, as well as how it is stored (compressed or uncompressed).

These requirements may not be suitable for certain types of material - for example,
audio-visual material requires more processing power than images or documents.

.. _instructions:

Instructions for new installations
==================================

Archivematica can be installed using packages or Ansible scripts in either
CentOS/Redhat or Ubuntu environments. It can also be installed using Docker.
At this time, installation instructions are provided for officially tested and
supported installation environments:

* :ref:`Automated install from Github on Ubuntu (14.04 and 16.04) <ansible-git-ubuntu>`
* :ref:`Manual install of OS packages on Ubuntu (14.04 and 16.04) <install-ubuntu>`
* :ref:`Manual install of OS packages on CentOS/Redhat <install-pkg-centos>`

Installing Archivematica using :ref:`Docker <docker>` is not officially
supported for production deployments. However, it is the preferred development
environment for those who work on Archivematica's code.

For more information about installation environments, please see the
`ansible-archivematica-src`_ repo, the `deploy-pub`_ repo, and ask on the
`archivematica-tech`_ mailing list for more details.

If you are upgrading from a previous version of Archviematica, please see the
:ref:`upgrading instructions <upgrade>`.

.. _advanced:

Advanced installation options
=============================

There are many ways to install Archivematica, depending on the needs of the
individual user. We have documented some common advanced installation setups.

* :ref:`Installing for development <development>`
* :ref:`Installing across multiple machines <multiple-machines>`
* :ref:`Configure Archivematica with SSL <SSL-support>`

:ref:`Back to the top <installation>`

.. _`archivematica-tech`: https://groups.google.com/forum/#!forum/archivematica-tech
.. _`deploy-pub`: https://github.com/artefactual/deploy-pub
.. _`ansible-archivematica-src`: https://github.com/artefactual-labs/ansible-archivematica-src
.. _`Archivematica google group`: https://groups.google.com/a/artefactual.com/forum/#!forum/archivematica
.. _`docker`: https://github.com/artefactual-labs/am/tree/master/compose
