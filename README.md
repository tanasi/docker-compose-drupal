# Drupal 8 Docker Compose Development

### Services included

_Every service is optional as declared in the yml file._

- Apache
- Nginx
- Php 7.3 / 7.4 fpm with Xdebug
- MariaDB
- PostgreSQL
- [Memcache](https://hub.docker.com/_/memcached)
- [Redis](https://redis.io/)
- [Mailhog](https://github.com/mailhog/MailHog)
- [Solr](http://lucene.apache.org/solr)
- [Dashboard](https://cloud.docker.com/u/mogtofu33/repository/docker/mogtofu33/dashboard): a very minimal docker dashboard for this stack
- [Portainer](https://github.com/portainer/portainer): (Optional) a full Docker dashboard / manager

#### Database management

- [Adminer](https://www.adminer.org)

## Installation and configuration

### Project installation

_Optional:_ Edit configuration

Recommended on Unix add your local uid/gid.

```bash
vi .env
```

_Optional:_ Customize the stack

Choose a database, remove or add services, add your composer cache folder if needed on service `php`.
Do not touch for a default quick stack.

```bash
vi docker-compose.yml
```

Check the yml file and fix if there is an error message

```bash
docker-compose config
```

### Option 1: existing Drupal 8 project

#### Copy codebase

For an existing Drupal 8 project, copy it here in a folder named `drupal`.

So you have your composer.json file in `drupal\composer.json`

Note that based on Composer template web root must be under `drupal/web`
folder. If not you need to adapt Apache vhost config from
`config/apache/vhost.conf`

```bash
cp -r _YOUR_DRUPAL_ drupal
```

#### Import database

For **MySQL**, copy your database dump **uncompressed** in `./database/mysql-init/*.sql`, it
will be automatically imported on the first launch of the stack.

For **PostgresSQL**, copy your database dump **uncompressed** in `./database/pgsql-init/*.pg_dump`, it
will be automatically imported on the first launch of the stack.

If you want to manually import your database with adminer or included helper scripts
with _Linux_you can skip this step.

#### Launch the stack

```bash
docker-compose up --build -d
```

Wait and check the import of your database dump with (change mysql to pgsql if needed)

```bash
docker-compose logs mysql
```

#### Access the minimal dashboard

- [http://localhost:8181](http://localhost:8181)

### Option 2: setup Vanilla Drupal 8 with Composer

#### Code download

Setup a new Drupal 8 based on a Composer project.

Based on [Drupal 8 template](https://github.com/drupal-composer/drupal-project), include [Drush](http://www.drush.org) and [Drupal console](https://drupalconsole.com/), using [Composer](https://getcomposer.org) locally:


#### Launch the stack

```bash
docker-compose up --build -d
```

#### Get into container and run composer to bring drupal into localhost = drupal dir outside
```bash
sudo docker exec -it dcd-php /bin/bash
cd /var/www/
composer create-project drupal-composer/drupal-project:8.x-dev localhost --stability dev --no-interaction
```

#### Install Drupal 8

To use **PostGreSQL** change **mysql** to **pgsql**

You can replace **standard** by an other profile as **minimal** or **demo_umami** for [Drupal 8.6+](https://www.drupal.org/project/demo_umami).

```bash
docker exec -it -u apache dcd-php /var/www/localhost/vendor/bin/drush -y site:install standard \
    --root=/var/www/localhost/web \
    --account-name=admin \
    --account-pass=password \
    --db-url=mysql://drupal:drupal@mysql/drupal
    #--db-url=pgsql://drupal:drupal@pgsql/drupal
```

#### Access your Drupal 8

- [http://localhost](http://localhost)

Login with _admin_ / _password_:

- [http://localhost/user/login](http://localhost/user/login)

## Daily usage

### Add some modules

```bash
docker exec -it -u apache dcd-php \
    composer --working-dir=/var/www/localhost require \
    drupal/admin_toolbar drupal/ctools drupal/pathauto drupal/token drupal/panels
```

With _Linux_, you can use included helper script

```bash
scripts/composer require drupal/admin_toolbar drupal/ctools drupal/pathauto drupal/token drupal/panels
```

### Enable some modules

```bash
docker exec -it -u apache dcd-php \
    /var/www/localhost/vendor/bin/drush -y en \
    --root=/var/www/localhost/web \
    admin_toolbar ctools ctools_block ctools_views panels token pathauto
```

With _Linux_, you can use included helper script

```bash
scripts/drush -y en admin_toolbar ctools ctools_block ctools_views panels token pathauto
```

### Run a command on the server

```bash
docker exec -it -u apache dcd-php \
    ls -lah /var/www/localhost/web
```

### Access the server with bash

```bash
docker exec -it -w /var/www/localhost -u apache dcd-php bash
```

## Reset the stack

### Destroy containers 

_Note:_ `./drupal/` is persistent but NOT the database files!

Save your database under _Linux_

```bash
scripts/mysql dump
scripts/pgsql dump
```

Stop and remove containers

```bash
docker-compose down
```

### Remove your persistent data (and lost everything!)

```bash
rm -rf data
```

## Linux helpers

For Ubuntu (16+) or Linux you can find in `./scripts/` multiple helpers to quickly
run some daily commands from root folder, and drush/drupal links at the root.

```bash
# Run drush or drupal within the container
scripts/drush status
scripts/drupal site:status
# Quickly dump/restore/drop your DB
scripts/mysql --help
scripts/pgsql --help
# Run composer as a service without local installation
scripts/composer --help
scripts/composer status
```

Install Drupal 8 variant helpers (This delete and replace existing Drupal in `./drupal` folder)

```bash
scripts/install-drupal.sh
scripts/install-drupal.sh list
scripts/install-drupal.sh install -p drupal-demo
```

## Upgrade

Because this project is mainly focused on a one time usage, there is currently
**no upgrade path** and the best way to upgrade is to dump and copy your project to
a new version of this project.

## Suggested tools

- [Opcache GUI](https://github.com/amnuts/opcache-gui)
- [Phpmemcacheadmin](https://github.com/wp-cloud/phpmemcacheadmin)
- [Xdebug GUI](https://github.com/splitbrain/xdebug-trace-tree)
- [Adminer extended](https://github.com/dg/adminer-custom)
- [Php Redis Admin](https://github.com/ErikDubbelboer/phpRedisAdmin)

You can find a script for Linux in `scripts/get-tools.sh` folder to download or update all tools

```bash
chmod +x scripts/get-tools.sh
./scripts/get-tools.sh install
```

## Troubleshooting

### General problem

In case of any problem, first step is to check the configuration and the logs

```bash
docker-compose config
docker-compose logs
```

### Port 80

If you have already a web server running on port __80__ or __443__ on your machine
you must stop it or change __APACHE_HOST_HTTP_PORT__ in __.env__

### Windows

Windows support very partial, before running docker-compose you must run in Powershell

```powershell
$Env:COMPOSE_CONVERT_WINDOWS_PATHS=1
```

Some permissions and privileged problems

- [This issue](https://github.com/docker/for-win/issues/1829)

----
Want some help implementing this on your project? I provide Drupal 8 expertise as a freelance, just [contact me](https://developpeur-drupal.com/en).
