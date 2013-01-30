# Validação

- [Uso Básico](#basic-usage)
- [Trabalhando com Mensagens de Erro](#working-with-error-messages)
- [Mensagens de Erro & Views](#error-messages-and-views)
- [Regras de Validações Disponíveis](#available-validation-rules)
- [Mensagens de Erro Personalizadas](#custom-error-messages)
- [Regras de Erro Personalizadas](#custom-validation-rules)

<a name="basic-usage"></a>
## Uso Básico

Lavarel vem com métodos simples e fáceis para validar e recuperar mensagens de erro através da classe `Validation`.

**Exemplo Básico de Validação**

    $validator = Validator::make(
        array('name' => 'Dayle'),
        array('name' => 'required|min:5')
    );

O primeiro argumento passado para o método `make` sãos os dados a ser validados. O segundo argumento são as regras de validação que serão aplicados aos dados.

Várias regras devem ser delimitadas usando o caractere "pipe", ou como elementos separados em um array.

**Usando Arrays Para Especificar Regras**

    $validator = Validator::make(
        array('name' => 'Dayle'),
        array('name' => array('required', 'min:5'))
    );

Quando uma instância `Validator` for criar, o método `fails` (ou `passes`) pode ser usado para realizar a validação.

    if ($validator->fails())
    {
        // The given data did not pass validation
    }

Se a validação falhar, você receberá mensagens de erro de validação.

    $messages = $validator->messages();

**Validação Arquivos**

A classe `Validator` fornece muitas regras para validar arquivos, como `size`, `mimes`, e outros. Ao validar arquivos, você pode simplesmente passá-los no validador com os outros dados.

<a name="working-with-error-messages"></a>
## Trabalhando com Mensagens de Erro

Depois de chamar o método `messages` de uma instância `Validator`, você receberá uma instância de `MessageBag`, que tem uma variedade de métodos convenientes para o trabalho com as mensagens de erro.

**Recuperando A Primeira Mensagem De Erro De Um Campo**

    echo $messages->first('email');

**Recuperando Todas As Mensagens de Erro De Um Campo**

    foreach ($messages->get('email') as $message)
    {
        //
    }

**Recuperando Todas As Mensagens De Erros De Todos Os Campos**

    foreach ($messages->all() as $message)
    {
        //
    }

**Determinando Se Uma Mensagem De Erro De Um Campo Existe**

    if ($messages->has('email'))
    {
        //
    }

**Recuperando Uma Mensagem De Erro Formatada**

    echo $messages->first('email', '<p>:message</p>');

> **Nota:** Por padrão, as mensagens são formatdas usando sintaxe compatível com o Bootstrap.

**Recuperando Todas As Mensagens De Erro Com Um Formato**

    foreach ($messages->all('<li>:message</li>') as $message)
    {
        //
    }

<a name="error-messages-and-views"></a>
## Mensagens de Erro & Views

Depois de ter realizado a validação, você vai precisar de uma maneira fácil de obter as mensagens de erro de volta em suas views. Isto é convenientemente tratado por Laravel. Considere as seguintes rotas como exemplo:

    Route::get('register', function()
    {
        return View::make('user.register');
    });

    Route::post('register', function()
    {
        $rules = array(...);

        $validator = Validator::make(Input::all(), $rules);

        if ($validator->fails())
        {
            return Redirect::to('register')->withErrors($validator);
        }
    });

Observe que quando a validação falhar, nós passamos uma instância de `Validator` para o Redirect(redirecionamento) usando o método `withErrors`. Esse método irá guardar as mensagens de erro na sessão para então estar disponíveis na próxima requisição.

No entanto, observe que não temos de chamar explicitamente as mensagens de erro para a view da nossa rota GET. Isto porque Laravel sempre verifica se há erros nos dados da sessão e, automaticamente, vinculá-os na view, se estiverem disponíveis.  **Assim, é importante notar que a variável `$errors` estará sempre disponível nas suas views, em cada requisição**, permitindo você convenientemente que a variável `$errors` é sempre definida e pode ser usado com segurança. A variável `$errors` será uma instância de `MessageBag`.

Sendo assim, depois de redirecionar, utilize automaticamente a variárel `$errors` vinculada na sua view:

    <?php echo $errors->first('email'); ?>

<a name="available-validation-rules"></a>
## Regras de Validações Disponíveis

Segue uma lista de todas as regras de validação disponíveis e suas funções:

- [Aceito](#rule-accepted)
- [URL Ativa](#rule-active-url)
- [Depois (Data)](#rule-after)
- [Alfa(Alfabeto)](#rule-alpha)
- [Alfa Traço](#rule-alpha-dash)
- [Alfa Numérico](#rule-alpha-num)
- [Antes (Data)](#rule-before)
- [Entre](#rule-between)
- [Confirmado](#rule-confirmed)
- [Diferente](#rule-different)
- [E-Mail](#rule-email)
- [Existe (Banco de Dados)](#rule-exists)
- [Imagem (Arquivo)](#rule-image)
- [In(contém)](#rule-in)
- [Inteiro](#rule-integer)
- [Endereço de IP](#rule-ip)
- [Máximo](#rule-max)
- [MIME Types](#rule-mimes)
- [Mínimo](#rule-min)
- [Númerico](#rule-numeric)
- [Expressão Regular](#rule-regex)
- [Obrigatório](#rule-required)
- [Obrigatório com](#rule-required-with)
- [Igual](#rule-same)
- [Tamanho](#rule-size)
- [Único (Banco de Dados)](#rule-unique)
- [URL](#rule-url)

<a name="rule-accepted"></a>
#### accepted(aceito)

O campo em fase de validação deve ser _yes_, _on_, ou _1_. Isto é útil para a validação de aceitação de "Termos de Serviço".

<a name="rule-active-url"></a>
#### active_url(url ativa)

O campo em fase de validação deve ser uma URL válida de acordo com a função `checkdnsrr` do PHP.

<a name="rule-after"></a>
#### after:_date_(depois, data)

O campo em fase de validação deve ser um valor anterior a uma data informada. As datas serão passadas para a função `strtotime` do PHP.

<a name="rule-alpha"></a>
#### alpha (alfabeto)

O campo em fase de validação deve ser totalmente com caractéres do alfabeto.

<a name="rule-alpha-dash"></a>
#### alpha_dash (alfabeto e traços)

O campo em fase de validação deve possuir caracteres alfa-numéricos, e traços e underscores.

<a name="rule-alpha-num"></a>
#### alpha_num (alfa-numérico)

O campo em fase de validação deve possuir caracteres alfa-numéricos

<a name="rule-before"></a>
#### before:_date_ (antes, data)

O campo em fase de validação deve ser um valor anterior a data informada. As datas serão passadas para a função `strtotime` do PHP.

<a name="rule-between"></a>
#### between:_min_,_max_ (entre, minimo, máximo)

O campo em fase de validação deve ter um tamanho entre um minimo e máximo. Strings, números, e arquivos são avaliados da mesma forma da regra `size`.

<a name="rule-confirmed"></a>
#### confirmed (confirmado)

O campo em fase de validação deve ter um campo de casamento. Exemplo, se você tem um campo `senha` a ser validado, um campo de casamento `senha_confirmation` deve estar presente na entrada.

<a name="rule-different"></a>
#### different:_field_ (diferente)

O campo recebido _field_ deve ser diferente do campo a ser validado.

<a name="rule-email"></a>
#### email

O campo em fase de validação deve ter o formato de endereço de email.

<a name="rule-exists"></a>
#### exists:_table_,_column_ (existe, tabela e coluna)

O campo em fase de validação deve existir na tabela informada.

**Uso Básico Da Regra Exists**

    'state' => 'exists:states'

**Especificando Nomes De Colunas Personalizas**

    'state' => 'exists:states,abbreviation'

<a name="rule-image"></a>
#### image (imagem)

O campo em fase de validação deve uma imagem (jpeg, png, bmp, or gif)

<a name="rule-in"></a>
#### in:_foo_,_bar_,... (contém)

O campo em fase de validação deve estar incluso numa lista de valores informada.

<a name="rule-integer"></a>
#### integer

O campo em fase de validação deve ter um valor inteiro.

<a name="rule-ip"></a>
#### ip

O campo em fase de validação deve estar no formato de endereço de IP.

<a name="rule-max"></a>
#### max:_value_ (máximo)

O campo em fase de validação deve ser menor que o _value_ máximo. Strings, números, e arquivos são avaliados da mesma forma que a regra `size`.

<a name="rule-mimes"></a>
#### mimes:_foo_,_bar_,...

O arquivo em fase de validação deve ter um MIME type correspondente com uma lista de extensões.

**Uso Básico da Regra MIME**

    'photo' => 'mimes:jpeg,bmp,png'

<a name="rule-min"></a>
#### min:_value_ (mínimo)

O campo em fase de validação deve ter um _value_ mínimo. Strings, números, e arquivos são avaliados da mesma forma que a regra `size`.

<a name="rule-numeric"></a>
#### numeric

O campo em fase de validação deve have a numeric value.

<a name="rule-regex"></a>
#### regex:_pattern_

O campo em fase de validação deve match the given regular expression.

**Note:** When using the `regex` pattern, it may be necessary to specify rules in an array instead of using pipe delimiters, especially if the regular expression contains a pipe character.

<a name="rule-required"></a>
#### required

O campo em fase de validação deve be present in the input data.

<a name="rule-required-with"></a>
#### required_with:_foo_,_bar_,...

O campo em fase de validação deve be present _only if_ the other specified fields are present.

<a name="rule-same"></a>
#### same:_field_

The given _field_ must match the field under validation.

<a name="rule-size"></a>
#### size:_value_

O campo em fase de validação deve have a size matching the given _value_. For string data, _value_ corresponds to the number of characters. For numeric data, _value_ corresponds to a given integer value. For files, _size_ corresponds to the file size in kilobytes.

<a name="rule-unique"></a>
#### unique:_table_,_column_,_except_,_idColumn_

O campo em fase de validação deve be unique on a given database table. If the `column` option is not specified, the field name will be used.

**Basic Usage Of Unique Rule**

    'email' => 'unique:users'

**Specifying A Custom Column Name**

    'email' => 'unique:users,email_address'

**Forcing A Unique Rule To Ignore A Given ID**

    'email' => 'unique:users,email_address,10'

<a name="rule-url"></a>
#### url

O campo em fase de validação deve be formatted as an URL.

<a name="custom-error-messages"></a>
## Custom Error Messages

If needed, you may use custom error messages for validation instead of the defaults. There are several ways to specify custom messages.

**Passing Custom Messages Into Validator**

    $messages = array(
        'required' => 'The :attribute field is required.',
    );

    $validator = Validator::make($input, $rules, $messages);

*Note:* The `:attribute` place-holder will be replaced by the actual name of the field under validation. You may also utilize other place-holders in validation messages.

**Other Validation Place-Holders**

    $messages = array(
        'same'    => 'The :attribute and :other must match.',
        'size'    => 'The :attribute must be exactly :size.',
        'between' => 'The :attribute must be between :min - :max.',
        'in'      => 'The :attribute must be one of the following types: :values',
    );

Sometimes you may wish to specify a custom error messages only for a specific field:

**Specifying A Custom Message For A Given Attribute**

    $messages = array(
        'email.required' => 'We need to know your e-mail address!',
    );

In some cases, you may wish to specify your custom messages in a language file instead of passing them directly to the `Validator`. To do so, add your messages to `custom` array in the `app/lang/xx/validation.php` language file.

**Specifying Custom Messages In Language Files**

    'custom' => array(
        'email' => array(
            'required' => 'We need to know your e-mail address!',
        ),
    ),

<a name="custom-validation-rules"></a>
## Custom Validation Rules

Laravel provides a variety of helpful validation rules; however, you may wish to specify some of your own. One method of registering custom validation rules is using the `Validator::extend` method:

**Registering A Custom Validation Rule**

    Validator::extend('foo', function($attribute, $value, $parameters)
    {
        return $value == 'foo';
    });

The custom validator Closure receives three arguments: the name of the `$attribute` being validated, the `$value` of the attribute, and an array of `$parameters` passed to the rule.

Note that you will also need to define an error message for your custom rules. You can do so either using an inline custom message array or by adding an entry in the validation language file.

Instead of using Closure callbacks to extend the Validator, you may also extend the Validator class itself. To do so, write a Validator class that extends `Illuminate\Validation\Validator`. You may add validation methods to the class by prefixing them with `validate`:

**Extending The Validator Class**

    <?php

    class CustomValidator extends Illuminate\Validation\Validator {

        public function validateFoo($attribute, $value, $parameters)
        {
            return $value == 'foo';
        }

    }

Next, you need to register your custom Validator extension:

**Registering A Custom Validator Resolver**

    Validator::resolver(function()
    {
        return new CustomValidator;
    });

When creating a custom validation rules, you may sometimes need to define custom place-holder replacements for error messages. You may do so by creating a custom Validator as described above, and adding a `replaceXXX` function to the validator.

    protected function replaceFoo($message, $attribute, $rule, $parameters)
    {
        return str_replace(':foo', $parameters[0], $message);
    }
