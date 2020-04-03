
### Aula4
> Funcionalidade de agendamentos

#### Sumário
- [Listagem de prestadores de serviço](#listagem-de-prestadores-de-serviço)



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


[<- Aula anterior](Aula3.md)
