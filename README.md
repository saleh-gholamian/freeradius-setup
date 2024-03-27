# freeradius-setup
Freeradius installation guide on Ubuntu server


**#0 Update the ubuntu server**
```bash
apt update && apt upgrade -y
```


**#1 Installation requirements**
```bash
sudo apt install php apache2 freeradius libapache2-mod-php mariadb-server freeradius-mysql freeradius-utils php-{gd,common,mail,mail-mime,mysql,pear,db,mbstring,xml,curl} wget unzip zip -y
```



**#2 Start apache2 and enable FreeRadius**
```bash
sudo systemctl enable --now apache2 && sudo systemctl enable freeradius
```



**#3 Setup the sql server (mariadb server)**
```bash
sudo mysql_secure_installation
```
That's all the questions you need to answer, you can set a root password, but it doesn't matter to me.

1. _Enter current password for root (enter for none):_  **enter**
2. _Switch to unix_socket authentication [Y/n]_  **n**
3. _Change the root password? [Y/n]_  **n**
4. _Remove anonymous users? [Y/n]_  **y**
5. _Disallow root login remotely? [Y/n]_  **y**
6. _Remove test database and access to it? [Y/n]_  **y**
7. _Reload privilege tables now? [Y/n]_  **y**

**#4 Setting up mariadb**Setting up mariadb
```bash
sudo mysql -u root -p
```

Now that you're logged in, you need to do a few things. 

1. Create the databsae
2. Create a user for FreeRadius
3. Give the user permissions

> We can create the database by running this “CREATE DATABASE” command:
> ```bash
> CREATE DATABASE radius;
> ```
> 
> Next we need to create the user account. I would recommend changing PASSWORD to a secure password.
> ```bash
> CREATE USER 'radius'@'localhost' IDENTIFIED by 'PASSWORD';
> ```
> 
> To grant the privileges, run this command:
> ```bash
> GRANT ALL PRIVILEGES ON radius.* TO 'radius'@'localhost';
> ```
> 
> Finally, run these commands to reload the privileges in the sql database and to quit the session.
> ```bash
> FLUSH PRIVILEGES;
> quit;
> ```


**#5 Setting up radius to use mysql **
If you're logged in as root, the _**sudo su -**_ command isn't necessary, and also probably not need to _**exit**_, but it's okay if you run them.
```bash
sudo su -
```
```bash
mysql -u root -p radius < /etc/freeradius/3.0/mods-config/sql/main/mysql/schema.sql
```
```bash
exit
```
```bash
sudo ln -s /etc/freeradius/3.0/mods-available/sql /etc/freeradius/3.0/mods-enabled/
```



**#6 Configure free radius sql file**
Open the sql config file with an editor like nano
```bash
sudo nano /etc/freeradius/3.0/mods-enabled/sql
```

Find the following lines in the file and change them exactly to the specified values:

> - **driver = "rlm_sql_null"** needs to be **driver = "rlm_sql_${dialect}"**
> - **dialect = "sqlite"** needs to be **dialect = "mysql"**
> - **read_clients = yes** needs to be uncommented (No # in front of that line)
> - **client_table = “nas”** needs to be uncommented (No # in front of that line)

You should also find the mysql block and comment the entire tls block inside it with # (must be done as follows):

- **_Before:_**

> ```bash
> mysql {
> 	# If any of the files below are set, TLS encryption is enabled
> 	tls {
> 		ca_file = "/etc/ssl/certs/my_ca.crt"
> 		ca_path = "/etc/ssl/certs/"
> 		certificate_file = "/etc/ssl/certs/private/client.crt"
> 		private_key_file = "/etc/ssl/certs/private/client.key"
> 		cipher = "DHE-RSA-AES256-SHA:AES128-SHA"
> 
> 		tls_required = yes
> 		tls_check_cert = no
> 		tls_check_cert_cn = no
> 	}
> 
> 	# If yes, (or auto and libmysqlclient reports warnings are
> 	# available), will retrieve and log additional warnings from
> 	# the server if an error has occured. Defaults to 'auto'
> 	warnings = auto
> }
> ```

- _Affter:_

> ```bash
> mysql {
> 	# If any of the files below are set, TLS encryption is enabled
> #		tls {
> #			ca_file = "/etc/ssl/certs/my_ca.crt"
> #			ca_path = "/etc/ssl/certs/"
> #			certificate_file = "/etc/ssl/certs/private/client.crt"
> #			private_key_file = "/etc/ssl/certs/private/client.key"
> #			cipher = "DHE-RSA-AES256-SHA:AES128-SHA"
> 
> #			tls_required = yes
> #			tls_check_cert = no
> #			tls_check_cert_cn = no
> #		}
> 
> 	# If yes, (or auto and libmysqlclient reports warnings are
> 	# available), will retrieve and log additional warnings from
> 	# the server if an error has occured. Defaults to 'auto'
> 	warnings = auto
> }
> ```

Finally, you need to find and uncomment the Connection info and finally change its values as below

- **_Before:_**

> ```bash
> #	server = "localhost"
> #	port = 3306
> #	login = "radius"
> #	password = "radpass"
> ```

- **_Affter:_**

> ```bash
> 	server = "localhost"
> 	port = 3306
> 	login = "radius"
> 	password = "PASSWORD"
> ```

```bash

```

**#**
```bash

```



**#**
```bash

```



**#**
```bash

```



**#**
```bash

```



**#**
```bash

```



**#**
```bash

```
