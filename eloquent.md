# Eloquent ORM

- [Introducão](#introduction)
- [Uso Básico](#basic-usage)
- [Insert, Update, Delete](#insert-update-delete)
- [Timestamps](#timestamps)
- [Relacionamentos](#relationships)
- [Carregamento Ansioso](#eager-loading)
- [Inserindo em Modelos Relacionados](#inserting-related-models)
- [Trabalhando com Tabelas Dinâmicas](#working-with-pivot-tables)
- [Coleções](#collections)
- [Acessadores & Modificadores](#accessors-and-mutators)
- [Atribuição em Massa](#mass-assignment)
- [Covertendo para Arrays / JSON](#converting-to-arrays-or-json)

<a name="introduction"></a>
## Introdução

O Eloquent ORM incluso no Laravel fornece uma bonita e simples implementação ActiveRecord para trabalhar com seu banco de dados. Cada tabela do banco de dados tem um "Model" correspondente que será usado para interagir com a tabela.

Antes de iniciarmos, certifique se de configurar uma conexão com o banco de dados no `app/config/database.php`.

<a name="basic-usage"></a>
## Uso Básico

Para começar, crie um modelo Eloquent. Modelos ficam armazenados no diretório `app/models`, mas você é livre para colocar em qualquer lugar que você quiser desde que possam ser automaticamente carregados de acordo com seu arquivo `composer.json`.

**Definindo um Modelo Eloquent**

	class User extends Eloquent {}

Note que não informamos ao Eloquent que tabela o modelo `User` irá usar. O nome da classe em letra minúscula e no plural, será usado como o nome da tabela, a menos que outro nome seja especificado. Sendo assim, o Eloquent assume que modelo `User` armazena os registros na tabela `users`. Especifique a propriedade `table` no seu modelo, caso queira usar um nome personalizado para a tabela:

	class User extends Eloquent {

		protected $table = 'my_users';

	}

> **Nota:** Eloquent irá assumir também, que cada tabela possui uma chave primária com o nome `id`. Caso queira usar outro nome, defina a propriedade `primaryKey`.

Uma vez que o modelo está definido, você está pronto para começar a recuperar e criar registros na sua tabela. Note que você vai precisar colocar as colunas `updated_at` e `created_at` em sua tabela por padrão. Se você não deseja ter essas colunas mantidas automaticamente, defina a propriedade `$timestamps` no seu modelo para `false`

**Recuperando Todos Os Modelos(Registros)**

	$users = User::all();

**Recuperando Um Registro Através Da Chave Primária**

	$user = User::find(1);

	var_dump($user->name);

> **Nota:** Todos os métodos disponíveis no [construtor de consultas](/docs/queries) também estão disponíveis ao consultar modelos do Eloquent.

**Consultando Usando Modelos Eloquent**

	$users = User::where('votes', '>', 100)->take(10)->get();

	foreach ($users as $user)
	{
		var_dump($user->name);
	}

Claro, você pode também usar funções agregadoras do construtor de consultas.

**Agregadores No Eloquent**

	$count = User::where('votes', '>', 100)->count();

<a name="insert-update-delete"></a>
## Insert, Update, Delete

Para criar um novo registro em seu banco de dados por um modelo, simplesmente crie uma nova instância do modelo e chame o método `save`.

**Salvando Um Novo Modelo**

	$user = new User;

	$user->name = 'John';

	$user->save();

Você pode também usar o método `create` para salvar um novo modelo em uma só linha. A instância do modelo inserido será retornada do método:

**Usando O Método Create Do Modelo**

	$user = User::create(array('name' => 'John'));

Para atualizar um modelo, você o recupera, muda o atributo, e usa o método `save`:

**Atualizando Um Modelo Recuperado**

	$user = User::find(1);

	$user->email = 'john@foo.com';

	$user->save();

Você pode executar atualizações encadeando de consultas dos modelos:

	$affectedRows = User::where('votes', '>', 100)->update(array('status' => 2));

Para apagar um modelo, simplesmente chame o método `delete` method em uma instância:

**Apagando Um Modelo Existente**

	$user = User::find(1);

	$user->delete();

Sim, você pode apagar encadando de uma consulta do modelo:

	$affectedRows = User::where('votes', '>', 100)->delete();

<a name="timestamps"></a>
## Timestamps

Por padrão, Eloquent mantém as colunas `created_at` e `updated_at` nas tabelas do seu banco de dados, automaticamente. Basta adicionar essas colunas `datetime` para a sua tabela e o Eloquent vai cuidar do resto. Se você não deseja que o Eloquent mantenha essas colunas, defina a propriedade:

**Disabilitando Auto Timestamps**

	class User extends Eloquent {

		protected $table = 'users';

		public $timestamps = false;

	}

Se desejar personalizar o formato de seus timestamps, sobrescreva o método `freshTimestamp` em seu modelo:

**Fornecendo Um Formato Personalizado Para Timestamp**

	class User extends Eloquent {

		public function freshTimestamp()
		{
			return time();
		}

	}

<a name="relationships"></a>
## Relacionamentos

Suas tabelas do banco de dados provavelmente estão relacionadas a alguma outra. Por exemplo, um publicação de blog podem ter muitos comentários, ou pode estar relacionado ao usuário que publicou. Eloquent faz o gerenciamento e trabalho com essas relações, fácil. Laravel suporta quatro tipos de relações:

- [Um Para Um](#one-to-one)
- [Um Para Muitos](#one-to-many)
- [Muitos Para Muitos](#many-to-many)
- [Relacionamentos Polimórficos](#polymorphic-relations)

<a name="one-to-one"></a>
### Um para Um

Um relacionamento um-para-um é uma relação muita básica. Exemplo, um modelo `User` pode ter um `Phone`. Podemos definir essa relação no Eloquent:

**Definindo Uma Relação Um Para Um**

	class User extends Eloquent {

		public function phone()
		{
			return $this->hasOne('Phone');
		}

	}

O primeiro argumento passado para o método `hasOne` é o nome do modelo relacionado. Assim que relação é definida, podemos recuperar isso usando propriedades dinâmicas do Eloquent:

	$phone = User::find(1)->phone;

O Sql realizado por essa instrução será como a seguinte:

	select * from users where id = 1

	select * from phones where user_id = 1

Note que Eloquent assume que a chave estrangeira do relacionamento é baseada no nome do modelo. No exemplo, o modelo `Phone` assume que a chave estrangeira usada é `user_id`. Se desejar sobrescrever essa convenção, passe um segundo argumento para o método `hasOne`:

	return $this->hasOne('Phone', 'chave_personalizada');

Para definir o inverso, a relação no modelo `Phone`, usamos o método `belongsTo`:

**Definindo O Inverso De Uma Relação**

	class Phone extends Eloquent {

		public function user()
		{
			return $this->belongsTo('User');
		}

	}

<a name="one-to-many"></a>
### Um para Muitos

Um exemplo de relacionamento um-para-muitos é um publicação de blog que "tem muitos" comentários. Podemos modelar essa relação:

	class Post extends Eloquent {

		public function comments()
		{
			return $this->hasMany('Comment');
		}

	}

Agora podemos acessar os comentários da publicação através da propriedade dinâmica:

	$comments = Post::find(1)->comments;

If you need to add further constraints to which posts are retrieved, you may call the `comments` method and continue chaining conditions:

	$comments = Post::find(1)->comments()->where('title', '=', 'foo')->first();

Novamente, você pode sobrescrever a convencional chave estrangeira passando um segundo argumento ao método `hasMany`:

	return $this->hasMany('Comment', 'custom_key');

Para definir o inverso dessa relação no modelo `Comment`, nos usámos o método `belongsTo`:

**Definindo O Inverso De Uma Relação**

	class Comment extends Eloquent {

		public function post()
		{
			return $this->belongsTo('Post');
		}

	}

<a name="many-to-many"></a>
### Many To Many

Relações muitos-para-muitos é o tipo de relacionamento mais complicado. Um exemplo de um relacionamento é um usuário com muitos papéis(roles), onde os papéis também compartilhados por outros usuários. Exemplo, muitos usuários tem o papel de "Admin". Três tabelas são necessárias para este relacionamento: `users`, `roles`, e `role_user`. A tabela `role_user` é derivada da ordem alfabética dos nomes dos modelos relacionados, e devem ter as colunas `user_id` e `role_id`.

Podemos definir relações muitos-para-muitos usando o método `belongsToMany`:

	class User extends Eloquent {

		public function roles()
		{
			return $this->belongsToMany('Role');
		}

	}

Agora podemos recuperar os papéis(roles) através do modelo `User`:

	$roles = User::find(1)->roles;

Se você gostaria de usar um nome de tabela não convencional para a sua tabela dinâmica, você pode passar como um segundo argumento para o método `belongsToMany`:

	return $this->belongsToMany('Role', 'user_roles');

Você pode sobrescrever a convenção de chaves associadas:

	return $this->belongsToMany('Role', 'user_roles', 'user_id', 'foo_id');

<a name="polymorphic-relations"></a>
### Relacionamentos Polimórficos

Relacionamentos Polimorficos permitem um modelo pertecer a mais de um modelo numa simples associação. Exemplo, você pode ter um modelo photo(foto) que pertence a um modelo staff(equipe) ou a um modelo order(pedido). Definiriamos essa relação assim:

	class Photo extends Eloquent {

		public function imageable()
		{
			return $this->morphTo();
		}

	}

	class Staff extends Eloquent {

		public function photos()
		{
			return $this->morphMany('Photo', 'imageable');
		}

	}

	class Order extends Eloquent {

		public function photos()
		{
			return $this->hasMany('Photo', 'imageable');
		}

	}

Agora, podemos recuperar as photos(fotos) de um membro da staff(equipe) ou de um order(pedido):

**Recuperando Um Relacionamento Polimórfico**

	$staff = Staff::find(1);

	foreach ($staff->photos as $photo)
	{
		//
	}

Mas, a verdadeira magia do "polimórfico" é quando você acessa staff(equipe) ou order(pedido) do modelo `Photo`:

**Recuperando O Dono De Um Relacionamento Polimórfico**

	Photo::find(1);

	$imageable = $photo->imageable;

A relação `imageable` no modelo `Photo` retornará uma instância de `Staff` ou `Order`, dependendo de qual tipo é o dono do modelo photo.

Para ajudar a entender como isso funciona, vamos explorar a estrutura do banco de dados dessa relação polimórfica:

**Estruturas da Tabela da Relação Polimórfica**

	staff
		id - integer
		name - string

	orders
		id - integer
		price - integer

	photos
		id - integer
		path - string
		imageable_id - integer
		imageable_type - string

Os campos chave para observar aqui são `imageable_id` e `imageable_type` na tabela `photos`. O imageable_id conterá o valor do ID, que neste exemplo, será o staff ou order, enquanto que imageable_type irá conter o nome da classe dona do modelo. Isso é o que permite o ORM determinar que tipo possui o modelo para retornar quando acessar a relação `imageable`.

<a name="eager-loading"></a>
## Carregamento Ansioso

Eager loading exists to alleviate the N + 1 query problem. For example, consider a `Book` model that is related to `Author`. The relationship is defined like so:

	class Book extends Eloquent {

		public function author()
		{
			return $this->belongsTo('Author');
		}

	}

Now, consider the following code:

	foreach (Book::all() as $book)
	{
		echo $book->author->name;
	}

This loop will execute 1 query to retrieve all of the books on the table, then another query for each book to retrieve the author. So, if we have 25 books, this loop would run 26 queries.

Thankfully, we can use eager loading to drastically reduce the number of queries. The relationships that should be eager loaded may be specified via the `with` method:

	foreach (Book::with('author')->get() as $book)
	{
		echo $book->author->name;
	}

In the loop above, only two queries will be executed:

	select * from books

	select * from authors where id in (1, 2, 3, 4, 5, ...)

Wise use of eager loading can drastically increase the performance of your application.

Of course, you may eager load multiple relationships at one time:

	$books = Book::with('author', 'publisher')->get();

You may even eager load nested relationships:

	$books = Book::with('author.contacts')->get();

In the example above, the `author` relationship will be eager loaded, and the author's `contacts` relation will also be loaded.

### Eager Load Constraints

Sometimes you may wish to eager load a relationship, but also specify a condition for the eager load. Here's an example:

	$users = User::with(array('posts' => function($query)
	{
		$query->where('title', 'like', '%first%');
	}))->get();

In this example, we're eager loading the user's posts, but only if the post's title column contains the word "first".

### Lazy Eager Loading

It is also possible to eagerly load related models directly from an already existing model collection. This may be useful when dynamically deciding whether to load related models or not, or in combination with caching.

	$books = Book::all();

	$books->load('author', 'publisher');

<a name="inserting-related-models"></a>
## Inserindo em Modelos Relacionados

You will often need to insert new related models. For example, you may wish to insert a new comment for a post. Instead of manually setting the `post_id` foreign key on the model, you may insert the new comment from its parent `Post` model directly:

**Attaching A Related Model**

	$comment = new Comment(array('message' => 'A new comment.'));

	$post = Post::find(1);

	$comment = $post->comments()->save($comment);

In this example, the `post_id` field will automatically be set on the inserted comment.

### Inserting Related Models (Many To Many)

You may also insert related models when working with many-to-many relations. Let's continue using our `User` and `Role` models as examples. We can easily attach new roles to a user using the `attach` method:

**Attaching Many To Many Models**

	$user = User::find(1);

	$user->roles()->attach(1);

You may also pass an array of attributes that should be stored on the pivot table for the relation:

	$user->roles()->attach(1, array('expires' => $expires));

You may also use the `sync` method to attach related models. The `sync` method accepts an array of IDs to place on the pivot table. After this operation is complete, only the IDs in the array will be on the intermediate table for the model:

**Using Sync To Attach Many To Many Models**

	$user->roles()->sync(array(1, 2, 3));

Sometimes you may wish to create a new related model and attach it in a single command. For this operation, you may use the `save` method:

	$role = new Role(array('name' => 'Editor'));

	User::find(1)->roles()->save($role);

In this example, the new `Role` model will be saved and attached to the user model. You may also pass an array of attributes to place on the joining table for this operation:

	User::find(1)->roles()->save($role, array('expires' => $expires));

<a name="working-with-pivot-tables"></a>
## Trabalhando com Tabelas Dinâmicas

As you have already learned, working with many-to-many relations requires the presence of an intermediate table. Eloquent provides some very helpful ways of interacting with this table. For example, let's assume our `User` object has many `Role` objects that it is related to. After accessing this relationship, we may access the `pivot` table on the models:

	$user = User::find(1);

	foreach ($user->roles as $role)
	{
		echo $role->pivot->created_at;
	}

Notice that each `Role` model we retrieve is automatically assigned a `pivot` attribute. This attribute contains a model representing the intermediate table, and may be used as any other Eloquent model.

By default, only the keys will be present on the `pivot` object. If your pivot table contains extra attributes, you must specify them when defining the relationship:

	return $this->belongsToMany('Role')->withPivot('foo', 'bar');

Now the `foo` and `bar` attributes will be accessible on our `pivot` object for the `Role` model.

If your want your pivot table to have automatically maintained `created_at` and `updated_at` timestamps, use the `withTimestamps` method on the relationship definition:

	return $this->belongsToMany('Role')->withTimestamps();

To delete all records on the pivot table for a model, you may use the `delete` method:

**Deleting Records On A Pivot Table**

	User::find(1)->roles()->delete();

Note that this operation does not delete records from the `roles` table, but only from the pivot table.

<a name="collections"></a>
## Coleções

All multi-result sets returned by Eloquent either via the `get` method or a relationship return an Eloquent `Collection` object. This objects implements the `IteratorAggregate` PHP interface so it can be iterated over like an array. However, this object also has a variety of other helpful methods for working with result sets.

For example, we may determine if a result set contains a given primary key using the `contains` method:

**Checking If A Collection Contains A Key**

	$roles = User::find(1)->roles;

	if ($roles->contains(2))
	{
		//
	}

Collections may also be converted to an array or JSON:

	$roles = User::find(1)->roles->toArray();

	$roles = User::find(1)->roles->toJson();

If a collection is cast to a string, it will be returned as JSON:

	$roles = (string) User::find(1)->roles;

Sometimes, you may wish to return a custom Collection object with your own added methods. You may specify this on your Eloquent model by overriding the `newCollection` method:

**Returning A Custom Collection Type**

	class User extends Eloquent {

		public function newCollection(array $models = array())
		{
			return new CustomCollection($models);
		}

	}

<a name="accessors-and-mutators"></a>
## Acessadores & Modificadores

Eloquent provides a convenient way to transform your model attributes when getting or setting them. Simplify define a `getFoo` method on your model to declare an accessor. Keep in mind that the methods should follow camel-casing, even though your database columns are snake-case:

**Defining An Accessor**

	class User extends Eloquent {

		public function giveFirstName($value)
		{
			return ucfirst($value);
		}

	}

In the example above, the `first_name` column has an accessor. Note that the value of the attribute is passed to the accessor.

Mutators are declared in a similar fashion:

**Defining A Mutator**

	class User extends Eloquent {

		public function takeFirstName($value)
		{
			$this->attributes['first_name'] = strtolower($value);
		}

	}

<a name="mass-assignment"></a>
## Atribuição em Massa

When creating a new model, you pass an array of attributes to the model constructor. These attributes are then assigned to the model via mass-assignment. This is convenient; however, can be a **serious** security concern when blindly passing user input into a model. If user input is blindly passed into a model, the user is free to modify **any** and **all** of the model's attributes.

A more secure approach to assigning attributes is either to manually assign them, or to set the `fillable` or `guarded` properties on your model.

The `fillable` property specifies which attributes should be mass-assignable. This can be set at the class or instance level.

**Defining Fillable Attributes On A Model**

	class User extends Eloquent {

		protected $fillable = array('first_name', 'last_name', 'email');

	}

In this example, only the three listed attributes will be mass-assignable.

The inverse of `fillable` is `guarded`, and serves as a "black-list" instead of a "white-list":

**Defining Guarded Attributes On A Model**

	class User extends Eloquent {

		protected $guarded = array('id', 'password');

	}

In the example above, the `id` and `password` attributes may **not** be mass assigned. All other attributes will be mass assignable. You may also block **all** attributes from mass assignment using the guard method:

**Blocking All Attributes From Mass Assignment**

	protected $guarded = array('*');

<a name="converting-to-arrays-or-json"></a>
## Covertendo para Arrays / JSON

When building JSON APIs, you may often need to convert your models and relationships to arrays or JSON. So, Eloquent includes methods for doing so. To convert a model and its loaded relationship to an array, you may use the `toArray` method:

**Converting A Model To An Array**

	$user = User::with('roles')->first();

	return $user->toArray();

Note that entire collections of models may also be converted to arrays:

	return User::all()->toArray();

To convert a model to JSON, you may use the `toJson` method:

**Converting A Model To JSON**

	return User::find(1)->toJson();

Note that when a model or collection is cast to a string, it will be converted to JSON, meaning you can return Eloquent objects directly from your application's routes!

**Returning A Model From A Route**

	Route::get('users', function()
	{
		return User::all();
	});

Sometimes you may wish to limit the attributes that are included in your model's array or JSON form, such as passwords. To do so, add a `hidden` property definition to your model:

**Hiding Attributes From Array Or JSON Conversion**

	class User extends Eloquent {

		protected $hidden = array('password');

	}
