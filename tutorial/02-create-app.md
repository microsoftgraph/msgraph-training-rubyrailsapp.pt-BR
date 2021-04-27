<!-- markdownlint-disable MD002 MD041 -->

Neste exercício, você usará [Ruby on Rails](https://rubyonrails.org/) para criar um aplicativo Web.

1. Se você ainda não tiver Rails instalado, poderá instalá-lo a partir de sua interface de linha de comando (CLI) com o seguinte comando.

    ```Shell
    gem install rails -v 6.1.3.1
    ```

1. Abra sua CLI, navegue até um diretório onde você tem direitos para criar arquivos e execute o seguinte comando para criar um novo aplicativo Rails.

    ```Shell
    rails new graph-tutorial
    ```

1. Navegue até esse novo diretório e insira o seguinte comando para iniciar um servidor Web local.

    ```Shell
    rails server
    ```

1. Abra o navegador e vá até `http://localhost:3000`. Se tudo estiver funcionando, você verá um "Yay! Você está em Rails!" Mensagem. Se você não vir essa mensagem, verifique o guia [Rails getting started](http://guides.rubyonrails.org/).

## <a name="install-gems"></a>Instalar gemas

Antes de continuar, instale algumas gemas adicionais que você usará posteriormente:

- [omniauth-oauth2 para](https://github.com/omniauth/omniauth-oauth2) manipular fluxos de tokens OAuth e de entrada.
- [omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) para adicionar proteção CSRF ao OmniAuth.
- [httparty](https://github.com/jnunemaker/httparty) para fazer chamadas para o Microsoft Graph.
- [activerecord-session_store](https://github.com/rails/activerecord-session_store) para armazenar sessões no banco de dados.

1. Abra **./Gemfile** e adicione as seguintes linhas.

    :::code language="ruby" source="../demo/graph-tutorial/Gemfile" id="GemFileSnippet":::

1. Em sua CLI, execute o seguinte comando.

    ```Shell
    bundle install
    ```

1. Em sua CLI, execute os seguintes comandos para configurar o banco de dados para armazenar sessões.

    ```Shell
    rails generate active_record:session_migration
    rake db:migrate
    ```

1. Crie um novo arquivo chamado `session_store.rb` no **diretório ./config/initializers** e adicione o código a seguir.

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/session_store.rb" id="SessionStoreSnippet":::

## <a name="design-the-app"></a>Design do aplicativo

Nesta seção, você criará a interface do usuário básica do aplicativo.

1. Abra **./app/views/layouts/application.html.erb e** substitua seu conteúdo pelo seguinte.

    :::code language="html" source="../demo/graph-tutorial/app/views/layouts/application.html.erb" id="LayoutSnippet":::

    Este código adiciona [Bootstrap](http://getbootstrap.com/) para estilo simples e [Fabric Core](https://developer.microsoft.com/fluentui#/get-started#fabric-core) para alguns ícones simples. Ele também define um layout global com uma barra de nav.

1. Abra **./app/assets/stylesheets/application.css** e adicione o seguinte ao final do arquivo.

    :::code language="css" source="../demo/graph-tutorial/app/assets/stylesheets/application.css" id="CssSnippet":::

1. Gere um controlador de home page com o seguinte comando.

    ```Shell
    rails generate controller Home index
    ```

1. Configure a `index` ação no controlador como a página padrão do `Home` aplicativo. Abra **./config/routes.rb e** substitua seu conteúdo pelo seguinte

    ```ruby
    Rails.application.routes.draw do
      get 'home/index'
      root 'home#index'

      # Add future routes here

    end
    ```

1. Abra **./app/view/home/index.html.erb e** substitua seu conteúdo pelo seguinte.

    :::code language="html" source="../demo/graph-tutorial/app/views/home/index.html.erb" id="HomeSnippet":::

1. Adicione um arquivo PNG chamado **no-profile-photo.png** no **diretório ./app/assets/images.**

1. Salve todas as suas alterações e reinicie o aplicativo. Agora, o aplicativo deve ter uma aparência muito diferente.

    ![Captura de tela da home page](./images/create-app-01.png)
