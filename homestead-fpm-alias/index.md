---
title: Homestead PHP-FPM Aliases
slug: homestead-php-fpm-aliases
description: Create your own bash alias to change Homestead's PHP-FPM settings per site.
date: 2022-08-15
tags: [snippet, laravel, homestead, bash, fpm, nginx]
sources: []
---
# Homestead PHP-FPM Aliases

Laravel Homestead comes prepackaged with a number of aliases (shortcuts for commands) one of which is `php81`. Calling this will execute the following commands:

```bash
$ php81

# sudo update-alternatives --set php /usr/bin/php8.1
# sudo update-alternatives --set php-config /usr/bin/php-config8.1
# sudo update-alternatives --set phpize /usr/bin/phpize8.1
```

This only changes the CLI version of PHP - the web server version is not affected.

Let's create our own function to take a PHP version number and optional test site url to change.

Here's a summary of the steps:

1. Open the `aliases` file from your Homestead directory
2. Add the [function](#fpm-function) and save the file
3. Re-provision the Homestead VM `vagrant reload --provision`

## Creating the function

```bash
function fpm() {
    version=$1
    project=${PWD##*/}

    if (( $# > 1 ))
    then
        site="$2"
    else
        site="$project.test"
    fi

    echo "version: $version"
    echo "currentproject: $project"
    echo "site: $site"

    sitePath=/etc/nginx/sites-available/"$site"
    echo "Switching $sitePath to php-fpm $version and restarting nginx..."

    c1="sed -Ei 's/php[0-9].[0-9]-fpm.sock/php"$version"-fpm.sock/g' $sitePath"
    c2="php-fpm$version -t"
    c3="service php$version-fpm restart"

    bash -c "sudo $c1"
    bash -c "sudo $c2"
    bash -c "sudo $c3"

    sudo service nginx restart
}
```

Now you can run the command from your project root

```bash
$ cd code/project-name
# /home/vagrant/code/project-name

$ fpm 8.1
# or fpm 8.1 different-project-name.test
```

If you want to keep your functions consistent with Homestead's aliases you can create a function for each version:

```bash
function fpm81() {
    fpm 8.1 $1 $2
}

function fpm80() {
    fpm 8.0 $1 $2
}

# etc
```

and use in the same way:

```php
$ fpm81

# or
$ fpm81 project.test
```