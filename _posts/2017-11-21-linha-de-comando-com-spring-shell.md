---
layout: post
title: Linha de comando com Spring Shell
subtitle: Aprenda a criar uma aplicação de linha de comando.
bigimg: /img/posts/2017/spring-shell.png
tags: [java, spring, spring shell, linha de comando]
author: author-normandes.html
---

Nesse post você vai aprender algo bem legal, criar aplicações interativas por linha de comando! Bora? É simples e bem útil, eu prometo! :)


Em primeiro lugar, desculpa a sumida desde o último post, estou na reta final da entrega de um projeto, por isso o desaparecimento, você me entende, né? :)

Mas, o legal é que nesse mesmo projeto foi me passado um desafio, a conversa foi mais ou menos assim:

- _Normandes, precisamos criar uma aplicação que fará migrações entre dois ambientes, isso será útil para nosso cliente não precisar cadastrar/configurar a mesma coisa duas vezes._

Nessa aplicação eu precisaria criar um consumidor da API REST do serviço que estaria migrando,  teria que acessar o banco de dados e até consumir uma planilha Excel que o nosso cliente possuía.

Como eram vários comandos ora alguém pediria: _vamos migrar apenas a parte x, agora a parte y..._ 

Então pensei em criar uma aplicação que me permitisse fazer tudo isso de uma forma simples.

### Solução

Comecei então a pensar em como criar essa aplicação, seria web? Um script shell? Uma aplicação Java simples?

Fui levantando os prós e contras de cada uma delas e decidi que criaria uma aplicação de linha de comando iterativa, ou seja, acesso a aplicação e lá tenho os vários comandos que posso executar.

Eu gostei dessa ideia pois eu poderia ter apenas um jar, e ao executar teria um "help" para ir lembrando dos comandos disponíveis para as migrações.

### Spring Shell

Antes de chegar no Spring Shell encontrei o "Commons CLI" da Apache - [https://commons.apache.org/proper/commons-cli/](https://commons.apache.org/proper/commons-cli/){:target="_blank"}, fiz alguns testes e vi que teria um certo trabalho manual com vários "ifs" que não fariam parte do negócio que eu estava desenvolvendo.

Durante minhas pesquisas encontrei também o [Spring Shell](https://projects.spring.io/spring-shell/){:target="_blank"}, e logo nos primeiros testes já percebi, esse é o cara que estou procurando! :)

Por que o Spring Shell? Porque ele é muito simples, com poucas anotações já tenho tudo pronto, tem autocomplete, histórico, validações e muito mais!

E como estaria com uma aplicação Spring, poderia usar suas APIs de acesso a banco de dados e  REST de graça, nessa hora eu vi vantagem demais.

### Testando a aplicação

A ideia desse post é te mostrar que existe essa alternativa fácil de usar para criar aplicações de linha de comando que podem ser muito úteis na sua vida de programador.

Para te mostrar a ideia desse excelente projeto Spring eu criei uma aplicação que lista os filmes e personagens dos filmes de Star Wars, não que eu seja um grande conhecedor da franquia, mas por existir uma API REST gratuita na internet pra gente brincar. :)

Veja você mesmo aqui: [https://swapi.co/](https://swapi.co/){:target="_blank"}

Antes de te explicar o código, gostaria que você já tentasse usar a aplicação que está no Github.

Faça o clone ou o download em [https://github.com/normandesjr/spring-shell-example](https://github.com/normandesjr/spring-shell-example){:target="_blank"}

Acesse pela linha de comando, claro, e execute (Linux ou Mac, no Windows não precisa do ./):

~~~
./mvnw spring-boot:run
~~~

Deve demorar um pouco na primeira vez até baixar todas as dependências, e depois você verá algo como:

![Spring Shell](/img/posts/2017/spring-shell-print-inicial.png){: .center}

Digite o comando "help" e veja as opções. 

No grupo "Filmes" temos dois comandos, o "filmes" e o "pesquisar-filme-por-titulo". E no grupo "Personagens" o comando "personagens" e o "pesquisar-personagens-por-nome".

![Spring Shell Help](/img/posts/2017/spring-shell-help.png){: .center}

Se quiser ver todos os filmes da franquia, digite "filmes" e pressione enter, você verá algo como:

![Spring Shell Help](/img/posts/2017/spring-shell-filmes.png){: .center}

OBS: Como eu fiz esse banner da Hibicode? Eu sou um artista! kkkk brincadeira, foi nesse site aqui: [http://patorjk.com/software/taag/](http://patorjk.com/software/taag/){:target="_blank"}

### Código fonte

Se você acessar o código fonte e ler um pouco já já irá entender tudo que precisa para começar, é realmente muito simples.

Veja a classe "FilmeCommandLine", é onde eu crio todos os comandos relacionados aos filmes de Star Wars.

Tudo começa com as anotações "@ShellComponent" e "@ShellCommandGroup". Esta última é apenas para agruparmos os comandos em categorias.

{% highlight java %}
@ShellComponent
@ShellCommandGroup("Filmes")
public class FilmeCommandLine {
}
{% endhighlight %}

Depois temos os comandos que são os métodos públicos anotados com "@ShellMethod". A string que colocamos no valor da anotação é o que irá aparecer no help do comando, ou seja, se você digitar: "help filmes" por exemplo.

{% highlight java %}
@ShellMethod("Listar filmes")
public String filmes() {
  // ...
}
{% endhighlight %}

O conteúdo do método será a lógica de negócio que você queira executar, no meu caso foi buscar os filmes na swapi.co.

E você também pode receber parâmetros, como é o caso do método "pesquisarFilmePorTitulo":

{% highlight java %}
@ShellMethod("Pesquisar filme por título")
public String pesquisarFilmePorTitulo(
    @ShellOption(help = "título do filme que deseja pesquisar") String titulo) {
  // ...
}
{% endhighlight %}

Repare que a anotação "@ShellOption" é que define que "titulo" é um parâmetro, ou seja, na hora de pesquisar você irá fazer algo como:

~~~
shell:> pesquisar-filme-por-titulo --titulo "a new"
~~~

### Próximos passos

Existem muito mais opções para você programar com o Spring Shell, essa foi só uma pequena introdução para você saber que existe. :)

Para saber mais, recomendo fortemente a documentação: [https://docs.spring.io/spring-shell/docs/2.0.0.M2/reference/htmlsingle/](https://docs.spring.io/spring-shell/docs/2.0.0.M2/reference/htmlsingle/){:target="_blank"}

Gostou? Compartilhe com seus amigos, poderá ajudar alguém.

Abraço e até a próxima.
