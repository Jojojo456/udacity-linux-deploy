
# Udacity Full Stack Dev - Linux Deployment

### Description

All this runs on an AWS EC2 instance running an Ubuntu Linux distribution. The Linux distro has all the newest updates installed and is secured against a number of attack vectors. It also features a web server with a Postgres DB running the `Catalog App` from the Udacity project 4.

- IP address: 18.185.139.220

-  Accessible SSH port: 2200

- Application URL: http://ec2-18-185-139-220.eu-central-1.compute.amazonaws.com/

``
  A README file is included in the GitHub repo containing the following information: 
  IP address, URL, summary of software installed, summary of configurations made, and a list of third-party resources used to complete this project.
``


### Steps done

1. Create new user named grader and give it the permission to sudo
  - SSH to the server with the provided key `grader_key`
  - Run `$ sudo adduser grader` to create a new user named grader
  - Create a new file in the sudoers directory with `sudo nano /etc/sudoers.d/grader`
  - Add the following text `grader ALL=(ALL:ALL) ALL`
   
2. Update all currently installed packages
  -  `sudo apt-get update`
  -  `sudo apt-get upgrade`

3. Change SSH port from 22 to 2200
  - Run `sudo nano /etc/ssh/sshd_config`
  - Change the port from 22 to 2200
  
4. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
  - `sudo ufw allow 2200/tcp`
  - `sudo ufw allow www`
  - `sudo ufw allow ntp`
  - `sudo ufw enable`
 
5. Configured key-based authentication for grader user
  - created keys with `ssh-keygen`
  - added the public key to the server

7. Disable ssh login for root user
  - Run `sudo nano /etc/ssh/sshd_config`
  - Change `PermitRootLogin without-password` line to `PermitRootLogin no`
  - Restarted ssh with `sudo service ssh restart`
 
8. Installd Apache
  - `sudo apt-get install apache2`

9. Installed mod_wsgi
  - Run `sudo apt-get install libapache2-mod-wsgi python-dev`
  - Enable mod_wsgi with `sudo a2enmod wsgi`
  - Start the web server with `sudo service apache2 start`

10. Clone the Catalog app from Github
  - `cd /var/www`
  - `sudo mkdir catalog`
  - Change owner of the newly created catalog folder `sudo chown -R grader:grader catalog`
  - `cd /catalog`
  - copied the website files (Catalog App) over to the server
  - Create a catalog.wsgi file, then add this inside:
  ```
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/catalog/")
  
  from catalog import app as application
  application.secret_key = 'MYKEYHERE'
  ```
  - Renamed main python script to`__init__.py`
  
11. Installed virtual environment
  - Installed the virtual environment `sudo pip install virtualenv`
  - Created a new virtual environment with `sudo virtualenv venv`
  - Activated the virutal environment `source venv/bin/activate`
  - Changed permissions `sudo chmod -R 777 venv`

12. Installed Flask and other dependencies with active venv
  - Install pip with `sudo apt-get install python-pip`
  - Installed required libs by `pip install -r requirements.txt`

13. Updated path of client_secrets.json file
  - `nano __init__.py`
  - Changed client_secrets.json path to `/var/www/catalog/catalog/client_secrets.json`
  
14. Configured and enabled a new virtual host
  - Run this: `sudo nano /etc/apache2/sites-available/catalog.conf`
  - Pasted this code: 
  ```
  <VirtualHost *:80>
      ServerName 18.185.139.220
      ServerAlias ec2-18-185-139-220.eu-central-1.compute.amazonaws.com
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
  - Enabled the virtual host `sudo a2ensite catalog`

15. Installed and configured PostgreSQL
  - `sudo apt-get install libpq-dev python-dev`
  - `sudo apt-get install postgresql postgresql-contrib`
  - `sudo su - postgres`
  - `psql`
  - `CREATE USER catalog WITH PASSWORD 'password';`
  - `ALTER USER catalog CREATEDB;`
  - `CREATE DATABASE catalog WITH OWNER catalog;`
  - `\c catalog`
  - `REVOKE ALL ON SCHEMA public FROM public;`
  - `GRANT ALL ON SCHEMA public TO catalog;`
  - `\q`
  - `exit`
  - Change create engine line in your `__init__.py` and `database_setup.py` to: 
  `engine = create_engine('postgresql://catalog:MYPASSWORD@localhost/catalog')`
  - `python /var/www/catalog/catalog/database_setup.py`
  - populated sample data into postgres with `python /var/www/catalog/catalog/lotsofitems.py`
  
16. Restart Apache 
  - `sudo service apache2 restart`
  
17. Visit site at http://ec2-18-185-139-220.eu-central-1.compute.amazonaws.com/


# References

https://stackoverflow.com/questions/35017160/how-to-use-virtualenv-with-python
https://docs.aws.amazon.com/efs/latest/ug/wt2-apache-web-server.html
https://www.shellhacks.com/modwsgi-hello-world-example/
