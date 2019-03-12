# <a name="how-to-run-the-completed-project"></a><span data-ttu-id="da636-101">Como executar o projeto concluído</span><span class="sxs-lookup"><span data-stu-id="da636-101">How to run the completed project</span></span>

## <a name="prerequisites"></a><span data-ttu-id="da636-102">Pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="da636-102">Prerequisites</span></span>

<span data-ttu-id="da636-103">Para executar o projeto concluído nessa pasta, você precisará do seguinte:</span><span class="sxs-lookup"><span data-stu-id="da636-103">To run the completed project in this folder, you need the following:</span></span>

- <span data-ttu-id="da636-104">[Ruby](https://www.ruby-lang.org/en/downloads/) instalado em sua máquina de desenvolvimento.</span><span class="sxs-lookup"><span data-stu-id="da636-104">[Ruby](https://www.ruby-lang.org/en/downloads/) installed on your development machine.</span></span> <span data-ttu-id="da636-105">Se você não tiver o Ruby, visite o link anterior para opções de download.</span><span class="sxs-lookup"><span data-stu-id="da636-105">If you do not have Ruby, visit the previous link for download options.</span></span> <span data-ttu-id="da636-106">(**Observação:** este tutorial foi escrito com a versão do Ruby 2.4.4.</span><span class="sxs-lookup"><span data-stu-id="da636-106">(**Note:** This tutorial was written with Ruby version 2.4.4.</span></span> <span data-ttu-id="da636-107">As etapas deste guia podem funcionar com outras versões, mas que não foram testadas.</span><span class="sxs-lookup"><span data-stu-id="da636-107">The steps in this guide may work with other versions, but that has not been tested.)</span></span>
- <span data-ttu-id="da636-108">Uma conta pessoal da Microsoft com uma caixa de correio no Outlook.com ou uma conta corporativa ou de estudante da Microsoft.</span><span class="sxs-lookup"><span data-stu-id="da636-108">Either a personal Microsoft account with a mailbox on Outlook.com, or a Microsoft work or school account.</span></span>

<span data-ttu-id="da636-109">Se você não tem uma conta da Microsoft, há algumas opções para obter uma conta gratuita:</span><span class="sxs-lookup"><span data-stu-id="da636-109">If you don't have a Microsoft account, there are a couple of options to get a free account:</span></span>

- <span data-ttu-id="da636-110">Você pode [se inscrever para uma nova conta pessoal da Microsoft](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span><span class="sxs-lookup"><span data-stu-id="da636-110">You can [sign up for a new personal Microsoft account](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span></span>
- <span data-ttu-id="da636-111">Você pode [se inscrever no programa para desenvolvedores do office 365](https://developer.microsoft.com/office/dev-program) para obter uma assinatura gratuita do Office 365.</span><span class="sxs-lookup"><span data-stu-id="da636-111">You can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

## <a name="register-a-web-application-with-the-application-registration-portal"></a><span data-ttu-id="da636-112">Registrar um aplicativo Web com o portal de registro do aplicativo</span><span class="sxs-lookup"><span data-stu-id="da636-112">Register a web application with the Application Registration Portal</span></span>

1. <span data-ttu-id="da636-113">Abra um navegador e navegue até o [portal de registro do aplicativo](https://apps.dev.microsoft.com).</span><span class="sxs-lookup"><span data-stu-id="da636-113">Open a browser and navigate to the [Application Registry Portal](https://apps.dev.microsoft.com).</span></span> <span data-ttu-id="da636-114">Faça logon usando uma **conta pessoal** (aka: conta da Microsoft) ou **conta corporativa ou de estudante**.</span><span class="sxs-lookup"><span data-stu-id="da636-114">Login using a **personal account** (aka: Microsoft Account) or **Work or School Account**.</span></span>

1. <span data-ttu-id="da636-115">Selecione **Adicionar um aplicativo** na parte superior da página.</span><span class="sxs-lookup"><span data-stu-id="da636-115">Select **Add an app** at the top of the page.</span></span>

    > <span data-ttu-id="da636-116">**Observação:** Se você vir mais de um botão **Adicionar um aplicativo** na página, selecione aquele que corresponde à lista **aplicativos convergidos** .</span><span class="sxs-lookup"><span data-stu-id="da636-116">**Note:** If you see more than one **Add an app** button on the page, select the one that corresponds to the **Converged apps** list.</span></span>

1. <span data-ttu-id="da636-117">Na página **registrar seu aplicativo** , defina o tutorial de gráfico do **nome do aplicativo** para o **Ruby on Rails** e selecione **criar**.</span><span class="sxs-lookup"><span data-stu-id="da636-117">On the **Register your application** page, set the **Application Name** to **Ruby on Rails Graph Tutorial** and select **Create**.</span></span>

    ![Captura de tela da criação de um novo aplicativo no site do portal de registro de aplicativo](/Images/arp-create-app-01.png)

1. <span data-ttu-id="da636-119">Na página **registro do tutorial do gráfico Ruby on Rails** , na seção **Propriedades** , copie a **ID do aplicativo** , pois você precisará dela mais tarde.</span><span class="sxs-lookup"><span data-stu-id="da636-119">On the **Ruby on Rails Graph Tutorial Registration** page, under the **Properties** section, copy the **Application Id** as you will need it later.</span></span>

    ![Captura de tela da ID do aplicativo recém-criado](/Images/arp-create-app-02.png)

1. <span data-ttu-id="da636-121">Role para baixo até a seção **segredos do aplicativo** .</span><span class="sxs-lookup"><span data-stu-id="da636-121">Scroll down to the **Application Secrets** section.</span></span>

    1. <span data-ttu-id="da636-122">Selecione **gerar nova senha**.</span><span class="sxs-lookup"><span data-stu-id="da636-122">Select **Generate New Password**.</span></span>
    1. <span data-ttu-id="da636-123">Na caixa de diálogo **nova senha gerada** , copie o conteúdo da caixa, pois você precisará dela mais tarde.</span><span class="sxs-lookup"><span data-stu-id="da636-123">In the **New password generated** dialog, copy the contents of the box as you will need it later.</span></span>

        > <span data-ttu-id="da636-124">**Importante:** Essa senha nunca é mostrada novamente, portanto, certifique-se de copiá-la agora.</span><span class="sxs-lookup"><span data-stu-id="da636-124">**Important:** This password is never shown again, so make sure you copy it now.</span></span>

    ![Captura de tela da senha do aplicativo recém-criado](/Images/arp-create-app-03.png)

1. <span data-ttu-id="da636-126">Role para baixo até a seção **plataformas** .</span><span class="sxs-lookup"><span data-stu-id="da636-126">Scroll down to the **Platforms** section.</span></span>

    1. <span data-ttu-id="da636-127">Selecione **Adicionar plataforma**.</span><span class="sxs-lookup"><span data-stu-id="da636-127">Select **Add Platform**.</span></span>
    1. <span data-ttu-id="da636-128">Na caixa de diálogo **Adicionar plataforma** , selecione **Web**.</span><span class="sxs-lookup"><span data-stu-id="da636-128">In the **Add Platform** dialog, select **Web**.</span></span>

        ![Captura de tela criando uma plataforma para o aplicativo](/Images/arp-create-app-04.png)

    1. <span data-ttu-id="da636-130">Na caixa plataforma **da Web** , digite a URL `http://localhost:3000/auth/microsoft_graph_auth/callback` das **URLs**de redirecionamento.</span><span class="sxs-lookup"><span data-stu-id="da636-130">In the **Web** platform box, enter the URL `http://localhost:3000/auth/microsoft_graph_auth/callback` for the **Redirect URLs**.</span></span>

        ![Captura de tela da nova plataforma Web adicionada para o aplicativo](/Images/arp-create-app-05.png)

1. <span data-ttu-id="da636-132">Role até a parte inferior da página e selecione **salvar**.</span><span class="sxs-lookup"><span data-stu-id="da636-132">Scroll to the bottom of the page and select **Save**.</span></span>

## <a name="configure-the-sample"></a><span data-ttu-id="da636-133">Configurar o exemplo</span><span class="sxs-lookup"><span data-stu-id="da636-133">Configure the sample</span></span>

1. <span data-ttu-id="da636-134">ReNomear `./config/oauth_environment_variables.rb.example` o `oauth_environment_variables.rb`arquivo para.</span><span class="sxs-lookup"><span data-stu-id="da636-134">Rename the `./config/oauth_environment_variables.rb.example` file to `oauth_environment_variables.rb`.</span></span>
1. <span data-ttu-id="da636-135">Edite `oauth_environment_variables.rb` o arquivo e faça as seguintes alterações.</span><span class="sxs-lookup"><span data-stu-id="da636-135">Edit the `oauth_environment_variables.rb` file and make the following changes.</span></span>
    1. <span data-ttu-id="da636-136">Substitua `YOUR_APP_ID_HERE` pela **ID do aplicativo** obtida do portal de registro do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="da636-136">Replace `YOUR_APP_ID_HERE` with the **Application Id** you got from the App Registration Portal.</span></span>
    1. <span data-ttu-id="da636-137">Substitua `YOUR APP PASSWORD HERE` pela senha obtida do portal de registro do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="da636-137">Replace `YOUR APP PASSWORD HERE` with the password you got from the App Registration Portal.</span></span>
1. <span data-ttu-id="da636-138">Na sua interface de linha de comando (CLI), navegue até este diretório e execute o seguinte comando para instalar os requisitos.</span><span class="sxs-lookup"><span data-stu-id="da636-138">In your command-line interface (CLI), navigate to this directory and run the following command to install requirements.</span></span>

    ```Shell
    bundle install
    ```

1. <span data-ttu-id="da636-139">Na sua CLI, execute o seguinte comando para inicializar o banco de dados do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="da636-139">In your CLI, run the following command to initialize the app's database.</span></span>

    ```Shell
    rake db:migrate
    ```

## <a name="run-the-sample"></a><span data-ttu-id="da636-140">Executar o exemplo</span><span class="sxs-lookup"><span data-stu-id="da636-140">Run the sample</span></span>

1. <span data-ttu-id="da636-141">Execute o comando a seguir em sua CLI para iniciar o aplicativo.</span><span class="sxs-lookup"><span data-stu-id="da636-141">Run the following command in your CLI to start the application.</span></span>

    ```Shell
    rails server
    ```

1. <span data-ttu-id="da636-142">Abra um navegador e navegue até `http://localhost:3000`.</span><span class="sxs-lookup"><span data-stu-id="da636-142">Open a browser and browse to `http://localhost:3000`.</span></span>