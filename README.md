# Linux Server Configuration



## About the project
This project shows how to access, secure, and perform the initial configuration of a bare-bones Linux server. It highlights how to install and configure a web and database server as well as hosting a web application.

## IP & Hostname
- Host Name: http://ec2-18-222-114-195.us-east-2.compute.amazonaws.com
- IP Address: 18.222.114.195
- Port : 2200

##  Setting Up Amazon Lightsail

 1. Open [Amazon Lightsail](https://aws.amazon.com/lightsail/)
 2. Click get started for free
 3. Create Account
 4. Create your first instance ( it might take 24 hours or more in order to activate your account to create your first instance )
5. Select OS Only and Ubuntu
6. Scroll to the bottom of the screen and rename your instance
7. Download the private SSH key by navigating to the **Account Page** in the connect tab
7. As required only allow connections for SSH (port 2200), HTTP (port 80), and NTP (port 123) from the networking tab

## Linux Configuration

### For Security -- part 1 --
 1. Place your downloaded private key into`.ssh` at the root of your user directory. For example `Macintosh HD/Users/[Your username]/.ssh/`.
 2. In the terminal `$ chmod 600 ~/.ssh/[PRIVATE KEY].pem` .
 3. Log into the server `$ ssh -i ~/.ssh/[PRIVATE KEY].pem ubuntu@[IP address]`
 4. Update all application with the following commands
 ```
 $ sudo apt-get update
 $ sudo apt-get upgrade
 $ sudo apt-get sudo apt-get autoremove

 ```

### User Management
 1. Install Finger package using `sudo apt-get install finger `
 2.  Create User grader as request `sudo adduser grader`
 3.  Configure sudoer file `sudo /usr/sbin/visudo`
 4. add **grader ALL=(ALL:ALL)** below root ALL=(ALL:ALL), then exist and save the changes.
 5. Assure that grader user has granted a superuser permission , search for it in the root users by typing `cut -d: -f1 /etc/passwd`

### For Security -- part 2 --
 1. Generate key pair in local machine not on server by opening new terminal and typing ` new terminal run the command: `$ ssh-keygen -f ~/.ssh/[SSH KEY NAME]`
 2.  Copy the value of the public key from here`$ cat ~/.ssh/[SSH KEY NAME].pub`
 3. Turn back to the server terminal then `cd /home/grader`.
 4. `$ mkdir .ssh` to create ssh directly
 5. `$ touch .ssh/authorized_keys` to create authorized_keys file to store the public key
 6. `$ nano .ssh/authorized_keys`, paste the copied key from step2 , then exist and save the file
 7. Change the permission for .ssh folder
 ```
 $ chmod 700 .ssh
 $ chmod 644 .ssh/authorized_keys
 ```
 8. `$ sudo chown -R grader:grader /home/grader/.ssh` to change the owner of ssh folder
 9. restart the server `$ sudo service ssh restart`
 10. `logout` from server
 11. Login `$ ssh -i ~/.ssh/[SSH KEY NAME] grader@[IP address]`
 12. To enforce key authentication from the ssh configuration file by editing `$ sudo nano /etc/ssh/sshd_config`. Find the line that says **PasswordAuthentication** and change it to no. Also find the line that says **Port 22** and change it to **Port 2200**. Lastly change **PermitRootLogin** to no.
 13. Restart ssh again: `$ sudo service ssh restart`
 14. `logout` and try step "**10.**" and add **-p 2200** at the end to connect.
 15. configure the firewall as requested:
 ```
 $ sudo ufw allow 2200/tcp
 $ sudo ufw allow 80/tcp
 $ sudo ufw allow 123/udp
 $ sudo ufw enable
 ```
 16. To check firewall status type `$ sudo ufw status`


# Setup process for delopying Flask application
 Hosting this application will require the Python virtual environment, Apache with mod_wsgi, PostgreSQL, and Git.
 1. Start by installing the required software
```
$ sudo apt-get install apache2
$ sudo apt-get install libapache2-mod-wsgi python-dev
$ sudo apt-get install git
$ sudo apt-get install python-pip
```
 2. `$ sudo a2enmod wsgi` to enable mod_wsgi
 3. `$ sudo service apache2 restart`. to restart Apache

### Clone Item Catalog Project
1. Clone the item catalog and change the owner of the catalog folder
```
$ cd /var/www
$ sudo mkdir catalog
$ sudo chown -R grader:grader catalog
$ cd catalog
$ git clone [Repository URL] catalog
```

### Install and Activate the Virtual Environment and Required Packages to Run Flask Application
```
$ sudo apt-get install apache2
$ sudo apt-get install libapache2-mod-wsgi python-dev
$ sudo apt-get install git
$ sudo apt-get install python-pip
$ sudo pip install virtualenv
$ export LC_ALL=C
$ sudo virtualenv venv
$ source venv/bin/activate
$ sudo chmod -R 777 venv
$ sudo apt-get install python-setuptools
$ sudo apt-get install python-psycopg2
$ sudo pip install Flask
$ sudo pip install oauth2client
$ sudo pip install requests
$ sudo pip install httplib2
$ sudo pip install sqlalchemy
$ sudo service apache2 restart
```
### Python Code Modifications
1. `nano [Application Name].py` and update the path to client_secrets.js to `/var/www/catalog/catalog/client_secrets.json`
### Virtual Host Configuration
1. `$ sudo nano /etc/apache2/sites-available/catalog.conf` to create configuration file
```
<VirtualHost *:80>
    ServerName [IP ADDRESS]
    DocumentRoot /var/www/catalog
    ServerAdmin admin@[IP ADDRESS]
    WSGIDaemonProcess catalog
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog>
        WSGIProcessGroup catalog
        WSGIApplicationGroup %{GLOBAL}
        Order allow,deny
        Allow from all
    </Directory>
</VirtualHost>
```
2. `sudo a2ensite catalog.conf` to enable the new host
3. `sudo a2dissite 000-default.conf ` to disable the default host
4. `service apache2 reload` to reload service.

### Google Sign-in
1. Update redirect url
2. Update clients_json with the new URIs

### Database Configuration
1.Install required package
```
$ sudo apt-get install postgresql postgresql-contrib
$ sudo su - postgres ( inside app directory)
$ sudo su - postgres -i
```
2. Then add the following
```
postgres=# CREATE USER catalog WITH PASSWORD [password];
postgres=# ALTER USER catalog CREATEDB;
postgres=# CREATE DATABASE catalog with OWNER catalog;
postgres=# \c catalog
catalog=# REVOKE ALL ON SCHEMA public FROM public;
catalog=# GRANT ALL ON SCHEMA public TO catalog;
catalog=# \q
$ exit
```
3. Update database_setup.py , data_seeder.py and catalog.py with the new database
`postgresql://username:password@localhost/catalog`
### Run the Application
1. `$ nano database_setup.py
2.  `nano data_seeder.py`

### References:
 - [How to configure Sudoers in Ubuntu](https://www.youtube.com/watch?v=8PIVZh6Mao0&t=0s&list=LLJJ7ZTgDL6RVMGrwnL1I3eg&index=2)
 - https://github.com/mulligan121/Udacity-Linux-Configuration
