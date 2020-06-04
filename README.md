# Google Cloud

- [App Engine](#app-engine)
  - [Deploy de aplicação Angular no Google App Engine](#deploy-de-aplicação-angular-no-google-app-engine)
    - [Pré-requisitos](#pré-requisitos)
    - [Procedimentos](#procedimentos)
  - [Redirecionamento entre serviços](#redirecionamento-entre-serviços)
  - [Google App Engine com Dockerfile](#google-app-engine-com-dockerfile)


## Deploy de aplicação Angular no Google App Engine

### Pré-requisitos

* Google Cloud SDK
* Aplicação construída

### Procedimentos

#### Criando arquivo de implantação
 
Após realizar o build da aplicação Angular o qual irá gerar um diretório de saída com todos os fontes transpilados de Typescript para Javascript, vamos criar um arquivo chamado `app.yaml` o qual irá descrever para o Google Cloud SDK as configurações de implantação da nossa aplicação.

O arquivo `app.yaml`: 

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

Como pode ser visto, a maior parte das configurações, referem-se a rotas. Isso porque como a nossa aplicação é uma SPA (Single Page Application), é preciso que todos os caminhos de rotas sejam direcionados para o arquivo index.html, com exceção dos arquivos estáticos.

Não menos importante, o arquivo `app.yaml` também irá implantar nossa aplicação como se fosse uma aplicação em Python. Contudo, isso é somente um artificio para que nossa aplicação execute no App Engine com o menor custo possível e entregue os arquivos estáticos. Não obstante, nossa aplicação não oferece nenhum código fonte em Python.

Outro ponto importante ressaltar é a configuração `service`. Como provavelmente nosso projeto no Google Cloud Platform já possua, ou vai possuir, um serviço backend executando no App Engine, especificamos no arquivo `app.yaml` o valor `front`. Dessa forma teremos um serviço no App Engine denominado `front` executando simultâneamente a outros serviços sem impactá-los. Isso é importante para depois conseguirmos utilizar o mesmo domínio gerado pelo Google para redirecionar requisições para diferentes serviços, dependendo de caminhos de URL. Essa configuração é explicada em [Redirecionamento entre serviços](#redirecionamento-entre-serviços).

#### Deploy

Criar um diretório qualquer e dentro deste diretório colocar o diretório de saída do processo de build de nossa aplicação Angular e o arquivos `app.yaml`. 

Abaixo uma representação de como ficará:

```
new_directory
  app.yaml
  dist
```

Dentro do diretório `new_directory`, no caso, executar o comando:

```
gcloud app deploy --version 1
```

Isso irá realizar a implantação no App Engine na nossa aplicação, e no final do processo será disponibilizado a URL para acesso da mesma.

## Redirecionamento entre serviços

No App Engine podemos ter em um mesmo projeto Google Cloud, diversos serviços executando simultâneamente. Isso nos permite modularizar nossas soluções sem a necessidade de separá-las entre diferentes projetos Google Cloud e sem termos a necessidade de criar uma única solução monolítica. Outra vantagem é que podemos ter configurações de infraestrutura diferentes entre serviços. Por exemplo: o serviço A pode ser muito mais utilizado que o serviço B. Portanto é lógico que o serviço A possua uma configuração de instância de máquina mais potente que o serviço B.

Independente da quantidade de serviços que possam estar executando no App Engine, sempre deverá existir um serviço denominado `default`. Geralmente o serviço `default` é o serviço principal, o core de nossa solução. É comum que este serviço seja o backend principal de uma solução.

Cada serviço executando é acessado através de um subdomínio do domínio principal, ou seja, uma variante do domínio disponibilizado pelo serviço `default`. 

Portanto imaginemos o seguinte cenário: Um serviço backend denominado `default` e um serviço frontend denominado `front`. No processo de implantação de cada um desses serviços será informado a URL de acesso ao mesmo. Por exemplo: se ID do projeto for oceanic-toolbox-1234, nosso serviço `default` será acessado pela URL https://oceanic-toolbox-1234.appspot.com.br, e o serviço `front` pela URL https://front-dot-oceanic-toolbox-1234.appspot.com.br. Paralelamente a isto, digamos que possuímos um domínio personalizado, a saber: meudominiopersonalizado.com.br, e que este domínio personalizado já esteja validado e vinculado ao nosso projeto Google Cloud. Como poderíamos especificar ao App Engine que toda requisição https://meudominiopersonalizado.com.br seja redirecionado para o serviço `front` e que toda requisição https://meudominiopersonalizado.com.br/api/* seja redirecionado para o serviço `default`? Para isso precisaremos informar estas regras ao App Engine através da configuração de um arquivo denominado [dispatch.yaml](https://cloud.google.com/appengine/docs/standard/java/reference/dispatch-yaml?hl=pt-br#maven).


## Google App Engine com Dockerfile

Neste link https://cloud.google.com/appengine/docs/flexible/custom-runtimes/quickstart contem um guia para realizar deploy no Google App Engine através de um arquivo Dockerfile. Com isto podemos usufruir do serviço do Google App Engine para realizar o deploy de uma imagem qualquer do Docker contendo uma tecnologia qualquer que não é possivel nativamente ao serviço. Isso é bom porque o Google App Engine é muito mais simples de se gerenciar do que o Google Compute Engine que trabalha com clusters e nós/nodos dificultando a sua manutenção.
