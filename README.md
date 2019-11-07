# Udacity - Linux Server Administration Project

Udacity's Full Stack Web Developer III Nanodegree Project. *[Link to II Project](https://github.com/ConanCode96/Items-Catalog)

## Projcet Overview

This tutorial will guide you through the steps to take a baseline installation of a Linux server and prepare it to host your Web applications. You will then secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing Flask-based Web applications onto it.

The hosted web application is written on Python framework Flask, PostgreSQL is used as a data storage, and Apache2 with mod_wsgi as the webserver of choice.

In this project, I used the following:

- **Amazon AWS LightSail Platform**
- **Ubuntu 18.04 LTS Box**
- **SSH/HTTP/NTP protocols**
- **Linux Bash Shell**
- **Vagrant & VirtualBoxes**

### Technical Information About the Server

- **Server (Static) IP Address:** ~3.124.92.8~ (Server is now down)
- **(non-default) SSH port:** 2200
- **SSH tunnel username:** grader
- **SSH tunnel password:** ~~N/A(disabled feature!)~~
- **Application URL:** ~http://item.catalog.3.124.92.8.xip.io/~


**Note to reader: `(#) denotes SuperUser (sudo)`**

## Steps to reproduce my Server 

### 1. Create the RSA Key Pair

Produce a public/private keypair using the following command

   ```console
   $ ssh-keygen
   ```
Keep the private key on you local machine, add the public key to the user directory file `/home/${USER}/.ssh/` authorize_keys you wish to ssh into on the server.

### 2. Create Amazon AWS LightHouse instance

1. Log in or create an account on [Amazon LightSail](https://lightsail.aws.amazon.com).

2. Go to the Dashboard, and click **Create Instance**.

3. Choose **Ubuntu 18.04 x64** image from the list of given boxes.

4. Choose a preferred size. In this project, I have chosen the **512MB RAM/1 vCPU/25GB SSD** configuration.

5. Amazon LightSail gives you the ability to drop your previously generated `SSH private key` rather than copying it manually to your preferred user directory.

6. Click **Create** to create the instance. This will take about 2 minutes to complete. After the droplet has been created successfully, a public IP address will be assigned. 
You can also choose to make the IP static to avoid dynamic IP assignment in case you restart your instance.


### 3 Updating the Server

Run the following command to update the virtual server:


```
 # apt-get update && apt-get upgrade
```

This will update all the packages. If the available update is a kernel update, you might need to reboot the server by running the following command:

```
 # reboot
```

### 

### 4. Set SSH port to 2200 and disabling password authentication:

1. 
   ```
   # nano /etc/ssh/sshd_config
   ```

2. Locate the line `#Port 22` and change it to `Port 2200`, and save the file.

3. Locate the line `#PasswordAuthentication yes` and change it to `PasswordAuthentication no`, and save the file.

4. Restart the SSH server to reflect those changes:
   ```
   # service ssh restart
   ```

5. `reboot` and `ssh` again to confirm changes are in effect.

### 5. Configure Timezone to Universal UTC

To configure the timezone to use UTC, run the following command:

```
    # unlink /etc/localtime
    # ln -s /usr/share/zoneinfo/UTC /etc/localtime
```

### 6. Setting Up the Firewall

We mostly should follow the rule of least privileges, so we will configure the firewall to allow only incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123):

```
# ufw allow 2200/tcp
# ufw allow 80/tcp
# ufw allow 123/udp
```

To up-run/enable the above firewall rules, run:

(**CAUTION!** : This step is DANGEROUS, You could lock yourself out of the system, make sure this step won't affect your access to the server)

```
# ufw enable
```

To confirm the status of the firewall, run:

```
# ufw status
```

You should see something like this:

```
Status: active

To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
123/udp                    ALLOW       Anywhere
2200/tcp (v6)              ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
123/udp (v6)               ALLOW       Anywhere (v6)
```

### 7. Create the Reviewer User `grader` and Add it to the `sudoers` Group

Run The following commands in sequence:

```
  # adduser grader
  cd /etc/sudoers.d/
  # echo 'grader ALL=(ALL) NOPASSWD:ALL' >> grader
  # chmod 440 grader
```

### 8. Adding SSH Access for the user `grader`

First log into the account of the user `grader` from your virtual server:

```
# sudo -s -u grader
```

Now you should be at `grader`'s home directory, to make sure you are run 
```
 pwd 
```
it should output `/home/grader/`
 
Now enter the following commands to allow SSH access to the user `grader`:

```
$ mkdir .ssh
$ chmod 700 .ssh
$ cd .ssh/
$ touch authorized_keys
$ chmod 644 authorized_keys
```

Now go back to your local machine and copy the content of the public key file, paste them in `authorized_keys` file using `nano` or any other text editor, and save. (make sure you're logged in as `grader`)

After that, run `exit`. You would now be back to your local machine. To confirm that it worked, run the following command in your local machine:

```console
hossam@vagrant:~$ ssh grader@3.124.92.8 -p 2200 -i grader-pairs
```

You should now be able to log in as `grader` with no passwords required

### 9. Disabling Root Login / N

1. Open the file `/etc/ssh/sshd_config` with `nano`:
   ```
   # nano /etc/ssh/sshd_config
   ```
   
2. Locate the line `PermitRootLogin yes` and change it to `PermitRootLogin no`.

3. Restart SSH and exit

   ```
   # service ssh restart
   # exit
   ```

4. Try to log in as `root`, you should get an error:

   ```console
    root@3.124.92.8: Permission denied (publickey).
   ```

### 10. Installing Required Packages

`Note`: (Python 3.6 & Git are installed by default on the instance)

```
# sudo apt update
# apt-get install apache2
# apt-get install python3-pip
# apt-get install postgresql python-psycopg2 postgresql-plpython python-dev libpq-dev python-flask python-sqlalchemy
# pip3 install flask packaging oauth2client redis passlib flask-httpauth sqlalchemy flask-sqlalchemy psycopg2-binary bleach requests
```

### 11. Configuring PostgreSQL

1. Log in as the user `postgres` that was automatically created during the installation of PostgreSQL Server:

   ```
   # sudo -s -u postgres
   ```

2. Open the `psql` shell:

   ```
   $ psql
   ```

3. This will open the `psql` shell. Now type the following commands one-by-one:

   ```sql
   postgres=# CREATE DATABASE catalog;
   postgres=# CREATE USER `hossam`;
   postgres=# ALTER ROLE `hossam` WITH PASSWORD 'yourpassword';
   postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO `hossam`;
   ```

   Then exit from the terminal by typing `\q` followed by a line feed;

### 12. Setting Up Apache to Run the Flask Application

#### 12.1. Installing `mod_wsgi`

The module `mod_wsgi` will allow your Python applications to run from Apache server. 

If you are running Python 2, install it by running the following command:

```
# apt install libapache2-mod-wsgi-py3
# service apache2 restart
```

#### 12.2. Cloning the Item-Catalog Flask application

1.
   ```
   $ cd /var/www/
   # mkdir FlaskApp
   $ cd FlaskApp/
   ```

2. Clone your GitHub repository of your _Item Catalog application project_ (Flask project) rename it to `FlaskApp`.

   ```
   # git clone https://github.com/ConanCode96/Items-Catalog FlaskApp
   $ cd FlaskApp/
   ```

#### 12.3. Setting Up the VirtualHost Configuration

1. Run the following command in terminal to set up a file called `FlaskApp.conf` to configure the virtual hosts:

   ```
   # nano /etc/apache2/sites-available/FlaskApp.conf
   ```

2. Add the following lines to it:

   ```

   <VirtualHost *:80>
      ServerName 3.124.92.8
      ServerAlias item.catalog.3.124.92.8.xip.io
      ServerAdmin hossamf.doma@gmail.com
      WSGIScriptAlias / /var/www/FlaskApp/FlaskApp/flaskapp.wsgi
      <Directory /var/www/FlaskApp/FlaskApp/>
          Require all granted
      </Directory>
      Alias /static /var/www/FlaskApp/FlaskApp/static
      <Directory /var/www/FlaskApp/FlaskApp/static/>
          Require all granted
      </Directory>
      ErrorLog ${APACHE_LOG_DIR}/error.log
      LogLevel warn
      CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>

   ```
   
3. Disable default Apache page & Enable the new virtual host then Restart the server:

   ```
   # a2dissite 000-default.conf
   # a2ensite FlaskApp.conf
   # service apache2 restart
   ```

4. Create the Web Server Gateway Interface(WSGI) File

   Apache uses the `.wsgi` file to serve the Flask app

   ```
   $ cd /var/www/FlaskApp/FlaskApp/
   # nano flaskapp.wsgi
   ```

   Add the following lines to the `flaskapp.wsgi` file:

   ```python
   import sys
   import logging
   logging.basicConfig(stream=sys.stderr)
   sys.path.insert(0, "/var/www/FlaskApp/FlaskApp/")

   from application import app as application
   ```
   
   In the above code, replace `application` with the name of the main module. In my case it was `catalogApp`
   
5. Restart Apache server:

   ```
   # service apache2 restart
   ```
   
   Now you should be able to run the application at <http://item.catalog.3.124.92.8.xip.io/>.

## References

[1] <https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps>

[2] <https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04>

[3] https://www.ntu.edu.sg/home/ehchua/programming/sql/PostgreSQL_GetStarted.html

```
......

*And Almost the whole World Wide Web XD ")))
Been searching for days, LITERALLY!
Learned a lot in the process ^_^
```
