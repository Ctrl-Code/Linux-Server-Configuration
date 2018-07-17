>   # LINUX SERVER CONFIGURATION

***

This configuration file corresponds to Term 2 of [Udacity's Full Stack Web Developer Nanodegree Programme](https://in.udacity.com/course/full-stack-web-developer-nanodegree--nd004).

***
***

>   ## ABOUT THE PROJECT

This project is about hosting the choosen [web-application](https://github.com/Ctrl-Code/Item-Catalog) on a server based on [Ubuntu](www.ubuntu.com) 16.04 LTS linux using [Apache](https://httpd.apache.org/) open web-server so that the web-application can be readily accessed throughout the world.

***
***

>   ## RUNNING THE WEB-APPLICATION

1. The web-application can be accessed via below mentioned IP and incorporates SSH port as mentioned.

    * **IP** - [13.126.86.132](http://13.126.86.132)
    * **SSH Port** - 2200

2. The user `grader` can login using ssh command in the terminal.

    ```bash
    ssh -p 2200 -i <PRIVATE_KEY_FILE_PATH> grader@13.126.86.132
    ```
***
***

> ## SOFTWARES/ PACKAGES REQUIRED

1. Latest updation and upgradation of Packages. This is done as shown below
    
    ```$
    sudo apt-get update
    sudo apt-get upgrade
    ```
    This automatically updates and upgrades `python` and other packages to latest stable version.

2.  Some required python packages can be installed using `python-pip`.
    
    Install python-pip and upgrade it as
    
    ```bash
    sudo apt-get install python-pip
    sudo pip install --upgrade pip
    ```

    And install other required python packages through `pip` as
    
    ```bash
    sudo pip install sqlalchemy psycopg2 oauth2client requests
    ```

3. Install `Apache` a open web-server, `mod_wsgi` request handler, `PostGreSQL` a database management tool and `python-dev` by

    ```bash
    sudo apt-get install apache2 libapache2-mod-wsgi python-dev postgresql
    ```

***
***

>## CONFIGURATION FOR **GRADER** SETUP

1. Type in below given commands to create a new user `grader`.

    ```bash
    sudo adduser grader
    ```
    Type in the credentials and save it.

2. Now save the `public RSA key` generated using `ssh-keygen` command in the file /home/grader/.ssh/authorized_keys

    ```bash
    sudo -u grader mkdir /home/grader/.ssh
    sudo -u grader nano /home/grader/.ssh/authorized_keys
    ```

3. Now give `sudo access` to the **grader** by opening the file /etc/sudoers.d/90-cloud-init-users and making changes as follows

    ```bash
    sudo nano /etc/sudoers.d/90-cloud-init-users

    ```

    Now paste in the keys and save the file.

    ```
    # User rules for grader
    grader ALL=(ALL) NOPASSWD:ALL
    ```

    Save the file and close it.

This gives the sudo rights to the user 'grader'.

***
***

>## CONFIGURATION FOR FIREWALL SETUP

1.  It involves a list to be followed to allow particular ports and need no explanation. The command itself specifies what it is doing.

    ```bash
    sudo ufw default allow outgoing
    sudo ufw default deny incoming
    sudo ufw allow 80
    sudo ufw allow 123
    sudo ufw allow 2200
    sudo ufw enable
    ```

    Here 2200 is the new ssh port.

2. Edit the ssh port to become **2200** from **22** by making `PORT 22` as `PORT 2200` in the file /etc/ssh/sshd_config.

    ```bash
    sudo nano /etc/ssh/sshd_config
    ```

     Also disable password login such as login is only allowed through `public-private` key by making 
     
     `PasswordAuthentication` ~~no~~ `yes` in the same file.

3. Now restart the `ssh service so that the user **grader** can connect to it using its private key.
    ```bash
    sudo service ssh restart
    ```
***
***

>## CONFIGURATION FOR SERVER SETUP

1. Open up psql as admin

    ```bash
    sudo -u postgres psql
    ```

2. Create a user named **catalog** in 'psql' or 'postgresql' with superuser and login rights.
    
    ```bash
    create role catalog superuser;
    alter role catalog login;
    ```
  
3. Add password **cba** to the user **catalog**

    ```bash
    \password catalog
    ```

    Type `cba` + `<enter>` twice when prompted.

4. Now exit from psql as admin using psql `\q` meta and connect as user **catalog**
    
    To quit psql
    ```bash
    \q
    ```

    Connect as user **catalog**

    ```bash
    psql -U abc -h localhost postgres
    ```

    Type `cba` + `<enter>` when prompted.

5. Create a database named **mazak** and exit from psql console
    
    ```bash
    create database mazak;
    ```
  
    To quit
    ```bash
    \q
    ```
6. Steps involved to configure the directory structure for the web-application

    ```bash
    cd /var/www
    sudo mkdir itemCatalog
    cd itemCatalog
    sudo mkdir itemCatalog
    cd itemCatalog
    sudo mkdir static templates
    ```

    Here **itemCatalog** could be anyname choosen and is a identifier, but the same name is followed throughout the configuration so as not to get confused.

    Now paste in your <`main-python.py`> file here along with the other files in the same file structure with reference to the main file and rename it as `__init__.py`.

7. Setup python `virtual environment` as shown in the same directory as we are in by typing in the commands

    ```bash
    sudo pip install virtualenv
    sudo virtualenv venv
    source venv/bin/activate
    python __init__.py
    ```
    Here **venv** is the object of virtualenv and thus could have been given any name.
    `<ctrl>` + `C` to cancel the file run

    If the file is running without any errors; implies that the virtual environment is successfully setup else repeat the process again to make sure it runs successfully.

    Now deactive the environment
    
    ```bash
    deactivate
    ```

8. Create the `mod-wsgi` configuration file using below given comman as in given directory

    ```bash
    sudo nano /etc/apache2/sites-available/itemCatalog.conf
    ```

    Here **itemCatalog** can be any name.

9. Now paste this content in the nano file which opened up in step 8 above

    ```bash
    <VirtualHost *:80>
		ServerName 13.126.86.132
		ServerAdmin chebomsta@gmail.com
		WSGIScriptAlias / /var/www/itemCatalog/itemCatalog.wsgi
		<Directory /var/www/itemCatalog/itemCatalog/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/itemCatalog/itemCatalog/static
		<Directory /var/www/itemCatalog/itemCatalog/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
    ```

    Above replace **ServerName** with your Server Name and **ServerAdmin** with your way of contacting admin. See that the directory order is followed and is not mistaken.

10. To enable the config file for the web-application, type command
    
    ```bash
    sudo a2ensite itemCatalog
    ```

11. Now we will create the wsgi simple program file which will see the requests and pass it on. This file would be contacted whenever we try to communicate using above IP.

    ```bash
    cd /var/www/itemCatalog
    sudo nano itemCatalog.wsgi
    ```

12. Now paste in the below given contents and restart apache server.

    ```bash
    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/itemCatalog/")

    from itemCatalog import app as application
    application.secret_key = 'Add your secret key'
    ```

    Save and Close the file. Open terminal and type in below given command to restart `apache` service.

    ```bash
    sudo service apache2 restart
    ```
***
***

>### REFERENCES

1. Udacity's Course on **Linux Server Configuration**
2. **[Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)** for providing explanation and steps for flask **flask** application using **mod-wsgi** on **ubuntu**.
3. **[Flask Documentation](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/)** for knowing about the mod_wsgi setup.