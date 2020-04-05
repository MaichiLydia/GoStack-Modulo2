
### Aula5
> Envio de notificações

#### Sumário
- [Configurando MongoDB](#configurando-mongodb)

#### Configurando MongoDB
Utilizaremos banco de dados não relacionais porque teremos dados na nossa aplicação que não serão estruturados e não terão relacionamentos e que precisarão ser performáticos

Como a gente tinha um docker para o postgres, agora criaremos um para o mongo também, utilizando essa [imagem do dockerhub](https://hub.docker.com/_/mongo):
```
docker run --name mongobarber -p 27017:27017 -d -t mongo
```
Para testar se tudo deu certo, podemos acessar `localhost:27017` pelo navegador e checar se a resposta é: `It looks like you are trying to access MongoDB over HTTP on the native driver port.`

![Navegador acessando localhost:27017 e a mensagem 'It looks like you are trying to access MongoDB over HTTP on the native driver port.'](../README_FILES/images/response/mongo.png)

E agora para a aplicação se conectar com o mongodb, utilizaremos o mongoose:
```
yarn add mongoose
```
E configuraremos o mongo também no arquivo de [index da database](../src/database/index.js) importando o mongoose e:
```
mongo() {
    this.mongoConnection = mongoose.connect(
        'mongodb://localhost:27017/gobarber',
        { useNewUrlParser: true, useFindAndModify: true }
    );
}
```
Caso de algum erro com a conexão, é possível fzer a listagem dos containers e ver os logs que nem vimos na [Aula1](Aula1.md#configurando-docker):
```
docker ps
```
ou
```
docker ps -a
```
e para ver os logs:
```
docker logs mongobarber
```
utilizando o nome ou o id do container.
[<- Aula anterior](Aula4.md)
