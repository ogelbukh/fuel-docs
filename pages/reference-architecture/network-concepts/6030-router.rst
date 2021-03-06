
.. _router-plan:

Router
------

Your network must have an IP address in the Public :ref:`Logical Networks
<logical-networks-arch>` set on a router port as an "External Gateway".
Without this, your VMs are unable to access the outside world. In many
of the examples provided in these documents, that IP is :ref:`12.0.0.1 in VLAN 101 <conf_netw>`.

If you add a new router, be sure to set its gateway IP:

.. image:: /_images/new_router.png

.. image:: /_images/set_gateway.png

The Fuel UI includes a field on the networking tab for the gateway address.
When OpenStack deployment starts,
the network on each node is reconfigured
to use this gateway IP address as the default gateway.

If Floating addresses are from another L3 network,
then you must configure the IP address
(or multiple IPs if Floating addresses are from more than one L3 network)
for them on the router as well.
Otherwise, Floating IPs on nodes will be inaccessible.

Consider the following routing recommendations
when you configure your network:

- Use the default routing via a router in the Public network
- Use the the management network to access your management
  infrastructure (L3 connectivity if necessary)
- The Storage and VM networks should be configured without access to
  other networks (no L3 connectivity)

