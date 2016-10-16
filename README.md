# GameSwap Linux Server configuration

## web app Url:
http://52.25.77.20
port 2200

##First steps:
Get your aws instance from udacity.
Download key.rsa and then move key to ~/.ssh/.
`mv ~/Downloads/udacity/_key.rsa ~/.ssh/`
Then change access to the key.
`chmod 600 ~/.ssh/udacity_key.rsa`
log into the server.
`ssh -i ~/.ssh/udacity_key.rsa root@00.00.00.00
Update packages list
`sudo apt-get update`
update system:
`sudo apt-get upgrade`

##setup User grader
Add user
`adduser grader`
enter editor
`visudo`
scroll down to `root ALL=(ALL:ALL) ALL`
add this line below it
`grader ALL=(ALL:ALL) ALL`
aves and exit(ctrl+x, y and press ENTER)

## setup firewall and setup ports.
enter the editor
type: `nano /etc/ssh/sshd_config`
change port 22 to 2200
change PermitRootLogin without-password to PermitRootLogin no
change PasswordAuthentication no to PasswordAuthentication yes
add UseDNS no
add AllowUsers grader
exit(ctrl+x, y ENTER)
restart 
`sudo service ssh restart`


##create keypair
in your local environment type
`ssh-keygen`
pick your keygen path
if you choose add a passphrase
copy ssh key
`cat ~/.ssh/keygen-pair.rsa`
access server
`ssh -i ~/.ssh/keygen-pair.rsa grader@00.00.00.00 -p 2200`
make a directory for the key
`mkdir .ssh`
add key gen to .ssh
`touch .ssh/authorized_keys`
`sudo nano .ssh/authorized_keys`
exit(ctrl+x, y ENTER)
change file permissions
chmod 700 .ssh
chmod 644 .ssh/authorized_keys

##remove unable to resolve host warning(after using sudo)
type `sudo nano /etc/hostname`
copy the ip string
exit (ctrl+x)
open hosts
`sudo nano /etc/hosts`
exit(ctrl+x, y, ENTER)
paste the ip right behind the address and before local host

##configure sshd
open sshd_config
`sudo nano /etc/ssh/sshd_config`
change PasswordAuthentication yes to PasswordAuthentication no`
exit(ctrl+x, y, ENTER)

##configure ufw
check status
`sudo ufw status`
deny all incoming by default
`sudo ufw default deny incoming`
all only the ports needed
`sudo ufw allow 2200/tcp`
`sudo ufw deny 22/tcp`
`sudo ufw allow 80/tcp`
`sudo ufw allow 123/udp`
turn the firewall on 
`sudo ufw enable`
##set time zone
`sudo dpkg-reconfigure tzdata`
hit ENTER
select None of the above 
hit ENTER
hit ENTER
select UTC 
hit ENTER
used for better time on the server
`sudo apt-get install ntp`

##Install apache2
`sudo apt-get install apache2`
to check to see if it works. In the browser go to http://00.00.00.00
`sudo apt-get install python-setuptools libapache2-mod-wsgi`
configure apache to handle wsgi
`sudo nano cat /etc/apache2/sites-enabled/000-default.conf`
add WSGIScriptAlias / /var/www/html/myapp.wsgi in between <VirtualHost *:80></VirtualHost>
exit(ctrl+x, y, ENTER)
restart
sudo apache2ctl restart
create apache config file
`echo "ServerName 00.00.00.00" | sudo tee /etc/apache2/conf-available/fqdn.conf`
`sudo a2enconf fqdn`
reload apache2
`sudo service apache2 reload`


##Insatll git
`sudo apt-get install git`
set up git config
`git config -- global user.name "YourName"`
`git config -- global user.email "youremail@gmail.com"`

## install package to enable apache serve Flask
`sudo apt-get install libapache2-mod-wsgi python-dev`

navigate www directory
`cd  /var/www`
add directory Catalog
`mkdir Catalog`
navigate into Catalog
`cd Catalog`
make another directory named catalog
`sudo mkdir catalog`
navigate into catalog
`cd catalog`
add templates and static folders
`sudo mkdir static templates`
create init logic file
`sudo nano _\_\init\_\_.py`
add this to the file
`from flask import Flask
app = Flask(__name__)
@app.route("/")
def hello():
    return "Hello, Catalog app coming up soon!"
if __name__ == "__main__":
  app.run()
`
test flask out on server

## Installation of packages 
installl pip
`sudo apt-get install python-pip`
install virtual environment
`sudo apt-get install virtualenv`
create a new virtual environment
`sudo virtualenv venv`
activate virtual environment
`source venv/bin/activate`
install Flask
'pip install Flask'
test flask app
`python __init__.py`
deactivate venv
`deactivate`
create virtual host config files
`sudo nano /etc/apache2/sites-available/catalog.conf`
paste this into file
`<VirtualHost *:80>
    ServerName 52.25.77.20
    ServerAdmin admin@52.25.77.20
    ServerAlias ec2-52-25-77-20.us-west-2.compute.amazonaws.com
    WSGIScriptAlias / /var/www/Catalog/catalog.wsgi
    <Directory /var/www/Catalog/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/Catalog/catalog/static
    <Directory /var/www/Catalog/catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>`
* to get ServerAlias go to hcidata.info/host2ip.cgi
exit(ctrl+x, y ENTER)
sudo a2ensite catalog
create .wsgi file
put this in a file
`import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/Catalog/catalog/")
from catalog import app as application
application.secret_key = "add_your_secret_key"`
restart apache2
`sudo apache2ctl restart`
## clone project
`git clone https://github.com/phyrenight/up3.git`
move all content into  Catalog
`mv up3/* /var/www/Catalog/catalog`

##render inaccessible
create a .htaccess file
`sudo nano .htaccess`
add redirect to file
`RedirectMatch 404 /\.git

##Install software to run app
activae venv
`source venv/bin/activate`
add packages
`sudo apt-get install python-setuptools
sudo pip install Flask
pip install httplib2
pip install requests
sudo pip install flask-seasurf
pip install sqlalchemy
sudo pip install oauth2client
sudo apt-get install postgresql postgresql-contrib
sudo apt-get install libpg-dev
pip install psycopg2'
restart apache2
`sudo apache2ctl restart`

##Set up postgres
make sure no remote connections are allowed
`sudo nano /etc/postgresql/9.3/main/pg`
create a new user for postgresql
`sudo adduser catalog`
make a password for uer
change to postgres user
`sudo su - postgres`
run postgresql
`psql`
create user catalog with password
`CREATE USER catalog WITH PASSWORD 'password';`
make catalog able to make tables
`ALTER USER catalog CREATEDB;`
create a new db called catalog
`CREATE DATABASE catalog WITH OWNER catalog;`
revoke database schema
`REVOKE ALL ON SCHEMA public FROM public;`
grant access to catalog
`GRANT ALL ON SCHREMA public to catalog;`
exit postgresql
`\q`
run database setup file
`python database_setup.py`
restart apache2
`sudo apache2ctl restart`
setup your Oauth2 with your Oauth2 providers
