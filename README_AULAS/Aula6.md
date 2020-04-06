
### Aula5
> Envio de notificações

#### Sumário
- [Cancelamento de agendamento](#cancelamento-de-agendamento)
- [Configurando Nodemailer](#configurando-nodemailer)

#### Cancelamento de agendamento

Nessa parte faremos o cancelamento de um agendamento, porém o cliente só poderá cancelar caso esteja com no mínimo 2h de antecedência, começamos criando a rota de delete no [arquivo de rotas](../src/routes.js) e depois criando esse método na [controller de agendamento](../src/app/controllers/AppointmentController.js), aqui fazemos as verificações se o appointment existe, se não é um usuário diferente do dono que está tentando cancelar e também validamos a regra das 2h de antecedência.
Fica assim a requisição:
```
curl --request DELETE \
  --url http://localhost:3333/appointments/7 \
  --header 'authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NSwiaWF0IjoxNTg1ODgzOTMxLCJleHAiOjE1ODY0ODg3MzF9.mYiP3Ij0lD_OUb1jeyczPHkrKIM25IEN56KVK2r5n6c'
```
Response:
```
{
  "id": 7,
  "date": "2020-08-30T15:00:00.000Z",
  "canceled_at": "2020-04-06T01:32:01.445Z",
  "createdAt": "2020-04-05T16:32:30.532Z",
  "updatedAt": "2020-04-06T01:32:01.445Z",
  "user_id": 5,
  "provider_id": 1
}
```
Aqui já aparece a data de `canceled_at`.

Conseguimos testar também utilizando [essa collection do insomnia do cancelamento de agendamento](../README_FILES/insomnia/GoBarber_Cancellation.json), que também tem as chamadas com erro, lembrando de seguir as configurações de ambiente do insomnia [citadas anteriormente](Aula2.md#cadastro-de-usuários), a collection completa dessa aula ficará no final dessa página quando passarmos por todas seções do sumário, caso queiram ver todas as requisições atualizadas da Aula5 é só checar [essa collection](../README_FILES/insomnia/GoBarber_Aula5.json)

#### Configurando Nodemailer
Para notificarmos por email o prestador de serviço sobre o cancelamento utilizaremos o nodemailer:
```
yarn add nodemailer
```
E criamos o [arquivo de configuraçãod e email](../src/config/mail.js).
O serviço que utilizaremos para poder preencher host e port e finalmente fazer o envio sera o [mailtrap](http://mailtrap.io/), porém ele só funcionara em ambiente de desenvolvimento, acessamos o [site do mailtrap](https://mailtrap.io/pricing) e escolhemos a opção free:

![Captura de tela na página na parte de preços do mailtrap, a primeira opção é a free, e é a qual utilizaremos, porém existem as outras opções que são individual, team, business, premium e enterprise](../README_FILES/images/response/mailtrap.png)

E em seguida efetuamos o cadastro para começar a utilizá-lo.

Depois de logado, criaremos nossa própria caixa na [parte de inboxes do mailtrap](https://mailtrap.io/inboxes).

![Captura de tela na página de inboxes, aqui aparece uma inbox já criada, porém apagaremos](../README_FILES/images/response/mailtrap_inboxes.png)


Nessa parte de inboxes, já tera uma inbox criada chamada `Demo Inbox` porém apagaremos essa e criaremos a nossa digitando o nome `GoBarber` na parte de Inbox Name e clicando em `Create inbox`, depois disso clicamos na que criamos e cairemos na pagina com as informações de host, port, username e password necessários, atualizei [o arquivo de configuração de email](../src/config/mail.js) com as minhas informações, mas é necessário que vocês utilizem a de vocês, pois **caso vocês configurem com as minhas informações, não aparecerá o email para vocês no mailtrap.**

Depois disso criaremos uma pasta dentro de `src` chamada `lib`, onde colocaremos arquivos de configuração adicional a nossa aplicação, nesse caso criaremos [um arquivo para o email](../src/lib/Mail.js)

Feito essas modificações, podemos criar essa parte de envio de email [na controller de agendamento](../src/app/controllers/AppointmentController.js) e depois testamos se tudo deu certo fazendo a requisição de um cancelamento:
```
curl --request DELETE \
  --url http://localhost:3333/appointments/7 \
  --header 'authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NSwiaWF0IjoxNTg1ODgzOTMxLCJleHAiOjE1ODY0ODg3MzF9.mYiP3Ij0lD_OUb1jeyczPHkrKIM25IEN56KVK2r5n6c'
```
A response levará um tempo a mais, pois estamos utilizando um serviço externo, mas logo mudaremos para que seja tão rápido como antes:
```
{
  "id": 7,
  "date": "2020-08-30T15:00:00.000Z",
  "canceled_at": "2020-04-06T01:32:01.445Z",
  "createdAt": "2020-04-05T16:32:30.532Z",
  "updatedAt": "2020-04-06T01:32:01.445Z",
  "user_id": 5,
  "provider_id": 1
}
```
A checagem do envio do email é feito no mailtrap:

![Captura de tela do mailtrap de email enviado, canto esquerdo tem quais emails foram enviados, com o assunto e para quem, canto direito tem todo o email, com assunto: Agendamento cancelado, destinatario: email do provedor, remetente: Equipe GoBarber <noreply@gobarber.com>, e corpo: Você tem um novo cancelamento, que são as informações que configuramos](../README_FILES/images/response/mailtrap_mailsent.png)

Esse email nunca chegará a caixa do destinatário porque utilizamos o serviço gratuíto do mailtrap, quando chegarmos na parte de produção utilizaremos um serviço que enviará de verdade para o email, mas como estamos apenas testando desenvolvimento já conseguimos testar dessa forma sem custo.

[<- Aula anterior](Aula5.md)
