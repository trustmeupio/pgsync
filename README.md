# pgsync

Quickly and securely sync data between environments

:tangerine: Battle-tested at [Instacart](https://www.instacart.com/opensource)

## Installation

```sh
gem install pgsync
```

And in your project directory, run:

```sh
pgsync --setup
```

This creates `.pgsync.yml` for you to customize. We recommend checking this into your version control (assuming it doesn’t contain sensitive information). `pgsync` commands can be run from this directory or any subdirectory.

## How to Use

Sync all tables

```sh
pgsync
```

Sync specific tables

```sh
pgsync table1,table2
```

Sync specific rows (existing rows are overwritten)

```sh
pgsync products "where id < 1000"
```

You can also preserve existing rows

```sh
pgsync products "where id < 1000" --preserve
```

Or truncate them

```sh
pgsync products "where id < 1000" --truncate
```

### Exclude Tables

```sh
pgsync --exclude users
```

To always exclude, add to `.pgsync.yml`.

```yml
exclude:
  - table1
  - table2
```

For Rails, you probably want to exclude schema migrations.

```yml
exclude:
  - schema_migrations
```

### Groups

Define groups in `.pgsync.yml`:

```yml
groups:
  group1:
    - table1
    - table2
```

And run:

```sh
pgsync group1
```

You can also use groups to sync a specific record and associated records in other tables.

To get user `123` with his or her orders, last 10 visits, and favorite store, use:

```yml
groups:
  user:
    users: "where id = {id}"
    orders: "where user_id = {id}"
    visits: "where user_id = {id} order by created_at desc limit 10"
    stores: "where id in (select favorite_store_id from users where id = {id})
```

And run:

```sh
pgsync user:123
```

### Schema

Sync schema

```sh
pgsync --schema-only
```

Specify tables

```sh
pgsync table1,table2 --schema-only
```

## Sensitive Information

Prevent sensitive information - like passwords and email addresses - from leaving the remote server.

Define rules in `.pgsync.yml`:

```yml
data_rules:
  email: unique_email
  last_name: random_letter
  birthday: random_date
  users.auth_token:
    value: secret
  visits_count:
    statement: "(RANDOM() * 10)::int"
  encrypted_*: null
```

`last_name` matches all columns named `last_name` and `users.last_name` matches only the users table. Wildcards are supported, and the first matching rule is applied.

Options for replacement are:

- null
- value
- statement
- unique_email
- unique_phone
- random_letter
- random_int
- random_date
- random_time
- random_ip
- untouched

## Multiple Databases

To use with multiple databases, run:

```sh
pgsync --setup db2
```

This creates `.pgsync-db2.yml` for you to edit. Specify a database in commands with:

```sh
pgsync --db db2
```

## Safety

To keep you from accidentally overwriting production, the destination is limited to `localhost` or `127.0.0.1` by default.

To use another host, add `to_safe: true` to your `.pgsync.yml`.

## Upgrading

Run:

```sh
gem install pgsync
```

To use master, run:

```sh
gem install specific_install
gem specific_install ankane/pgsync
```

## Thanks

Inspired by [heroku-pg-transfer](https://github.com/ddollar/heroku-pg-transfer).

## TODO

- Support for schemas other than `public`

## Contributing

Everyone is encouraged to help improve this project. Here are a few ways you can help:

- [Report bugs](https://github.com/ankane/pgsync/issues)
- Fix bugs and [submit pull requests](https://github.com/ankane/pgsync/pulls)
- Write, clarify, or fix documentation
- Suggest or add new features
