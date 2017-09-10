---
layout: post
title: Docker com Java  - parte 3
subtitle: Finalizando a série com uma aplicação Spring Java no Docker
bigimg: /img/posts/2017/docker.png
tags: [docker, java, spring]
author: author-normandes.html
---

Nessa última parte da série sobre Docker no ambiente de desenvolvimento, irei te mostrar como colocar uma aplicação com Spring Boot para rodar em um contêiner.

Não leu a parte 1, [clique aqui](http://blog.hibicode.com/2017-08-20-docker-com-banco-de-dados/){:target="_blank"}.

E a parte 2, [clique aqui](http://blog.hibicode.com/2017-09-02-docker-com-banco-de-dados-parte-2/){:target="_blank"}.

### Onde aplicar

Você irá perceber, mas aqui na Hibi Code você só aprenderá assuntos que são usados na prática mesmo, que as empresas usam no seu dia a dia.

Portanto, esse cenário que você vai aprender agora pode ser usado por exemplo em um ambiente de homologação da aplicação, a empresa que você trabalha poderia usar o [Rancher](http://rancher.com/){:target="_blank"} para gerenciar os contêineres.

### Aplicação

Iremos usar o Spring Boot para criar uma API REST muito simples, o código você pode pegar no [GitHub](https://github.com/normandesjr/api-exemplo-docker){:target="_blank"}.

Por falar nisso, como está seu conhecimento em Git? GitHub? Quer algum conteúdo sobre esse tema? Comente aqui pra eu saber. ;)

A aplicação retorna uma lista de clientes na URL /clientes:

{% highlight java %}
@RestController
@RequestMapping("/clientes")
public class ClienteResource {

	@GetMapping
	public List<Cliente> findClientes() {
		return Arrays.asList(new Cliente(1L, "Pedro Silva", "pedro@email.com"),
				new Cliente(2L, "Maria Santos", "maria@email.com"));
	}

}
{% endhighlight %}

### Dockerfile

Quando você baixar o código do GitHub, irá perceber um arquivo chamado Dockerfile na raiz do projeto.

Esse arquivo irá nos ajudar a criar uma imagem Docker. Lembra do primeiro post sobre imagem? Então, a imagem será a base para criarmos nossos contêineres.

O conteúdo desse arquivo é bem simples, deixei o mínimo pra você poder começar, veja só:

~~~
FROM openjdk:8-jdk-alpine
ADD target/api-exemplo-docker-1.0.0-SNAPSHOT.jar app.jar
ENTRYPOINT [ "sh", "-c", "java -jar /app.jar" ]
~~~

Na primeira linha estamos dizendo que a imagem que estamos criando agora, é baseada na _openjdk:8-jdk-alpine_, que vem com o Java 8 pronto para usarmos.

Na segunda linha estamos adicionando o arquivo da aplicação _api-exemplo-docker-1.0.0-SNAPSHOT.jar_ com o nome de app.jar.

E na última linha ensinando ao contêiner como executar a aplicação, que é simplesmente executar na linha de comando: java -jar app.jar

### Criando a imagem

Para criar a imagem, usamos um plugin que o Spotify criou [https://github.com/spotify/docker-maven-plugin](https://github.com/spotify/docker-maven-plugin){:target="_blank"}.

Vá pela linha de comando até a pasta do projeto e rode:

~~~
$ ./mvnw install dockerfile:build
~~~

Se você está no Windows use o mvnw.cmd.

Na primeira vez o comando irá demorar um pouco, pois tem muita coisa pra baixar na internet. :)

Nesse momento você criou apenas a imagem, o contêiner ainda não foi criado. Para listar as imagens que você tem baixada, use o comando:

~~~
$ docker image list
~~~

Repare a entrada _hibicode/api-exemplo-docker_, é justamente a imagem que acabamos de criar! Esse nome está configurado no pom.xml na tag _repository_ formada :

{% highlight xml %}
<plugin>
  <groupId>com.spotify</groupId>
  <artifactId>dockerfile-maven-plugin</artifactId>
  <version>1.3.5</version>
  <configuration>
    <repository>${docker.image.prefix}/${project.artifactId}</repository>
  </configuration>
</plugin>
{% endhighlight %}

O _docker.image.prefix_ é uma propriedade definida no pom.xml e o _project.artifactId_ é o _artifactId_ do projeto.

### Criando o contêiner

Pronto, agora é só criar o contêiner e usar a aplicação.

Repare que com o _Dockerfile_ você pode distribuir sua aplicação em qualquer ambiente que rode o Docker, simples assim mesmo!

Eu poderia facilmente falar para o pessoal da infraestrutura fazer o deploy da aplicação no ambiente com Docker, nós iremos ver isso acontecendo a cada dia mais nas empresas.

Mas vamos lá, criar o contêiner agora é muito fácil, você já até sabe como é:

~~~
$ docker run -p 8080:8080 --name hibicode-spring-docker -d hibicode/api-exemplo-docker
~~~

Pronto, abra o browser e acesse - http://localhost:8080/clientes 

### Próximos passos

O objetivo dessa série foi fazer uma breve introdução prática sobre como você pode usar o Docker no seu dia a dia como programador.

Mas foi uma introdução mesmo, temos muito mais a aprender com ele, direto eu tenho aprendido algo novo enquanto vou testando e aplicando aqui na empresa.

Meu conselho pra você é, sempre que possível use-o! Só assim você irá se sentir mais seguro e confiante, e com certeza irá ajudar muito na sua carreira.

Gostaria muito do seu comentário final, o que achou da série? Gostaria de mais conteúdos assim?

Eu tenho alguns assuntos em mente para os próximos conteúdos, logo logo tem mais, por isso, se inscreva na lista e siga no Facebook, assim você ficará sabendo mais rápido sobre os novos conteúdos.

Abraço.
