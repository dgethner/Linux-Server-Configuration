# Linux Server Configuration

Below you will find the method I used to set up an Ubuntu Linux Server on Lightsail.

The server was built to host a Flask application from the Item Catalog project.

##Project Overview
You will take a baseline installation of a Linux server and prepare it to host
your web applications. You will secure your server from a number of
attack vectors, install and configure a database server, and deploy one of your
existing web applications onto it.

##Why this Project?
A deep understanding of exactly what your web applications are doing, how they
are hosted, and the interactions between multiple systems are what define you as
a Full Stack Web Developer. In this project, you’ll be responsible for turning a
brand-new, bare bones, Linux server into the secure and efficient web application
host your applications need.

##What will I Learn?
You will learn how to access, secure, and perform the initial configuration of a
bare-bones Linux server. You will then learn how to install and configure a web
and database server and actually host a web application.


##My Server's Details

1. IP Address: 18.218.147.158

2. SSH port: `2200`.

3. Links to the page: http://18.218.147.158/ or	http://ec2-18-218-147-158.us-east-2.compute.amazonaws.com

#Software/Programs
* Ubuntu 16.04 LTS
* VPS Amazon Lighsail
* Web Application Item Catalog project
* Database PostgreSQL

#Modules
* Apache2
* mod_wsgi
* PostgreSQL
* git
* pip
* virtualenv
* httplib2
* Python Requests
* oauth2client
* SQLAlchemy
* Flask
* libpq-dev
* Psycopg2


##Configuration steps
### Get started on Lightsail
1. Create or Sign in to [Amazon Lightsail](https://amazonlightsail.com) using an Amazon Web Services account

2. Click on the 'Create instance' link

3. Choose the 'OS Only' and 'Ubuntu 16.04 LTS' options

4. Choose a payment plan (the lowest priced plan is fine)

5. Name your instance and click 'Create'

6. The instance will take a few minutes to start


### Connect to the instance on a local machine

1. Download the instance's private key click on 'Account' in the top bar

2. Click on 'SSH keys' then 'Download'

3. A file will be downloaded named LightsailDefaultPrivateKey.pem or
LightsailDefaultPrivateKey-YOUR-AWS-REGION.pem will be downloaded

4. Open this file in a text editor (I used Atom)

5. Copy the text and put it in a file called lightrail_key.rsa

6. Change the file permissions by running `chmod 600 ~/.ssh/lightrail_key.rsa`

7. Connect with the server by inputting `ssh -i ~/.ssh/lightrail_key.rsa ubuntu@XX.XX.XX.XX`,
where XX.XX.XX.XX needs to be the public IP address of the instance


### Updating Your Server
1. Find package updates by running `sudo apt-get update`

2. Download available package updates by running `sudo apt-get upgrade`


### Configure the firewall

Warning: When changing the SSH port, make sure that the firewall is open for
port 2200 first, so that you don't lock yourself out of the server. When you
change the SSH port, the Lightsail instance will no longer be accessible through
the web app 'Connect using SSH' button. The button assumes the default port is
being used.

1. Change the SSH port from `22` to `2200`
     * Open up the /etc/ssh/sshd_config file
		 * Change the port number on line 5 to `2200`
		 * Restart SSH by running `sudo service ssh restart`
		 * Do not miss restarting the SSH service

2. Check the status of ufw (Ubuntu's preinstalled firewall) is active
     * Run `sudo ufw status`

3. Set the ufw firewall to block everything coming in
     * Run `sudo ufw default deny incoming`

4. Set the ufw firewall to allow everything outgoing
     * Run `sudo ufw default allow outgoing`

5. Set the ufw firewall to allow SSH
     * Run `sudo ufw allow ssh`

6. Allow all tcp connections for port `2200` so allow SSH will work
     * Run `sudo ufw allow 2200/tcp`

7. Set the ufw firewall to allow a basic HTTP server
    * Run `sudo ufw allow www`

8. set the ufw firewall to allow NTP
    * Run `sudo ufw allow 123/udp`

9. Deny port `22` (we are denying this port since it is not being used. We have reconfigured the VM to work with port `2200` instead of its default port for SSH)
   * Run `sudo ufw deny 22`

10. Enable the ufw firewall
   * Run `sudo ufw enable`

11. Check which ports are open and if the ufw is active(mine looks like below)
   * Run `sudo ufw status`

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

12. Update the firewall in Amazon Lightsail
    * Go to your browser
		* Be logged-in to Lightsail
		* Click on the 3 dots on your instance and select 'Manage'
		* Click on the 'Networking' tab
		* Change the firewall configuration to match the internal firewall settings
		  above (only ports `80`(TCP), `123`(UDP), and `2200`(TCP) should be allowed
			 * make sure to deny the default port `22`)

13. Login by opening up the Terminal:

	  * `ssh -i ~/.ssh/lightrail_key.rsa -p 2200 ubuntu@XX.XX.XX.XX`,
	     change XX.XX.XX.XX to your instance's public IP address

Note: As mentioned above, connecting to the instance through a browser now no longer works; this is because Lightsail's browser-based SSH access only works through port `22`, which is now denied.


### Create a new user named `grader`
1. Run `sudo adduser grader`

2. Enter in a new UNIX password (twice) when prompted

3. Fill out information for the new `grader` user

4. To switch to the `grader` user, run `su - grader`, and enter the password


### Give `grader` user sudo permissions
1. Run `sudo visudo`

2. Find a line similar this:

	`root    ALL=(ALL:ALL) ALL`

3. Add the following line below it:

	`grader	   ALL=(ALL:ALL) ALL`

4. Save and close the visudo file

5. Verify `grader` has sudo permissions
   * Run `su - grader`
	 * Enter the password
	 * Run `sudo -l`
	 * Enter the password again
	 * The following output should appear, showing `grader` has sudo permissions:

	```
	Matching Defaults entries for grader on
	    ip-XX-XX-XX-XX.ec2.internal:
	    env_reset, mail_badpass,
	    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

	User grader may run the following commands on
		ip-XX-XX-XX-XX.ec2.internal:
	    (ALL : ALL) ALL
	```


### Give `grader` access to log in to the virtual machine
1. On your local machine
   * Run `ssh-keygen`

2. Choose a file name for the key pair (example: grader_key)

3. Enter in a passphrase twice
   * Two files will be generated
	 * One will end in .pub

4. Login to the virtual machine

5. Switch to `grader`'s home directory

6. Create a new directory called `.ssh`
   * Run `mkdir .ssh`
   * Run `touch .ssh/authorized_keys`

7. On the local machine
   * Run `cat ~/.ssh/insert-name-of-file.pub`

8. Copy the contents of the file

9. Go onto the VM
   * Paste them in the .ssh/authorized_keys

10. On the virtual machine
    * Run `chmod 700 .ssh`
    * Run `chmod 644 .ssh/authorized_keys`

11. Make sure key-based authentication is enforced
    * Login as `grader`
		* Open the `/etc/ssh/sshd_config` file
		* Check line '# Change to no to disable tunnelled clear text passwords'
		* If the next line says, 'PasswordAuthentication yes'
		* Change the 'yes' to 'no'
		* Save and exit the file
		* Run `sudo service ssh restart` (make sure to run this command after)

12. Login as the `grader` using the following command:

	`ssh -i ~/.ssh/grader_key -p 2200 grader@XX.XX.XX.XX`

Note that a window will pop-up asking for `grader`'s password


### Configure the local timezone
1. Run `sudo dpkg-reconfigure tzdata`

2. Test the timezone is configured correctly by running `date`


### Install and configure Apache
1. Install Apache
   * Run `sudo apt-get install apache2`

2. Verify it worked by entering the public IP of the Amazon Lightsail instance
   as a URL in a browser
	 * If Apache is working correctly, a page with the title 'Apache2 Ubuntu Default Page' should load


### Install mod_wsgi
1. Install the mod_wsgi package
   * Run `sudo apt-get install libapache2-mod-wsgi python-dev`

2. Verify mod_wsgi is enabled
   * Run `sudo a2enmod wsgi`


### Install PostgreSQL and verify PostgreSQL is not allowing remote connections
1. Install PostgreSQL
  * Run `sudo apt-get install postgresql`

2. Check the /etc/postgresql/9.5/main/pg_hba.conf file

3. The output of the file should look like this:

	```
	local   all             postgres                                peer
	local   all             all                                     peer
	host    all             all             127.0.0.1/32            md5
	host    all             all             ::1/128                 md5
	```

### Verify Python is installed
1. Python should already be installed on an Ubuntu 16.04 machine
   * Run `python`
2. The following output should appear:

Python 2.7.12 (default, Nov 12 2018, 14:36:49)
[GCC 5.4.0 20160609] on linux2
Type "help", "copyright", "credits" or "license" for more information.

### Create a PostgreSQL user named `catalog` with limited permissions
1. PostgreSQL creates a Linux user with the name `postgres` during installation
2. Switch to this user
  * Runn `sudo su - postgres`

3. Connect to psql
   * Run `psql`

4. Create the `catalog` user
   * Run `CREATE ROLE catalog WITH LOGIN;`

5. Give `catalog` user the ability to create databases
   * Run `ALTER ROLE catalog CREATEDB;`

6. Give `catalog` user a password
   * Run `\password catalog`

7. Verify `catalog` user was created
  * Run `\du`
8.	A table will appear, looking like this:

	```
					   List of roles
	 Role name |                         Attributes                         | Member of
	-----------+------------------------------------------------------------+-----------
	 catalog   | Create DB                                                  | {}
	 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
	```

9. Exit psql
   * Run `\q`

10. Change back to the `ubuntu` user
    * Run `exit`


### Create a Linux user named `catalog` and a new PostgreSQL database
1. Create a new Linux user called `catalog`:
   * Run `sudo adduser catalog`
	 * Enter in a new UNIX password (twice) when prompted
	 * fill out information for `catalog`

2. Give `catalog` user sudo permissions:
   * Run `sudo visudo`

   * Find a line similar this:

	`root    ALL=(ALL:ALL) ALL`

   * Add the following line below it:

	`catalog	   ALL=(ALL:ALL) ALL`
	 * Save and close the visudo file

	 * Verify `catalog` has sudo permissions
	   * Run `su - catalog`
		 * Enter the password
		 * Run `sudo -l`
		 * Enter the password again
		 * The following output should appear, showing `grader` has sudo permissions:

		```
		User catalog may run the following commands on
			ip-XX-XX-XX-XX.ec2.internal:
		    (ALL : ALL) ALL
		```

3. While logged in as `catalog`

4. Create a database named catalog
   * Run  `createdb catalog`
	 * Run  `psql`
	 * Run  `\l` (to verify a new database has been created)


5. Change back to the `ubuntu` user
   * Run `exit`


### Install git and clone the catalog project
1. Run `sudo apt-get install git`

2. Create a directory called 'catalog' in the /var/www/ directory

3. `cd` in to the 'catalog' directory

4. Clone the catalog project:

	 * Run `sudo git clone https://github.com/dgethner/Item-Catalog.git catalog`

	 * the word catalog at the end is to make the fold structure easier

5. Change the ownership of the 'catalog' directory to `ubuntu`
   * Run `sudo chown -R ubuntu:ubuntu catalog/`

6. `cd` to the /var/www/catalog/catalog directory

7. Change the name of the catalog.py file to \_\_init__.py
   * Run `mv application.py __init__.py`

8. In __init__.py, find the `app.run` line:

	 * `app.run(host='0.0.0.0', port=8000)`

	 * Change this line to:

   * `app.run()`

### Change the database in the application from SQLite to PostgreSQL
Replace the following line in __init__.py, database_setup.py, and lotsofcars.py

   * engine = create_engine('sqlite:///carTypes.db?check_same_thread=False')
with the following:

	 * engine = create_engine('postgresql://catalog:Your_Database_Password@localhost/catalog')


### Add client_secrets.json
1. Authenticate login through Google:
   * Create a new project on the Google API Console
	 * Create an OAuth Client ID (under the Credentials tab)
	 * Add http://18.218.147.158 and http://ec2-18-218-147-158.us-east-2.compute.amazonaws.com as authorized JavaScript origins
   * Add http://ec2-18-218-147-158.us-east-2.compute.amazonaws.com/oauth2callback,
	   http://ec2-18-218-147-158.us-east-2.compute.amazonaws.com/login, and
		 http://ec2-18-218-147-158.us-east-2.compute.amazonaws.com/gconnect as authorized redirect URIs
   * Download the corresponding JSON file
	 * Open it and copy the contents
   * Open /var/www/catalog/catalog/client_secret.json
	 * Paste the contents of the downloaded file into the this file
   * Replace the client ID in the templates/login.html file in the project directory


### Set up a vitual environment and install dependencies
1. Install pip
   * Run `sudo apt-get install python-pip`

2. Install virtualenv
   * Run `sudo apt-get install python-virtualenv`

3. `cd` to the /var/www/catalog/catalog/ directory

4. Pick a name for a temporary environment and create this environment
   * Run `virtualenv venv`
	 * `sudo` was not needed to create this environment

5. Activate the new environment, `venv`
   * Run `. venv/bin/activate`

6. While the virtual environment active, install the following dependencies
	* Run `pip install httplib2`
	* Run `pip install requests`
	* Run `pip install --upgrade oauth2client`
	* Run `pip install sqlalchemy`
	* Run `pip install flask`
	* Run `sudo apt-get install libpq-dev`
	* Run `pip install psycopg2`

7. Verify everything was installed correctly
   * Run `python __init__.py`
	 * The following output should appear:

	`* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)`

8. Deactivate the virtual environment
   * Run `deactivate`


### Set up and enable a virtual host
1. Create a file in /etc/apache2/sites-available/ named catalog.conf

2. Add the following info into the file:

	```
	<VirtualHost *:80>
                ServerName 18.218.147.158
                ServerAlias ec2-18-218-147-158.us-east-2.compute.amazonaws.com
                ServerAdmin dgethner@gmail.com
                WSGIScriptAlias / /var/www/catalog/catalog.wsgi
                <Directory /var/www/catalog/catalog/>
                        Order allow,deny
                        Allow from all
                        Options -Indexes
                </Directory>
                Alias /static /var/www/catalog/catalog/static
                <Directory /var/www/catalog/catalog/static/>
                        Order allow,deny
                        Allow from all
                        Options -Indexes
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
	```

3. Enable the virtual host
   * Run `sudo a2ensite nuevoMexico`

	The following prompt will be returned:

	```
	Enabling site catalog.
	To activate the new configuration, you need to run:
	  service apache2 reload
	```

  * Run `sudo service apache2 reload`


### Write a .wsgi file
1. Apache serves Flask applications by using a .wsgi file

2. Create a file called catalog.wsgi in /var/www/catalog

3. Add the following info to the file:

	```
	activate_this = '/var/www/catalog/catalog/venv/bin/activate_this.py'
execfile(activate_this, dict(__file__=activate_this))

#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'super_secret_key'
	```

  * Run `sudo service apache2 reload`



### Disable the default Apache site
1. To Disable the default Apache site
   * Run `sudo a2dissite 000-default.conf`

	The following prompt will be returned:

	```
	Site 000-default disabled.
	To activate the new configuration, you need to run:
	  service apache2 reload
	```

   * Run `sudo service apache2 reload`


### Change owners of the project directories
1. To change the ownership of the project directories and files to the
   `www-data` user
2. `cd` in to the /var/www directory
    * Run	`sudo chown -R www-data:www-data catalog/`
3. To edit these files use this example
    * sudo -u www-data nano File_Name


### Configure the database schema and populate
1. `cd` in to /var/www/catalog/catalog/
2. Activate the virtualenv
   * Run `. venv/bin/activate`
   * Run `python lotsofcars.py`

3. Deactivate the virtualenv
   * Run `deactivate`

4. Restart Apache
   * Run `sudo service apache2 restart`

5. Go to your browser and verify the app is working by visiting the following links
   * http://18.218.147.158
	 * http://ec2-18-218-147-158.us-east-2.compute.amazonaws.com


## Commands to assist you

1. `sudo service apache2 restart`<br>
This restarts the Apache service, when you make changes run this to see them in the app

2. `nano /var/log/apache2/error.log`<br>
You can View Apache error logs


## Sources
Below is a list of sources I used to complete this project.

Udacity course: [Configuring Linux Web Servers](https://www.udacity.com/course/configuring-linux-web-servers--ud299)

Flask [documentation](http://flask.pocoo.org/docs/0.12/installation/) on virtualenv

tutorialspoint [tutorial](https://www.tutorialspoint.com/postgresql/postgresql_create_database.htm) on creating a database with PostgreSQL

concrete5 [tutorial](https://legacy-documentation.concrete5.org/tutorials/access-virtual-hosts-from-any-device-using-xip-io)

xip.io [website](http://xip.io/)

Xip.io and Wildcard Subdomains for Development [blog](https://serversforhackers.com/c/xipio-and-wildcard-subdomains-for-development)

GitHub Repositories
   * [boisalai/udacity-linux-server-configuration](https://github.com/boisalai/udacity-linux-server-configuration)
   * [bencam/linux-server-configuration](https://github.com/bencam/linux-server-configuration)
   * [iliketomatoes/linux_server_configuration](https://github.com/iliketomatoes/linux_server_configuration)

Udacity forum posts: [Google Sign In Nothing Happening](https://knowledge.udacity.com/questions/27496), switching from SQLite to [PostgreSQL](https://discussions.udacity.com/t/how-to-move-a-flask-app-from-using-a-sqlite3-db-to-postgresql/7004/5), virtualenv [packages](https://discussions.udacity.com/t/importerror-no-module-named-psycopg2-project5/35018/2), Apache [errors](https://discussions.udacity.com/t/running-apache-problem/35719/3)

Stack Overflow: servername [error](http://askubuntu.com/questions/329323/problem-with-restarting-apache-2) with Apache,[reload](http://serverfault.com/questions/61383/reloading-postgresql-after-configuration-changes), logging in as [www-data](https://ubuntuforums.org/showthread.php?t=2280551) changing [permissions](http://superuser.com/questions/19318/how-can-i-give-write-access-of-a-folder-to-all-users-in-linux)
