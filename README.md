# ci-test

Repositório utilizado para testes do fluxo de CI e deploy via ftp.

## FTP
Foi necessário um servidor ftp para realização dos testes.
Neste caso fooi utilizada uma hospedagem gratuita do 000webhost.

## Github Actions
Para utilizar as Actions o repositório deve conter o diretório [`.github/workflows`](.github/workflows) onde serão salvos os arquivos `.yml` que definem os fluxos de trabalho.

### Fluxo de trabalho
Os nomes dos arquivos de workflow não precisam seguir um padrão, porém devem ser sempre arquivos `.yml`.

Ambos os fluxos utilizados para os testes de deploy ([deploy-homolog](.github/workflows/deploy-homolog.yml) e [deploy-prod](.github/workflows/deploy-prod.yml)) são quase idênticos, sua única diferença está nos gatilhos para iniciar o workflow, onde o fluxo de homologação pode ser iniciado, além de manualmente, mas também por _push_ e _pull requests_ ao branch `homolog`, enquanto o fluxo de produção somente pode ser iniciado manualmente.

### Arquivos de workflow
Os arquivos de workflow utilizados são compostos pelos seguintes atributos:

#### name
O nome do fluxo de trabalho.
```yaml
name: deploy-homolog
```

#### on
Os gatilhos para o fluxo de trabalho.

*workflow-dispatch:* configura o fluxo para ser iniciado manualmente.

*push:* configura o fluxo para ser iniciado por _push_ em um brach determinado.

*pull_request:* configura o fluxo para ser iniciado por _pull requests_ em um brach determinado.

```yaml
on:
    push:
        branches: [homolog]
    pull_request:
        branches: [homolog]
    
    workflow-dispatch:
```

#### jobs
Lista de _jobs_ que serão executados pelo fluxo de trabalho.

No ppresente momento somente o _job_ `upload` é utilizado.

*UPLOAD*
Esse job é responsável por fazer o upload do repositório no ftp e composto pelos seguintes atributos:

*runs-on:* determina o tipo de _runner_ que executará o _job_.

*steps:* os passos a serem executados, informando quais (quando necessário) actions extrnas serão utilizadas em um step, os parâmetros dessas actions e o nome do passo.
> A actioon da comunidade [airvzxf/ftp-deployment-action](https://github.com/marketplace/actions/ftp-deployment) é utilizada para fazer o upload do repositório via ftp.

```yaml
jobs:
    upload:
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v2
            - name: ftp-action
              uses: airvzxf/ftp-deployment-action@latest
              with:
                server: ${{ secrets.FTP_SERVER }}
                user: ${{ secrets.FTP_USERNAME }}
                password: ${{ secrets.FTP_PASSWORD }}
                local_dir: "./html"
                remote_dir: "./public_html"
```

> No workflow de `deploy-prod` um *step* adicional é utilizado antes do *step* `ftp-action` para realizar automaticamente o *merge* do *branch* `homolog` para o *branch* `main`.:
> ```yaml
> - uses: actions/checkout@v2
>
> - name: merge
>   uses: devmasx/merge-branch@master
>   with:
>     type: now
>     from_branch: homolog
>     target_branch: main
>     github_token: ${{ secrets.GITHUB_TOKEN }}
Esse *step* utiliza uma action da comunidade [devmasx/merge-branch](https://github.com/marketplace/actions/merge-branch) para realizar o *merge* do *branch* `homolog` para o *branch* `main`.

### Credenciais
Por motivos de segurança as credenciais de comunicação com o servidor ftp, assim como o endereço do mesmo, foram salvas e acessadas por meio de `secrets`, variáveis de ambiente voltadas para esse tipo de informações.


Um _secret_ pode ser definido no repositório indo em `Settings > Secrets > Actions` e clicando em `New repository secret`.

Após isso o _secret_ pode ser acessado por com a sintaxe `{{ secrets.SECRET_NAME }}` conforme visto no código acima.