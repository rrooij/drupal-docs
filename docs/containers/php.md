# PHP

You can configure PHP via environment variables that listed at https://github.com/wodby/drupal-php

## Composer

Composer package `hirak/prestissimo:^0.3` installed globally for wodby user to download dependencies in parallel.

## Installed PHP extensions

See https://github.com/wodby/php#php-extensions

## Drush

PHP container comes with pre-installed drush and drush launcher. Drush launcher will help you use drush that comes with your project without specifying the full path to it. 

You can execute drush commands remotely via drush aliases. Download drush aliases from `Profile > Misc > Drush aliases` page and place them to `~/.drush`. Execute commands (replace `[tokens]` with the real values) like this:

```bash
$  drush @[organization].[application].[instance] [drush command]
```

## Drupal Console

PHP container comes with installed drupal console launcher (not the same as drupal console), the launcher used to be able run drupal console without specified the full path to it. Please note that starting Drupal Console ~1.0 you have to install it manually (via composer) per project.

## Environment variables

In addition to [global environment variables](https://help.wodby.com/infrastructure/environment-variables), we provide the following variables in PHP container that you can use in your post-deployment scripts:

| Variable              | Description                                                      |
| --------------------- | ---------------------------------------------------------------- |
| `$APP_ROOT`           | `/var/www/html` by default                                       |
| `$HTTP_ROOT`          | e.g. `/var/www/html/web`                                         |
| `$CONF_DIR`           | `/var/www/conf` by default                                       |
| `$DRUPAL_SITE`        | Drupal site directory, e.g. `default`                            |
| `$WODBY_APP_NAME`     | My app                                                           |
| `$WODBY_HOST_PRIMARY` | example.com                                                      |
| `$WODBY_URL_PRIMARY`  | http://example.com                                               |
| `$WODBY_HOSTS`        | `[ "example.com", "dev.example.org", "dev.example.org.wod.by" ]` |

!!! warning "WARNING" 
    Some environment variables used by PHP may be overridden in [`wodby.settings.php`](../drupal-settings.md) file

Deprecated variables:

| Variable             | Instead use    |
| -------------------- | -------------- |
| `$WODBY_APP_ROOT`    | `$APP_ROOT`    |
| `$WODBY_APP_DOCROOT` | `$HTTP_ROOT`   |
| `$WODBY_CONF`        | `$CONF_DIR`    |
| `$WODBY_DIR_CONF`    | `$CONF_DIR`    |
| `$WODBY_APP_SUBSITE` | `$DRUPAL_SITE` |

!!! info "Variables availability" 
    Environment variables provided by Wodby are always available in PHP even if `PHP_FPM_CLEAR_ENV` set to `no`. 

## XDebug

Follow these steps to debug your application instance remotely with [xdebug](http://xdebug.org/docs/install):

1. Enable xdebug for your instance from `[Instance] > Stack > Settings`
2. Set up forwarding for xdebug: copy _Xdebug SSH tunnel_ command from `[Instance] > Stack > PHP` and run on your local machine
3. Make sure you have your IDE xdebug listener running on port 9000
4. Start debugging in IDE
5. Start your browser debug helper plugin ([Chrome](https://chrome.google.com/webstore/detail/xdebug-helper/eadndfjplgieldjbigjakmdgkmoaaaoc?hl=en) or [Firefox](https://addons.mozilla.org/en-us/firefox/addon/the-easiest-xdebug)) and open the page you want to debug

!!! info "Debugging locally" 
    To learn how to use xdebug locally see [this article](../local/xdebug.md) 

## Redirects

If you need to make a redirect from one domain to another you can do it by customizing configuration files of nginx or by adding the snippets below to your `settings.php` file.

Redirect from one domain to another:

```php
if (isset($_SERVER['WODBY_ENVIRONMENT_TYPE']) && $_SERVER['WODBY_ENVIRONMENT_TYPE'] == 'prod' && php_sapi_name() != "cli") {
    if ($_SERVER['HTTP_HOST'] == 'redirect-from-domain.com') {
      header('HTTP/1.0 301 Moved Permanently');
      header('Location: http://redirect-to-domain.com' . $_SERVER['REQUEST_URI']);
      exit();
    }
}
```

Redirect from multiple domains:

```php
if (isset($_SERVER['WODBY_ENVIRONMENT_TYPE']) && $_SERVER['WODBY_ENVIRONMENT_TYPE'] == 'prod' && php_sapi_name() != "cli") {
    $redirect_from = array(
      'redirect-from-domain-1.com',
      'redirect-from-domain-2.com',
    );

    if (in_array($_SERVER['HTTP_HOST'], $redirect_from)) {
      header('HTTP/1.0 301 Moved Permanently');
      header('Location: http://redirect-to-domain.com' . $_SERVER['REQUEST_URI']);
      exit();
    }
}
```

Redirect from HTTP to HTTPS can be enabled on a domain edit page from the dashboard.

## PHPUnit

1. Inside your drupal/core directory, copy the file `phpunit.xml.dist` and rename it to `phpunit.xml`
2. Open that file and make sure that you update `SIMPLETEST_BASE_URL` to `http://nginx`
3. In order to make sure that your DB connection is working as well, update `SIMPLETEST_DB` to `mysql://drupal:drupal@mariadb/drupal`

## NewRelic APM monitoring

You can add NewRelic APM monitoring for PHP by adding environment variables `PHP_NEWRELIC_ENABLED=1` and `PHP_NEWRELIC_LICENSE` with your license number to PHP-FPM container. Application name will be automatically set to `[Wodby Application Name] - [Wodby Instance Name]`, if you want to change it, use `PHP_NEWRELIC_APPNAME`. 

## Profiling

You can profile your PHP application either via Xdebug traces (+[Webgrind](webgrind.md)) or Tideways XHProf extensions.