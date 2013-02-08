# Eventos

- [Uso Básico](#basic-usage)
- [Usando Classes como Listeners(Ouvintes)](#using-classes-as-listeners)
- [Subscribers(Assinantes) de Eventos](#event-subscribers)

<a name="basic-usage"></a>
## Uso Básico

A Classe `Event` do Laravel fornece uma simples implementação de observador, permitindo você subscribe(assinar) e listen(ouvir) eventos na sua aplicação.

**Assinando Um Evento**

	Event::listen('user.login', function($user)
	{
		$user->last_login = new DateTime;

		$user->save();
	});

**Disparando Um Evento**

	$event = Event::fire('user.login', array($user));

Você podem também especificar a prioridade quando subscribing(assinar) eventos. Listeners(ouvintes) que tem alta prioridade serão executados primeiro, enquanto listeneres(ouvintes) que possuem a mesma prioridade serão executados pela ordem de subscription(assinatura).

**Assinando Eventos Com Prioridade**

	Event::listen('user.login', 'LoginHandler', 10);

	Event::listen('user.login', 'OtherHandler', 5);

Algumas vezes, você pode desejar interromper a propagação de um evento ou outro. Você pode fazer apensar usando o método `$event->stop()`.

**Interrompendo A Propagação De Um Evento**

	Event::listen('user.login', function($event)
	{
		// Handle the event...

		return false;
	});

<a name="using-classes-as-listeners"></a>
## Usando Classes como Listeners(Ouvintes)

Em alguns casos, você queira usar uma classe como manipuladora de um evento do que uma Closure. Classes de listeners(ouvintes) de evento fora do [Laravel IoC container](/docs/ioc), fornecendo um completo poder de injeção de dependências para os seus listeners(ouvintes).

**Registrando Uma Classe Listener(Ouvinte)**

	Event::listen('user.login', 'LoginHandler');

Por padrão, o método `handle` na classe `LoginHandler` será chamada:

**Definindo Uma Classe De Event Listener(Ouvinte)**

	class LoginHandler {

		public function handle($data)
		{
			//
		}

	}

Se você não quiser o método `handle` como padrão, especifique o método que deverá ser subscribed(assinado):

**Especificando Que Método Será Subscribe(assinado)**

	Event::listen('user.login', 'LoginHandler@onLogin');

<a name="event-subscribers"></a>
## Subscribers(Assinantes) de Eventos

Subscribers(Assinantes) de eventos são classes que podem subscribe(assinar) mútiplos eventos de dentro da própria classe. Subscribers devem possuir um método `subscribe`, quer irá ser passado a uma instância despachadora do evento:

**Definindo Um Subscriber(assinante) de Evento**

	class UserEventHandler {

		/**
		 * Handle user login events.
		 */
		public function onUserLogin($event)
		{
			//
		}

		/**
		 * Handle user logout events.
		 */
		public function onUserLogout($event)
		{
			//
		}

		/**
		 * Register the listeners for the subscriber.
		 * @param  Illuminate\Events\Dispatcher  $events
		 * @return array
		 */
		public static function subscribes($events)
		{
			$events->listen('user.login', 'UserEventHandler@onUserLogin');
 	 		$events->listen('user.logout', 'UserEventHandler@onUserLogout');
		}

	}

Uma vez que a subscriber(assinatura) foi definida, ela pode ser registrada com a classe `Event`.

**Registrando Um Subscriber(assinante) de Evento**

	$subscriber = new UserEventHandler();
	Event::subscribe($subscriber);
