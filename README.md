# Docker: Utilização prática no cenário de Microsserviços
### Denilson Bonatti, Instrutor - Digital Innovation One


Muito se tem falado de containers e consequentemente do Docker no ambiente de desenvolvimento. Mas qual a real função de um container no cenários de microsserviços? Qual a real função e quais exemplos práticos podem ser aplicados no dia a dia?


## Situação a ser resolvida

Sakura Haruno deseja mover os dados de seus pacientes e agendas de consultas do Datacenter local para a Nuvem, e além disso deseja mudar a estrutura do software utizado atualmente de monolítico para microsserviços.


## Como resolver

Será criado um Docker Swarm. Para isso crie no ambiente de nuvem escolhido 3 máquinas virtuais (podem ser máquinas básicas de 1 GB de RAM e 1 core de processamento apenas).

Em uma das VMs crie um container do Docker com o microsserviço desejado, nesse caso um banco de dados.

Acesse o site do Docker Hub https://hub.docker.com/ procure pela imagem do MySQL para conferir os comandos para criação do banco em contâiner Docker.

Utilizando um aplicativo gerenciador de banco de dados (por exemplo Sequeler), adicione um novo banco de dados, neste caso será o MySQL criado anteriormente, informando o IP público, os dados de acesso, usuário, senha e a porta 3306.

![D1 01](https://github.com/user-attachments/assets/948839c5-3a09-4b29-9e9d-8d764b91b8b8)

Crie uma tabela no banco de dados, como a tabela do arquivo banco.sql.

Crie um microsserviço dentro do APP, se estiver uitlizando Linux siga o caminho abaixo:
'cd /var/lib/docker/volumes'

De um comando ls para listar o conteúdo, com o comando cd app entre na pasta app e dê o comando ls para listar.

Com o comando cd _data entre na pasta data onde será criado o microsserviço:

![D1 02](https://github.com/user-attachments/assets/d8d415ac-b8b7-4646-9736-f55244c1ccaa)

Digite o comando abaixo para abrir o editor de código nano criando arquivo index.php:
'nano index.php'

Obs.: Neste código php foi utilizado o mysqli para realizar a conexão, o que não é aconselhável, mas foi usado apenas a título de demonstração para que seja rápido.

Dê um Ctrl O para salvar e um Ctrl X para sair do editor de código.

Crie um container Docker para armazenar o arquivo index.php, com o servidor web Apache e o PHP instalado:
'/var/lib/docker/volumes/app/_data# docker run --name web-server -dt -p 80:80 -mount type=volume,src=app,dst=/app/ webdevops/php-apache:alpine-php7'

Após isso, execute o comando abaixo para verificar os containers rodando:
'docker ps'

### Estressando o container


