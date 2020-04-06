
### Aula5
> Envio de notificações

#### Sumário
- [Cancelamento de agendamento](#cancelamento-de-agendamento)

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

[<- Aula anterior](Aula5.md)
