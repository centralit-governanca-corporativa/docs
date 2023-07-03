# Como criar um ambiente local usando o orquestrador de container Docker Swarm
Existem duas maneiras de se executar a aplicação, uma delas é usando o [Docker Compose](https://docs.docker.com/compose/) para gerar uma build da aplicação, a outra forma é a que será coberta nessa documento é executando a aplicação diretamente de sua máquina subindo os serviços necessários no [Docker Swarm](https://docs.docker.com/engine/swarm/).

## Passo 1: Download do repositório
Após receber acesso à organização do Github [Central IT](https://github.com/centralit-governanca-corporativa), é necessário clonar o repositório de nossa aplicação principal [citbot](https://github.com/centralit-governanca-corporativa/citbot), conhecido também como projeto AURA.

>Para obter mais informações sobre repositórios, confira "[Sobre repositórios remote](https://docs.github.com/pt/get-started/getting-started-with-git/about-remote-repositories)"

## Passo 2: Instalação do Docker
Docker é uma plataforma aberta para desenvolvimento, envio e execução de aplicativos. O Docker permite que você separe seus aplicativos de sua infraestrutura para que você possa entregar software rapidamente.

> Para obter mais informações sobre o Docker acesse a [documentação oficial](https://docs.docker.com/)

Siga o passo a passo oficial para realizar a instalação do Docker escolhendo entre a versão desktop ou de servidor.

[Instalar o Docker Engine](https://docs.docker.com/engine/install/)

## Passo 3: Salvar os arquivos  `yml` de serviços Docker
Arquivo `postgres.yml` contendo os serviços de banco de dados [Postgres](https://www.postgresql.org/) e  o SGBD [pgAdmin4](https://www.pgadmin.org/).
```yml
version: '3.2'
services:
  postgres-master:
    image: docker.io/bitnami/postgresql
    hostname: postgresql-master
    volumes:
      - "postgres_volume:/bitnami/postgresql"
    ports: 
      - "5432:5432"
    environment:
      - POSTGRESQL_REPLICATION_MODE=master
      - POSTGRESQL_REPLICATION_USER=replication_user
      - POSTGRESQL_REPLICATION_PASSWORD=admin
      - POSTGRES_USERNAME=postgres
      - POSTGRES_PASSWORD=admin
      - POSTGRESQL_DATABASE=postgres
  pgadmin:
    image: dpage/pgadmin4
    environment: 
      - PGADMIN_DEFAULT_EMAIL=admin@admin.com
      - PGADMIN_DEFAULT_PASSWORD=admin
    ports:
      - "2222:80"
    volumes:
      - "pgadmin_volume:/var/lib/pgadmin"
volumes:
    postgres_volume:
    pgadmin_volume:
```

Arquivo `redis.yml`  contendo três serviços banco de dados [Redis](https://redis.io/)
```yml
version: '3.2'
services:
  redis-cache: 
    image: redis
    hostname: redis-cache
    ports: 
      - "6379:6379"
  redis-sockets:
    image: redis
    hostname: redis-sockets
    ports:
      - "6380:6379"
  redis-jobs:
    image: redis
    hostname: redis-jobs
    ports:
      - 6381:6379
```

## Passo 4: Configurando o Docker Swarm
1. Abra o terminal e execute o seguinte comando para Inicialize o cluster do docker swarm: 
`docker swarm init`

2. Com o cluster inicializado, use o seguinte comando para adicionar cada um dos serviços:
`docker stack deploy --compose-file arquivoSalvo.yml nomeDoServico`

>Comandos Úteis:
>- Listar stacks: `docker stack ls`
>- Remover stack: `docker stack rm nomeDaStack`
>- Adicionar stack: `docker stack deploy --compose-file arquivoSalvo.yml nomeDoServico`

## Passo 5: Configuração do arquivo de variáveis de ambiente:

Existem três arquivos que devem ser criados em nossa aplicação:

`/apps/api/.env`
`/apps/front/.env`
`/apps/wanderson-api/.env`

1. Após a criação dos arquivos copie o conteúdo dos arquivos `.env-exemple` para dentro do arquivo `.env` em cada um dos respectivos diretórios.

2. Depois de copiar o conteúdo de exemplo preencha os valores respectivos à configuração do banco de dados que foi inserida no arquivo do serviços do banco de dados.

> PS: Cada pasta de projeto citada acima possui o seu próprio arquivo de exemplo `.env-exemple`.

## Passo 6: Criar o token de leitura do Registry e inserir no projeto:
Esse token é necessário para fazer o download de nossos pacotes publicados no registry do Github, sem ele a aplicação falha durante a instalação dos pacotes do `package.json`.

1. Com a sua conta corporativa, acesse a página de [Developer Settings](https://github.com/settings/tokens) no Github.
2. Clique em `Generate new token`, selecione a opção `Generate new token (classic)`.
3. Escolha um nome e uma data de expiração selecionando as permissões `repo` e `write:packages` e gere o token.

Se estiver usando `yarn`, coloque seu token no arquivo `.yarnrc.yml`
Se estiver usando `npm`, coloque seu token no arquivo `.npmrc`

## Passo 7: Instale as dependencias do projeto

Abra seu terminal na raiz do projeto do projeto execute o seguinte comando:

Se estiver usando **npm**: `npm install`
Se estiver usando **yarn**: `yarn`

Aguarde a instalação.

## Passo 8: Execute a aplicação
Nossa aplicação é um monorepo e a tecnologia adotada para orquestrar as aplicações foi o `nx`.

> Leia mais em [Nx.dev](https://nx.dev/)

Executar FRONTEND e BACKEND:
`yarn nx run front:serve-with-services`

Executar FRONTEND
`yarn nx run front:serve`

Executar BACKEND
`yarn nx run api:serve`

>PS: O **Nx** possui uma extensão para a IDE Visual Studio Code que simplifica as ações dentro do monorepo, Caso use VSCode é recomendável instalar o [Nx Console](https://marketplace.visualstudio.com/items?itemName=nrwl.angular-console).
