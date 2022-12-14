# Magento 2.4.3 CE Install


Installation of Magento 2.4.3-p1, 2.4.3-p2, 2.4.3-p2 CE. Even it's well documentated here some hints and solutions in a short form worked on MAMP and hosting providers.

## General

* [Requirements](https://experienceleague.adobe.com/docs/commerce-operations/installation-guide/system-requirements.html?lang=en#phpunit) 
* PHP Version > use 7.4 (not 8)
* Elasticsearch 7 is **MUST**. Disabling like in older M2 Version won't work and magento 2 catalog disappears.

## Hints

* Use DBeaver for your database tool
* Use an Online Service for Elasticsearch with Elasticsearch 7.1
* Use n98-magerun2.phar

## PHP

* some hosting platforms or MAMP display wrong PHP Version by using php -v
* use for Magento 2.4.3 
```php
php7.4 bin/magento
```

## Install Errors

With Magento 2.4.3 some configurations changed. Some solutions to errors.

#### Error Maria DB Version

Current version of RDBMS is not supported. Used Version: 10.5.16-MariaDB-1:10.5.16+maria~bullseye. Supported versions: MySQL-8, MySQL-5.7, MariaDB-(10.2-10.4)```
  
Installing is aborted if Maria Database 10.5 is installed. Alter the di.xml file to parameter 10.[2-5] and run again.
```php
nano <magento-root>/app/etc/di.xml
```

```xml
<item name="MariaDB-(10.2-10.5)" xsi:type="string">^10\.[2-5]\.</item>
```
#### Error Magento Catalog disappeared or don't show up
Enable a valid Elastic Search 7 connection. Note: If an older version i.e. Elastic Search is installed Magento 2 backend configuration confirms a valid connection but it won't work. Elastic Search 7.1 ist MUST.

#### Error PHP Code Dependeny Injection 
Check your PHP Version you are really running. By command or by `<?php phpinfo();?>` in your `<magento-root>/pub/` directory. Consider to run the php cli with php7.4 -f instead of php -f
```php
php7.4 <magento-root>/bin/magento setup:upgrade
php7.4 <magento-root>/bin/magento setup:di:compile
php7.4 <magento-root>/bin/magento setup:static-content:deploy
php7.4 <magento-root>/bin/magento cache:clean
```
#### Error HTTP 500
Check your error.log regarding misconfigurations and .htaccess directives. Some hosting enviroments don't support FollowSymLinks. Use SymLinksIfOwnerMatch instead.

Find `.htaccess` files with option FollowSymLinks
```sh
  find ./ -name .htaccess -type f -exec grep -Hni "FollowSymLinks" {} \;
```

Find and Replace `.htaccess` files FollowSymLinks by SymLinksIfOwnerMatch
```sh
find ./ -name .htaccess -type f -exec sed -i -e 's/FollowSymLinks/SymLinksIfOwnerMatch/g' {} \;
```

Check php memory - 2G required and modules
```sh
php -i | grep 'memory_limit'
php -r 'phpinfo(INFO_MODULES);'
```
  
#### Error HTTP 400

CSS, JS and Media resources resulting in Error 400 or Error 500. Following directories have to be checked.

```sh
<magento-root>/pub/static/
<magento-root>/pub/media/
```

[Adobe Help configure permissions](https://experienceleague.adobe.com/docs/commerce-operations/installation-guide/prerequisites/file-system/configure-permissions.html?lang=en)
* Access
* Folder / File Permissions
* SymLinksIfOwnerMatch in .htaccess files

Run Magento CLI
 ```php
<magento-root>/bin/magento setup:static-content:deploy
<magento-root>/bin/magento cache:flush
``` 

#### Error Static CSS JS Files not loaded or generated
In some hosting enviroments an error could be tracked down to the php runtime config php_flag engine. By default the value is 0. However apart from security considerations turning value to 1 loads static files.

Open .htaccess in static directory
 ```php
 nano <magento-root>/pub/static/.htaccess
 ``` 
Edit values

 ```php
<IfModule mod_php7.c>
php_flag engine 1
</IfModule>
```
or comment the line
 ```php
<IfModule mod_php7.c>
# php_flag engine 1
</IfModule>
``` 

## Elastic Search 7

Magento 2.4.3 requires Elastic Search 7. If there is no connection you Product Catalog will not display.

### Example with Online Service at bonsai.io
https://app.bonsai.io/signup
  
```php
bin/magento config:set catalog/search/elasticsearch7_server_hostname https://xxxxxx-000000.us-east-1.bonsaisearch.net
bin/magento config:set catalog/search/elasticsearch7_server_port 443
bin/magento config:set catalog/search/elasticsearch7_enable_auth 1
bin/magento config:set catalog/search/elasticsearch7_index_prefix magento2
bin/magento config:set catalog/search/elasticsearch7_username <username>
bin/magento config:set catalog/search/elasticsearch7_password <password>
```
### Example Elastic Search local on server system

```php
bin/magento config:set catalog/search/elasticsearch7_server_hostname localhost
bin/magento config:set catalog/search/elasticsearch7_server_port 9200
bin/magento config:set catalog/search/elasticsearch7_enable_auth 0
bin/magento config:set catalog/search/elasticsearch7_index_prefix magento2
```

You may check your index at the server in case you Magento Catalog don't show i.e. after an upgrade from older versions.

### Example Elastic Search at elastic.io
[Elastic Cloud Sign-In](https://cloud.elastic.co/login)

```php
bin/magento config:set catalog/search/elasticsearch7_server_hostname i-o-optimized-deployment-be8624.es.us-west1.gcp.cloud.es.io
bin/magento config:set catalog/search/elasticsearch7_port 9243
bin/magento config:set catalog/search/elasticsearch7_enable_auth 1
bin/magento config:set catalog/search/elasticsearch7_username <username>
bin/magento config:set catalog/search/elasticsearch7_password <password>
```
If errors occur check the service by using curl command (Note: -u option with username:password)
```shell
curl -u <username>:<password> https://i-o-optimized-deployment-be8624.es.us-west1.gcp.cloud.es.io:9243
```

### Magento CLI config search engine

show search engine
```php
bin/magento config:show catalog/search/engine
```
set search engine
```php
bin/magento config:set catalog/search/engine 'lmysql'
```
Enable Elastic Search (Elasticsearch7 is default from M 2.4 CE)
```php
bin/magento setup:install --enable-modules=Magento_InventoryElasticsearch,Magento_Elasticsearch7,Magento_Elasticsearch6,Magento_Elasticsearch
bin/magento module:enable Magento_Elasticsearch7 Magento_InventoryElasticsearch
```
