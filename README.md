# Linux Server Configuration Project:

This project takes a baseline installation of a Linux distribution on a virtual
machine and prepares it to host a web application and includes installing
updates, securing it from a number of attack vectors and installing/configuring
web and database servers.

## Server Connection Details
* Public IP Address: 13.59.25.207
* Port: 2200
* User: grader

## Webserver URL
* URL: http://13.59.25.207/

## Software Installed
* `sudo apt-get update`
* `sudo apt-get upgrade`
* `sudo apt-get install apache2`
* `sudo apt-get install libapache2-mod-wsgi`
* `sudo apt-get install postgresql postgresql-contrib`
* `sudo apt-get install python-pip`
* `sudo pip install SQLAlchemy`
* `sudo pip install Flask`
* `sudo pip install oauth2client`
* `sudo pip install requests`
* `sudo pip install psycopg2`

## Configurations Made

### Setup Firewall
* `sudo ufw status`
* `sudo ufw allow 2200/tcp`
* `sudo ufw allow www`
* `sudo ufw allow ntp`
* `sudo ufw enable`
* Note: also required adding/removing the same ports to the Amazon Lightsail Firewall

### Disable Password Login
* `sudo vim /etc/ssh/sshd_config`
* changed PasswordAuthentication to no
* changed PermitRootLogin to no
* removed Port 22
* added Port 2200
* `sudo service ssh restart`

### Setup Grader User & Permissions
* `sudo adduser grader`
* set pwd as grader
* `su - grader`
* `mkdir .ssh`
* `chmod 700 .ssh`
* Created grader rsa token on local machine
* `vim authorized_users`
* added rsa.pub token to grader:~/.ssh/authorized_users file
* `chmod 644 authorized_user`
* `sudo usermod -a -G adm grader`

### Add Grader to Sudoers
* `sudo cp /rtc/sudoers.d/90-cloud-init-users grader`
* edited file grader to modify user to grader

### Install Catalog App
* `mkdir /scripts`
* `cd /scripts`
* `git clone https://github.com/lowjack98/fullstack-nanodegree-vm.git`
* `sudo mkdir /var/www/catalog`
* `sudo cp -R /scripts/fullstack-nanodegree-vm/vagrant/catalog /var/www/catalog`
* `cd var/www/catalog/catalog`
* `sudo cp application.py __init__.py`
* `sudo vim /var/www/catalog/catalog.wsgi`
* add the following to that file:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")
from catalog import app as application
application.secret_key = 'super_secret_ninja_key'
```

### Setup Apache to serve python application
* modified /etc/apache2/sites-enabled/000-default.conf
```
<VirtualHost *:80>
        ServerName 13.59.25.207
        ServerAdmin admin@mywebsite.com
        WSGIScriptAlias / /var/www/catalog/catalog.wsgi
        <Directory /var/www/catalog/catalog/>
                Order allow,deny
                Allow from all
        </Directory>
        Alias /static /var/www/catalog/catalog/static
        <Directory /var/www/catalog/catalog/static/>
                Order allow,deny
                Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
* `sudo apache2ctl restart`

### Setup Postgre SQL DB
* `sudo -u postgres createuser --interactive`
*	create user catalog, not superuser
* `sudo -u postgres createdb catalog`
* `sudo -u postgres psql`
* `\connect catalog`
* `GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`
* `ALTER USER catalog WITH PASSWORD 'catalog';`
* `sudo vim pg_hba.conf`
* add: `local   catalog         catalog                                 password`
* `sudo service postgresql restart`

### Modify Catalog App to user Postgre SQL DB
* modified __init__.py
  * added full path name to client_secret.json in both places it was listed
  *	modified dbengine to: `engine = create_engine("postgresql+psycopg2://catalog:catalog@/catalog")`

* modified database_setup.py
  * changed auth_id Column from Integer to String(80)
  *	modified dbengine to: `engine = create_engine("postgresql+psycopg2://catalog:catalog@/catalog")`

* modified initialize_db.py
  * modified dbengine to: `engine = create_engine("postgresql+psycopg2://catalog:catalog@/catalog")`

### Google+ Api
* Login to google account and update the api with new webserver info.
* Update client_secret.json with updated info.

## Third-Party Resources
* https://discussions.udacity.com/c/nd004-full-stack-broadcast
* http://flask.pocoo.org
* https://www.sqlalchemy.org
* https://www.postgresql.org
* https://modwsgi.readthedocs.io
* https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
