.. index:: HowTo: Configure Horizon on HTTPS

.. _configure-https-horizon-op:

HowTo: Configure Horizon on HTTPS
===========================================

Horizon can be configured in HTTPS modality following the
`official Horizon guide <http://docs.openstack.org/juno/config-reference/content/configure-dashboard.html>`_
but in Mirantis OpenStack environments with
High Availability (HA), some further steps are required.
In this three-controllers example, Horizon will be configured with
SSL on the openstack.example.com domain.

.. warning:: If you are using Mirantis OpenStack on a
   production server, the recommendation is to purchase
   a certificate from a Trusted Certificate Authority

#. On any of the Controllers, generate the self-signed key and certificate:
   ::

       openssl genrsa -out openstack.example.com.key 2048
       openssl req -new -key openstack.example.com.key -out openstack.example.com.csr
       openssl x509 -req -days 365 -in openstack.example.com.csr \
       -signkey openstack.example.com.key -out openstack.example.com.crt

#. Copy these files to the ``/etc/ssl/certs`` directory on all Сontrollers.

#. On every Сontroller, enable the ``mod_ssl`` Apache module.
   For CentOS, install ``mod_ssl``:
   ::

       yum install mod_ssl openssl

   For Ubuntu, enable it:
   ::

       a2enmod ssl
       a2enmod rewrite

#. On every Controller, enable the binding on the 443 port.
   For CentOS, add this line to the ``/etc/httpd/conf/ports.conf`` file:
   ::

       NameVirtualHost *:443

   For Ubuntu, no steps are required, as the binding is already present
   when the SSL module is enabled.

#. On every Controller, modify
   the ``/etc/httpd/conf.d/10-horizon_vhost.conf`` file on CentOS or
   the ``/etc/apache2/conf.d/openstack-dashboard.conf`` file on Ubuntu.
   You can follow this template for CentOS:

   ::

      <VirtualHost *:80>
        ServerName openstack.example.com
        <IfModule mod_rewrite.c>
          RewriteEngine On
          RewriteCond %{HTTPS} off
          RewriteRule (.*) https://%{HTTP_HOST}%{REQUEST_URI}
        </IfModule>
        <IfModule !mod_rewrite.c>
          RedirectPermanent / https://openstack.example.com
        </IfModule>
      </VirtualHost>

      <VirtualHost *:443>
        ServerName openstack.example.com

        ## Vhost docroot
        DocumentRoot "/var/www/"
        ## Alias declarations for resources outside the DocumentRoot
        Alias /static "/usr/share/openstack-dashboard/static"

        ## Directories, there should at least be a declaration for /var/www/

        <Directory "/var/www/">
          Options Indexes FollowSymLinks MultiViews
          AllowOverride None
          Order allow,deny
          Allow from all
        </Directory>

        ## Logging
        ErrorLog "/var/log/httpd/horizon_error.log"
        ServerSignature Off
        CustomLog "/var/log/httpd/horizon_access.log" combined

        ## RedirectMatch rules
        RedirectMatch permanent  ^/$ /dashboard

        ## Server aliases
        ServerAlias node-1.domain.tld
        WSGIDaemonProcess dashboard group=apache processes=2 threads=9 user=apache
        WSGIProcessGroup dashboard
        WSGIScriptAlias /dashboard \
            "/usr/share/openstack-dashboard/openstack_dashboard/wsgi/django.wsgi"

        ## Custom fragment

        <Directory /usr/share/openstack-dashboard/openstack_dashboard/wsgi>
          <IfModule mod_deflate.c>
            SetOutputFilter DEFLATE
            <IfModule mod_headers.c>
              # Make sure proxies dont deliver the wrong content
              Header append Vary User-Agent env=!dont-vary
            </IfModule>
          </IfModule>
          Order allow,deny
          Allow from all
        </Directory>

        <Directory /usr/share/openstack-dashboard/static>
          <IfModule mod_expires.c>
            ExpiresActive On
            ExpiresDefault "access 6 month"
          </IfModule>
          <IfModule mod_deflate.c>
            SetOutputFilter DEFLATE
          </IfModule>
          Order allow,deny
          Allow from all
        </Directory>

        ## Enable SSL

        SSLEngine On
        SSLCertificateFile /etc/ssl/certs/openstack.example.com.crt
        SSLCACertificateFile /etc/ssl/certs/openstack.example.com.crt
        SSLCertificateKeyFile /etc/ssl/certs/openstack.example.com.key
        SetEnvIf User-Agent ".*MSIE.*" nokeepalive ssl-unclean-shutdown
      </VirtualHost>

   And this one for Ubuntu:

   ::

      <VirtualHost *:80>
        ServerName openstack.example.com
        <IfModule mod_rewrite.c>
          RewriteEngine On
          RewriteCond %{HTTPS} off
          RewriteRule (.*) https://%{HTTP_HOST}%{REQUEST_URI}
        </IfModule>
        <IfModule !mod_rewrite.c>
          RedirectPermanent / https://openstack.example.com
        </IfModule>
      </VirtualHost>

      <VirtualHost *:443>
        ServerName openstack.example.com

        ## Vhost docroot
        DocumentRoot "/var/www/"
        ## Alias declarations for resources outside the DocumentRoot
        Alias /static "/usr/share/openstack-dashboard/static"

        ## Directories, there should at least be a declaration for /var/www/

        <Directory "/var/www/">
          Options Indexes FollowSymLinks MultiViews
          AllowOverride None
          Order allow,deny
          Allow from all
        </Directory>

        ## Logging
        ErrorLog "/var/log/horizon/horizon_error.log"
        ServerSignature Off
        CustomLog "/var/log/horizon/horizon_access.log" combined

        ## RedirectMatch rules
        RedirectMatch permanent  ^/$ /dashboard

        ## Server aliases
        ServerAlias node-1.domain.tld
        WSGIScriptAlias /horizon \
            /usr/share/openstack-dashboard/openstack_dashboard/wsgi/django.wsgi
        WSGIDaemonProcess horizon user=horizon group=horizon processes=3 threads=10
        WSGIProcessGroup horizon

        ## Custom fragment

        <Directory /usr/share/openstack-dashboard/openstack_dashboard/wsgi>
          <IfModule mod_deflate.c>
            SetOutputFilter DEFLATE
            <IfModule mod_headers.c>
              # Make sure proxies dont deliver the wrong content
              Header append Vary User-Agent env=!dont-vary
            </IfModule>
          </IfModule>
          Order allow,deny
          Allow from all
        </Directory>

        <Directory /usr/share/openstack-dashboard/static>
          <IfModule mod_expires.c>
            ExpiresActive On
            ExpiresDefault "access 6 month"
          </IfModule>
          <IfModule mod_deflate.c>
            SetOutputFilter DEFLATE
          </IfModule>
          Order allow,deny
          Allow from all
        </Directory>

        ## Enable SSL

        SSLEngine On
        SSLCertificateFile /etc/ssl/certs/openstack.example.com.crt
        SSLCACertificateFile /etc/ssl/certs/openstack.example.com.crt
        SSLCertificateKeyFile /etc/ssl/certs/openstack.example.com.key
        #SetEnvIf User-Agent ".*MSIE.*" nokeepalive ssl-unclean-shutdown
      </VirtualHost>

#. On every Controller, restart Apache:
   For CentOS:
   ::

      service httpd restart

   For Ubuntu:
   ::

      service apache2 restart

#. On every Controller, configure HAProxy enabling SSL. In this example,
   a pool of three servers is used. Modify ``/etc/haproxy/haproxy.cfg``,
   adding this section:

   ::

      frontend horizon-ssl
        bind <external-virtual-ip>:443
        balance roundrobin
        mode http
        option ssl-hello-chk
        server node-1 <node-1-ip>:443 check
        server node-2 <node-2-ip>:443 check
        server node-3 <node-3-ip>:443 check

#. On every Controller, restart HAProxy:
   ::

      service haproxy restart
