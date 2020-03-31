## Go Stack Bootcamp
- Treinamento imersivo nas tecnologias mais modernas de desenvolvimento web e mobile.

[Modulo 1](https://github.com/MaichiLydia/GoStack-Modulo1)

### Modulo 2

#### Requisitos

- [Vscode](https://code.visualstudio.com/Download)
- [node download](https://nodejs.org/en/download/) e [node documentação](https://nodejs.org/en/docs/)
- [yarn](https://yarnpkg.com/)
- [HTTP Métodos](https://www.w3schools.com/tags/ref_httpmethods.asp)
- [API REST](https://becode.com.br/o-que-e-api-rest-e-restful/)
- [express getting started](https://expressjs.com/en/starter/installing.html) e [express reference](https://expressjs.com/en/4x/api.html)
- [Docker get started](https://www.docker.com/get-started) e [Docker documentação](https://docs.docker.com/)
- [Postbird](https://www.electronjs.org/apps/postbird)
- ESLint, Prettier e EditorConfig

#### Índice das aulas:
- [Aula1 - Ambiente e conceitos](#aula1)
- [Aula2 - Cadastro e autenticação de usuários](#aula2)

#### Aula1
> Ambiente e conceitos

#### Sumário
- [Configurando estrutura](#configurando-estrutura)
- [Nodemon e Sucrase](#nodemon-e-sucrase)
- [Configurando debugger](#configurando-debugger)
- [Conceitos de Docker](#conceitos-de-docker)
- [Configurando docker](#configurando-docker)
- [Conceitos ORM e Sequelize](#conceitos-orm-e-sequelize)
- [Configurando Sequelize](#configurando-sequelize)

#### Configurando estrutura
Começamos esse módulo criando a pasta e inicializando um projeto com yarn, logo após instalando o express
```
yarn init -y
yarn add express
```
Depois criamos uma pasta chamada `src` na raiz do projeto e nela adicionamos os arquivos `app.js` `server.js` e `routes.js`.

- Separamos a nossa aplicação do servidor, pois ao fazermos testes automatizados a parte do servidor não será utilizada, veremos mais em frente

#### Nodemon e Sucrase
O node não suporta as novas funcionalidade do js, como a sintaxe de import e export, para conseguirmos utilizar isso nós instalamos `sucrase`:
`yarn add nodemon sucrase -D`

(e já instalamos o nodemon junto, ambos na opção de desenvolvimento)

Com isso conseguimos mudar disso, por exemplo no arquivo `routes.js`:

`const { Router } = require('express');`

para:

`import { Router } from 'express';`

E também:

`module.exports = routes;`

para:

`export default routes;`

e com isso mudaremos em todos os arquivos, `app.js` e `server.js`.

Se a gente tentar rodar a aplicação passando:
`node src/server.js`

dará erro, para que dê certo devemos utilizar:
`yarn sucrase-node src/server.js`

Para que o comando fique fácil e não necessite reinicializar após cada modificação, utilizamos o `nodemon`, adicionamos a seção de `scripts` no `package.json`:
```
  "scripts": {
    "dev": "nodemon src/server.js"
  },
```
e é necessário configurar o nodemon para que ele execute o sucrase antes, criando um arquivo `nodemon.json` na raiz do projeto e passando as informações necessárias:
```
{
    "execMap": {
        "js": "node -r sucrase/register"
    }
}
```

#### Configurando debugger
Para configurar o debugger, será necessário mudar no launch.json o request e o protocol, ficando assim:
```
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "type": "node",
            "request": "attach",
            "name": "Attach Program",
            "restart": true,
            "skipFiles": [
                "<node_internals>/**"
            ],
            "protocol": "inspector"
        }
    ]
}
```
 E também adicionar ao `scripts`:
`"dev:debug": "nodemon --inspect src/server.js"`
isso faz com que nossa configuração do nodemon pegue para o debugger também.

#### Conceitos de Docker
 Controlar serviços da aplicação - banco de dados, envio de email, etc
##### Como funciona ?
 - Criação de ambientes isolados(container) -> Para que a configuração e pastas não criem dependencias ou conflitos na nossa máquina, isolamos esse serviço num ambiente e tornamos a exclusão, atualização ou até mesmo a substituição mais simples.
 - Containers expõem portas para comunicação.

##### Principais conceitos:
- Imagem: Serviços/ferramentas disponíveis para utilização
- Container: instância de uma imagem.
- Docker Registry (Docker Hub) -> onde encontrar as imagens
- Dockerfile - Receita para criar uma imagem.

#### Configurando Docker
Para configurar o docker será necessário seguir as instruções do site: https://www.docker.com/get-started conforme o seu sistema operacional, após instalado é possível verificar se tudo deu certo com os seguintes comandos: `docker --version` e `docker help`

Utilizamos o docker com a imagem do postgres: https://hub.docker.com/_/postgres

`docker run --name dabase -e POSTGRES_PASSWORD=qualquer -p 5432:5432 -d postgres`
--name ${nome}: para o nome da sua aplicação
-e: para variaveis, podemos ver mais opções no link da imagem, exemplo: POSTGRES_PASSWORD=${senha}
-p ${Porta na sua maquina}:${Porta na imagem do docker} - redirecionamento de porta
-d ${nome da imagem no docker hub} - para passar qual imagem queremos
Ao terminar de executar o docker retorna um id único do container e podemos utilizá-lo para testar se tudo funcionou corretamente:
`docker ps`
E verificar se o id gerado está na tabela q o `docker ps` printou no console.

Para visualizarmos o nosso banco, é possível baixar um client de postgres, nesse curso utilizamos o [postbird](https://www.electronjs.org/apps/postbird)

Deixando as configuraçoes assim:
![Configurações do postbird - host: localhost - port: porta que foi escolhida na hora do comando docker run - username: postgres - password: senha que foi escolhida na hora do comando docker run ](README_FILES/images/postbird/connect.png)

E criamos um novo database já utilizando o postbird:
![Selecionar 'create database' no canto esquerdo superior, em 'select databse' e colocar como nome 'gobarber', deixar template em branco e selecionar no encoding UTF8](README_FILES/images/postbird/database_create.png)

Para garantirmos que a mesma instancia do banco de dados seja iniciada corretamente, mesmo quando reiniciarmos nossa máquina ou parar esse container com `docker stop database` podemos utilizar o id único ou o nome no comando `docker start`, por exemplo:
```
docker start database
```
Caso de algum erro é possível veriricar os logs:
```
docker logs database
```

Também é possível utilizar esse mesmo container para outras aplicações, mas o ideal é que seja isolado e especifico para cada aplicação e veremos isso em outra aula.

#### Conceitos ORM e Sequelize
##### ORM - Object Relational Mapping
- Abstração do banco de dados - mudando a comunicação com o banco
- Tabelas viram models
- A manipulação dos dados ficam sem queries SQL, no caso da nossa aplicação utilizaremos apenas código javascript para que isso aconteça.
##### Migrations
- Controle de versão para base de dados, para que a base esteja sempre atualizada entre todos os envolvidos no projeto e no ambiente de produção.
- Cada arquivo contém instruções para criação, alteração ou remoção de tabelas ou colunas e a ordenação ocorre por data.
- É possível desfazer uma migração se errarmos algo enquanto estivermos desenvolvendo a feature
- **Não** é possível editar a migration após implementada para outras pessoas e/ou ambientes.
- Cada migration deve realizar alterações em apenas uma tabela, é possível criar várias migrations para alterações maiores.

##### Seeds
- arquivos que populam a base de dados para desenvolvimento
- Muito utilizados para testes
- Executável apenas por código
- Jamais utilizado em produção
- Caso sejam dados que precisam ir para produção, a própria migration pode manipular dados das tabelas

##### Arquitetura MVC - Model, View, Controller
- Model: ele abstrai as figuras principais da aplicação, nesse tutorial utilizaremos para abstração do banco, utilizado para manipular os dandos contidos nas tabelas e não possuem responsabilidades sobre a regra de negócio da nossa aplicação.
- Controller: é o ponto de entrada das requisições da nossa aplicação, uma rota geralmente está associada diretamente com um método do controller. Podemos incluir a grande parte das regras de negócio da aplicação nos controllers(conforme a aplicação cresce podemos isolar as regras).
    - Classes
    - Sempre retorna um JSON
    - Não chama outro controller/método
    - Quando criar um controller:
      - Toda vez que tiver uma nova entidade
      - Apenas 5 métodos;
- View: é o retorno ao cliente, em aplicações que não utilizam o modelo de API REST o retorno pode ser um HTML ou txt, mas no nosso caso a view é apenas nosso JSON que será retornado ao front-end e depois manipulado pelo ReactJS ou React Native.

#### Configurando Sequelize
Para começar, criaremos algumas pastas

```
src
└───app
│   └───controllers
│   └───models
└───config
│   │   database.js
└───database
│   └───migrations

```
e depois rodamos `yarn add sequelize` e `yarn add sequelize-cli -D` para facilitar migrations e outros comandos por linha no nosso console

Após isso será necessário criar na raiz do projeto um arquivo `.sequelicerc` e colocar as configurações especificas, para ver [clique aqui](.sequelizerc)

instalar `yarn add pg pg-hstore` e adicionar as configurações em [database.js](src/config/database.js), aqui ficaram as informações para conexão no postgres e as definições necessárias, como:
- `timestamps` para adicionar `created_at` e `updated_at` nas tabelas e conseguirmos verificar datas de criação e edição.
- `underscored` e `underscoredAll` para padronizar a nomenclatura para _ e não camelCase, exemplo: userGroups ficariaw user_groups

#### Aula2
> Cadastro e autenticação de usuários

#### Sumário
- [Migration de usuário](#migration-de-usuário)
- [Model de usuário](#model-de-usuário)
- [Criando loader de models](#criando-loader-de-models)


#### Migration de usuário

Começamos com `yarn sequelize migration:create --name=create-users` que gerará um arquivo na pasta `src/database/migrations`, nesse arquivo gerado(que nesse projeto foi [esse](src/database/migrations/20200331034210-create-users.js)) escolheremos os campos necessários pra um usuário, quando terminado testaremos com o comando: `yarn sequelize db:migrate`.

E ficarará assim no postbird:

![No postbird após dar refresh conseguimos ver a tabela users criadas com todos os campos configurados no arquivo gerado no migrations](README_FILES/images/postbird/users_created.png)

Esse migrate também gerará uma tabela chamada `SequelizeMeta`, ela serve para armazenar todas as migrations que esse banco já recebeu para poder checar a necessidade de atualizações/criações/exclusões sempre que receber um migrate novo, por isso o migrate sempre sabe quando e o que atualizar.

Caso a migrate que eu fiz teve algum erro antes de eu enviar para outros desenvolvedores ou antes de ir para produção, é possível executar `yarn sequelize db:migrate:undo:all` para desfazer todas as migrations sem o `:all` para desfazer a última.

Executando isso conseguimos ver que a tabela de `users` some e o registro na tabela `SequelizeMeta` também some.

#### Model de usuário

Criamos a model de [User](src/app/models/User.js) passando todas as informações que podem ser recebidas na criação de um usuário, tirando as informações que já são populadas automaticamente como chaves primárias e datas de criação e edição.

#### Criando loader de models
O arquivo [index.js](src/database/index.js) para que realize a conexão com o banco de dados postgres e carregue todos os nossos models. O método `init` fará tudo isso:

- A variável `this.connection` no método `init` do nosso arquivo `index.js` é o parametro esperado nos models em seus próprios metodos init, então acessaremos cada model, acessando também seus métodos e passando esse valor, que é o que é feito nesse trecho de código:
```
models.map((model) => model.init(this.connection));
```
E chamamos esse arquivo no [app.js](src/app.js)

**Somente para testar** podemos no arquivo [routes.js](src/routes.js) inserir um usuário direto na rota '/' no método GET:
```
routes.get('/', async (req, res) => {
    const user = await User.create({
        name: 'Lydia Rodrigues',
        email: 'mlydiarodrigues@gmail.com',
        password_hash: '123457567',
    });
    return res.json(user);
});
```
E assim ao acessar nossa aplicação conseguimos ver o todos os campos que são gerados automaticamente quando o registro é salvo no banco com informações e também as informações que colocamos para criar o usuário:

![Aqui conseguimos ver a tela com as informações do usuário criado, com os campos que foram preenchidos antes e o adicional de id: 1, updatedAt: "2020-03-31T04:30:12.401Z", createdAt: "2020-03-31T04:30:12.401Z", provider: false](README_FILES/images/response/user_created.png)

E também coseguimos visualizar a criação no postbird:
![A tela do postbird com as mesmas informações mencionadas na imagem de cima porém na tabela de users](README_FILES/images/postbird/user_created.png)


E como estamos em modo de desenvolvimento também é retornado no log a query de inserção sem valores:
![Aqui aparece o log com a query de inserção: Executing (default): INSERT INTO "users" ("id","name","email","password_hash","created_at","updated_at") VALUES (DEFAULT,$1,$2,$3,$4,$5) RETURNING *;](README_FILES/images/postbird/sequelize_log.png)
