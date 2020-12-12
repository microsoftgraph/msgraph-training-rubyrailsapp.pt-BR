<!-- markdownlint-disable MD002 MD041 -->

Neste exercício, você usará o [Ruby on Rails](https://rubyonrails.org/) para compilar um aplicativo Web.

1. Se você ainda não tiver os Rails instalados, você pode instalá-lo a partir da sua CLI (interface de linha de comando) com o seguinte comando.

    ```Shell
    gem install rails -v 6.0.3.4
    ```

1. Abra sua CLI, navegue até um diretório no qual você tem direitos para criar arquivos e execute o seguinte comando para criar um novo aplicativo Rails.

    ```Shell
    rails new graph-tutorial
    ```

1. Navegue até este novo diretório e digite o seguinte comando para iniciar um servidor Web local.

    ```Shell
    rails server
    ```

1. Abra o navegador e vá até `http://localhost:3000`. Se tudo estiver funcionando, você verá um "Yay! Você está nos trilhos! " Mensagem. Se você não vir essa mensagem, consulte o [Guia de introdução ao Rails](http://guides.rubyonrails.org/).

## <a name="install-gems"></a>Instalar Gems

Antes de prosseguir, instale algumas Gems adicionais que serão usadas posteriormente:

- [omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) para manipulação de entrada e fluxos de token OAuth.
- [omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) para adicionar proteção do CSRF ao omniauth.
- [httparty](https://github.com/jnunemaker/httparty) para fazer chamadas para o Microsoft Graph.
- [ActiveRecord-session_store](https://github.com/rails/activerecord-session_store) para armazenar sessões no banco de dados.

1. Abra **./Gemfile** e adicione as linhas a seguir.

    :::code language="ruby" source="../demo/graph-tutorial/Gemfile" id="GemFileSnippet":::

1. Na sua CLI, execute o seguinte comando.

    ```Shell
    bundle install
    ```

1. Na sua CLI, execute os seguintes comandos para configurar o banco de dados para armazenar sessões.

    ```Shell
    rails generate active_record:session_migration
    rake db:migrate
    ```

1. Crie um novo arquivo chamado `session_store.rb` no diretório **./config/Initializers** e adicione o código a seguir.

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/session_store.rb" id="SessionStoreSnippet":::

## <a name="design-the-app"></a>Projetar o aplicativo

Nesta seção, você criará a interface do usuário básica para o aplicativo.

1. Abra **./app/views/layouts/application.html. erb** e substitua seu conteúdo pelo seguinte.

    :::code language="html" source="../demo/graph-tutorial/app/views/layouts/application.html.erb" id="LayoutSnippet":::

    Este código adiciona [Bootstrap](http://getbootstrap.com/) para estilo simples e [Fabric Core](https://developer.microsoft.com/fluentui#/get-started#fabric-core) para alguns ícones simples. Também define um layout global com uma barra de navegação.

1. Abra **./app/assets/stylesheets/Application.css** e adicione o seguinte ao final do arquivo.

    :::code language="css" source="../demo/graph-tutorial/app/assets/stylesheets/application.css" id="CssSnippet":::

1. Gere um controlador de home page com o comando a seguir.

    ```Shell
    rails generate controller Home index
    ```

1. Configure a `index` ação no `Home` controlador como a página padrão para o aplicativo. Abra **./config/routes.rb** e substitua seu conteúdo pelo seguinte

    ```ruby
    Rails.application.routes.draw do
      get 'home/index'
      root 'home#index'

      # Add future routes here

    end
    ```

1. Abra **./app/view/home/index.html. erb** e substitua seu conteúdo pelo seguinte.

    :::code language="html" source="../demo/graph-tutorial/app/views/home/index.html.erb" id="HomeSnippet":::

1. Adicione um arquivo PNG chamado **no-profile-photo.png** no diretório **./app/assets/images** .

1. Salve todas as suas alterações e reinicie o servidor. Agora, o aplicativo deve ser muito diferente.

    ![Uma captura de tela da página inicial reprojetada](./images/create-app-01.png)
