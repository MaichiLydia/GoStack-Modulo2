
### Aula5
> Envio de notificações

#### Sumário
- [Configurando MongoDB](#configurando-mongodb)
- [Notificando novos agendamentos](#notificando-novos-agendamentos)
- [Listando notificações do usuário](#listando-notificações-do-usuário)
- [Marcar notificações como lidas](#marcar-notificações-como-lidas)

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
ou esse para listar todos os containers:
```
docker ps -a
```
e para ver os logs:
```
docker logs mongobarber
```
utilizando o nome ou o id do container.

#### Notificando novos agendamentos
Aqui enviaremos uma notificação para o prestador de serviço toda vez que um novo agendamento ocorrer, e quem guardará essas notificações será o mongo, para isso criaremos um "schema", seriam como nossos models, mas no padrão de [schema tree](https://docs.mongodb.com/manual/applications/data-models-tree-structures/) e aqui não teremos as migrations, porém é necessário ter cuidado e estrategias para modificações.

Começaremos criando uma pasta `schemas` dentro de `src/app` e o nosso [Schema de notificação](../src/app/schemas/Notification.js), depois de adicionar todo o código no nosso "model schema", também podemos notar uma diferença entre o mongo e o postgres,para o mongo não precisaremos criar a parte de load de models, apenas importamos e utilizamos onde é necessário, como vemos na [AppointmentControler](../src/app/controllers/AppointmentController.js), aqui não fazemos nenhum relacionamento, pois o que importa é o estado atual de cada modelo, não precisaremos atualizar essas partes caso alguma coisa nos registros de appointment ou usuário sejam modificadas.

Assim como utilizamos o postbird para visualizar os dados do postgres, para o mongo utilizaremos o [mongodb compass](https://www.mongodb.com/download-center/compass), feito a conexão, já conseguimos visualizar:

![Tela do mongoDB compass com as databases padrão: admin, config, local](../README_FILES/images/mongo/first_connection.png)
Aqui percebemos que a nossa database gobarber ainda não existe, para isso precisaremos registrar um novo agendamento:

Request:
```
curl --request POST \
  --url http://localhost:3333/appointments \
  --header 'authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NSwiaWF0IjoxNTg1ODgzOTMxLCJleHAiOjE1ODY0ODg3MzF9.mYiP3Ij0lD_OUb1jeyczPHkrKIM25IEN56KVK2r5n6c' \
  --header 'content-type: application/json' \
  --data '{
	"provider_id": 1,
	"date": "2020-08-30T12:00:00-03:00"
}'
```
Response:
```
{
  "id": 7,
  "user_id": 5,
  "provider_id": 1,
  "date": "2020-08-30T15:00:00.000Z",
  "updatedAt": "2020-04-05T16:32:30.532Z",
  "createdAt": "2020-04-05T16:32:30.532Z",
  "canceled_at": null
}
```

E aí após a criação de um nvoo agendamento, o database gobarber estará no mongodb compass:

![No mongodb compass aparecere, além das databases iniciais, a database gobarber criada após o registro de um novo agendamento](../README_FILES/images/mongo/database_gobarber.png)

E o documento(registro) de notificação também:

![No mongodb compass aparece o documento de notificação basicamente em formato json com _id, read:false, content: "Novo agendamento de Lydia Jorge Rodrigues para dia 30 de agosto, às 12:00h", user: 1, createdAt: 2020-04-05T16:32:30.563+00:00, updatedAt: 2020-04-05T16:32:30.563+00:00](../README_FILES/images/mongo/notification.png)

#### Listando notificações do usuário
Começamos criando a rota no [arquivo de rotas](../src/routes.js) e depois a [controller de notificação](../src/app/controllers/NotificationController.js), aqui na parte de pegar os registros existentes, os métodos são diferentes do sequelize, para saber mais é possível checar na [documentação do mongoose](https://mongoosejs.com/docs/api.html), feito essas modificações é possível fazer a chamada:
```
curl --request GET \
  --url http://localhost:3333/notifications \
  --header 'authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MSwiaWF0IjoxNTg2MDMxMzg4LCJleHAiOjE1ODY2MzYxODh9.NGAhIq3JDaxXNgC1yBVqreu8M2KyhD2qacltgU-0O6E'
```
Response:
```
[
  {
    "read": false,
    "_id": "5e8a102fb3d27c0516858e97",
    "content": "Novo agendamento de Lydia Jorge Rodrigues para dia 24 de agosto, às 12:00h",
    "user": 1,
    "createdAt": "2020-04-05T17:06:55.553Z",
    "updatedAt": "2020-04-05T17:06:55.553Z",
    "__v": 0
  },
  {
    "read": false,
    "_id": "5e8a081e3956886e874a569e",
    "content": "Novo agendamento de Lydia Jorge Rodrigues para dia 30 de agosto, às 12:00h",
    "user": 1,
    "createdAt": "2020-04-05T16:32:30.563Z",
    "updatedAt": "2020-04-05T16:32:30.563Z",
    "__v": 0
  }
]
```
Conseguimos testar utilizando também [essa collection do insomnia de listagem de notificações](../README_FILES/insomnia/GoBarber_Notifications.json), lembrando de seguir as configurações de ambiente do insomnia [citadas anteriormente](Aula2.md#cadastro-de-usuários)

Nesse capítulo também modificamos a [controller de agendamento](../src/app/controllers/AppointmentController.js) para impedir que um usuário consiga marcar um agendamento com ele mesmo.

#### Marcar notificações como lidas
Começamos criando a rota no [arquivo de rotas](../src/routes.js) e depois criar o método update na [controller de notificação](../src/app/controllers/NotificationController.js). Aqui utilizamos o método [findByIdAndUpdate](https://mongoosejs.com/docs/api.html#model_Model.findByIdAndUpdate), onde passamos o id que queremos achar e atualizar, a chave e valor do campo que será modificado, e o `{ new: true }` para que retorne o documento já com o valor atualizado, feito isso podemos fazer a requisição:

```
curl --request PUT \
  --url http://localhost:3333/notifications/5e8a081e3956886e874a569e \
  --header 'authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MSwiaWF0IjoxNTg2MDMxMzg4LCJleHAiOjE1ODY2MzYxODh9.NGAhIq3JDaxXNgC1yBVqreu8M2KyhD2qacltgU-0O6E'
```
Response:
```
{
  "read": true,
  "_id": "5e8a081e3956886e874a569e",
  "content": "Novo agendamento de Lydia Jorge Rodrigues para dia 30 de agosto, às 12:00h",
  "user": 1,
  "createdAt": "2020-04-05T16:32:30.563Z",
  "updatedAt": "2020-04-05T17:30:44.991Z",
  "__v": 0
}
```

Conseguimos testar utilizando [essa collection do insomnia da aula5](../README_FILES/insomnia/GoBarber_Aula5.json), utilizando a parte de PUT de Notifications, lembrando de seguir as configurações de ambiente do insomnia [citadas anteriormente](Aula2.md#cadastro-de-usuários)



[<- Aula anterior](Aula4.md)
