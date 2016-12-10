# LINUX SERVER CONFIGURATION
  This project covers deploying Flask application on amazon web server, Configure firewall to prevent attacks, Change ssh port number for security reason, set up database with password.

## Lauch VM using Udacity credentials
   The private key along with the hostname and public IP address is provided in the course development environment.
   You can set your amazon EC2 instance by following,
   http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/get-set-up-for-amazon-ec2.html

## Change SSH port and create user
   SSH into the linux using the amazon EC2 instance using the following command,
    ssh -i /path/to/rsa_file.rsa root@PUBLIC_IP_ADDRESS           --The public IP address is generated while setting up EC2 instance.

    add user by,
     1.sudo adduser NEW_USER
     2. navigate to /etc/sudoers.d and create a file
        The content should be,
        <New_user> ALL=(ALL) NOPASSWD:ALL
    Update installed pacakges,
         sudo apt-get update

##Change time to UTC
    sudo dpkg-reconfigure tzdata

##Change ssh port 22 to 2200
    The port 22 is common ssh port, you can face the probablity of attack. It's better to change the ssh port to avoid intruder attack.
    1. sudo nano /etc/ssh/sshd_config
    2. add port 2200 next to port 22 in sshd_config file

##Configure Firewall and restart ssh at port 2200
    The ufw can be configured to allow http, ntp, ssh using the commands:
    sudo ufw allow 22
    sudo ufw allow 2200
    sudo ufw allow www
    sudo ufw enable
     
    Now restart ssh by,
    sudo service ssh restart

    Login again into EC2 instance by the following command,
    sudo -p 2200 -i /path/to/.rsa_file root@<Public_ip_address>

    you can check the ssh port number by following command,
    echo ${SSH_CLIENT##* }
    It should return 2200. 

##Install apache2 server
  Do $sudo apt-get update 
  Install apache2 by,
  sudo apt-get install apache2
  Install mod_wsgi dependency to host python based web apps,
  sudo apt-get install libapache2-mod-wsgi
  Then restart the apache2,
  $ sudo service apache2 restart

##Clone catalog project from git and change hosts config to serve catalog project
  1. git clone <giturl_of_the project>
  2. inside /var/www create a directory catalog.
  3. Inside /var/www/catalog create another directory catalog and store the files cloned fron git repo.
  4.create .wsgi file inside /var/www/catalog 

  To make git unaccessable,
  $ sudo nano .htaccess
  and paste,
  RedirectMatch 404 /\.git

##Install virtual environment and Flask
  1. $sudo apt-get install libapache2-mod-wsgi python-dev
  2. $sudo a2enmod wsgi 
  The above command will enable wsgi.

  Install virtualenv to isolate our project and it's dependencies
  1. $sudo pip install virtualenv
  Name your virtualenv 
  2. $sudo virtualenv enter_your_virtualenv_name
  Enable permissions 
  3. $sudo chmod -R 777 enter_your_virtualenv_name
  activate the virtualenv by,
  source enter_your_virtualenv_name/bin/activate

  Inside virtualenv,
  1.Install flask and all project dependencies,
  $ sudo pip install Flask
  pip install httplib2
  $ pip install requests
  $ sudo pip install flask-seasurf
  $ sudo pip install --upgrade oauth2client
  $ sudo pip install sqlalchemy
  $ sudo apt-get install python-psycopg2
  
  exit the virtualenv by the following command,
  $deactivate

##Configure sites-available
  1. move to /etc/apache2/sites-available, open your app config or create one
  2. Paste the following,
  <VirtualHost *:80>
        ServerName <PUBLIC_IP>
        ServerAdmin admin@<PUBLIC_IP>
        WSGIScriptAlias / /var/www/<app_name>/<app_name>.wsgi
        <Directory /var/www/<app_name>/<app_name>/>
            Order allow,deny
            Allow from all
        </Directory>
        Alias /static /var/www/<app_name>/<app_name>/static
        <Directory /var/www/<app_name>/<app_name>/static/>
            Order allow,deny
            Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
  3. sudo a2ensite <app_name>

##Create wsgi file
  1. inside /var/www/your_app
  2. create a .wsgi file
  3. change the path to your app path,
     sys.path.insert(0,"/var/www/your_app_path/")
  4. sudo service apache2 restart

##Install PSQL
  The psql db installed by,
  sudo apt-get install postgresql postgresql-contrib
  $ sudo adduser catalog
  Enter into psql,
  # sudo -i -u catalog
  psql
  Inside PSQL databse, 
  CREATE USER catalog WITH PASSWORD '<USER_PASSWORD>';
  CREATE DATABASE <DB_NAME> WITH OWNER catalog;
  \c <DB_NAME>
  To restrict access, first revoke all public access and then allow access only to the catalog,
  REVOKE ALL ON SCHEMA public FROM public;
  GRANT ALL ON SCHEMA public TO catalog;

##Enable Google+ and Facebook oauth
  1. In both Google+ and facebook developers console, enter the host name of public IP in return URI.
  2. Change the path of client.json and fbsecret.json
  3. $ sudo nano /etc/apache2/sites-available/<app_name>.conf
  4. paste the following,
     ServerAlias <HOST_NAME>
  5. sudo a2ensite <VIRTUAL_HOST_NAME>


##Configre firewall
  1. $sudo apt-get install fail2ban
  2. $sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
  3. $sudo nano /etc/fail2ban/jail.local
  4. After making required changes, 
     $sudo service fail2ban start

##References
https://postgresapp.com/documentation/configuration-python.html
https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04
http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
