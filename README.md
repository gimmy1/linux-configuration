# INFORMATION
- IP ADDRESS: 54.172.170.196
- URL: http://ec2-54-172-170-196.compute-1.amazonaws.com/
- AVAILABLE PORTS: 2200, 80, 123

# WALKTHROUGH

### 1 - Create a new Amazon LightSail Instance
1. Create an instance. Choose an instance image:Ubuntu, give your instance a hostname.
2. When you ssh you will be logged as the ubuntu user.
3. Add a custom port to 2200.
4. -Download the default private key and save it in your computer in form of .pem.
5. chmod 400 .pem - Now you will have the right access to log-in from your local machine with `ssh -i ~/.ssh/.pem ubuntu@54.164.142.228`

### 2 - Change SSH port from 22 to 2200
1. `sudo nano /etc/ssh/sshd_config`
2. Change Port 22 to Port 2200
3. `sudo service ssh restart`
4. Now you wont be able to use the Amazon Lightsail terminal and can only log-in through your local machine.
5. Use this command to log-in `ssh -i ~/.ssh/.pem ubuntu@54.164.142.228 -p 2200`


### 3 - Give grader access (create a new user grader and grant them sudo permissions)
1. Log into VM on your local machine as ubuntu user
2. Add a new user grader: `sudo adduser grader`
3. Add Full Name: grader. Skip rest of options
4. Add sudo permissions for grader `sudo usermod -aG sudo grader`
5. Log-in as grader `su - grader`
6. You are now logged-in remotely as grader and can use `sudo`
7. Create .ssh and authorized_keys
- `mkdir .ssh`
- `touch .ssh/authorized_keys`
- `sudo chmod 700 .ssh`
- `sudo chmod 644 .ssh/authorized_keys`
8. Create key on local machine and not on server - you can not claim it is secure if you create it on the server or remotely logged-in!
- `ssh-keygen`
- name it linuxCourse
- you will have two new files: `linuxCourse` and `linuxCourse.pub`
- open linuxCourse.pub `cat linuxCourse.pub` and copy the contents of the file
- open authorized_keys `sudo nano .ssh/authorized_keys` and paste. Save file.
- exit out of grader and ubuntu.
- run `ssh -i ~/.ssh/linuxCourse grader@54.164.142.228 -p 2200` and you are now logged in as grader!


### 4 - Update all currently installed packages
* All available package sources are in this file `cat /etc/apt/sources.list`

1. The first step to upgrading installed software is to update package source list. Type this command: `sudo apt-get update`. This will search through all of the installed packages and see what software it is and versions they are. Note - this does not actually commit any changes. It lets your system aware of all the latest information stored within all repositories that we're making use of.
2. Update software repositories with this command `sudo apt-get upgrade`


### 4 - Configure UFW (Uncomplicated Firewall) to allow only incoming connections for SSH(2200), HTTP(80), and NTP(123).
1. On Amazon LightSail Gui, click on networking and add add Port Range 123 for incoming connections - this allows you to control which ports on this instance accept connections.
2. check firewall status `sudo ufw status`
3. block all incoming connections on all ports `sudo ufw default deny incoming`
4. allow outgoing connections on all ports `sudo ufw default allow outgoing`
5. allow incoming for Port 2200 `sudo ufw allow 2200`
6. allow incoming for Port 80 `sudo ufw allow 80`
7. allow incoming for Port 123 `sudo ufw allow 123`
8. check ports just added `sudo ufw show added`
9. enable firewall `sudo ufw enable`
10. check status of firewall `sudo ufw status`
- `Status active`

### 5 - Configure the local Timezone
1. `sudo dpkg-reconfigure tzdata`
- choose `none of the above`
- set to 'utc'

### 6 - Install Apache / Install mod_wsgi / Install Flask / Restart Apache
1. `sudo apt-get install apache2`
2. To make sure it runs use the Public IP address offered by Amazon Lightsail. You will now see the default Ubuntu page.
3. `sudo apt-get install libapache2-mod_wsgi python-dev`
4. `sudo apt-get install python-flask`
5. `sudo apt-get upgrade`
4. `sudo service apache2 restart`


### 7 - Configure Flask Website to ensure Flask will work
1. `cd /var/www`
2. `sudo mkdir FlaskApps`
3. `cd FlaskApps`
4. `sudo mkdir SimpleApp`
5. Now to edit our main file and add a simple Flask program
- `sudo nano /var/www/FlaskApps/SimpleApp/home.py`
- ``` from flask import Flask
app=Flask(__name__)

@app.route('/')
def home():
    return  "This is from Flask!!!"

if __name__ == "__main__":
    app.run()```

6. Edit the config file to point to our new Flask site
- `sudo nano /etc/apache2/sites-available/SimpleApp.conf`
- Paste this
```
<VirtualHost *:80>
    ServerName http://54.164.142.228/
    ServerAdmin admin@mywebsite.com
    WSGIScriptAlias / /var/www/FlaskApps/FlaskApps.wsgi
    <Directory /var/www/FlaskApps/SimpleApp/>
        Order allow,deny
        Allow from all
    </Directory>
    <Directory /var/www/FlaskApps/SimpleApp/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
7. `sudo a2enmod wsgi`
8. `sudo apachectl restart`
9. `sudo a2ensite SimpleApp`
10. `service apache2 reload`
11. Now we have to create a Web Server Gateway Interface (WSGI) to tell Apache how to run Flask - it is the link and entry point of our application.
- `sudo nano /var/www/FlaskApps/SimpleApp.wsgi`

### 8 - Deploy Flask application
1. `cd /var/www`
2. `sudo mkdir catalog`
3. `cd catalog`
4. `sudo mkdir catalog`
5. `sudo mkdir templates static`
6. `sudo nano home.py`
7. Add the following to the home.py file
```
from flask import Flask
app=Flask(__name__)

@app.route('/')
def home():
    return  "This is from Flask!!!"

if __name__ == "__main__":
    app.run()
```
- save the file
8. Create a Virtual Environment
- `sudo apt-get install python-pip`
- `sudo pip install virtualenv`
- Set the virtual environment name `sudo virtualenv venv`
- Enable virtual environment `source venv/bin/activate` and install Flask `sudo pip install Flask`
- Run the file `sudo python home.py`. This will claim that your file is running "Running on http://http://54.172.170.196/". This will indicate that you have configured the file correctly. YES!
- `sudo a2dissite default.conf` to disable ubuntu default page.
- `ls /etc/apache2/sites-enabled` to see all available sites.
- `sudo nano /etc/apache2/sites-available/catalog.conf`. Be sure to paste your own public IP provided by Amazon.
-
```
<VirtualHost *:80>
    ServerName 54.172.170.196
    ServerAdmin admin@mywebsite.com
    WSGIScriptAlias / /var/www/FlaskApps/SimpleApp.wsgi
    <Directory /var/www/FlaskApps/SimpleApp/>
        Order allow,deny
        Allow from all
    </Directory>
    <Directory /var/www/FlaskApps/SimpleApp/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
- Enable virtualhost with the following command `sudo a2ensite catalog`
- `sudo nano /var/www/FlaskApp/FlaskApp.wsgi`. Add the following lines of
```
#! /usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApps/SimpleApp/")

# home points to the home.py file
from home import app as application
application.secret_key = "somesecretsessionkey"
```
- Restart apache to apply changes `sudo service apache2 restart`

Resources: http://amunategui.github.io/idea-to-pitch/

### 9 - Install Github / Clone Item Catalog /
1. `sudo apt-get install git`
2. Configure Github
- `git config --global user.name 'gali7221'`
- `git config --global user.email 'gali7221@gmail.com'`
3. To view information `git config --list`
4. Clone the catalog with `git clone https://github.com/gali7221/musiccatalog.git`
5. Save it into the directory `/var/www/catalog/`


### 10 - Install all Dependencies
1. Enable virtual environment `venv /bin/activate`
2. `sudo pip install httplib2`
3. `sudo pip install requests`
4. `sudo pip install flask-seasurf`
5. `sudo pip install oauth2client`
6. `sudo pip install sqlalchemy`
7. `sudo pip install python-psycopg2`

### 11 - Install PostgreSQL
1. `sudo apt-get install postgresql postgresql-contrib`
2. Now update `engine = create_engine('sqlite://...')` to `engine=create_engine('postgresql://catalog:a_password@localhost/catalog')`
- Change the above line in both app.py & lotsofitems.py
3. Rename app.py to home.py
4. Change to default user postgres
- `sudo su - postgres` & connect to psql with `psql`
5. Create a user with login role and set a password
- `Create User catalog with Password 'a_password';`
- `ALTER user catalog CREATEDB;`
- `Create DATABASE catalog WITH Owner catalog;`
9. Connect to database
- `\c catalog;`
- `REVOKE ALL ON SCHEMA public FROM public;`
- Grant only access to the catalog role `GRANT ALL ON SCHEMA public TO catalog;`
12. Log out of psql and then exit
- Exit postgres & postgres user
- `\q`
- `exit`
13. Setup database
- `sudo nano database_setup.py`
14. Populate the db.
- `sudo python lotsofitems.py`.  


Resources: Thank you to Hari Vedam for explaining the necessary postgresql steps!

### 12 - Deploy Catalog
1. Be sure to make the github repository inaccessible by `cd /var/www/catalog/`
- `sudo nano .htaccess`
- Paste `RedirectMatch 404 /\.git`
2. Create the database schema `sudo python database_setup.py`
3. In home.py update the lines `CLIENT_ID = json.loads(
    open('client_secrets.json', 'r').read())['web']['client_id']`
    to
    `CLIENT_ID = json.loads(
        open('/var/www/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']`

- It is the full path to client_secrets.json.
4. Restart apache `sudo service apache2 restart`
5. You should now see the itemcatalog deployed.
6. If any errors `sudo /var/log/apache2/error.log`

### 13 - Oauth login
1. Use the website http://www.hcidata.info/host2ip.cgi
2. Type in your Public IP address to receive a host name
- `ec2-54-172-170-196.compute-1.amazonaws.com`
3. Update the apache configuration file `sudo nano /etc/apache2/sites-available/catalog.conf` and update the ServerAlias Hostname `ec2-54-172-170-196.compute-1.amazonaws.com`
4. enable virtual host `sudo a2ensite catalog`
5. Add Google Authorization
-  https://console.developers.google.com/project
- Go to credentials and add the hostname to javascript origins!
- Download the new .json file and update the client_secrets.json in the project!
- Peace out project!


Resources https://www.digitalocean.com/community/tutorials/how-to-install-git-on-ubuntu-14-04#how-to-install-git-with-apt
Thank you to Trish and Steve and the Slack Community!
