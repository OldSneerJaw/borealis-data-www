---
title: Borealis Isolated Postgres
subtitle: Add-on Management
description: Instructions for managing a Borealis Isolated Postgres add-on
---

[Borealis Isolated Postgres](https://elements.heroku.com/addons/borealis-pg) add-ons can be managed with the [borealis-pg-cli](https://www.npmjs.com/package/borealis-pg-cli) plugin for the [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli).

Each add-on database is deliberately isolated from the open internet in a virtual private cloud, which makes it impossible to access the database directly. Therefore, borealis-pg-cli may be used to create a secure tunnel to an add-on database to perform ad hoc SQL queries, run database migration scripts and manage PostgreSQL extensions. The following describes the use of borealis-pg-cli.

## Pre-requisites

These utilities are required to proceed:

- [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
- [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli#download-and-install)

## Getting started

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

To start a secure tunnel session for an add-on database, use the `borealis-pg:tunnel` command. For example, to connect with a dedicated personal user with read-only access to an add-on database that is attached to the Heroku app `sushi` using an attachment name of `DATABASE`, you can execute the following:

```shell
$ heroku borealis-pg:tunnel --app sushi --addon DATABASE_URL
```

Or, to connect with a dedicated personal user that has both read and write access to the same add-on database:

```shell
$ heroku borealis-pg:tunnel --app sushi --addon DATABASE --write-access
```

Once the session has started, you can use a tool like [psql](https://www.postgresql.org/docs/current/app-psql.html) or [pgAdmin](https://www.pgadmin.org/) to interact with the database using the credentials provided in the `borealis-pg:tunnel` command's output.

To execute non-interactive commands (e.g. database migrations) as the Heroku application database user, use the `borealis-pg:run` command. For example, to execute the database migrations for a Ruby on Rails application, you can run:

```shell
$ heroku borealis-pg:run --app sushi --addon DATABASE --shell-cmd 'rake db:migrate' --write-access
```

Alternatively, to execute raw SQL non-interactively from a file using the same add-on database:

```shell
$ heroku borealis-pg:run --app sushi --addon DATABASE --db-cmd-file './prepopulate.sql' --write-access
```

## Notes

A list of PostgreSQL extensions that are supported by Borealis Isolated Postgres may be found [here](./pg-extensions-support).

By default, the `borealis-pg:run` command executes given commands as the Heroku application's database user, but it can be made to execute its commands as a personal user (i.e. a database user that is tied to the current Heroku CLI user account) instead by including the `--personal-user` option. The `borealis-pg:tunnel` command always runs as a personal user, though. In either case, if creating tables, indexes, views or other objects as a personal user, the corresponding objects will be owned by that personal user unless/until explicitly reassigned. To reassign every object owned by the current user to the application's read/write user, you can execute the following SQL (assuming the application user's name is `app_rw_102719c813b42c81eb94f6b441784828`):

```sql
REASSIGN OWNED BY CURRENT_USER TO app_rw_102719c813b42c81eb94f6b441784828;
```

Under most circumstances, the Heroku CLI will periodically check for updates and keep the borealis-pg-cli plugin up to date automatically as new releases are published, but you can force it to update the plugin at any time as follows on the command line:

```shell
$ heroku plugins:update
```
