---
layout: post
title: Docker, banco de dados e ambiente de desenvolvimento - parte 1
subtitle: Não instale banco de dados direto no seu computador, use o Docker
bigimg: /img/posts/2017/docker.png
tags: [docker, mysql, postgresql]
author: author-normandes.html
---

Quer começar a aprender Docker? Quer melhorar o gerenciamento dos bancos de dados na sua máquina local? Quer ter duas ou mais versões do MySQL ou Postgres instalado no seu computador? 

Continue comigo até o fim dessa série, tenho certeza que você vai gostar muito!

Normalmente nós desenvolvedores precisamos de um banco de dados instalado para poder desenvolver nossas aplicações, seja o MySQL, PostgreSQL ou algum outro.

Você também pode precisar de duas versões diferentes de um mesmo banco, por exemplo, o projeto A usa o MySQL 5.6, já a aplicação B precisa do MySQL 5.7.

Pode ser que você alterne entre projetos, e um use o MySQL e o outro o PostgreSQL.

Agora imagine você instalando, desinstalando e mantendo essa quantidade de bancos de dados ai no seu computador? É bem trabalhoso, não é verdade?

### Docker

E é nesse momento que você fica muito feliz em conhecer o **Docker**!

Meu objetivo com esse post é te mostrar como o Docker vai facilitar sua vida para instalar, iniciar, parar e fazer tudo que você precisa para ter um banco de dados local para desenvolvimento.

O legal é que vai funcionar se sua máquina é Linux, Mac ou Windows! Eu nunca tentei usar o Docker no Windows, apesar de ver na documentação que funciona, por favor, se tiver algum problema, comente aqui pra nós aprendermos juntos.

### Docker CE

O primeiro passo é instalar o Docker Community Edition que é grátis. Baixe a versão para o seu sistema operacional [clicando aqui](https://www.docker.com/community-edition#/download){:target="_blank"}.

Depois de instalado, vá até a prompt de comando e veja se está tudo certo com o comando mostrado abaixo (claro que a versão pode ser diferente dependendo de quando você estiver lendo esse post):

![docker --version](/img/posts/2017/docker-version.png){: .center}

### Imagem e contêiner

Dois conceitos muito importantes que você precisa aprender sobre o Docker é a diferença entre uma _imagem_ e um _contêiner_.

E, como imagino que você é programador, sabe a diferença entre uma _classe_ e um _objeto_, certo?

No Docker, uma _imagem_ está para uma _classe_, e um _contêiner_ seria mais parecido com um _objeto_.

Ou seja, a partir de uma imagem podemos criar vários contêiners.

Portanto, quando você precisar de alguma aplicação, como no nosso caso, vamos instalar o MySQL e o PostgreSQL, nos perguntamos: _"Já existe alguma imagem pronta pro que eu preciso?"_

### Docker Hub

Talvez sua próxima pergunta seja, será que existe um repositório onde dá pra pesquisar imagens? E a resposta é sim, é só acessar o [Docker Hub](https://hub.docker.com/){:target="_blank"}. Existem milhares de imagens prontas pra nós usarmos.

Mas vamos seguir, meu objetivo com esse post é te deixar usando o MySQL e o PostgreSQL sem precisar instalar direto na sua máquina, mas sim através do Docker, em outro momento eu crio outro post para explicar mais detalhes, combinado?

### Criando um contêiner do MySQL

Voltando para a linha de comando, vamos baixar uma imagem e criar um contêiner com o MySQL instalado e pronto para usarmos.

Você pode estar pensando? _Vamos para a sequência de comandos..._ na verdade não, será apenas um único comando para baixarmos a imagem, criamos o contêiner e já deixar o MySQL pronto!

Vamos imaginar que você precisa da versão 5.6 do MySQL e que a senha do usuário root também seja _root_. O comando abaixo pode demorar um pouquinho pra terminar:

~~~
$ docker run -p 3306:3306 --name mysql-5.6 -e MYSQL_ROOT_PASSWORD=root -d mysql:5.6
~~~

Na primeira vez que você executar o comando acima, o Docker procura a imagem do MySQL no seu computador, como não a encontra, irá baixar na internet e isso pode demorar um pouco, são aproximadamente 300MB.

Vamos entender os detalhes do comando acima:

* run: serve para rodarmos um comando em um contêiner, no caso do MySQL estamos iniciando o serviço de banco de dados

* -p 3306:3306: aqui estamos dizendo que queremos que a porta 3306 de dentro do contêiner seja mapeada para a porta 3306 local

* --name mysql-5.6: é o nome do contêiner, você que escolhe. Normalmente eu coloco o nome do projeto que estou trabalhando com esse banco, por exemplo: --name mysql-projeto-a

* -e MYSQL_ROOT_PASSWORD=root: as propriedades passadas com o -e são variáveis de ambiente passadas para o contêiner, nesse caso o contêiner do MySQL pode ser configurado com _MYSQL_ROOT_PASSWORD_ para trocar a senha do usuário root.

* -d: serve para iniciar o contêiner em background

* mysql:5.6: o nome da imagem e versão. Repare que eu especifiquei a versão depois dos dois pontos, caso eu não informasse, seria baixado a última versão do MySQL disponível no Docker Hub.

Use a IDE de sua preferência para conectar no MySQL, você usará o endereço _localhost_ com a porta 3306 e usuário e senha definidos como _root_.

Para conferir se o contêiner está no ar, rode:

~~~
$ docker ps
~~~

Você verá algo como: 

![docker --version](/img/posts/2017/docker-ps-mysql-1.png){: .center}

### Criando um contêiner do PostgreSQL

Para finalizar esse post vamos criar um contêiner com o PostgreSQL, bem rápido e simples também.

~~~
$ docker run -p 5432:5432 --name postgres-projeto-a -e POSTGRES_USER=usuario -e POSTGRES_PASSWORD=senha -e POSTGRES_DB=instancia_banco_de_dados -d postgres:9.6.1
~~~

Da mesma forma que o MySQL, o que fizemos no comando acima foi:

* -p 5432:5432: expondo a porta interna do contêiner para acesso local, na sua máquina

* --name postgres-projeto-a: é o nome do contêiner, normalmente o nome do projeto que você vai trabalhar

* -e <PROPRIEDADE>=<VALOR>: todos esses valores são para configurar o contêiner, no caso do PostgreSQL, trocamos o usuário, senha e o nome da instância do banco que ele já vai criar

* postgres:9.6.1 é a imagem e a versão do Postgres que iremos utilizar

Pronto! Sim, já temos um Postgres instalado e pronto para usar. Use a IDE de sua preferência e veja você mesmo. :)

### Parando e iniciando um contêiner

Agora é bem provável que você esteja se perguntando? Mas como eu faço para parar o MySQL ou o Postgre?

$ docker stop _nome-do-contêiner_

Agora vem um segredo... se você usar o _docker ps_ agora não irá ver nenhum contêiner listado, pois o _ps_ mostra apenas os contêiners iniciados, mas se passar a opção _-a_ ai sim você verá a lista de todos eles.

~~~
$ docker ps -a
~~~

E para iniciar o contêiner novamente, use:

$ docker start _nome-do-contêiner_

Simples assim mesmo!

### Próximo artigo

Para o próximo artigo iremos avançar mais no que eu uso no meu dia a dia com o Docker para desenvolvimento.

Irei te mostrar como mapear uma pasta no meu computador para executar scripts SQL ao iniciar o contêiner e outra dica muito útil será como carregar um dump no Postgres.

Se quiser ser notificado quando o post sair, é só colocar seu e-mail na nossa lista abaixo. Fique tranquilo, eu também detesto spam, só te enviarei conteúdo que eu achar muito relevante, e você também poderá se desinscrever a qualquer momento.

Também comente o que achou do artigo e o que você espera do próximo, combinado?

Grande abraço e até a próxima.
