# IoC Container

- [Introdução](#introduction)
- [Uso Básico](#basic-usage)
- [Resolução Automatica](#automatic-resolution)
- [Uso Prático](#practical-usage)
- [Provedores de Serviço](#service-providers)

<a name="introduction"></a>
## Introdução

O IoC(Inversão de Controle) container do Laravel é uma poderosa ferramenta para gerenciar dependências de classes. Injeção de dependências é um método de remoção de alta dependências de classes. Em disso, as dependências são injetadas em tempo de execução permitindo uma maior flexibilidade, assim implementações de dependências podem ser trocadas facilmente.

Entender o Laravel IoC container é essencial para contruir poderosas e grandes aplicações, bem como contribuir para o núcleo em si do Laravel.

<a name="basic-usage"></a>
## Uso Básico

São duas as maneiras de resolver dependências com o IoC container: via Closure callbacks ou resolução automática. Primeiro, iremos explorar Closure callbacks. Um "tipo" pode ser ligado dentro do container:

**Ligando Um Tipo No Container**

	App::bind('foo', function()
	{
		return new FooBar;
	});

**Resolvendo Um Tipo Do Container**

	$value = App::make('foo');

Quando o método `App::make`, a Closure callback é executada e o resultado é retornado.

Algumas vezes, talvez você queira ligar algo no container que só deve ser resolvida uma vez, e na mesma instância, ele deve ser devolvido em chamadas subseqüentes para o container:

**Ligando Um Tipo "Compartilhado" No Container**

	App::singleton('foo', function()
	{
		return new FooBar;
	});

Você também pode ligar uma instancia de um objeto existente no container usando o método `instance`:

**Ligando Uma Instância Existente No Container**

	$foo = new Foo;

	App::instance('foo', $foo);

<a name="automatic-resolution"></a>
## Resolução Automática

O IoC container é poderoso o suficiente para resolver classes sem nenhuma configuração em muitos cenários. Por exemplo:

**Resolvendo Uma Classe**

	class FooBar {

		public function __construct(Baz $baz)
		{
			$this->baz = $baz;
		}

	}

	$fooBar = App::make('FooBar');

Note-se que mesmo que não registrando a classe FooBar no container, ele ainda será capaz de resolver a classe, mesmo injetando a dependência `Baz` automaticamente!

Quando um tipo não está vinculada no container, ele irá utilizar as meios de Reflexão do PHP para inspecionar a classe e ler o construtor do type-hints(indutor de tipo). Usando essas informações, o container pode automaticamente construir uma instância da classe.

No entanto, alguns casos, uma classe pode depender de uma implementação de interface, e não um "tipo concreto". Quando for assim, o método `App::bind` deve ser utilizado para informar o cointainer que implementação de interface injetar:

**Ligando Uma Interace Para Uma Implementação**

	App::bind('UserRepositoryInterface', 'DbUserRepository');

Agora considere o seguinte controlador:

	class UserController extends BaseController {

		public function __construct(UserRepositoryInterface $users)
		{
			$this->users = $users;
		}

	}

Uma vez que ligamos o `UserRepositoryInterface` a um tipo concreto, o `DbUserRepository` será automaticamente injetado este controlador quando ele é criado.

<a name="practical-usage"></a>
## Uso Prático

Laravel fornece muitas oportunidade para usar o IoC container e adicionar flexibilidade e testabilidade para a sua aplicação. Um exemplo é ao resolver controladores. Todos os controladores são resolvidos através do IoC container, significando que você pode type-hint(induzir o tipo) das dependências no seu construtor do controlador, e eles serão automaticamente injetados.

**Type-Hinting Controller Dependencies**

	class OrderController extends BaseController {

		public function __construct(OrderRepository $orders)
		{
			$this->orders = $orders;
		}

		public function getIndex()
		{
			$all = $this->orders->all();

			return View::make('orders', compact('all'));
		}

	}

In this example, the `OrderRepository` class will automatically be injected into the controller. This means that when [unit testing](/docs/testing) a "mock" `OrderRepository` may be bound into the container and injected into the controller, allowing for painless stubbing of database layer interaction.

[Filters](/docs/routing#route-filters), [composers](/docs/responses#view-composers), and [event handlers](/docs/events#using-classes-as-listeners) may also be resolved out of the IoC container. When registering them, simply give the name of the class that should be used:

**Other Examples Of IoC Usage**

	Route::filter('foo', 'FooFilter');

	View::composer('foo', 'FooComposer');

	Event::listen('foo', 'FooHandler');

<a name="service-providers"></a>
## Provedores de Serviços

Service providers are a great way to group related IoC registrations in a single location. In fact, most of the core Laravel components include service providers. All of the registered service providers for your application are listed in the `providers` array of the `app/config/app.php` configuration file.

To create a service provider, simply extend the `Illuminate\Support\ServiceProvider` class and define a `register` method:

**Defining A Service Provider**

	use Illuminate\Support\ServiceProvider;

	class FooServiceProvider extends ServiceProvider {

		public function register()
		{
			$this->app->bind('foo', function()
			{
				return new Foo;
			});
		}

	}

Note that in the `register` method, the application IoC container is available to you via the `$this->app` property. Once you have created a provider and are ready to register it with your application, simply add it to the `providers` array in your `app` configuration file.
