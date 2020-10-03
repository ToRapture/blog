---
title: Install and Config MySQL 8 on Ubuntu
date: 2020-06-10 12:11:00
tags:
- MySQL
- Database
---

# Remove Completely and Install
```bash
sudo apt-get remove --purge "mysql*"
sudo apt-get purge "mysql*"
sudo apt-get autoremove
sudo apt-get autoclean
sudo apt-get remove dbconfig-mysql
sudo apt-get dist-upgrade
sudo rm -rf /etc/mysql /var/lib/mysql
sudo apt-get update && sudo apt-get install mysql-server
```

# Enable Remote Access
```bash
su root
mysql -uroot -e " \
    ALTER USER 'root'@'localhost' IDENTIFIED WITH caching_sha2_password BY 'yourpasswd'; \
    CREATE USER 'root'@'%' IDENTIFIED BY 'yourpasswd'; \
    GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'; \
    FLUSH PRIVILEGES;"
```
In `/etc/mysql/mysql.conf.d/mysqld.cnf`:
* replace `bind-address = 127.0.0.1` with `bind-address = 0.0.0.0`

Then restart mysql by `sudo service mysql restart`.