### Aula3
> Envio de arquivos

#### Sumário
- [Configurando Multer](#configurando-multer)
- [Avatar de usuário](#avatar-de-usuário)


#### Configurando Multer

Utilizaremos o upload de arquivos para poder criar as fotos do avatar de cada usuário, para isso criamos a funcionalidade disso.

Para isso utilizamos a biblioteca multer(multipart formdata): `yarn add multer`
E também criaremos algumas pastar e o [arquivo de configuração do multer](../src/config/multer.js) onde ficará toda a configuração da parte de upload:
```
tmp
└───uploads
src
└───config
│   └───multer.js
```
Ao utilizar o multer, escolheremos a opção de guardar os arquivos dentro de uma pasta na nossa aplicação, existem outras formas de guardar como utilizando um cdn(content delivery network), porém não faremos isso nesse projeto agora.
Nesse arquivo também utilizamos o `crypto` para fazer com que o nome da imagem seja único e não ocorra substituições caso alguem tente salvar uma imagem com o mesmo nome:
```
filename: (req, file, cb) => {
    crypto.randomBytes(16, (err, res) => {
        if (err) return cb(err);

        return cb(
            null,
            res.toString('hex') + extname(file.originalname)
        );
    });
}
```

Para testarmos, mudaremos o [arquivo de rotas](../src/routes.js), porém arrumaremos isso na próxima aula e utilizaremos [essa collection do insomnia de envio de arquivo](../README_FILES/insomnia/GoBarber_UploadFile.json), lembrando de seguir as configurações de ambiente do insomnia [citadas anteriormente](Aula2.md#cadastro-de-usuários)

Após o envio da requisição, se tudo deu certo, conseguiremos ver a imagem na [nossa pasta de imagens](../tmp/uploads/0a44751cd54b7ee7b09b08f8b441bfd2.jpg)

Na próxima parte da aula arrumaremos o recurso corretamente e ajustaremos [arquivo de rotas](../src/routes.js)

#### Avatar de usuário
Para fazer que o id das imagens fiquem guardados numa tabela e a gente consiga associar a cada usuários criamos uma [controller de arquivos](../src/app/controllers/FileController.js) e também precisaremos criar uma tabela no banco, para isso utilizamos:
```
yarn sequelize migration:create --name=create-files
```
E modificamos o [arquivo gerado](../src/database/migrations/20200403033200-create-files.js) e rodamos:
```
yarn sequelize db:migrate
```
Para poder criar a tabela no postgres, depois disso criamos a [model](../src/app/models/File.js), depois de feita todas as modificações nesse arquivo, atualizamos a [index de database](../src/database/index.js), feito toda essa configuração, é possível atualizar a [controller de arquivos](../src/app/controllers/FileController.js) com o Model atualizado.

Feito isso o nosso arquivo é salvo no banco de dados:
![Aqui podemos ver o registro da imagem registrada no banco, com os campos id=1, name=lydia2.png, path=85a95fb0d236a0dbdeec40755a27e22b.jpg, e respectivas datas de criação e edição](../README_FILES/images/postbird/upload_file.png)

Porém ainda não foi feita a associação entre o usuário e a imagem, para isso precisaremos criar uma nova migration para adicionar uma nova coluna na tabela de users:
```
yarn sequelize migration:create --name=add-avatar-filed-to-users
```
E modificamos o [arquivo gerado](../src/database/migrations/20200403035439-add-avatar-filed-to-users.js), depois é só rodar:
```
yarn sequelize db:migrate
```
E logo conseguimos ver a tabela com a coluna extra:
![Imagem da tabela users com a nova coluna avatar_id e com um registro atualizado de avatar_id=1, fazendo referencia ao id da tabela files](../README_FILES/images/postbird/users_avatar_id.png)
Caso a gente tente mudar o avatar_id pra um número que não seja um id da tabela files, dará erro, pois é o efeito da chave estrangeira que fizemos associação.

Agora para que a atualização dê certo, precisaremos atualizar a model para que ela aceite o campo `avatar_id`, porém faremos de uma forma diferente: associations, para saber mais é possível ler na [documentação](https://sequelize.org/master/manual/assocs.html)
```
static associate(models) {
    this.belongsTo(models.File, { foreignKey: 'avatar_id' });
}
```
E mudamos o arquivo de [index do database](../src/database/index.js), adicionando mais um map:
```
models
    .map((model) => model.init(this.connection))
    .map((model) =>
            model.associate && model.associate(this.connection.models)
    );
```
E agora ao fzer o update:
```
curl --request PUT \
  --url http://localhost:3333/users \
  --header 'authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NSwiaWF0IjoxNTg1NjM1MDU0LCJleHAiOjE1ODYyMzk4NTR9.ljR-qC3R6AjOR2CwEIZppfJKzGgByHr0ARlAHk8MViw' \
  --header 'content-type: application/json' \
  --data '{
	"avatar_id": 1
}'
```
Consequimos atualizar o valor no banco:
![Imagem do registro com id 5 com o valor do avatar_id atualizado para 1](../README_FILES/images/postbird/update_avatar_id.png)

[<- Aula anterior](Aula2.md) | [Próxima aula ->](Aula4.md)
