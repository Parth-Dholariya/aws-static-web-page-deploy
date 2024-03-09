# Introduction
This is a static ecommerce web page

Here's how to deploy it on AWS-Linux
## Deploy Pre-Requisites

1. Install FirewallD

```
sudo yum install firewalld -y
sudo systemctl start firewalld
sudo systemctl enable firewalld
sudo systemctl status firewalld
```

## Deploy and Configure Database

1. Install MariaDB

```
sudo yum list mariadb*
sudo yum install mariadb105-server
sudo vi /etc/my.cnf
sudo systemctl start mariadb
sudo systemctl enable mariadb
```

2. Configure firewall for Database

```
sudo firewall-cmd --permanent --zone=public --add-port=3306/tcp
sudo firewall-cmd --reload
```

3. Configure Database

```
$ mysql
MariaDB > CREATE DATABASE ecomdb;
MariaDB > CREATE USER 'ecomuser'@'localhost' IDENTIFIED BY 'ecompassword';
MariaDB > GRANT ALL PRIVILEGES ON *.* TO 'ecomuser'@'localhost';
MariaDB > FLUSH PRIVILEGES;
```

> ON a multi-node setup remember to provide the IP address of the web server here: `'ecomuser'@'web-server-ip'`

Create the db-load-script.sql

```
cat > db-load-script.sql <<-EOF
USE ecomdb;
CREATE TABLE products (id mediumint(8) unsigned NOT NULL auto_increment,Name varchar(255) default NULL,Price varchar(255) default NULL, ImageUrl varchar(255) default NULL,PRIMARY KEY (id)) AUTO_INCREMENT=1;

INSERT INTO products (Name,Price,ImageUrl) VALUES ("Laptop","100","c-1.png"),("Drone","200","c-2.png"),("VR","300","c-3.png"),("Tablet","50","c-5.png"),("Watch","90","c-6.png"),("Phone Covers","20","c-7.png"),("Phone","80","c-8.png"),("Laptop","150","c-4.png");

EOF
```

Run sql script

```
sudo mysql < db-load-script.sql
```

## Update
```
sudo yum update
```

## Deploy and Configure Web

1. Install required packages

```
sudo yum install -y httpd php php-mysqlnd
sudo firewall-cmd --permanent --zone=public --add-port=80/tcp
sudo firewall-cmd --reload
```

2. Configure httpd

Change `DirectoryIndex index.html` to `DirectoryIndex index.php` to make the php page the default page

```
sudo sed -i 's/index.html/index.php/g' /etc/httpd/conf/httpd.conf
```

3. Start httpd

```
sudo systemctl start httpd
sudo systemctl enable httpd
```

4. Download code

```
sudo yum install -y git
sudo git clone https://github.com/Parth-Dholariya/aws-static-web-page-deploy.git /var/www/html/
```

5. Update index.php

Update [index.php](https://github.com/Parth-Dholariya/aws-static-web-page-deploy/blob/main/index.php) file to connect to the right database server. In this case `localhost` since the database is on the same server.

```
sudo sed -i 's/172.20.1.101/localhost/g' /var/www/html/index.php

              <?php
                        $link = mysqli_connect('172.20.1.101', 'ecomuser', 'ecompassword', 'ecomdb');
                        if ($link) {
                        $res = mysqli_query($link, "select * from products;");
                        while ($row = mysqli_fetch_assoc($res)) { ?>
```

> ON a multi-node setup remember to provide the IP address of the database server here.

```
sudo sed -i 's/172.20.1.101/localhost/g' /var/www/html/index.php
```

6. Test

```
curl http://localhost
```

7. Put your public-ip of aws-linux with port number on the browser
