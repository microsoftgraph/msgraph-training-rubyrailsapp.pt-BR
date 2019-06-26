<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="1ac0d-101">Neste exercício, você usará o [Ruby on Rails](https://rubyonrails.org/) para compilar um aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="1ac0d-101">In this exercise you will use [Ruby on Rails](https://rubyonrails.org/) to build a web app.</span></span> <span data-ttu-id="1ac0d-102">Se você ainda não tiver os Rails instalados, você pode instalá-lo a partir da sua CLI (interface de linha de comando) com o seguinte comando.</span><span class="sxs-lookup"><span data-stu-id="1ac0d-102">If you don't already have Rails installed, you can install it from your command-line interface (CLI) with the following command.</span></span>

```Shell
gem install rails
```

<span data-ttu-id="1ac0d-103">Abra sua CLI, navegue até um diretório no qual você tem direitos para criar arquivos e execute o seguinte comando para criar um novo aplicativo Rails.</span><span class="sxs-lookup"><span data-stu-id="1ac0d-103">Open your CLI, navigate to a directory where you have rights to create files, and run the following command to create a new Rails app.</span></span>

```Shell
rails new graph-tutorial
```

<span data-ttu-id="1ac0d-104">O Rails cria um novo diretório chamado `graph-tutorial` e estruturará um aplicativo Rails.</span><span class="sxs-lookup"><span data-stu-id="1ac0d-104">Rails creates a new directory called `graph-tutorial` and scaffolds a Rails app.</span></span> <span data-ttu-id="1ac0d-105">Navegue até este novo diretório e digite o seguinte comando para iniciar um servidor Web local.</span><span class="sxs-lookup"><span data-stu-id="1ac0d-105">Navigate to this new directory and enter the following command to start a local web server.</span></span>

```Shell
rails server
```

<span data-ttu-id="1ac0d-106">Abra o navegador e vá até `http://localhost:3000`.</span><span class="sxs-lookup"><span data-stu-id="1ac0d-106">Open your browser and navigate to `http://localhost:3000`.</span></span> <span data-ttu-id="1ac0d-107">Se tudo estiver funcionando, você verá um "Yay!</span><span class="sxs-lookup"><span data-stu-id="1ac0d-107">If everything is working, you will see a "Yay!</span></span> <span data-ttu-id="1ac0d-108">Você está nos trilhos! "</span><span class="sxs-lookup"><span data-stu-id="1ac0d-108">You're on Rails!"</span></span> <span data-ttu-id="1ac0d-109">Mensagem.</span><span class="sxs-lookup"><span data-stu-id="1ac0d-109">message.</span></span> <span data-ttu-id="1ac0d-110">Se você não vir essa mensagem, consulte o [Guia de introdução ao Rails](http://guides.rubyonrails.org/).</span><span class="sxs-lookup"><span data-stu-id="1ac0d-110">If you don't see that message, check the [Rails getting started guide](http://guides.rubyonrails.org/).</span></span>

<span data-ttu-id="1ac0d-111">Antes de prosseguir, instale algumas Gems adicionais que serão usadas posteriormente:</span><span class="sxs-lookup"><span data-stu-id="1ac0d-111">Before moving on, install some additional gems that you will use later:</span></span>

- <span data-ttu-id="1ac0d-112">[omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) para manipulação de entrada e fluxos de token OAuth.</span><span class="sxs-lookup"><span data-stu-id="1ac0d-112">[omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) for handling sign-in and OAuth token flows.</span></span>
- <span data-ttu-id="1ac0d-113">[httparty](https://github.com/jnunemaker/httparty) para fazer chamadas para o Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="1ac0d-113">[httparty](https://github.com/jnunemaker/httparty) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="1ac0d-114">[nokogiri](https://github.com/sparklemotion/nokogiri) para processar corpos de email HTML.</span><span class="sxs-lookup"><span data-stu-id="1ac0d-114">[nokogiri](https://github.com/sparklemotion/nokogiri) to process HTML bodies of email.</span></span>
- <span data-ttu-id="1ac0d-115">[ActiveRecord-session_store](https://github.com/rails/activerecord-session_store) para armazenar sessões no banco de dados.</span><span class="sxs-lookup"><span data-stu-id="1ac0d-115">[activerecord-session_store](https://github.com/rails/activerecord-session_store) for storing sessions in the database.</span></span>

<span data-ttu-id="1ac0d-116">Execute os seguintes comandos em sua CLI.</span><span class="sxs-lookup"><span data-stu-id="1ac0d-116">Run the following commands in your CLI.</span></span>

```Shell
bundle add omniauth-oauth2
bundle add httparty
bundle add nokogiri
bundle add activerecord-session_store
rails generate active_record:session_migration
```

<span data-ttu-id="1ac0d-117">O último comando gera uma saída como a seguinte:</span><span class="sxs-lookup"><span data-stu-id="1ac0d-117">The last command generates output like the following:</span></span>

```Shell
create  db/migrate/20180618172216_add_sessions_table.rb
```

<span data-ttu-id="1ac0d-118">Abra o arquivo que foi criado e localize a linha a seguir.</span><span class="sxs-lookup"><span data-stu-id="1ac0d-118">Open the file that was created and locate the following line.</span></span>

```ruby
class AddSessionsTable < ActiveRecord::Migration
```

<span data-ttu-id="1ac0d-119">Altere essa linha para o seguinte.</span><span class="sxs-lookup"><span data-stu-id="1ac0d-119">Change that line to the following.</span></span>

```ruby
class AddSessionsTable < ActiveRecord::Migration[5.2]
```

> [!NOTE]
> <span data-ttu-id="1ac0d-120">Isso pressupõe que você esteja usando Rails 5.2. x.</span><span class="sxs-lookup"><span data-stu-id="1ac0d-120">This assumes that you are using Rails 5.2.x.</span></span> <span data-ttu-id="1ac0d-121">Se você estiver usando uma versão diferente, substitua `5.2` pela sua versão.</span><span class="sxs-lookup"><span data-stu-id="1ac0d-121">If you are using a different version, replace `5.2` with your version.</span></span>

<span data-ttu-id="1ac0d-122">Salve o arquivo e execute o comando a seguir.</span><span class="sxs-lookup"><span data-stu-id="1ac0d-122">Save the file and run the following command.</span></span>

```Shell
rake db:migrate
```

<span data-ttu-id="1ac0d-123">Por fim, configure Rails para usar o novo repositório de sessão.</span><span class="sxs-lookup"><span data-stu-id="1ac0d-123">Finally, configure Rails to use the new session store.</span></span> <span data-ttu-id="1ac0d-124">Crie um novo arquivo chamado `session_store.rb` no `./config/initializers` diretório e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="1ac0d-124">Create a new file called `session_store.rb` in the `./config/initializers` directory, and add the following code.</span></span>

```ruby
Rails.application.config.session_store :active_record_store, key: '_graph_app_session'
```

## <a name="design-the-app"></a><span data-ttu-id="1ac0d-125">Projetar o aplicativo</span><span class="sxs-lookup"><span data-stu-id="1ac0d-125">Design the app</span></span>

<span data-ttu-id="1ac0d-126">Comece atualizando o layout global do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="1ac0d-126">Start by updating the global layout for the app.</span></span> <span data-ttu-id="1ac0d-127">Abra `./app/views/layouts/application.html.erb` e substitua seu conteúdo pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="1ac0d-127">Open `./app/views/layouts/application.html.erb` and replace its contents with the following.</span></span>

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Ruby Graph Tutorial</title>
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>

    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/css/bootstrap.min.css"
      integrity="sha384-WskhaSGFgHYWDcbwN70/dfYBj47jz9qbsMId/iRN3ewGhXQFZCSftd1LZCfmhktB" crossorigin="anonymous">
    <link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.1.0/css/all.css"
      integrity="sha384-lKuwvrZot6UHsBSfcMvOkWwlCMgc0TaWr+30HWe3a4ltaBwTZhyTEggF5tJv8tbt" crossorigin="anonymous">
    <script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.3/umd/popper.min.js"
      integrity="sha384-ZMP7rVo3mIykV+2+9J3UJ46jBk0WLaUAdn689aCwoqbBJiSnjAK/l8WvCWPIPm49" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/js/bootstrap.min.js"
      integrity="sha384-smHYKdLADwkXOn1EmN1qk/HfnUcbVRZyYmZ4qpPea6sjB/pTJ0euyQp0Mk8ck+5T" crossorigin="anonymous"></script>
    <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload' %>
    <%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
  </head>

  <body>
    <nav class="navbar navbar-expand-md navbar-dark fixed-top bg-dark">
      <div class="container">
        <%= link_to "Ruby Graph Tutorial", root_path, class: "navbar-brand" %>
        <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarCollapse"
          aria-controls="navbarCollapse" aria-expanded="false" aria-label="Toggle navigation">
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarCollapse">
          <ul class="navbar-nav mr-auto">
            <li class="nav-item">
              <%= link_to "Home", root_path, class: "nav-link#{' active' if controller.controller_name == 'home'}" %>
            </li>
            <% if @user_name %>
              <li class="nav-item" data-turbolinks="false">
                <a class="nav-link" href="#">Calendar</a>
              </li>
            <% end %>
          </ul>
          <ul class="navbar-nav justify-content-end">
            <li class="nav-item">
              <a class="nav-link" href="https://developer.microsoft.com/graph/docs/concepts/overview" target="_blank">
                <i class="fas fa-external-link-alt mr-1"></i>Docs
              </a>
            </li>
            <% if @user_name %>
              <li class="nav-item dropdown">
                <a class="nav-link dropdown-toggle" data-toggle="dropdown" href="#" role="button" aria-haspopup="true" aria-expanded="false">
                  <% if @user_avatar %>
                    <img src=<%= @user_avatar %> class="rounded-circle align-self-center mr-2" style="width: 32px;">
                  <% else %>
                    <i class="far fa-user-circle fa-lg rounded-circle align-self-center mr-2" style="width: 32px;"></i>
                  <% end %>
                </a>
                <div class="dropdown-menu dropdown-menu-right">
                  <h5 class="dropdown-item-text mb-0"><%= @user_name %></h5>
                  <p class="dropdown-item-text text-muted mb-0"><%= @user_email %></p>
                  <div class="dropdown-divider"></div>
                  <a href="#" class="dropdown-item">Sign Out</a>
                </div>
              </li>
            <% else %>
              <li class="nav-item">
                <a href="#" class="nav-link">Sign In</a>
              </li>
            <% end %>
          </ul>
        </div>
      </div>
    </nav>
    <main role="main" class="container">
      <% if @errors %>
        <% @errors.each do |error| %>
          <div class="alert alert-danger" role="alert">
            <p class="mb-3"><%= error[:message] %></p>
            <%if error[:debug] %>
              <pre class="alert-pre border bg-light p-2"><code><%= error[:debug] %></code></pre>
            <% end %>
          </div>
        <% end %>
      <% end %>
      <%= yield %>
    </main>
  </body>
</html>
```

<span data-ttu-id="1ac0d-128">Este código adiciona a [inicialização](http://getbootstrap.com/) para estilos simples e a [fonte incrível](https://fontawesome.com/) para alguns ícones simples.</span><span class="sxs-lookup"><span data-stu-id="1ac0d-128">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Font Awesome](https://fontawesome.com/) for some simple icons.</span></span> <span data-ttu-id="1ac0d-129">Também define um layout global com uma barra de navegação.</span><span class="sxs-lookup"><span data-stu-id="1ac0d-129">It also defines a global layout with a nav bar.</span></span>

<span data-ttu-id="1ac0d-130">Agora, `./app/assets/stylesheets/application.css` abra e adicione o seguinte ao final do arquivo.</span><span class="sxs-lookup"><span data-stu-id="1ac0d-130">Now open `./app/assets/stylesheets/application.css` and add the following to the end of the file.</span></span>

```css
body {
  padding-top: 4.5rem;
}

.alert-pre {
  word-wrap: break-word;
  word-break: break-all;
  white-space: pre-wrap;
}
```

<span data-ttu-id="1ac0d-131">Agora substitua a página padrão por uma nova.</span><span class="sxs-lookup"><span data-stu-id="1ac0d-131">Now replace the default page with a new one.</span></span> <span data-ttu-id="1ac0d-132">Gere um controlador de home page com o comando a seguir.</span><span class="sxs-lookup"><span data-stu-id="1ac0d-132">Generate a home page controller with the following command.</span></span>

```Shell
rails generate controller Home index
```

<span data-ttu-id="1ac0d-133">Em seguida, `index` configure a ação `Home` no controlador como a página padrão para o aplicativo.</span><span class="sxs-lookup"><span data-stu-id="1ac0d-133">Then configure the `index` action on the `Home` controller as the default page for the app.</span></span> <span data-ttu-id="1ac0d-134">Abra `./config/routes.rb` e substitua o conteúdo pelo seguinte</span><span class="sxs-lookup"><span data-stu-id="1ac0d-134">Open `./config/routes.rb` and replace the contents with the following</span></span>

```ruby
Rails.application.routes.draw do
  get 'home/index'
  root 'home#index'

  # Add future routes here

end
```

<span data-ttu-id="1ac0d-135">Agora, abra `./app/view/home/index.html.erb` o arquivo e substitua seu conteúdo pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="1ac0d-135">Now open the `./app/view/home/index.html.erb` file and replace its contents with the following.</span></span>

```html
<div class="jumbotron">
  <h1>Ruby Graph Tutorial</h1>
  <p class="lead">This sample app shows how to use the Microsoft Graph API to access Outlook and OneDrive data from Ruby</p>
  <% if @user_name %>
    <h4>Welcome <%= @user_name %>!</h4>
    <p>Use the navigation bar at the top of the page to get started.</p>
  <% else %>
    <a href="#" class="btn btn-primary btn-large">Click here to sign in</a>
  <% end %>
</div>
```

<span data-ttu-id="1ac0d-136">Salve todas as suas alterações e reinicie o servidor.</span><span class="sxs-lookup"><span data-stu-id="1ac0d-136">Save all of your changes and restart the server.</span></span> <span data-ttu-id="1ac0d-137">Agora, o aplicativo deve ser muito diferente.</span><span class="sxs-lookup"><span data-stu-id="1ac0d-137">Now, the app should look very different.</span></span>

![Uma captura de tela da página inicial reprojetada](./images/create-app-01.png)
