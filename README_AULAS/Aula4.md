
### Aula4
> Funcionalidade de agendamentos

#### Sumário
- [Listagem de prestadores de serviço](#listagem-de-prestadores-de-serviço)
- [Migration e model de agendamento](#migration-e-model-de-agendamento)



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

[<- Aula anterior](Aula3.md)
