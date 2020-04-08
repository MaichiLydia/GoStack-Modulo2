
### Aula7
> Configurações avançadas

#### Sumário
- [Tratamento de exceções](#tratamento-de-exceções)
- [Variáveis ambiente](#variáveis-ambiente)

#### Tratamento de exceções
Controlaremos e trataremos a parte de exceção, para isso utilizaremos o [sentry](https://sentry.io/), cadastraremos [nessa página](https://sentry.io/auth/login/) e após logado é necessário criar uma organização para seguir, para uso pessoal é possível utilizar o próprio nome ou qualquer outra coisa, para esse projeto utilizaremos `GoBarber`.

Na parte de plataforma selecionamos Express e Create Project:

![Aqui selecionaremos o que utilizamos no nosso projeto, tem as opções C#, Elixir, Java, Angular, Javascript, React, Express, Node, Laravel, PHP, Symfony, Django, Flask, Python, React-Native, Rails, Ruby. Selecionaremos Express](../README_FILES/images/response/sentry_platform.png)

E adicionamentos a dependencia do sentry no nosso projeto:
```
yarn add @sentry/node@5.15.4
```
Após instalado, criamos o [arquivo de configuração do sentry](../src/config/sentry.js), passando o dsn que é fornecido no site após selecionada a plataforma, e depois importamos o sentry e o arquivo de configuração no nosso arquivo de [app](../src/app.js), fazendo assim a inicialização dele no nosso constructor() e as demais configurações no restante da app.


Quando utilizamos async/await em projetos com express, temos dificuldades em pegar as exceções, portanto para facilitar isso utilizaremos `express-async-errors`:
```
yarn add express-async-errors
```
Após instalado, importaremos também no nosso arquivo de [app](../src/app.js)

Para testar digitamos propositalmente algo errado em alguma controller, aqui utilizamos a [controller de Disponibilidade](../src/app/controllers/AvailableController.js) e no método `findAll` mudamos para `finddAll`,
Após fzer a requisição:
```
curl --request GET \
  --url 'http://localhost:3333/providers/1/available?date=1586401200000' \
  --header 'authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NSwiaWF0IjoxNTg1ODgzOTMxLCJleHAiOjE1ODY0ODg3MzF9.mYiP3Ij0lD_OUb1jeyczPHkrKIM25IEN56KVK2r5n6c'
```
E checamos se a exceção ocorreu na nossa organization no sentry do navegador:
![Aqui aparece a issue criada referente a exceção que ocorreu na nossa chamada](../README_FILES/images/response/sentry_exception.png)
No sentry na parte de erros, conseguimos ver todos os detalhes, inclusive qual exceção, em que arquivo e em que linha.

Agora que já colocamos o sentry, podemos fazer com que a resposta de erro seja mais rápida e com mais informação, sem a espera infinita de uma resposta, criamos um middleware de tratamento de exceção no nosso arquivo de [app](../src/app.js) utilizando youch, ele faz uma tratativa nas analises de erro para dar uma resposta melhor para o desenvolvedor:
```
yarn add youch
```
Após feito isso fazemos uma requisição para testar:
```
curl --request GET \
  --url 'http://localhost:3333/providers/1/available?date=1586401200000' \
  --header 'authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NSwiaWF0IjoxNTg1ODgzOTMxLCJleHAiOjE1ODY0ODg3MzF9.mYiP3Ij0lD_OUb1jeyczPHkrKIM25IEN56KVK2r5n6c'
```
e a Response é bem rápida e detalhada:
```
{
  "error": {
    "message": "_Appointment2.default.finddAll is not a function",
    "name": "TypeError",
    "frames": [
      {
        "file": "src/app/controllers/AvailableController.js",
        "filePath": "/home/lydia/workspace-estudo/GoStack/modulo02/src/app/controllers/AvailableController.js",
        "method": "index",
        "line": 21,
        "column": 58,
        "context": {
          "start": 16,
          "pre": "\n        if (!date) {\n            return res.status(400).json({ error: 'Invalid date' });\n        }\n        const searchDate = Number(date);",
          "line": "        const appointments = await Appointment.finddAll({",
          "post": "            where: {\n                provider_id: req.params.providerId,\n                canceled_at: null,\n                date: {\n                    [Op.between]: ["
        },
        "isModule": false,
        "isNative": false,
        "isApp": true
      },
      {
        "file": "node_modules/express-async-errors/index.js",
        "filePath": "/home/lydia/workspace-estudo/GoStack/modulo02/node_modules/express-async-errors/index.js",
        "method": "newFn",
        "line": 16,
        "column": 20,
        "context": {
          "start": 11,
          "pre": "  return newFn;\n}\n\nfunction wrap(fn) {\n  const newFn = function newFn(...args) {",
          "line": "    const ret = fn.apply(this, args);",
          "post": "    const next = (args.length === 5 ? args[2] : last(args)) || noop;\n    if (ret && ret.catch) ret.catch(err => next(err));\n    return ret;\n  };\n  Object.defineProperty(newFn, 'length', {"
        },
        "isModule": true,
        "isNative": false,
        "isApp": false
      },
      {
        "file": "node_modules/express/lib/router/layer.js",
        "filePath": "/home/lydia/workspace-estudo/GoStack/modulo02/node_modules/express/lib/router/layer.js",
        "method": "Layer.handle [as handle_request]",
        "line": 95,
        "column": 5,
        "context": {
          "start": 90,
          "pre": "    // not a standard request handler\n    return next();\n  }\n\n  try {",
          "line": "    fn(req, res, next);",
          "post": "  } catch (err) {\n    next(err);\n  }\n};\n"
        },
        "isModule": true,
        "isNative": false,
        "isApp": false
      },
      {
        "file": "node_modules/express/lib/router/route.js",
        "filePath": "/home/lydia/workspace-estudo/GoStack/modulo02/node_modules/express/lib/router/route.js",
        "method": "next",
        "line": 137,
        "column": 13,
        "context": {
          "start": 132,
          "pre": "    }\n\n    if (err) {\n      layer.handle_error(err, req, res, next);\n    } else {",
          "line": "      layer.handle_request(req, res, next);",
          "post": "    }\n  }\n};\n\n/**"
        },
        "isModule": true,
        "isNative": false,
        "isApp": false
      },
      {
        "file": "node_modules/express/lib/router/route.js",
        "filePath": "/home/lydia/workspace-estudo/GoStack/modulo02/node_modules/express/lib/router/route.js",
        "method": "Route.dispatch",
        "line": 112,
        "column": 3,
        "context": {
          "start": 107,
          "pre": "    method = 'get';\n  }\n\n  req.route = this;\n",
          "line": "  next();",
          "post": "\n  function next(err) {\n    // signal to exit route\n    if (err && err === 'route') {\n      return done();"
        },
        "isModule": true,
        "isNative": false,
        "isApp": false
      },
      {
        "file": "node_modules/express-async-errors/index.js",
        "filePath": "/home/lydia/workspace-estudo/GoStack/modulo02/node_modules/express-async-errors/index.js",
        "method": "newFn",
        "line": 16,
        "column": 20,
        "context": {
          "start": 11,
          "pre": "  return newFn;\n}\n\nfunction wrap(fn) {\n  const newFn = function newFn(...args) {",
          "line": "    const ret = fn.apply(this, args);",
          "post": "    const next = (args.length === 5 ? args[2] : last(args)) || noop;\n    if (ret && ret.catch) ret.catch(err => next(err));\n    return ret;\n  };\n  Object.defineProperty(newFn, 'length', {"
        },
        "isModule": true,
        "isNative": false,
        "isApp": false
      },
      {
        "file": "node_modules/express/lib/router/layer.js",
        "filePath": "/home/lydia/workspace-estudo/GoStack/modulo02/node_modules/express/lib/router/layer.js",
        "method": "Layer.handle [as handle_request]",
        "line": 95,
        "column": 5,
        "context": {
          "start": 90,
          "pre": "    // not a standard request handler\n    return next();\n  }\n\n  try {",
          "line": "    fn(req, res, next);",
          "post": "  } catch (err) {\n    next(err);\n  }\n};\n"
        },
        "isModule": true,
        "isNative": false,
        "isApp": false
      },
      {
        "file": "node_modules/express/lib/router/index.js",
        "filePath": "/home/lydia/workspace-estudo/GoStack/modulo02/node_modules/express/lib/router/index.js",
        "method": null,
        "line": 281,
        "column": 22,
        "context": {
          "start": 276,
          "pre": "      if (err) {\n        return next(layerError || err);\n      }\n\n      if (route) {",
          "line": "        return layer.handle_request(req, res, next);",
          "post": "      }\n\n      trim_prefix(layer, layerError, layerPath, path);\n    });\n  }"
        },
        "isModule": true,
        "isNative": false,
        "isApp": false
      },
      {
        "file": "node_modules/express/lib/router/index.js",
        "filePath": "/home/lydia/workspace-estudo/GoStack/modulo02/node_modules/express/lib/router/index.js",
        "method": "param",
        "line": 354,
        "column": 14,
        "context": {
          "start": 349,
          "pre": "    if (err) {\n      return done(err);\n    }\n\n    if (i >= keys.length ) {",
          "line": "      return done();",
          "post": "    }\n\n    paramIndex = 0;\n    key = keys[i++];\n    name = key.name;"
        },
        "isModule": true,
        "isNative": false,
        "isApp": false
      },
      {
        "file": "node_modules/express/lib/router/index.js",
        "filePath": "/home/lydia/workspace-estudo/GoStack/modulo02/node_modules/express/lib/router/index.js",
        "method": "param",
        "line": 365,
        "column": 14,
        "context": {
          "start": 360,
          "pre": "    paramVal = req.params[name];\n    paramCallbacks = params[name];\n    paramCalled = called[name];\n\n    if (paramVal === undefined || !paramCallbacks) {",
          "line": "      return param();",
          "post": "    }\n\n    // param previously called with same value or error occurred\n    if (paramCalled && (paramCalled.match === paramVal\n      || (paramCalled.error && paramCalled.error !== 'route'))) {"
        },
        "isModule": true,
        "isNative": false,
        "isApp": false
      },
      {
        "file": "node_modules/express/lib/router/index.js",
        "filePath": "/home/lydia/workspace-estudo/GoStack/modulo02/node_modules/express/lib/router/index.js",
        "method": "Function.process_params",
        "line": 410,
        "column": 3,
        "context": {
          "start": 405,
          "pre": "    } catch (e) {\n      paramCallback(e);\n    }\n  }\n",
          "line": "  param();",
          "post": "};\n\n/**\n * Use the given middleware function, with optional path, defaulting to \"/\".\n *"
        },
        "isModule": true,
        "isNative": false,
        "isApp": false
      },
      {
        "file": "node_modules/express/lib/router/index.js",
        "filePath": "/home/lydia/workspace-estudo/GoStack/modulo02/node_modules/express/lib/router/index.js",
        "method": "next",
        "line": 275,
        "column": 10,
        "context": {
          "start": 270,
          "pre": "      ? mergeParams(layer.params, parentParams)\n      : layer.params;\n    var layerPath = layer.path;\n\n    // this should be done for the layer",
          "line": "    self.process_params(layer, paramcalled, req, res, function (err) {",
          "post": "      if (err) {\n        return next(layerError || err);\n      }\n\n      if (route) {"
        },
        "isModule": true,
        "isNative": false,
        "isApp": false
      },
      {
        "file": "src/app/middlewares/auth.js",
        "filePath": "/home/lydia/workspace-estudo/GoStack/modulo02/src/app/middlewares/auth.js",
        "method": "exports.default",
        "line": 15,
        "column": 16,
        "context": {
          "start": 10,
          "pre": "\n    const [, token] = authHeader.split(' ');\n    try {\n        const decoded = await promisify(jwt.verify)(token, authConfig.secret);\n        req.userId = decoded.id;",
          "line": "        return next();",
          "post": "    } catch (err) {\n        return res.status(401).json({ error: 'Token invalid' });\n    }\n};\n"
        },
        "isModule": false,
        "isNative": false,
        "isApp": true
      },
      {
        "file": "internal/process/task_queues.js",
        "filePath": "internal/process/task_queues.js",
        "method": "processTicksAndRejections",
        "line": 89,
        "column": 5,
        "context": {},
        "isModule": false,
        "isNative": true,
        "isApp": false
      }
    ]
  }
}
```

Aqui vimos como utilizar o [sentry](https://sentry.io) para monitorar os erros da nossa aplicação. Há muitas outras funcionalidades e integrações além dessas que utilizamos, [possível ler mais sobre na documentação do sentry](https://docs.sentry.io/), fazendo com que seja mais fácil ter métricas sobre o erro e como tratá-lo/corrigi-lo.


#### Variáveis ambiente
Aqui configuramos as variáveis de ambiente, que são variáveis que podem ter valores diferentes entre os ambientes de desenvolvimento, homologação e produção, para isso criamos o [arquivo de variáveis](../.env) e o adicionamos ao [.gitignore](../.gitignore) por conter informações confidenciais.

Após adicionar as variavéis de ambiente utilizaremos dotenv:
```
yarn add dotenv
```
E importamos no [arquivo da app](../src/app.js) e no [arquivo da queue](../src/queue.js), pois executam separadamente, após isso substituimos todos os valores que estavam nos arquivos pelas variaveis de ambiente, e agora todos os dados confidenciais ficam num arquivo separado e não vão para o nosso repositório.

Podemos criar um [env exemplo](../.env-example) que tenha as variaveis com os nomes, porém sem o valor, para que caso um novo desenvolvedor pegue a aplicação, saiba o que é necessário preencher, mas sem os acessos confidenciais

[<- Aula anterior](Aula6.md)
