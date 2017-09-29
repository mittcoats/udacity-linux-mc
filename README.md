# Setup Instructions for Udacity FSND Linux Course Project
This file describes how to configure a Flask App on a Ubuntu machine, hosted on AWS LightSail using an Apache server. Each section below gives instructions for the above mentioned sections.

## Overview
### Server info
__IP:__ [18.221.185.248]

__URL:__ [udacity-linux-link]

### Software Installed
* LightSail
* SSH
* UFW
* Git
* Flask
* PostgreSQL
* Apache
* WSGI

## Server Configuration
### Users
__Add grader user__
- `sudo adduser grader`
- Created password: "grader"


__Add grader to sudoers__
Create sudoers file for grader
- `touch /etc/sudoers.d/grader`

Open file and add following grant grader sudo powers
- `nano /etc/sudoers.d/grader`
- `grader ALL=(ALL) NOPASSWD:ALL`

### Security
__Firewall__

__Key-based SSH__
Create a new set us keys on local machine
- `ssh-keygen`

Return to server, logged in as grader
- `su - grader`

Add directory for keys
- `mkdir .ssh`

Update change permissions to 700 to allow writing
- `chmod 700 .ssh`

Paste public key from local machine `.ssh` directory (example_key.pub) in "authorized_keys" file.
- `nano .ssh/authorized_keys`

Update permission to prevent changes to keys
- `chmod 644 .ssh/authorized_keys`

Restart ssh service
`sudo service ssh restart`

__Update timezone UTC__
Configure the local timezone to UTC
- `sudo dpkg-reconfigure tzdata`
- Then set location to "none of the above"; Then select UTC and hit ok

__Update system packages__
- `sudo apt-get update`
- `sudo apt-get upgrade`


__Host SSH on non-default port__

### App
__Web server at port 80__
Install Apache2
- `sudo apt-get install apache2`

Install python
- `sudo apt-get install python`
- `sudo apt-get install python-setuptools`

Install mod_wsgi package
- `sudo apt-get install apache2 libapache2-mod-wsgi`

Start apache server
- `sudo service apache2 restart`

__Database server configuration__
Install PostgreSQL
- `sudo apt-get install postgresql`

Create database for store
- `sudo -u postgres psql`

Create database with admin user
- `create user admin with password 'admin'`
- `create database store with owner admin`

__Serve Flask App, "Item Catalogue" as WSGI App__
Install git
- `sudo apt-get install git`

Create new directory for store app and clone git repo
- `cd /var/www`
- `sudo git clone https://github.com/mittcoats/mittcoatshouse.git store`

Create config file using Apache and mod_wsgi
- `sudo nano /etc/apache2/sites-available/store.conf`
```
<VirtualHost *:80>
    ServerName 18.221.185.248
    ServerAdmin admin@18.221.185.248
    ServerAlias udacity-linux.mittcoats.com
    ServerAlias ec2-PUBLIC_IP.us-west-2.compute.amazonaws.com
    WSGIScriptAlias / /var/www/store/wsgi.py
    <Directory /var/www/store/store>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
</VirtualHost>
```

Configure Flask app with WSGI by adding following file and code
- `/var/www/store$ sudo nano store.wsgi`
```
import sys
import logging
logging.basicConfig(stream=stderr)
sys.path.insert(0,"/var/www/store/")

from views import app as application
application.secret_key == 'CUSTOM_SECRET_KEY'

application.config['SQLALCHEMY_DATABASE_URI'] = (
    'postgresql://'
    'admin:admin@localhost/store')
```

Start and restart server
- `sudo a2ensite store`
- `sudo service apache2 reload`

Check for server errors...
- `sudo cat /var/log/apache2/error.log`

## Gotchas
- Make sure to move `app.secret` and `app.config` out of `if __name__ == '__main__'` because this if clause is not evaluated in WSGI app
- Make sure to add complete path names to secrets file for Google auth, otherwise will through error
- Make sure to add custom domain name to OAuth 2.0 client IDs, otherwise Google with throw cross origin error. And Google won't let you use only IP address, so you're stuck needing a custom domain. I used a subdomain on my personal web address.
- Make sure that the `views` in your WSGI file point to wherever you have your app init code. In my case, it was in `views.py` file, however most flask apps should use an `__init__.py` file

## Sources
- https://www.phusionpassenger.com/library/walkthroughs/deploy/python/ownserver/apache/enterprise/xenial/deploy_app.html
- https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
- https://github.com/aviaryan/flask-postgres-apache-server
- https://github.com/kongling893/Linux-Server-Configuration-UDACITY
- https://github.com/sharkwhistle/Udacity-FSND-Linux-Server-Configuration

[udacity-linux-link]: http://www.udacity-linux.mittcoats.com
