
.. _name-distro-ug:

Name Environment and Choose Distribution
----------------------------------------

When you click on the "New OpenStack Environment" icon
on the Fuel UI, the following screen is displayed:

.. image:: /_images/user_screen_shots/name_environ.png

Give the environment a name
and select the Linux distribution from the drop-down list:

::

    Kilo on Ubuntu 14.04.1 (2015.1.0-7.0) (default)

This is the operating system that will be installed
on the target nodes in the environment.

The number in parentheses
is the version number for the environment;
it is formed by concatenating the Kilo Release number
and the Mirantis OpenStack Release number.
In this case, the "2015.1" string corresponds to the Kilo release version;
the "7.0" string is the Mirantis OpenStack release number.

Note that the list displayed under the "Releases" tab
at the top of the Fuel home page
lists all the releases that Fuel 7.0 can manage
in your environment.
If you upgraded Fuel
from an earlier Mirantis OpenStack release,
Fuel 7.0 can manage environments that were previously deployed
using those releases.
It cannot, however, deploy a new environment using those releases.
