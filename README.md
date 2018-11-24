 linux-server-configuration 

1. Details specific to the server I set up 

  

The IP address is 18.184.170.29. 

  

The SSH port used is 2200. 

  

The URL to the hosted webpage is: https://ec2-18-184-170-29.eu-central-1.compute.amazonaws.com 

 

2.Create an instance with Amazon Lightsail 

   1.Sign in to Amazon Lightsail using an Amazon Web Services account 

   2.Follow the 'Create an instance' link 

   3.Choose the 'OS Only' and 'Ubuntu 16.04 LTS' options 

   4.Choose a payment plan 

   5.Give the instance a unique name and click 'Create' 

   6. Wait for the instance to start up 

   7.The last thing we will need to do is configure the ports Amazon Lightsail will allow. By default the firewall is set to o    only allow connects from port 22 and port 80. We need to set up port 2200 and 123 Click the networking tab  

   8. From this tab click add another under "Firewall" and choose Custom for application, TCP for protocol, and the port number under Port Range. Then click save.  

 

3. Linux Configuration steps 

Connect to the instance on a local machine 

   9.Download the instance's private key by navigating to the Amazon Lightsail 'Account page' 

   10. Click on 'Download default key' 

   11.A file called LightsailDefaultPrivateKey.pem or LightsailDefaultPrivateKey-YOUR-AWS-REGION.pem will be downloaded; open    this in a text editor 

   12. Copy the text and put it in a file called udacity.rsa in the local ~/.ssh/ directory 

   13.Run chmod 600 ~/.ssh/udacityy.rsa 

   14.Log in with the following command: ssh -i ~/.ssh/udacity.rsa ubuntu@18.184.170.29, where 18.184.170.29  is the public IP address of the instance (note that Lightsail will not allow someone to log in as root; ubuntu is the default user for Lightsail instances) 

Create a new user named grader 

   15.Run sudo adduser grader 

   16. Enter in a new UNIX password (twice) when prompted 

   17.Fill out information for the new grader user 

   18. To switch to the grader user, run su - grader, and enter the password 

Give grader user sudo permissions 

   19.Run sudo visudo 

   20.Search for a line that looks like this: 

   root ALL=(ALL:ALL) ALL 

   Add the following line below this one: 

   grader ALL=(ALL:ALL) ALL 

   21.Save and close the visudo file 

   One of the first things you should always do when configuring a Linux server is updating it's package list, upgrading the current packages, and install new updates with these three commands: 

   $ sudo apt-get update 

   $ sudo apt-get upgrade 

   $ sudo apt-get dist-upgrade 

  22. We will also install a useful tool called Finger with the command: 

  $ sudo  apt-get install finger. This tool will allow us to see the users on this server. 

Allow grader to log in to the virtual machine 

23.We will also install a useful tool called Finger with the command 

 $ sudo apt-get install finger. This tool will allow us to see the users on this server. 

24.Now we must create an SSH Key for our new user grader. From a new terminal run the command: $ ssh-keygen -f ~/.ssh/udacity.rsa 

25.In the same terminal we need to read and copy the public key using the command: 

 $ cat ~/.ssh/udacity.rsa.pub. Copy the key from the terminal. 

26.Back in the server terminal locate the folder for the user grader, it should be /home/grader. Run the command  

   $ cd /home/grader to move to the folder. 

27.Create a directory called .ssh with the command $ mkdir .ssh 

28.Create a file to store the public key with the command: 

   $ touch .ssh/authorized_keys 

29.Edit that file using  

   $ nano .ssh/authorized_keys 

30.Now paste in the public key 

31.We must change the permissions of the file and its folder by running 

   $ sudo chmod 700 /home/grader/.ssh 

   $ sudo chmod 644 /home/grader/.ssh/authorized_keys  

32.Change the owner of the .ssh directory from root to grader by using the command: 

$ sudo chown -R grader:grader /home/grader/.ssh 

33.The last thing we need to do for the SSH configuration is restart its service with 

 $ sudo service ssh restart 

Disconnect from the server 

 34.Now we need to login with the grader account using ssh. From your local terminal type $ ssh -i ~/.ssh/udacity.rsa grader@18.184.170.29 

You should now be logged into your server via SSH 

 

35.Lets enforce key authentication from the ssh configuration file by editing $ sudo nano /etc/ssh/sshd_config. Find the line that says PasswordAuthentication and change it to no. Also find the line that says Port 22 and change it to Port 2200. Lastly change PermitRootLogin to no. 

36.Restart ssh again: $ sudo service ssh restart 

37.Disconnect from the server and try step "34." again BUT also adding -p 2200 at the end this time. You should be connected. 

38.From here we need to configure the firewall using these commands: 

   $ sudo ufw allow 2200/tcp 

   $ sudo ufw allow 80/tcp 

   $ sudo ufw allow 123/udp 

   $ sudo ufw enable  

39.Running $ sudo ufw status should show all of the allowed ports with the firewall configuration. 

40.That pretty much wraps up the Linux configuration, now onto the app deployment. 

Application Deployment 

Hosting this application will require the Python virtual environment, Apache with mod_wsgi, PostgreSQL, and Git. 

   1.Start by installing the required software 

   $ sudo apt-get install apache2 

   $ sudo apt-get install libapache2-mod-wsgi python-dev 

   $ sudo apt-get install git  

2.Enable mod_wsgi with the command  

  $ sudo a2enmod wsgi  

  and restart Apache using 

  $ sudo service apache2 restart. 

3.If you input the servers IP address into a web browser you'll see the Apache2 Ubuntu Default Page 

4.We now have to create a directory for our catalog application and make the user grader the owner. 

   $ cd /var/www$ sudo mkdir catalog 

   $ sudo chown -R grader:grader catalog 

   $ cd catalog  

5.In this directory we will have our catalog.wsgi file var/www/catalog/catalog.wsgi, our virtual environment directory which we will create soon and call venv /var/www/catalog/venv, and also our application which will sit inside of another directory called catalog /var/www/catalog/catalog. 

6.First lets start by cloning our Catalog Application repository by $ git clone https://github.com/wafaamohamed123/item_catalog catalog 

7.Create the .wsgi file by 

 $ sudo nano catalog.wsgi  

and make sure your secret key matches with your project secret key then paste  

   import sys 

   import logging 

   logging.basicConfig(stream=sys.stderr) 

   sys.path.insert(0, "/var/www/catalog/") 

 

   from catalog import app as application 

   application.secret_key = 'super_secret_key'  

   8. ctrl+x save and exit 

   9.cd catalog  

10.Rename your project.py, or whatever you called it in your catalog application folder to __init__.py by $ mv item_catalog_project.py __init__.py 

11.Now lets create our virtual environment, make sure you are in /var/www/catalog. 

   $sudo apt-get install python-pip 

   $ sudo pip install virtualenv     

   $ sudo virtualenv venv     

   $ source venv/bin/activate     

   $ sudo chmod -R 777 venv  

This is what your command line should look like enter image description here 

12.While our virtual environment is activated we need to install all packages required for our Flask application. Here are some defaults but you may have more to install. 

   $ sudo pip install flask 

   $ sudo pip install httplib2 

   $ sudo pip install requests 

   $ sudo pip install --upgrade oauth2client 

   $ sudo pip install sqlalchemy 

   $ sudo apt-get install libpq-dev 

   $ pip install psycopg2 

13.Now for our application to properly run we must do some tweaking to the __init__.py file. 

14.Anywhere in the file where Python tries to open client_secrets.json must be changed to its complete path ex: /var/www/catalog/catalog/client_secrets.json enter image description here 

15.Time to configure and enable our virtual host to run the site 

   $ sudo nano /etc/apache2/sites-available/catalog.conf  

   Paste in the following: 

<VirtualHost *:80> 

 ServerName 18.184.170.29 

 ServerAlias ec2-18-184-170-29.eu-central-1.compute.amazonaws.com 

 ServerAdmin wafaaelashry95@gmail.com 

 WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog /venv/lib/python2.7/site-packages 

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

   LogLevel warn    CustomLog ${APACHE_LOG_DIR}/access.log combined 

</VirtualHost>  

If you need help finding your servers hostname go to https://whatismyipaddress.com/ip-hostname and paste the IP address. Save and quit nano 

16.Enable to virtual host:  

$ sudo a2ensite catalog.conf  

and DISABLE the default host  

$ a2dissite 000-default.conf 

 otherwise your site will not load with the hostname. 

17.The final step is setting up the database 

   $ sudo apt-get install libpq-dev python-dev 

   $ sudo apt-get install postgresql postgresql-contrib 

   $ sudo su – postgres  

   $ psql  

18.Create a database user and password 

   postgres=# CREATE USER catalog WITH PASSWORD wafaa123; 

   postgres=# ALTER USER catalog CREATEDB; 

   postgres=# CREATE DATABASE catalog with OWNER catalog; 

   postgres=# \c catalogcatalog=# REVOKE ALL ON SCHEMA public FROM public; 

   catalog=# GRANT ALL ON SCHEMA public TO catalog; 

   catalog=# \q 

   $ exit  

   Your command line should now be back to grader. 

19.Now use nano again to edit your __init__.py, database_setup.py, and lotsofmenus.py files to change the database engine from sqlite://catalog.db to postgresql://catalog:wafaa123@localhost/catalog  

20.Restart your apache server  

$ sudo service apache2 restart and now your IP address and hostname should both load your application. 

 

 

 

 

 

 

 
