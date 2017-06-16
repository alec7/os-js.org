---
title: Authentication Modules
layout: layout.html
---

# Authentication Modules

You can change the login behaviour of OS.js by changing the configuration to a module that is included. You can also, of course; make your own!

## Demo

For demonstration purposes. Does not do any actual athentication and has no restrictions.

*Please note that this module has no support for user management.*

---

## Database

This is just a simple database backend that allows you to store your users (and settings etc.).

**Also set up the [Database Storage](/manual/storage/modules/#database) module for this to work as intended.**

### Sqlite

```bash
# Install dependencies
$ npm install sqlite3 bcrypt

# Set up configuration
$ node osjs config:set --name=authenticator --value=database
$ node osjs config:set --name=server.modules.auth.database.driver --value=sqlite

# Set up database
$ cp src/templates/misc/authstorage.sqlite src/server/

# Rebuild
$ node osjs build

# Now add yourself as an admin
$ node bin/add-user.js add anders admin
$ mkdir vfs/home/anders
```

### Mysql

```bash
# Install node dependencies
$ npm install mysql bcrypt

# If you are on PHP 5.5 or below (and you actually use the PHP backend):
$ cd src/server/php
$ composer install

# Change the configured authenticator and its options
$ node osjs config:set --name=authenticator --value=database
$ node osjs config:set --name=server.modules.auth.database.driver --value=mysql
$ node osjs config:set --name=server.modules.auth.database.mysql.host --value=localhost
$ node osjs config:set --name=server.modules.auth.database.mysql.user --value=osjsuser
$ node osjs config:set --name=server.modules.auth.database.mysql.password --value=osjspassword
$ node osjs config:set --name=server.modules.auth.database.mysql.database --value=osjs

# Make OS.js reload after you log out
$ node osjs config:set --name=client.ReloadOnShutdown --value=true

# Rebuild
$ node osjs build:config build:core

# Set up database
$ mysql -h localhost -u root -p

mysql> CREATE DATABASE osjs;
mysql> GRANT USAGE ON *.* TO osjsuser@localhost IDENTIFIED BY 'osjspassword';
mysql> GRANT ALL PRIVILEGES ON osjs.* TO osjsuser@localhost;

# Then set up database tables
$ mysql -h localhost -u root -p osjs < src/templates/misc/authstorage.sql

# Now add yourself as an admin
$ node bin/add-user.js add anders admin
$ mkdir vfs/home/anders
```

**NOTE:** Remember to restart the server to reload configuration changes.

---

## PAM and Shadow

Log in via the system (`PAM` or `Shadow`) authentication system. **This is only available for Linux and Node**.

**Also set up the [System storage](/manual/storage/modules/#system) module for this to work as intended.**

### Setup:

```bash
# You need the PAM developer package to build node gyp module
$ sudo apt-get install libpam0g-dev

# PAM dependencies (you need PAM development package on your system)
$ npm install nan@1.1.0
$ npm install authenticate-pam
$ npm install userid

# Shadow dependencies
$ npm install git+https://github.com/andersevenrud/passwd-linux
$ npm install userid

# Set up groups
$ mkdir /etc/osjs
$ edit /etc/osjs/groups.json

# Set up package blacklist (optional)
$ edit /etc/osjs/blacklist.json

# Change the configured authenticator (or use "shadow" here)
$ node osjs config:set --name=authenticator --value=pam

# Make OS.js reload after you log out
$ node osjs config:set --name=client.ReloadOnShutdown --value=true

# Update configuration and template files
$ node osjs build:config build:core
```

By default, this module expects you to store the data in `/etc/osjs`, but you can modify this (see `server.modules.*` tree for settings).

### groups.json

This is an example file for `groups.json`

```json
{
  "anders": ["admin"],
  "guest": ["api", "application", "upload", "fs"],
  "marcello": ["api", "application", "curl", "upload", "fs"]
}
```

### blacklist.json

This is an example file for `blacklist.json`

```json
{
  "anders": ["default/ApplicationDraw"]
}
```

**NOTE:** On some systems you might have to install `authenticate-pam` with `npm install -g` or else you might get a *Error in service module* upon request.

**NOTE:** Also, on some systems you might have to run OS.js server as an administrator (`sudo`) depending on the PAM setup.

**NOTE:** This module has no support for user management at the moment. You'll have to use the system tools.

**NOTE:** Remember to restart the server to reload configuration changes.
