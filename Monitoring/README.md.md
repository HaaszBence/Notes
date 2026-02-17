# LibreNMS | DokuWiki | NGINX | TeemIP | (on Debian)

## Optional setup

> [!IMPORTANT]
> Please take into consideration that Debian does not come by default with the "sudo" package in the following setup you should install that for optimized install time and make sure that the user executing the commands are part of the sudo group. Run the following commands with root privilages.

```bash
su -c "apt update && apt upgrade && apt install sudo && /usr/sbin/usermod -aG sudo $USER" && newgrp sudo
```

## NGINX

### Prerequisites

> Import an official NGINX signing key so apt could verify the packages authenticity.
```bash
sudo apt install curl gnupg2 ca-certificates lsb-release debian-archive-keyring && \
curl -fsSL https://nginx.org/keys/nginx_signing.key -o /tmp/nginx.key && \
gpg --dearmor < /tmp/nginx.key | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg > /dev/null
```

> Verify that the downloaded file contains the proper key
```bash
mkdir -p ~/.gnupg && chmod 700 ~/.gnupg && \
gpg --dry-run --quiet --no-keyring --import --import-options import-show /usr/share/keyrings/nginx-archive-keyring.gpg
```

> [!IMPORTANT] 
> Should be able to see the following: `8540A6F18833A80E9C1653A42FD21310B49F6B46`
> the output can contain other keys used to sign the packages

> Set up repository pinning to prefer NGINX packages over distribution-provided ones
```bash
echo -e "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" \
    | sudo tee /etc/apt/preferences.d/99nginx
```

### Install

> To install NGINX, run the following commands
```bash
sudo apt update && \
sudo apt install nginx -y
```

### Test

> To see if NGINX install was successfully. Replace {server_ip} with your server's address.
```
curl http://{server_ip}/
```
> This should output the following:
```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

### Optional
> [!NOTE]
> This part is optional but is strongly recommended

> Ensure that web root exists
```bash
sudo mkdir -p /var/www/html
```

> After seeing if if the service is available you should disable the default site by removing the config from `/etc/nginx/sites-available`
```bash
sudo rm /etc/nginx/sites-enabled/default
```
## DokuWiki

### Prerequisites

> Install the required packages
```bash
sudo apt install php-fpm php-cli php-xml php-mbstring php-gd php-curl php-intl php-zip -y && \
cd /tmp && \
wget -q https://download.dokuwiki.org/src/dokuwiki/dokuwiki-stable.tgz && \
sudo mkdir -p /var/www/html/dokuwiki && \
sudo tar xzf dokuwiki-stable.tgz -C /var/www/html/dokuwiki --strip-components=1 && \
rm dokuwiki-stable.tgz
```

> Set permissions
```bash
sudo chown -R www-data:www-data /var/www/html/dokuwiki
```
### Config

> Now that everything is installed we can add the config file that we'll use for the website. First lets create a config file in `/etc/nginx/sites-enabled`

> [!IMPORTANT]
> For the following use Edit and Execute[^1]
> Manually edit the provided config so that it matches your environment
> {server_name} -> the address of the designated server (e.g.: 192.168.0.254)
> {listen_port} -> the port listening (e.g.: 80)


```bash
sudo tee /etc/nginx/sites-available/dokuwiki > /dev/null <<'EOF'
server {
    listen {listen_port};
    server_name {server_name};
    root /var/www/html/dokuwiki;
    index index.php index.html;
    client_max_body_size 8M;
    location / {
        try_files $uri $uri/ =404;
    }
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
    location ~ /\.ht { deny all; }
    location ~ /data/ { internal; }
    location ~ /conf/ { deny all; }
}
EOF
```

> Now let's link the config file with the dokuwiki source folder
```bash
sudo ln -s /etc/nginx/sites-available/dokuwiki /etc/nginx/sites-enabled/dokuwiki
```

> [!TIP]
> Test the configuration's syntax
> ```bash
> sudo nginx -t
>```

> Now that the configuration is complete at last let's restart the service
```bash
sudo systemctl reload nginx
```

### Setting up a new DokuWiki

> Now the install site is available on `http://{your_address}:{listening_port}/install.php`
> You should see something like this
![dokuwiki_install_page.png](assets/images/dokuwiki_install_page.png)
> Provide the required information to setup the page and delete the `install.php` from `/var/www/html/dokuwiki/`
```bash
rm /var/www/html/dokuwiki/install.php
```

## TeemIP

### Prerequisites

> Install the required packages
```bash
sudo apt update && \
sudo apt install nginx unzip php-fpm php-mysql php-curl php-gd php-mbstring php-xml php-zip php-soap php-apcu php-ldap graphviz -y && \
cd /tmp && \
wget -O teemip.zip https://sourceforge.net/projects/teemip/files/latest/download && \
sudo mkdir -p /var/www/html/teemip && \
sudo unzip teemip.zip -d /var/www/teemip && \
rm teemip.zip
```

> Set permissions
```bash
sudo chown -R www-data:www-data /var/www/html/teemip
```

### Config

> Now let's add the config file within NGINX

> [!IMPORTANT]
> For the following use Edit and Execute[^1]


```bash
sudo tee /etc/nginx/sites-available/teemip > /dev/null <<'EOF'
server {
    listen {listen_port];
    server_name {server_address};
    root /var/www/html/teemip/web;
    index index.php index.html;
    client_max_body_size 8M;
    
    fastcgi_read_timeout 600;
    fastcgi_send_timeout 600;
    proxy_read_timeout 600;
    
    location / {
        try_files $uri $uri/ =404;
    }
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.4-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
    location ~ /\.ht { deny all; }
    location ~ /data/ { internal; }
    location ~ /conf/ { deny all; }
}
EOF
```

> Let's make a link between the link and the source
```bash
sudo ln -s /etc/nginx/sites-available/teemip /etc/nginx/sites-enabled/teemip
```

> [!TIP]
> Test the configuration's syntax
> ```bash
> sudo nginx -t
>```

> Restart the NGINX
```bash
sudo systemctl reload nginx
```

### Setting up a new TeemIP

> Now the install wizard site is available on `http://[your_address]:[listening_port]/setup/wizard.php`
> You should see something like this:

![teemip_welcome_page.png](assets/images/teemip_welcome_page.png)
> [!NOTE]
> We can ignore the version warning

> Let's select the `Install a new TeemIP`
![teemip_install&upgrade.png](assets/images/teemip_install&upgrade.png)

> Accept the EULA
![teemip_eula.png](assets/images/teemip_eula.png)

> [!NOTE]
> If you don't yet have a database set up take a look at the MariaDB section of this document

![teemip_database_connection.png](assets/images/teemip_database_connection.png)

> Create local administrator
![teemip_admin_create.png](assets/images/teemip_admin_create.png)

> [!NOTE]
> Don't forget to set `Sample Data` if you install for produiction 

> Set misc options
![teemip_misc.png](assets/images/teemip_misc.png)

> Check the overview
![teemip_overview.png](assets/images/teemip_overview.png)
> Press install and wait for the magic to happen

## MariaDB

### Prerequisites

> Install the required packages
```bash
sudo apt install mariadb-server mariadb-client
```

> Start the installation
```bash
sudo mariadb-secure-installation
```

> For recommended setup follow the table's instruction below.
> When you run the command you will be prompted to input the root password

>[!TIP] 
> If at anytime you mess up the configuration simply use the `CTRL+C` shortcut to abandon installation

|                          Promp                          | Select |                      What it does                       |
| :-----------------------------------------------------: | :----: | :-----------------------------------------------------: |
|          Switch to unix_socket authentication?          |   N    |          Enable socket authentication for root          |
|                Change the root password?                |   Y    |          Set/Change the MariaDB root password           |
|                 Remove anonymous users?                 |   Y    |            Delete default anonymous account             |
|              Disallow root login remotely               |   Y    |  Prevent root user from connecting from other machines  |
|          Remove test database and access to it          |   Y    | Delete the default test database that anyone can access |
| Delete the default test database that anyone can access |   Y    |              Apply all changes immediately              |

> Enable and start the service
```bash
sudo systemctl enable mariadb && \
sudo systemctl start mariadb
```

>[!TIP]
> To verify if MariaDB is properly running use `systemctl status mariadb`

### Create MariaDB user

> To create a database user you can log into the root user and run the following commands
```bash
sudo mysql -u root -p
```

> Now you are logged in the MariaDB CLI to create a database.
> Replace {db_name} of your choice. To exit type `EXIT`
```SQL
CREATE DATABASE {db_name} CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

> [!IMPORTANT]
> Replace {username}, {password} and {database} sections with the choice of your desire.
> Use Edit and Execute[^1] for the followings

> To create a user that has access to the database created
```bash
sudo mysql -u root -p -e \
"CREATE USER '{username}'@'localhost' IDENTIFIED BY '{password}';
GRANT ALL PRIVILEGES ON {database}.* TO '{username}'@'localhost';
FLUSH PRIVILEGES;"
```

### Test

> To see if the user creation succeeded try to log into the created user with
> ```bash
> sudo mysql -u admin -p
> ```

[^1]: Edit and Execute a way to paste and execute multiple commands. To use press `CTRL+X` and `CTRL+E` in your terminal