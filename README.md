# Docker: Utilização prática no cenário de Microsserviços
### Denilson Bonatti, Instrutor - Digital Innovation One


Muito se tem falado de containers e consequentemente do Docker no ambiente de desenvolvimento. Mas qual a real função de um container no cenários de microsserviços? Qual a real função e quais exemplos práticos podem ser aplicados no dia a dia?


## Situação a ser resolvida

Sakura Haruno deseja mover os dados de seus pacientes e agendas de consultas do Datacenter local para a Nuvem, e além disso deseja mudar a estrutura do software utizado atualmente de monolítico para microsserviços.


## Criação do container MySQL

Crie no ambiente de nuvem escolhido 3 máquinas virtuais (podem ser máquinas básicas de 1 GB de RAM e 1 core de processamento apenas).

Em uma das VMs crie um container do Docker com o microsserviço desejado, nesse caso um banco de dados.

Acesse o site do [Docker Hub](https://hub.docker.com/) procure pela imagem do MySQL para conferir os comandos para criação do banco em contâiner Docker.

Execute o comando *docker pull mysql* para realizar o download da imagem na máquina desejada.

Com o comando *docker images* é possível checar as imagens já baixadas.

Para executar imagem utilize o comando *docker run mysql* (sendo mysql o nome da imagem).

Para checar os containers em execução utilize *docker ps*. E com o comando *docker ps -a* é possível verificar os containers executados recentemente.

Usando o comando *docker run mysql sleep 10* o container irá executar por 10 segundos e irá parar quando o tempo indicado acabar.

Utilize o comando *docker stop* para parar um container em execução, para especificar o container que será parado basta informar o nome ou o ID do container após *stop*.

Com o comando *docker run --help* é possível verificar as demais funcionalidades que podem ser usadas junto com o comando *docker run*. Por exemplo, *docker run -it ubuntu* sendo que o t aloca um pseudo terminal e o i para acionar o modo interativo no container, para poder acessar o bash e trabalhar com o sistema operacional dentro do container.

Utilizando um aplicativo gerenciador de banco de dados (por exemplo Sequeler), adicione um novo banco de dados, neste caso será o MySQL criado anteriormente, informando o IP público, os dados de acesso, usuário, senha e a porta 3306.

![D1 01](https://github.com/user-attachments/assets/948839c5-3a09-4b29-9e9d-8d764b91b8b8)

Crie uma tabela no banco de dados, como a tabela do arquivo banco.sql.

Crie um microsserviço dentro do APP, se estiver uitlizando Linux siga o caminho abaixo:
- cd /var/lib/docker/volumes

De um comando ls para listar o conteúdo, com o comando cd app entre na pasta app e dê o comando ls para listar.

Com o comando cd _data entre na pasta data onde será criado o microsserviço:

![D1 02](https://github.com/user-attachments/assets/d8d415ac-b8b7-4646-9736-f55244c1ccaa)

Digite o comando abaixo para abrir o editor de código nano criando arquivo index.php:
- nano index.php

Obs.: Neste código php foi utilizado o mysqli para realizar a conexão, o que não é aconselhável, mas foi usado apenas a título de demonstração para que seja rápido.

Dê um Ctrl O para salvar e um Ctrl X para sair do editor de código.

Crie um container Docker para armazenar o arquivo index.php, com o servidor web Apache e o PHP instalado:
- /var/lib/docker/volumes/app/_data# docker run --name web-server -dt -p 80:80 -mount type=volume,src=app,dst=/app/ webdevops/php-apache:alpine-php7

Após isso, execute o comando abaixo para verificar os containers rodando:
- docker ps


### Estressando o container

Acesse o site [loader](https://loader.io/), crie uma conta, crie um Target Host informando o domínio IP, por exemplo http//:34.201.104.100

No container crie um arquivo com o nome gerado:

![D1 03](https://github.com/user-attachments/assets/9f281ea1-ce58-471a-beb7-a2e10a07235f)

![D1 04](https://github.com/user-attachments/assets/d29a9ad0-3eec-45ac-8fc1-c20c818ea36f)

E no conteúdo do arquivo informe o mesmo nome.

Lembre-se de criar o arquivo no diretório /var/lib/docker/volumes/app/_data.

Retornando ao site [loader](https://loader.io/) clique em Verify e deverá aparecer uma mensagem informando que a verificação passou.

Na aba Tests crie um novo teste informando:
- Nome
- Quantidade de clientes: 250
- Duração: 1 minuto
- Método: GET
- Protocolo: HTTP
- Host: 34.201.104.100
- Path: index.php

Após clique em *Run test* e analise as informações do teste:

![D1 05](https://github.com/user-attachments/assets/02139851-806d-4331-a4f7-0547d8a6dda0)


### Iniciando um Cluster Swarm

Em /var/lib/docker/volumes/app/_data xecute o comando docker ps para verificar os containers ativos.

Execute o comando abaixo para excluir o container da aplicação web-server:
- docker rm --force web-server

Execute o comando abaixo para iniciar um Docker Swarm:
- docker swarm init

Agora será necessário adicionar os demais servidores ou VMs ao Swarm, para isso acesse a VM desejada remotamente, entre como usuário root, e execute o comando abaixo:

![D1 06](https://github.com/user-attachments/assets/0e84b8bf-7d5e-45f0-b4f1-31fd4dbcdaa9)

Nesse caso o IP 172.31.0 127 é o IP de acesso local entre as máquinas virtuais onde o Docker Swarm foi criado, e a porta 2377 é a porta específica utilizada pelo Docker Swarm e esta deve ser liberada no firewall para que ele funcione.

Execute o mesmo comando acima na terceira máquina virtual.

Após execute o comando abaixo para verificar os nós pertencentes ao cluster:
- docker node ls

Deverá retornar a seguinte tela:

![D1 07](https://github.com/user-attachments/assets/0efa3cfd-603f-435c-a373-9fcbfeb5c65c)


### Criando um serviço no Cluster Swarm

Execute o comando abaixo na VM ou servidor principal:

![D1 08](https://github.com/user-attachments/assets/8e6c4c0e-5b5a-43d6-976a-8d0dafc1ff6e)

Onde:
- docker service create: é o comando para criar um serviço de containers
- --name web-server: é o nome do serviço
- --replicas 10: quantidade de réplicas para o container desejado, no caso o container php
- -dt: executar em segundo plano
- -p 80:80: liberar o container na porta 80
- --mount type=volume: montar o volume
- src=app: onde está a aplicação
- dst=/app/: para onde vai o container criado
- webdevops/php-apache:alpine-php7: é a imagem

Execute o comando abaixo para saber onde foram replicados os containers:
- docker service ps web-server


### Replicando um volume dentro do Cluster

Da forma como o cluster foi configurado ele ainda não está replicando os volumes para as demais VMs.

Para isso será necessário replicar a pasta /var/lib/docker/volumes/app/_data para as VMs 2 e 3.

Na VM 1 execute o seguinte comando:
- apt-get install nfs-server

No segundo e terceiro servidores ou VMs execute o comando para instalar somente as bibliotecas necessárias para a máquina cliente:
- apt-get install nfs-common

Retornando ao servidor 1, na pasta /var/lib/docker/volumes/app/_data crie umarquivo conforme o comando abaixo:
- nano /etc/exports

Abra o qruivo e configure-o da seguinte forma:

![D1 09](https://github.com/user-attachments/assets/aa0ced44-5039-424f-a1a0-686168fcd6a1)

Obs.: não é indicado usar o * pois ele irá liberar as permissões indicadas (leitura, escrita, sincronização e checar todas as subpastas) para qualquer um que tiver acesso à pasta.

Volte para a pasta /var/lib/docker/volumes/app/_data e execute o comando abaixo para exportar a pasta:
- exportfs -ar

Executando o comando **showmount -e** ele mostra as pastas da VM que estão sendo compartilhadas:

![D1 10](https://github.com/user-attachments/assets/f83a39e1-770e-45cf-b6b7-0e4d37720ded)

Na VM 2 e 3 execute o comando abaixo que vai criar a pasta /var/lib/docker/volumes/app/_data na mesma pasta /var/lib/docker/volumes/app/_data das VMs:

![D1 11](https://github.com/user-attachments/assets/4f6e3adc-3499-4ae8-bda6-d29fe58fe876)

Após isso ao executar o comando ls nas demais máquinas virtuais será possível verificar que os arquivos da VM 1 foram replicados para elas:

![D1 12](https://github.com/user-attachments/assets/6abd788a-85d9-47bb-bdc5-f31432e562c6)


### Criando um proxy utilizando NGINX

Será criado um proxy para que quando uma requisição chegar em uma das máquinas ela seja replicada para todas as demais automaticamente. Dessa forma, quando a aplicação estiver recebendo muitos acessos, esses acessos serão espalhados para cada uma das máquinas.

Será um proxy interno criado dentro de um container utilizando NGINX. O NGINX também faz o balanceamento de carga entre os servidores se tiver as configurações para isso.

Digite o comando abaixo para criar uma pasta chamada proxy:
![D1 13](https://github.com/user-attachments/assets/c3c5dc0d-413b-4db8-8ed0-44b74f731bd6)

Crie um arquivo chamado nginx.conf conforme comando abaixo com o conteúdo conforme o arquivo [nginx.conf](https://github.com/Hisly-A/sakura-haruno/blob/main/nginx.conf):
![D1 14](https://github.com/user-attachments/assets/51c9b13b-3278-4280-bbd4-2bfca6832b5d)

Após deve ser criado um arquivo de configuração do container para especificar a imagem que será usada e o que será feito com a imagem, nesse caso será uma imagem do nginx e ela será enviada para o container, com o conteúdo do arquivo [dockerfile](https://github.com/Hisly-A/sakura-haruno/blob/main/dockerfile):
![D1 15](https://github.com/user-attachments/assets/8989cadd-defb-46f1-8bcf-b3828668023c)

Crie um containercom essa configuração que acabou de criar:
![D1 16](https://github.com/user-attachments/assets/49268462-a057-4f05-a043-e2c320910d44)

O ponto final no comando acima indica o arquivo dockerfile.

Após executar o comando acima execute o comando docker image ls para verificar os arquivos criados no container:
![D1 17](https://github.com/user-attachments/assets/2846b612-8166-4b1f-8937-9416e6cd417e)

Utilize o comando docker container run para rodar o container com a imagem recém criada:
![D1 18](https://github.com/user-attachments/assets/350bd146-4c21-4656-af41-fca38a438570)

Verifique que o container proxy-app está em execução:
![D1 19](https://github.com/user-attachments/assets/54c31851-e47d-4358-be72-e0db82926607)


### Estressando o cluster

Agora vamos estressar o proxy para verificar se ele realmente vai distribuir a carga entre os servidores do cluster.

No site [loader](https://loader.io/) crie um novo target host com o IP do servidor principal utilizando a porta 4500:

![D1 20](https://github.com/user-attachments/assets/1df90754-c152-498f-b0ec-2ccf43e5e65a)

Com o novo host criado será necessário informar o novo arquivo de identificação no servidor:

![D1 21](https://github.com/user-attachments/assets/0456259c-2a83-4536-89af-a9053cf40e5c)

![D1 22](https://github.com/user-attachments/assets/1fb1ac68-cc5b-4a9a-8b97-0167661b3733)

![D1 23](https://github.com/user-attachments/assets/002d9943-8714-48cf-ba95-3b6d9847d820)

![D1 24](https://github.com/user-attachments/assets/51876cee-3052-43f0-a08a-e790dc8e8339)

Na aba Tests crie um novo teste informando:
- Nome
- Quantidade de clientes: 300
- Duração: 1 minuto
- Método: GET
- Protocolo: HTTP
- Host: 34.201.104.100:4500
- Path: index.php

Execute o teste e verifique os resultados:

![D1 25](https://github.com/user-attachments/assets/ae0e7418-6337-4bf1-947c-9e32e6748d58)

Ao verificar a inclusão dos dados no banco será possível ver que os dados agora foram criados a partir de IPs diferentes, ou seja IPs das máquinas 2 e 3:

![D1 26](https://github.com/user-attachments/assets/341c3979-5e8d-4ca8-943d-33a7b3f69097)


