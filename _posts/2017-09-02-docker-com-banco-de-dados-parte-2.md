---
layout: post
title: Docker, banco de dados e ambiente de desenvolvimento - parte 2
subtitle: Não instale banco de dados direto no seu computador, use o Docker
bigimg: /img/posts/2017/docker.png
tags: [docker, postgresql]
author: author-normandes.html
---

Na parte 2 da série, vamos aprender a fazer um restore do banco de dados e como executar um script assim que o banco for criado. 

Não leu a parte 1, [clique aqui](http://blog.hibicode.com/2017-08-20-docker-com-banco-de-dados/){:target="_blank"} para ler.

### Restore de backup

Uma situação muito comum que acontece quando estamos programando um sistema grande, é pegar uma base de dados de alguma outra pessoa que já contém uma série de registros inseridos, e assim a gente implementa/testa alguma nova funcionalidade ou mesmo simula algum possível bug.

Dessa forma você ganha tempo não tendo que inserir todos os dados que a outra pessoa já fez, lembre-se, esse é um cenário de um sistema grande. ;)

### Postgres

Para fazermos o restore no Postgres precisamos usar a ferramenta "pg_restore" junto com o arquivo que a outra pessoa te enviou.

O problema nesse caso é - nós não instalamos o Postgres no nosso computador, lembra? Estamos usando o Docker!

Mas no contêiner do Postgres nós temos essa ferramenta instalada, basta apenas você aprender a usá-la a partir do *docker*.

É bem simples mesmo, veja o comando abaixo e logo depois as explicações:

~~~
$ docker exec postgres-projeto-a pg_restore -U usuario -d instancia_banco_de_dados < arquivo_dump
~~~

O *exec* faz a execução do comando *pg_restore* dentro do contêiner de nome *postgres-projeto-a*, as opções -U e -d dizem que é para executar com o usuário na instância do banco respectivamente, por fim passamos o arquivo de dump gerado pelo(a) seu(sua) colega.

Isso é muito útil! Irá salvar muito do seu tempo. :)

### Executando arquivos de inicialização

Essa outra dica irá te ajudar quando precisar executar algum script assim que o banco for criado, a maior parte dos parâmetros abaixo você já conhece, a única diferença é o -v:

~~~
$ docker run -p 5432:5432 --name postgres-projeto-a -e POSTGRES_USER=usuario -e POSTGRES_PASSWORD=senha -e POSTGRES_DB=instancia_banco_de_dados -v ~/projetos/projeto-a/scripts:/docker-entrypoint-initdb.d -d postgres:9.6.1
~~~

O *-v* diz que iremos mapear uma pasta no nosso computador para o *docker-entrypoint-initdb.d* que está depois dos dois pontos.

Isso faz com que os scripts sql que você colocar em *~/projetos/projeto-a/scripts* serão executados no banco de dados.

Eu usei essa opção em um projeto que não usava migrações de dados (Flyway por exemplo) ainda, e lá eu coloquei os scripts de inicialização do banco.

Para você testar, coloque qualquer *create table* em um arquivo .sql, assim que acessar o banco de dados com seu IDE preferido irá ver a magia acontecendo! :)

### Próximo post

No próximo post irei mostrar como colocar uma aplicação Java rodando no Docker, sugestão dada pelo Vinicus nos comentários da parte 1.

Gostou do que leu? Comente e principalmente compartilhe com seus amigos, pode ajudar alguém por aí. :)

E se inscreva na minha lista, te prometo não mandar spam, serão só e-mails com conteúdos da hibi code. 

Abraço.
