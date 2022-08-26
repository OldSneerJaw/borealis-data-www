---
title: Borealis Isolated Postgres
subtitle: Command Line Interface
description: Instructions for managing a Borealis Isolated Postgres add-on via CLI
---

[Borealis Isolated Postgres](https://elements.heroku.com/addons/borealis-pg) add-ons can be managed with the [borealis-pg-cli](https://www.npmjs.com/package/borealis-pg-cli) plugin for the [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli).

Each add-on database is securely isolated from the open internet in a virtual private cloud, which makes it impossible to access the database across the internet directly. Therefore, use borealis-pg-cli to establish a secure tunnel to an add-on database to perform ad hoc SQL queries, run database migration scripts and manage PostgreSQL extensions and database users. The following describes the use of borealis-pg-cli.

## Pre-requisites

These utilities are required to proceed:

- [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
- [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli#download-and-install)
- [psql](https://www.postgresql.org/download/) (required only for the `borealis-pg:psql` command)

## Instructions

#### Getting started

With the pre-requisites installed, simply execute the following on the command line to install the plugin:

```shell
$ heroku plugins:install borealis-pg-cli
```

If you notice build warnings from `gyp` related to `cpu-features` during the preceeding operation, don't worry; this is an optional dependency that will not impact use of the plugin.

When installation is complete, execute the following to see the root documentation for the borealis-pg-cli plugin:

```shell
$ heroku help borealis-pg
```

You can use `heroku help` to see the documentation for any command or sub-command. For example, for help with the command that installs PostgreSQL extensions:

```shell
$ heroku help borealis-pg:extensions:install
```

#### Database connections

To connect securely to an add-on database using [psql](https://www.postgresql.org/docs/current/app-psql.html), use the `borealis-pg:psql` command. For example:

```shell
$ heroku borealis-pg:psql --app sushi
```

To start a persistent secure tunnel session for an add-on database, use the `borealis-pg:tunnel` command. For example, to connect with a dedicated personal user with read-only access to an add-on database that is attached to the example Heroku app `sushi`, you can execute the following:

```shell
$ heroku borealis-pg:tunnel --app sushi
```

Or, for a persistent secure tunnel with a dedicated personal user that has both read and write access to the same add-on database:

```shell
$ heroku borealis-pg:tunnel --app sushi --write-access
```

Once the session has started, you can use a tool like [pgAdmin](https://www.pgadmin.org/) to interact with the database using the credentials provided in the `borealis-pg:tunnel` command's output. Press _Ctrl_+_C_ to close the tunnel and end the session.

To execute non-interactive commands (e.g. database migrations) as the Heroku app database user, use the `borealis-pg:run` command. For example, to execute the database migrations for a Ruby on Rails application, you can run:

```shell
$ heroku borealis-pg:run --app sushi --shell-cmd 'rake db:migrate' --write-access
```

Alternatively, to execute raw SQL non-interactively from a file using the same add-on database:

```shell
$ heroku borealis-pg:run --app sushi --db-cmd-file './prepopulate.sql' --write-access
```

By default, the `borealis-pg:run` command executes given commands as the Heroku app's database user, but it can be made instead to execute its commands as a personal user (i.e. a database user role that is tied to a specific Heroku CLI user account) by specifying the `--personal-user` option. The `borealis-pg:psql` and `borealis-pg:tunnel` commands always run as a personal user, however. In any case, if creating tables, indexes, views or other objects as a personal user, the corresponding objects will be owned by that personal user unless/until explicitly reassigned. To reassign every object owned by the current user to the application's read/write user, you can execute the following SQL (assuming the application read/write user's name is `app_rw_0123456789abcdef`):

```sql
REASSIGN OWNED BY CURRENT_USER TO app_rw_0123456789abcdef;
```

#### PostgreSQL extensions

To install a PostgreSQL extension on an add-on database, use the `borealis-pg:extensions:install` command. For example, to install the [PostGIS](https://postgis.net/) extension:

```shell
$ heroku borealis-pg:extensions:install --app sushi postgis
```

Removing a PostgreSQL extension is done with the `borealis-pg:extensions:remove` command:

```shell
$ heroku borealis-pg:extensions:remove --app sushi uuid-ossp
```

And to see a list of all installed PostgreSQL extensions, use the `borealis-pg:extensions` command:

```shell
$ heroku borealis-pg:extensions --app sushi
```

A list of PostgreSQL extensions that are supported by Borealis Isolated Postgres may be found [here](./pg-extensions-support).

#### Database users

The Heroku app is automatically assigned read/write and read-only database user credentials when an add-on is provisioned. And personal database user credentials are automatically created/rotated every time an authorized user executes one of the `borealis-pg:psql` or `borealis-pg:tunnel` commands (or `borealis-pg:run` with the `--personal-user` option). To list the active database users, execute the `borealis-pg:users` command:

```shell
$ heroku borealis-pg:users --app sushi
```

To seamlessly reset all of an add-on database's user credentials, use the `borealis-pg:users:reset` command:

```shell
$ heroku borealis-pg:users:reset --app sushi
```

A full database credentials reset will generate new usernames and passwords for the Heroku app's read/write and read-only user roles. The previous Heroku app user credentials will continue to remain valid for several minutes afterward to ensure there is an overlap between them to prevent application downtime. All personal database user roles will be deactivated, but the next time an affected user executes one of the `borealis-pg:psql` or `borealis-pg:tunnel` commands (or `borealis-pg:run` with the `--personal-user` option), their database user roles will be reactivated and new credentials will be generated. No database objects (tables, views, indexes, etc.) or table data will be lost during a full database credentials reset.

## Updates

Under most circumstances, the Heroku CLI will periodically check for updates and keep the borealis-pg-cli plugin up to date automatically as new releases are published, but you can force it to update the plugin at any time as follows on the command line:

```shell
$ heroku plugins:update
```
