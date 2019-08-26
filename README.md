# Project: Udacity Full Stack Web Development Nano Degree: Linux Server Configuration

This project uses the Item Catalog project and deploys it to a Ubuntu 16.04 operating system on an Amazon Lightsail EC2 instance configured with an Apache web server.

- Ubuntu 16.04 LTS - Xenial (HVM). Lean, fast and powerful, Ubuntu Server delivers services reliably, predictably and economically. It is the perfect base on which to build your instances. Ubuntu is free and will always be, and you have the option to get support and Landscape from Canonical.

- https://lightsail.aws.amazon.com  
- https://httpd.apache.org/

- https://github.com/patricksts/ps-item-catalog

## 1. Server Access

IP address: 54.161.178.0
SSH port: 2200
URL: http://54.161.178.0/ or http://54.161.178.0.xip.io


## 2. Requirements

- Apache2
- Flask
- git
- httplib2
- libpq-dev
- mod_wsgi
- oauth2client
- pip
- PostgreSQL
- Psycopg2
- Python Requests
- SQLAlchemy
- virtualenv

## 3. Configuration

### Amazon Lightsail

You will need to create an AWS account to prodceed.

1. Sign in and navigate to Amazon Lightsail.

2. Click the button for: 'Create an instance'

3. Select the 'OS Only' and 'Ubuntu 16.04 LTS'

4. Select payment plan

5. Name the instance and click 'Create'

6. The instance will take a moment to start up

### Connect to the instance from your terminal/bash

1. Navigate to to the Amazon Lightsail 'Account page' and download the instance's private key

2. Click on 'Download default key'

3. A file called LightsailDefaultPrivateKey.pem or LightsailDefaultPrivateKey-YOUR-AWS-REGION.pem will be downloaded; open this in a text editor

4. Copy the text and put it in a file called lightsail_key.rsa in the local ~/.ssh/ directory

5. Enter `chmod 600 ~/.ssh/lightrail_key.rsa`

6. SSH into the remote instance usig: `ssh -i ~/.ssh/lightrail_key.rsa ubuntu@YOUR_EC2_PUBLIC_IP_ADDRESS`

### Update/Upgrade packages

1. Enter `sudo apt-get update`

2. Download updates. Enter: `sudo apt-get upgrade`

### Firewall Configuration

1. SSH port should change from `22` to `2200`
    a. `sudo nano /etc/ssh/sshd_config` edit the port number to `2200`
    b. restart SSH by running `sudo service ssh restart`

2. Check firewall status: `sudo ufw status`

3. Run the following commands to configure the firewall:
    a. `sudo ufw default deny incoming`
    b. `sudo ufw default allow outgoing`
    c. `sudo ufw allow ssh`
    d. `sudo ufw allow 2200/tcp` (SSH connectivity)
    e. `sudo ufw allow www` (HTTP server)
    f. `sudo ufw allow 123/udp`
    g. `sudo ufw deny 22` to deny port `22` (deny default port for SSH)
    h. `sudo ufw enable` to enable the ufw firewall
    i. `sudo ufw status` to check configuration.

   See below for results of firewall configuration

   ` To                         Action      From`
    `--                         ------      ----`
    `22                         DENY        Anywhere`
    `2200/tcp                   ALLOW       Anywhere`
    `80/tcp                     ALLOW       Anywhere`
    `123/udp                    ALLOW       Anywhere`
    `22 (v6)                    DENY        Anywhere (v6)`
    `2200/tcp (v6)              ALLOW       Anywhere (v6)`
    `80/tcp (v6)                ALLOW       Anywhere (v6)`
    `123/udp (v6)               ALLOW       Anywhere (v6)`

4. Lightsail configuration must match the internal firewall settings. Clicking on the 'Manage' option, then the 'Networking' tab, and then edit settings.

5. Reconnect on Terminal:
    `ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@YOUR_EC2_PUBLIC_IP_ADDRESS`

### Create 'Grader' user

1. `sudo adduser grader`

2. Enter a new UNIX password when prompted - to keep things simple, my password for this user was 'grader'

3. Complete optional information for the new `grader` user - 'full name' is enough.

4. Switch users with `su - grader`, and enter the password

### Give udo permissions

1. Run `sudo visudo`

2. Find:

    `root    ALL=(ALL:ALL) ALL`

3. Add below:

    `grader   ALL=(ALL:ALL) ALL`

4. Save and close the visudo file `CTRL+O, ENTER, CTRL+X`

5. Verify `grader` permissions:  `su - grader`, enter the password, and run `sudo -l`

    `Matching Defaults entries for grader on`
        `ip-YOUR_EC2_IP_ADDRESS.ec2.internal:`
        `env_reset, mail_badpass,`
        `secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin`

    `User grader may run the following commands on`
        `ip-YOUR_EC2_IP_ADDRESS.ec2.internal:`
        `(ALL : ALL) ALL`

### Generate Grader ssh keys

1. `ssh-keygen` locally

2. Choose a file name for the key pair (I used 'udacity_key')

3. Enter in a passphrase twice (two files will be generated; the second one will end in .pub)

4. Log in to the virtual machine

5. Switch to `grader`'s home directory, and create a new directory:  `mkdir .ssh`

6. `touch .ssh/authorized_keys`

7. `cat ~/.ssh/insert-name-of-file.pub`

8. Copy contents of the file, and paste them in  `.ssh/authorized_keys` file on the VM

9. `chmod 700 .ssh` on the VM

10. `chmod 644 .ssh/authorized_keys` on the VM

11. Open the `/etc/ssh/sshd_config` file, locate: '# Change to no to disable tunnelled clear text passwords'
   Ensure the next line says, 'PasswordAuthentication no'
   If an edit was required: `sudo service ssh restart`

12. Log in as the grader: `ssh -i ~/.ssh/grader_key -p 2200 grader@YOUR_EC2_IP_ADDRESS`
   Enter `grader`'s password.

### Timezone configuration to UTC

1. `sudo dpkg-reconfigure tzdata`  
   follow the instructions (UTC is in 'None of the above' category)

### Apache

1. `sudo apt-get install apache2`

2. Navigate to public IP address of your instance and ensure that the default Apache page is showing

### Mod_wsgi

1. Install the mod_wsgi package: `sudo apt-get install libapache2-mod-wsgi python-dev`

2. Activate: `sudo a2enmod wsgi`

### PostgreSQL

1. `sudo apt-get install postgresql`

2. `sudo nano /etc/postgresql/9.5/main/pg_hba.conf file`

3. Verify contents:

    `local   all             postgres                                peer`
    `local   all             all                                     peer`
    `host    all             all             127.0.0.1/32            md5`
    `host    all             all             ::1/128                 md5`

### Python

1. Verify Python installation: `python`

   `Python 2.7.12 (default, Nov 19 2016, 06:48:10)`
   `[GCC 5.4.0 20160609] on linux2`
   `Type "help", "copyright", "credits" or "license" for more information.`
   `>>>`

### Create a new PostgreSQL user named `catalog`

1. `sudo su - postgres`

2. `psql`

3. Create `catalog`: `CREATE ROLE catalog WITH LOGIN;`

4. `ALTER ROLE catalog CREATEDB;`

5. `\password catalog`

6. Ensure `catalog` user: `\du`:

   `                List of roles                                                       `
   `Role name |                         Attributes                         | Member of `
   `-----------+------------------------------------------------------------+-----------`
   `catalog   | Create DB                                                  | {}        `
  ` postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}        `

7. Exit psql: `\q`

8. Switch back `ubuntu` user: `exit`

### Create user `catalog` and a new PostgreSQL database

1. Create user`catalog`:
     a. `sudo adduser catalog`
     b. - enter in a new password when prompted - to keep things simple, my password for this user was 'catalog'
     c. Complete optional information for the new `grader` user - 'full name' is enough.

2. Give the `catalog` user sudo permissions:
    a. `sudo visudo`
    b. look for this: `root    ALL=(ALL:ALL) ALL`
    c. add the following line below this one: `catalog    ALL=(ALL:ALL) ALL`
    d. Save and close the visudo file `CTRL+O, ENTER, CTRL+X`
    e. Verify `grader` permissions:  `su - grader`, enter the password, and run `sudo -l`

    `User grader may run the following commands on`
        `ip-YOUR_EC2_IP_ADDRESS.ec2.internal:`
        `(ALL : ALL) ALL`

3. Logged in as `catalog`, create a database called catalog: `createdb catalog`

4. `psql`  then  `\l` to view new database

5. Switch back to the `ubuntu` user: `exit`

### Install git and clone project

1. `sudo apt-get install git`

2. `mkdir catalog` in the /var/www/ directory

3. `cd` to 'catalog' directory, and clone the project: `sudo git clone https://github.com/patricksts/psitemcatalog`

4. Change the ownership of the directory to `ubuntu` (while in /var/www): `sudo chown -R ubuntu:ubuntu catalog/`

5. `cd /var/www/nuevoMexico/psitemcatalog`

6. Change the name of the 'app.py' file to 'init__.py': `mv application.py __init__.py`

7. `sudo nano __init__.py`

8. Find and change `app.run(host='0.0.0.0', port=8000)` to: `app.run()`

### Google Authentication

1. Create a new project on the Google API Console

2. Create an OAuth Client ID (under the Credentials tab) and add "http://YOUR_EC2_PUBLIC_IP_ADDRESS" and "http://YOUR_EC2_PUBLIC_IP_ADDRESS.xip.io" as authorized JavaScript origins

3. Add "http://YOUR_EC2_PUBLIC_IP_ADDRESS.xip.io/login", "http://YOUR_EC2_PUBLIC_IP_ADDRESS.xip.io/gconnect", and "http://YOUR_EC2_PUBLIC_IP_ADDRESS.xip.io/oauth2callback" as authorized redirect URIs

4. `sudo touch /var/www/catalog/psitemcatalog/client_secrets.json`

5. Download the JSON file from the Google Oauth Credentials, copy the contents, and paste the contents into the client_secrets.json file you created in the previous step

6. `sudo nano /var/www/catalog/psitemcatalog/templates/login.html`
Add the client ID to lines 4 and 11 of the templates/login.html file in the project directory

7. `sudo nano /var/www/catalog/psitemcatalog/__init__.py`
Add the complete file path for the client_secrets.json file in line 19 of the __init__.py file

### Set up a vitual environment and install dependencies

1. Install pip: `sudo apt-get install python-pip`

2. Install virtualenv: `sudo apt-get install python-virtualenv`

3. In `/var/www/catalog/psitemcatalog/`: choose a name for a temporary environment (I used 'venv') and run: `virtualenv venv`

4. Activate the new environment: `. venv/bin/activate`

5. With the virtual environment active, install the following dependenies:
   a. `pip install httplib2`
   b. `pip install requests`
   c. `pip install --upgrade oauth2client`
   d. `pip install sqlalchemy`
   e. `pip install flask`
   f. `sudo apt-get install libpq-dev`
   g. `pip install psycopg2`

6. Run: `python __init__.py`
   You should see this:
    `* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)`

7. Deactivate the virtual environment:`deactivate`

### Set up and enable a virtual host

1. `sudo touch /etc/apache2/sites-available/catalog.conf`

2. `sudo nano /etc/apache2/sites-available/catalog.conf`
   Copy and paste the following into the .conf file:

    `<VirtualHost *:80>`
         `ServerName XX.XX.XX.XX`
         `ServerAdmin YOUREMAIL@gmail.com`
         `WSGIScriptAlias / /var/www/catalog/catalog.wsgi`
         `<Directory /var/www/catalog/psitemcatalog/>`
              `Order allow,deny`
              `Allow from all`
              `Options -Indexes`
         `</Directory>`
         `Alias /static /var/www/catalog/psitemcatalog/static`
         `<Directory /var/www/catalog/psitemcatalog/static/>`
              `Order allow,deny`
              `Allow from all`
              `Options -Indexes`
         `</Directory>`
         `ErrorLog ${APACHE_LOG_DIR}/error.log`
         `LogLevel warn`
         `CustomLog ${APACHE_LOG_DIR}/access.log combined`
    `</VirtualHost>`

3. `sudo a2ensite nuevoMexico` to enable the virtual host
    The following prompt will be returned:

    `Enabling site nuevoMexico.`
    `To activate the new configuration, you need to run:`
      `service apache2 reload`

4. `sudo service apache2 reload`

### Write a .wsgi file

1. `sudo touch /var/www/catalog/catalog.wsgi`

2. Copy and paste the following:

    `activate_this = '/var/www/catalog/psitemcatalog/venv/bin/activate_this.py'`
    `execfile(activate_this, dict(__file__=activate_this))`

    `#!/usr/bin/python`
    `import sys`
    `import logging`
    `logging.basicConfig(stream=sys.stderr)`
    `sys.path.insert(0,"/var/www/catalog/")`

    `from psitemcatalog import app as application`
    `application.secret_key = 'super_secret_key'`

3. Resart Apache: `sudo service apache2 restart`

### Change database to PostgreSQL

Replace line 24 in ___init___.py, line 56 in database_setup.py, and line 5 in dbseed.py with the following:

    engine = create_engine('postgresql://catalog:INSERT_PASSWORD_FOR_DATABASE_HERE@localhost/catalog')

### Disable the default Apache site

1. Disable the default Apache site: `sudo a2dissite 000-default.conf`

    You should see the following:

    `Site 000-default disabled.`
    `To activate the new configuration, you need to run:`
      `service apache2 reload`

2. Run `sudo service apache2 reload`  

### Change the ownership of the project direcotries

Change the ownership of the project directories and files to the `www-data` user. Apache runs as `www-data` 

1. in `/var/www`:
`sudo chown -R www-data:www-data catalog/`

If editing of these files after this point, use the following command:
     `sudo -u www-data nano INSERT_NAME_OF_FILE`

### Set up the database schema and populate the database

1. In the /var/www/catalog/psitem/ directory, activate the virtualenv: 
`. venv/bin/activate`

2. Run `python populator.py`

3. Deactivate the virtualenv (run `deactivate`)

4. Resart Apache again: `sudo service apache2 restart`

5. Now open up a browser and check to make sure the app is working by going to http://YOUR_EC2_PUBLIC_IP_ADDRESS.xip.io


## 6. Sources

Udacity course: [Configuring Linux Web Servers](https://www.udacity.com/course/configuring-linux-web-servers--ud299)

Udacity course: [Linux Command Line Basics](https://www.udacity.com/course/linux-command-line-basics--ud595)

Digital Ocean tutorial: [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

Digital Ocean tutorial: [How To Use Roles and Manage Grant Permissions in PostgreSQL on a VPS](https://www.digitalocean.com/community/tutorials/how-to-use-roles-and-manage-grant-permissions-in-postgresql-on-a-vps--2)

Digital Ocean tutorial: [How To Secure PostgreSQL on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)

Flask [documentation](http://flask.pocoo.org/docs/0.12/installation/) on virtualenv

tutorialspoint [tutorial](https://www.tutorialspoint.com/postgresql/postgresql_create_database.htm) on creating a database with PostgreSQL

SQLAlchemy [documentation](http://docs.sqlalchemy.org/en/latest/orm/cascades.html) on cascade delete

SQLAlchemy [documentation](http://docs.sqlalchemy.org/en/rel_1_0/orm/basic_relationships.html) on one-to-many relationships

Udacity forum posts
Stack Overflow
