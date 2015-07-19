# Configure a Flask Application on an Ubuntu server
  This is a part of course at Udacity

### How to access
- IP:52.26.7.82
- Port:2200
- Link:http://ec2-52-26-7-82.us-west-2.compute.amazonaws.com/

### Procedure
```
#upgrade system
sudo apt-get update
sudo apt-get upgrade

#create user grader with sudo premission
sudo adduser grader
sudo adduser grader sudo

#make ssh key on client
cd ~/.ssh
ssh-keygen -t rsa -f grader_catalog
pbcopy < grader_catalog.pub

#swich to grader
su grader
#create .ssh dir
mkdir .ssh
sudo chmod 700 .ssh

#make key file
nano .ssh/authorized_keys
#copy the key to authorized_key and save
#logout grader
exit

#change ssh port 22 to 2200
sudo nano /etc/ssh/sshd_config
#find "Port 22" and change to "Port 2200"
#restart ssh server
sudo service ssh restart
#logout server
logout

#login to server
ssh -i ~/.ssh/grader_catalog grader@52.26.7.82 -p 2200

#setup firewall
sudo ufw allow 80
sudo ufw allow 2200
sudo ufw allow 123
sudo ufw enable

#check timezone
date
# timezone is already set to UTC - do nothing

#install neccessary libraries
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi python-dev
sudo a2enmod wsgi 
sudo apt-get install python-pip 
sudo pip install virtualenv

#clone source code from github
cd /var/www
sudo git clone https://github.com/edmondliang/udacity_catalog.git
sudo mv udacity_catalog catalog

#create python virtual enviroment
cd catalog
sudo virtualenv env
source env/bin/activate

#install python dependencies
sudo pip install -r requirement.txt
#download google oauth secret file from your google consle(https://console.developers.google.com/)
#open google secret file and copy the content
sudo nano client_secret.json
#copy the content and save

#check if work
python run.py
#Ctrl+c to exit
#exit virtual enviroment
deactivate

#create virtualhost
sudo nano /etc/apache2/sites-available/catalog.conf 
#copy the content below and save
#---------------------
<VirtualHost *:80>
                ServerName localhost
                ServerAdmin edmond.liang.sf@gmail.com
                WSGIScriptAlias / /var/www/catalog/catalog.wsgi
                <Directory /var/www/catalog/app/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /static /var/www/catalog/app/static
                <Directory /var/www/catalog/app/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
#---------------------

#create wsgi file
sudo nano /var/www/catalog/catalog.wsgi
#copy the content below and save
#---------------------
#!/usr/bin/python
import sys,os
import logging
logging.basicConfig(stream=sys.stderr)
PROJECT_DIR="/var/www/catalog"
sys.path.insert(0,PROJECT_DIR)
from app import app as application
#---------------------

#activate catalog.conf
sudo a2ensite catalog
sudo a2dissite 000-default
#reboot apache2
sudo service apache2 restart

#check error
cat /var/log/apache2/error.log

#install Postgresql
sudo apt-get install postgresql postgresql-contrib python-psycopg2

#change 
sudo nano /etc/postgresql/9.3/main/pg_hba.conf 
#check if any remote access(There is no remote access allowed by default)
#---------------------
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
#---------------------

#enter postgresql
sudo su - postgres
psql

#create postgresql user
createuser -P -S catalog --interactive
#Enter password for new role: 
#Enter it again: 
#Shall the new role be allowed to create databases? (y/n) n
#Shall the new role be allowed to create more new roles? (y/n) n

#create postgresql database
createdb -U catalog --locale=en_US.utf-8 -E utf-8 -O catalog app -T template0

#change database connection
nano /var/www/catalog/config.py
#find "SQLALCHEMY_DATABASE_URI = 'sqlite:///' + os.path.join(BASE_DIR, 'app.db')"
#set "SQLALCHEMY_DATABASE_URI = 'postgresql://catalog:(password)@localhost/catalog"

```


### References
- https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
- http://www.enigmeta.com/2012/08/16/starting-flask/
- http://flask.pocoo.org/docs/0.10/deploying/mod_wsgi/