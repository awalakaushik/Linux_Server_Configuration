# Linux Server Configuration
A baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.

# Get your Server
## Start a new Ubuntu Server instance on Lightsail
	Ubuntu Server instance details:
	Public IP: 18.221.92.92
	SSH Port: 2200
	Username: grader
	Access the hosted application here: http://18.221.92.92

## SSH into your Server
	Login to Amazon Lightsail and login to your newly created instance

# Secure your Server
## Update all currently installed packages
```
sudo apt-get update
```
## Update all updated packages
```
sudo apt-get upgrade
```

## Change the SSH port from 22 to 2200. Make sure to configure Lightsail firewall to allow it
```
sudo nano /etc/ssh/sshd_config
```

## Configure the uncomplicated firewall to only allow incoming connections for SSH (port 2200), HTTP (port 80), NTP (port 123)
	sudo ufw default deny incoming
	sudo ufw default allow outgoing
	sudo ufw allow 2200/tcp
	sudo ufw allow 80/tcp
	sudo ufw allow 123/udp
	sudo ufw enable
	sudo ufw status

# Give Grader Access
	In order to be able to login and review the project, the grader needs an account
## Create a new account named "grader"
	sudo adduser grader
## Give grader the permission to sudo
* Edit the sudoers.d file to add in sudo permission for the grader
```
sudo nano /etc/sudoers.d/grader
```
* Add the following line to enable sudo access to the user
```
grader ALL=(ALL) NOPASSWD:ALL
```
# Create an SSH keypair for the grader using ssh-keygen
## Create a key-pair using the "ssh-keygen" tool
```
$ ssh-keygen
```
* Save the private key on the local machine
* Paste the public key on the server machine

## Saving public key on Server
```
$ su -u grader
$ mkdir .ssh
$ touch .ssh/authorized_keys
$ nano .ssh/authorized_keys
```
Paste the public key inside .ssh/authorized_keys on the server

## Change the file permissions as follows
```
chmod 700 .ssh
chmod 644 .ssh/authorized_keys
```
## Reload SSH
```
sudo service ssh restart
```
## Login to Server
```
ssh -i /path/<private key filename> grader@18.217.71.17
```
# Prepare to deploy your project
## Configure the local time zone to UTC
```
sudo dpkg-reconfigure tzdata
```
* Select none of the above from the options given
* Select UTC and click OK

## Install and configure Apache to serve a Python mod_wsgi application
* If you used Python 3 to build your project, you will need to install the Python 3 mod_wsgi package on your server
```
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi-py3
sudo apt-get install python3-setuptools libapache2-mod-wsgi-py3
sudo service apache2 restart
```
## Install and configure PostgreSQL
* Do not allow remote connections
* Create a new database, user named catalog that has limited permissions to your catalog application database
```
sudo apt-get install postgresql
```
* Check  if no remote connections are allowed in the client authentication configuration file
```
sudo nano /etc/postgresql/9.5/main/pg_hba.conf
```
* Login as postgres user
```
sudo su - postgres
```
* Turn on postgreSQL shell prompt
```
psql
```
* Create a new database and a new user named catalog
```
postgres=# CREATE DATABASE catalog;
postgres=# CREATE USER catalog;
```
* Set password for the user
```
postgres=# ALTER ROLE catalog WITH PASSWORD 'grader@catalog';
```
* Give user catalog permissions to catalog application
```
postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
```
* Quit Shell
```
postgres=# \q
```
* Exit from postgres user prompt
```
exit
```
## Install Git
```
sudo apt-get install git
```
# Prepare to delpoy your project
## Install and Configure Apache mod-wsgi application.
1. Install Apache
```sudo apt-get install apache2```
2. Install mod-wsgi
```sudo apt-get install libapache2```
* If using python3, use the following command.
```sudo apt-get install libapache2-mod-wsgi-py3```
3. Restart Apache service.
```sudo service apache2 restart```
## Install and configure PostgreSQL
1. Install PostgreSQL 
```sudo apt-get install postgresql```
2. make sure that remote connections are not allowed. 
```sudo vim /etc/postgresql/9.3/main/pg_hba.conf```
3. Login as postgres user 
```sudo su - postgres```
4. Open PostgreSQL shell. 
```psql```
5. Create a new database and user named catalog in postgreSQL shell.
```
CREATE DATABASE catalog;
CREATE USER catalog;
```
5. Create a password for catalog user
```
ALTER ROLE catalog WITH PASSWORD 'graderCatalog';
```
6. Grant all privileges to catalog user on catalog database
```
GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
```
7. Quit postgreSQL.
```\q```
8. Exit from user "postgres".
```
exit
```
## Clone and setup Item Catalog project
1. Move to /var/www directory
```
cd /var/www/
```
2. Create ItemCatalog directory
```
mkdir ItemCatalog
```
3. Clone the github repository here
```
git clone <repository-url>
Here, 
git clone https://github.com/awalakaushik/ItemCatalog.git
```
8. Rename `application.py` to `__init__.py` using `sudo mv website.py __init__.py`
9. Edit `catalog_database_setup.py`, `__init__.py` and `populate_database.py` and change `engine = create_engine('sqlite:///itemcatalogwithusers.db')` to `engine = create_engine('postgresql://catalog:graderCatalog@localhost/catalog')`
10. Install pip 
```sudo apt-get install python-pip```
11. Install dependencies
```
pip install httplib2
pip install requests
pip install flask
pip install sqlalchemy
pip install psycopg2
pip install oauth2client
pip install Flask-SQLAlchemy
```
12. Install psycopg2 
```
sudo apt-get install postgresql python-psycopg2
```
13. Create database schema 
```
sudo python catalog_database_setup.py
```

## Configure and Enable a New Virtual Host
1. Create ItemCatalog.conf 
```
sudo nano /etc/apache2/sites-available/ItemCatalog.conf
```
2. Add the following lines of code to the file to configure the virtual host. 	
```
<VirtualHost *:80>
	ServerName 18.217.71.17
	ServerAdmin admin@18.217.71.17
	WSGIDaemonProcess ItemCatalog python-path=/var/www/ItemCatalog:/var/www/ItemCatalog/lib/python2.7/site-packages
	WSGIProcessGroup ItemCatalog
	WSGIScriptAlias / /var/www/ItemCatalog/itemcatalog.wsgi
	<Directory /var/www/ItemCatalog/ItemCatalog/>
		Order allow,deny
		Allow from all
	</Directory>
	Alias /static /var/www/ItemCatalog/ItemCatalog/static
	<Directory /var/www/ItemCatalog/ItemCatalog/static/>
		Order allow,deny
		Allow from all
	</Directory>
	ErrorLog ${APACHE_LOG_DIR}/error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
3. Enable the virtual host with the following command: 
```
sudo a2ensite ItemCatalog
```

## Create the .wsgi File
1. Create the .wsgi File under /var/www/ItemCatalog: 	
```
cd /var/www/ItemCatalog
sudo nano itemcatalog.wsgi 
```
2. Add the below script to the flaskapp.wsgi file:	
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/ItemCatalog/")

from ItemCatalog import app as application
application.secret_key = 'Add your secret key'
```

## Restart Apache
1. Restart Apache `sudo service apache2 restart `

# References
* https://www.digitalocean.com/community/tutorials/how-to-run-django-with-mod_wsgi-and-apache-with-a-virtualenv-python-environment-on-a-debian-vps
* https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
