.. _glance-rn:

OpenStack image (Glance)
------------------------

Resolved glance issues
++++++++++++++++++++++

* In case of slow connection to the external storage, Glance
  was not able to upload an image. It happened because of a one-minute
  timeout that broke connection if 16 MB chunk had not been
  downloaded within that time period. To prevent these timeouts,
  the chunk size is reduced to 4 MB. See `LP1401118`_.

* The token caching was disabled for ``glance-api`` to prevent
  authentication errors. See `LP1443913`_.

Known Glance issues
+++++++++++++++++++

* A big image cannot be uploaded, as well as an instance snapshot
  fails if the upload image time is longer than the Keystone token
  lifespan. Glance returns an authentication failure indicating that
  the token is invalid. See `LP1456573`_ and `LP1441156`_.

* After all the Ceph compute nodes destruction followed by their
  recreation and the environment redeployment, ``glance-api`` stops
  responding (with an HTTP 503 error). As a workaround, restart the
  ``glance-api`` service. For more information, see `LP1459743`_.

.. _`LP1401118`: https://bugs.launchpad.net/mos/+bug/1401118
.. _`LP1443913`: https://bugs.launchpad.net/fuel/7.0.x/+bug/1443913
.. _`LP1456573`: https://bugs.launchpad.net/mos/7.0.x/+bug/1456573
.. _`LP1441156`: https://bugs.launchpad.net/fuel/6.0.x/+bug/1441156
.. _`LP1459743`: https://bugs.launchpad.net/fuel/+bug/1459743