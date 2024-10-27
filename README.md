# docker_magento
Magento development setup using docker compose. Not for production!

You need to have the latest docker compose installed. 
My advice is to user docker desktop. The installation 
of docker is beyond the scope of this read me.

**Install Magento 2.4.7 from this GitHub repository**
- check out the files: https://github.com/johnverzijden/docker_magento.git
```bash
docker compose up --build -d
```
```bash
gunzip empty_magento_247.sql.gz
```

You now have to install the database with mysql in the PHP container,
while the sql file is not in the container. You have to options here: 
1. copy the file first
2. run the mysql command from the docker compose root

Easiest way is to copy it first:
```bash
docker compose cp empty_magento_247.sql php:/var/www/html/magento.sql
```
During the mount files of the /var/www/html/app will be mounted to the
/src folder in the docker root. That is where you will build, change and
custom code. Docker will mount the folders as "root" during the initial 
mount. You need to set them to owner app:app. (note: the double slash you 
will only need on a windows docker installation)
```bash
docker compose exec --user root php chown -R app:app //var/www/html
```

From here the easiest way to go, is open a shell in de php container. 
If you used to work with docker compose you can also start magento cli commands 
from the docker compose root. Below is assumed you will open a bash shell in 
the PHP container.

```bash
docker compose exec php bash
```

You are now on the Magento command line root (/var/www/html).

***Magento command line tasks***

Import the database:
```bash
mysql -h mariadb -uroot -pmagento magento < magento.sql
```

For the following you need user and password tokens from Magento. 
Go to your Magento account or open a new Magento account. Got to
"marketplace" and click on your profile. Find the menu option with
"keys". Copy or create new keys, and use them when asked for during 
composer install.
```bash
composer install
```
```bash
bin/magento setup:upgrade
```

Choose a way to open the website. 

Default in this setup
(see docker-compose.yaml) is: ports: - "80:80". If you have webserver running
on your workstation this could conflict. You can change
the port (like: 8080:80).

The default server name (see nginx.conf) is "local.magento".
So you should be able to open the magento website by
"http://local.magento" or if you changed the port "http://local.magento:8080"

Some things to consider:

The vendor folder is not mounted by default. After running composer install
in the container it will still be empty in the /src/vendor. However, debugging 
in your IDE will need the vendor folder. If you would like to keep your
performance in your PHP container, do not mount the vendor, but just copy
the vendor now end then, after a composer install/update. I considered making
a special script, but hey, better do that yourself, soo you know what it
is doing plus you can customise it.

Such a script could look like:
```bash
docker compose exec php composer update && /
docker compose cp php:/var/www/html/vendor/. src/vendor/
```

