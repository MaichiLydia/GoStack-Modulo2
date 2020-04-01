## Go Stack Bootcamp
- Treinamento imersivo nas tecnologias mais modernas de desenvolvimento web e mobile.

[Modulo 1](https://github.com/MaichiLydia/GoStack-Modulo1)

### Modulo 2

#### Requisitos

Instação
- [Vscode](https://code.visualstudio.com/Download)
- [node download](https://nodejs.org/en/download/) e [node documentação](https://nodejs.org/en/docs/)
- [yarn](https://yarnpkg.com/)
- [express getting started](https://expressjs.com/en/starter/installing.html) e [express reference](https://expressjs.com/en/4x/api.html)
- [Docker get started](https://www.docker.com/get-started) e [Docker documentação](https://docs.docker.com/)
- [Postbird](https://www.electronjs.org/apps/postbird)

- Conceituais
    - [HTTP Métodos](https://www.w3schools.com/tags/ref_httpmethods.asp)
    - [API REST](https://becode.com.br/o-que-e-api-rest-e-restful/)
    - [JWT](https://jwt.io/introduction/)

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
- [Cadastro de usuários](#cadastro-de-usuários)
- [Gerando hash da senha](#gerando-hash-da-senha)
- [Conceitos de JWT](#conceitos-jwt)
- [Autenticação JWT](#autenticação-jwt)
- [Middleware de autenticação](#middleware-de-autenticação)
- [Update de usuário](#update-de-usuário)



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

#### Cadastro de usuários
Para efetuarmos um cadastro recebendo de fato as informações de uma chamada com metodo POST criamos o arquivo [UserController](src/app/controllers/UserController.js) e nele fazemos a inserção:
```
const { id, name, email, provider } = await User.create(req.body);
```
Pegando apenas as informações desejadas para mostrar ao usuário:
```
return res.json({
    id,
    name,
    email,
    provider,
});
```
E como colocamos o email como único, devemos checar se já existe algum registro com essa informação e retornar o erro para o usuário, para isso fizemos essa checagem antes da inserção:
```
const userExists = await User.findOne({
    where: { email: req.body.email },
});
if (userExists) {
    return res.status(400).json({ error: 'User already exists.' });
}
```

Para testarmos conseguimos utilizar a [essa collection do insomnia](README_FILES/insomnia/GoBarber_Users.json),
antes de fazer a chamada configuramos a `base_url` em No Environment > Manage Environment > Base Environment:
```
{
  "base_url": "http://localhost:3333"
}
```
Caso não consiga importar a collection também é possível utilizar o cUrl

Um exemplo de sucesso é:

**Request**:
```
curl --request POST \
  --url http://localhost:3333/users \
  --header 'content-type: application/json' \
  --data '{
	"name": "Lydia Rodrigues da Silva",
	"email": "mlydiarodrigues3@gmail.com",
	"password_hash":"21321321321"
}'
```
**Response**:
```
{
  "id": 3,
  "name": "Lydia Rodrigues da Silva",
  "email": "mlydiarodrigues3@gmail.com",
  "provider": false
}
```
E um de erro:

**Request**:
```
curl --request POST \
  --url http://localhost:3333/users \
  --header 'content-type: application/json' \
  --data '{
	"name": "Lydia Rodrigues da Silva",
	"email": "mlydiarodrigues@gmail.com",
	"password_hash":"21321321321"
}'
```
**Response**:
```
{
  "error": "User already exists."
}
```
#### Gerando hash da senha
Para gerar o hash da senha utilizaremos a extensão bcryptjs
```
yarn add bcryptjs
```
E depois devemos passar a receber o password, para isso atualizamos no [model](src/models/User.js):
```
password: Sequelize.VIRTUAL,
```
Utilizamos o `VIRTUAL` para que esse campo não exista no banco, somente do lado do código.

E utilizaremos um `hook`, funcionalidade do sequelize, que é um trecho de código que é executado de forma automatica depois de alguma ação que acontece no nosso model, como um gatilho, nesse caso o gatilho será a inserção de um novo usuário, porém faremos algo antes disso acontecer, por isso o `beforeSave`.

Esse hook fara a criptografia da senha do usuário, fazendo com que a gente não salve a senha de fato e torne a aplicação mais segura:
```
this.addHook('beforeSave', async (user) => {
    if (user.password) {
        user.password_hash = await bcrypt.hash(user.password, 8);
    }
});
```
E agora para criar um usuario a requisição fica assim:
```
curl --request POST \
  --url http://localhost:3333/users \
  --header 'content-type: application/json' \
  --data '{
	"name": "Lydia Rodrigues da Silva",
	"email": "mlydiarodrigues4@gmail.com",
	"password":"21321321321"
}'
```
Response continua o mesmo:
```
{
  "id": 4,
  "name": "Lydia Rodrigues da Silva",
  "email": "mlydiarodrigues4@gmail.com",
  "provider": false
}
```
E no postbird conseguimos ver a senha criptografada:
![Na imagem é possível visualizar as informações do usuário inserido e no campo password_hash a senha criptografada, que fica assim: $2a$08$0m6tDCLgrCpk7kyGWE4OGexOZ9m3iKd1hdAhW/OzOqxk81Sn7KKwm](README_FILES/images/postbird/password_hashed.png)
#### Conceitos JWT
**JWT** - **J**son **W**eb **T**oken - token no formato de json
Utilizamos em autenticação, pois ao passarmos o usuario e senha corretos, conseguimos gerar um token com informações criptografadas e uteis para verificação de acessos e outras utilidades.
![Imagem explicando cada parte de um token jwt, ele tem as informações de header, com o tipo de token e algoritmo, payload com dados não sensiveis como email ou id, e a assinatura](https://miro.medium.com/max/1400/1*0SEbHdFcVpaejejGA-1DDw.png)

[Imagem retirada desse post](https://codeburst.io/jwt-to-authenticate-servers-apis-c6e179aa8c4e)

#### Autenticação JWT
Criamos um [SessionController](src/app/controllers/SessionController.js), não faremos na UserController porque por mais que uma sessão seja relacionada com o usuário, ela tem caracteristicas diferentes e é complexa, podendo então tratar como uma entidade diferente de usuário.

Para gerar o token, utilizamos a extensão jsonerbtoken:
```
yarn add jsonwebtoken
```

E no nosso arquivo [SessionController](src/app/controllers/SessionController.js) após fazer todas as verificações de usuário necessárias, finalmente criamos o token:
```
token: jwt.sign({ id }, '3a3d6bc75757af03fd7aa45d90a8aab4', { expiresIn: '7d' })
```
Passando:
- primeiro parametro o payload, que é a informação sensível que poderemos reutilizar em outras chamadas.
- segundo é um texto seguro que apenas nós teremos acesso, é possível gerar um token no site https://www.md5online.org/ e passando alguma frase segura que dificilmente alguém saberia como por exemplo: gobarbergostackbackend
- terceiro são configurações adicionais como data de expiração

Para que essas informações fiquem mais seguras e concetradas em apenas um local, criamos o arquivo de [config da autenticação](src/config/auth.js) e exportamos na nossa [SessionController](src/app/controllers/SessionController.js)


Para testarmos conseguimos utilizar a [essa collection do insomnia](README_FILES/insomnia/GoBarber_Session.json), lembrando de seguir as configurações de ambiente do insomnia [citadas anteriormente](#cadastro-de-usuários)

Caso não consiga importar a collection também é possível utilizar o cUrl

Um exemplo de sucesso é:

**Request**:
```
curl --request POST \
  --url http://localhost:3333/sessions \
  --header 'content-type: application/json' \
  --data '{
	"email": "mlydiarodrigues7@gmail.com",
	"password":"123456"
}'
```
**Response**:
```
{
  "user": {
    "id": 5,
    "email": "mlydiarodrigues7@gmail.com"
  },
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NSwiaWF0IjoxNTg1NjM1MDU0LCJleHAiOjE1ODYyMzk4NTR9.ljR-qC3R6AjOR2CwEIZppfJKzGgByHr0ARlAHk8MViw"
}
```
Um de erro com email errado:

**Request**:
```
curl --request POST \
  --url http://localhost:3333/sessions \
  --header 'content-type: application/json' \
  --data '{
	"email": "emailestranho@gmail.com",
	"password":"123456"
}'
```
**Response**:
```
{
  "error": "User not found"
}
```
Um de erro com senha incorreta:

**Request**:
```
{
	"email": "mlydiarodrigues7@gmail.com",
	"password":"12345622"
}
```
**Response**:
```
{
  "error": "User not allowed, please verify email/password"
}
```
#### Middleware de autenticação
Na seção anterior conseguimos gerar o token retornado quando o usuário loga, agora a gente precisa fazer com que esse token seja solicitado nas rotas necessárias, passaremos essa informação nos Headers dessas chamadas com o par chave:valor `Authorization: Bearer ${token gerado}`(no insomnia também conseguimos colocar o token sem a palavra `Bearer` indo em Auth > Bearer Token > e adicionando o valor no campo Token), mas antes criamos um [middleware de autenticação](src/app/middlewares/auth.js):

- Aqui utilizamos promisify, que pega uma função de callback e transforma em uma promise de async await, fizemos isso pois o jwt verify utiliza função de callback, porém esse projeto tem como padronização utilizar async / await.

Para utilizar middleware de autenticação, importamos nas [rotas](src/routes.js) e conseguimos utilizar da segunte forma:
```
routes.post('/users', UserController.store);
routes.post('/sessions', SessionController.store);

routes.use(authMiddleware);

routes.put('/users', UserController.update);
```
Quando a gente utiliza o routes.use ele só passa a funcionar nas rotas abaixo dele, então essa autenticação só será chamada nas rotas necessárias, nesse caso, a rota de atualização.

Para testarmos conseguimos utilizar a [essa collection do insomnia](README_FILES/insomnia/GoBarber_Update.json), lembrando de seguir as configurações de ambiente do insomnia [citadas anteriormente](#cadastro-de-usuários)

Colocarei o curl dessas chamadas na próxima parte pois também trabalharemos nessa chamada.

#### Update de usuário

Nesse módulo atualizamos o [controller de users na parte de update](src/app/controller/UserController.js) para que os dados corretos sejam passados na chamada e altere a partir do id do token.

Para testarmos conseguimos utilizar a [essa collection do insomnia](README_FILES/insomnia/GoBarber_UpdateData.json), lembrando de seguir as configurações de ambiente do insomnia [citadas anteriormente](#cadastro-de-usuários)

Um exemplo de sucesso é:

**Request**:
```
curl --request PUT \
  --url http://localhost:3333/users \
  --header 'authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NSwiaWF0IjoxNTg1NjM1MDU0LCJleHAiOjE1ODYyMzk4NTR9.ljR-qC3R6AjOR2CwEIZppfJKzGgByHr0ARlAHk8MViw' \
  --header 'content-type: application/json' \
  --data '{
	"name": "Lydia Jorge Rodrigues",
	"email": "mlydiarodrigues7@gmail.com",
	"oldPassword": "123123",
	"password": "111111"
}'
```
**Response**:
```
{
  "id": 5,
  "name": "Lydia Jorge Rodrigues",
  "email": "mlydiarodrigues7@gmail.com",
  "provider": false
}
```
Um de erro com email já usado:

**Request**:
```
{
	"name": "Lydia Jorge Rodrigues",
	"email": "emailjausado@gmail.com",
	"oldPassword": "111111",
	"password": "123123"
}
```
**Response**:
```
{
  "error": "Email already used"
}
```
Um de erro com senha incorreta:

**Request**:
```
{
	"name": "Lydia Jorge Rodrigues",
	"email": "mlydiarodrigues7@gmail.com",
	"oldPassword": "senhaErrada",
	"password": "111111"
}
```
**Response**:
```
{
  "error": "Password does not match"
}
```
