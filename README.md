# Linux server configuration project

Implement a baseline installation of a Linux distribution on a virtual machine and prepare it to host my web application [Item Catalog](https://github.com/FatmaMagdy/Item-Catalog-Application.git), to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

## Getting Started

These instructions will walk you through the project and running it for testing purposes. See Running section for notes on how to run the project on a live system.

## Requirements

IP address: 18.236.75.38

SSH port: 2200

[URL](http://18.236.75.38.xip.io)

## Running

### 1 - Create a new user account named grader, give grader the permission to sudo

1. ssh into the VM : `$ ssh root@18.236.75.38`.
2. Add a new user called *grader*: `$ sudo adduser grader`.
3. Create a new file in suoders directory: `$ sudo vim /etc/sudoers.d/grader`.
   add "grader ALL=(ALL:ALL) ALL", save and exit.
4. Edit the hosts to resolve host errors:
	1. `$ sudo vim /etc/hosts`.
	2. Add the host: `127.0.1.1 ip-18.236.75.38`.

### 2 - Update all currently installed packages

1. `$ sudo apt-get update`.
2. `$ sudo apt-get upgrade`.

### 3 - Configure the local timezone to UTC

1. set the time to UTC: `$ sudo dpkg-reconfigure tzdata`.
2. Install: `$ sudo apt-get install ntp`.

### 4 - Create an SSH key pair for grader using the ssh-keygen tool.

1. Generate an key: `$ ssh-keygen -f ~/.ssh/udacity_key.rsa`.
2. Log into the remote VM: `$ touch /home/grader/.ssh/authorized_keys`.
3. change permissions:
	1. `$ sudo chmod 700 /home/grader/.ssh`.
	2. `$ sudo chmod 644 /home/grader/.ssh/authorized_keys`.
	3. `$ sudo chown -R grader:grader /home/grader/.ssh`.
  4. `$ ssh -i ~/.ssh/udacity_key.rsa grader@18.236.75.38`.
  5. `$ sudo vim /etc/ssh/sshd_config`.
  6. `$ sudo service ssh restart`.

### 5 - Change the SSH port from 22 to 2200. and configure the Lightsail firewall to allow it
1. `$ sudo vim /etc/ssh/sshd_config`.
2. `$ sudo service ssh restart`.
3. `$ ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@52.34.208.247`.
4. `$ sudo ufw allow 2200/tcp`.
5. `$ sudo ufw allow 80/tcp`.
6. `$ sudo ufw allow 123/udp`.
7. `$ sudo ufw enable`.

### 6 - Configure firewall to monitor for repeated unsuccessful login attempts and ban attackers

1. `$ sudo apt-get update`.
2. `$ sudo apt-get install fail2ban`.
3. `$ sudo apt-get install sendmail`.
4. `$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local` .
5. `$ sudo vim /etc/fail2ban/jail.local`. add my Email address.

### 7 - Install and configure Apache to serve a Python mod_wsgi application

1. `$ sudo apt-get install apache2`.
2. `$ sudo apt-get install libapache2-mod-wsgi python-dev`.
3. `$ sudo a2enmod wsgi`.
3. `$ sudo service apache2 start`.

### 8 - Install Git and clone the Item Catalog Application Projcet

1. `$ sudo apt-get install git`.
2. `$ git config --global user.name <username>`.
3. `$ git config --global user.email <email>`.
4. `$ cd /var/www`. Then: `$ sudo mkdir catalog`.
5. `$ sudo chown -R grader:grader catalog`.
6. Create a catalog.wsgi file, then add this.
 ```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
```
7. Rename app.py to init.py mv app.py __init__.py
8. `sudo pip install virtualenv` .
9. `sudo virtualenv venv` .
10. `sudo chmod -R 777 venv` .
11. Install Flask and sqlalchemy `sudo apt-get install python-pip` .
    pip install Flask
    sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils

12. vim __init__.py
13. Change client_secrets.json path to /var/www/catalog/catalog/client_secrets.json
Configure and enable a new virtual host
Run this: sudo vim /etc/apache2/sites-available/catalog.conf

### 9 - Configure and enable a new virtual host

1. Create a virtual host conifg file: `$ sudo nano /etc/apache2/sites-available/catalog.conf`.
2. Paste in the following lines of code:
```
<VirtualHost *:80>
    ServerName 52.34.208.247
    ServerAlias ec2-52-34-208-247.us-west-2.compute.amazonaws.com
    ServerAdmin admin@52.34.208.247
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
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
3. `$ sudo a2ensite catalog`.


### 10 - Install and configure PostgreSQL

1. `$ sudo apt-get install libpq-dev python-dev`.
2. `$ sudo apt-get install postgresql postgresql-contrib`.
3. `$ sudo su - postgres`, then connect to the database system with `$ psql`.
4. `# \c catalog`.
5. `$ exit`.
6. Change the database connection to:
```
engine = create_engine('postgresql://catalog:mypassword@localhost/catalog')
```
7. Setup the database with: `$ python /var/www/catalog/catalog/setup_database.py`.

### 11 - Install system monitor tools

1. `$ sudo apt-get update`.
2. `$ sudo apt-get install glances`.
3. To start this system monitor program just type this from the command line: `$ glances`.
4. Type `$ glances -h` to know more about this program's options.

### 12 - Restart and launch the app
1. `$ sudo service apache2 restart`.



#### Notes
I have used some other resources as a guide.
- [Fourm](https://discussions.udacity.com/t/aws-dns-connection-error/531024)
- [Repo](https://github.com/rrjoson/udacity-linux-server-configuration)
- [Repo](https://github.com/stueken/FSND-P5_Linux-Server-Configuration)
