====================
Capítulo 4: Templates
====================

No capítulo anterior, você deve ter notado algo peculiar em como nós retornamos o texto em nossos exemplos de views. Isto é, o HTML foi codificado diretamente em nosso código Python, tal como::

    def current_datetime(request):
        now = datetime.datetime.now()
        html = "<html><body>Agora é %s.</body></html>" % now
        return HttpResponse(html)

Embora essa técnica seja conveniente para o propósito de demonstrar como as views trabalham, não é uma boa idéia codificar HTML diretamente em suas views. Aqui está o por que:

* Qualquer modificação no design da página requer modificação no código Python.
  O design de um site tende a mudar com mais freqüência do que o código Python subjacente, por isso seria coveniente se o design pude-se ser modificado sem precisar modificar o código Python.

* Escrever código Python e design HTML são duas diciplinas diferentes,
  e a maioria dos ambientes de desenvolvimento Web professional dividem essas
  responsabilidades entre pessoas distintas (ou departamentos distintos).
  Designers e codificadores HTML/CSS não devem necessitar editar código Python
  para fazer o seu trabalho.

* É mais eficiente se programadores trabalharem em código Python e designers
  em templates, ao mesmo tempo, ao em vez de uma pessoa esperar a outra
  terminar a edição de um simples arquivo que contém código Python e HTML.

Por essas razões, é muito mais limpo e sustentável separar o design da página
do próprio código Python. Nós podemos fazer isso com o *sistema de template* do Django,
que discutiremos neste capítulo.

Sistema básico de template
==========================

Um template Django é uma seqüência de texto que se destina a separar a
apresentação de um documento a partir dos seus dados. Um template define espaços
reservados e vários pedaços básicos de lógica (template tags) que determinam a forma
como o documento deve ser exibido. Normalmente, templates são usados para gerar HTML,
mas os templates do Django são capazes de gerar qualquer formato baseado em texto.

Vamos iniciar com um exemplo simples de template. Esse template Django descreve uma
página HTML que agradece a uma pessoa por realizar um pedido de uma empresa. Imagine
isso como uma carta formulário::

    <html>
    <head><title>Ordem</title></head>

    <body>

    <h1>Ordem</h1>

    <p>Prezado {{ person_name }},</p>

    <p>Obrigado por fazer o pedido de {{ company }}. Está agendado para enviar em {{ ship_date|date:"F j, Y" }}.</p>

    <p>Aqui estão os itens que você encomendou::</p>

    <ul>
    {% for item in item_list %}
        <li>{{ item }}</li>
    {% endfor %}
    </ul>

    {% if ordered_warranty %}
        <p>A sua informação de garantia será incluída na embalagem.</p>
    {% else %}
        <p>Você não solicitou garantia, sendo assim é por sua conta quando o produto parar de funcionar.</p>
    {% endif %}

    <p>Sinceramente,<br />{{ company }}</p>

    </body>
    </html>

Este template é um HTML básico com algumas variáveis e seus template tags jogado os
valores para dentro. Vamos análisa-lo:

* Qualquer texto cercado por um par de chaves (e.g., ``{{ person_name }}``) é
  uma *váriavel*. Isto significa "insira o valor da váriavel com o nome dado."
  (Como podemos especificar os valores das váriaveis? Nós vamos chegar nisso em breve.)

* Qualquer texto contido entre chaves e sinais de porcento (e.g., ``{% if
  ordered_warranty %}``) é um *template tag*. A definição de tag é bastante
  amplo: uma tag apenas diz ao sistema de template para "fazer alguma coisa".

  Este exemplo de template contem uma tag ``for`` (``{% for item in item_list %}``)
  e uma tag ``if`` (``{% if ordered_warranty %}``).

  Uma tag ``for`` trabalha de forma semelhante a declaração ``for`` em Python,
  permitindo você fazer um laço sobre cada item em uma seqüência. Uma tag ``if``,
  como você pode esperar, age como uma declaração lógica "if". Neste caso
  particular, a tag verifica se o valor da váriavel ``ordered_warranty`` está
  ``True``. Se sim, o sistema de template exibirá tudo que está entre ``{% if ordered_warranty %}`` e ``{% else %}``. Se não, o sistema de template exibirá
  tudo que está entre ``{% else %}`` e ``{% endif %}``. Perceba que o ``{% else
  %}`` é opcional.

* Finalmente, o segundo parágrafo deste template contém um exemplo de *filtro*,
  sendo a forma mais conveniente de alterar a formatação de uma váriavel.
  Neste exemplo, ``{{ ship_date|date:"F j, Y" }}``, nós estamos passando a váriavel
  ``ship_date`` para o filtro ``date``, dando ao filtro ``date`` os argumentos
  "F j, Y". O filtro ``date`` formata datas no formato passado, como especificado
  pelo argumento. Os filtros são anexados usando o character pipe (``|``), como
  referência aos pipes do Unix.

Cada template Django tem acesso a vários tags e filtros embutidos, muitos dos
quais são discutidos nas sessões que seguem. No apêndice F contém a lista completa
de tags e filtros, e é uma boa idéia você se familiarizar com essa lista, assim
saberá quais as possíbilidades. Também é possível criar os seus próprios filtros
e tags; nós vamos cobrir isso no capítulo 9.


Usando o sistema de templates
=============================

Agora vamos mergulhar no sistema de templates do Django para que você veja como
funciona - mas nós ainda não vamos integrar com as views criadas no capítulo
anterior. Nosso objetivo aqui é mostrar para você como o sistema funciona de
forma idependente do restante do Django. (Dito de outra forma: geralmente você
usará o sistema de template dentro de uma view do Django, mas nós queremos deixar
claro que o sistema de template é somente uma biblioteca Python que você pode usar
em *qualquer lugar*, não somente nas views do Django).

Aqui está a maneira mais básica que você pode usar o sistema de templates do
Django em código Python:

1. Crie um objeto ``Template`` fornecendo  *******the raw template code*******
   como uma string.

2. Chame o método ``render()`` do objeto ``Template`` com um determinado
   conjunto de váriaveis (o *contexto*). Isto retorna  o template completamente
   renderizado como uma string, com todas as váriaveis e template tags
   avaliadas de acordo com o contexto.

Em código, é assim que se parece::

    >>> from django import template
    >>> t = template.Template('Meu nome é {{ name }}.')
    >>> c = template.Context({'name': 'Adrian'})
    >>> print t.render(c)
    Meu nome é Adrian.
    >>> c = template.Context({'name': 'Fred'})
    >>> print t.render(c)
    Meu nome é Fred.

As sessões seguintes descrevem cada etapa com muito mais detalhe.

Criando objetos Template
-------------------------

O caminho mais fácil para criar um objeto ``Template`` é instância-lo diretamente.
A classe ``Template`` está no módulo ``django.template``, e o construtor tem um
argumento, o raw template code. Vamos mergulhar no interpretador interativo do Python
para ver como isto funciona no código.

Apartir do diretorio ``mysite`` criado por ``django-admin.py startproject`` (como
descrito no capítulo 2), digite ``python manage.py shell`` para iniciar o interpretador
interativo.

.. admonition::  Um prompt Python especial

    Se você anteriormente usou Python, você pode estar se perguntando porque
    estamos executando ``python manage.py shell`` ao invés de apenas ``python``.
    Ambos os comandos iniciam o interpretador interativo, mas o comando ``manage.py shell``
    possui uma diferença chave: antes de iniciar o interpretador, ele informa ao Django
    qual arquivo de configuração usar. Muitas partes do Django, incluindo o sistema de
    template, dependem de suas configurações, e você não conseguirá usá-los, a menos
    que o framework saiba quais configurações usar.

    Se você está curioso, aqui está como funciona por detrás das cenas. O Django
    procura por uma variável de ambiente chamada ``DJANGO_SETTINGS_MODULE``, que deve
    ser definido para o caminho de importação do seu ``settings.py``. Por exemplo,
    ``DJANGO_SETTINGS_MODULE`` deve ser definido como ``'mysite.settings'``, assumindo
    que ``mysite`` está no seu caminho Python.

    Quando você executa ``python manage.py shell``, o comando se preocupa em definir
    a variável ``DJANGO_SETTINGS_MODULE`` para você. Nós estamos encorajando você a usar
    ``python manage.py shell`` nestes exemplos, de modo que minimize a quantidade de ajustes e configurações que você deva fazer.

Vamos passar por alguns princípios básicos do sistema de template::

    >>> from django.template import Template
    >>> t = Template('Meu nome é {{ name }}.')
    >>> print t

Se você está seguindo a forma interativa, você vai ver algo como isso::

    <django.template.Template object at 0xb7d5f24c>

O ``0xb7d5f24c`` será diferente toda vez, e isso não é relevante; é algo do
Python (a "identidade" Python do objeto ``Template``, se você precisar saber).

Quando você cria um objeto ``Template``, o sistema de template compila o código
do template cru em uma forma otimizada, pronta para renderização. Mas se o código
do seu template possuir qualquer erro de sintaxe, a chamada de ``Template()`` irá
causar uma exceção ``TemplateSyntaxError``::

    >>> from django.template import Template
    >>> t = Template('{% notatag %}')
    Traceback (most recent call last):
      File "<stdin>", line 1, in ?
      ...
    django.template.TemplateSyntaxError: Invalid block tag: 'notatag'

O termo "block tag" aqui se refere a ``{% notatag %}``. "Block tag" e
"template tag" são sinônimos.

O sistema gera uma exceção ``TemplateSyntaxError`` para qualquer um dos seguintes
casos:

* Tags inválidas
* Argumentos inválidos para tags válidas
* Filtros inválidos
* Argumentos inválidos para filtros válidos
* Sintaxe de template inválido
* Tags não fechadas (para tags que requerem fechamento)

Processando um template
--------------------

Uma vez que você tenha um objeto de ``Template``, você pode passar os
dados, dando-lhe um *contexto*. Um contexto é uma simples definição de
nomes de váriaveis e seus valores associados. Um template usa isto para
popular as váriaveis e avaliar as tags.

Um contexto é representado no Django pela classe ``Context``, a qual está
no módulo ``django.template``. Seu construtor tem um argumento optional:
***a dictionary mapping variable names to variable values***. Chame o método
``render()`` do objeto ``Template`` com o contexto para "preencher" o template::

    >>> from django.template import Context, Template
    >>> t = Template('Meu nome é {{ name }}.')
    >>> c = Context({'name': 'Stephane'})
    >>> t.render(c)
    u'Meu nome é Stephane.'

Uma coisa que devemos salientar, é que o valor de retorno de ``t.render(c)``
é um objeto Unicode -- não uma string normal Python. Você pode tratar isto
pelo uso do ``u`` em frente a string. Django usa objetos Unicode ao invés de
strings normais em seu framework. Se você entende a repercurssão disso, seja
grato pelas coisas sofisticadas que o Django faz nos bastidores para isto funcionar.
Se você não entende a repercussão disso, não se preocupe agora; apenas entenda que
o Unicode do Django torna simples que os seus aplicativos tenham suporte a uma grande variedade de conjuntos de caracteres além do básico "A-Z" da língua Inglesa.

.. admonition:: Dicionários e contextos

   Um dicionário Python é um mapeamento entre chaves conhecidas
   e valores váriaveis. Um ``Context`` é similar ao dicionário, mas
   o ``Context`` possui uma funcionalidade adicional, como descrito
   no capítulo 9.

Nomes de váriaveis devem iniciar com letras (A-Z or a-z)  podem contem
mais letras, digitos, sublinhados e pontos (Pontos são um caso especial, vamos ver em breve). Nomes de váriaves são case sensitive.

Aqui está um exemplo de modelo de compilação e renderização, usando um template
semelhante ao exemplo no início deste capítulo::

    >>> from django.template import Template, Context
    >>> raw_template = """<p>Prezado {{ person_name }},</p>
    ...
    ... <p>Obrigado por fazer o pedido na {{ company }}. Está agendado
    ... para enviar em {{ ship_date|date:"F j, Y" }}.</p>
    ...
    ... {% if ordered_warranty %}
    ... <p>A sua informação de garantia será incluída na embalagem.</p>
    ... {% else %}
    ... <p>Você não solicitou garantia, sendo assim é por sua
    ... conta quando o produto parar de funcionar.</p>
    ... {% endif %}
    ...
    ... <p>Sinceramente,<br />{{ company }}</p>"""
    >>> t = Template(raw_template)
    >>> import datetime
    >>> c = Context({'person_name': 'John Smith',
    ...     'company': 'Outdoor Equipment',
    ...     'ship_date': datetime.date(2009, 4, 2),
    ...     'ordered_warranty': False})
    >>> t.render(c)
    u"<p>Prezado John Smith,</p>\n\n<p>Obrigado por fazer o pedido naa Outdoor
    Equipment. Está agendado\n para enviar em April 2, 2009.</p>\n\n\n<p>Você não \n
    solicitou garantia, sendo assim é por sua\n conta quando o produto
    parar de funcionar.</p>\n\n\n<p>Sinceramente,<br />Outdoor Equipment
    </p>"

Vamos passar as instruções de código uma por vez:

* Primeiro, nós importamos as classes ``Template`` e ``Context``, ambas
  ficam nó módulo ``django.template``.

* Nós salvamos o texto bruto do nosso template na váriavel
  ``raw_template``. Perceba que usamos aspas triplas para definir a string,
  porque envolve várias linhas; em contraste, strings com aspas simples não
  podem ser usadas em multiplas linhas.

* Em seguida, nós criamos o objeto template, ``t``, passando ``raw_template``
  para o construtor da classe ``Template`` .

* Nós importamos o módulo ``datetime`` da biblioteca padrão do Python,
  porque vamos precisar dele na declaração seguinte.

* Depois, criamos um objeto ``Context``, ``c``. O construtor ``Context``
  recebe um dicionário Python, que mapeia os nomes das váriaveis para valores.
  Aqui, por exemplo, nós especificamos que ``person_name`` é  ``'John Smith'``,
  ``company`` é ``'Outdoor Equipment'``, e assim por diante.

* Finalmente, chamamos o método ``render()`` em seu objeto template, passando
  o contexto. Este retorna o template renderizado, ou seja, ele substitui
  as váriaveis do template com os valores reais das váriaveis, e executa
  as tags de template.

  Note que o páragrafro "Você não solicitou garantia" é exibido porque
  a váriavel ``ordered_warranty`` tem seu valor como ``False``. Além
  disso, observer a data, ``April 2, 2009``, que é exibido de acordo com
  o formato da string ``'F j, Y'``. (Vamos explicar a formatação de strings
  para os filtros ``date`` em breve).

  Se você é novo com Python, você deve estar se perguntado porque incluir
  caracteres de nova linha(``'\n'``) ao invés de exibir as quebras de linhas.
  Isso está acontecendo por causa de uma detalhe no interpretador interativo
  do Python: a chamada para ``t.render(c)``, retorna uma string, e por padrão
  o interpretador interativo exibe a *representação* da string, ao invés do
  valor impresso na string. Se deseja ver a string com quebras de linha
  verdadeiramente, ao invés de dos caracteres ``'\n'`` , use a declaração
  ``print`` : ``print t.render(c)``.

Esses são os fundamentos para usar o sistema de templates do Django: basta
escrever um template string, criar um objeto ``Template``, criar um ``Context``,
e chamar o método ``render()``.

Múltiplos contextos, mesmo template
--------------------------------

Uma vez que você tem um objeto ``Template``, você pode processar múltiplos
contextos por ele. Por exemplo::

    >>> from django.template import Template, Context
    >>> t = Template('Olá, {{ name }}')
    >>> print t.render(Context({'name': 'John'}))
    Olá, John
    >>> print t.render(Context({'name': 'Julie'}))
    Olá, Julie
    >>> print t.render(Context({'name': 'Pat'}))
    Olá, Pat

Sempre que você está usando o mesmo código de template para renderizar
multiplos contextos, como isso, é mais eficiente criar o objeto
``Template`` *uma vez*, e depois chamar o ``render()`` por várias vezes::

    # Ruim
    for name in ('John', 'Julie', 'Pat'):
        t = Template('Olá, {{ name }}')
        print t.render(Context({'name': name}))

    # Bom
    t = Template('Olá, {{ name }}')
    for name in ('John', 'Julie', 'Pat'):
        print t.render(Context({'name': name}))

A análise de templates do Django é bastante rápida. Nos bastidores, a maior
parte da análise acontece através da chamada a uma única expressão regular.
Isso é um contraste gritante com as engines de template baseadas em XML, o qual
provoca uma sobrecarga ao parser XML e tendem a ser na ordem de magnitude mais
lentos que a engine de renderização de template do Django.

Pesquisa váriavel de contexto
-----------------------------

Nos exemplos até agora, passamos valores simples nos contextos -- na maior parte
strings, álem de um exemplo com ```datetime.date``. No entanto, o sistema de
template manipula de forma elegante estruturas de dados mais complexas, como
listas, dicionários e objetos personalizados.

A chave para percorer estruturas complexas de dados nos templates Django é
o caracter ponto (``.``). Use o ponto para acessar as chaves do dicionário,
atributos, métodos ou índices em um objeto.

Isso é melhor ilustrado com alguns exemplos. Por exemplo, suponha que
você está passando um dicionário Python a um template. Para acessar o
valor desse dicionário por chave de dicionário, use o ponto::

    >>> from django.template import Template, Context
    >>> person = {'name': 'Sally', 'age': '43'}
    >>> t = Template('{{ person.name }} is {{ person.age }} years old.')
    >>> c = Context({'person': person})
    >>> t.render(c)
    u'Sally is 43 years old.'

Da mesma forma, pontos também permitem o acesso a atributos de objetos. Por
exemplo, um objeto Python ``datetime.date`` possui atributos ``year``, ``month``
e ``day``, e você pode usar o ponto para acessar esses atributos em um template
Django::

    >>> from django.template import Template, Context
    >>> import datetime
    >>> d = datetime.date(1993, 5, 2)
    >>> d.year
    1993
    >>> d.month
    5
    >>> d.day
    2
    >>> t = Template('The month is {{ date.month }} and the year is {{ date.year }}.')
    >>> c = Context({'date': d})
    >>> t.render(c)
    u'The month is 5 and the year is 1993.'

Esse exemplo usa uma classe customizada, demonstrando que pontos váriaveis
também permitem o acesso a objetos arbitrários::

    >>> from django.template import Template, Context
    >>> class Person(object):
    ...     def __init__(self, first_name, last_name):
    ...         self.first_name, self.last_name = first_name, last_name
    >>> t = Template('Hello, {{ person.first_name }} {{ person.last_name }}.')
    >>> c = Context({'person': Person('John', 'Smith')})
    >>> t.render(c)
    u'Hello, John Smith.'

Pontos também podem remeter a *métodos* em objetos. Por exemplo, cada string
Python tem os métodos ``upper()`` e ``isdigit()``, e você pode chama-los
nos templates Django usando a mesma sintaxe do ponto::

    >>> from django.template import Template, Context
    >>> t = Template('{{ var }} -- {{ var.upper }} -- {{ var.isdigit }}')
    >>> t.render(Context({'var': 'hello'}))
    u'hello -- HELLO -- False'
    >>> t.render(Context({'var': '123'}))
    u'123 -- 123 -- True'

Perceba que você *não* incluiu parenteses na chamada do método. Além disso,
não é possível passar argumentos para os métodos, você só pode chamar
métodos que não tem argumentos requeridos (Nós explicáremos essa filosofia
adiante nesse cápitulo).

Finalizando, pontos são usados também para acessar índices de listas, por exemplo::

    >>> from django.template import Template, Context
    >>> t = Template('Item 2 is {{ items.2 }}.')
    >>> c = Context({'items': ['apples', 'bananas', 'carrots']})
    >>> t.render(c)
    u'Item 2 is carrots.'

Índices negativos em listas não são permitidos. Por exemplo, a váriavel
de template ``{{ items.-1 }}`` causará um ``TemplateSyntaxError``.

.. admonition:: Listas Python

   Um lembrete: listas Python possuem índices baseados em 0. O primeiro item é
   o índice 0, o segundo é o índice 1 e assim por diante.

Pesquisa por ponto pode ser resumida assim: quando o sistema de template
encontra um ponto em nome de váriavel, ele tenta as pesquisas a seguir, nesta
ordem:

* Pesquisa de dicionário (ex. ``foo["bar"]``)
* Pesquisa de atributo (ex. ``foo.bar``)
* Chamada de método  (ex. ``foo.bar()``)
* Pesquisa em índice de lista (ex. ``foo[2]``)

O sistema usa o primeiro tipo de pesquisa que funcionar. É um circuito lógico
curto.

Pesquisa por ponto podem ser aninhados em vários níveis de profundidade. Por
exemplo, o exemplo a seguir usa ``{{ person.name.upper }}``, que se traduz
em uma pesquisa de dicionário (``person['name']``) e depois em uma chamada
de método (``upper()``)::

    >>> from django.template import Template, Context
    >>> person = {'name': 'Sally', 'age': '43'}
    >>> t = Template('{{ person.name.upper }} is {{ person.age }} years old.')
    >>> c = Context({'person': person})
    >>> t.render(c)
    u'SALLY is 43 years old.'

Comportamento para chamada de método
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Chamada de métodos são levemente mais complexa do que outros tipos de pesquisa.
Aqui estão algumas coisas que devemos ter em mente:

* Se, durante a pesquisa de método, o método escapar uma exceção, a exceção
  será propagada, a não ser que a exceção tenha um atributo ``silent_variable_failure``
  cujo o valor seja ``True``. Se a exceção naõ tem um atributo ``silent_variable_failure``,
  a váriavel vai renderizar uma string vazia, por exemplo::

        >>> t = Template("My name is {{ person.first_name }}.")
        >>> class PersonClass3:
        ...     def first_name(self):
        ...         raise AssertionError, "foo"
        >>> p = PersonClass3()
        >>> t.render(Context({"person": p}))
        Traceback (most recent call last):
        ...
        AssertionError: foo

        >>> class SilentAssertionError(AssertionError):
        ...     silent_variable_failure = True
        >>> class PersonClass4:
        ...     def first_name(self):
        ...         raise SilentAssertionError
        >>> p = PersonClass4()
        >>> t.render(Context({"person": p}))
        u'My name is .'

* Uma chamada de métodos funcionará se o método não tenha argumentos
  requeridos. Caso contrário, o sistema irá para o próximo tipo de pesquisa
  (pesquisa em índice de lista).

* Obviamente, alguns métodos tem efeitos colaterais, e seria insensato e
  uma possível falha de segurança, permitir que o sistema de template pudesse
  acessá-los.

  Digamos, por exemplo, você tem um objeto ``BankAccount`` que tem um método
  ``delete()``. Se o template inclui algo como ``{{ account.delete }}``,
  onde ``account`` é um objeto ``BankAccount``, o objeto seria excluído
  quando o template for renderizado!

  Para previnir isso, defina o atributo ``alters_data`` no método::

      def delete(self):
          # Excluí um conta
      delete.alters_data = True

  O sistema de template não irá executar metodos marcados dessa maneira.
  Continuando exemplo acima, se o template incluir ``{{ account.delete }}``
  e o método ``delete()`` tem o ``alters_data=True``, então o método
  ``delete()` não será executado quando o template é renderizado. Ao invés
  disso, ele irá falhar silenciosamente.

Como váriaveis inválidas são tratadas
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Por padrão, se uma váriavel não existe, o sistema de templates mostra ela como
uma string vazia, falhando silenciosamente. Por exemplo::

    >>> from django.template import Template, Context
    >>> t = Template('Your name is {{ name }}.')
    >>> t.render(Context())
    u'Your name is .'
    >>> t.render(Context({'var': 'hello'}))
    u'Your name is .'
    >>> t.render(Context({'NAME': 'hello'}))
    u'Your name is .'
    >>> t.render(Context({'Name': 'hello'}))
    u'Your name is .'

O sistema falha silenciosamente, ao invés de levantar uma exceção porque
ele entende ser resiliente a um erro humano. Nesse caso, todas as pesquisas
falharam porque os nomes das váriaveis foram escritas com o tamanho ou nome
na forma errada. No mundo real, é inaceitaǘel para um web site tornar-se
inacessível devido a um pequeno erro de sintaxe em template.

Jogando com objetos de contexto
-------------------------------

Na maioria das vezes, você vai instanciar objetos ``Context`` passando um
dicionário totalmente preenchido para ``Context()``. Mas você pode adicionar
e excluir items de um objeto ``Context`` uma vez que estanciado, também, usando
a sintaxe padrão de dicionários Python::

    >>> from django.template import Context
    >>> c = Context({"foo": "bar"})
    >>> c['foo']
    'bar'
    >>> del c['foo']
    >>> c['foo']
    Traceback (most recent call last):
      ...
    KeyError: 'foo'
    >>> c['newvariable'] = 'hello'
    >>> c['newvariable']
    'hello'

Básico de Template Tags e Filtros
=================================

Como já mencionado, the template system ships with built-in tags and
filters. As seções seguintes fornecem um resumo das tags e filtros mais
comuns.

Tags
----

if/else
~~~~~~~

A tag ``{% if %}`` avalia uma váriavel e se a váriavel é "True" (ou seja,
ela existe, não está vazia e não é um valor booleano falso), o sistema
irá exibir tudo entre ``{% if %}`` e ``{% endif %}``, por example::

    {% if today_is_weekend %}
        <p>Welcome to the weekend!</p>
    {% endif %}

E a tag ``{% else %}`` é opcional::

    {% if today_is_weekend %}
        <p>Welcome to the weekend!</p>
    {% else %}
        <p>Get back to work.</p>
    {% endif %}

.. admonition:: Python "Truthiness"

   Em Python e no sistema de template do Django, estes objetos apresentam
   valor ``False`` em um contexto booleano::

   * Uma lista vazia (``[]``)
   * Uma tupla vazia (``()``)
   * Um dicionário vazio (``{}``)
   * Uma string vazia (``''``)
   * Zero (``0``)
   * O objeto especial ``None``
   * O objeto ``False`` (obviamente)
   * Objetos customizados que definem seu próprio contexto de comportamento
   booleano (isso é um uso avançado do Python)

   Todo o resto é avaliado com ``True``.

The ``{% if %}`` tag accepts ``and``, ``or``, or ``not`` for testing multiple
variables, or to negate a given variable. For example::

    {% if athlete_list and coach_list %}
        Both athletes and coaches are available.
    {% endif %}

    {% if not athlete_list %}
        There are no athletes.
    {% endif %}

    {% if athlete_list or coach_list %}
        There are some athletes or some coaches.
    {% endif %}

    {% if not athlete_list or coach_list %}
        There are no athletes or there are some coaches.
    {% endif %}

    {% if athlete_list and not coach_list %}
        There are some athletes and absolutely no coaches.
    {% endif %}

``{% if %}`` tags don't allow ``and`` and ``or`` clauses within the same tag,
because the order of logic would be ambiguous. For example, this is invalid::

    {% if athlete_list and coach_list or cheerleader_list %}

The use of parentheses for controlling order of operations is not supported. If
you find yourself needing parentheses, consider performing logic outside the
template and passing the result of that as a dedicated template variable. Or,
just use nested ``{% if %}`` tags, like this::

    {% if athlete_list %}
        {% if coach_list or cheerleader_list %}
            We have athletes, and either coaches or cheerleaders!
        {% endif %}
    {% endif %}

Multiple uses of the same logical operator are fine, but you can't
combine different operators. For example, this is valid::

    {% if athlete_list or coach_list or parent_list or teacher_list %}

There is no ``{% elif %}`` tag. Use nested ``{% if %}`` tags to accomplish
the same thing::

    {% if athlete_list %}
        <p>Here are the athletes: {{ athlete_list }}.</p>
    {% else %}
        <p>No athletes are available.</p>
        {% if coach_list %}
            <p>Here are the coaches: {{ coach_list }}.</p>
        {% endif %}
    {% endif %}

Make sure to close each ``{% if %}`` with an ``{% endif %}``. Otherwise, Django
will throw a ``TemplateSyntaxError``.

for
~~~

The ``{% for %}`` tag allows you to loop over each item in a sequence. As in
Python's ``for`` statement, the syntax is ``for X in Y``, where ``Y`` is the
sequence to loop over and ``X`` is the name of the variable to use for a
particular cycle of the loop. Each time through the loop, the template system
will render everything between ``{% for %}`` and ``{% endfor %}``.

For example, you could use the following to display a list of athletes given a
variable ``athlete_list``::

    <ul>
    {% for athlete in athlete_list %}
        <li>{{ athlete.name }}</li>
    {% endfor %}
    </ul>

Add ``reversed`` to the tag to loop over the list in reverse::

    {% for athlete in athlete_list reversed %}
    ...
    {% endfor %}

It's possible to nest ``{% for %}`` tags::

    {% for athlete in athlete_list %}
        <h1>{{ athlete.name }}</h1>
        <ul>
        {% for sport in athlete.sports_played %}
            <li>{{ sport }}</li>
        {% endfor %}
        </ul>
    {% endfor %}

A common pattern is to check the size of the list before looping over it, and
outputting some special text if the list is empty::

    {% if athlete_list %}
        {% for athlete in athlete_list %}
            <p>{{ athlete.name }}</p>
        {% endfor %}
    {% else %}
        <p>There are no athletes. Only computer programmers.</p>
    {% endif %}

Because this pattern is so common, the ``for`` tag supports an optional
``{% empty %}`` clause that lets you define what to output if the list is
empty. This example is equivalent to the previous one::

    {% for athlete in athlete_list %}
        <p>{{ athlete.name }}</p>
    {% empty %}
        <p>There are no athletes. Only computer programmers.</p>
    {% endfor %}

There is no support for "breaking out" of a loop before the loop is finished.
If you want to accomplish this, change the variable you're looping over so that
it includes only the values you want to loop over. Similarly, there is no
support for a "continue" statement that would instruct the loop processor to
return immediately to the front of the loop. (See the section "Philosophies and
Limitations" later in this chapter for the reasoning behind this design
decision.)

Within each ``{% for %}`` loop, you get access to a template variable called
``forloop``. This variable has a few attributes that give you information about
the progress of the loop:

* ``forloop.counter`` is always set to an integer representing the number
  of times the loop has been entered. This is one-indexed, so the first
  time through the loop, ``forloop.counter`` will be set to ``1``.
  Here's an example::

      {% for item in todo_list %}
          <p>{{ forloop.counter }}: {{ item }}</p>
      {% endfor %}

* ``forloop.counter0`` is like ``forloop.counter``, except it's
  zero-indexed. Its value will be set to ``0`` the first time through the
  loop.

* ``forloop.revcounter`` is always set to an integer representing the
  number of remaining items in the loop. The first time through the loop,
  ``forloop.revcounter`` will be set to the total number of items in the
  sequence you're traversing. The last time through the loop,
  ``forloop.revcounter`` will be set to ``1``.

* ``forloop.revcounter0`` is like ``forloop.revcounter``, except it's
  zero-indexed. The first time through the loop, ``forloop.revcounter0``
  will be set to the number of elements in the sequence minus 1. The last
  time through the loop, it will be set to ``0``.

* ``forloop.first`` is a Boolean value set to ``True`` if this is the first
  time through the loop. This is convenient for special-casing::

      {% for object in objects %}
          {% if forloop.first %}<li class="first">{% else %}<li>{% endif %}
          {{ object }}
          </li>
      {% endfor %}

* ``forloop.last`` is a Boolean value set to ``True`` if this is the last
  time through the loop. A common use for this is to put pipe
  characters between a list of links::

      {% for link in links %}{{ link }}{% if not forloop.last %} | {% endif %}{% endfor %}

  The above template code might output something like this::

      Link1 | Link2 | Link3 | Link4

  Another common use for this is to put a comma between words in a list::

      Favorite places:
      {% for p in places %}{{ p }}{% if not forloop.last %}, {% endif %}{% endfor %}

*  ``forloop.parentloop`` is a reference to the ``forloop`` object for the
   *parent* loop, in case of nested loops. Here's an example::

      {% for country in countries %}
          <table>
          {% for city in country.city_list %}
              <tr>
              <td>Country #{{ forloop.parentloop.counter }}</td>
              <td>City #{{ forloop.counter }}</td>
              <td>{{ city }}</td>
              </tr>
          {% endfor %}
          </table>
      {% endfor %}

The magic ``forloop`` variable is only available within loops. After the
template parser has reached ``{% endfor %}``, ``forloop`` disappears.

.. admonition:: Context and the forloop Variable

   Inside the ``{% for %}`` block, the existing variables are moved
   out of the way to avoid overwriting the magic ``forloop``
   variable. Django exposes this moved context in
   ``forloop.parentloop``. You generally don't need to worry about
   this, but if you supply a template variable named ``forloop``
   (though we advise against it), it will be named
   ``forloop.parentloop`` while inside the ``{% for %}`` block.

ifequal/ifnotequal
~~~~~~~~~~~~~~~~~~

The Django template system deliberately is not a full-fledged programming
language and thus does not allow you to execute arbitrary Python statements.
(More on this idea in the section "Philosophies and Limitations.") However,
it's quite a common template requirement to compare two values and display
something if they're equal -- and Django provides an ``{% ifequal %}`` tag for
that purpose.

The ``{% ifequal %}`` tag compares two values and displays everything between
``{% ifequal %}`` and ``{% endifequal %}`` if the values are equal.

This example compares the template variables ``user`` and ``currentuser``::

    {% ifequal user currentuser %}
        <h1>Welcome!</h1>
    {% endifequal %}

The arguments can be hard-coded strings, with either single or double quotes,
so the following is valid::

    {% ifequal section 'sitenews' %}
        <h1>Site News</h1>
    {% endifequal %}

    {% ifequal section "community" %}
        <h1>Community</h1>
    {% endifequal %}

Just like ``{% if %}``, the ``{% ifequal %}`` tag supports an optional
``{% else %}``::

    {% ifequal section 'sitenews' %}
        <h1>Site News</h1>
    {% else %}
        <h1>No News Here</h1>
    {% endifequal %}

Only template variables, strings, integers, and decimal numbers are allowed as
arguments to ``{% ifequal %}``. These are valid examples::

    {% ifequal variable 1 %}
    {% ifequal variable 1.23 %}
    {% ifequal variable 'foo' %}
    {% ifequal variable "foo" %}

Any other types of variables, such as Python dictionaries, lists, or Booleans,
can't be hard-coded in ``{% ifequal %}``. These are invalid examples::

    {% ifequal variable True %}
    {% ifequal variable [1, 2, 3] %}
    {% ifequal variable {'key': 'value'} %}

If you need to test whether something is true or false, use the ``{% if %}``
tags instead of ``{% ifequal %}``.

Comments
~~~~~~~~

Just as in HTML or Python, the Django template language allows for comments. To
designate a comment, use ``{# #}``::

    {# This is a comment #}

The comment will not be output when the template is rendered.

Comments using this syntax cannot span multiple lines. This limitation improves
template parsing performance. In the following template, the rendered output
will look exactly the same as the template (i.e., the comment tag will
not be parsed as a comment)::

    This is a {# this is not
    a comment #}
    test.

If you want to use multi-line comments, use the ``{% comment %}`` template tag,
like this::

    {% comment %}
    This is a
    multi-line comment.
    {% endcomment %}

Filters
-------

As explained earlier in this chapter, template filters are simple ways of
altering the value of variables before they're displayed. Filters use a pipe
character, like this::

    {{ name|lower }}

This displays the value of the ``{{ name }}`` variable after being filtered
through the ``lower`` filter, which converts text to lowercase.

Filters can be *chained* -- that is, they can be used in tandem such that the
output of one filter is applied to the next. Here's an example that takes the
first element in a list and converts it to uppercase::

    {{ my_list|first|upper }}

Some filters take arguments. A filter argument comes after a colon and is
always in double quotes. For example::

    {{ bio|truncatewords:"30" }}

This displays the first 30 words of the ``bio`` variable.

The following are a few of the most important filters. Appendix E covers the rest.

* ``addslashes``: Adds a backslash before any backslash, single quote, or
  double quote. This is useful if the produced text is included in
  a JavaScript string.

* ``date``: Formats a ``date`` or ``datetime`` object according to a
  format string given in the parameter, for example::

      {{ pub_date|date:"F j, Y" }}

  Format strings are defined in Appendix E.

* ``length``: Returns the length of the value. For a list, this returns the
  number of elements. For a string, this returns the number of characters.
  (Python experts, take note that this works on any Python object that
  knows how to determine its length -- i.e., any object that has a
  ``__len__()`` method.)

Philosophies and Limitations
============================

Now that you've gotten a feel for the Django template language, we should point
out some of its intentional limitations, along with some philosophies behind why
it works the way it works.

More than any other component of Web applications, template syntax is highly
subjective, and programmers' opinions vary wildly. The fact that Python alone
has dozens, if not hundreds, of open source template-language implementations
supports this point. Each was likely created because its developer deemed all
existing template languages inadequate. (In fact, it is said to be a rite of
passage for a Python developer to write his or her own template language! If
you haven't done this yet, consider it. It's a fun exercise.)

With that in mind, you might be interested to know that Django doesn't require
that you use its template language. Because Django is intended to be a
full-stack Web framework that provides all the pieces necessary for Web
developers to be productive, many times it's *more convenient* to use Django's
template system than other Python template libraries, but it's not a strict
requirement in any sense. As you'll see in the upcoming section "Using Templates
in Views", it's very easy to use another template language with Django.

Still, it's clear we have a strong preference for the way Django's template
language works. The template system has roots in how Web development is done at
World Online and the combined experience of Django's creators. Here are a few of
those philosophies:

* *Business logic should be separated from presentation logic*. Django's
  developers see a  template system as a tool that controls presentation and
  presentation-related logic -- and that's it. The template system shouldn't
  support functionality that goes beyond this basic goal.

  For that reason, it's impossible to call Python code directly within
  Django templates. All "programming" is fundamentally limited to the scope
  of what template tags can do. It *is* possible to write custom template
  tags that do arbitrary things, but the out-of-the-box Django template
  tags intentionally do not allow for arbitrary Python code execution.

* *Syntax should be decoupled from HTML/XML*. Although Django's template
  system is used primarily to produce HTML, it's intended to be just as
  usable for non-HTML formats, such as plain text. Some other template
  languages are XML based, placing all template logic within XML tags or
  attributes, but Django deliberately avoids this limitation. Requiring
  valid XML to write templates introduces a world of human mistakes and
  hard-to-understand error messages, and using an XML engine to parse
  templates incurs an unacceptable level of overhead in template processing.

* *Designers are assumed to be comfortable with HTML code*. The template
  system isn't designed so that templates necessarily are displayed nicely
  in WYSIWYG editors such as Dreamweaver. That is too severe a limitation
  and wouldn't allow the syntax to be as friendly as it is. Django expects
  template authors to be comfortable editing HTML directly.

* *Designers are assumed not to be Python programmers*. The template system
  authors recognize that Web page templates are most often written by
  *designers*, not *programmers*, and therefore should not assume Python
  knowledge.

  However, the system also intends to accommodate small teams in which the
  templates *are* created by Python programmers. It offers a way to extend
  the system's syntax by writing raw Python code. (More on this in Chapter
  9.)

* *The goal is not to invent a programming language*. The goal is to offer
  just enough programming-esque functionality, such as branching and
  looping, that is essential for making presentation-related decisions.

Using Templates in Views
========================

You've learned the basics of using the template system; now let's use this
knowledge to create a view. Recall the ``current_datetime`` view in
``mysite.views``, which we started in the previous chapter. Here's what it looks
like::

    from django.http import HttpResponse
    import datetime

    def current_datetime(request):
        now = datetime.datetime.now()
        html = "<html><body>It is now %s.</body></html>" % now
        return HttpResponse(html)

Let's change this view to use Django's template system. At first, you might
think to do something like this::

    from django.template import Template, Context
    from django.http import HttpResponse
    import datetime

    def current_datetime(request):
        now = datetime.datetime.now()
        t = Template("<html><body>It is now {{ current_date }}.</body></html>")
        html = t.render(Context({'current_date': now}))
        return HttpResponse(html)

Sure, that uses the template system, but it doesn't solve the problems we
pointed out in the introduction of this chapter. Namely, the template is still
embedded in the Python code, so true separation of data and presentation isn't
achieved. Let's fix that by putting the template in a *separate file*, which
this view will load.

You might first consider saving your template somewhere on your
filesystem and using Python's built-in file-opening functionality to read
the contents of the template. Here's what that might look like, assuming the
template was saved as the file ``/home/djangouser/templates/mytemplate.html``::

    from django.template import Template, Context
    from django.http import HttpResponse
    import datetime

    def current_datetime(request):
        now = datetime.datetime.now()
        # Simple way of using templates from the filesystem.
        # This is BAD because it doesn't account for missing files!
        fp = open('/home/djangouser/templates/mytemplate.html')
        t = Template(fp.read())
        fp.close()
        html = t.render(Context({'current_date': now}))
        return HttpResponse(html)

This approach, however, is inelegant for these reasons:

* It doesn't handle the case of a missing file. If the file
  ``mytemplate.html`` doesn't exist or isn't readable, the ``open()`` call
  will raise an ``IOError`` exception.

* It hard-codes your template location. If you were to use this
  technique for every view function, you'd be duplicating the template
  locations. Not to mention it involves a lot of typing!

* It includes a lot of boring boilerplate code. You've got better things to
  do than to write calls to ``open()``, ``fp.read()``, and ``fp.close()``
  each time you load a template.

To solve these issues, we'll use *template loading* and *template directories*.

Template Loading
================

Django provides a convenient and powerful API for loading templates from the
filesystem, with the goal of removing redundancy both in your template-loading
calls and in your templates themselves.

In order to use this template-loading API, first you'll need to tell the
framework where you store your templates. The place to do this is in your
settings file -- the ``settings.py`` file that we mentioned last chapter, when
we introduced the ``ROOT_URLCONF`` setting.

If you're following along, open your ``settings.py`` and find the
``TEMPLATE_DIRS`` setting. By default, it's an empty tuple, likely containing
some auto-generated comments::

    TEMPLATE_DIRS = (
        # Put strings here, like "/home/html/django_templates" or "C:/www/django/templates".
        # Always use forward slashes, even on Windows.
        # Don't forget to use absolute paths, not relative paths.
    )

This setting tells Django's template-loading mechanism where to look for
templates. Pick a directory where you'd like to store your templates and add it
to ``TEMPLATE_DIRS``, like so::

    TEMPLATE_DIRS = (
        '/home/django/mysite/templates',
    )

There are a few things to note:

* You can specify any directory you want, as long as the directory and
  templates within that directory are readable by the user account under
  which your Web server runs. If you can't think of an appropriate
  place to put your templates, we recommend creating a ``templates``
  directory within your project (i.e., within the ``mysite`` directory you
  created in Chapter 2).

* If your ``TEMPLATE_DIRS`` contains only one directory, don't forget the
  comma at the end of the directory string!

  Bad::

      # Missing comma!
      TEMPLATE_DIRS = (
          '/home/django/mysite/templates'
      )

  Good::

      # Comma correctly in place.
      TEMPLATE_DIRS = (
          '/home/django/mysite/templates',
      )

  The reason for this is that Python requires commas within single-element
  tuples to disambiguate the tuple from a parenthetical expression. This is
  a common newbie gotcha.

* If you're on Windows, include your drive letter and use Unix-style
  forward slashes rather than backslashes, as follows::

      TEMPLATE_DIRS = (
          'C:/www/django/templates',
      )

* It's simplest to use absolute paths (i.e., directory paths that start at
  the root of the filesystem). If you want to be a bit more flexible and
  decoupled, though, you can take advantage of the fact that Django
  settings files are just Python code by constructing the contents of
  ``TEMPLATE_DIRS`` dynamically. For example::

      import os.path

      TEMPLATE_DIRS = (
          os.path.join(os.path.dirname(__file__), 'templates').replace('\\','/'),
      )

  This example uses the "magic" Python variable ``__file__``, which is
  automatically set to the file name of the Python module in which the code
  lives. It gets the name of the directory that contains ``settings.py``
  (``os.path.dirname``), then joins that with ``templates`` in a
  cross-platform way (``os.path.join``), then ensures that everything uses
  forward slashes instead of backslashes (in case of Windows).

  While we're on the topic of dynamic Python code in settings files, we
  should point out that it's very important to avoid Python errors in your
  settings file. If you introduce a syntax error, or a runtime error, your
  Django-powered site will likely crash.

With ``TEMPLATE_DIRS`` set, the next step is to change the view code to
use Django's template-loading functionality rather than hard-coding the
template paths. Returning to our ``current_datetime`` view, let's change it
like so::

    from django.template.loader import get_template
    from django.template import Context
    from django.http import HttpResponse
    import datetime

    def current_datetime(request):
        now = datetime.datetime.now()
        t = get_template('current_datetime.html')
        html = t.render(Context({'current_date': now}))
        return HttpResponse(html)

In this example, we're using the function
``django.template.loader.get_template()`` rather than loading the template from
the filesystem manually. The ``get_template()`` function takes a template name
as its argument, figures out where the template lives on the filesystem, opens
that file, and returns a compiled ``Template`` object.

Our template in this example is ``current_datetime.html``, but there's nothing
special about that ``.html`` extension. You can give your templates whatever
extension makes sense for your application, or you can leave off extensions
entirely.

To determine the location of the template on your filesystem,
``get_template()`` combines your template directories from ``TEMPLATE_DIRS``
with the template name that you pass to ``get_template()``. For example, if
your ``TEMPLATE_DIRS`` is set to ``'/home/django/mysite/templates'``, the above
``get_template()`` call would look for the template
``/home/django/mysite/templates/current_datetime.html``.

If ``get_template()`` cannot find the template with the given name, it raises
a ``TemplateDoesNotExist`` exception. To see what that looks like, fire up the
Django development server again by running ``python manage.py runserver``
within your Django project's directory. Then, point your browser at the page
that activates the ``current_datetime`` view (e.g.,
``http://127.0.0.1:8000/time/``). Assuming your ``DEBUG`` setting is set to
``True`` and you haven't yet created a ``current_datetime.html`` template, you
should see a Django error page highlighting the ``TemplateDoesNotExist`` error.

.. figure:: graphics/chapter04/missing_template.png
   :alt: Screenshot of a "TemplateDoesNotExist" error.

   Figure 4-1: The error page shown when a template cannot be found.

This error page is similar to the one we explained in Chapter 3, with one
additional piece of debugging information: a "Template-loader postmortem"
section. This section tells you which templates Django tried to load, along with
the reason each attempt failed (e.g., "File does not exist"). This information
is invaluable when you're trying to debug template-loading errors.

Moving along, create the ``current_datetime.html`` file within your template
directory using the following template code::

    <html><body>It is now {{ current_date }}.</body></html>

Refresh the page in your Web browser, and you should see the fully rendered
page.

render()
--------

We've shown you how to load a template, fill a ``Context`` and return an
``HttpResponse`` object with the result of the rendered template. We've
optimized it to use ``get_template()`` instead of hard-coding templates and
template paths. But it still requires a fair amount of typing to do those
things. Because this is such a common idiom, Django provides a shortcut that
lets you load a template, render it and return an ``HttpResponse`` -- all in
one line of code.

This shortcut is a function called ``render()``, which lives in the
module ``django.shortcuts``. Most of the time, you'll be using
``render()`` rather than loading templates and creating ``Context``
and ``HttpResponse`` objects manually -- unless your employer judges your work
by total lines of code written, that is.

Here's the ongoing ``current_datetime`` example rewritten to use
``render()``::

    from django.shortcuts import render
    import datetime

    def current_datetime(request):
        now = datetime.datetime.now()
        return render(request, 'current_datetime.html', {'current_date': now})

What a difference! Let's step through the code changes:

* We no longer have to import ``get_template``, ``Template``, ``Context``,
  or ``HttpResponse``. Instead, we import
  ``django.shortcuts.render``. The ``import datetime`` remains.

* Within the ``current_datetime`` function, we still calculate ``now``, but
  the template loading, context creation, template rendering, and
  ``HttpResponse`` creation are all taken care of by the
  ``render()`` call. Because ``render()`` returns
  an ``HttpResponse`` object, we can simply ``return`` that value in the
  view.

The first argument to ``render()`` is the request, the second is the name of
the template to use. The third argument, if given, should be a dictionary to
use in creating a ``Context`` for that template. If you don't provide a third
argument, ``render()`` will use an empty dictionary.

Subdirectories in get_template()
--------------------------------

It can get unwieldy to store all of your templates in a single directory. You
might like to store templates in subdirectories of your template directory, and
that's fine. In fact, we recommend doing so; some more advanced Django
features (such as the generic views system, which we cover in
Chapter 11) expect this template layout as a default convention.

Storing templates in subdirectories of your template directory is easy.
In your calls to ``get_template()``, just include
the subdirectory name and a slash before the template name, like so::

    t = get_template('dateapp/current_datetime.html')

Because ``render()`` is a small wrapper around ``get_template()``,
you can do the same thing with the second argument to ``render()``,
like this::

    return render(request, 'dateapp/current_datetime.html', {'current_date': now})

There's no limit to the depth of your subdirectory tree. Feel free to use
as many subdirectories as you like.

.. note::

    Windows users, be sure to use forward slashes rather than backslashes.
    ``get_template()`` assumes a Unix-style file name designation.

The ``include`` Template Tag
----------------------------

Now that we've covered the template-loading mechanism, we can introduce a
built-in template tag that takes advantage of it: ``{% include %}``. This tag
allows you to include the contents of another template. The argument to the tag
should be the name of the template to include, and the template name can be
either a variable or a hard-coded (quoted) string, in either single or double
quotes. Anytime you have the same code in multiple templates,
consider using an ``{% include %}`` to remove the duplication.

These two examples include the contents of the template ``nav.html``. The
examples are equivalent and illustrate that either single or double quotes
are allowed::

    {% include 'nav.html' %}
    {% include "nav.html" %}

This example includes the contents of the template ``includes/nav.html``::

    {% include 'includes/nav.html' %}

This example includes the contents of the template whose name is contained in
the variable ``template_name``::

    {% include template_name %}

As in ``get_template()``, the file name of the template is determined by adding
the template directory from ``TEMPLATE_DIRS`` to the requested template name.

Included templates are evaluated with the context of the template
that's including them. For example, consider these two templates::

    # mypage.html

    <html>
    <body>
    {% include "includes/nav.html" %}
    <h1>{{ title }}</h1>
    </body>
    </html>

    # includes/nav.html

    <div id="nav">
        You are in: {{ current_section }}
    </div>

If you render ``mypage.html`` with a context containing ``current_section``,
then the variable will be available in the "included" template, as you would
expect.

If, in an ``{% include %}`` tag, a template with the given name isn't found,
Django will do one of two things:

* If ``DEBUG`` is set to ``True``, you'll see the
  ``TemplateDoesNotExist`` exception on a Django error page.

* If ``DEBUG`` is set to ``False``, the tag will fail
  silently, displaying nothing in the place of the tag.

Template Inheritance
====================

Our template examples so far have been tiny HTML snippets, but in the real
world, you'll be using Django's template system to create entire HTML pages.
This leads to a common Web development problem: across a Web site, how does
one reduce the duplication and redundancy of common page areas, such as
sitewide navigation?

A classic way of solving this problem is to use *server-side includes*,
directives you can embed within your HTML pages to "include" one Web page
inside another. Indeed, Django supports that approach, with the
``{% include %}`` template tag just described. But the preferred way of
solving this problem with Django is to use a more elegant strategy called
*template inheritance*.

In essence, template inheritance lets you build a base "skeleton" template that
contains all the common parts of your site and defines "blocks" that child
templates can override.

Let's see an example of this by creating a more complete template for our
``current_datetime`` view, by editing the ``current_datetime.html`` file::

    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">
    <html lang="en">
    <head>
        <title>The current time</title>
    </head>
    <body>
        <h1>My helpful timestamp site</h1>
        <p>It is now {{ current_date }}.</p>

        <hr>
        <p>Thanks for visiting my site.</p>
    </body>
    </html>

That looks just fine, but what happens when we want to create a template for
another view -- say, the ``hours_ahead`` view from Chapter 3? If we want again
to make a nice, valid, full HTML template, we'd create something like::

    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">
    <html lang="en">
    <head>
        <title>Future time</title>
    </head>
    <body>
        <h1>My helpful timestamp site</h1>
        <p>In {{ hour_offset }} hour(s), it will be {{ next_time }}.</p>

        <hr>
        <p>Thanks for visiting my site.</p>
    </body>
    </html>

Clearly, we've just duplicated a lot of HTML. Imagine if we had a more
typical site, including a navigation bar, a few style sheets, perhaps some
JavaScript -- we'd end up putting all sorts of redundant HTML into each
template.

The server-side include solution to this problem is to factor out the
common bits in both templates and save them in separate template snippets,
which are then included in each template. Perhaps you'd store the top
bit of the template in a file called ``header.html``::

    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">
    <html lang="en">
    <head>

And perhaps you'd store the bottom bit in a file called ``footer.html``::

        <hr>
        <p>Thanks for visiting my site.</p>
    </body>
    </html>

With an include-based strategy, headers and footers are easy. It's the
middle ground that's messy. In this example, both pages feature a title --
``<h1>My helpful timestamp site</h1>`` -- but that title can't fit into
``header.html`` because the ``<title>`` on both pages is different. If we
included the ``<h1>`` in the header, we'd have to include the ``<title>``,
which wouldn't allow us to customize it per page. See where this is going?

Django's template inheritance system solves these problems. You can think of it
as an "inside-out" version of server-side includes. Instead of defining the
snippets that are *common*, you define the snippets that are *different*.

The first step is to define a *base template* -- a skeleton of your page that
*child templates* will later fill in. Here's a base template for our ongoing
example::

    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">
    <html lang="en">
    <head>
        <title>{% block title %}{% endblock %}</title>
    </head>
    <body>
        <h1>My helpful timestamp site</h1>
        {% block content %}{% endblock %}
        {% block footer %}
        <hr>
        <p>Thanks for visiting my site.</p>
        {% endblock %}
    </body>
    </html>

This template, which we'll call ``base.html``, defines a simple HTML skeleton
document that we'll use for all the pages on the site. It's the job of child
templates to override, or add to, or leave alone the contents of the blocks.
(If you're following along, save this file to your template directory as
``base.html``.)

We're using a template tag here that you haven't seen before: the
``{% block %}`` tag. All the ``{% block %}`` tags do is tell the template
engine that a child template may override those portions of the template.

Now that we have this base template, we can modify our existing
``current_datetime.html`` template to use it::

    {% extends "base.html" %}

    {% block title %}The current time{% endblock %}

    {% block content %}
    <p>It is now {{ current_date }}.</p>
    {% endblock %}

While we're at it, let's create a template for the ``hours_ahead`` view from
Chapter 3. (If you're following along with code, we'll leave it up to you to
change ``hours_ahead`` to use the template system instead of hard-coded HTML.)
Here's what that could look like::

    {% extends "base.html" %}

    {% block title %}Future time{% endblock %}

    {% block content %}
    <p>In {{ hour_offset }} hour(s), it will be {{ next_time }}.</p>
    {% endblock %}

Isn't this beautiful? Each template contains only the code that's *unique* to
that template. No redundancy needed. If you need to make a site-wide design
change, just make the change to ``base.html``, and all of the other templates
will immediately reflect the change.

Here's how it works. When you load the template ``current_datetime.html``,
the template engine sees the ``{% extends %}`` tag, noting that
this template is a child template. The engine immediately loads the
parent template -- in this case, ``base.html``.

At that point, the template engine notices the three ``{% block %}`` tags
in ``base.html`` and replaces those blocks with the contents of the child
template. So, the title we've defined in ``{% block title %}`` will be
used, as will the ``{% block content %}``.

Note that since the child template doesn't define the ``footer`` block,
the template system uses the value from the parent template instead.
Content within a ``{% block %}`` tag in a parent template is always
used as a fallback.

Inheritance doesn't affect the template context. In other words, any template
in the inheritance tree will have access to every one of your template
variables from the context.

You can use as many levels of inheritance as needed. One common way of using
inheritance is the following three-level approach:

1. Create a ``base.html`` template that holds the main look and feel of
   your site. This is the stuff that rarely, if ever, changes.

2. Create a ``base_SECTION.html`` template for each "section" of your site
   (e.g., ``base_photos.html`` and ``base_forum.html``). These templates
   extend ``base.html`` and include section-specific styles/design.

3. Create individual templates for each type of page, such as a forum page
   or a photo gallery. These templates extend the appropriate section
   template.

This approach maximizes code reuse and makes it easy to add items to shared
areas, such as section-wide navigation.

Here are some guidelines for working with template inheritance:

* If you use ``{% extends %}`` in a template, it must be the first
  template tag in that template. Otherwise, template inheritance won't
  work.

* Generally, the more ``{% block %}`` tags in your base templates, the
  better. Remember, child templates don't have to define all parent blocks,
  so you can fill in reasonable defaults in a number of blocks, and then
  define only the ones you need in the child templates. It's better to have
  more hooks than fewer hooks.

* If you find yourself duplicating code in a number of templates, it
  probably means you should move that code to a ``{% block %}`` in a
  parent template.

* If you need to get the content of the block from the parent template,
  use ``{{ block.super }}``, which is a "magic" variable providing the
  rendered text of the parent template. This is useful if you want to add
  to the contents of a parent block instead of completely overriding it.

* You may not define multiple ``{% block %}`` tags with the same name in
  the same template. This limitation exists because a block tag works in
  "both" directions. That is, a block tag doesn't just provide a hole to
  fill, it also defines the content that fills the hole in the *parent*.
  If there were two similarly named ``{% block %}`` tags in a template,
  that template's parent wouldn't know which one of the blocks' content to
  use.

* The template name you pass to ``{% extends %}`` is loaded using the same
  method that ``get_template()`` uses. That is, the template name is
  appended to your ``TEMPLATE_DIRS`` setting.

* In most cases, the argument to ``{% extends %}`` will be a string, but it
  can also be a variable, if you don't know the name of the parent template
  until runtime. This lets you do some cool, dynamic stuff.

What's next?
============

You now have the basics of Django's template system under your belt. What's next?

Many modern Web sites are *database-driven*: the content of the Web site is
stored in a relational database. This allows a clean separation of data and logic
(in the same way views and templates allow the separation of logic and display.)

The :doc:`next chapter <chapter05>` covers the tools Django gives you to
interact with a database.
