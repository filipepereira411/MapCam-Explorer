# Trabalhando com o repositório npm

Você pode configurar o npm para publicar pacotes em GitHub Packages e usar pacotes armazenados em GitHub Packages como dependências em um projeto npm.

<!-- 2148AF7B-5FF8-4B28-A808-D692FEE2225A -->

## Autenticando para GitHub Packages

> [!NOTE]
> GitHub Packages só dá suporte à autenticação usando um personal access token (classic). Para saber mais, confira [Gerenciar seus tokens de acesso pessoal](/pt/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens).

Você precisa de um token de acesso para publicar, instalar e excluir pacotes privados, públicos e internos.

Você pode usar um personal access token (classic) para se autenticar no GitHub Packages ou na API do GitHub. Ao criar um personal access token (classic), você pode atribuir diferentes escopos de token, dependendo da sua necessidade. Para obter mais informações sobre escopos relacionados a pacotes para personal access token (classic), confira [Sobre permissões para o GitHub Packages](/pt/packages/learn-github-packages/about-permissions-for-github-packages#about-scopes-and-permissions-for-package-registries).

Para efetuar a autenticação em um registro do GitHub Packages dentro de um fluxo de trabalho de GitHub Actions, você pode utilizar:

* `GITHUB_TOKEN` para publicar pacotes associados ao repositório do fluxo de trabalho.
* Um personal access token (classic) com pelo menos escopo `read:packages` para instalar pacotes associados a outros repositórios privados (`GITHUB_TOKEN` pode ser usado se o repositório receber acesso de leitura ao pacote. Confira, [Configurando o controle de acesso e visibilidade de um pacote](/pt/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility)).

### Autenticação em um GitHub Actions fluxo de trabalho

Esse registro dá suporte a permissões granulares. Para registros que dão suporte a permissões granulares, se o fluxo GitHub Actions de trabalho estiver usando um personal access token para autenticar em um registro, recomendamos que você atualize seu fluxo de trabalho para usar o `GITHUB_TOKEN`. Para obter diretrizes sobre como atualizar seus fluxos de trabalho que se autenticam em um registro com um personal access token[, consulte AUTOTITLE](/pt/packages/managing-github-packages-using-github-actions-workflows/publishing-and-installing-a-package-with-github-actions#upgrading-a-workflow-that-accesses-a-registry-using-a-personal-access-token).

> [!NOTE]
> A capacidade dos fluxos de trabalho do GitHub Actions de excluir e restaurar pacotes usando a API REST está em prévia pública e está sujeita a alterações.

Você poderá usar um `GITHUB_TOKEN` em um fluxo de trabalho GitHub Actions para excluir ou restaurar um pacote usando a API REST se o token tiver a permissão `admin` para o pacote. Repositórios que publicam pacotes usando um fluxo de trabalho e repositórios conectados explicitamente a pacotes recebem permissão `admin` para pacotes no repositório automaticamente.

Para obter mais informações sobre `GITHUB_TOKEN`, confira [Usar GITHUB\_TOKEN para autenticação em fluxos de trabalho](/pt/actions/tutorials/authenticate-with-github_token#using-the-github_token-in-a-workflow). Para obter mais informações sobre as melhores práticas ao usar um registro em ações, confira [Executores comprometidos](/pt/actions/concepts/security/compromised-runners#cross-repository-access).

Você também pode optar por conceder permissões de acesso a pacotes de forma independente paraGitHub Codespaces eGitHub Actions. Para saber mais, confira [Configurando o controle de acesso e visibilidade de um pacote](/pt/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility#ensuring-github-codespaces-access-to-your-package) e [Configurando o controle de acesso e visibilidade de um pacote](/pt/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility#ensuring-workflow-access-to-your-package).

### Autenticando com um personal access token

Você precisa usar um personal access token (classic) com os escopos apropriados para publicar e instalar pacotes no GitHub Packages. Para saber mais, confira [Introdução ao GitHub Packages](/pt/packages/learn-github-packages/introduction-to-github-packages#authenticating-to-github-packages).

Você pode se autenticar no GitHub Packages com o npm, seja editando o arquivo `~/.npmrc` do usuário para incluir o seu personal access token (classic), seja fazendo login no npm pela linha de comando usando seu nome de usuário e personal access token.

Para autenticar adicionando seu personal access token (classic) ao seu arquivo `~/.npmrc`, edite o arquivo `~/.npmrc` do seu projeto para incluir a seguinte linha, substituindo TOKEN pelo seu personal access token. Crie um arquivo `~/.npmrc` se ele não existir.

```shell
//npm.pkg.github.com/:_authToken=TOKEN
```

Para se autenticar fazendo login no npm, use o comando `npm login`, substituindo USERNAME pelo seu nome de usuário do GitHub, TOKEN pelo seu personal access token (classic) e PUBLIC-EMAIL-ADDRESS pelo seu endereço de e-mail.

Se você estiver usando a CLI do npm versão 9 ou superior e entrando ou saindo de um registro privado usando a linha de comando, use a opção `--auth-type=legacy` para ler os detalhes da autenticação nos prompts em vez de usar o fluxo de logon padrão em um navegador. Para obter mais informações, consulte [`npm-login`](https://docs.npmjs.com/cli/v10/commands/npm-login).

Se GitHub Packages não for o registro de pacotes padrão para usar o npm e você quiser usar o comando `npm audit`, recomendamos usar a flag `--scope` com o namespace que hospeda o pacote (a conta pessoal ou a organização à qual o pacote está vinculado) ao se autenticar no GitHub Packages.

```shell
$ npm login --scope=@NAMESPACE --auth-type=legacy --registry=https://npm.pkg.github.com

> Username: USERNAME
> Password: TOKEN
```

## Publicando um pacote

> [!NOTE]

> * Os nomes e os escopos dos pacotes só podem usar letras minúsculas.
> * O tarball para uma versão npm precisa ter menos de 256 MB de tamanho.

O GitHub Packages registro armazena pacotes npm em sua organização ou conta pessoal e permite associar um pacote a um repositório. Você pode escolher se deve herdar permissões de um repositório ou definir permissões granulares, independentemente de um repositório.

Ao publicar um pacote pela primeira vez a visibilidade-padrão será privada. Para mudar a visibilidade ou definir permissões de acesso, confira [Configurando o controle de acesso e visibilidade de um pacote](/pt/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility). Para obter mais informações sobre como vincular um pacote publicado a um repositório, consulte [Conectar um repositório a um pacote](/pt/packages/learn-github-packages/connecting-a-repository-to-a-package).

Você pode conectar um pacote a um repositório assim que o pacote for publicado incluindo um campo `repository` no arquivo `package.json`. Você também pode usar esse método para conectar vários pacotes ao mesmo repositório. Para obter mais informações, confira [Como publicar vários pacotes no mesmo repositório](#publishing-multiple-packages-to-the-same-repository).

> [!NOTE]
> Se você publicar um pacote vinculado a um repositório, o pacote herdará automaticamente as permissões de acesso do repositório vinculado e os fluxos de trabalho do GitHub Actions no repositório vinculado obterão automaticamente acesso ao pacote, a menos que a sua organização tenha desabilitado a herança automática das permissões de acesso. Para saber mais, confira [Configurando o controle de acesso e visibilidade de um pacote](/pt/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility#about-inheritance-of-access-permissions).

Configure o mapeamento do escopo para seu projeto usando um arquivo `.npmrc` local no projeto ou usando a opção `publishConfig` em `package.json`.
GitHub Packages só dá suporte a pacotes npm com escopo. Os pacotes com escopo têm nomes com o formato de `@NAMESPACE/PACKAGE-NAME`. Os pacotes com escopo sempre começam com um símbolo `@`. Talvez seja necessário atualizar o nome no `package.json` para usar o nome com escopo. Por exemplo, se você for o usuário `octocat` e seu pacote for nomeado `test`, você atribuirá o nome do pacote com escopo da seguinte maneira: `"name": "@octocat/test"`.

Após publicar um pacote, você poderá visualizá-lo no GitHub. Para saber mais, confira [Visualizar pacotes](/pt/packages/learn-github-packages/viewing-packages).

### Publicando um pacote usando um arquivo `.npmrc` local

Use um arquivo `.npmrc` para configurar o mapeamento de escopo para o projeto. No arquivo `.npmrc`, use a URL GitHub Packages e o proprietário da conta para que GitHub Packages saiba para onde encaminhar solicitações de pacotes. O uso de um `.npmrc` arquivo impede que outros desenvolvedores publiquem acidentalmente o pacote para npmjs.org em vez de GitHub Packages.

1. Autenticar para GitHub Packages. Para saber mais, confira [Autenticação no GitHub Packages](#authenticating-to-github-packages).

2. No mesmo diretório do arquivo `package.json`, crie ou edite um arquivo `.npmrc` para incluir uma linha especificando a URL do GitHub Packages e o namespace no qual o pacote está hospedado. Substitua `NAMESPACE` pelo nome da conta de usuário ou organização ao qual o pacote terá o escopo definido.

   ```shell
   @NAMESPACE:registry=https://npm.pkg.github.com
   ```

3. Adicione o arquivo *.npmrc* ao repositório em que o GitHub Packages pode encontrar seu projeto. Para saber mais, confira [Adicionar um arquivo a um repositório](/pt/repositories/working-with-files/managing-files/adding-a-file-to-a-repository).

4. Verifique o nome do pacote no `package.json` do projeto. O campo `name` precisa conter o escopo e o nome do pacote. Por exemplo, se o seu pacote se chama "test" e você o está publicando na organização "My-org" GitHub, o campo `name` no seu `package.json` deve ser `@my-org/test`.

5. Verifique o campo `repository` no `package.json` do seu projeto. O campo `repository` deve corresponder à URL do repositório do GitHub. Por exemplo, se a URL do repositório for `github.com/my-org/test`, o campo de repositório deverá ser `https://github.com/my-org/test.git`.

6. Publicar um pacote:

   ```shell
   npm publish
   ```

### Como publicar um pacote usando `publishConfig` no arquivo `package.json`

Use o elemento `publishConfig` no arquivo `package.json` para especificar o registro em que deseja publicar o pacote. Para obter mais informações, confira [publishConfig](https://docs.npmjs.com/files/package.json#publishconfig) na documentação do npm.

1. Edite o arquivo `package.json` do pacote e inclua uma entrada `publishConfig`.

   ```shell
   "publishConfig": {
     "registry": "https://npm.pkg.github.com"
   },
   ```

2. Verifique o campo `repository` no `package.json` do seu projeto. O campo `repository` deve corresponder à URL do repositório do GitHub. Por exemplo, se a URL do repositório for `github.com/my-org/test`, o campo de repositório deverá ser `https://github.com/my-org/test.git`.

3. Publicar um pacote:

   ```shell
   npm publish
   ```

## Publicar vários pacotes no mesmo repositório

Para publicar vários pacotes e vinculá-los ao mesmo repositório, você pode incluir a URL do GitHub repositório no `repository` campo do `package.json` arquivo para cada pacote. Para obter mais informações, confira [Criar um arquivo package.json](https://docs.npmjs.com/creating-a-package-json-file) e [Criar módulos Node.js](https://docs.npmjs.com/creating-node-js-modules) na documentação do npm.

Para garantir que a URL do repositório esteja correta, substitua `REPOSITORY` pelo nome do repositório que contém o pacote que você deseja publicar e `OWNER` pelo nome da conta pessoal ou organização que GitHub possui o repositório.

GitHub Packages corresponderá ao repositório com base na URL do pacote.

```shell
"repository":"https://github.com/OWNER/REPOSITORY",
```

## Instalando um pacote

Você pode instalar pacotes a partir de GitHub Packages, adicionando os pacotes como dependências no arquivo `package.json` do seu projeto. Para obter mais informações sobre como usar um `package.json` no seu projeto, consulte [Como trabalhar com package.json](https://docs.npmjs.com/getting-started/using-a-package.json) na documentação do npm.

Por padrão, você pode adicionar pacotes a partir de uma organização. Para obter mais informações, confira [Como instalar pacotes de outras organizações](#installing-packages-from-other-organizations).

Você também precisa adicionar o `.npmrc` arquivo ao seu projeto para que todas as solicitações para instalar pacotes passem GitHub Packages. Quando você roteia todas as solicitações de pacote por meio de GitHub Packages, pode usar pacotes com e sem escopo de *npmjs.org*. Para obter mais informações, consulte [npm-scope](https://docs.npmjs.com/misc/scope) na documentação do npm.

1. Autenticar para GitHub Packages. Para saber mais, confira [Autenticação no GitHub Packages](#authenticating-to-github-packages).

2. No mesmo diretório do arquivo `package.json`, crie ou edite um arquivo `.npmrc` para incluir uma linha especificando a URL do GitHub Packages e o namespace no qual o pacote está hospedado. Substitua `NAMESPACE` pelo nome da conta de usuário ou organização ao qual o pacote terá o escopo definido.

   ```shell
   @NAMESPACE:registry=https://npm.pkg.github.com
   ```

3. Adicione o arquivo *.npmrc* ao repositório em que o GitHub Packages pode encontrar seu projeto. Para saber mais, confira [Adicionar um arquivo a um repositório](/pt/repositories/working-with-files/managing-files/adding-a-file-to-a-repository).

4. Configure `package.json` no projeto para usar o pacote que você está instalando. Para adicionar as dependências do seu pacote ao arquivo `package.json` para GitHub Packages, especifique o nome completo do pacote com escopo, como `@my-org/server`. Para pacotes de *npmjs.com*, especifique o nome completo, como `@babel/core` ou `lodash`. Substitua `ORGANIZATION_NAME/PACKAGE_NAME` pela sua dependência de pacote.

   ```json
   {
     "name": "@my-org/server",
     "version": "1.0.0",
     "description": "Server app that uses the ORGANIZATION_NAME/PACKAGE_NAME package",
     "main": "index.js",
     "author": "",
     "license": "MIT",
     "dependencies": {
       "ORGANIZATION_NAME/PACKAGE_NAME": "1.0.0"
     }
   }
   ```

5. Instale o pacote.

   ```shell
   npm install
   ```

### Instalar pacotes de outras organizações

Por padrão, você só pode usar GitHub Packages pacotes de uma organização. Se você quiser rotear solicitações de pacote para várias organizações e usuários, poderá adicionar linhas adicionais ao arquivo`.npmrc`, substituindo`HOSTNAME`pelos nomes das contas pessoais ou organizações para as quais os pacotes têm escopo.

```shell
@NAMESPACE1:registry=https://npm.pkg.github.com
@NAMESPACE2:registry=https://npm.pkg.github.com
```
