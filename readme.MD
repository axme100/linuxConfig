# Linux Server Configuration Project

## I.P Address and port to acsess server via SSH
grader@52.65.31.177 -p 2200

## URL to hosted web application
`http://ec2-52-65-31-177.ap-southeast-2.compute.amazonaws.com/`

# Steps taken to complete this project/reference

## Update the available package list
`sudo apt-get update`

## Upgrade those packages from the previous list
`sudo apt-get upgrade`

## Download the default key of the instance and use it to login to the system
Note: Before I could log on to the instance, I needed to change the permissions of the default private key
that I had saved on my local machine. (See post: https://discussions.udacity.com/t/warning-unprotected-private-key-file/369966) `sudo chmod 600 /Users/maxcarey/.ssh/LightsailDefaultPrivateKey-ap-southeast-2.pem`

## Log-in on default port 22 using the default key
`ssh ubuntu@52.65.31.177 -p 22 -i /Users/maxcarey/.ssh/LightsailDefaultPrivateKey-ap-southeast-2.pem`

## Use ssh key-gen to create a new public private key and upload it to the server. Once done, make sure you can login with:
`ssh ubuntu@52.65.31.177 -p 22 -i /Users/maxcarey/.ssh/linuxProj`

## Add port to 2200 in the sshd_config file (keep both of the ports up and test them before deleting 22). Open the configuration file with:
`sudo nano /etc/ssh/sshd_config`

## On the Lightsail interface allow create a `CUSTOM`rule for allowing port `2200`. (Note: it doesn't say SSH necessarily)

## Since ssh configuration file is modified, we need to run the following command to restart the service:
`sudo service ssh restart`

## Make sure you can log in via both ports
`ssh ubuntu@52.65.31.177 -p 22 -i pathToFile`
`ssh ubuntu@52.65.31.177 -p 2200 -i pathToFile`

## Delete port 22 in configuration file

## Configure uncomplicated firewall
1. Establish default rule for incoming connections `sudo ufw default deny incoming`
2. Establish a default rule for outgoing requests `sudo ufw default allow outgoing`
3. Open port 2200 (Since SSH is going to be on port 2200, which is not the default we must run `sudo ufw allow 2200`. On the contrary, `sudo ufw allow ssh` would automatically open port 22.
4. Open NTP and HTTP ports 
`sudo ufw allow ntp` (port 80)
`sudo ufw allow htp` (123)
5. Enabnle Firewall
`sudo ufw enable`

## Create a new user called grader
`sudo adduser grader`
Give them a password of "grader" and leave all the other settings blank

## Add this new user to the sudo list
`sudo usermod -aG sudo grader`

## Switch into the new user to and list the contents of the root directory to make sure they have the right permissions
`su - grader`
`sudo ls -al /`
(Question: when running `su - ubuntu` to switch back, it looks like I never stored a password for the Ubuntu user)

## On local machine generate a key-pair (always do this on local machine so that private key is ALWAYS private)
`ssh-keygen`
Don't give the key a password

## On the server, in the location /home/grader
`sudo nano .ssh/authorized_keys Paste in the puublic key in one line in the file authorized_keys`

## Change some of the permissions on these newly created keys
`sudo chmod700 .ssh`
`sudo chmod644 authorized_keys`

## Reference for octal permissions
r = 4 (read)
w = 2 (write)
x = 1 (execute)

## Set owner and group of each of these files to grader
`sudo chown grader .ssh`
`sudo chgrp grader .ssh`
`sudo chown grader .ssh/authorized_keys`
`sudo chgrp grader .ssh/authorized_keys`

## Change to Universal Coordinate Time
### Follow the instructions here:
https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa
(Already done)

## Install the Apache HTTP Server
`sudo apt-get install apache2`

## Use the I.P address at the top of this file to see if the default apache server index.html `/var/www/html/index.html`

##Install Apache 2 application handler (for Python 3 in my case)
Python 3: `sudo apt-get install libapache2-mod-wsgi-py3`
Python 2: `sudo apt-get install libapache2-mod-wsgi`

## Restart Apache
sudo service apache2 restart

## Install postgresql
sudo apt-get install postgresql

## Check that no remote connections are allowed:
Follow instructions here: https://discussions.udacity.com/t/disallowing-remote-connections-to-my-postgresql-db/36364/2
(Either way we can see by default that the file does not allow any remote connections so no changes are necessary)
`sudo nano /etc/postgresql/9.5/main/pg_hba.conf`

## Follow kongling893 to set up catalog database, configure apache to handle requests to custom handler and to create the wsgi file (https://github.com/kongling893/Linux-Server-Configuration-UDACITY). Many custom steps had to be taken especially when configuring the "catalog app" (see below)

## Switch user to posgres
`sudo su - postgres`

## Get into psql shell
`psql`

## Create a catalog database
`CREATE DATABASE catalog;`

## Add a user called catalog
`CREATE USER catalog;`

## Give that user a password
`ALTER ROLE catalog WITH PASSWORD 'cataloghorsechair';`

## Grant all privelages to that user
`GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`

## Configure apache to hand off requests to the applcation handler. (This can be done in the file `/etc/apache2/sites-enabled/000-default.conf`.
1. `sudo nano /etc/apache2/sites-available/FlaskApp.conf`
2. Add the following code to the virtual host:
`<VirtualHost *:80>
	ServerName MyServerName
	ServerAdmin MyEmail
	WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
	<Directory /var/www/FlaskApp/FlaskApp/>
		Order allow,deny
		Allow from all
	</Directory>
	Alias /static /var/www/FlaskApp/FlaskApp/static
	ErrorLog ${APACHE_LOG_DIR}/error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>`
* Note that I removed the <directory> for static as my app doesn't have any static CSS.
3. Enable virtual host: `sudo a2ensite FlaskApp`

## Configure "Catalog App": Install git, clone project, and make adjustments (Steps adapted from kongling893)
1. Install Git using `sudo apt-get install git`
2. Use `cd /var/www` to move to the /var/www directory 
3. Create the application directory `sudo mkdir FlaskApp`
4. Move inside this directory using `cd FlaskApp`
5. Clone the Catalog App to the virtual machine `git clone https://github.com/kongling893/Item_Catalog_UDACITY.git`
6. Rename the project's name `sudo mv ./Item_Catalog_UDACITY ./FlaskApp`
7. Move to the inner FlaskApp directory using `cd FlaskApp`
8. Rename `website.py` to `__init__.py` using `sudo mv website.py __init__.py`
9. Edit `database_setup.py`, `website.py` and change `engine = create_engine('sqlite:///storyAndWord.db')` to `engine = create_engine('postgresql://catalog:<password created in posgresql console>@localhost/catalog')`
10. Install pip `sudo apt-get install python-pip3`
11. Since I didn't create a requirements.txt file on my local machine, use PIGAR to automatically create one: (https://github.com/Damnever/pigar)
12. Use pip to install dependencies `sudo -H pip3 install -r requirements.txt` (The -H makes sure that all of the packages get installed on the root directory and not the user ubuntu)
12. At: https://console.developers.google.com/apis add new JavaScript path to the default domain name.
13. Change path of CLIENT_SECRETS file to ABSOLUTE PATH both at top of __init.py__ folder and inside the gConnect() method.
14. Add a sheBang line to the top of __init__.py

## Create the .wsgi File
`cd /var/www/FlaskApp`
`sudo nano flaskapp.wsgi`

## Add the following lines of code to the wsgi
`
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")

from FlaskApp import app as application
application.secret_key = 'PUT IN SECRET KEY FOUND AT BOTTOM OF __init__.py file'
`
## Restart APACHE 
`sudo service apache2 restart`


# For debugging

## Display outputs of apache2 error log file
`cat /var/log/apache2/error.log`

## Reload apache2 server
`sudo service apache2 reload`

# Notes from the class are copied below:
# Additional notes taken from class
## To disable password based authentication, so that users or forced to log-in via public-private key, I need to disable to password configuration, log in-system. To do this, run the following command to open the following file `sudo nano /etc/ssh/sshd_config` Change the line that says PasswordAuthentication from `yes` to `no`