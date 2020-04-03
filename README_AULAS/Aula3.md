### Aula3
> Envio de arquivos

#### Sumário
- [Configurando Multer](#configurando-multer)


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
