# Desenvolvendo Pacotes

- [Introdução](#introduction)
- [Criando um Pacote](#creating-a-package)
- [Estrutura do Pacote](#package-structure)
- [Provedores de Serviço](#service-providers)
- [Convenções de Pacotes](#package-conventions)
- [Fluxo de Desenvolvimento](#development-workflow)
- [Roteamento do Pacote](#package-routing)
- [Configuração do Pacote](#package-configuration)
- [Migrações do Pacote](#package-migrations)
- [Assets do Pacote](#package-assets)
- [Publicando Pacotes](#publishing-packages)

<a name="introduction"></a>
## Introdução

Pacotes são a principal forma de adicionar funcionalidades ao Laravel. Pacotes podem ser, desde uma ótima maneira de trabalhar com datas como [Carbon](https://github.com/briannesbitt/Carbon), uma estrutura de testes BDD completa como [Behat](https://github.com/Behat/Behat).

Naturalmente, existem diferentes tipos de pacotes. Alguns pacotes são independentes, ou seja, eles funcionam com qualquer estrutura, não apenas com Laravel. Carbono e Behat são exemplos de pacotes autônomos. Qualquer um desses pacotes pode ser usado com Laravel simplesmente requisitando-os em seu arquivo `composer.json`.

Porém, outros pacotes são destinados especificamente para uso com Laravel. Nas versões anteriores do Laravel, chamavam-se "bundles". Estes pacotes podiam ter rotas, controladores, view, configuração e migrações especificamente destinadas a melhorar o Laravel. Como nenhum processo especial é necessário para desenvolver pacotes autônomos, este guia abrange principalmente o desenvolvimento daqueles que são específicos para Laravel.

Todos os pacotes do Laravel são distribuídos via [Packagist](http://packagist.org) e [Composer](http://getcomposer.org), então aprender sobre essas maravilhosas ferramentas de distribuição de pacotes do PHP é essencial.

<a name="creating-a-package"></a>
## Criando um Pacote

A maneira mais fácil de criar um pacote para uso com Laravel é o comando `workbench` do Artisan.

**Emitindo O Comando Workbench Do Artisan**

	php artisan workbench

Este comando irá pedir várias informações, tais como o vendor e nome do pacote, bem como o seu nome e endereço de email. Essas informações são usadas para criar o namespace e arquivo `composer.json` de seu pacote.

O nome de vendor é uma forma de diferenciar o seu pacote de outros pacotes com o mesmo nome, de pacotes de autores diferentes. Por exemplo, se eu (Taylor Otwell) criar um novo pacote chamado "Zapper", o nome do fornecedor poderá ser `Taylor`, enquanto o nome do pacote seria `Zapper`.

Uma vez que o comando `workbench` for executado, o pacote vai estar disponível dentro do diretório `workbench` da sua instalação Laravel. Primeiro, você deve executar o comando `composer install` **a partir do diretório raiz de seus pacotes workbench**, no qual instalará dependências e criará os arquivos autoload do Composer do seu pacote. Você pode instruir o comando `workbench` a fazer isso automaticamente ao criar um pacote usando a directiva `--composer`:

**Criando Um Pacote WorkBench E Executando o Composer**

	php artisan workbench --composer

Agora, você deverá registrar o `ServiceProvider` que foi criado para o seu pacote. Para registrar o provedor adicionando no array `providers` no arquivo `app/config/app.php`. Isso irá instruir o Laravel carregar seu pacote quando seu aplicativo for iniciado. Provedores de Serviço usa uma nomenclatura `[Pacote]ServiceProvider` por convenção. Então, usando o exemplo acima, você adicionaria `Taylor\Zapper\ZapperServiceProvider` para o array `providers`.

Uma vez que o provedor tenha sido registrado, você está pronto para começar a desenvolver o seu pacote! No entanto, antes de mergulhar, você pode querer dar uma olhada nas seções abaixo para se familiarizadar melhor com a estrutura de pacotes e fluxo de desenvolvimento.

<a name="package-structure"></a>
## Estrutura do Pacote

Usando o comando `workbench`, seu pacote será configurado com as convenções que permitam o funcionamento correto com outras partes do framework Laravel:

**Estrutura de Diretório de Pacote Básico**

	/src
		/Vendor
			/Package
				PackageServiceProvider.php
		/config
		/lang
		/migrations
		/views
	/tests
	/public

Vamos explorar melhor essa estrutura. O diretório `src/Vendor/Package` é a home de todas as classes do seu pacote, incluindo o `ServiceProvider`. Diretórios `config`, `lang`, `migrations`, e `views`, como você pode imaginar, irá conter os recursos correspondentes para o seu pacote. Os pacotes podem ter qualquer um desses recursos, assim como aplicações "comuns".

<a name="service-providers"></a>
## Provedores de Serviço

Os provedores de serviços são simplesmente classes de inicialização(bootstrap) para pacotes. Por padrão, elas possuem dois métodos: `boot` e `register`. Nesses métodos você pode fazer o que quiser, como: incluir arquivos de rota, registrar bindings no IoC container, atribuir eventos, ou qualquer outra coisa.

O método `register` é chamado imediatamente quando o provedor de serviço é registrando, enquanto o comando `boot` somente é chamado corretante quando um pedido é encaminhado. Então, se ações em seu provedor de serviço depender que um outro provedor de serviço esteja registrado, ou você está substituindo serviços vinculados por um outro provedor, certifique-se de usar o método `boot`.

Quando criar um pacote usando `workbench`, o comando `boot` já conterá uma ação:

	$this->package('vendor/package');

Este método permite ao Laravel saber como carregar corretamente os views, a configuração e outros recursos para a sua aplicação. Em geral, não deve haver necessidade de alterar esta linha, uma vez que irá configurar o pacote usando as convenções do workbench.

<a name="package-conventions"></a>
## Convenções de Pacotes

Ao utilizar recursos de um pacote, como configuração de items ou views, a sintaxe com dois pontos duplo será geralmente utilizada:

**Carregando Uma View De Um Pacote**

	return View::make('package::view.name');

**Recuperando A Configuração de Um Item do Pacote**

	return Config::get('package::group.option');

> **Nota:** Se o pacote contém migrações, considere prefixar o nome de migração com o nome do pacote a fim de evitar conflitos de classe com potenciais nome de outros pacotes.

<a name="development-workflow"></a>
## Fluxo de Desenvolvimento

When developing a package, it is useful to be able to develop within the context of an application, allowing you to easily view and experiment with your templates, etc. So, to get started, install a fresh copy of the Laravel framework, then use the `workbench` command to create your package structure.

After the `workbench` command has created your package. You may `git init` from the `workbench/[vendor]/[package]` directory and `git push` your package straight from the workbench! This will allow you to conveniently develop the package in an application context without being bogged down by constant `composer update` commands.

Since your packages are in the `workbench` directory, you may be wondering how Composer knows to autoload your package's files. When the `workbench` directory exists, Laravel will intelligently scan it for packages, loading their Composer autoload files when the application starts!

<a name="package-routing"></a>
## Roteamento do Pacote

In prior versions of Laravel, a `handles` clause was used to specify which URIs a package could respond to. However, in Laravel 4, a package may respond to any URI. To load a routes file for your package, simply `include` it from within your service provider's `register` method.

**Including A Routes File From A Service Provider**

	public function register()
	{
		$this->package('vendor/package');

		include __DIR__.'/routes.php';
	}

<a name="package-configuration"></a>
## Configuração do Pacote

Some packages may require configuration files. These files should be defined in the same way as typical application configuration files. And, when using the default `$this->package` method of registering resources in your service provider, may be accessed using the usual "double-colon" syntax:

**Accessing Configuração do Pacote Files**

	Config::get('package::file.option');

However, if your package contains a single configuration file, you may simply name the file `config.php`. When this is done, you may access the options directly, without specifying the file name:

**Accessing Single File Configuração do Pacote**

	Config::get('package::option');

### Cascading Configuration Files

When other developers install your package, they may wish to override some of the configuration options. However, if they change the values in your package source code, they will be overwritten the next time Composer updates the package. Instead, the `config:publish` artisan command should be used:

**Executing The Config Publish Command**

	php artisan config:publish vendor/package

When this command is executed, the configuration files for your application will be copied to `app/config/packages/vendor/package` where they can be safely modified by the developer!

> **Note:** The developer may also create environment specific configuration files for your package by placing them in `app/config/packages/vendor/package/environment`.

<a name="package-migrations"></a>
## Migrações do Pacote

You may easily create and run migrations for any of your packages. To create a migration for a package in the workbench, use the `--bench` option:

**Creating Migrations For Workbench Packages**

	php artisan migrate:make create_users_table --bench="vendor/package"

**Running Migrations For Workbench Packages**

	php artisan migrate --bench="vendor/package"

To run migrations for a finished package that was installed via Composer into the `vendor` directory, you may use the `--package` directive:

**Running Migrations For An Installed Package**

	php artisan migrate --package="vendor/package"

<a name="package-assets"></a>
## Assets do Pacote

Some packages may have assets such as JavaScript, CSS, and images. However, we are unable to link to assets in the `vendor` or `workbench` directories, so we need a way to move these assets into the `public` directory of our application. The `asset:publish` command will take care of this for you:

**Moving Assets do Pacote To Public**

	php artisan asset:publish vendor/package

If the package is still in the `workbench`, use the `--bench` directive:

	php artisan asset:publish --bench="vendor/package"

This command will move the assets into the `public/packages` directory according to the vendor and package name. So, a package named `userscape/kudos` would have its assets moved to `public/packages/userscape/kudos`. Using this asset publishing convention allows you to safely code asset path in your package's views.

<a name="publishing-packages"></a>
## Publicando Pacotes

When your package is ready to publish, you should submit the package to the [Packagist](http://packagist.org) repository. If the package is specific to Laravel, consider adding a `laravel` tag to your package's `composer.json` file.

Also, it is courteous and helpful to tag your releases so that developers can depend on stable versions when requesting your package in their `composer.json` files. If a stable version is not ready, consider using the `branch-alias` Composer directive.

Once your package has been published, feel free to continue developing it within the application context created by `workbench`. This is a great way to continue to conveniently develop the package even after it has been published.

Some organizations choose to host their own private repository of packages for their own developers. If you are interested in doing this, review the documentation for the [Satis](http://github.com/composer/satis) project provided by the Composer team.
