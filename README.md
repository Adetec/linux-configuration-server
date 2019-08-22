# Linux Server Configuration

### About the project
Baseline installation of a Linux distribution on a virtual machine and configuration to host web applications, including installation packages updates, securing it from a number of attack vectors and installing/configuring web and database servers

* IP Address: [185.247.117.194](http://185.247.117.194/)
* SSH Port: 2200
* User: grader


### Server configuration steps

#### 1. Update packages
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade
```
##### Enable automatic security updates

```
sudo apt-get install unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

#### 2. Change timezone to UTC and Fix language issues 
```
sudo timedatectl set-timezone UTC
sudo update-locale LANG=en_US.utf8 LANGUAGE=en_US.utf8 LC_ALL=en_US.utf8
```

#### 3. Create a new user grader and Give him `sudo` access
```
sudo adduser grader
sudo nano /etc/sudoers.d/grader 
```
Add the following text `grader ALL=(ALL) ALL`

#### 4. Setup SSH keys for grader
* On local machine 
`ssh-keygen`
Choose the path for storing public and private keys
* On remote machine home as user grader
```
sudo su - grader
mkdir .ssh
touch .ssh/authorized_keys 
sudo chmod 700 .ssh
sudo chmod 600 .ssh/authorized_keys 
nano .ssh/authorized_keys 
```
Paste the contents of the public key created inside that file

#### 5. Change the SSH port from 22 to 2200, Enforce key-based authentication and Disable Login for root user
```
sudo nano /etc/ssh/sshd_config
```
Change the following:
* Find the `Port` line and edit it to `2200`.
* Find the `PasswordAuthentication` line and edit it to `no`.
* Find the `PermitRootLogi`n line and edit it to `no`.
* Save the file and run `sudo service ssh restart`

#### 6. Configure and enable the Uncomplicated Firewall (UFW)
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow ntp
sudo ufw enable
```

#### 7. Install Apache2, mod-wsgi and Git
```
sudo apt-get install apache2 libapache2-mod-wsgi-py3 git
```

#### 8. Install and configure PostgreSQL
```
sudo apt-get install libpq-dev python3-dev
sudo apt-get install postgresql postgresql-contrib
sudo su - postgres
psql
```
Then
```
CREATE USER music WITH PASSWORD 'password';
CREATE DATABASE music WITH OWNER music;
\c music
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO music;
\q
exit
```
In music project you should change database engine to
```
engine = create_engine('postgresql://music:password@localhost/music')
```

#### 9. Clone the music catalog app from GitHub and Configure it
```
cd /var/www/
sudo mkdir music
sudo chown grader:grader music
sudo git clone https://github.com/Adetec/best-instrumental-music.git music
cd music
nano music.wsgi
```
Add the following in `music.wsgi` file
```python
#!/usr/bin/python3
import sys
sys.stdout = sys.stderr

sys.path.insert(0,"/var/www/music")

from app import app as application
```
Then Install all required dependecies the project needs to run with

#### 10. Configure apache server
Create an other apache site configuration
```
sudo nano /etc/apache2/sites-enabled/udacity-music-project.conf
```
Add the following content:
```
# serve catalog app
<VirtualHost *:80>
  ServerName http://185.247.117.194/
  ServerAdmin adetechtv@gmail.com
  DocumentRoot /var/www/music  
  WSGIDaemonProcess music user=grader group=grader
  WSGIScriptAlias / /var/www/music/music.wsgi

  <Directory /var/www/music>
    WSGIProcessGroup music  
    WSGIApplicationGroup %{GLOBAL}
    Require all granted
  </Directory>

  ErrorLog ${APACHE_LOG_DIR}/error.log
  LogLevel warn
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Change default apache hosted site to udacity-music-project.conf
```
sudo a2dissite 000-default.conf
sudo a2ensite udacity-music-project.conf
service apache2 reload
service apache2 restart

```


#### 11. Reload and Restart Apache Server
```
sudo service apache2 reload
sudo service apache2 restart
```


### Resources
* [VPS Server](https://vpsserver.com)
* [Flask mod_wsgi (Apache)](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/)
* [Apache Server Configuration Files](https://httpd.apache.org/docs/current/configuring.html)
* [Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
* [Set Up Apache Virtual Hosts on Ubuntu ](https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-14-04-lts)
* [mod_wsgi documentation](https://modwsgi.readthedocs.io/en/develop/)
* [Automatic Security Updates](https://help.ubuntu.com/community/AutomaticSecurityUpdates#Using_the_.22unattended-upgrades.22_package)
* [Ask Ubuntu](https://askubuntu.com/)
* [Stack Overflow](https://stackoverflow.com/)
