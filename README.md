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


#### Sumário
[Configurando estrutura](#configurando-estrutura)
[Nodemon e Sucrase](#nodemon-e-sucrase)
[Configurando debugger](#configurando-debugger)






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