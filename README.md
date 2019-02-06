# Linux-Server-Configuration-Udacity

##IP and URL:
	Public IP: http://35.154.248.64
	Host name: ec2-35-154-248-64.ap-south-1.compute.amazonaws.com

##How To:
#Amazon Lightsail
	1. Create Lightsail account and new Instance
	2. Connect using SSH
	3. Download private key
	4. In the Network tab, add two new custom ports - 123 and 2200

#Server Configuration
	5. Place private key in .ssh
	6. chmod 600 ~/.ssh/LightsailDefaultPrivateKey-ap-south-1.pem
	7. rename the key from LightsailDefaultPrivateKey-ap-south-1.pem to lightsail_key.pem
	8. ssh -i ~/.ssh/lightsail_key.pem ubuntu@35.154.248.64

#Create new account grader
	9. Type sudo su - to become a root user.
	10. sudo adduser grader (to create new user) #password:(grader1234)
	#passphrase: (graderuser)
	11. Open sudo nano /etc/sudoers.d/grader . Fill that file with grader ALL=(ALL:ALL) ALL, save it.
	12. Edit the hosts by sudo nano /etc/hosts , and then add 127.0.0.1 ip-172-26-0-44 under 127.0.0.1:localhost

#Install updates and finger
	13. sudo apt-get update
	14. sudo apt-get upgrade
	15. sudo apt-get finger

#If you face any problem like:
	locale.Error: unsupported locale setting
	then just run the command

	$ export LC_ALL=C

#Key Gen
	16. Open a new terminal window and input $ ssh-keygen -f ~/.ssh/udacity_key.rsa
	17. $ cat ~/.ssh/udacity_key.rsa.pub and copy the public key
	18. Going back to the first terminal window where you are logged into amazon Lightsail as the root user, move to grader's folder $ cd /home/grader
	19. Create .ssh directory $ mkdir .ssh
	20. Create a file to store the public key: $ touch .ssh/authorized_keys
	21. $ nano .ssh/authorized_keys to edit the file
	22. Change permission $ sudo chmod 700 /home/grader/.ssh and $ chmod 644 /home/grader/.ssh/authorized_keys
	23. Change the owner from root to grader: $ sudo chown -R grader:grader /home/grader/.ssh
	24. Restart the ssh $ sudo service ssh restart
	25. $ ~. disconnect
	26. ssh -i ~/.ssh/udacity_key.rsa grader@35.154.248.64

#Enforce Key based authentication
	27. $ sudo nano /etc/ssh/sshd_config find the PasswordAuthentication line and change text after to no. After this, restart ssh again: $ sudo service ssh restart

#Change PORT
	28. $ sudo nano /etc/ssh/ssdh_config Find the Port line and change 22 to 2200. Restart ssh: $ sudo service ssh restart.
	29. $ ~. disconnect and again log back using $ ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@35.154.248.64

#Disable root login
	30. $ sudo nano /etc/ssh/sshd_config. Find the PermitRootLogin line and edit to no. Restart ssh $ sudo service ssh restart.

#Configure UFW
	31. $ sudo ufw allow 2200/tcp
		$ sudo ufw allow 80/tcp
		$ sudo ufw allow 123/udp
		$ sudo ufw enable

#Install Apache and GIT
	32. $ sudo apt-get install apache2
	33. $ sudo apt-get install libapache2-mod-wsgi python-dev
	34. $ sudo apt-get install git

# Enable mod_wsgi
	35. $ sudo a2enmod wsgi
		$ sudo service apache2 restart
	36. You should input the public IP address and you should see the ubuntu default page in your browser.

#Setup Folders
	36. $ cd /var/www
		$ sudo mkdir catalog
		$ sudo chown -R grader:grader catalog
		$ cd catalog

#Clone the Catalog Project
	37. git clone [your link] catalog

#Create a .wsgi file
	38. $ sudo nano catalog.wsgi

		import sys
    	import logging
    	logging.basicConfig(stream=sys.stderr)
    	sys.path.insert(0, "/var/www/catalog/")

    	from catalog import app as application
    	application.secret_key = 'super_secret_key'
    39. rename the application.py to __init__.py

#virtual Machine
	40. $ sudo pip install virtualenv
		$ sudo virtualenv venv
		$ source venv/bin/activate
		$ sudo chmod -R 777 venv

#Install flask and other packages
	41. $ sudo apt-get install python-pip
		$ sudo pip install Flask
		$ sudo pip install httplib2
		$ sudo pip install oauth2client
		$ sudo pip install sqlalchemy
		$ sudo pip install psycopg2
		$ sudo pip install sqlaclemy_utils
		$ sudo pip install requests
		$ sudo pip install render_template
		$ sudo pip install redirect
	42. $nano __init__.py Change the client_secrets.json line to /var/www/catalog/catalog/client_secrets.json
	43. Change the host to 35.154.248.64 and port to 80

#Configure virtual host
	44. $ sudo nano /etc/apache2/sites-available/catalog.conf

	<VirtualHost *:80>
    	ServerName 35.154.248.64
    	ServerAlias ec2-35-154-248-64.ap-south-1.compute.amazonaws.com
    	ServerAdmin admin@35.154.248.64
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

#Setup the DATABASE
	45. $ sudo apt-get install libpq-dev python-dev
		$ sudo apt-get install postgresql postgresql-contrib
		$ sudo -u postgres -i
		$ psql
	46. $ CREATE USER catalog WITH PASSWORD 'catalog';
		$ ALTER USER catalog CREATEDB;
		$ CREATE DATABASE catalog WITH OWNER catalog;
		Connect to database $ \c catalog
		$ REVOKE ALL ON SCHEMA public FROM public;
		$ GRANT ALL ON SCHEMA public TO catalog;
		Quit the postgres command line: $ \q and then $ exit
	47. $ nano __init__.py Edit database_setup.py, and 
		moresitems.py files to change the database engine from sqlite://catalog.db to postgresql://catalog:catalog@localhost/catalog
	48. Restart Apache server $ sudo service apache2 restart
		and enter the public ip address or host name into the browser.
	49. Add ec2-35-154-248-64.ap-south-1.compute.amazonaws.com
		to Authorized JavaScript Origins and Authorised redirect URIs on Google Developer Console.
	50. $ sudo service apache2 restart
