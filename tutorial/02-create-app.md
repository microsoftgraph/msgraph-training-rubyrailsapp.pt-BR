<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="67105-101">Neste exercício, você usará [Ruby on Rails](https://rubyonrails.org/) para criar um aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="67105-101">In this exercise you will use [Ruby on Rails](https://rubyonrails.org/) to build a web app.</span></span>

1. <span data-ttu-id="67105-102">Se você ainda não tiver Rails instalado, poderá instalá-lo a partir de sua interface de linha de comando (CLI) com o seguinte comando.</span><span class="sxs-lookup"><span data-stu-id="67105-102">If you don't already have Rails installed, you can install it from your command-line interface (CLI) with the following command.</span></span>

    ```Shell
    gem install rails -v 6.1.3.1
    ```

1. <span data-ttu-id="67105-103">Abra sua CLI, navegue até um diretório onde você tem direitos para criar arquivos e execute o seguinte comando para criar um novo aplicativo Rails.</span><span class="sxs-lookup"><span data-stu-id="67105-103">Open your CLI, navigate to a directory where you have rights to create files, and run the following command to create a new Rails app.</span></span>

    ```Shell
    rails new graph-tutorial
    ```

1. <span data-ttu-id="67105-104">Navegue até esse novo diretório e insira o seguinte comando para iniciar um servidor Web local.</span><span class="sxs-lookup"><span data-stu-id="67105-104">Navigate to this new directory and enter the following command to start a local web server.</span></span>

    ```Shell
    rails server
    ```

1. <span data-ttu-id="67105-105">Abra o navegador e vá até `http://localhost:3000`.</span><span class="sxs-lookup"><span data-stu-id="67105-105">Open your browser and navigate to `http://localhost:3000`.</span></span> <span data-ttu-id="67105-106">Se tudo estiver funcionando, você verá um "Yay!</span><span class="sxs-lookup"><span data-stu-id="67105-106">If everything is working, you will see a "Yay!</span></span> <span data-ttu-id="67105-107">Você está em Rails!"</span><span class="sxs-lookup"><span data-stu-id="67105-107">You're on Rails!"</span></span> <span data-ttu-id="67105-108">Mensagem.</span><span class="sxs-lookup"><span data-stu-id="67105-108">message.</span></span> <span data-ttu-id="67105-109">Se você não vir essa mensagem, verifique o guia [Rails getting started](http://guides.rubyonrails.org/).</span><span class="sxs-lookup"><span data-stu-id="67105-109">If you don't see that message, check the [Rails getting started guide](http://guides.rubyonrails.org/).</span></span>

## <a name="install-gems"></a><span data-ttu-id="67105-110">Instalar gemas</span><span class="sxs-lookup"><span data-stu-id="67105-110">Install gems</span></span>

<span data-ttu-id="67105-111">Antes de continuar, instale algumas gemas adicionais que você usará posteriormente:</span><span class="sxs-lookup"><span data-stu-id="67105-111">Before moving on, install some additional gems that you will use later:</span></span>

- <span data-ttu-id="67105-112">[omniauth-oauth2 para](https://github.com/omniauth/omniauth-oauth2) manipular fluxos de tokens OAuth e de entrada.</span><span class="sxs-lookup"><span data-stu-id="67105-112">[omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) for handling sign-in and OAuth token flows.</span></span>
- <span data-ttu-id="67105-113">[omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) para adicionar proteção CSRF ao OmniAuth.</span><span class="sxs-lookup"><span data-stu-id="67105-113">[omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) for adding CSRF protection to OmniAuth.</span></span>
- <span data-ttu-id="67105-114">[httparty](https://github.com/jnunemaker/httparty) para fazer chamadas para o Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="67105-114">[httparty](https://github.com/jnunemaker/httparty) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="67105-115">[activerecord-session_store](https://github.com/rails/activerecord-session_store) para armazenar sessões no banco de dados.</span><span class="sxs-lookup"><span data-stu-id="67105-115">[activerecord-session_store](https://github.com/rails/activerecord-session_store) for storing sessions in the database.</span></span>

1. <span data-ttu-id="67105-116">Abra **./Gemfile** e adicione as seguintes linhas.</span><span class="sxs-lookup"><span data-stu-id="67105-116">Open **./Gemfile** and add the following lines.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/Gemfile" id="GemFileSnippet":::

1. <span data-ttu-id="67105-117">Em sua CLI, execute o seguinte comando.</span><span class="sxs-lookup"><span data-stu-id="67105-117">In your CLI, run the following command.</span></span>

    ```Shell
    bundle install
    ```

1. <span data-ttu-id="67105-118">Em sua CLI, execute os seguintes comandos para configurar o banco de dados para armazenar sessões.</span><span class="sxs-lookup"><span data-stu-id="67105-118">In your CLI, run the following commands to configure the database for storing sessions.</span></span>

    ```Shell
    rails generate active_record:session_migration
    rake db:migrate
    ```

1. <span data-ttu-id="67105-119">Crie um novo arquivo chamado `session_store.rb` no **diretório ./config/initializers** e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="67105-119">Create a new file called `session_store.rb` in the **./config/initializers** directory, and add the following code.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/session_store.rb" id="SessionStoreSnippet":::

## <a name="design-the-app"></a><span data-ttu-id="67105-120">Design do aplicativo</span><span class="sxs-lookup"><span data-stu-id="67105-120">Design the app</span></span>

<span data-ttu-id="67105-121">Nesta seção, você criará a interface do usuário básica do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="67105-121">In this section you'll create the basic UI for the app.</span></span>

1. <span data-ttu-id="67105-122">Abra **./app/views/layouts/application.html.erb e** substitua seu conteúdo pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="67105-122">Open **./app/views/layouts/application.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/layouts/application.html.erb" id="LayoutSnippet":::

    <span data-ttu-id="67105-123">Este código adiciona [Bootstrap](http://getbootstrap.com/) para estilo simples e [Fabric Core](https://developer.microsoft.com/fluentui#/get-started#fabric-core) para alguns ícones simples.</span><span class="sxs-lookup"><span data-stu-id="67105-123">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Fabric Core](https://developer.microsoft.com/fluentui#/get-started#fabric-core) for some simple icons.</span></span> <span data-ttu-id="67105-124">Ele também define um layout global com uma barra de nav.</span><span class="sxs-lookup"><span data-stu-id="67105-124">It also defines a global layout with a nav bar.</span></span>

1. <span data-ttu-id="67105-125">Abra **./app/assets/stylesheets/application.css** e adicione o seguinte ao final do arquivo.</span><span class="sxs-lookup"><span data-stu-id="67105-125">Open **./app/assets/stylesheets/application.css** and add the following to the end of the file.</span></span>

    :::code language="css" source="../demo/graph-tutorial/app/assets/stylesheets/application.css" id="CssSnippet":::

1. <span data-ttu-id="67105-126">Gere um controlador de home page com o seguinte comando.</span><span class="sxs-lookup"><span data-stu-id="67105-126">Generate a home page controller with the following command.</span></span>

    ```Shell
    rails generate controller Home index
    ```

1. <span data-ttu-id="67105-127">Configure a `index` ação no controlador como a página padrão do `Home` aplicativo.</span><span class="sxs-lookup"><span data-stu-id="67105-127">Configure the `index` action on the `Home` controller as the default page for the app.</span></span> <span data-ttu-id="67105-128">Abra **./config/routes.rb e** substitua seu conteúdo pelo seguinte</span><span class="sxs-lookup"><span data-stu-id="67105-128">Open **./config/routes.rb** and replace its contents with the following</span></span>

    ```ruby
    Rails.application.routes.draw do
      get 'home/index'
      root 'home#index'

      # Add future routes here

    end
    ```

1. <span data-ttu-id="67105-129">Abra **./app/view/home/index.html.erb e** substitua seu conteúdo pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="67105-129">Open **./app/view/home/index.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/home/index.html.erb" id="HomeSnippet":::

1. <span data-ttu-id="67105-130">Adicione um arquivo PNG chamado **no-profile-photo.png** no **diretório ./app/assets/images.**</span><span class="sxs-lookup"><span data-stu-id="67105-130">Add a PNG file named **no-profile-photo.png** in the **./app/assets/images** directory.</span></span>

1. <span data-ttu-id="67105-131">Salve todas as suas alterações e reinicie o aplicativo.</span><span class="sxs-lookup"><span data-stu-id="67105-131">Save all of your changes and restart the server.</span></span> <span data-ttu-id="67105-132">Agora, o aplicativo deve ter uma aparência muito diferente.</span><span class="sxs-lookup"><span data-stu-id="67105-132">Now, the app should look very different.</span></span>

    ![Captura de tela da home page](./images/create-app-01.png)
