.. index:: Upgrade Environment Quick Start

.. _Upg_QuickStart:

Upgrade Environment Quick Start
-------------------------------

This guide lists commands required to upgrade a 5.1.1 (5.2.9) environment
to version 7.0 with brief comments. For the detailed description of what
each command does and why, see the following sections:

* :ref:`Detailed upgrade procedure<upg_sol>`
* :ref:`Detailed description of commands<upg_script>`

.. CAUTION::

    Do not run the following commands unless you understand exactly
    what you are doing. It can completely destroy your OpenStack
    environment. Please, read the detailed sections below carefully
    before you proceed with these commands.

Upgrade the Fuel Master node
++++++++++++++++++++++++++++

Upgrade the Fuel installer from version 5.1.1 to version 7.0 using
upgrade tarballs. You will need 3 tarballs:

* upgrade from 5.1.1 (5.2.9) to 6.0
* upgrade from 6.0 to 6.1
* upgrade from 6.1 to 7.0

The sequence of steps for each version of tarball is pretty similar:

* upload the tarball to the Fuel Master node and extract it into temporary
  dir (``/var/tmp/fuel-upgrade-<VERSION>``);
* execute ``upgrade.sh`` wrapper script to start upgrade procedure;
* enter password for ``admin`` user when prompted.

Install packages
++++++++++++++++

Install packages required for managing plugins and other tasks in this
scenario:

::

    yum install -y git python-pip python-paramiko
    pip install pyzabbix

Install Fuel plugins
++++++++++++++++++++

Download required plugins and build them with ``fuel-plugin-builder```:

::

    git clone https://github.com/stackforge/fuel-plugins.git
    pip install fuel-plugins/fuel-plugin-builder
    git clone https://github.com/stackforge/fuel-plugin-external-zabbix/tree/master
    git clone https://github.com/stackforge/fuel-plugin-external-emc/tree/master
    git clone https://github.com/stackforge/fuel-plugin-zabbix-snmptrapd/tree/master
    git clone https://github.com/stackforge/fuel-plugin-zabbix-monitoring-emc
    git clone https://github.com/stackforge/fuel-plugin-zabbix-monitoring-extreme-networks

Build the plugins using ``fpb`` command:

::

    fpb --build fuel-plugin-external-zabbix
    fpb --build fuel-plugin-external-emc
    fpb --build fuel-plugin-zabbix-snmptrapd
    fpb --build fuel-plugin-zabbix-monitoring-emc
    fpb --build fuel-plugin-zabbix-monitoring-extreme-networks

Use ``fuel plugins`` command to install RPM packages that were built:

::

    fuel plugins --install fuel-plugin-external-zabbix/\*.rpm
    fuel plugins --install fuel-plugin-external-emc/\*.rpm
    fuel plugins --install fuel-plugin-zabbix-snmptrapd/\*.rpm
    fuel plugins --install fuel-plugin-zabbix-monitoring-emc/\*.rpm
    fuel plugins --install fuel-plugin-zabbix-monitoring-extreme-networks/\*.rpm

Create a mirror of package repositories
+++++++++++++++++++++++++++++++++++++++

To be able to install OpenStack without having access to Internet from the nodes
in environment, install ``fuel-createmirror`` tool with the following
command:

::

    yum install -y fuel-createmirror

Run the following command to create a local mirror of Ubuntu base repository:

::

    fuel-createmirror -U

Run the following command to create a local mirror of MOS repository:

::

    fuel-createmirror -M

Install the Upgrade Script
++++++++++++++++++++++++++

Run the following command on the Fuel Master node to download and
install the Upgrade Script in the system:

::

    yum install -y fuel-octane

Pick environment to upgrade
+++++++++++++++++++++++++++

Run the following command and pick an environment to upgrade from the
list:

::

    fuel2 env list

Note the ID of the environment and store it in a variable:

::

    export ORIG_ID=<ID>

Create an Upgrade Seed environment
++++++++++++++++++++++++++++++++++

Run the following command to create a new environment of version 7.0
and store its ID to a variable:

::

    SEED_ID=$(octane upgrade-env $ORIG_ID)

Update plugins configuration
____________________________

Execute the following command to synchronize settings of the original
environment with settings of plugins in the Upgrade Seed environment:

::

    octane update-plugin-settings $ORIG_ID $SEED_ID

Sync network groups configuration
_________________________________

Prepare network template by copying it to the current directory and rename
it to ``network_template_${SEED_ID}.yaml``.

Run the following command to upload network template to the Upgrade Seed
cluster:

::

    fuel network-template --env $SEED_ID --upload

Copy network groups from the original environment to the Upgrade Seede
using the following command:

::

    octane sync-networks $ORIG_ID $SEED_ID

Install 7.0 Controllers in isolation
++++++++++++++++++++++++++++++++++++

Pick one of the Controllers in your environment by ID and remember
that ID:

::

    fuel node list --env ${ORIG_ID} | awk -F\| '$7~/controller/{print($0)}'

Use the ID of the Controller to upgrade it with the following command:

::

    octane upgrade-node --isolated $SEED_ID <ID>

Sync Glance images data
+++++++++++++++++++++++

To replicate Glance images from original environment to the Upgrade Seed, use
the following command:

::

    octane sync-images $ORIG_ID $SEED_ID \
        <orig-glance-user> <seed-glance-user> <swift-endpoint>

Replace ``orig-glance-user`` with the name of user for Glance service in the
original environment. Replace ``seed-glance-user`` with the name of user for
Glance service in the Upgrade Seed environment. Replace ``swift-endpoint`` with
URL of swift-proxy in the Upgrade Seed environment.

Start Maintenance window
++++++++++++++++++++++++

At this point we need to place the cloud in Maintenance mode, i.e. block access
to public API endpoints and stop all services that talk to OpenStack state DB.
This is required for dump, restore and upgrade of the DB.

Upgrade State Database
++++++++++++++++++++++

Run the following command to upgrade the state databases of OpenStack services:

::

    octane upgrade-db $ORIG_ID $SEED_ID

Switch control plane to 7.0
+++++++++++++++++++++++++++

Run the following command to switch the OpenStack environment to the
7.0 control plane:

::

    octane upgrade-control $ORIG_ID $SEED_ID

Upgrade Compute nodes
+++++++++++++++++++++

Repeat the following command for every node in the 5.2.9 environment
identified by ID:

::

    octane upgrade-node $SEED_ID <ID>

Uninstall Octane script
+++++++++++++++++++++++

When no nodes remain in the 5.2.9 environment, run the following
command to restore the original state of the 7.0 Fuel Master node:

::

    octane cleanup-fuel

Delete the original 6.1 environment
+++++++++++++++++++++++++++++++++++++

After verification of the upgraded 7.0 environment, delete the
original 5.2.9 environment with the following command:

::

    fuel env --env $ORIG_ID --delete
