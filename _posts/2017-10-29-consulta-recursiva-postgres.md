---
layout: post
title: Consulta recursiva com Postgres
subtitle: Aprenda a fazer uma consulta recursiva, um dia você irá precisar. 
bigimg: /img/posts/2017/postgres-logo-otim.png
tags: [postgres, postgresql, consulta recursiva, with recursive, hierarquia, consulta hierarquica]
author: author-normandes.html
---

Se em algum dia você tiver uma estrutura de dados que envolva hierarquia, ou seja, funcionário, coordenador, diretor, presidente... então você precisa aprender o que tem nesse post.

Como de costume, vou te ensinar o que precisei implementar no meu trabalho essa semana, que são as consultas recursivas do Postgres.

Se você não sabe como instalar o Postgres, aprenda aqui no blog usando o Docker [clicando aqui]({{ site.baseurl }}{% post_url 2017-08-20-docker-com-banco-de-dados %}){:target="_blank"}

### O problema

Quando você estiver programando um cenário que envolva hierarquia, as consultas recursivas serão uma mão na roda pra você.

A hierarquia ocorre quando tempos uma entidade que é superior a outra, temos o funcionário, acima dele o coordenador, que tem acima o diretor e assim por diante.

A consulta hierárquica irá te ajudar a retornar, por exemplo, me fale o nome de todos que estão abaixo do diretor.

No caso que precisei implementar aqui no trabalho foi algo bem parecido com isso, caso o diretor entrasse no sistema, ele precisava ver todos que estavam abaixo dele para fazer uma operação, mas ele não poderia fazer o mesmo com os seus superiores.

### Modelo de dados

Para a gente implementar algo que você possa testar ai na sua casa, vamos criar uma única tabela chamada de *funcionário*.

Nessa tabela iremos adicionar um código, um nome e a chave estrangeira para ela mesma que representará o funcionário superior.

{% highlight sql %}
CREATE TABLE funcionario (
  id INTEGER NOT NULL PRIMARY KEY,
  nome varchar(30),
  id_funcionario_superior INTEGER REFERENCES funcionario(id)
);
{% endhighlight %}

Depois da tabela criada, vamos criar a estrutura representada na imagem abaixo:

![hierarquia](/img/posts/2017/hibi-hierarquia-funcionarios.png){: .center}

Para isso, basta fazer o insert:

{% highlight sql %}
INSERT INTO funcionario VALUES 
 (1, 'João Silva', null), 
 (2, 'Vanessa Santos', 1), 
 (3, 'Felipe Castro', 1), 
 (4, 'Aline Pereira', 2), 
 (5, 'Caio Silva', 2),
 (6, 'Taís Naves', 3)
{% endhighlight %}

### Consulta recursiva

Agora que já temos o banco de dados preparado, podemos começar a brincar com a consulta recursiva.

A regra será: retorne todos os nomes dos funcionários que estão abaixo de um funcionário específico.

Nosso modelo está simples, para facilitar o entendimento, mas você poderia ter outras tabelas de relacionamento também, não precisa ser apenas uma tabela, para implementar a recursividade na consulta, você precisa apenas desse conceito de superior e subordinado.

A consulta recursiva tem um formato diferente do tradicional "SELECT * FROM tabela", veja como é a sintaxe básica abaixo, mas não se assuste, vou te explicar todas as partes:

{% highlight sql %}
WITH RECURSIVE tabela_auxilixar_consulta AS (
   <CONSULTA_NAO_RECURSIVA>
  UNION [ALL]
  <CONSULTA_RECURSIVA>
) <CONSULTA_FINAL>

{% endhighlight %}

Vamos começar pela consulta não recursiva primeiro, pois é nela que vamos definir a base para a recursividade acontecer.

Veja a simplicidade da consulta abaixo:

{% highlight sql %}
SELECT id
     , nome
 FROM funcionario
 WHERE nome = 'Felipe Castro'
{% endhighlight %}

Repare que não nada diferente, é uma consulta simples que nos retornará o id e o nome do funcionário filtrado, no caso por *Felipe Castro*.

Essa consulta forma a base para a recursividade, nós agora temos o ponto de partida, ou seja, vamos filtrar todos os funcionários que estão abaixo do *Felipe*.

Noss consulta está assim agora:

{% highlight sql %}
WITH RECURSIVE tabela_auxilixar_consulta AS (
   SELECT id
        , nome
    FROM funcionario
    WHERE nome = 'Felipe Castro'
  UNION [ALL]
  <CONSULTA_RECURSIVA>
) <CONSULTA_FINAL>

{% endhighlight %}

A consulta recursiva irá usar o resultado gerado pela primeira consulta que fizemos, então é necessário uma forma de fazer esse relacionamento entre as duas consultas, e é aí que o nome da *tabela_auxiliar_consulta* entre em cena. Na verdade esse nome é usado também para o resultado da consulta recursiva, mas espere um pouco, já já vou voltar nesse ponto.

Pensa comigo, para eu retornar todos os funcionários que são subordinados do *Felipe*, é só adicionar um filtro usando *id_funcionario_superior*, concorda? Veja só, execute a seguinte consulta:

{% highlight sql %}
SELECT id
     , nome
 FROM funcionario
 WHERE id_funcionario_superior = 3 -- 3 é o id do Felipe
{% endhighlight %}

Iremos obter como resposta a *Taís Naves*. Se usar o id da *Vanessa Santos*, irá obter a *Aline Pereira* e o *Caio Silva*, mas e se usarmos o id do *João Silva*? Teremos como resultado a *Vanessa Santos* e o *Felipe Castro*, mas nós gostaríamos que trouxesse todo mundo, pois ele é o maior na hierarquia, está me acompanhando?

Acontece que a consulta recursiva faz isso pra gente, existe um certo *loop* que continuará a fazer as consultas até não existir um retorno mais.

Nesse momento que nós vamos precisar usar o nome da *tabela_auxiliar*, pois ela armazenará pra gente o resultado de cada iteração, complicado? Parece, né? Mas não é, vamos continuar na implementação e vou te passar um macete para você entender 100%! ;)

Vou te mostrar a consulta final agora, e logo depois começo a explicar os detalhes:

{% highlight sql %}
WITH RECURSIVE subordinados AS (
   SELECT id
        , nome
   FROM funcionario
   WHERE nome = 'João Silva'

  UNION ALL

  SELECT f.id
       , f.nome
   FROM funcionario f
    INNER JOIN subordinados ON f.id_funcionario_superior = subordinados.id
) SELECT nome
  FROM subordinados;
{% endhighlight %}

A consulta recursiva faz um *join* com a tabela *subordinados* que é o nome da consulta recursiva, percebeu? 

Vamos executar passo a passo essa consulta:

1. Primeiro a consulta não recursiva é executada e nós teremos o id e o nome do *João Silva*
1. O resultado é gerado na tabela *subordinados* (é uma tabela temporária)
1. Então a consulta recursiva é executada, mas como a tabela *subordinados* tem algum dado, o filtro funcionará e um resultado será gerado
1. O resultado TODO dessa consulta recursiva é então entregue novamente na tabela *subordinados* e, para cada linha dela, a consulta recursiva é executada novamente, até que nenhum resultado seja retornado da consulta recursiva
1. No final, tudo estará na tabela *subordinados* e você poderá fazer a consulta final, no caso acima só selecionei o nome

Um macete que usei para aprender bem a fazer essas consultas foi fazer o trabalho do *WITH RECURSIVE* na mão, ou seja, na consulta recursiva se eu tirar *subordinados* como obterei o resultado esperado? Terei que filtrar o *id_funcionario_superior*, então comecei a fazer assim:

{% highlight sql %}
  SELECT f.id
       , f.nome
   FROM funcionario f
    WHERE f.id_funcionario_superior = 3
--    INNER JOIN subordinados ON f.id_funcionario_superior = subordinados.id
{% endhighlight %}

Repare que quando comento a parte da tabela temporária, dá pra ver claramente o que a recursividade irá fazer por nós.

### Conclusão

Eu achei importante compartilhar essa consulta pois é muito útil no nosso dia a dia como programador, essa estrutura de dados de superior e subordinado pode aparecer em vários casos, e quando você se defrontar com ela, lembre-se de voltar aqui para se inspirar. :)

Não se esqueça de se inscrever na nossa lista de e-mails para eu te mandar conteúdos assim e também a página do Facebook.

E claro, comente o que achou e se ainda estiver com dúvidas, é só me falar.

Abraço e até a próxima.
