<!-- markdownlint-disable MD002 MD041 -->

Neste exercício, você estenderá o aplicativo do exercício anterior para oferecer suporte à autenticação com o Azure AD. Isso é necessário para obter o token de acesso OAuth necessário para chamar o Microsoft Graph. Nesta etapa, você integrará o gem [omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) ao aplicativo e criará uma estratégia de omniauth personalizada.

1. Crie um arquivo separado para manter a ID do aplicativo e o segredo. Crie um novo arquivo chamado `oauth_environment_variables.rb` na pasta **./config** e adicione o código a seguir.

    :::code language="ruby" source="../demo/graph-tutorial/config/oauth_environment_variables.rb.example":::

1. Substitua `YOUR_APP_ID_HERE` pela ID do aplicativo do portal de registro do aplicativo e substitua `YOUR_APP_SECRET_HERE` pela senha gerada.

    > [!IMPORTANT]
    > Se você estiver usando o controle de origem como o Git, agora seria uma boa hora para excluir `oauth_environment_variables.rb` o arquivo do controle de origem para evitar vazar inadvertidamente sua ID de aplicativo e sua senha.

1. Abra **./config/Environment.rb** e adicione o código a seguir antes `Rails.application.initialize!` da linha.

    :::code language="ruby" source="../demo/graph-tutorial/config/environment.rb" id="LoadOAuthSettingsSnippet" highlight="4-6":::

## <a name="setup-omniauth"></a>Configurar OmniAuth

Você já instalou o `omniauth-oauth2` Gem, mas para fazê-lo funcionar com os pontos de extremidade do Azure OAuth, é necessário [criar uma estratégia de OAuth2](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy). Esta é uma classe Ruby que define os parâmetros para fazer solicitações OAuth para o provedor do Azure.

1. Crie um novo arquivo chamado `microsoft_graph_auth.rb` na pasta **./lib**' * * e adicione o código a seguir.

    :::code language="ruby" source="../demo/graph-tutorial/lib/microsoft_graph_auth.rb" id="AuthStrategySnippet":::

    Reserve um tempo para revisar o que esse código faz.

    - Ele define o `client_options` para especificar os pontos de extremidade da plataforma de identidade da Microsoft.
    - Especifica que o `scope` parâmetro deve ser enviado durante a fase de autorização.
    - Ele mapeia a `id` Propriedade do usuário como a ID exclusiva para o usuário.
    - Ele usa o token de acesso para recuperar o perfil do usuário do Microsoft Graph para preencher o `raw_info` hash.
    - Ele substitui a URL de retorno de chamada para garantir que ela corresponda ao retorno de chamada registrado no portal de registro do aplicativo.

1. Crie um novo arquivo chamado `omniauth_graph.rb` na pasta **./config/Initializers** e adicione o código a seguir.

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/omniauth_graph.rb" id="ConfigureOmniAuthSnippet":::

    Este código será executado quando o aplicativo for iniciado. Ele carrega o middleware OmniAuth com o `microsoft_graph_auth` provedor, configurado com as variáveis de ambiente definidas em **oauth_environment_variables. rb**.

## <a name="implement-sign-in"></a>Implementar logon

Agora que o middleware OmniAuth está configurado, você pode prosseguir para adicionar a entrada ao aplicativo.

1. Execute o comando a seguir em sua CLI para gerar um controlador para entrar e sair.

    ```Shell
    rails generate controller Auth
    ```

1. Open **./app/controllers/auth_controller. rb**. Adicione um método de retorno de `AuthController` chamada à classe. Este método será chamado pelo middleware OmniAuth depois que o fluxo OAuth estiver concluído.

    ```ruby
    def callback
      # Access the authentication hash for omniauth
      data = request.env['omniauth.auth']

      # Temporary for testing!
      render json: data.to_json
    end
    ```

    Por ora, tudo isso é renderizar o hash fornecido pelo OmniAuth. Você o usará para verificar se a entrada está funcionando antes de prosseguir.

1. Adicione as rotas a **./config/routes.rb**.

    ```ruby
    # Add route for OmniAuth callback
    match '/auth/:provider/callback', to: 'auth#callback', via: [:get, :post]
    ```

1. Inicie o servidor e navegue até `https://localhost:3000`. Clique no botão entrar e você deverá ser redirecionado para `https://login.microsoftonline.com`o. Faça logon com sua conta da Microsoft e concorde com as permissões solicitadas. O navegador redireciona para o aplicativo, mostrando o hash gerado pelo OmniAuth.

    ```json
    {
      "provider": "microsoft_graph_auth",
      "uid": "eb52b3b2-c4ac-4b4f-bacd-d5f7ece55df0",
      "info": {
        "name": null
      },
      "credentials": {
        "token": "eyJ0eXAi...",
        "refresh_token": "OAQABAAA...",
        "expires_at": 1529517383,
        "expires": true
      },
      "extra": {
        "raw_info": {
          "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users/$entity",
          "id": "eb52b3b2-c4ac-4b4f-bacd-d5f7ece55df0",
          "businessPhones": [
            "+1 425 555 0109"
          ],
          "displayName": "Adele Vance",
          "givenName": "Adele",
          "jobTitle": "Retail Manager",
          "mail": "AdeleV@contoso.onmicrosoft.com",
          "mobilePhone": null,
          "officeLocation": "18/2111",
          "preferredLanguage": "en-US",
          "surname": "Vance",
          "userPrincipalName": "AdeleV@contoso.onmicrosoft.com"
        }
      }
    }
    ```

## <a name="storing-the-tokens"></a>Armazenar tokens

Agora que você pode obter tokens, é hora de implementar uma maneira de armazená-los no aplicativo. Como este é um aplicativo de exemplo, por questões de simplicidade, você os armazenará na sessão. Um aplicativo real usaria uma solução de armazenamento segura mais confiável, como um banco de dados.

1. Open **./app/controllers/application_controller. rb**. Adicione o método a seguir à classe `ApplicationController`.

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="SaveInSessionSnippet":::

    O método utiliza o hash OmniAuth como um parâmetro e extrai os bits relevantes de informações e, em seguida, armazena isso na sessão.

1. Adicione funções de assessor à `ApplicationController` classe para recuperar o nome de usuário, o endereço de email e o token de acesso de volta da sessão.

    ```ruby
    def user_name
      session[:user_name]
    end

    def user_email
      session[:user_email]
    end

    def access_token
      session[:graph_token_hash][:token]
    end
    ```

1. Adicione o código a seguir à `ApplicationController` classe que será executada antes que qualquer ação seja processada.

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="BeforeActionSnippet":::

    Este método define as variáveis que o layout (em **Application. html. erb**) usa para mostrar as informações do usuário na barra de navegação. Adicionando-o aqui, não é necessário adicionar esse código em cada ação de controlador único. No entanto, isso também será executado para ações `AuthController`no, o que não é ideal.

1. Adicione o código a seguir à `AuthController` classe em **./app/Controllers/auth_controller. rb** para ignorar a ação anterior.

    ```ruby
    skip_before_action :set_user
    ```

1. Atualize a `callback` função na `AuthController` classe para armazenar os tokens na sessão e redirecionar de volta para a página principal. Substitua a função `callback` existente pelo seguinte.

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/auth_controller.rb" id="CallbackSnippet":::

## <a name="implement-sign-out"></a>Implementar a saída

Antes de testar esse novo recurso, adicione uma maneira de sair.

1. Adicione a ação a seguir à `AuthController` classe.

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/auth_controller.rb" id="SignOutSnippet":::

1. Adicione esta ação a **./config/routes.rb**.

    ```ruby
    get 'auth/signout'
    ```

1. Reinicie o servidor e vá pelo processo de entrada. Você deve terminar de volta na Home Page, mas a interface do usuário deve ser alterada para indicar que você está conectado.

    ![Uma captura de tela da Home Page após entrar](./images/add-aad-auth-01.png)

1. Clique no avatar do usuário no canto superior direito para **acessar o link sair.** Clicar **em sair** redefine a sessão e retorna à Home Page.

    ![Uma captura de tela do menu suspenso com o link sair](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>Atualizando tokens

Se você observar atentamente o hash gerado pelo OmniAuth, observará que há dois tokens no hash: `token` e. `refresh_token` O valor em `token` é o token de acesso, que é enviado no `Authorization` cabeçalho das chamadas de API. Este é o token que permite que o aplicativo acesse o Microsoft Graph em nome do usuário.

No entanto, esse token é de vida curta. O token expira uma hora após sua emissão. É onde o `refresh_token` valor se torna útil. O token de atualização permite que o aplicativo solicite um novo token de acesso sem exigir que o usuário entre novamente. Atualize o código de gerenciamento de token para implementar a atualização de token.

1. Abra **./app/controllers/application_controller. rb** e adicione as seguintes `require` instruções na parte superior:

    ```ruby
    require 'microsoft_graph_auth'
    require 'oauth2'
    ```

1. Adicione o método a seguir à classe `ApplicationController`.

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="RefreshTokensSnippet":::

    Esse método usa o gem [oauth2](https://github.com/oauth-xx/oauth2) (uma dependência do `omniauth-oauth2` gem) para atualizar os tokens e atualiza a sessão.

1. Substitua o método `access_token` atual pelo seguinte.

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="AccessTokenSnippet":::

    Em vez de apenas retornar o token da sessão, ele primeiro verificará se está próximo de expirar. Se for, ele será atualizado antes de retornar o token.
