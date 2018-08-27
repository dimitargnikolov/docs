# Install

1. Download generic Linux tarball, untar, define MYSQL_HOME, update PATH.
2. Run
```
$ mysqld --initialize --basedir=$MYSQL_HOME --datadir=$MYSQL_HOME/data
```

# Starting
```
$ mysqld_safe --basedir=$MYSQL_HOME --datadir=$MYSQL_HOME/data
```

# Stopping
```
```

# Useful Commands

1. Change root password
```mysql
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass';
```
2. Create database and grant privileges
```mysql
mysql> CREATE DATABASE dbname;
```
3. Create new user
```mysql
mysql> CREATE USER 'finley'@'localhost' IDENTIFIED BY 'password';
mysql> GRANT ALL PRIVILEGES ON dbname.* TO 'finley'@'localhost' WITH GRANT OPTION;
mysql> CREATE USER 'finley'@'%' IDENTIFIED BY 'password';
mysql> GRANT ALL PRIVILEGES ON dbname.* TO 'finley'@'%' WITH GRANT OPTION;
```
4. Show user info
```mysql
SELECT User, Host, authentication_string FROM mysql.user;
SHOW GRANTS FOR finley@localhost;
```
