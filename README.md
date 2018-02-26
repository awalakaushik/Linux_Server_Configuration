# Linux Server Configuration
A baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.

# Get your Server
## Start a new Ubuntu Server instance on Lightsail
	Ubuntu Server instance details:
	Public IP: 18.217.71.17
	Username: grader

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
ssh -i /path/<private key filename> grader@18.219.227.189
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
