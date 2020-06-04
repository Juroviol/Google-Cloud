# Google Cloud

- [Deploy de aplicação Angular no Google App Engine](#deploy-de-aplicação-angular-no-google-app-engine)
  - [Pré-requisitos](#pré-requisitos)
- [Google App Engine com Dockerfile](#google-app-engine-com-dockerfile)


## Deploy de aplicação Angular no Google App Engine

### Pré-requisitos

* Google Cloud SDK
* Aplicação construída

### Procedimento
 
Após realizar o build da aplicação Angular o qual irá gerar um diretório de saída com todos os fontes transpilados de Typescript para Javascript, vamos criar um arquivo chamado app.yaml o qual irá descrever para o Google Cloud SDK as configurações de implantação.

Criando o arquivo `app.yaml`: 

```
runtime: python27
api_version: 1
threadsafe: true
service: front
handlers:

  # Initial route that will serve up index.html, main entry point to your app
  - url: /
    secure: always
    static_files: dist/index.html
    upload: dist/.*

  # Routing for typedoc, assets and favicon.ico to serve directly
  - url: /((?:assets|docs)/.*|favicon\.ico|.*\.(?:png|svg|ttf|woff2|woff|eot))
    secure: always
    redirect_http_response_code: 301
    static_files: dist/\1
    upload: dist/.*

  # Routing for any js files
  - url: /(.*\.js)
    secure: always
    redirect_http_response_code: 301
    static_files: dist/\1
    upload: dist/.*\.js

  # Routing for any css files
  - url: /(.*\.css)
    secure: always
    redirect_http_response_code: 301
    static_files: dist/\1
    mime_type: text/css
    upload: dist/.*\.css

  # Routing for anything (wild card) after
  - url: /.*
    secure: always
    static_files: dist/index.html
    upload: dist/.*
```

## Google App Engine com Dockerfile

Neste link https://cloud.google.com/appengine/docs/flexible/custom-runtimes/quickstart contem um guia para realizar deploy no Google App Engine através de um arquivo Dockerfile. Com isto podemos usufruir do serviço do Google App Engine para realizar o deploy de uma imagem qualquer do Docker contendo uma tecnologia qualquer que não é possivel nativamente ao serviço. Isso é bom porque o Google App Engine é muito mais simples de se gerenciar do que o Google Compute Engine que trabalha com clusters e nós/nodos dificultando a sua manutenção.
