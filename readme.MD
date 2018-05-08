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
1. I followed the steps of kongling893 (https://github.com/kongling893/Linux-Server-Configuration-UDACITY)
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


# Notes from the class are copied below:
# Additional notes taken from class
## To disable password based authentication, so that users or forced to log-in via public-private key, I need to disable to password configuration, log in-system. To do this, run the following command to open the following file `sudo nano /etc/ssh/sshd_config` Change the line that says PasswordAuthentication from `yes` to `no`

















