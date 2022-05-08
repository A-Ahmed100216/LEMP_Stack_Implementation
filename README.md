# LEMP Stack Implementation

A LEMP Stack is very similar to a LAMP Stack, they key difference being the webserver - NGINX

## 1. Installing NGINX
* NGINX is a high performance web server. It is also installed using the apt - the package manager for ubuntu
```bash
sudo apt update
sudo apt install nginx
```
* To confirm the service is running:
```
sudo systemctl status nginx
```
![NGINX Status](/images/nginx_status.png)
* Check Security Group settings to ensure port 80 is opened to allow access to the interent. Once this is done, we can run the following curl commands to confirm access:
```bash
curl http://localhost:80
```
![curl localhost](/images/curl_localhost_dns.png)

```bash 
curl http://127.0.0.1:80
```
![curl localhost](/images/curl_localhost.png)
* Similarly, navigating to the webpage (public IP) should render the following output.  
![nginx webpage](/images/nginx_default_webpage.png)

## 2. Installing MySQL
* Install MySQL
```
sudo apt install mysql-server
```
* The next step is to run the secure installation however this faces the same [blocker](https://github.com/A-Ahmed100216/LAMP_Stack_Implementation/blob/main/Project1.md#blocker) as Project 1 therefore the same solution was implemented.
![sql](/images/sql.png)
![sql secure installation](/images/sql_secure_installation.png)


## 3. Installing PHP 

The following php packages must be installed:
1. `php-fpm` PHP fastCGI process manager. NGINX requires this to process PHP. 
2. `php-mysql` Communicate with MySQL
```
sudo apt install php-fpm php-mysql
```
![install php](/images/install_php.png)

## 4. Configuring NGINX to use PHP Processor 
* NGINX uses server blocks to encapsulate config details (similar to virtual hosts in Apache)
* The default server block serves documents located in /var/www/html. For multiple sites we create a directory structure within /var/www 
* This leaves /var/www/html in case the client request does not match any other site.
* Create a root web directory for your domain:
```
sudo mkdir /var/www/projectLEMP
```
* Assign ownership of directory to current user
```
sudo chown -R $USER:$USER /var/www/projectLEMP
```
![configure nginx](/images/create_domain.png)

* Create a new config file in NGINX's sites-available directory with a basic config:
```bash
sudo nano /etc/nginx/sites-available/projectLEMP

# Paste in file 

#/etc/nginx/sites-available/projectLEMP
server {
    # What port nginx will listen to (port 80 is default for http)
    listen 80; 
    
    # Domain names and/or IP addresses server block should respond to
    server_name projectLEMP www.projectLEMP;

    # Document root where files served by this website are stored
    root /var/www/projectLEMP;

    # Order Nginx will prioritize index files for this website (index.html usually prioritised over index.php)
    index index.html index.htm index.php;

    # Checks for the existence of files/directories matching a URL request, otherwise returns 404.
    location / {
        try_files $uri $uri/ =404;
    }

    # handles PHP processing - points Nginx to the fastcgi-php.conf 
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
     }

    # NGINX does not process .htaccess files therefore deny all prevents them from being served to visitors. 
    location ~ /\.ht {
        deny all;
    }

}
```
* The configuration can be activated by linking to NGINX's sites-enabled directory. This will ensure NGINX uses this configuration next time it is reloaded. 
```
sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
```
* Check config for syntax errors:
```
sudo nginx -t
```
![nginx conf](/images/nginx_conf.png)

* Disable the default NGINX host 
```
sudo unlink /etc/nginx/sites-enabled/default
```
* Reload nginx
```
sudo systemctl reload nginx
```
* The website is now active but there is no content. This can be achieved by creating an index.html in /var/www/projectLEMP/
```
sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html
```
![simple webpage](/images/simple_webpage.png)

## Testing PHP with NGINX
* The LEMP stack is now completely installed and operational. We now need to test whether NGINX can send .php file to the PHP processor.

* Create a test php file
```bash
sudo nano /var/www/projectLEMP/info.php
# paste in file 
<?php
phpinfo();
```
* This should render the following output 
![php webpage](/images/php_webpage.png)

* Once complete, remove the file as it contains sensitive information about your server. 
```bash  
sudo rm /var/www/projectLEMP/info.php
```
## Retrieving data from MySQL database with PHP
* Create a database which NGINX will query and display. 
* Connect to MySQL 
```
sudo mySQL -u root -p 
```
* Create a new database;
```sql
CREATE DATABASE `example_database`;
```
* MySQL PHP library mysqlnd doesn’t support caching_sha2_authentication. Create a new user  to connect to the MySQL database from PHP.
```sql
CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
```
* Grant the user permissions on the example database
```sql
GRANT ALL ON example_database.* TO 'example_user'@'%';
```
![create database](/images/create_database.png)

* Exit and log in as the new user
```
mysql -u example_user -p
```
* Once you've logged in, confirm the user has access to the test database.
```sql
SHOW DATABASES;
```
![show databases](/images/Show_databases.png)
* Create a test table and insert a few values:
```sql 
CREATE TABLE example_database.todo_list (
    item_id INT AUTO_INCREMENT,
    content VARCHAR(255),
    PRIMARY KEY(item_id)
);

INSERT INTO example_database.todo_list (content) VALUES ("My first important item");
SELECT * FROM example_database.todo_list;
```
![to do list table](/images/todo_list_add.png)

* Create a php script to connect to MySQL
```bash
nano /var/www/projectLEMP/todo_list.php
```
* Copy below code into the file
```php
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
![final webpage](/images/final_webpage.png)
