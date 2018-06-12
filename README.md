# Introduction
This is a guideline of how to configure an linux server and host a Flask web application on Amazon Lightsail.
- IP address of this example is: [18.216.108.90.xip.io](18.216.108.90.xip.io) *Adjust it to your server IP if you want to your own server.*
- SSH port: 2200

# Configuration
After you log into the remote server using SSH, you can configure the server as follows.

##  Update installed packages
 1. In the terminal connecting to the remote server, run `sudo apt-get update`. This will update the latest package list.
 2. Then run `sudo apt-get upgrade` to upgrade installed packages.


##  Configure Connection options
It is important to keep the **least privilege** principle when runing a server. In other words, the connection ports should be limited.

#### Check Firewall status
1.  In the terminal connecting to the remote server, run `sudo ufw status` to check the firewall status. It should be in default **inactive**.
2.  If it is active, type `sudo ufw disable` to shutdown the firewall just for now.
3.  run `sudo ufw default deny incoming` to reject unauthrorized connection request
4.  run `sudo ufw default allow outgoing` to allow any outgoing connection from server

#### Change SSH port
Here we are going to the default ssh port **22** to **2200**
1.  In the **Amazon Lightsail** website, manage the setting of your running server. In the **network** settings, create a custom application with **TCP** and port range **2200**. This will open a port of 2200.
2.  In the terminal connecting to the remote server, run `sudo ufw allow ssh` to enable ssh
3.  `sudo ufw allow 2200/tcp` to allow connection from port 2200.
4.  `sudo nano /etc/ssh/sshd_config`. SSH at default listen to port 22, **change 22 to 2200**
5.  `sudo service ssh restart` to let the changes make effect.
6.  `sudo ufw delete allow 22/tcp` to remove old ssh ports
7.   In the **Amazon Lightsail** website, manage the setting of your running server. In the **network** settings, delete the ssh application with port 22.

#### Allow HTTP & NTP ports
Here we are using the default ports for http and ntp, so we only need to allow them in the firewall.
1.  In the terminal connecting to the remote server, run `sudo ufw allow www` to enable http default port **80**.
2.  run `sudo ufw allow ntp` to enable ntp default port **123**

#### Enable the Firewall
We have made all needed changes, run `sudo ufw enable` to start the firewall. From now on, you will use port **2200** to connect to the remote server. Port 22 is disabled from now on.


## Configure User options
Here we will create a new user `grade` and give him basic privileges. We will modify log-in settings so that all users **must use key** instead of password to connect to the server. Finally, we will disable user logging in as root.  

#### Create user - grader
1.  In the terminal connecting to the remote server, run `sudo adduser grader`. This creates a new user called **grader** and initializes his folder under **/home**.
2.  `sudo nano /etc/sudoers.d/grader` and add the text `grader ALL=(ALL:ALL) ALL`. This will give user grader the ability to use all commands.
3.  Generate a pair of rsa keys **on your local machine**. You can run `ssh-keygen` which will generate a **id_rsa** and **id_rsa.pub** pair.
4.  Save **id_rsa** which is your private key to log in with. Copy the content in **id_rsa.pub**
5.  In the terminal connecting to the remote server, `cd /home/grader/` to go into the grader's folder.
6.  `mkdir .ssh` to create a hidden folder which will store your key.
7.  `touch .ssh/authorized_keys` to create a file **authorized_keys** inside the folder
8.  `nano .ssh/authorized_keys` will open the file. Paste the content in **id_rsa.pub** here.
9.  `chmod 700 .ssh; chmod 644 .ssh/authorized_keys` to restrict the access of your ssh keys.
10. Now you can log in as **grader** from your local machine using `ssh -i [the path of your id_rsa] grader@18.216.108.90 -p 2200`.

#### Restrict log in
Now we are going to enforce that every user should log in using their ssh keys. Also, we do not want anyone to act as **root** the time when they log in, so we will disable that for log in.
1. In the terminal connecting to the remote server, run `sudo nano /etc/ssh/sshd_config`. This will open the ssh config file....again :)
2. Find the line **PermitRootLogin somemode**. The **mode** can be **yes/no/prohibited-password**. Make sure it is **no**.
3. In the same file, find the line **PasswordAuthentication somemode**. Make sure it is **no** also, which should be default.
4.  Save the file, run `sudo service ssh restart` to let changes make affects.


## Configure Time options
Here we will change the local time to UTC time standard.
1.  In the terminal connecting to the remote server, run `sudo dpkg-reconfigure tzdata`
2.  Choose your timezone


## Install Apache server
Here we will install Apache packages in python to later host our application.
1.  In the terminal connecting to the remote server, run `sudo apt-get install apache2`. Now there should be a basic html when visiting http://18.216.108.90
2.  `sudo apt-get install libapache2-mod-wsgi python-dev` to install mod_wsgi, an interface between web servers and web apps for python.
3.  `sudo a2enmod wsgi` to enable wsgi
4.  `sudo service apache2 restart` to restart apache (not sure if necessary)


## Install PostGRESQL
Next we will install a SQL database--PostGRESQL. The package we use are python.
1.  In the terminal connecting to the remote server, run `sudo apt-get install postgresql`
2.  `sudo su postgres` to switch to the super user
3.  `psql` to enable an interactive shell
4.  `CREATE USER catalog;` creates a user **catalog**
5.  `ALTER ROLE catalog WITH PASSWORD 'catalog';` to set its password as **catalog**
6.  `CREATE DATABASE catalog WITH OWNER catalog;` to create a database called **catalog** owned by user **catalog**.
7.  `\l` to make sure the database is created correctly.
8.  '\q' and then `exit` to return to user **grader**


## Host an Flask App
Finally, we are going to host a flask app. The example app here is a catalog website. We will also install several packages including: **pip, sqlalchemy, psycopg2**
1.  `sudo apt-get install git` to install Git
2.  `cd /var/www` to move the directory where we will place the app
3.  `sudo mkdir FlaskApp` to create a directory called **FlaskApp**
4.  `cd FlaskApp` to move into the repo
5.  `sudo git clone https://github.com/ziz19/item-catelog.git FlaskApp` to clone to app and call it **FlaskApp**
6.  `cd FlaskApp` to move into the git repo
7.  `sudo mv project.py __init__.py` to rename the main script
8.  `sudo nano database_setup.py` will open the python script. Find the line where `engine = create_engine(XXX)`. Change it to `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`
9.  `sudo nano __init__.py` will open another script. Change the **engine** code as above. Also, change the location of `"client_secrets.json"` to `"/var/www/FlaskApp/FlaskAPp/client_secrets.json"` (There are **two places**)
10.  `sudo apt-get install python-pip` to install pip
11.  `sudo pip install SQLAlchemy` to install sqlalchemy
12. `sudo pip install psycopg2 binary` to install psycopg2 for using postgresql in python
13. `sudo pip install flask` to install flask
14.
```
sudo pip install oauth2client  
sudo pip install requests   
sudo pip install httplib2
```  
to install specific packages for this app.
15. `sudo nano /etc/apache2/sites-available/FlaskApp.conf` to create a new config file for the app
16. Copy the following content and save it
```
<VirtualHost *:80>
        ServerName 18.216.108.90
        ServerAdmin zhuangzinanmaoxiong@gmail.com
        ServerAlias 18.216.108.90.xip.io
        WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
        <Directory /var/www/FlaskApp/FlaskApp/>
                Order allow,deny
                Allow from all
        </Directory>
        Alias /static /var/www/FlaskApp/FlaskApp/static
        <Directory /var/www/FlaskApp/FlaskApp/static/>
                Order allow,deny
                Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
17. `sudo a2ensite FlaskApp` to enable the app
18. `sudo nano /var/www/FlaskApp/flaskapp.wsgi` to create a wsgi file
19. Copy the following content inside

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")

from FlaskApp import app as application
application.secret_key = 'super secret key'
```


## Restart Apache
Now we have completed all configuration.
`sudo service apache2 restart` to restart your server.
