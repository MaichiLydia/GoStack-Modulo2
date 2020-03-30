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






#### Configurando estrutura
Começamos esse módulo criando a pasta e inicializando um projeto com yarn, logo após instalando o express
```
yarn init -y
yarn add express
```
Depois criamos uma pasta chamada `src` na raiz do projeto e nela adicionamos os arquivos `app.js` `server.js` e `routes.js`.

- Separamos a nossa aplicação do servidor, pois ao fazermos testes automatizados a parte do servidor não será utilizada, veremos mais em frente