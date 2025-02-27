---
slug: wsgi-using-uwsgi-and-nginx-on-fedora-14
deprecated: true
author:
  name: Linode
  email: docs@linode.com
description: 'This guide will show you how to configure the uWSGI server to deploy Python application servers in conjunction with the Nginx web server on Fedora 14.'
keywords: ["uwsgi", "wsgi", "nginx", "python"]
tags: ["web server","python","fedora","nginx"]
license: '[CC BY-ND 4.0](https://creativecommons.org/licenses/by-nd/4.0)'
aliases: ['/websites/nginx/wsgi-using-uwsgi-and-nginx-on-fedora-14/','/web-servers/nginx/wsgi-using-uwsgi-and-nginx-on-fedora-14/','/web-servers/nginx/python-uwsgi/fedora-14/']
modified: 2013-10-07
modified_by:
  name: Linode
published: 2010-12-28
title: WSGI using uWSGI and nginx on Fedora 14
relations:
    platform:
        key: wsgi-uwsgi-nginx
        keywords:
            - distribution: Fedora 14
---



The uWSGI server provides a non-FastCGI method for deploying Python applications with the nginx web server. In coordination with nginx, uWSGI offers great stability, flexibility, and performance. However, to deploy applications with uWSGI and nginx, you must compile nginx manually with the included uwsgi module.

## Set the Hostname

Before you begin installing and configuring the components described in this guide, please make sure you've followed our instructions for [setting your hostname](/docs/getting-started#setting-the-hostname). Issue the following commands to make sure it is set properly:

    hostname
    hostname -f

The first command should show your short hostname, and the second should show your fully qualified domain name (FQDN).

## Install uWSGI

Begin by issuing the following command to update your system and install dependencies for uWSGI:

    yum update
    yum install python-devel libxml2 libxml2-devel python-setuptools zlib-devel wget openssl-devel pcre pcre-devel sudo gcc make autoconf automake

Issue the following sequence of commands to download and compile uWSGI:

    cd /opt/
    wget http://projects.unbit.it/downloads/uwsgi-0.9.6.5.tar.gz
    tar -zxvf uwsgi-0.9.6.5.tar.gz
    mv uwsgi-0.9.6.5/ uwsgi/
    cd uwsgi/
    python setup.py build
    make

Issue the following command to create a dedicated system user to run the uwsgi processes:

    useradd -M -r --shell /bin/sh --home-dir /opt/uwsgi uwsgi

Send the following sequence of commands to set the required file permissions:

    chown -R uwsgi:uwsgi /opt/uwsgi
    touch /var/log/uwsgi.log
    chown uwsgi /var/log/uwsgi.log

## Configure uWSGI

Issue the following command to download an init script to manage the uWSGI process, located at `/etc/init.d/uwsgi`:

    cd /opt/
    wget -O init-rpm.sh init-rpm.sh
    mv /opt/init-rpm.sh /etc/init.d/uwsgi
    chmod +x /etc/init.d/uwsgi

Create an `/etc/default/uwsgi` file to specify specific settings for your Python application. The `MODULE` specifies the name of the Python module that contains your `wsgi` specification. Consider the following example:

{{< file "/etc/default/uwsgi" bash >}}
PYTHONPATH=/srv/www/example.com/application
MODULE=wsgi_configuration_module

{{< /file >}}


If you want to deploy a "Hello World" application, insert the following code into the `/srv/www/example.com/application/wsgi_configuration_module.py` file:

{{< file "/srv/www/example.com/application/wsgi\\_configuration\\_module.py" python >}}
import os
import sys

sys.path.append('/srv/www/example.com/application')

os.environ['PYTHON_EGG_CACHE'] = '/srv/www/example.com/.python-egg'

def application(environ, start_response):
    status = '200 OK'
    output = 'Hello World!'

    response_headers = [('Content-type', 'text/plain'),
                        ('Content-Length', str(len(output)))]
    start_response(status, response_headers)

    return [output]

{{< /file >}}


Issue the following commands to make this init script executable, ensure that uWSGI is restarted following the next reboot sequence, and start the service:

    chkconfig --add uwsgi
    chkconfig uwsgi on
    /etc/init.d/uwsgi start

## Install nginx

Issue the following commands to install and configure the nginx web server:

    yum install nginx
    chkconfig --add nginx
    chkconfig nginx on
    /etc/init.d/nginx start

## Configure nginx

Create an nginx server configuration that resembles the following for the site where the uWSGI app will be accessible:

{{< file "nginx virtual host configuration" nginx >}}
server {
    listen   80;
    server_name www.example.com example.com;
    access_log /srv/www/example.com/logs/access.log;
    error_log /srv/www/example.com/logs/error.log;

    location / {
        include        uwsgi_params;
        uwsgi_pass     127.0.0.1:9001;
    }

    location /static {
        root   /srv/www/example.com/public_html/static/;
        index  index.html index.htm;
    }
}

{{< /file >}}


All requests to URLs ending in `/static` will be served directly from the `/srv/www/example.com/public_html/static` directory. Restart the web server by issuing the following command:

    /etc/init.d/nginx restart

## Additional Application Servers

If the Python application you've deployed requires more application resources than a single Linode instance can provide, all of the methods for deploying a uWSGI application server are easily scaled to rely on multiple uSWGI instances that run on additional Linodes with the request load balanced using nginx's `upstream` capability. See our documentation of [proxy and software load balancing with nginx](/docs/uptime/loadbalancing/how-to-use-nginx-as-a-front-end-proxy-server-and-software-load-balancer) for more information. For a basic example configuration, see the following example:

{{< file "nginx configuration" nginx >}}
upstream uwsgicluster {
     server 127.0.0.1:9001;
     server 192.168.100.101:9001;
     server 192.168.100.102:9001;
     server 192.168.100.103:9001;
     server 192.168.100.104:9001;
}

server {
    listen   80;
    server_name www.example.com example.com;
    access_log /srv/www/example.com/logs/access.log;
    error_log /srv/www/example.com/logs/error.log;

    location / {
        include        uwsgi_params;
        uwsgi_pass     uwsgicluster;
    }

    location /static {
        root   /srv/www/example.com/public_html/static/;
        index  index.html index.htm;
    }
}

{{< /file >}}


In this example, we create the `uwsgicluster` upstream, which has five components. One runs on the local interface, and four run on the local network interface of distinct Linodes (the `192.168.` addresses or the private "back end" network). The application servers that run on those dedicated application servers are identical to the application servers described above. However, the application server process must be configured to bind to the appropriate network interface to be capable of responding to requests.

## More Information

You may wish to consult the following resources for additional information on this topic. While these are provided in the hope that they will be useful, please note that we cannot vouch for the accuracy or timeliness of externally hosted materials.

- [Installing Nginx on Fedora 14](/docs/web-servers/nginx/installation/fedora-14)
- [Deploy a LEMP Server on Fedora 14](/docs/lemp-guides/fedora-14/)
- [Configure nginx Proxy Servers](/docs/uptime/loadbalancing/how-to-use-nginx-as-a-front-end-proxy-server-and-software-load-balancer)
