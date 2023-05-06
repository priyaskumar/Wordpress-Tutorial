_Steps to setup the requirements to create a website using wordpress_
==

## _**Step 1**_ : _Install LAMP stack on Ubuntu_

i) _Create a non root sudo user in ubuntu:_

    adduser username
    usermod -aG sudo username
    su - username (logins as username)
    sudo whoami (to check)

To login as non root sudo user :

    su - username

ii) _Install UFW (Uncomplicated Firewall) :_

    sudo apt install ufw

iii) _Setting-up firewall :_

1. _Enable firewall_ : 

    ```
    sudo ufw enable
    ```
    loads the firewall and enables it to start on boot
    

2. _Modify default UFW firewall polices if required_ :

    ```
    sudo ufw default allow outgoing
   
    sudo ufw default deny incoming
    ```
    By default, the UFW firewall denies every incoming connections and only allow all outbound connections to server.    
       
    The default UFW firewall polices are placed in the `/etc/default/ufw` file   

3. _Allow all SSH connections_ :

    UFW would block all incoming connections.   
    
    If you are connected to your server over SSH from a remote location, you will no longer able to connect it again.

    So enable SSH connections to your server.

    ```
    sudo ufw allow ssh
    ```

    _**NOTE**_ : 
    - To see the list of application profiles available on your server :

        ```
        sudo ufw app list
        ```

    - To get more information about a particular profile and defined rules :

        ```
        sudo ufw app info 'Apache'
        ```

    - To check the status of firewall :  

        ```
        sudo ufw status 
        sudo ufw status numbered
        ```
    - To disable the firewall

        ```
        sudo ufw disable
        ```
    - To reset the firewall rules

        ```
        sudo ufw reset
        ```

iv) _Update the package manager cache_: 

    
    sudo apt update -y
    

- Updates the package sources list to get the latest list of available packages in the repositories.

- That is, by entering “apt update” command, you're asking your Linux machine to go browse the package sources lists and copy the latest version of them and put them in your local machine.

v) _Upgrade the packages_ : 

    sudo apt upgrade -y

- Updates all the packages presently installed in your Linux system to their latest versions.

- That is, by entering the “apt upgrade” command you tell your Linux machine to compare the versions of all the presently installed packages with the ones in the list you've just downloaded through “apt update” and upgrade the packages which are lagging in terms of version numbers.

- _**NOTE**_ :   
-y flag indicates automatic yes to prompts.   
Assumes "yes" as answer to all prompts and runs non-interactively.

vi) _Install apache (web server)_ :

    sudo apt install apache2

vii) _Adjust firewall settings to allow HTTP traffic_:

    sudo ufw allow in "Apache"

- Allows traffic on port 80.

- _**NOTE**_ : _To find your Server’s Public IP address_

    ```
    ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'
    ```
    or
    ```
    curl http://icanhazip.com
    ```

viii) _Install MySQL_ :

    sudo apt install mysql-server

ix) _Run a security script that comes pre-installed with MySQL_ :

    sudo mysql_secure_installation

- This script will remove some insecure default settings and lock down access to your database system.

- Your server will ask you to select and confirm a password for the MySQL root user.

- Setup the password and answer the following prompts with 'Y' or 'N':

    - remove some anonymous users and the test database -> Enter'Y'

    - disable remote root logins -> Enter 'N'

    - load these new rules so that MySQL immediately respects the changes you have made -> Enter'Y'  

x) _Install php_ :

    sudo apt install php libapache2-mod-php php-mysql

To Confirm the php version

    php -v

xi) _Create a virtual host for the website_ :

1. _Create the directory for your domain_ :

    ```
    sudo mkdir /var/www/your_domain
    ```

2. _Assign ownership of the directory_ :

    ```
    sudo chown -R $USER:$USER /var/www/your_domain
    ```
3. _Open a new configuration file in Apache’s sites-available directory_ :

    ```
    sudo nano /etc/apache2/sites-available/your_domain.conf
    ```
    Opens the configuration file in the 'nano' command-line editor

    Paste the following into the configuration file.

    ```
    <VirtualHost *:80>
        ServerName your_domain
        ServerAlias www.your_domain 
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/your_domain
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
    ```
    Save and close the file - `CTRL+X` -> `Y` -> `ENTER`

4. _Use a2ensite to enable the new virtual host_:

    ```
    sudo a2ensite your_domain
    ```
5. _Disable the default website that comes installed with Apache_ :

    This is required if you’re not using a custom domain name.
    In this case Apache’s default configuration would overwrite your virtual host. 

    ```
    sudo a2dissite 000-default
    ```
6. _Make sure your configuration file doesn’t contain syntax errors_ :

    ```
    sudo apache2ctl configtest
    ```
7. _Reload Apache so these changes take effect_ :

    ```
    sudo systemctl reload apache2
    ```
8. _Test if virtual host works as expected_ :

    Create index.html in the same directory

    ```
    nano /var/www/your_domain/index.html
    ```
    Include the following contents

    ```
    <html>
    <head>
        <title>your_domain website</title>
    </head>
    <body>
        <h1>Hello World!</h1>

        <p>This is the landing page of <strong>your_domain</strong>.</p>
    </body>
    </html>
    ```

    Check your website : http://server_domain_or_IP

9. _Modify the precedence of files in your directory_ :

    index.php should take precedence over index.html 

    ```
    sudo nano /etc/apache2/mods-enabled/dir.conf
    ```
    Add these contents
    ```
    <IfModule mod_dir.c>
        DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
    </IfModule>
    ```

    Save and reload Apache

xii) _Test PHP Processing on your Web Server_ :

1. _Create a new file named info.php_ :

    ```
    nano /var/www/your_domain/info.php
    ```
    
2. _Add the contents_ :

    ```
    <?php
    phpinfo();
    ```
    
3. _Check the website_ : http://server_domain_or_IP/info.php  
If it displayes the php version, then the php installation was successful!

4. _Delete the info.php file_:

    ```
    sudo rm /var/www/your_domain/info.php
    ```
    
xiii) _Test database connection from php_ :

1. _Connect to the MySQL console using root account_ : 

    ```
    sudo mysql
    ```
    
2. _Create a test database_ :

    ```
    CREATE DATABASE example_database;
    ```
    
3. _Create a new MySQL user using mysql native password as default authentication method_ :

    ```
    CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
    ```
    

4. _Give permission to the new sql user over the database_ :

    ```
    GRANT ALL ON example_database.* TO 'example_user'@'%';
    ```
    

5. _Exit the MySQL shell_ :
    
    ```
    exit
    ```
    

6. _Test if the new user has the proper permissions by logging in to the MySQL console again with custom user credentials_ :

    ```
    mysql -u example_user -p
    SHOW DATABASES;
    ```
    
7. _Create a test table with dummy data_ : 

    ```
    CREATE TABLE example_database.todo_list (
        item_id INT AUTO_INCREMENT,
        content VARCHAR(255),
        PRIMARY KEY(item_id)
    );

    INSERT INTO example_database.todo_list (content) VALUES ("My first important item");

    INSERT INTO example_database.todo_list (content) VALUES ("My second important item");
    ```

8. _Confirm if the data was saved successfully and exit_ :

    ```
    SELECT * FROM example_database.todo_list;
    
    exit
    ```

9. _Create a PHP script to add queries for the database_ :

    ```
    nano /var/www/your_domain/todo_list.php
    ```
    
10. _Add the following contents to the php script_ : 
    
    ```
    <?php
    $user = "example_user";
    $password = "password";
    $database = "example_database";
    $table = "todo_list";

    try {
    $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
    echo "<h2>TODO</h2><ol>"; 
    foreach($db->query("SELECT content FROM $table") as $row) {
        echo "<li>" . $row['content'] . "</li>";
    }
    echo "</ol>";
    } catch (PDOException $e) {
        print "Error!: " . $e->getMessage() . "<br/>";
        die();
    }
    ```
    
11. _Save and close the file_

12. _Check the website_ : http://your_domain_or_IP/todo_list.php



## _**Step 2**_ : _Secure the website with Let's Encrypt on Ubuntu_

Let’s Encrypt is a Certificate Authority (CA) that facilitates obtaining and installing free TLS/SSL certificates, thereby enabling encrypted HTTPS on web servers.    
  
Certbot, a software client provided by Let's Encrypt automates the entire process of obtaining and installing a certificate on both Apache and Nginx.

You'll use Certbot to obtain a free SSL certificate for Apache on Ubuntu 20.04, and make sure this certificate is set up to renew automatically.

i) _Install Certbot_ :

```
sudo apt install certbot python3-certbot-apache
```
ii) _Check your Apache Virtual Host Configuration_ :

```
sudo nano /etc/apache2/sites-available/your_domain.conf
```
Check the contents.

```
...
ServerName your_domain
ServerAlias www.your_domain
...
```

Validate your changes

```
sudo apache2ctl configtest
```

Reload Apache

iii) _Allow HTTPS through Firewall_ :

Adjust the UFW settings to allow HTTPS traffic. 

```
sudo ufw allow 'Apache Full'
sudo ufw delete allow 'Apache'
```
iv) _Obtain a SSL Certificate_ :

```
sudo certbot --apache
```
Answer the prompts accordingly after running the above command

v) _Verify Certbot Auto-Renewal_ :

```
sudo systemctl status certbot.timer
sudo certbot renew --dry-run
```

## _**Step 3**_ : _Install WordPress on Ubuntu_

i) _Create a MySQL Database and User for WordPress_ :

1. _Login to the MySQL root account_ :

    ```
    mysql -u root -p
    ```
2. _Create a Database_ :



    ```
    CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
    ```
3. _Create a seperate MySQL user account exclusively for the new database_ :

    ```
    CREATE USER 'wordpressuser'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
    ```
4. _Give all access to the new user_ :

    ```
    GRANT ALL ON wordpress.* TO 'wordpressuser'@'%';
    ```
5. _Flush the privileges_ :

    ```
    FLUSH PRIVILEGES;
    ```
6. _Exit out of MySQL_

ii) _Install additional PHP Extensions_:

```
sudo apt update
sudo apt install php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip
sudo systemctl restart apache2
```

iii) _Adjust Apache's Configuration to allow .htaccess Overrides and Rewrites

1. _Open the Apache configuration file for your website_ :

    ```
    sudo nano /etc/apache2/sites-available/your_domain.conf
    ```

2. _Update the conf file_ :

    ```
    <Directory /var/www/your_domain/>
        AllowOverride All
    </Directory>
    ```
3. _Enable Rewrite module_ :

    ```
    sudo a2enmod rewrite
    ```

4. _Enable the changes_ :

    ```
    sudo apache2ctl configtest
    sudo systemctl restart apache2
    ```

iv) _Download WordPress_ :

1. _Change into a writable directory and download the compressed release_ :

    ```
    cd /tmp
    curl -O https://wordpress.org/latest.tar.gz
    ```
2. _Extract the compressed file to create the WordPress directory structure_ :

    ```
    tar xzvf latest.tar.gz
    ```
3. _Add a dummy .htaccess file so that this will be available for WordPress to use later_ :

    ```
    touch /tmp/wordpress/.htaccess
    ```
4. _Copy over the sample configuration file to the filename that WordPress reads_ :

    ```
    cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php
    ```
5. _Create the upgrade directory, so that WordPress won’t run into permissions issues when trying to do this on its own following an update to its software_ :

    ```
    mkdir /tmp/wordpress/wp-content/upgrade
    ```
6. _Copy the entire contents of the directory into your document root_ :

    ```
    sudo cp -a /tmp/wordpress/. /var/www/wordpress
    ```
v) _Configure the WordPress Directory_ :

1. _Adjust the Ownership and Permissions_ :

    ```
    sudo chown -R www-data:www-data /var/www/wordpress
    sudo find /var/www/wordpress/ -type d -exec chmod 750 {} \;
    sudo find /var/www/wordpress/ -type f -exec chmod 640 {} \;
    ```

2. _Set up the wordPress Configuration File_ :

    ```
    curl -s https://api.wordpress.org/secret-key/1.1/salt/
    ```

    Copy the output. 
    Open the WordPress configuration file
    
    ```
    sudo nano /var/www/wordpress/wp-config.php
    ```
    Add the copied contents in the wordpress Configuration file.
    Change the MySQL settings with your db name, db user, db password.

vi) _Complete the Installation Through the Web Interface_ :

Navigate to your website and give the required info to complete the installation process. 
