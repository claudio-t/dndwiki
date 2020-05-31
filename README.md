# D&D wiki setup guide

## NGINX & HTTPS

```(bash)
sudo apt-install nginx certbot python-certbot-nginx
```

If the installation is broken run:

```(bash)
sudo -H apt-get --purge remove nginx*
sudo -H apt-get install nginx
```

Setup server config (`engwhy.duckdns.org.conf` in this case),
enable it and test config:

```(bash)
sudo cp nginx/engwhy.duckdns.org.conf /etc/nginx/sites-available
sudo ln -s /etc/nginx/sites-available /etc/nginx/sites-enabled
sudo nginx -t && nginx -s reload
```

Create certificates for the domain (when prompted select 2 for
enabling HTTPS everywhere).

```(bash)
sudo certbot --nginx -d engwhy.duckdns.org -d www.engwhy.duckdns.org
```

Edit `/etc/hosts` to map public IP to hostname:

```()
[...]
XXX.XXX.XXX.XXX   engwhy.duckdns.org
```

## Docker environment

Create a file named `.env` for storing credentials that will
be made automatically available to the `docker-compose.yml` file.

```()
MYSQL_ROOT_PASSWORD=XXX
WIKI_USER=dndwiki
WIKI_PASSWORD=XXXX
WIKI_DB=dndwikidb
```

## MariaDB setup

Restore a previous backup with the following command:

```()
mysql -u root -p < path.to.backup
```

Grant access to wiki user to its dedicated DB:

```()
mysql -u root -p -e "GRANT ALL PRIVILEGES ON dndwikidb.* TO 'dndwiki'
```

## Mediawiki setup

Connect to `localhost:8080`, setup mediawiki, download `LocalSettings.php`
from the container and place it under `./data/mediawiki directory`.  
**NB:** use container IP address to connect to MariaDB instance.

Edit `LocalSettings.php` settings as follows:

```()
[...]

# Change the server name to your domain name
$wgServer = "https://engwhy.duckdns.org";

[...]

# Change the DB server to DB hostname (= Docker container name)
$wgDBserver = "wikidb";

[...]

# Set logo (this assumes that the file exists under images folder)
$wgLogo = "$wgScriptPath/images/tower_lion.png";

[...]

# Edit the cache type
$wgMainCacheType = CACHE_ANYTHING;

# Optionally enable user to register themselves in a private wiki
$wgWhitelistRead = array('Special:RequestAccount', 'Main Page', 'Special:CreateAccount');
```

Then uncomment the relative mount directive along with the autostart
directives in `docker-compose.yml` and restart the containers.

## Automation

To schedule a backup run

```
crontab -e
```

and enter the following line

```
0 16 * * * /home/pi/Docker/dndwiki/scripts/backup-database >> /home/pi/Docker/dndwiki/log/backup-database.log 2>&1
```

In order to automatically renew SSL certificates run instead

```
sudo crontab -e
```

and enter the following line

```
0 17 * * * /home/pi/Docker/dndwiki/scripts/renew-certs >> /home/pi/Docker/dndwiki/log/renew-certs.log 2>&1
```
