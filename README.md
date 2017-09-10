# Linux_Server
Configuring linux web servers to secure and set up a Linux server. The instructions are written to run an app called Catalog, live on a secure web server using an Amazon Lightsail.

# Server Details
 IP address : 13.126.183.240 <br>
 SSH port : 2200 <br>
 URLs : http://13.126.183.240/<br>
        http://ec2-13-126-183-240.ap-south-1.compute.amazonaws.com/<br>
#### Note that since I have now graduated, the original server has been disabled. You might find something else there now.
        
# Configuration

## Creating Amazon Lightsail Instance
1. Log in!
If you don't already have an Amazon Web Services account, create a new account.
2. Create an instance.
Once you're logged in, create an instance. A Lightsail instance is a Linux server running on a virtual machine inside an Amazon datacenter.
3. Choose an instance plan.
4. Give the instance a hostname.
5. Wait for instance to start up.
  It may take a few minutes for the instance to start up.

## Connect to the instance on local machine
* Download the SSH key for your server.
* Open a new terminal window on your local system.
* Set the permissions for your private key file to 600 using a command `chmod 600 KEYFILE`.
* Connect to the server using the following command: `ssh -i KEYFILE ubuntu@SERVER-IP`

## Updates
`sudo apt-get update` <br>
`sudo apt-get upgrade`

## Change the SSH port from 22 to 2200
- Edit `nano /etc/ssh/sshd_config` to change the SSH port from 22 to 2200. 
- Restart the server `sudo service ssh restart`

## Configure Firewall  
HTTP (port 80), and NTP (port 123)
- Check current UFW status `sudo ufw status` This will show that UFW is Inactive.
- Set-up UFW using these commands
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 2200/tcp 
sudo ufw allow www
sudo ufw allow 123/udp
sudo ufw deny 22
sudo ufw enable
```
- Run `sudo ufw status` This will show which ports are open and the ufw will have the settings below:

```
To                         Action      From
--                         ------      ----
22                         DENY        Anywhere
2200/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
123/udp                    ALLOW       Anywhere
22 (v6)                    DENY        Anywhere (v6)
2200/tcp (v6)              ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
123/udp (v6)               ALLOW       Anywhere (v6)
```
- Update the external (Amazon Lightsail) firewall on the browser to match the internal firewall settings (only ports 80(TCP), 123(UDP), and 2200(TCP) should be allowed; make sure to deny the default port 22)

- Confirm that root can SSH and login from local computer, 
`ssh -i ~/path/KEYFILE -p 2200 ubuntu@AWS_IP_ADDRESS`

## Create a new user
As root, type `sudo adduser grader`. Then follow prompts to add a password and fill the details.

## Give grader sudo permission
* As the root user, type `sudo visudo`
* Add the following line below `root ALL=(ALL:ALL) ALL`: <br>
  `grader ALL=(ALL:ALL) ALL`

##  Set-up Key-based Authentication
* Run ssh-keygen on the local machine and create a key for the user grader.
* In grader's home directory, and create a new directory called .ssh. Run : `mkdir .ssh`
* Run `touch .ssh/authorized_keys`
* Copy the contents of the .pub file, and paste them in the .ssh/authorized_keys file.
* Set the permissions by running:<br>
  `chmod 700 .ssh` <br>
  `chmod 644 .ssh/authorized_keys`
* On the server, logged in as the grader, edit the sshd_config file <br> 
`sudo nano /etc/ssh/sshd_config` 
* Change the line with Password Authentication from yes to no
* Restart the ssh service <br> 
`sudo service ssh restart`
* Log in as the grader using the following command:<br>
  `ssh -i ~/.ssh/KEYFILE -p 2200 grader@XX.XX.XX.XX`
  
## Disable remote login of root user
* On the server, logged in as root, edit the sshd_config file <br>
  `sudo nano /etc/ssh/sshd_config`
* Change to `PermitRootLogin no`
* Add `AllowUsers grader`
* Restart the ssh service
  `sudo service ssh restart`
  
## Configure local Timezone to UTC
- Run `sudo dpkg-reconfigure tzdata`, and follow the instructions.
- Confirm time change by typing `date` on the command line

##  Install and configure Apache 
* Install Apache 
`sudo apt-get install apache2` <br>
* Confirm successful installation by visiting http://13.126.183.240. It should say "It Works" and display other Apache information on the page.
* Install Python mod_wsgi `sudo apt-get install libapache2-mod-wsgi` 
* Make sure mod_wsgi is enabled by running `sudo a2enmod wsgi`.

## Install and configure PostgreSQL
* Install PostgreSQL by running `sudo apt-get install postgresql`
* Check that remote connections are not allowed <br>
`sudo less /etc/postgresql/9.3/main/pg_hba.conf`
* Basic server set-up <br> 
`sudo -u postgres psql postgres`
* Set-up a password for user postgres <br>
`\password postgres` and enter a password

## Create a new database user catalog
* Connect to database as the user postgres
`sudo su - postgres` 
* Type `psql` to generate PostgreSQL prompt
* Create a new user <br> 
`CREATE USER catalog WITH PASSWORD 'your_passwd';`
* Confirm that the user was created by running `\du`
* Change permissions for catalog user: <br>
```
ALTER ROLE catalog WITH LOGIN;
ALTER USER catalog CREATEDB;
```
* Create catalog database `CREATE DATABASE catalog WITH OWNER catalog;`
* Login to the database `\c catalog`
* Revoke all the rights : `REVOKE ALL ON SCHEMA public FROM public;`
* Grant the access to catalog: `GRANT ALL ON SCHEMA public TO catalog;`
* Exit Postgresql using `\q`then exit from postgresql user.
* Restart postgresql -<br> `sudo service postgresql restart`

## Install Git and clone project
* Run `sudo apt-get install git`
* Create a directory catalog in the /var/www directory.
* Cone the catalog project in the /var/www/catalog directory by running :<br> 
`sudo git clone https://github.com/Rashi406/Catalog-App.git catalog`
* Rename project.py file to __init__.py by running `mv project.py __init__.py`
* In __init__.py, replace `app.run(host='0.0.0.0', port=8000)` to `app.run()`
* At the root of the web directory, add a .htaccess file and include this line: <br> 
`RedirectMatch 404 /\.git`

## Create a virtual environment and install dependencies
```
sudo apt-get install python-pip
sudo pip install virtualenv
```
* `cd` to /var/www/catalog/catalog/ directory; choose a name for temporary environment ('venv' is used in this example), and create this environment by running `sudo virtualenv venv`.
* Activate the virtual environment with the following command <br>
`source venv/bin/activate`
* Install required packages:
```
sudo pip install Flask
sudo pip install sqlalchemy
sudo pip install Flask-SQLAlchemy
sudo pip install psycopg2
sudo apt-get install python-psycopg2 
sudo pip install flask-seasurf
sudo pip install oauth2client
sudo pip install httplib2
sudo pip install requests
```
* Run `sudo python __init__.py` to check if everything was installed correctly.
* Deactivate the virtual environment by using  `deactivate`.
* Create a file
`sudo nano /etc/apache2/sites-available/catalog.conf`
Add following in the catalog.conf file
``` 
<VirtualHost *:80>
      ServerName PUBLIC-IP-ADDRESS
      ServerAdmin admin@PUBLIC-IP-ADDRESS
      ServerAlias ec2-XX-XX-XX-XX.XXX.compute.amazonaws.com
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
* Run `sudo a2ensite catalog` to enable the virtual host
* Create a WSGI file
```
cd /var/www/catalog
sudo nano catalog.wsgi
```
* Add the following to the file:
```
#!/usr/bin/python 
import sys 
import logging 

logging.basicConfig(stream=sys.stderr) 
sys.path.insert(0,"/var/www/catalog/")  

from catalog import app as application 
application.secret_key = 'secret_key'
```
* Run `sudo service apache2 restart` to restart Apache

## Switch the database from SQLite to PostgreSQL
* Replace `engine = create_engine('sqlite:///catalog.db')` with  `engine create_engine('postgresql://catalog:catalog_passwd@localhost/catalog')` 

## Disable default Apache site
* Run `sudo a2dissite 000-default.conf`
* Run `sudo service apache2 reload`

## Update OAuth Information for Google
Authenticate login through Google:

- Create a new project on the Google API Console
- Create an OAuth Client ID, and add http://XX.XX.XX.XX and http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com in authorized JavaScript origins
- Add http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com/gconnect and http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com/oauth2callback in authorized redirect URIs.
- Download the client_secrets_XXXXXXXXXX.json file.
- Replace the contents of file called client_secrets.json with contents of client_secrets_XXXXXXXXXX.json file.
- Change the client ID in templates/login.html file.
- Downgrad packages to enable Google+ Login
  ```
  pip install werkzeug==0.8.3 
  pip install flask==0.9 
  pip install Flask-Login==0.1.3
  ```
- Use the full path to client_secrets.json in the __init__.py file. 

## Steps to display the site
* Activate the virtualenv by running . venv/bin/activate.
* Run `python items.py`
* Deactivate the virtualenv.
* Add EC2 URL (without the http://) to 'hosts' file <br>
`sudo nano /etc/hosts` 
* Check what sites are enabled <br>
`sudo ls -alh /etc/apache2/sites-enabled/`
- Restart Apache <br>
`sudo service apache2 restart`
- Run app <br>
`sudo python __init__.py`

## Application status monitoring
- Install Glances 
`sudo pip install glances`
- Run Glances by typing `glances`
 
# Softwares Installed
- Apache2
- mod_wsgi
- PostgreSQL
- Git
- pip
- virtualenv
- httplib2
- Python Requests
- Oauth2client
- SQLAlchemy
- Flask
- Psycopg2
- Glances

# Resources
* Udacity course: [Configuring Linux Web Servers](https://www.udacity.com/course/configuring-linux-web-servers--ud299)
* Bitnami Documentation Pages : [Frequently Asked Questions For AWS Cloud](https://docs.bitnami.com/aws/faq/)
* DigitalOcean tutorials : [How To Add and Delete Users on an Ubuntu 14.04 VPS](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps)
* AWS : [Discussion Forums](https://forums.aws.amazon.com/thread.jspa?threadID=244112)
* ask ubuntu : [How do I change my timezone to UTC/GMT?](https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt)
* DigitalOcean tutorials : [How To Secure PostgreSQL on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)<br>
[How To Install and Use PostgreSQL on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04)
* serverfault : [How do I prevent apache from serving the .git directory?](https://serverfault.com/questions/128069/how-do-i-prevent-apache-from-serving-the-git-directory)
* DigitalOcean tutorials : [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
* SQLAlchemy
* Udacity Discussion forums
* Stack Overflow
