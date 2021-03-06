# Linux Server Configuration Project
This project has been developed for Udacity to complete a Full Stack Web Developer Nanodegree.

## About
In this project, we will take a baseline installation of a Linux server and prepare it to host our web applications. we will secure our server from a number of attack vectors, install and configure a database server, and deploy one of our existing web applications onto it.
I deployed my web app of Item Catalog Project.

## Note For Reviewer

**Public IP**
18.185.95.250

**SSH port**
2200

**The URL to my hosted web application**
http://18.185.95.250.xip.io/

**Login to the server by grader user**
`ssh grader@18.185.95.250 -p 2200 -i ~/.ssh/grader-key`

**Third party resources**

- Google OAuth
- Amazon Lightsail AWS
- Flask
- Apache

**Summary of install apps**

- Apache2
- mod_wsgi module
- Sqlalchemy
- postgresql
- Python
- pip
- Python Virtual Environments
- Flask
- psycopg2-binary
- git
- Oauth2client

# How TO Deployed this site
## Create The Server
create an account on any web hosting service, I used Amazon Lightsail AWS, as steps below:

- create instance
- Pick your instance image
- Select a platform ( Linux/Unix)
- Select a blueprint (OS Only)
- chose(Ubuntu - 18.04 LTS)
- Choose your instance plan (Price per month - $3.50 USD)
- Identify your instance (your-server-name)

After the server activate you can download the private key.
"You can download your default private key from the **Account page**"

## Updata & Upgrade Packages
select **Connect using SSH** then do this steps to update and upgrade all currently installed packages
```
 sudo apt-get update
 sudo apt-get upgrade
 sudo apt-get dist-upgrade
 sudo apt install unattended-upgrades
```
## Change The Port For SSH To 2200
edit the file **sshd_config** and locate the line `Port 22`
`sudo nano /etc/ssh/sshd_config`
put this line under Port 22 line
`Port 2200`

## Configuring the Uncomplicated Fire Wall (UFW)
put this command to see the status of Firewall
`sudo ufw status`
it must be inactive , now put these commands
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/udp
sudo ufw enable
```
now see all the changes to the firewall
`sudo ufw status`
reboot server ( from aws website - reboot button)
now you can login again with this commaned from local shell
`ssh -i .ssh/LightsailDefaultKey-eu-central-1.pem ubuntu@18.185.95.250 -p 2200`

## Configure the local timezone to UTC
select None of the above then select UTC
`sudo dpkg-reconfigure tzdata`

## create a grader user
`sudo adduser grader`
`sudo ls /etc/sudoers.d`

To give the user the root privillegs,I created new file under the sudoers.d directory
`sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader`

Edit the file and change the user name
`sudo nano /etc/sudoers.d/grader`

Using ssh-keygen, I have generated the xenial-key.pub file as c/Users/.ssh/grader-key
```
cd /home/grader
sudo mkdir .ssh
sudo touch .ssh/authorized_keys
sudo nano .ssh/authorized_keys
```

I have copied the contents of the pub file gnerated from the ssh-keygen to the authorized_keys file
from my shell local put this commaned
`cat .ssh/grader-key.pub`

then make this premsion cd then
```
sudo chmod 700 /home/grader/.ssh
sudo chmod 644 /home/grader/.ssh/authorized_keys
sudo chown -R grader:grader /home/grader/.ssh
```

if you want to make a password for this user
`sudo passwd grader`


then run
`sudo service ssh restart`

try login with in new local shell
`ssh grader@18.185.95.250 -p 2200 -i ~/.ssh/grader-key`


## Prevent Root Login using SSH:
`sudo nano /etc/ssh/sshd_config`

Locate the following line: '#PermitRootLogin prohibit-password' , put these lines next to it
```
PermitRootLogin no
AllowUsers grader
```
Lastly restart the ssh service
`sudo service ssh restart`


## Install and configure PostgreSQL

to install the dataabse, run the command
`sudo apt install postgresql postgresql-contrib`

-switch to Postgresdb using:
`sudo -i -u postgres`

type
`psql`

Now,create a new database user with least privilleges.
`CREATE DATABASE catalog;`

Now,create a user
`CREATE USER catalog;`

Set a password for user catalog which is alaa222
`ALTER ROLE catalog WITH PASSWORD 'YourPassWord';`

Grant privilleges to catalogdb to catalog user
`GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`

Quit postgreSQL
`\q`

Exit from user "postgres"
`exit`


**if you want to delete a live database**

First, delete the Database File
`rm -v database_setup.pyc`

Then
`REVOKE CONNECT ON DATABASE catalog FROM public;`

Then put this query
```
SELECT
    pg_terminate_backend(pid)
FROM
    pg_stat_activity
WHERE
    -- don't kill my own connection!
    pid <> pg_backend_pid()
    -- don't kill the connections to other databases
    AND datname = 'catalog'
    ;
```

Then delete the database by this
`DROP DATABASE catalog;`

you can delete the user by this 
`DROP USER catalog;`

also you can delete the password for user by this
`alter role postgres password null;`
 
## Python Version

Use this in your code application file to see python version you used:

```
import platform
print platform.python_version()
```

## Install some Requirements
Install Git using
`sudo apt-get install git`

Install Apache
`sudo apt-get install apache2`

Install mod_wsgi
`sudo apt-get install python-setuptools`
`sudo apt-get install libapache2-mod-wsgi`

Restart Apache
`sudo /etc/init.d/apache2 start`
`sudo service apache2 restart`

## Deloying catalog Porject to the server:
Create a folder catalog under the /var/www directory
`cd /var/www`

connect to the remote github repository to pull the project files
`sudo git clone https://github.com/Alaa-1989/Item-Catalog-Project catalog`

#### if you clone your project in wrong file
if you want delete repo
`sudo rm -R catalog/*`

if you want delete a dir
`sudo rm -r catalog`

## Virtualenironment:

install pip
`sudo apt-get install python-pip`

Install pyhton
`sudo apt-get install python`

Instal Virtual Environments
`sudo apt-get install python-virtualenv`

Install virtual environment for Python
```
sudo apt-get install python-venv
mkdir ~/virtualenvironment
virtualenv ~/virtualenvironment/my_catalog_app
cd ~/virtualenvironment/my_catalog_app/bin
source activate
```

## Installing Some Requirement

after activating the environment install all the dependecies.

```
sudo python -m pip install sqlalchemy
sudo python -m pip install flask
sudo python -m pip install psycopg2-binary
sudo python -m pip install Pillow
sudo -H python -m pip install oauth2client
sudo python -m pip install requests
```
log out from environment
`deactivate`

## Configure and Enable a New Virtual Host
Create FlaskApp.conf and edit it
`sudo nano /etc/apache2/sites-available/FlaskApp.conf`

Add the following to the file to configure the virtual host.

```
<VirtualHost *:80>
             ServerName 18.185.95.250.xip.io
             ServerAdmin 18.185.95.250
             WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/sit$
             WSGIProcessGroup catalog
             WSGIScriptAlias / /var/www/catalog/flaskapp.wsgi
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
Enable the virtual host with the following command: sudo a2ensite FlaskApp


## Create wsgi File :
Create the wsgi File under /var/www/catalog

`cd /var/www/catalog`
`sudo nano flaskapp.wsgi `
`
**Add the following to the flaskapp.wsgi file**
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/catalog")

from __init__ import app as application
application.secret_key = 'YOUR-SECRET-KEY'
```

## configuration changes in the code app
## In this file `__init__.py`

- Add this code
```
app = Flask(__name__)
APP_PATH = '/var/www/catalog/catalog/'
app.secret_key = 'your secret key'
```
- Adding the path for json file
```
CLIENT_ID = json.loads(
    open('/var/www/catalog/catalog/g_client_secret_.json', 'r').read())['web']['client_id']
APPLICATION_NAME = "Item Catalog"
```
- Change the engine with this code
```
engine = create_engine('postgresql://catalog:alaa222@localhost/catalog')
```
- Delete host and port fron app tun
```
if __name__ == '__main__':
    app.debug = True
    app.run()
```
## In this file `database_setup.py`

- Change the engine with this code
```
engine = create_engine('postgresql://catalog:alaa222@localhost/catalog')
```

## In this file `seeder.py`

- Change the engine with this code
```
engine = create_engine('postgresql://catalog:alaa222@localhost/catalog')
```
## In this file `g_client_secret_.json`

- we redownloaded after make the changes in google console after add the DNS




**change the authorized URI host and add DNS**

![pic1](https://user-images.githubusercontent.com/42184553/50744003-6f34b980-122f-11e9-9f21-332598f37b95.JPG)

**Change the redirect URIs with our IP and DNS**

![pic2](https://user-images.githubusercontent.com/42184553/50744029-b91d9f80-122f-11e9-8196-419c8d91a68e.JPG)


**First add the Authorized domains from "OAuth consent screen" we uesed [xip.io](http://xip.io/)**

![pic3](https://user-images.githubusercontent.com/42184553/50744057-013cc200-1230-11e9-9704-a29d5cc4da53.JPG)

## Run The App:

Restart Apache by this command
`sudo apache2ctl restart`
`sudo service apache2 restart`

open your project file
`cd /var/www/catalog/catalog`

to created the database tables
`python database_setup.py`

to inserted the data
`python seeder.py`

to ran the application file
`python __init__.py`

## In the browser run this url:
http://18.185.95.250.xip.io/

## Useful commands
To get log error messages from Apache server:
`sudo tail /var/log/apache2/error.log`

## Author
**Alaa Alaboud** - _GitHub Profile:_ [Alaa-1989](https://github.com/Alaa-1989)

## License
Licensed under the [MIT License](LICENSE)
