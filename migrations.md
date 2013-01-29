# Migrações & Seeding

- [Introdução](#introduction)
- [Criando Migrações](#creating-migrations)
- [Executando Migrações](#running-migrations)
- [Revertendo Migrações](#rolling-back-migrations)
- [Seeding(Alimentando) Banco de Dados](#database-seeding)

<a name="introduction"></a>
## Introdução

Migrações é tipo de controle de versão para o seu banco de dados. Eles permitem um time modificar o esquema de um banco de dados e manter-se atualizados do esquema atual. Migrações é tipicamente emparelhado com o [Construtor de Esquemas](/docs/schema) para facilmente gerenciar seus esquemas da aplicação.

<a name="creating-migrations"></a>
## Criando Migrações

To create a migration, you may use the `migrate:make` command on the Artisan CLI:

**Creating A Migration**

	php artisan migrate:make create_users_table

The migration will be placed in your `app/database/migrations` folder, and will contain a timestamp which allows the framework to determine the order of the migrations.

You may also specify a `--path` option when creating the migration. The path should be relative to the root directory of your installation:

	php artisan migrate:make foo --path=app/migrations

The `--table` and `--create` options may also be used to indicate the name of the table, and whether the migration will be creating a new table:

	php artisan migrate:make create_users_table --table=users --create

<a name="running-migrations"></a>
## Executando Migrações

**Running All Outstanding Migrations**

	php artisan migrate

**Running All Outstanding Migrations For A Path**

	php artisan migrate --path=app/foo/migrations

**Running All Outstanding Migrations For A Package**

	php artisan migrate --package=vendor/package

> **Note:** If you receive a "class not found" error when running migrations, try running the `composer update` command.

<a name="rolling-back-migrations"></a>
## Revertendo Migrações

**Rollback The Last Migration Operation**

	php artisan migrate:rollback

**Rollback all migrations**

	php artisan migrate:reset

**Rollback all migrations and run them all again**

	php artisan migrate:refresh

	php artisan migrate:refresh --seed

<a name="database-seeding"></a>
## Database Seeding

Laravel also includes a simple way to seed your database with test data using seed files. All seed files are stored in `app/database/seeds`. Seed files should be named according to the table they seed, and simply return an array of records.

**Example Database Seed File**

	<?php

	return array(

		array(
			'email' => 'john@example.com',
			'votes' => 10,
			'created_at' => new DateTime,
			'updated_at' => new DateTime,
		),

		array(
			'email' => 'smith@example.com',
			'votes' => 20,
			'created_at' => new DateTime,
			'updated_at' => new DateTime,
		),

	);

To seed your database, you may use the `db:seed` command on the Artisan CLI:

	php artisan db:seed

You may also seed your database using the `migrate:refresh` command, which will also rollback and re-run all of your migrations:

	php artisan migrate:refresh --seed
