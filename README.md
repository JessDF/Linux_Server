# Linux_Server
Udacity Fullstack Web Development Nanodegree Program - Project 8: Linux Server

### Check it out
Instance Name: udacity-jessie-dowding <br>
IP Address: 52.10.188.173 <br>
Public URL: http://52.10.188.173/

## 1. Login and create your server
 - Go to https://lightsail.aws.amazon.com to create an aws account
 - Create a LightSail instance with Ubuntu
 - Create a instance plan, the $5 a month plan should be enough
 - Create an instance host name
 
## 2. Update all currently installed packages by running:

    ```
     sudo apt-get update
     sudo apt-get upgrade
     sudo apt-get dist-upgrade
    ```
## 3. Change the **SSH port** from *22* to *2200*

    ```
    # Open the SSH configuration file
    nano /etc/ssh/sshd_config

    # Change this:
    Port 22
    # To:
    Port 2200

    # Save and exit

    # Restart the service
    service ssh restart
    ```

## 4. Configure the **Uncomplicated Firewall (UFW)** to allow incoming connections for *SSH (2200)*, *HTTP (80)* and *NTP(123)*

    ```
    # Check the current status of the UFW
    sudo ufw status

    # Default to deny incoming connections and allow outgoing ones
    sudo ufw default deny incoming
    sudo ufw default allow outgoing

    # Allow the specified connections
    sudo ufw allow 2200/tcp
    sudo ufw allow www
    sudo ufw allow ntp

    # Enable the UFW
    sudo ufw enable
    ```
 - Check UFW status after updates `sudo ufw status` This will show that UFW is now active with the settings below:
   ```
   To                         Action      From
   --                         ------      ----
   22                         ALLOW       Anywhere
   2200/tcp                   ALLOW       Anywhere
   80/tcp                     ALLOW       Anywhere
   123/udp                    ALLOW       Anywhere
   22 (v6)                    ALLOW       Anywhere (v6)
   2200/tcp (v6)              ALLOW       Anywhere (v6)
   80/tcp (v6)                ALLOW       Anywhere (v6)
   123/udp (v6)               ALLOW       Anywhere (v6)
   ```
## 5. Add a new user
 - As root and then follow the prompts to add a password and name for this new account. 
 - Once you do that, you will need to add the user to the sudoers file.
    ```
    sudo adduser grader
    
    # Confirm user
    sudo cat /etc/passwd
    
    # Add user to sudoers with this command
    sudo visudo
    grader   ALL=(ALL:ALL) ALL  # add this line to the file
    ```
 
 ## 6. Set-up Key-based Authentication/Public Key Encryption
 - Using ssh-keygen on your local machine, create a key for the user grader. <br>
    ```
    # Create the keys using an email as comment
    ssh-keygen -C grader@udacity.com
    ```
   Follow the prompts to provide a name for this key and use the default key location (~/.ssh). <br>
   This process will create two keys on your local machine, the file with extension .pub is the public key to be transferred to the server.

## 7. Add and authenticate with Public Key
 - Login to the server as the new grader user
 - Run these commands in your home directory 
    ```
    mkdir .ssh
    touch .ssh/authorized_keys

    # Change both, directory and file, permissions to the grader user
    chmod 700 /home/grader/.ssh
    chmod 644 /home/grader/.ssh/authorized_keys

    # Change both, directory and file, ownership to the grader user
    chown grader /home/grader/.ssh
    chown grader /home/grader/.ssh/authorized_keys
    
    # You may also need to uncomment this line
    sudo nano /etc/ssh/sshd_config
    AuthorizedKeysFile %h/.ssh/authorized_keys
    ```
 - Copy-and-paste the contents of the .pub key file created on your local machine 
   above to the server as the contents of the authorized_keys file and run `service ssh restart`
   
## 8. Disable Password Based Login and Force Login using Key Pair and disable root user login
 - On the server, logged in as the grader user, edit the sshd_config file `sudo nano /etc/ssh/sshd_config` 
 - Change the line with Password Authentication from yes to no
 - This is read only when the service starts, so to restart the ssh service `sudo service ssh restart`
 - On the server, logged in as root, edit the sshd_config file `sudo nano /etc/ssh/sshd_config`
 - Change to `PermitRootLogin no`
 - Enable grader user remote ssh login `sudo nano /etc/ssh/sshd_config` and add `AllowUsers grader`
 - Restart the ssh service `sudo service ssh restart`

## 9. Configure local Timezone to UTC
 - Start the configuration process `dpkg-reconfigure tzdata`
 - In the window that appears, use the arrow keys to Scroll to the bottom of 
   the Continents list and select `Etc or None of the Above` and then in the second list, select UTC <br>
 - Confirm time change by typing `date` on the command line <br>
 - Reference Documentation: http://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt/138442 
 
 ## 10. Install and configure **Apache server** to serve a **WSGI** Python app.

    ```
    # Install Apache
    sudo apt-get install apache2

    # Install WSGI for Python 3
    sudo apt-get install libapache2-mod-wsgi-py3

    # Enable the WSGI module
    sudo a2enmod wsgi
    ```
 - In order to check if everything's working correctly you can open your server's address in a browser and the Apache's default page should display.
 - Now, to test that WSGI is working:

    ```
    # Open the apache's default site configuration file
    sudo nano /etc/apache2/sites-enabled/000-default.conf

    # Right before the </VirtualHost> closing tag, add the following:
    WSGIScriptAlias / /var/www/html/myapp.wsgi
    ```

- This tells apache to serve a `myapp.wsgi` app whenever the / route is requested. Now do:

    ```
    # Create the app file and add
    sudo nano /var/www/html/myapp.wsgi

    # Which is this:
    def application(environ, start_response):
        status = '200 OK'
        output = 'Hello Udacity'

        response_headers = [('Content-type', 'text/plain'$
        start_response(status, response_headers)

        return [output]

    # Now restart the apache server
    sudo apache2ctl restart
    ```
- If you reload the page in your browser and see the 'Hello Udacity' message. Now apache and wsgi are initalize and set up.

## 11a. Install **PostgreSQL**
    ```
    sudo apt-get install postgresql
    ```
## 11b. Create a new database user named with limited permissions to the database
 - Connect to database as the user postgres `sudo su - postgres` 
 - Type `psql` to generate PostgreSQL prompt
 - Create a new user: `CREATE USER catalog WITH PASSWORD 'your_passwd';`
 - Confirm that the user was created: `\du`
 - Reference Documentation: http://www.postgresql.org/docs/9.1/static/sql-createrole.html 

## 11c. Limit permissions to new database user 
 - Run `\du` to see what permissions the user _catalog_ has
 - To see possible user roles, type: `\h CREATE ROLE`
 - Update permissions for catalog user: <br>
   ```
   ALTER ROLE catalog WITH LOGIN;
   ALTER USER catalog CREATEDB;
   ```
 - Create the database: `CREATE DATABASE catalog WITH OWNER catalog;`
 - Login to the database: `\c catalog`
 - Revoke all rights: `REVOKE ALL ON SCHEMA public FROM public;` 
 - Grant only access to the catalog role: `GRANT ALL ON SCHEMA public TO catalog;` 
 - Exit out of PostgreSQL and the postgres user: `\q`, then `exit`
 - Restart postgresql: `sudo service postgresql restart`
 - Reference Documentation: https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps

## 12. Install **Git**
 - Intall git with: `sudo apt-get install git`
 - Edit Git Configuration
   ```
   git config --global user.name "Your Name"
   git config --global user.email youremail@domain.com
   ```

## 13. Clone Udacity Project: ItemCatalog
 - I did some file reorganization on my project after cloning (as I have my catalog folder within a vagrant folder)
 - To clone run the command: `git clone https://github.com/JessDF/itemCatalog`
 - Now reorganize the file structure to look like
   ```
   /var
      /www
         /catalog
            /catalog
               /images
               /static
               /templates
               database_operations.py
               install_db.py
               item_catalog.db
               item_catalog.py
               item_database_config.py
    ```
 - You will also need to add a client_secret.json file with in the /var/www/catalog/catalog folder
 - Also add a redirct so that .git can't be accessed with in a a .htaccess file and include this line: `RedirectMatch 404 /\.git`
 - Lastly, replace lines in:
 	- item_catalog.py # change file name to __init__.py
	- templates/main.html  # In this file, add client secret in spot that says: <CLIENT_SECRET_REPLACE>
	- catalog/item_catalog.py  # Where it says 'client_secret.json'  replace with '/var/www/catalog/catalog/client_secret.json'
	- templates/login.html # In this file, add client secrete in spot that says: <REPLACE CLIENT ID>
	- __init__.py (use to be item_catalog.py) # app.secret_key = 'super_secret_key' replace 'super_secret_key' with client secret
	- __init__.py (use to be item_catalog.py) # change app.run(host='0.0.0.0', port=8080) to app.run()


## 14. Install Flask and create Virtual Environment for the Catalog App
- Set up a *virtual environment* in order to keep the application and its dependencies isolated from the main system:
   ```
   # Install PIP for Python 3
   sudo apt-get install python3-pip

   sudo pip3 install virtualenv

   # Create environment
   sudo virtualenv venv

   # Activate the environment
   source venv/bin/activate
   ```
- Now, install the project dependencies. For this one:
  ```
  sudo -H pip3 install Flask
  sudo -H pip3 install sqlalchemy
  sudo -H pip3 install oauth2client
  sudo -H pip3 install requests
  sudo -H pip3 install psycopg2
  ```
## 15. Now tell Apache about our app
 - Create the site configuration file: `sudo nano /etc/apache2/sites-available/catalog.conf`
 - Add the following to the file:
   ```
   <VirtualHost *:80>
		  	ServerName catalog.com
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
 - References: http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/ and https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
 - Create the application entry point: `sudo nano /var/www/catalog/catalog.wsgi`
 - Add the following to the file:
   ```
   #!/usr/bin/python
   import sys
   import logging
   logging.basicConfig(stream=sys.stderr)
   sys.path.insert(0,"/var/www/catalog/")
 
   activate_this = '/var/www/catalog/catalog/venv/bin/activate_this.py'
   with open(activate_this) as file:
       exec(file.read(), dict(__file__=activate_this))

   from catalog import app as application
   application.secret_key = 'super_secret_key' # replace 'super_secret_key' with client secret
    ```
 - Now run the following commands:
    ```
    sudo a2dissite 000-default.conf
    sudo a2ensite catalog.conf
    sudo service apache2 restart
    ```
## 16. Set Up the database
 -  Change every `create_engine('sqlite:///catalog.db')` line to `create_engine('postgresql://catalog:item-catalog-passwd@localhost/catalog')`.
 	- Files to change include: `database_operations.py` and `item_database_config.py`
	- In the above line, don't forget to replace "item-catalog-passwd" with actual password
 - We will now need to populate the database by running:
   ```
   sudo python install_db.py
   sudo python item_database_config.py
   ```
<hr>
Everything should be working now. I want to thank Fernando Saavedra (@fsvdr) and Péter Szabó (@peta) for the help. I spoke with them on the udacity channel and they were a big help.
