## Project: Linux Server Configuration

This is a project for **Full Stack Web Devloper Nanodegree**.  `README.md` explains how to set up and secure a Linux distribution on a virtual machine, install and configure a web and a database server to host a web application.

The Linux distribution is [Ubuntu](https://ubuntu.com/download/server) 16.04.6 LTS (Xenial Xerus).

The virtual private server is [Amazon Lightsail](https://lightsail.aws.amazon.com).

The web application is my [Item Catalog project](https://github.com/cathiecli/ItemCatalog).

The web server is [Apache2](https://help.ubuntu.com/lts/serverguide/httpd.html).

The database server is [PostgreSQL](https://www.postgresql.org/).

The deployed website is http://3.135.204.225 or http://ec2-3-135-204-225.us-east-2.compute.amazonaws.com/.


**IP Address:** `3.135.204.225`

**URL:** http://ec2-3-135-204-225.us-east-2.compute.amazonaws.com/


#### Step 1: Set up a new Ubuntu Linux server instance on [Amazon Lightsail](https://lightsail.aws.amazon.com)

1) Create an **Amazon Web Services** account (free access for the next 12 months)
2) Create a Lightsail instance
  Instance location: `Zone A (us-east 2a) - Ohio`
  Platform: `Linux/Unix`
  Blueprint: `OS only`  `Ubuntu 16.04 LTS`
  Instance plan: `$3.5/month (First month free)`
  Unique instance name: `[use default or you name one]`
  Click **Create instance** button.  It takes 5-10 minutes for the instance to start up.
  The generated public IP is `3.135.204.225`.
3) Download default private key **SSH Keys** tab of **Account page**
   The downloaded file is at `~/Downloads/LightsailDefaultKey-us-east-2.pem`

   ```
   mv ~/Downloads/LightsailDefaultKey-us-east-2.pem ~/.ssh
   chmod 600 ~/.ssh/LightsailDefaultKey-us-east-2.pem
   ```
   To connect to the instance:

   ```
   ssh -i ~/.ssh/LightsailDefaultKey-us-east-2.pem ubuntu@3.135.204.225
   ```

4) Add two more TCP ports (`123` and `2200`) from **Networking** tab


#### Step 2: Secure the server.

1) Create a new user called `grader`

```
root@ip-[private IP]:~# sudo adduser grader
```

Create a new file `grader` under `/etc/sudoers.d`

```
ubuntu@ip-[private IP]:~$ sudo vi /etc/sudoers.d/grader
ubuntu@ip-[private IP]:~$ sudo cat /etc/sudoers.d/grader
grader ALL=(ALL:ALL) ALL
```

Add `127.0.1.1 ip-10-20-37-65` into `/etc/hosts` file

```ubuntu@ip-[private IP]:~$ sudo vi /etc/hosts***
ubuntu@ip-[private IP]:~$ sudo cat  /etc/hosts
127.0.0.1 localhost
127.0.1.1 ip-10-20-37-65
```

2) Update and upgrade all installed packages:

```
ubuntu@ip-[private IP]:~$ sudo apt-get update
ubuntu@ip-[private IP]:~$ sudo apt-get update
ubuntu@ip-[private IP]:~$ sudo apt-get install finger
```

To test, use `finger grader`

3) Set up Key-based Authentication

Open a new local terminal window, generate the key at ~/.ssh and name it
`udacity_key.rsa`

```
ssh-keygen -f ~/.ssh/udacity_key.rsa
```

Copy the generated public key by viewing it via `cat ~/.ssh/udacity_key.rsa.pub`
to the Amazon Lightsail server terminal

Go back to the virtual terminal window with the prompt of
`ubuntu@ip-[private IP]`, switch to `root` user

```
ubuntu@ip-[private IP]:~$ sudo su -
root@ip-[private IP]:/home/grader# whoami
root
```

Now go to `/home/grader`, create a directory called `.ssh`, then create a new file
called `authorized_keys` and paste the public key content in it.

```
root@ip-[private IP]:~# cd /home/grader
root@ip-[private IP]:/home/grader# mkdir .ssh
root@ip-[private IP]:/home/grader# touch .ssh/authorized_keys
root@ip-[private IP]:/home/grader# vi .ssh/authorized_keys
root@ip-[private IP]:/home/grader# cat .ssh/authorized_keys
……
```

Change file permission and ownership

```
root@ip-[private IP]:/home/grader# chmod 700 /home/grader/.ssh
root@ip-[private IP]:/home/grader# chmod 644 /home/grader/.ssh/authorized_keys
root@ip-[private IP]:/home/grader# chown -R grader:grader /home/grader/.ssh
root@ip-[private IP]:/home/grader# service ssh restart
root@ip-[private IP]:/home/grader# Connection to 3.135.204.225 closed.
```

Configure the setting for user `grader`
Log in to the virtual Amazon Lightsail server as `grader`

```
cathi@[My Local] ~/.ssh
$ ssh -i ~/.ssh/udacity_key.rsa grader@3.135.204.225
```

Make changes to SSH configuration at `/etc/ssh/sshd_config`

```
grader@ip-[private IP]:~$ sudo vi /etc/ssh/sshd_config
[sudo] password for grader:
```

Set `PasswordAuthentication` to `no`

Set `PermitRootLogin` to `no`

Change SSH login port from `22` to `2200`

Restart service and login in with the new port

```
grader@ip-[private IP]:~$ sudo service ssh restart
grader@ip-[private IP]:~$ exit
cathi@[My Local] ~
$ ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@3.135.204.225
```

4) Configure UFW

```
grader@ip-[private IP]:~$ sudo ufw allow 2200/tcp
[sudo] password for grader:
Rules updated
Rules updated (v6)
grader@ip-[private IP]:~$ sudo ufw allow 80/tcp
Rules updated
Rules updated (v6)
grader@ip-[private IP]:~$ sudo ufw allow 123/udp
Rules updated
Rules updated (v6)
grader@ip-[private IP]:~$ sudo ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
```


#### Step 3: Deploy Catalog Application

The virtual web server is **apache2**, do SSH to Amazon Lightsail by logging in as `grader`

1) Install required package: `apache2`, `libapache2-mod-wsgi` and `git`

```
grader@ip-[private IP]:~$ sudo apt-get install apache2
grader@ip-[private IP]:~$ sudo apt-get install libapache2-mod-wsgi python-dev
grader@ip-[private IP]:~$ sudo apt-get install git
```

Restart Apache server for `mod_wsgi` to load

```
grader@ip-[private IP]:~$ sudo service apache2 restart
```

Configure `git` (optional)

```
grader@ip-[private IP]:~$ git config --global user.name "[firstname lastname]"
grader@ip-[private IP]:~$ git config --global user.email "[email]"
```


2) Enable `mod_wsgi`

```
grader@ip-[private IP]:~$ sudo a2enmod wsgi
Module wsgi already enabled
```

Go to http://3.135.204.225, the Apache Ubuntu Default Page should be shown and
display **It works!**

3) Set up my web application directory and change its ownership to `grader`

```
grader@ip-[private IP]:~$ cd /var/www
grader@ip-[private IP]:/var/www$ sudo mkdir catalog
grader@ip-[private IP]:/var/www$ sudo chown -R grader:grader catalog
grader@ip-[private IP]:/var/www$ cd catalog
grader@ip-[private IP]:/var/www/catalog$
```

4) Clone my project from Github

```
grader@ip-[private IP]:/var/www/catalog$ git clone https://github.com/cathiecli/ItemCatalog catalog
```

5) Create a `.wsgi` file

```
grader@ip-[private IP]:~$ pwd
/var/www/catalog/
grader@ip-[private IP]:~$ sudo vi catalog.wsgi
grader@ip-[private IP]:~$ cat catalog.wsgi*
```

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/catalog/")
sys.path.insert(1, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'supersecretkey'
```

6) Rename `application.py` to `__init__.py`

```
grader@ip-[private IP]:/var/www/catalog/catalog$ pwd
/var/www/catalog/catalog
grader@ip-[private IP]:/var/www/catalog/catalog$ mv application.py __init__.py
```

7) Install and activate the virtual machine `(venv)`

```
grader@ip-[private IP]:/var/www/catalog/catalog$ sudo apt-get install python-pip
grader@ip-[private IP]:/var/www/catalog/catalog$ sudo pip install virtualenv
......
Successfully installed virtualenv-16.7.9
......
grader@ip-[private IP]:/var/www/catalog/catalog$ sudo virtualenv venv
New python executable in /var/www/catalog/catalog/venv/bin/python
Installing setuptools, pip, wheel...
done.
grader@ip-[private IP]:/var/www/catalog/catalog$ source venv/bin/activate
(venv) grader@ip-[private IP]:/var/www/catalog/catalog$
```

After activating, `(venv)` should be shown at the beginning of the prompt.  
Change venv permission.

```
(venv) grader@ip-[private IP]:/var/www/catalog/catalog$ sudo chmod -R 777 venv
```

8) Install Flask and other packages:

```
(venv) grader@ip-[private IP]:/var/www/catalog/catalog$ sudo pip install Flask
(venv) grader@ip-[private IP]:/var/www/catalog/catalog$ sudo pip install httplib2
(venv) grader@ip-[private IP]:/var/www/catalog/catalog$ sudo pip install oauth2client
(venv) grader@ip-[private IP]:/var/www/catalog/catalog$ sudo pip install sqlalchemy
(venv) grader@ip-[private IP]:/var/www/catalog/catalog$ sudo pip install Flask-SQLAlchemy
(venv) grader@ip-[private IP]:/var/www/catalog/catalog$ sudo apt-get install build-dep python-psycopg2
(venv) grader@ip-[private IP]:/var/www/catalog/catalog$ sudo apt-get install libpq-dev
(venv) grader@ip-[private IP]:/var/www/catalog/catalog$ sudo pip install psycopg2
(venv) grader@ip-[private IP]:/var/www/catalog/catalog$ sudo pip install sqlalchemy_utils
(venv) grader@ip-[private IP]:/var/www/catalog/catalog$ sudo pip install requests
(venv) grader@ip-[private IP]:/var/www/catalog/catalog$ sudo pip install render_template
(venv) grader@ip-[private IP]:/var/www/catalog/catalog$ sudo pip install redirect
```

9) Change the path of JSON security file in `__init__.py` file  

> Old:

```
CLIENT_ID = json.loads(
  open('client_secrets.json', 'r').read())['web']['client_id']
```

> New:

```
CLIENT_ID = json.loads(
  open('/var/www/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']
```

> Old:

```
oauth_flow = flow_from_clientsecrets('client_secrets.json', scope='')
```

> New:

```
oauth_flow = flow_from_clientsecrets('/var/www/catalog/catalog/client_secrets.json', scope='')
```

10) Change hostname and port in `__init__.py` file

> Old

```
if __name__ == '__main__':
  app.secret_ley = 'super_secret_key'
  app.debug = True
  app.run(host='0.0.0.0', port=8000)
```

> New

```
if __name__ == '__main__':
  app.secret_ley = 'super_secret_key'
  app.debug = True
  app.run(host='3.135.204.225', port=80)
```

11) Configure virtual hostname in `/etc/apache2/sites-available/catalog.conf`

You can find your hostname at [Convert Host/Domain Name to IP Address and vice versa](http://www.hcidata.info/host2ip.cgi).

```
(venv) grader@ip-[private IP]:/var/www/catalog/catalog$ sudo vi /etc/apache2/sites-available/catalog.conf
(venv) grader@ip-[private IP]:/var/www/catalog/catalog$ sudo cat /etc/apache2/sites-available/catalog.conf

<VirtualHost *:80>
  ServerName 3.135.204.225
  ServerAlias ec2-3-135-204-225.us-east-2.compute.amazonaws.com
  ServerAdmin admin@3.135.204.225
  WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
  WSGIProcessGroup catalog
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

Enable virtual host

```
(venv) grader@ip-[private IP]:/var/www/catalog/catalog$ sudo a2ensite catalog
Enabling site catalog.
To activate the new configuration, you need to run:
service apache2 reload
(venv) grader@ip-[private IP]:/var/www/catalog/catalog$ sudo service apache2 reload
```

12) Set up database

```
(venv) grader@ip-[private IP]:/var/www/catalog/catalog$ sudo apt-get install libpq-dev python-dev
(venv) grader@ip-[private IP]:/var/www/catalog/catalog$ sudo apt-get install postgresql postgresql-contrib
```

```
(venv) grader@ip-[private IP]:/var/www/catalog/catalog$ sudo su - postgres
postgres@ip-[private IP]:~$ psql
psql (9.5.19)
Type "help" for help.

postgres=#
```

13) Create database called `catalog` and database user called `catalog`

```
postgres=# CREATE USER catalog WITH PASSWORD 'welcome123';
CREATE ROLE
postgres=# ALTER USER catalog CREATEDB;
ALTER ROLE
postgres=# CREATE DATABASE catalog WITH OWNER catalog;
CREATE DATABASE
```

Connect to database

```
postgres=# \c catalog
You are now connected to database "catalog" as user "postgres".
catalog=#
catalog=# REVOKE ALL ON SCHEMA public FROM public;
REVOKE
catalog=# GRANT ALL ON SCHEMA public TO catalog;
GRANT
catalog-# \q                            -- this is quit postgres command line
postgres@ip-[private IP]:~$
postgres@ip-[private IP]:~$ exit        -- log out database
logout
(venv) grader@ip-[private IP]:/var/www/catalog/catalog$
```

14) Change database engine in the following files:
 `__init__.py`, `ic_database_setup.py`, `ic_database_insert.py`, `ic_database_delete.py` and `ic_database_select.py`

> Old

```
engine = create_engine('sqlite:///itemcategory.db')
```

> New

```
engine = create_engine('postgresql://catalog:welcome123@localhost/catalog')
```

  15) Run my database setup script

```
(venv) grader@ip-[private IP]:/var/www/catalog/catalog$ python ic_database_setup.py
Completed database setup
(venv) grader@ip-[private IP]:/var/www/catalog/catalog$ python ic_database_insert.py
Completed database setup
-- user data inserted
-- category data inserted
-- item data inserted
(venv) grader@ip-[private IP]:/var/www/catalog/catalog$ python ic_database_select.py
Completed database setup
------ showing user table:
User #1: Cathie email:cathiecli@gmail.com
User #2: John email:cftpgroup16@gmail.com
……
```

  16) Add `ec2-3-135-204-225.us-east-2.compute.amazonaws.com` as "Authorized JavaScript origins" and `http://ec2-3-135-204-225.us-east-2.compute.amazonaws.com/oauth2callback` as "Authorized redirect URIs"
https://console.developers.google.com/apis/credentials/oauthclient/289780704941-qa0je5g1ptfu16dt8n38koffr7k50fr4.apps.googleusercontent.com?folder=&organizationId=&project=category-item-app-264602

Re-generate JSON client secrets file, and update the content of `/var/www/catalog/catalog/client_secrets.json`

```
(venv) grader@ip-[private IP]:/var/www/catalog/catalog$ sudo service apache2 restart
```

  17) Disable the default Apache site

```
(venv) grader@ip-[private IP]:/var/www/catalog/catalog$ sudo a2dissite 000-default.conf
Site 000-default disabled.
To activate the new configuration, you need to run:
service apache2 reload

(venv) grader@ip-[private IP]:/var/www/catalog/catalog$ sudo service apache2 reload
(venv) grader@ip-[private IP]:/var/www/catalog/catalog$ sudo service apache2 restart
```

#### Useful commands

1) Use below two lines to see the error in `apache2.conf` file

```
(venv) grader@ip-[private IP]:/var/www/catalog/catalog$ cd /etc/apache2
(venv) grader@ip-[private IP]:/etc/apache2$ apache2ctl configtest
```

2) To view log messages from Apache server:

```
sudo tail /var/log/apache2/error.log.
```

### Authors

 Cong Li

### Version History

0.1 - Initial Release on 01-20-2020

### License

This project is licensed under the cl9451

### Acknowledgments

[boisalai/udacity-linux-server-configuration](https://github.com/boisalai/udacity-linux-server-configuration)

[chuanqin3/udacity-linux-configuration](https://github.com/chuanqin3/udacity-linux-configuration)

### Inspiration, code snippets, etc.

[Google Developer APIs & Services](https://console.developers.google.com/apis)

[facebook for developers](https://developers.facebook.com/apps)

[GitHub Basic writing and formatting syntax](https://help.github.com/en/github/writing-on-github/basic-writing-and-formatting-syntax)

[Dillinger](https://dillinger.io/)
