---
layout: post
title: Flask and WSGI on Centos 6
date: '2016-01-09 19:00:00'
---

In order to run a flask app with Apache and WSGI you will need to install the correct packages.

```
$ yum install httpd mod_wsgi policycoreutils-python python-flask python-jinja
```

Note that in order to install the flask and jinja packages you will need to have the EPEL 6 repository configured on your server. 

Once the packages are installed you will also need to change SELinux settings to ensure that the python application is able to execute. 

```
$ semanage boolean --modify --on httpd_execmem
```

Install your flask application into a directory under `/var/www` such as `/var/www/app`.  Next you will need to create a wsgi file that will be used for wsgi to determine the entrypoint into your application. 

```python
import sys sys.path.insert(0, '/var/www/app')
from flaskapp import app as application
```

You will also need a configuration file for apache to tell apache how to run your WSGI application.  This needs to be written to the `/etc/httpd/conf.d/` directory to a file like `app.conf`.

```apache 
WSGISocketPrefix /var/run/wsgi 
<VirtualHost *:80>
  ServerName example.com
  WSGIDaemonProcess flaskapp user=apache group=apache threads=5
  WSGIScriptAlias / /var/www/app/flaskapp.wsgi
  <Directory /var/www/app>
    WSGIProcessGroup flaskapp
    WSGIApplicationGroup %{GLOBAL}
    Order deny,allow
    Allow from all
  </Directory>
</VirtualHost>
```

Note that you must include the WSGISocketPrefix so that the socket used for WSGI is put in the `/var/run/` directory.  If this isn't included then the socket will be created in `/var/log/httpd/` instead.  This will cause permission denied errors when attempting to connect to the socket since your apache user does not have access to that directory. 
Last, to ensure that port 80 is accessible you will need to modify your firewall rules.  The default firewall for Centos 6 has this port closed.

``` 
$ iptables -I INPUT 4 -p tcp -m state --state NEW --dport 80 -j ACCEPT
```