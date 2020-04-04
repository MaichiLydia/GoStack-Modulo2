
### Aula4
> Funcionalidade de agendamentos

#### Sumário
- [Listagem de prestadores de serviço](#listagem-de-prestadores-de-serviço)
- [Migration e model de agendamento](#migration-e-model-de-agendamento)
- [Agendamento de serviço](#agendamento-de-serviço)



#### Listagem de prestadores de serviço

Começamos criando o recurso no arquivo de [rotas](../src/routes.js) e criaremos um controller separado para os prestadores de serviço, porque, mesmo sendo usuário, estamos tratando de uma entidade um pouco diferente de um usuário comum.
Na [Controller de prestador de serviço](../src/app/controllers/ProviderController.js) fizemos as modificações, conseguimos escolher somente os campos desejados utilizando attributes e também puxar as informações do avatar, passando um include e adicionando um alias na parte de associação na [Model de usuário](../src/app/models/User.js):
```
static associate(models) {
    this.belongsTo(models.File, { foreignKey: 'avatar_id', as: 'avatar' });
}
```
e passamos isso para [Controller de prestador de serviço](../src/app/controllers/ProviderController.js):
```
const providers = await User.findAll({
    where: {
        provider: true,
    },
    attributes: ['id', 'name', 'email', 'avatar_id'],
    include: [
        {
            model: File,
            as: 'avatar',
            attributes: ['name', 'path'],
        },
    ],
});
```
Porém o ideal é que a gente retorne para nosso frontend a url da imagem já formada, para isso vamos utilizar a [model de arquivos](../src/app/models/File.js) e criar um campo virtual(que não existira no nosso banco) e que já retorne essa url montada:
```
url: {
    type: Sequelize.VIRTUAL,
    get() {
        return `http://localhost:3333/files/${this.path}`;
    },
},
```
Feito isso adicionamos esse campo no attributes do [Controller de prestador de serviço](../src/app/controllers/ProviderController.js):
```
attributes: ['name', 'path', 'url'],

```
E para que a gente consiga acessar a imagem sem ter q passar token, vamos no [arquivo app](../src/app.js) e na parte de middlewares adicionamos:
```
middlewares() {
    this.server.use(express.json());
    this.server.use(
        '/files',
        express.static(path.resolve(__dirname, '..', 'tmp', 'uploads'))
    );
}
```
Conseguimos testar utilizando [essa collection do insomnia de listagem de prestadores de serviço](../README_FILES/insomnia/GoBarber_Providers.json), lembrando de seguir as configurações de ambiente do insomnia [citadas anteriormente](Aula2.md#cadastro-de-usuários) ou também utilizando [cUrl](https://curl.haxx.se/docs/manpage.html).
Requisição:
```
curl --request GET \
  --url http://localhost:3333/providers \
  --header 'authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NSwiaWF0IjoxNTg1ODgzOTMxLCJleHAiOjE1ODY0ODg3MzF9.mYiP3Ij0lD_OUb1jeyczPHkrKIM25IEN56KVK2r5n6c'
```

#### Migration e model de agendamento

Para conseguirmos fazer os agendamentos, será necessário criar uma tabela para isso:
```
yarn sequelize migration:create --name=create-appointments
```
E configurar quais tabelas queremos [nesse arquivo](../src/database/migrations/20200403215522-create-appointments.js), aqui fazemos dois relacionamentos com tabela user, no `user_id` e `provider_id`.
Depois disso, migrate:
```
yarn sequelize db:migrate
```
Depois criamos a [model de agendamento](../src/app/models/Appointment.js), aqui também criamos uma associação, para saber mais é possível ler na [documentação](https://sequelize.org/master/manual/assocs.html), nesse caso como temos duas associações com a mesma tabela/model, somos obrigatos a criar um alias para cada um para que o sequelize não confunda com qual associar cada um.

Atualizamos também a [index do database](../src/database/index.js) com o novo modelo de agendamento

#### Agendamento de serviço

Para que a gente consiga fazer a requisição de um agendamento de serviço precisaremos criar a [controller de agendamento](../src/app/controllers/AppointmentController.js) e já adicionar nas [rotas](../src/routes.js),
na controller faremos todas as validações de campo e checagem sobre a existência do prestador de serviço.
Antes de fazer a requisição, verificamos se o usuário que utilizaremos para fazer o agendamento não está com  `provider: true`, caso esteja mudamos no próprio postbird para `false`.

Na hora de fazer a requisição:
```
curl --request POST \
  --url http://localhost:3333/appointments \
  --header 'authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NSwiaWF0IjoxNTg1ODgzOTMxLCJleHAiOjE1ODY0ODg3MzF9.mYiP3Ij0lD_OUb1jeyczPHkrKIM25IEN56KVK2r5n6c' \
  --header 'content-type: application/json' \
  --data '{
	"provider_id": 1,
	"date": "2020-08-29T12:00:00-03:00"
}'
```
Passamos a hora já com o timezone, aqui no Brasil utilizamos a UTC -3, por isso o valor do campo fica com -03:00 no final e a nossa response já retorna com o valor correto, porque o sequeliza já lida com essa parte de fuso horário:
```
{
  "id": 1,
  "user_id": 5,
  "provider_id": 1,
  "date": "2020-08-29T15:00:00.000Z",
  "updatedAt": "2020-04-04T18:31:56.928Z",
  "createdAt": "2020-04-04T18:31:56.928Z",
  "canceled_at": null
}
```
Conseguimos testar também utilizando [essa collection do insomnia de agendamento de serviço](../README_FILES/insomnia/GoBarber_Appointment.json), lembrando de seguir as configurações de ambiente do insomnia [citadas anteriormente](Aula2.md#cadastro-de-usuários)

E conseguimos ver o registro no postbird também:
![Nessa imagem tem o resgistro de agendamento no postbird com os seguintes valores id: 1, date: 2020-08-29 15:00:00+00, user_id:5, provider_id: 1, canceled_at: NULL, created_at: 2020-04-04 18:31:56.928+00, update_at: 2020-04-04 18:31:56.928+00](../README_FILES/images/postbird/create_appointment.png)

[<- Aula anterior](Aula3.md)
