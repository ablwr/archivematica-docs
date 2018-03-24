.. _install-atom:

Using AtoM 2.x with Archivematica
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Archivematica |version| has been tested with and is recommended for use with the
latest version of AtoM. AtoM version 2.2 or higher is required for use with the
hierarchical DIP functionality; see :ref:`Arrange a SIP from backlog <arrange-sip>`.

Installation instructions for AtoM 2 are available on the
:ref:`accesstomemory.org documentation <atom:home>`. When following those
instructions, it is best to download AtoM from the git repository (rather than
use one of the supplied tarballs).

Once you have a working AtoM installation, you can configure dip upload
between Archivematica and AtoM. The basic steps are:

* Update AtoM dip upload configuration in the Archivematica dashboard

* Confirm atom-worker is configured on the AtoM server (copy the atom-
  worker.conf file from AtoM source to /etc/init/)

* Enable the Sword Plugin in the AtoM plugins page

* Enable job scheduling in the AtoM settings page (AtoM version 2.1 or lower only)

* Confirm gearman is installed on the AtoM server

* Configure ssh keys to allow rsync to work for the archivematica user, from
  the Archivematica server to the AtoM server

* Start gearman on the AtoM server

* Start the AtoM worker on the AtoM server
