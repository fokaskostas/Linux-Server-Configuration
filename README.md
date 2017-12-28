# Linux Server Configuration

## Project Description:
This is the final project for Udacity's Full Stack Web Developer Nanodegree. The project entails taking a baseline installation of a Linux server and prepare it to host a web application. To access the website visit http://18.195.132.155/ 
In order to complete the project the following steps were taken:

### Step 1: Start a new Ubuntu Linux server instance on Amazon Lightsail
* Login into Amazon Lightsail using an Amazon Web Services account.
* Once you login into the site, click Create instance.
* Choose Linux/Unix platform, OS Only and Ubuntu 16.04 LTS.
* Choose an instance plan.
* Keep the default name provided by AWS or rename your instance.
* Click the Create button to create the instance.
* Wait for the instance to start up.

### Step 2: SSH into the server
* From the Account menu on Amazon Lightsail, click on SSH keys tab and download the Default       Private Key.
* Move this private key file named LightsailDefaultPrivateKey-*.pem into the local folder ~/.ssh  and rename it lightsail_key.rsa.
* In your terminal, type: `chmod 600 ~/.ssh/lightsail_key.rsa`.
* To connect to the instance type: `ssh -i ~/.ssh/lightsail_key.rsa ubuntu@18.195.132.155`, in your terminal, where 18.195.132.155 is the public IP address of the instance.

### Step 3: Update and upgrade 
* In your terminal, type: `sudo apt-get update`, `sudo apt-get upgrade`

### Step 4: Change the SSH port from 22 to 2200
* Edit the /etc/ssh/sshd_config file: `sudo nano /etc/ssh/sshd_config`
* Change the port number on line 5 from 22 to 2200.
* Save and exit using CTRL+X and confirm with Y.
* Restart SSH: `sudo service ssh restart`

### Step 5: Configure the Uncomplicated Firewall (UFW)
* Configure the default firewall for Ubuntu to only allow incoming connections for SSH (port       2200), HTTP (port 80), and NTP (port 123).

  `sudo ufw status` # The UFW should be inactive.
  `sudo ufw default deny incoming` # Deny any incoming traffic.
  `sudo ufw default allow outgoing` # Enable outgoing traffic.
  `sudo ufw allow 2200/tcp` # Allow incoming tcp packets on port 2200.
  `sudo ufw allow www` # Allow HTTP traffic in.
  `sudo ufw allow 123/udp` # Allow incoming udp packets on port 123.
  `sudo ufw deny 22` # Deny tcp and udp packets on port 53.
  
* Turn UFW on: `sudo ufw enable`. The output should be like this:
`Command may disrupt existing ssh connections. Proceed with operation (y|n)? y`
`Firewall is active and enabled on system startup`

* Exit the SSH connection: `exit`.

* Click on the Manage option of the Amazon Lightsail Instance, then the Networking tab, and then change the firewall configuration to match the internal firewall settings above. 

* Allow ports 80 (TCP), 123 (UDP), and 2200 (TCP), and deny the default port 22.

* From your local terminal, run: `ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@18.195.132.155` where 18.195.132.155 is the public IP address of the instance.

### Step 6: Create user grader
* While logged in as ubuntu, add user: `sudo adduser grader`.
* Enter a password (twice) and fill out information for this new user.

### Step 7: Give grader the permission to sudo
* Edit the sudoers file: `sudo visudo`.
* Find the line `root ALL=(ALL:ALL) ALL` and add below: `grader ALL=(ALL:ALL) ALL`
* Save and exit using CTRL+X and confirm with Y.

### Step 8: Create an SSH key pair for grader using the ssh-keygen tool
* On the local machine:
    1. Run ssh-keygen
    2. Enter file in which to save the key (e.g. grader_key) in the local directory ~/.ssh
    3. Enter in a passphrase twice. Two files will be generated ( ~/.ssh/grader_key and ~/.ssh/grader_key.pub)
    4. Run `cat ~/.ssh/grader_key.pub` and copy the contents of the file
    5. Log in to the grader's virtual machine
  
 * On the grader's virtual machine:
     1. Create a new directory called ~/.ssh: `mkdir .ssh`
     2. Run `sudo nano ~/.ssh/authorized_keys` and paste the content into this file, save and exit
     3. Give the permissions: `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`
     4. Check in `/etc/ssh/sshd_config` file if PasswordAuthentication is set to no
     5. Restart SSH: `sudo service ssh restart`
 
* On the local machine, run: `ssh -i ~/.ssh/grader_key -p 2200 grader@18.195.132.155`.

### Step 9: Configure the local timezone to UTC
* While logged in as grader, configure the time zone: `sudo dpkg-reconfigure tzdata`.
* Choose UTC

### Step 10: Install and configure Apache to serve a Python mod_wsgi application
* While logged in as grader, install Apache: `sudo apt-get install apache2`.
* Enter public IP of the Amazon Lightsail instance into browser. If Apache is working, you should see the Apache2 Ubuntu Default Page.
* Project is build using Python3 so  install the Python 3 mod_wsgi package: `sudo apt-get install libapache2-mod-wsgi-py3`.
* Enable mod_wsgi using: `sudo a2enmod wsgi`.

### Step 11: Install and configure PostgreSQL
* While logged in as grader, install PostgreSQL: `sudo apt-get install postgresql`.
* Switch to the postgres user: `sudo su - postgres`.
* Open PostgreSQL interactive terminal with `psql`.
* Create the catalog user with a password and give them the ability to create databases:
`postgres=# CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';`
`postgres=# ALTER ROLE catalog CREATEDB;`
* Exit psql: `\q`.
* Switch back to the grader user:`exit`.
* Create a new Linux user called catalog: `sudo adduser catalog`. Enter password and fill out information.
* Give to catalog user the permission to sudo:`sudo visudo`.
* Below the lines:  
`root    ALL=(ALL:ALL) ALL`
`grader  ALL=(ALL:ALL) ALL` 
add a new line to give sudo privileges to catalog user:
`catalog  ALL=(ALL:ALL) ALL`
* Save and exit using CTRL+X and confirm with Y.
* While logged in as catalog, create a database: `createdb catalog`.
* Exit psql: `\q`.
* Switch back to the grader user: `exit`.

### Step 12: Install git
* While logged in as grader, install git: `sudo apt-get install git`.

### Step 13: Clone and setup the Item Catalog project from the GitHub repository
* While logged in as grader, create /var/www/catalog/ directory.
* Change to that directory and clone the catalog project:
`sudo git clone https://github.com/fokaskostas/Catalog-app.git catalog`.
* From the /var/www directory, change the ownership of the catalog directory to grader using: `sudo chown -R grader:grader catalog/`.
* Change directory to the /var/www/catalog/catalog directory.
* Rename the `project.py` file to `__init__.py` using: `mv application.py __init__.py`.
* In `__init__.py`, replace lines 337, 338 with: `app.run()`
* In `__init__.py`, replace the import statement in line 6 with: 
`from catalog.database_setup import Base, Category, Item, User`
* In `database_setup.py` replace line 58 with: 
`engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`
* In `database_init.py` replace line 7 with:
`engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`

### Step 14: Authenticate login through Google
* Go to [Google Cloud Plateform](https://console.cloud.google.com/).
* Click APIs & services on left menu.
* Click Credentials.
* Create an OAuth Client ID (under the Credentials tab), and add http://18.195.132.155/ and http://ec2-18-195-132-155.eu-central-1.compute.amazonaws.com as authorized JavaScript origins.
* Add http://ec2-18-195-132-155.eu-central-1.compute.amazonaws.com/gconnect and http://ec2-18-195-132-155.eu-central-1.compute.amazonaws.com/login as Authorized redirect URIs
* Download the corresponding JSON file, open it et copy the content.
* Open /var/www/catalog/catalog/client_secret.json and paste the content into the this file.
* Replace the client ID in line 28 of the templates/login.html file in the project directory.

### Step 15:  Install the virtual environment and dependencies
* While logged in as grader, install pip: `sudo apt-get install python3-pip`.
* Install the virtual environment: `sudo apt-get install python-virtualenv`.
* Change to the /var/www/catalog/catalog/ directory.
* Create the virtual environment: `sudo virtualenv -p python3 venv3`.
* Change the ownership to grader with: `sudo chown -R grader:grader venv3/`.
* Activate the new environment: `. venv3/bin/activate`.
* Install the project's dependencies:
`pip install httplib2`
`pip install requests`
`pip install --upgrade oauth2client`
`pip install sqlalchemy`
`pip install flask`
`sudo apt-get install libpq-dev`
`pip install psycopg2`
* Run `python3 __init__.py` and you should see: 
`Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)`
* Deactivate the virtual environment: `deactivate`.

### Step 16: Set up and enable a virtual host
* Add the following line in /etc/apache2/mods-enabled/wsgi.conf file to use Python 3:
`#WSGIPythonPath directory|directory-1:directory-2:...`
`WSGIPythonPath /var/www/catalog/catalog/venv3/lib/python3.5/site-packages`
* Create /etc/apache2/sites-available/catalog.conf and add the following lines to configure the virtual host:
```
<VirtualHost *:80>
                ServerName mywebsite.com
                ServerAdmin admin@mywebsite.com
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
</VirtualHost>`
```
* Enable virtual host: `sudo a2ensite catalog`.
* Reload Apache: `sudo service apache2 reload`.

### Step 17: Set up the Flask application
* Create /var/www/catalog/catalog.wsgi file add the following lines:
```
#!/usr/bin/python

activate_this = '/var/www/catalog/catalog/venv3/bin/activate_this.py'
with open(activate_this) as file_:
    exec(file_.read(), dict(__file__=activate_this))

import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = "super_secret_key"
```
* Restart Apache: `sudo service apache2 restart`.

### Step 18: Set up the database schema and populate the database
* From the /var/www/catalog/catalog/ directory, activate the virtual environment: 
`. venv3/bin/activate`.
* Run: `python database_init.py`.
* Deactivate the virtual environment: `deactivate`.

### Step 19: Disable the default Apache site
* Run `sudo a2dissite 000-default.conf`.
* Reload Apache: `sudo service apache2 reload`.

### Step 20:  Launch the Web Application
* Change the ownership of the project directories: `sudo chown -R www-data:www-data catalog/`.
* Reload Apache: `sudo service apache2 restart`.
* Open your browser and visit http://18.195.132.155/

### References:
* https://help.ubuntu.com/community/UFW
* https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
* https://www.digitalocean.com/community/tutorials/how-to-set-up-an-apache-mysql-and-python-lamp-server-without-frameworks-on-ubuntu-14-04
* https://github.com/boisalai/udacity-linux-server-configuration