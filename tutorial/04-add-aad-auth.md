<!-- markdownlint-disable MD002 MD041 -->

Neste exercício, você estenderá o aplicativo do exercício anterior para oferecer suporte à autenticação com o Azure AD. Isso é necessário para obter o token de acesso OAuth necessário para chamar o Microsoft Graph. Nesta etapa, você integrará o gem [omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) ao aplicativo e criará uma estratégia de omniauth personalizada.

Primeiro, crie um arquivo separado para manter a ID do aplicativo e o segredo. Crie um novo arquivo chamado `oauth_environment_variables.rb` na `./config` pasta e adicione o código a seguir.

```ruby
ENV['AZURE_APP_ID'] = 'YOUR_APP_ID_HERE'
ENV['AZURE_APP_SECRET'] = 'YOUR_APP_SECRET_HERE'
ENV['AZURE_SCOPES'] = 'openid profile email offline_access user.read calendars.read'
```

Substitua `YOUR_APP_ID_HERE` pela ID do aplicativo do portal de registro do aplicativo e substitua `YOUR_APP_SECRET_HERE` pela senha gerada.

> [!IMPORTANT]
> Se você estiver usando o controle de origem como o Git, agora seria uma boa hora para excluir `oauth_environment_variables.rb` o arquivo do controle de origem para evitar vazar inadvertidamente sua ID de aplicativo e sua senha.

Agora, adicione código para carregar esse arquivo se ele estiver presente. Abra o `./config/environment.rb` arquivo e adicione o código a seguir antes `Rails.application.initialize!` da linha.

```ruby
# Load OAuth settings
oauth_environment_variables = File.join(Rails.root, 'config', 'oauth_environment_variables.rb')
load(oauth_environment_variables) if File.exist?(oauth_environment_variables)
```

## <a name="setup-omniauth"></a>Configurar OmniAuth

Você já instalou o `omniauth-oauth2` Gem, mas para fazê-lo funcionar com os pontos de extremidade do Azure OAuth, é necessário [criar uma estratégia de OAuth2](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy). Esta é uma classe Ruby que define os parâmetros para fazer solicitações OAuth para o provedor do Azure.

Crie um novo arquivo chamado `microsoft_graph_auth.rb` na `./lib` pasta e adicione o código a seguir.

```ruby
require 'omniauth-oauth2'

module OmniAuth
  module Strategies
    # Implements an OmniAuth strategy to get a Microsoft Graph
    # compatible token from Azure AD
    class MicrosoftGraphAuth < OmniAuth::Strategies::OAuth2
      option :name, :microsoft_graph_auth

      DEFAULT_SCOPE = 'openid email profile User.Read'.freeze

      # Configure the Microsoft identity platform endpoints
      option  :client_options,
              site:          'https://login.microsoftonline.com',
              authorize_url: '/common/oauth2/v2.0/authorize',
              token_url:     '/common/oauth2/v2.0/token'

      # Send the scope parameter during authorize
      option :authorize_options, [:scope]

      # Unique ID for the user is the id field
      uid { raw_info['id'] }

      # Get additional information after token is retrieved
      extra do
        {
          'raw_info' => raw_info
        }
      end

      def raw_info
        # Get user profile information from the /me endpoint
        @raw_info ||= access_token.get('https://graph.microsoft.com/v1.0/me').parsed
      end

      def authorize_params
        super.tap do |params|
          params['scope'.to_sym] = request.params['scope'] if request.params['scope']
          params[:scope] ||= DEFAULT_SCOPE
        end
      end

      # Override callback URL
      # OmniAuth by default passes the entire URL of the callback, including
      # query parameters. Azure fails validation because that doesn't match the
      # registered callback.
      def callback_url
        options[:redirect_uri] || (full_host + script_name + callback_path)
      end
    end
  end
end
```

Reserve um tempo para revisar o que esse código faz.

- Ele define o `client_options` para especificar os pontos de extremidade da plataforma de identidade da Microsoft.
- Especifica que o `scope` parâmetro deve ser enviado durante a fase de autorização.
- Ele mapeia a `id` Propriedade do usuário como a ID exclusiva para o usuário.
- Ele usa o token de acesso para recuperar o perfil do usuário do Microsoft Graph para preencher o `raw_info` hash.
- Ele substitui a URL de retorno de chamada para garantir que ela corresponda ao retorno de chamada registrado no portal de registro do aplicativo.

Agora que definimos a estratégia, precisamos configurar o OmniAuth para usá-la. Crie um novo arquivo chamado `omniauth_graph.rb` na `./config/initializers` pasta e adicione o código a seguir.

```ruby
require 'microsoft_graph_auth'

Rails.application.config.middleware.use OmniAuth::Builder do
  provider :microsoft_graph_auth,
           ENV['AZURE_APP_ID'],
           ENV['AZURE_APP_SECRET'],
           scope: ENV['AZURE_SCOPES']
end
```

Este código será executado quando o aplicativo for iniciado. Ele carrega o middleware OmniAuth com o `microsoft_graph_auth` provedor, configurado com as variáveis de ambiente definidas no. `oauth_environment_variables.rb`

## <a name="implement-sign-in"></a>Implementar logon

Agora que o middleware OmniAuth está configurado, você pode prosseguir para adicionar a entrada ao aplicativo. Execute o comando a seguir em sua CLI para gerar um controlador para entrar e sair.

```Shell
rails generate controller Auth
```

Abra o arquivo `./app/controllers/auth_controller.rb`. Adicione o método a seguir à classe `AuthController`.

```ruby
def signin
  redirect_to '/auth/microsoft_graph_auth'
end
```

Todo esse método é redirecionado para a rota que o OmniAuth espera chamar nossa estratégia personalizada.

Em seguida, adicione um método de retorno `AuthController` de chamada à classe. Este método será chamado pelo middleware OmniAuth depois que o fluxo OAuth estiver concluído.

```ruby
def callback
  # Access the authentication hash for omniauth
  data = request.env['omniauth.auth']

  # Temporary for testing!
  render json: data.to_json
end
```

Por ora, tudo isso é renderizar o hash fornecido pelo OmniAuth. Usaremos isso para verificar se a entrada está funcionando antes de prosseguir. Antes de testarmos, precisamos adicionar as rotas ao `./config/routes.rb`.

```ruby
get 'auth/signin'

# Add route for OmniAuth callback
match '/auth/:provider/callback', to: 'auth#callback', via: [:get, :post]
```

Agora atualize os modos de exibição para `signin` usar a ação. Abrir `./app/views/layouts/application.html.erb`. Substitua a linha `<a href="#" class="nav-link">Sign In</a>` pelo seguinte.

```html
<%= link_to "Sign In", {:controller => :auth, :action => :signin}, :class => "nav-link" %>
```

Abra o `./app/views/home/index.html.erb` arquivo e substitua a `<a href="#" class="btn btn-primary btn-large">Click here to sign in</a>` linha pelo seguinte.

```html
<%= link_to "Click here to sign in", {:controller => :auth, :action => :signin}, :class => "btn btn-primary btn-large" %>
```

Inicie o servidor e navegue até `https://localhost:3000`. Clique no botão entrar e você deverá ser redirecionado para `https://login.microsoftonline.com`o. Faça logon com sua conta da Microsoft e concorde com as permissões solicitadas. O navegador redireciona para o aplicativo, mostrando o hash gerado pelo OmniAuth.

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

Abra o arquivo `./app/controllers/application_controller.rb`. Você adicionará todos os métodos de gerenciamento de token aqui. Como todos os outros controladores herdam a `ApplicationController` classe, eles poderão usar esses métodos para acessar os tokens.

Adicione o método a seguir à classe `ApplicationController`. O método utiliza o hash OmniAuth como um parâmetro e extrai os bits relevantes de informações e, em seguida, armazena isso na sessão.

```ruby
def save_in_session(auth_hash)
  # Save the token info
  session[:graph_token_hash] = auth_hash.dig(:credentials)
  # Save the user's display name
  session[:user_name] = auth_hash.dig(:extra, :raw_info, :displayName)
  # Save the user's email address
  # Use the mail field first. If that's empty, fall back on
  # userPrincipalName
  session[:user_email] = auth_hash.dig(:extra, :raw_info, :mail) ||
                         auth_hash.dig(:extra, :raw_info, :userPrincipalName)
end
```

Agora, adicione funções de assessor para recuperar o nome de usuário, o endereço de email e o token de acesso de fora da sessão.

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

Por fim, adicione um código que será executado antes que qualquer ação seja processada.

```ruby
before_action :set_user

def set_user
  @user_name = user_name
  @user_email = user_email
end
```

Este método define as variáveis que o layout (em `application.html.erb`) usa para mostrar as informações do usuário na barra de navegação. Adicionando-o aqui, não é necessário adicionar esse código em cada ação de controlador único. No entanto, isso também será executado para ações `AuthController`no, o que não é ideal. Adicione o seguinte código à `AuthController` classe no `./app/controllers/auth_controller.rb` para ignorar a ação anterior.

```ruby
skip_before_action :set_user
```

Em seguida, atualize `callback` a função na `AuthController` classe para armazenar os tokens na sessão e redirecionar de volta para a página principal. Substitua a função `callback` existente pelo seguinte.

```ruby
def callback
  # Access the authentication hash for omniauth
  data = request.env['omniauth.auth']

  # Save the data in the session
  save_in_session data

  redirect_to root_url
end
```

## <a name="implement-sign-out"></a>Implementar a saída

Antes de testar esse novo recurso, adicione uma maneira de sair. Adicione a ação a seguir à `AuthController` classe.

```ruby
def signout
  reset_session
  redirect_to root_url
end
```

Adicione esta ação a `./config/routes.rb`.

```ruby
get 'auth/signout'
```

Agora atualize o modo de exibição para `signout` usar a ação. Abrir `./app/views/layouts/application.html.erb`. Substitua a linha `<a href="#" class="dropdown-item">Sign Out</a>` por:

```html
<%= link_to "Sign Out", {:controller => :auth, :action => :signout}, :class => "dropdown-item" %>
```

Reinicie o servidor e vá pelo processo de entrada. Você deve terminar de volta na Home Page, mas a interface do usuário deve ser alterada para indicar que você está conectado.

![Uma captura de tela da Home Page após entrar](./images/add-aad-auth-01.png)

Clique no avatar do usuário no canto superior direito para **acessar o link sair.** Clicar **em sair** redefine a sessão e retorna à Home Page.

![Uma captura de tela do menu suspenso com o link sair](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>Atualizando tokens

Se você observar atentamente o hash gerado pelo OmniAuth, observará que há dois tokens no hash: `token` e. `refresh_token` O valor em `token` é o token de acesso, que é enviado no `Authorization` cabeçalho das chamadas de API. Este é o token que permite que o aplicativo acesse o Microsoft Graph em nome do usuário.

No entanto, esse token é de vida curta. O token expira uma hora após sua emissão. É onde o `refresh_token` valor se torna útil. O token de atualização permite que o aplicativo solicite um novo token de acesso sem exigir que o usuário entre novamente. Atualize o código de gerenciamento de token para implementar a atualização de token.

Abra `./app/controllers/application_controller.rb` e adicione as seguintes `require` instruções na parte superior:

```ruby
require 'microsoft_graph_auth'
require 'oauth2'
```

Em seguida, adicione o método a `ApplicationController` seguir à classe.

```ruby
def refresh_tokens(token_hash)
  oauth_strategy = OmniAuth::Strategies::MicrosoftGraphAuth.new(
    nil, ENV['AZURE_APP_ID'], ENV['AZURE_APP_SECRET']
  )

  token = OAuth2::AccessToken.new(
    oauth_strategy.client, token_hash[:token],
    refresh_token: token_hash[:refresh_token]
  )

  # Refresh the tokens
  new_tokens = token.refresh!.to_hash.slice(:access_token, :refresh_token, :expires_at)

  # Rename token key
  new_tokens[:token] = new_tokens.delete :access_token

  # Store the new hash
  session[:graph_token_hash] = new_tokens
end
```

Esse método usa o gem [oauth2](https://github.com/oauth-xx/oauth2) (uma dependência do `omniauth-oauth2` gem) para atualizar os tokens e atualiza a sessão.

Agora coloque esse método a ser usado. Para fazer isso, torne o `access_token` acessador `ApplicationController` na classe um bit mais inteligente. Em vez de apenas retornar o token da sessão, ele primeiro verificará se está próximo de expirar. Se for, ele será atualizado antes de retornar o token. Substitua o método `access_token` atual pelo seguinte.

```ruby
def access_token
  token_hash = session[:graph_token_hash]

  # Get the expiry time - 5 minutes
  expiry = Time.at(token_hash[:expires_at] - 300)

  if Time.now > expiry
    # Token expired, refresh
    new_hash = refresh_tokens token_hash
    new_hash[:token]
  else
    token_hash[:token]
  end
end
```
