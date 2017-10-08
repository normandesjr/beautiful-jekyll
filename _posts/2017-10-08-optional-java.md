---
layout: post
title: Usando a classe Optional do Java
subtitle: O Java 9 está aí, mas será que você já usa o Java 8?
bigimg: /img/posts/2017/logo-java-otim.png
tags: [java, java 8, boas práticas]
author: author-normandes.html
---

O Java 8 já está ficando pra trás, a versão 9 já foi lançada, mas eu tenho visto que muitas pessoas ainda não sabem usar bem as features da versão passada, bora começar a praticar?

Recentemente precisei fazer a transformação de uma lista de objetos para uma lista de inteiros para atender uma regra de negócio no projeto que estou trabalhando atualmente.

Lembrei que a solução que usei com *Optional* e *Stream* poderia ser útil para alguém, foi aí que tive a ideia de compartilhar aqui. :)

Quero te mostrar através de uma evolução como seria feito pré e pós Java 8, combinado?

### Modelo de dados

Tenho uma classe *NotaFiscal* que contém vários *produtos*, e a regra que implementei foi: dada uma nota fiscal, retorne todos os ids dos produtos contidos nela.

{% highlight java %}
public class NotaFiscal {

    private Long id;
    private String numero;
    private List<Produto> produtos;

    // ...
}
{% endhighlight %}

E produto seria:

{% highlight java %}
public class Produto {

    private Long id;
    private String nome;

    // ...
}
{% endhighlight %}

### Teste

Eu preciso criar um método em *NotaFiscal* que retorne a lista de ids dos produtos, veja o teste que criei:

{% highlight java %}
public class NotaFiscalTest {

    private NotaFiscal notaFiscal;

    @Before
    public void inicializar() {
        notaFiscal = new NotaFiscal();
    }

    @Test
    public void deve_recuperar_todos_ids_dos_produtos_da_nota_fiscal() throws Exception {
        notaFiscal.adicionarProduto(new Produto(1L, "Sabonete")).adicionarProduto(new Produto(2L, "Pasta Dental"))
                .adicionarProduto(new Produto(3L, "Desodorante"));

        final List<Long> idsProdutos = notaFiscal.getIdsProdutos();

        assertThat(idsProdutos, hasItems(1L, 2L, 3L));
    }

}
{% endhighlight %}

Talvez você estranhe duas coisas no código acima, primeiro o nome do método separado por underscore "_", e o segundo o *assertThat* com *hasItems*.

Bem, não é uma prática seguida por muitas pessoas usar o nome do método de testes separado por underscore, pois ele precisa dar uma boa descrição do que o teste faz, mas só faça isso nos testes, beleza? :)

E a segunda "estranheza" é a biblioteca do [Hamcrest](http://hamcrest.org/JavaHamcrest/) que deixa o nosso código muito mais fluído.

### Primeira implementação

Se rodarmos o código, obviamente irá falhar, nem implementei ainda a primeira versão! :)

Usar TDD é muito legal para regras assim, te ajuda muito a pensar na API e já deixar seu código testável.

Não conhece TDD? Quer aprender mais? Deixe seu comentário, isso me motivará a escrever mais sobre o tema.

Voltando ao código, vamos para a primeira implementação possível:

{% highlight java %}
public List<Long> getIdsProdutos() {
    final List<Long> ids = new ArrayList<>();
    if (produtos != null) {
        for (Produto produto : produtos) {
            ids.add(produto.getId());
        }
    }
    return ids;
}
{% endhighlight %}

Repare que estou protegendo o código de um possível *NullPointerException* ao verificar se a lista de produtos é diferente de *null*, e isso também é algo que você deve se preocupar, proteja seu código. ;)

### Primeira refatoração

Se rodarmos o código irá funcionar e iremos ver a barrinha verde no Eclipse (ou na sua IDE de preferência), porém o código está muito verboso, cheio de linhas de código para fazer algo aparentemente simples, não dá pra melhorar?

Dá sim, é aí que entra a API do Java 8 que você PRECISA começar a usar! hehe

Olha que legal, podemos refatorar sem medo, já temos um teste automatizado que roda em menos de 1 segundo e garante que o que fizermos não será estragado. 

Sabendo disso, refatorei para o seguinte código:

{% highlight java %}
public List<Long> getIdsProdutos() {
    final List<Long> ids = new ArrayList<>();
    if (produtos != null) {
        produtos.forEach(p -> ids.add(p.getId()));
    }
    return ids;
}
{% endhighlight %}

Rode o teste e você verá que tudo continua funcionando lindamente, mas aí pensei... não daria pra tirar esse "if" e esse *new ArrayList* daí? E a resposta é sim! Vamos lá!

### Segunda refatoração

Nós podemos usar a classe *Optional* para trabalhar com possíveis valores *null*, assim eu poderia facilmente tirar o *if*.

Com a função *map* e um *Collector* eu poderia tirar o *new ArrayList*, e o resultado final foi o seguinte:

{% highlight java %}
public List<Long> getIdsProdutos() {
    return Optional.ofNullable(produtos)
            .map(List::stream).orElse(Stream.empty())
            .map(Produto::getId)
            .collect(Collectors.toList());
}
{% endhighlight %}

Rode o teste, prometo que continuará funcionando. :)

Bem, o que fiz nessa última refatoração foi criar um *Optional* de produtos que pode ser nulo, e caso seja, um stream vazio é retornado.

Nesse stream, que pode ser ou não vazio, fazemos o mapeamento pelo id do produto e coletamos o resultado em uma lista.

A forma que está descrita nos dois parágrafos acima é a forma que você lê no seu código, muito mais simples e direto do que na primeira implementação, não acha?

### Conclusão

Eu sei, talvez sua resposta foi "não, não acho mais simples" para a pergunta acima. Acontece que é só uma questão de prática, toda mudança gera um desconforto, mas te garanto que logo logo você aprende e achará mais rápido e simples a forma mais fluída.

Eu usei isso de verdade, não foi exatamente esse exemplo, mas é muito parecido. No final eu só fiz uns imports estáticos para ficar menor o código.

Veja o código completo no meu repositório do GitHub em [https://github.com/normandesjr/exemplo-optional-1](https://github.com/normandesjr/exemplo-optional-1)

O que achou? Comente abaixo pra eu saber como posso te ajudar mais, combinado?

Abraço e até a próxima.
