# Cache

- [Configuração](#configuration)
- [Uso do Cache](#cache-usage)
- [Cache de Banco de Dados](#database-cache)

<a name="configuration"></a>
## Configuração

Laravel fornece uma API unificada para vários tipos de sistema de cache. A configuração do cache encontra-se em `app/config/cache.php`. Neste arquivo você pode especificar que driver de cache você prefere usar por padrão em toda sua aplicação. Laravel suporta caching backends populares como [Memcached](http://memcached.org) e [Redis](http://redis.io) fora da caixa.

O arquivo de configuração também contem varias outras opções, que são documentadas no próprio arquivo, então certifique-se de ler sobre essas opções. Por padrão, Laravel é configurado para usar o cache driver `file`, que armazena, serializado, o objetos de cache no sistema de arquivos. Para grandes aplicações, recomenda-se que você use um cache em memória como o Memcached ou APC.

<a name="cache-usage"></a>
## Uso do Cache

**Armazenando Um Item No Cache**

	Cache::put('key', 'value', $minutes);

**Recuperando Um Item Do Cache**

	$value = Cache::get('key');

**Recuperando Um Item Ou Retornando Um Valor Padrão**

	$value = Cache::get('key', 'default');

	$value = Cache::get('key', function() { return 'default'; });

**Armazenando Um Item No Cache Permanentemente**

	Cache::forever('key', 'value');

Algumas vezes você pode querer recuperar um item do cache, mas Sometimes you may wish to retrieve an item from the cache, but also store a default value if the requested item doesn't exist. You may do this using the `Cache::remember` method:

	$value = Cache::remember('users', $minutes, function()
	{
		return DB::table('users')->get();
	});

You may also combine the `remember` and `forever` methods:

	$value = Cache::rememberForever('users', function()
	{
		return DB::table('users')->get();
	});

Note that all items stored in the cache are serialized, so you are free to store any type of data.

**Removing An Item From The Cache**

	Cache::forget('key');

<a name="database-cache"></a>
## Database Cache

When using the `database` cache driver, you will need to setup a table to contain the cache items. Below is an example `Schema` declaration for the table:

	Schema::create('cache', function($t)
	{
		$t->string('key')->unique();
		$t->text('value');
		$t->integer('expiration');
	});
