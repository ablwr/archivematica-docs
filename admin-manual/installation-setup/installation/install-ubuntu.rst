.. _install-ubuntu:

==================================
Installing Archivematica on Ubuntu
==================================

Archivematica packages are hosted at packages.archivematica.org as a central
place to store packages for multiple operating systems. Packages for both Ubuntu
14.04 and 16.04 are available.

.. note:: Ubuntu 14.04 is currently in maintenance updates only mode and will
   cease to be officially supported in April 2019. We recommend upgrading to
   Ubuntu 16.04.

*On this page*

* :ref:`Installation instructions for Ubuntu 16.04 (Xenial) <xenial-instructions>`
* :ref:`Installation instructions for Ubuntu 14.04 (Trusty) <trusty-instructions>`
* :ref:`Post-installation configuration <ubuntu-post-install-config>`

.. _xenial-instructions:

Ubuntu 16.04 (Xenial) installation instructions
===============================================

1. Add packages.archivematica.org to your list of trusted repositories.

   Run these three commands right now (**and delete this section when the final
   release packages are made**):

   .. code:: bash

      sudo wget -O - https://packages.archivematica.org/1.7.x/key.asc | sudo apt-key add -
      sudo wget -O - http://jenkins-ci.archivematica.org/repos/devel.key | sudo apt-key add -
      sudo sh -c 'echo "deb http://jenkins-ci.archivematica.org/repos/apt/release-0.11-xenial/ ./" >> /etc/apt/sources.list'
      sudo sh -c 'echo "deb http://jenkins-ci.archivematica.org/repos/apt/release-1.7-xenial/ ./" >> /etc/apt/sources.list'
      sudo sh -c 'echo "deb [arch=amd64] http://packages.archivematica.org/1.7.x/ubuntu-externals xenial main" >> /etc/apt/sources.list'

   Run these three commands when the final release packages are made:

   .. code:: bash

      sudo wget -O - https://packages.archivematica.org/1.7.x/key.asc  | sudo apt-key add -
      sudo sh -c 'echo "deb [arch=amd64] http://packages.archivematica.org/1.7.x/ubuntu xenial main" >> /etc/apt/sources.list'
      sudo sh -c 'echo "deb [arch=amd64] http://packages.archivematica.org/1.7.x/ubuntu-externals xenial main" >> /etc/apt/sources.list'

2. Add Elasticsearch package source (optional). Elasticsearch comes from its own
   package repository.

   .. note:: Skip this step if you are planning to run Archivematica in indexless
      mode (without Elasticsearch).

   .. code:: bash

      sudo wget -O - http://packages.elasticsearch.org/GPG-KEY-elasticsearch | sudo apt-key add -
      sudo sh -c 'echo "deb http://packages.elasticsearch.org/elasticsearch/1.7/debian stable main" >> /etc/apt/sources.list'

   At this point you will need to restart dashboard and mcp-server services

   .. code:: bash

      sudo service archivematica-dashboard restart
      sudo service archivematica-mcp-server restart


3. Update to the most recent OS release - for Xenial, this is 16.04.4. This step
   will also fetch a list of the software from the package repositories you just
   added to your system.

   .. code:: bash

      sudo apt-get update
      sudo apt-get upgrade

4. Install Elasticsearch (optional)

   .. note:: Skip this step if you are planning to run Archivematica in indexless
      mode (without Elasticsearch).

   .. code:: bash

      sudo apt-get install elasticsearch

5. Install the Storage Service package.

   .. code:: bash

      sudo apt-get install -y archivematica-storage-service

6. Configure the Storage Service.

   .. code:: bash

      sudo rm -f /etc/nginx/sites-enabled/default
      sudo ln -s /etc/nginx/sites-available/storage /etc/nginx/sites-enabled/storage

   .. warning:: If you are planning to use the `Sword API`_ of the Archivematica
      Storage Service, then (due to a `known issue`_), you must instruct
      Gunicorn to use the ``sync`` worker class:

   .. code:: bash

      sudo sh -c 'echo "SS_GUNICORN_WORKER_CLASS=sync" >> /etc/default/archivematica-storage-service'

7. Update pip. This is used to install Python dependencies for both the Storage
   Service and the dashboard. There is a `known issue with pip`_ on Ubuntu 14.04
   that makes this step necessary. This step is optional on Ubuntu 16.04, but is
   still a good idea to get the most recent version of pip.

   .. code:: bash

      sudo wget https://bootstrap.pypa.io/get-pip.py
      sudo python get-pip.py

8. Install the Archivematica packages. The order of installation is important -
   the mcp-server package must be installed before the dashboard package. While
   it is possible to install the mcp-client package on a separate machine, that
   configuration is not documented in these instructions.

   The mcp-server package will install MySQL and configure the database used by
   Archivematica. Depending on the version of MySQL that gets installed the
   prompts you will see may differ. In all cases, you will be prompted to create
   a password for the 'root' user. Keep note of the password you create.
   On Ubuntu 14.04, MySQL 5.5 is installed, and
   the default 'archivematica' database user is automatically created with a
   default password of 'demo'. On Ubuntu 16.04, MySQL 5.7 is installed, and
   you are prompted to add a password for the 'archivematica' user. You must
   use 'demo' as the password during the install process. The password can be
   changed after the installation is complete.

   The fits package needs to be installed as a prerequisite, due to a
   circular dependencies problem that arises only on Ubuntu 14.04.

   .. code:: bash

      sudo apt-get install -y archivematica-mcp-server
      sudo apt-get install -y archivematica-dashboard
      sudo apt-get install -y fits
      sudo apt-get install -y archivematica-mcp-client

9. Configure the Archivematica components (optional). There are a number of
   environment variables that Archivematica recognizes which can be used to
   alter how it is configured. For the full list, see the
   `Dashboard install README`_, the `MCPClient install README`_, and the
   `MCPServer install README`_.

   .. note:: If you are planning on running Archivematica in indexless mode (i.e.
      without Elasticsearch), then modify the relevant systemd EnvironmentFile
      files by adding lines that set the relevant environment variables to ``false``:

   .. code:: bash

      sudo sh -c 'echo "ARCHIVEMATICA_DASHBOARD_DASHBOARD_SEARCH_ENABLED=false" >> /etc/default/archivematica-dashboard'
      sudo sh -c 'echo "ARCHIVEMATICA_MCPSERVER_MCPSERVER_SEARCH_ENABLED=false" >> /etc/default/archivematica-mcp-server'
      sudo sh -c 'echo "ARCHIVEMATICA_MCPCLIENT_MCPCLIENT_SEARCH_ENABLED=false" >> /etc/default/archivematica-mcp-client'

10. Configure the dashboard.

    .. code:: bash

       sudo ln -s /etc/nginx/sites-available/dashboard.conf /etc/nginx/sites-enabled/dashboard.conf

11. Start Elasticsearch (optional).

    .. note:: Skip this step if you are planning to run Archivematica in indexless
       mode (without Elasticsearch).

    .. code:: bash

       sudo service elasticsearch restart
       sudo update-rc.d elasticsearch defaults 95 10

12. Start the remaining services

    .. code:: bash

       sudo service clamav-freshclam restart
       sudo service clamav-daemon start
       sudo service gearman-job-server restart
       sudo service archivematica-mcp-server start
       sudo service archivematica-mcp-client start
       sudo service archivematica-storage-service start
       sudo service archivematica-dashboard start
       sudo service nginx restart
       sudo systemctl enable fits
       sudo service fits start

    If you have trouble with the gearman command try restarting it:

    .. code:: bash

       sudo service gearman-job-server restart

13. Complete :ref:`Post Install Configuration <post-install-config>`.

.. _trusty-instructions:

Ubuntu 14.04 (Trusty) installation instructions
===============================================

1. Add packages.archivematica.org to your list of trusted repositories.

   Run these three commands right now (**and delete this section when the final
   release packages are made.**):

   .. code:: bash

      sudo wget -O - https://packages.archivematica.org/1.7.x/key.asc | sudo apt-key add -
      sudo wget -O - http://jenkins-ci.archivematica.org/repos/devel.key | sudo apt-key add -
      sudo sh -c 'echo "deb http://jenkins-ci.archivematica.org/repos/apt/release-0.11-trusty/ ./" >> /etc/apt/sources.list'
      sudo sh -c 'echo "deb http://jenkins-ci.archivematica.org/repos/apt/release-1.7-trusty/ ./" >> /etc/apt/sources.list'
      sudo sh -c 'echo "deb [arch=amd64] http://packages.archivematica.org/1.7.x/ubuntu-externals trusty main" >> /etc/apt/sources.list'

   Run these three commands when the final release packages are made:

   .. code:: bash

      sudo wget -O - https://packages.archivematica.org/1.7.x/key.asc  | sudo apt-key add -
      sudo sh -c 'echo "deb [arch=amd64] http://packages.archivematica.org/1.7.x/ubuntu trusty main" >> /etc/apt/sources.list'
      sudo sh -c 'echo "deb [arch=amd64] http://packages.archivematica.org/1.7.x/ubuntu-externals trusty main" >> /etc/apt/sources.list'


2. Add Elasticsearch package source (optional). Elasticsearch comes from its own
   package repository.

   .. note:: Skip this step if you are planning to run Archivematica in indexless
      mode (without Elasticsearch).

   .. code:: bash

      sudo wget -O - http://packages.elasticsearch.org/GPG-KEY-elasticsearch | sudo apt-key add -
      sudo sh -c 'echo "deb http://packages.elasticsearch.org/elasticsearch/1.7/debian stable main" >> /etc/apt/sources.list'

   At this point you will need to restart dashboard and mcp-server services

   .. code:: bash

      sudo service archivematica-dashboard restart
      sudo service archivematica-mcp-server restart

3. Update to the most recent OS release - for Trusty, this is 14.04.5. This
   step will also fetch a list of the software from the package repositories you
   just added to your system.

   .. code:: bash

      sudo apt-get update
      sudo apt-get upgrade

4. Install Elasticsearch (optional)

   .. note:: Skip this step if you are planning to run Archivematica in indexless
      mode (without Elasticsearch).

   .. code:: bash

      sudo apt-get install elasticsearch

5. Install the Storage Service package.

   .. code:: bash

      sudo apt-get install -y archivematica-storage-service

6. Configure the Storage Service.

   .. code:: bash

      sudo rm -f /etc/nginx/sites-enabled/default
      sudo ln -s /etc/nginx/sites-available/storage /etc/nginx/sites-enabled/storage

   .. warning:: If you are planning to use the `Sword API`_ of the Archivematica
      Storage Service, then (due to a `known issue`_), you must instruct
      Gunicorn to use the ``sync`` worker class:

   .. code:: bash

      sudo sh -c 'echo "SS_GUNICORN_WORKER_CLASS=sync" >> /etc/default/archivematica-storage-service'

7. Update pip. This is used to install Python dependencies for both the Storage
   Service and the dashboard. There is a `known issue with pip`_ on Ubuntu 14.04
   that makes this step necessary. This step is optional on Ubuntu 16.04, but is
   still a good idea to get the most recent version of pip.

   .. code:: bash

      sudo wget https://bootstrap.pypa.io/get-pip.py
      sudo python get-pip.py

8. Install the Archivematica packages. The order of installation is important -
   the mcp-server package must be installed before the dashboard package. While
   it is possible to install the mcp-client package on a separate machine, that
   configuration is not documented in these instructions.

   The mcp-server package will install MySQL and configure the database used by
   Archivematica. Depending on the version of MySQL that gets installed the
   prompts you will see may differ. In all cases, you will be prompted to create
   a password for the 'root' user. Keep note of the password you create.
   On Ubuntu 14.04, MySQL 5.5 is installed, and
   the default 'archivematica' database user is automatically created with a
   default password of 'demo'. On Ubuntu 16.04, MySQL 5.7 is installed, and
   you are prompted to add a password for the 'archivematica' user. You must
   use 'demo' as the password during the install process. The password can be
   changed after the installation is complete.

   The fits package needs to be installed as a prerequisite, due to a
   circular dependencies problem that arises only on Ubuntu 14.04.

   .. code:: bash

      sudo apt-get install -y archivematica-mcp-server
      sudo apt-get install -y archivematica-dashboard
      sudo apt-get install -y fits
      sudo apt-get install -y archivematica-mcp-client

9. Configure the Archivematica components (optional). There are a number of
   environment variables that Archivematica recognizes which can be used to
   alter how it is configured. For the full list, see the
   `Dashboard install README`_, the `MCPClient install README`_, and the
   `MCPServer install README`_.

   .. note:: If you are planning on running Archivematica in indexless mode (i.e.
      without Elasticsearch), then modify the relevant systemd EnvironmentFile
      files by adding lines that set the relevant environment variables to ``false``:

   .. code:: bash

      sudo sh -c 'echo "ARCHIVEMATICA_DASHBOARD_DASHBOARD_SEARCH_ENABLED=false" >> /etc/default/archivematica-dashboard'
      sudo sh -c 'echo "ARCHIVEMATICA_MCPSERVER_MCPSERVER_SEARCH_ENABLED=false" >> /etc/default/archivematica-mcp-server'
      sudo sh -c 'echo "ARCHIVEMATICA_MCPCLIENT_MCPCLIENT_SEARCH_ENABLED=false" >> /etc/default/archivematica-mcp-client'

10. Configure the dashboard.

    .. code:: bash

       sudo ln -s /etc/nginx/sites-available/dashboard.conf /etc/nginx/sites-enabled/dashboard.conf

11. Start Elasticsearch (optional).

    .. note:: Skip this step if you are planning to run Archivematica in indexless
       mode (without Elasticsearch).

    .. code:: bash

       sudo service elasticsearch restart
       sudo update-rc.d elasticsearch defaults 95 10

12. Start the remaining services

    .. code:: bash

       sudo service clamav-freshclam restart
       sudo service clamav-daemon start
       sudo service gearman-job-server restart
       sudo service archivematica-mcp-server start
       sudo service archivematica-mcp-client start
       sudo service archivematica-storage-service start
       sudo service archivematica-dashboard start
       sudo service nginx restart
       sudo systemctl enable fits
       sudo service fits start

    If you have trouble with the gearman command try restarting it:

    .. code:: bash

       sudo service gearman-job-server restart

13. Complete :ref:`Post Install Configuration <post-install-config>`.

.. _ubuntu-post-install-config:

Post-install configuration
--------------------------

After successfully completing a new installation, follow these steps to complete
the configuration of your new server.

1. The Storage Service runs as a separate web application from the Archivematica
   dashboard. Go to the following link in a web browser and log in as user
   *test* with the password *test*: http://localhost:8000, or use the IP address
   of the machine you have been installing on.

   If you are running the storage service and the dashboard on the same host you
   should use:

   .. code:: bash

      localhost

   or

   .. code:: bash

      127.0.0.1

   If you are using a public IP address you'll need to configure your firewall
   rules and allow access only to port 80 and 8000 for Archivematica usage.

2. Create a new administrative user in the Storage Service. The Storage Service
   has its own set of users. Navigate to Administrators > Users and add at
   least one administrative user. We also recommend modifying the test user and
   changing the default password. After you have created an administrative user,
   copy the user's API key to your clipboard.

3. Log in to the Archivematica dashboard to finish the installation in a
   web browser: http://localhost. Again, you can use the IP address of the
   machine you have been installing on.

4. On the Welcome page, create an administrative user for the Archivematica
   pipeline by entering the organization name, the organization identifier,
   username, email, and password.

5. On the next screen, connect your pipeline to the Storage Service by entering
   the Storage Service URL and User and pasting the API key that you copied in
   Step 2.

:ref:`Back to the top <install-ubuntu>`

.. _`known issue`: https://github.com/artefactual/archivematica-storage-service/issues/312
.. _`Sword API`: https://wiki.archivematica.org/Sword_API
.. _`known issue with pip`: https://bugs.launchpad.net/ubuntu/+source/python-pip/+bug/1658844
.. _`Dashboard install README`: https://github.com/artefactual/archivematica/blob/stable/1.7.x/src/dashboard/install/README.md
.. _`MCPClient install README`: https://github.com/artefactual/archivematica/blob/stable/1.7.x/src/MCPClient/install/README.md
.. _`MCPServer install README`: https://github.com/artefactual/archivematica/blob/stable/1.7.x/src/MCPServer/install/README.md
