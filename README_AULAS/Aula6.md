
### Aula5
> Envio de notificações

#### Sumário
- [Cancelamento de agendamento](#cancelamento-de-agendamento)
- [Configurando Nodemailer](#configurando-nodemailer)
- [Configurando template de e-mail](#configurando-template-de-e\-mail)
- [Configurando fila com Redis](#configurando-fila-com-redis)
- [Monitorando falhas na fila](#monitorando-falhas-na-fila)
- [Listando horários disponíveis](#listando-horários-disponíveis)
- [Campos virtuais no agendamento](#campos-virtuais-no-agendamento)

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
  "provider_id": 1,
  "provider": {
    "name": "Lydia Rodrigues",
    "email": "mlydiasilva10@gmail.com"
  },
}
```
A checagem do envio do email é feito no mailtrap:

![Captura de tela do mailtrap de email enviado, canto esquerdo tem quais emails foram enviados, com o assunto e para quem, canto direito tem todo o email, com assunto: Agendamento cancelado, destinatario: email do provedor, remetente: Equipe GoBarber <noreply@gobarber.com>, e corpo: Você tem um novo cancelamento, que são as informações que configuramos](../README_FILES/images/response/mailtrap_mailsent.png)

Esse email nunca chegará a caixa do destinatário porque utilizamos o serviço gratuíto do mailtrap, quando chegarmos na parte de produção utilizaremos um serviço que enviará de verdade para o email, mas como estamos apenas testando desenvolvimento já conseguimos testar dessa forma sem custo.

#### Configurando template de e-mail
Para que a gente mande emails mais bonitos e personalizados, utilizaremos html e css, formando nossos próprios templates, para isso utilizaremos duas bibliotecas para conseguir chamar um template engine(arquivos html que aceitam variáveis do node), nesse projeto utilizaremos o Handlebars, para saber mais sobre [acesse a documentação](https://handlebarsjs.com/api-reference/):
```
yarn add express-handlebars nodemailer-express-handlebars
```
Criaremos dentro de `src/app` a pasta `views`, dentro dela a pasta `emails`, que terá o [arquivo de template de cancelamento](../src/app/views/emails/cancellation.hbs). e também as pastas `layouts`, que terá o [arquivo de padrão de layout](../src/app/views/emails/layouts/default.hbs) e `partials` vazia.

A estrutura ficará assim:
```
src
└───app
│   └───views
│   |   └───emails
|   |   |   └───layouts
│   │   |   |   └───default.hbs
|   |   |   └───partials
│   │   |   └───cancellation.hbs

```
E após isso importamos as dependencias de handlebars e configuraremos no [arquivo de libs de Email](../src/lib/Mail.js),
Feito isso começaremos a dar cara para nosso email criando o html nos [arquivo de padrão de layout](../src/app/views/emails/layouts/default.hbs), [arquivo de template de cancelamento](../src/app/views/emails/cancellation.hbs), e criando um [arquivo de footer](../src/app/views/emails/partials/footer.hbs) na nossa pasta de `partials`.
Com o término dos templates agora teremos que passar as variáveis e chamar o template correto na [controller de Agendamento](../src/app/controllers/AppointmentController.js).

Conseguiremos testar fazendo a requisição do cancelamento de um agendamento:
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
  "canceled_at": "2020-04-06T04:02:18.644Z",
  "createdAt": "2020-04-05T16:32:30.532Z",
  "updatedAt": "2020-04-06T04:02:18.645Z",
  "user_id": 5,
  "provider_id": 1,
  "provider": {
    "name": "Lydia Rodrigues",
    "email": "mlydiasilva10@gmail.com"
  },
  "user": {
    "id": 5,
    "name": "Lydia Jorge Rodrigues",
    "email": "mlydiasilva5@gmail.com",
    "password_hash": "$2a$08$TchiFaAl.0atNKDOFxL2f.nDQE4q8QeQAohVwzq7z40J2cWRrgMWC",
    "provider": false,
    "createdAt": "2020-03-31T06:04:55.400Z",
    "updatedAt": "2020-04-03T04:33:53.584Z",
    "avatar_id": 1
  }
}
```
A checagem do envio do email é feito no mailtrap:
![Captura de tela do email enviado na página do mailtrap, do lado esquerdo temos o email enviado com assunto: Agendamento cancelado, e destinatario: To:<mlydiasilva10@gmail.com> e a esquerda temos o template que configuramos juntos com todas as variáveis atualizadas com o prestador de serviço, usuário e data.](../README_FILES/images/response/mailtrap_template.png)

```
Olá Lydia Rodrigues
Houve um cancelamento de horário, confira os detalhes abaixo:

Cliente: Lydia Jorge Rodrigues
Data/hora: dia 30 de agosto, às 12:00h

O horário está novamente disponível para novos agendamentos.


Equipe GoBarber
```

#### Configurando fila com Redis
Conseguimos diminuir o tempo de resposta de requisições que chamem serviços externos como essa de envio de email de duas formas:
A forma mais rápida seria retirar o `await` da frente do envio do email no [arquivo da controller de agendamento](../src/app/controllers/AppointmentController.js):
```
await Mail.sendMail({
    to: `${appointment.provider.name} <${appointment.provider.email}>`,
    subject: 'Agendamento cancelado',
    template: 'cancellation',
    context: {
        provider: appointment.provider.name,
        user: appointment.user.name,
        date: format(
            appointment.date,
            "'dia' dd 'de' MMMM', às' H:mm'h'",
            {
                locale: pt,
            }
        ),
    },
});
```
Isso faz com que a nossa resposta não dependa do sucesso do envio do email, porém a **desvantagem** disso é que caso ocorra um erro no envio de email nós não saberemos, pois a resposta já teria sido disparada para o cliente.

A melhor forma para que a gente diminua esse tempo de resposta sem perder o controle, podendo até fazer retentativas em casos de erro, priorizar disparos e outras funcionalidades é utilizando `filas`(background jobs/trabalhos em segundo plano), para que a gente configure isso precisaremos de um banco chave: valor, e na nossa aplicação utilizaremos [Redis](https://redis.io/documentation), que também é não relacional, mas tem só trabalha com chave: valor, diferente do mongo que tem estrutura de dados e utiliza o schema, isso faz com que o Redis seja mais rápido e performático e funciona para o controle de erros que mencionamos.
Iniciaremos criando nosso container:
```
sudo docker run --name redisbarber -p 6379:6379 -d -t redis:alpine
```
Utilizamos o `alpine` porque ele vem só com as funcionalidades essenciais do linux, ficando bem mais leve.

Caso dê algum erro com a conexão, é possível fzer a listagem dos containers e ver os logs, como vimos na [Aula1](Aula1.md#configurando-docker):
```
docker ps
```
ou esse para listar todos os containers:
```
docker ps -a
```
e para ver os logs:
```
docker logs redisbarber
```
utilizando o nome ou o id do container.

Feito isso criaremos um [arquivo de configuração para o redis](../src/config/redis.js) passando as informações de host e port para conexão.
Obs: Aqui no meu eu usei a 6380, pois já estava usando a 6379 localmente.

Após isso instalaremos o [bee-queue](https://github.com/bee-queue/bee-queue), ferramenta de fila para o nodejs, ele é muito performatico, porém não tem todas as funcionalidades que outras ferramentas de fila tem, como prioridade entre jobs, caso seja necessário utilizar essas funcionalidades também existe o [kue](https://github.com/Automattic/kue), menos performático, mas mais robusto, como não precisaremos, usaremos o bee-queue:
```
yarn add bee-queue
```
E criaremos [o arquivo para configurar a fila](../src/lib/Queue.js) na nossa pasta `lib` e também uma pasta `jobs` dentro de `src/app/` para armazenar todos os nossos jobs, já criando o [job de email de cancelamento](../src/app/jobs/CancellationMail.js), nessa parte retiraremos da [controller de agendamento](../src/app/controllers/AppointmentController.js) o envio de email e deixaremos o job responsável por isso.

[O arquivo que configura as filas](../src/lib/Queue.js), tem o método `init` para inicializar as filas e o método `add` para adicionando itens nas filas e o `processQueue` para processar as filas.

Como nos retiramos o envio da [controller de agendamento](../src/app/controllers/AppointmentController.js), agora devemos importar a fila e o job.
```
await Queue.add(CancellationMail.key, {
    appointment,
});
```

Para finalizar criaremos dentro de `src` um [arquivo queue.js](../src/queue.js), pois não executaremos a fila e a aplicação juntos, para que a fila nunca interfira na performace da nossa aplicação e que ambos sejam independentes.
Para que seja possível utilizar o sucrase nele adicionaremos uma chamada na seção de scripts do [package.json](../package.json) e adicionamos `"queue": "nodemon src/queue.js"`.
```
  "scripts": {
    "dev": "nodemon src/server.js",
    "queue": "nodemon src/queue.js",
    "dev:debug": "nodemon --inspect src/server.js",
    "lint": "yarn eslint --fix src --ext .js"
  },
```

Para testar deixaremos um
```
console.log('A fila executou');
```
no handle do [job de email de cancelamento](../src/app/jobs/CancellationMail.js), iniciamos num terminal a fila `yarn queue` e a aplicação em outro terminal `yarn dev` e fazemos novamente a requisição de cancelamento de agendamento:
```
curl --request DELETE \
  --url http://localhost:3333/appointments/7 \
  --header 'authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NSwiaWF0IjoxNTg1ODgzOTMxLCJleHAiOjE1ODY0ODg3MzF9.mYiP3Ij0lD_OUb1jeyczPHkrKIM25IEN56KVK2r5n6c'
```
A resposta dessa vez voltará a ser rápida:
```
{
  "id": 7,
  "date": "2020-08-30T15:00:00.000Z",
  "canceled_at": "2020-04-06T05:16:43.953Z",
  "createdAt": "2020-04-05T16:32:30.532Z",
  "updatedAt": "2020-04-06T05:16:43.955Z",
  "user_id": 5,
  "provider_id": 1,
  "provider": {
    "name": "Lydia Rodrigues",
    "email": "mlydiasilva5@gmail.com"
  },
  "user": {
    "id": 5,
    "name": "Lydia Jorge Rodrigues",
    "email": "mlydiasilva10@gmail.com",
    "password_hash": "$2a$08$TchiFaAl.0atNKDOFxL2f.nDQE4q8QeQAohVwzq7z40J2cWRrgMWC",
    "provider": false,
    "createdAt": "2020-03-31T06:04:55.400Z",
    "updatedAt": "2020-04-03T04:33:53.584Z",
    "avatar_id": 1
  }
}
```
E receberemos o email da mesma forma, e mesmo que ela demore algum tempinho, não tem problema, não prendemos a resposta do usuário e ainda temos controle sobre o `job` de envio de email :)

#### Monitorando falhas na fila
Para que a gente consiga captar os erros na fila, precisaremos mexer no [arquivo de configuração da fila](../src/lib/Queue.js), criaremos o método `handleFailure` e adicionamos `on('failed', this.handleFailure)` no método `processQueue`.
Para testar mudaremos o nome o `Mail.sendMail` para `Mail.senddMail` no [job de email de cancelamento](../src/app/jobs/CancellationMail.js), ao fazer a requisição de cancelamento de agendamento conseguiremos ver nos logs do terminal:
```
Queue CancellationMail: FAILED TypeError: _Mail2.default.senddMail is not a function
...
```
Por enquanto deixaremos no console.log, mas o ideal é que tenhamos esse erro num painel para visualização e melhor controle

#### Listando horários disponíveis

As regras para nossa listagem são:
- Listar por prestador
- Não listar horários passados
- Não listar horários que já tenham agendamento
- Considerar horário comercial

Começamos criando a rota no [arquivo de rotas](../src/routes.js) e criaremos a [controller de horários disponíveis](../src/app/controllers/AvailableController.js).

Nós receberemos um query param `date` porém ele terá que ser no modelo timestamp(ex: `1586151529098`), pois o frontend terá um datepicker e timepicker e enviará os dados nesse formato, para saber qual é o atual podemos utilizar o console do navegador e digitar
```
new Date().getTime()
```
O meu resultado deu esse: 1586151529098, mas como é a data atual, vai variar com a data pedida, também é possível gerar uma data especifica passando o valor no Date().
```
new Date('2020 April 09').getTime()
```
Com isso conseguimos passar o query param no formato correto: `http://localhost:3333/providers/1/available?date=1586401200000`

Com a const `appointments` conseguimos pegar todos os horários com agendamentos

Com a `schedule` colocamos todos os horários que o prestador atende, nessa parte do código deixaremos com os horários listados nessa constante, mas podemos mais pra frente fazer uma atualização que pegue o horário que o prestador selecionar, já que nem todos os estabelecimentos funcionam no mesmo horário.
E aqui retornamos todos os horários com a disponibilidade `true` ou `false` dependendo da agenda e se não é um horário que já passou.

```
const available = schedule.map((time) => {
    const [hour, minute] = time.split(':');
    const value = setSeconds(
        setMinutes(setHours(searchDate, hour), minute),
        0
    );

    return {
        time,
        value: format(value, "yyyy-MM-dd'T'HH:mm:ssxxx"),
        available:
            isAfter(value, new Date()) &&
            !appointments.find((a) => format(a.date, 'HH:mm') === time),
    };
});
```
Request:
```
curl --request GET \
  --url 'http://localhost:3333/providers/1/available?date=1586401200000' \
  --header 'authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NSwiaWF0IjoxNTg1ODgzOTMxLCJleHAiOjE1ODY0ODg3MzF9.mYiP3Ij0lD_OUb1jeyczPHkrKIM25IEN56KVK2r5n6c'
```
Response:
```
[
  {
    "time": "07:00",
    "value": "2020-04-09T07:00:00-03:00",
    "available": true
  },
  {
    "time": "08:00",
    "value": "2020-04-09T08:00:00-03:00",
    "available": true
  },
  {
    "time": "09:00",
    "value": "2020-04-09T09:00:00-03:00",
    "available": true
  },
  {
    "time": "10:00",
    "value": "2020-04-09T10:00:00-03:00",
    "available": true
  },
  {
    "time": "11:00",
    "value": "2020-04-09T11:00:00-03:00",
    "available": true
  },
  {
    "time": "12:00",
    "value": "2020-04-09T12:00:00-03:00",
    "available": false
  },
  {
    "time": "13:00",
    "value": "2020-04-09T13:00:00-03:00",
    "available": true
  },
  {
    "time": "14:00",
    "value": "2020-04-09T14:00:00-03:00",
    "available": true
  },
  {
    "time": "15:00",
    "value": "2020-04-09T15:00:00-03:00",
    "available": true
  },
  {
    "time": "16:00",
    "value": "2020-04-09T16:00:00-03:00",
    "available": true
  },
  {
    "time": "17:00",
    "value": "2020-04-09T17:00:00-03:00",
    "available": true
  },
  {
    "time": "18:00",
    "value": "2020-04-09T18:00:00-03:00",
    "available": true
  },
  {
    "time": "19:00",
    "value": "2020-04-09T19:00:00-03:00",
    "available": true
  }
]
```


Conseguimos testar também utilizando [essa collection do insomnia das datas disponíveis](../README_FILES/insomnia/GoBarber_Available.json), lembrando de seguir as configurações de ambiente do insomnia [citadas anteriormente](Aula2.md#cadastro-de-usuários)

#### Campos virtuais no agendamento

Aqui colocamos informações a mais para que o frontend consiga montar telas com agendamentos passados, para isso começamos mexendo na [model de agendamento](../src/app/models/Appointment.js) e criamos um campo `past` e um `cancellable` com tipo `VIRTUAL`, que como vimos antes, não existirá no banco, mas conseguimos retornar pelo backend. Depois disso atuaizamos a [controller de agendamento](../src/app/controllers/AppointmentController.js) para que no attributes ele também mostre o `past` e o `cancellable`

Para testar é possível utilizar [a collection do insomnia da aula 6](../README_FILES/insomnia/GoBarber_Aula6.json), na parte de Listagem de agendamentos aqui estão todas as requisições da aplicação e atualizadas com essa aula.

[<- Aula anterior](Aula5.md)
