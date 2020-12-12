<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="504d0-101">Este tutorial ensina como criar um aplicativo Web do Ruby on Rails que usa a API do Microsoft Graph para recuperar informações de calendário de um usuário.</span><span class="sxs-lookup"><span data-stu-id="504d0-101">This tutorial teaches you how to build a Ruby on Rails web app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="504d0-102">Se preferir baixar o tutorial concluído, você pode baixá-lo de duas maneiras.</span><span class="sxs-lookup"><span data-stu-id="504d0-102">If you prefer to just download the completed tutorial, you can download it in two ways.</span></span>
>
> - <span data-ttu-id="504d0-103">Baixe o [início rápido do Ruby](https://developer.microsoft.com/graph/quick-start?platform=option-ruby) para obter o código de trabalho em minutos.</span><span class="sxs-lookup"><span data-stu-id="504d0-103">Download the [Ruby quick start](https://developer.microsoft.com/graph/quick-start?platform=option-ruby) to get working code in minutes.</span></span>
> - <span data-ttu-id="504d0-104">Baixe ou clone o [repositório do GitHub](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span><span class="sxs-lookup"><span data-stu-id="504d0-104">Download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="504d0-105">Pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="504d0-105">Prerequisites</span></span>

<span data-ttu-id="504d0-106">Antes de iniciar este tutorial, você deve ter as seguintes ferramentas instaladas em sua máquina de desenvolvimento.</span><span class="sxs-lookup"><span data-stu-id="504d0-106">Before you start this tutorial, you should have the following tools installed on your development machine.</span></span>

- [<span data-ttu-id="504d0-107">Ruby</span><span class="sxs-lookup"><span data-stu-id="504d0-107">Ruby</span></span>](https://www.ruby-lang.org/en/downloads/)
- [<span data-ttu-id="504d0-108">SQLite3</span><span class="sxs-lookup"><span data-stu-id="504d0-108">SQLite3</span></span>](https://sqlite.org/index.html)
- [<span data-ttu-id="504d0-109">Node.js</span><span class="sxs-lookup"><span data-stu-id="504d0-109">Node.js</span></span>](https://nodejs.org/en/)
- [<span data-ttu-id="504d0-110">PNPM</span><span class="sxs-lookup"><span data-stu-id="504d0-110">Yarn</span></span>](https://yarnpkg.com/)

<span data-ttu-id="504d0-111">Você também deve ter uma conta pessoal da Microsoft com uma caixa de correio no Outlook.com ou uma conta corporativa ou de estudante da Microsoft.</span><span class="sxs-lookup"><span data-stu-id="504d0-111">You should also have either a personal Microsoft account with a mailbox on Outlook.com, or a Microsoft work or school account.</span></span> <span data-ttu-id="504d0-112">Se você não tem uma conta da Microsoft, há algumas opções para obter uma conta gratuita:</span><span class="sxs-lookup"><span data-stu-id="504d0-112">If you don't have a Microsoft account, there are a couple of options to get a free account:</span></span>

- <span data-ttu-id="504d0-113">Você pode [se inscrever para uma nova conta pessoal da Microsoft](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span><span class="sxs-lookup"><span data-stu-id="504d0-113">You can [sign up for a new personal Microsoft account](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span></span>
- <span data-ttu-id="504d0-114">Você pode [se inscrever no programa para desenvolvedores do office 365](https://developer.microsoft.com/office/dev-program) para obter uma assinatura gratuita do Office 365.</span><span class="sxs-lookup"><span data-stu-id="504d0-114">You can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

> [!NOTE]
> <span data-ttu-id="504d0-115">Este tutorial foi escrito com as seguintes versões das ferramentas necessárias.</span><span class="sxs-lookup"><span data-stu-id="504d0-115">This tutorial was written with the following versions of the required tools.</span></span> <span data-ttu-id="504d0-116">As etapas deste guia podem funcionar com outras versões, mas que não foram testadas.</span><span class="sxs-lookup"><span data-stu-id="504d0-116">The steps in this guide may work with other versions, but that has not been tested.</span></span>
>
> - <span data-ttu-id="504d0-117">Versão do Ruby 2.6.6.</span><span class="sxs-lookup"><span data-stu-id="504d0-117">Ruby version 2.6.6.</span></span>
> - <span data-ttu-id="504d0-118">SQLite3 versão 3.31.1</span><span class="sxs-lookup"><span data-stu-id="504d0-118">SQLite3 version 3.31.1</span></span>
> - <span data-ttu-id="504d0-119">Node.js versão 14.15.0</span><span class="sxs-lookup"><span data-stu-id="504d0-119">Node.js version 14.15.0</span></span>
> - <span data-ttu-id="504d0-120">PNPM versão 1.22.0</span><span class="sxs-lookup"><span data-stu-id="504d0-120">Yarn version 1.22.0</span></span>

## <a name="feedback"></a><span data-ttu-id="504d0-121">Comentários</span><span class="sxs-lookup"><span data-stu-id="504d0-121">Feedback</span></span>

<span data-ttu-id="504d0-122">Forneça comentários sobre este tutorial no [repositório do GitHub](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span><span class="sxs-lookup"><span data-stu-id="504d0-122">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span></span>
