<!-- markdownlint-disable MD002 MD041 -->

Este tutorial ensina como criar um aplicativo Web Ruby on Rails que usa a API do Microsoft Graph para recuperar informações de calendário para um usuário.

> [!TIP]
> Se você preferir apenas baixar o tutorial concluído, poderá baixá-lo de duas maneiras.
>
> - Baixe o [início rápido do Ruby](https://developer.microsoft.com/graph/quick-start?platform=option-ruby) para obter o código de trabalho em minutos.
> - Baixe ou clone o [GitHub repositório](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).

## <a name="prerequisites"></a>Pré-requisitos

Antes de iniciar este tutorial, você deve ter as seguintes ferramentas instaladas em sua máquina de desenvolvimento.

- [Ruby](https://www.ruby-lang.org/en/downloads/)
- [SQLite3](https://sqlite.org/index.html)
- [Node.js](https://nodejs.org/en/)
- [Yarn](https://yarnpkg.com/)

Você também deve ter uma conta pessoal da Microsoft com uma caixa de correio em Outlook.com, ou uma conta de trabalho ou de estudante da Microsoft. Se você não tiver uma conta da Microsoft, há algumas opções para obter uma conta gratuita:

- Você pode [se inscrever em uma nova conta pessoal da Microsoft.](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)
- Você pode [se inscrever no programa Office 365 desenvolvedor para](https://developer.microsoft.com/office/dev-program) obter uma assinatura Office 365 gratuita.

> [!NOTE]
> Este tutorial foi escrito com as seguintes versões das ferramentas necessárias. As etapas neste guia podem funcionar com outras versões, mas que não foram testadas.
>
> - Ruby versão 3.0.1
> - SQLite3 versão 3.35.5
> - Node.js versão 14.15.0
> - Yarn versão 1.22.0

## <a name="feedback"></a>Comentários

Forneça qualquer comentário sobre este tutorial no [repositório GitHub .](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp)
