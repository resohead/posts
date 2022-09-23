---
title: Local composer repositories
slug: local-composer-repositories
description: Use local versions of packages in demo applications during development.
date: 2022-08-15
tags: [composer, php, bash, homestead, local, packages]
sources:
    - https://calebporzio.com/bash-alias-composer-link-use-local-folders-as-composer-dependancies
    - https://barryvanveen.nl/blog/44-package-development-run-a-package-from-a-local-directory
---

# Local composer repositories

You might be working on a Laravel package or a framework pull request where you need to test any changes in a real application.
It might be possible to create a proof of concept by editing the vendor file directly but any real work will need to be saved in a separate repository.

## 1. Create a local package repository
Fork a package and clone source code you want to make changes to
```
$ cd code/packages

$ git clone [URL] {directory}
# git clone https://github.com/your/package-fork.git
```

or create your own new project/package
```
$ cd code/packages/{vendor}/{project}
# cd code/packages/your/package-fork

$ composer init

$ git init
```

## 2. Prepare a demo application
Create a new Laravel project or open an existing app that will require our local repository/package.
```
$ laravel new demo
```

At this point we have package repository and a demo app. Next step is to get our demo app to use (require) our local package.

## 3. Update composer repository path
Edit the `composer.json` in the `demo` app and add a `path` repository and the relative URL to the package.

```json
// ...
"repositories": [
    {
        "type": "path",
        "url": "../packages/your/package-fork"
    }
],
// ...
```

## 4. Composer require dev branch
Edit `composer.json` and the required package version to use `-dev` flag.

The version specified in your demo app must match the checked-out branch of the locally forked framework with a `dev` prefix or suffix.

> :tip: Take a closer look...
> Versions with a `.` period use a `x-dev` suffix

| Branch | Composer version |
|---------|---------|
| main | dev-main   |
| master | dev-master   |
| 1.o | 1.0.x-dev   |
| 5.4 | 5.4.x-dev   |
| 9.x | 9.x.x-dev   |

Example composer.json
```json
"require": {
    "vendor-name/package-name": "dev-main"
},
```

## 5. Composer update
Let's pull in our local files by running `composer update` in your demo app.

```bash
## update entire project
composer update

## specify the repo
composer update vendor-name/package-name
```

Make sure you see a message about symlinked files, otherwise check:
- your package branch matches the version in `composer.json`
- your demo app `path` in `composer.json`

## 6. Add Bash alias (optional)

Here's a tip from Caleb Porzio on how to create a bash alias to link local composer repositories:

Read the full article here: https://calebporzio.com/bash-alias-composer-link-use-local-folders-as-composer-dependancies

Add the following command to your bash aliases located in:
- `~/.bashrc`
- `~/.bash_profile`
- `~/.zshrc`
- `aliases` (Homestead)

```bash
composer-link() {
    composer config repositories.local '{"type": "path", "url": "'$1'"}' --file composer.json
}
```

> Refresh your aliases after making changes by running `source ~/.bashrc` or `vagrant up --provision` in Homestead.


Run your new function from within the `demo` folder:
```bash
$ cd code/demo

$ composer-link ../vendor-name/package-name
# or whatever the path is to your local package repository
```

Then require the package as normal specifying the `@dev` flag if needed:
```bash
composer require vendor-name/package-name { @dev }
```

## Symlink failed?
If you are running your local packages through a virtual machine (e.g. Homestead) you need to start your VM from an administrator terminal.

Thanks to Barry van Veen for this tip. Read the full article here: https://barryvanveen.nl/blog/44-package-development-run-a-package-from-a-local-directory