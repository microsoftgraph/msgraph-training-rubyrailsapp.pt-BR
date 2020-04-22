<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="d7f9c-101">Neste exercício, você usará o [Ruby on Rails](https://rubyonrails.org/) para compilar um aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="d7f9c-101">In this exercise you will use [Ruby on Rails](https://rubyonrails.org/) to build a web app.</span></span>

1. <span data-ttu-id="d7f9c-102">Se você ainda não tiver os Rails instalados, você pode instalá-lo a partir da sua CLI (interface de linha de comando) com o seguinte comando.</span><span class="sxs-lookup"><span data-stu-id="d7f9c-102">If you don't already have Rails installed, you can install it from your command-line interface (CLI) with the following command.</span></span>

    ```Shell
    gem install rails -v 6.0.2.2
    ```

1. <span data-ttu-id="d7f9c-103">Abra sua CLI, navegue até um diretório no qual você tem direitos para criar arquivos e execute o seguinte comando para criar um novo aplicativo Rails.</span><span class="sxs-lookup"><span data-stu-id="d7f9c-103">Open your CLI, navigate to a directory where you have rights to create files, and run the following command to create a new Rails app.</span></span>

    ```Shell
    rails new graph-tutorial
    ```

1. <span data-ttu-id="d7f9c-104">Navegue até este novo diretório e digite o seguinte comando para iniciar um servidor Web local.</span><span class="sxs-lookup"><span data-stu-id="d7f9c-104">Navigate to this new directory and enter the following command to start a local web server.</span></span>

    ```Shell
    rails server
    ```

1. <span data-ttu-id="d7f9c-105">Abra o navegador e vá até `http://localhost:3000`.</span><span class="sxs-lookup"><span data-stu-id="d7f9c-105">Open your browser and navigate to `http://localhost:3000`.</span></span> <span data-ttu-id="d7f9c-106">Se tudo estiver funcionando, você verá um "Yay!</span><span class="sxs-lookup"><span data-stu-id="d7f9c-106">If everything is working, you will see a "Yay!</span></span> <span data-ttu-id="d7f9c-107">Você está nos trilhos! "</span><span class="sxs-lookup"><span data-stu-id="d7f9c-107">You're on Rails!"</span></span> <span data-ttu-id="d7f9c-108">Mensagem.</span><span class="sxs-lookup"><span data-stu-id="d7f9c-108">message.</span></span> <span data-ttu-id="d7f9c-109">Se você não vir essa mensagem, consulte o [Guia de introdução ao Rails](http://guides.rubyonrails.org/).</span><span class="sxs-lookup"><span data-stu-id="d7f9c-109">If you don't see that message, check the [Rails getting started guide](http://guides.rubyonrails.org/).</span></span>

## <a name="install-gems"></a><span data-ttu-id="d7f9c-110">Instalar Gems</span><span class="sxs-lookup"><span data-stu-id="d7f9c-110">Install gems</span></span>

<span data-ttu-id="d7f9c-111">Antes de prosseguir, instale algumas Gems adicionais que serão usadas posteriormente:</span><span class="sxs-lookup"><span data-stu-id="d7f9c-111">Before moving on, install some additional gems that you will use later:</span></span>

- <span data-ttu-id="d7f9c-112">[omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) para manipulação de entrada e fluxos de token OAuth.</span><span class="sxs-lookup"><span data-stu-id="d7f9c-112">[omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) for handling sign-in and OAuth token flows.</span></span>
- <span data-ttu-id="d7f9c-113">[omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) para adicionar proteção do CSRF ao omniauth.</span><span class="sxs-lookup"><span data-stu-id="d7f9c-113">[omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) for adding CSRF protection to OmniAuth.</span></span>
- <span data-ttu-id="d7f9c-114">[httparty](https://github.com/jnunemaker/httparty) para fazer chamadas para o Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="d7f9c-114">[httparty](https://github.com/jnunemaker/httparty) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="d7f9c-115">[ActiveRecord-session_store](https://github.com/rails/activerecord-session_store) para armazenar sessões no banco de dados.</span><span class="sxs-lookup"><span data-stu-id="d7f9c-115">[activerecord-session_store](https://github.com/rails/activerecord-session_store) for storing sessions in the database.</span></span>

1. <span data-ttu-id="d7f9c-116">Abra **./Gemfile** e adicione as linhas a seguir.</span><span class="sxs-lookup"><span data-stu-id="d7f9c-116">Open **./Gemfile** and add the following lines.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/Gemfile" id="GemFileSnippet":::

1. <span data-ttu-id="d7f9c-117">Na sua CLI, execute o seguinte comando.</span><span class="sxs-lookup"><span data-stu-id="d7f9c-117">In your CLI, run the following command.</span></span>

    ```Shell
    bundle install
    ```

1. <span data-ttu-id="d7f9c-118">Na sua CLI, execute os seguintes comandos para configurar o banco de dados para armazenar sessões.</span><span class="sxs-lookup"><span data-stu-id="d7f9c-118">In your CLI, run the following commands to configure the database for storing sessions.</span></span>

    ```Shell
    rails generate active_record:session_migration
    rake db:migrate
    ```

1. <span data-ttu-id="d7f9c-119">Crie um novo arquivo chamado `session_store.rb` no diretório **./config/Initializers** e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="d7f9c-119">Create a new file called `session_store.rb` in the **./config/initializers** directory, and add the following code.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/session_store.rb" id="SessionStoreSnippet":::

## <a name="design-the-app"></a><span data-ttu-id="d7f9c-120">Projetar o aplicativo</span><span class="sxs-lookup"><span data-stu-id="d7f9c-120">Design the app</span></span>

<span data-ttu-id="d7f9c-121">Nesta seção, você criará a interface do usuário básica para o aplicativo.</span><span class="sxs-lookup"><span data-stu-id="d7f9c-121">In this section you'll create the basic UI for the app.</span></span>

1. <span data-ttu-id="d7f9c-122">Abra **./app/views/layouts/Application.html.erb** e substitua seu conteúdo pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="d7f9c-122">Open **./app/views/layouts/application.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/layouts/application.html.erb" id="LayoutSnippet":::

    <span data-ttu-id="d7f9c-123">Este código adiciona a [inicialização](http://getbootstrap.com/) para estilos simples e a [fonte incrível](https://fontawesome.com/) para alguns ícones simples.</span><span class="sxs-lookup"><span data-stu-id="d7f9c-123">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Font Awesome](https://fontawesome.com/) for some simple icons.</span></span> <span data-ttu-id="d7f9c-124">Também define um layout global com uma barra de navegação.</span><span class="sxs-lookup"><span data-stu-id="d7f9c-124">It also defines a global layout with a nav bar.</span></span>

1. <span data-ttu-id="d7f9c-125">Abra **./app/assets/stylesheets/Application.css** e adicione o seguinte ao final do arquivo.</span><span class="sxs-lookup"><span data-stu-id="d7f9c-125">Open **./app/assets/stylesheets/application.css** and add the following to the end of the file.</span></span>

    :::code language="css" source="../demo/graph-tutorial/app/assets/stylesheets/application.css" id="CssSnippet":::

1. <span data-ttu-id="d7f9c-126">Gere um controlador de home page com o comando a seguir.</span><span class="sxs-lookup"><span data-stu-id="d7f9c-126">Generate a home page controller with the following command.</span></span>

    ```Shell
    rails generate controller Home index
    ```

1. <span data-ttu-id="d7f9c-127">Configure a `index` ação no `Home` controlador como a página padrão para o aplicativo.</span><span class="sxs-lookup"><span data-stu-id="d7f9c-127">Configure the `index` action on the `Home` controller as the default page for the app.</span></span> <span data-ttu-id="d7f9c-128">Abra **./config/routes.rb** e substitua seu conteúdo pelo seguinte</span><span class="sxs-lookup"><span data-stu-id="d7f9c-128">Open **./config/routes.rb** and replace its contents with the following</span></span>

    ```ruby
    Rails.application.routes.draw do
      get 'home/index'
      root 'home#index'

      # Add future routes here

    end
    ```

1. <span data-ttu-id="d7f9c-129">Abra **./app/view/home/index.html.erb** e substitua seu conteúdo pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="d7f9c-129">Open **./app/view/home/index.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/home/index.html.erb" id="HomeSnippet":::

1. <span data-ttu-id="d7f9c-130">Salve todas as suas alterações e reinicie o servidor.</span><span class="sxs-lookup"><span data-stu-id="d7f9c-130">Save all of your changes and restart the server.</span></span> <span data-ttu-id="d7f9c-131">Agora, o aplicativo deve ser muito diferente.</span><span class="sxs-lookup"><span data-stu-id="d7f9c-131">Now, the app should look very different.</span></span>

    ![Uma captura de tela da página inicial reprojetada](./images/create-app-01.png)
