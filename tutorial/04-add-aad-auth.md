<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="69d05-101">Neste exercício, você estenderá o aplicativo do exercício anterior para oferecer suporte à autenticação com o Azure AD.</span><span class="sxs-lookup"><span data-stu-id="69d05-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="69d05-102">Isso é necessário para obter o token de acesso OAuth necessário para chamar o Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="69d05-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="69d05-103">Nesta etapa, você integrará o gem [omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) ao aplicativo e criará uma estratégia de omniauth personalizada.</span><span class="sxs-lookup"><span data-stu-id="69d05-103">In this step you will integrate the [omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) gem into the application, and create a custom OmniAuth strategy.</span></span>

<span data-ttu-id="69d05-104">Primeiro, crie um arquivo separado para manter a ID do aplicativo e o segredo.</span><span class="sxs-lookup"><span data-stu-id="69d05-104">First, create a separate file to hold your app ID and secret.</span></span> <span data-ttu-id="69d05-105">Crie um novo arquivo chamado `oauth_environment_variables.rb` na `./config` pasta e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="69d05-105">Create a new file called `oauth_environment_variables.rb` in the `./config` folder, and add the following code.</span></span>

```ruby
ENV['AZURE_APP_ID'] = 'YOUR_APP_ID_HERE'
ENV['AZURE_APP_SECRET'] = 'YOUR_APP_SECRET_HERE'
ENV['AZURE_SCOPES'] = 'openid profile email offline_access user.read calendars.read'
```

<span data-ttu-id="69d05-106">Substitua `YOUR_APP_ID_HERE` pela ID do aplicativo do portal de registro do aplicativo e substitua `YOUR_APP_SECRET_HERE` pela senha gerada.</span><span class="sxs-lookup"><span data-stu-id="69d05-106">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_SECRET_HERE` with the password you generated.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="69d05-107">Se você estiver usando o controle de origem como o Git, agora seria uma boa hora para excluir `oauth_environment_variables.rb` o arquivo do controle de origem para evitar vazar inadvertidamente sua ID de aplicativo e sua senha.</span><span class="sxs-lookup"><span data-stu-id="69d05-107">If you're using source control such as git, now would be a good time to exclude the `oauth_environment_variables.rb` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

<span data-ttu-id="69d05-108">Agora, adicione código para carregar esse arquivo se ele estiver presente.</span><span class="sxs-lookup"><span data-stu-id="69d05-108">Now add code to load this file if it's present.</span></span> <span data-ttu-id="69d05-109">Abra o `./config/environment.rb` arquivo e adicione o código a seguir antes `Rails.application.initialize!` da linha.</span><span class="sxs-lookup"><span data-stu-id="69d05-109">Open the `./config/environment.rb` file and add the following code before the `Rails.application.initialize!` line.</span></span>

```ruby
# Load OAuth settings
oauth_environment_variables = File.join(Rails.root, 'config', 'oauth_environment_variables.rb')
load(oauth_environment_variables) if File.exist?(oauth_environment_variables)
```

## <a name="setup-omniauth"></a><span data-ttu-id="69d05-110">Configurar OmniAuth</span><span class="sxs-lookup"><span data-stu-id="69d05-110">Setup OmniAuth</span></span>

<span data-ttu-id="69d05-111">Você já instalou o `omniauth-oauth2` Gem, mas para fazê-lo funcionar com os pontos de extremidade do Azure OAuth, é necessário [criar uma estratégia de OAuth2](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy).</span><span class="sxs-lookup"><span data-stu-id="69d05-111">You've already installed the `omniauth-oauth2` gem, but in order to make it work with the Azure OAuth endpoints, you need to [create an OAuth2 strategy](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy).</span></span> <span data-ttu-id="69d05-112">Esta é uma classe Ruby que define os parâmetros para fazer solicitações OAuth para o provedor do Azure.</span><span class="sxs-lookup"><span data-stu-id="69d05-112">This is a Ruby class that defines the parameters for making OAuth requests to the Azure provider.</span></span>

<span data-ttu-id="69d05-113">Crie um novo arquivo chamado `microsoft_graph_auth.rb` na `./lib` pasta e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="69d05-113">Create a new file called `microsoft_graph_auth.rb` in the `./lib` folder, and add the following code.</span></span>

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

<span data-ttu-id="69d05-114">Reserve um tempo para revisar o que esse código faz.</span><span class="sxs-lookup"><span data-stu-id="69d05-114">Take a moment to review what this code does.</span></span>

- <span data-ttu-id="69d05-115">Ele define o `client_options` para especificar os pontos de extremidade da plataforma de identidade da Microsoft.</span><span class="sxs-lookup"><span data-stu-id="69d05-115">It sets the `client_options` to specify the Microsoft identity platform endpoints.</span></span>
- <span data-ttu-id="69d05-116">Especifica que o `scope` parâmetro deve ser enviado durante a fase de autorização.</span><span class="sxs-lookup"><span data-stu-id="69d05-116">It specifies that the `scope` parameter should be sent during the authorize phase.</span></span>
- <span data-ttu-id="69d05-117">Ele mapeia a `id` Propriedade do usuário como a ID exclusiva para o usuário.</span><span class="sxs-lookup"><span data-stu-id="69d05-117">It maps the `id` property of the user as the unique ID for the user.</span></span>
- <span data-ttu-id="69d05-118">Ele usa o token de acesso para recuperar o perfil do usuário do Microsoft Graph para preencher o `raw_info` hash.</span><span class="sxs-lookup"><span data-stu-id="69d05-118">It uses the access token to retrieve the user's profile from Microsoft Graph to fill in the `raw_info` hash.</span></span>
- <span data-ttu-id="69d05-119">Ele substitui a URL de retorno de chamada para garantir que ela corresponda ao retorno de chamada registrado no portal de registro do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="69d05-119">It overrides the callback URL to ensure that it matches the registered callback in the app registration portal.</span></span>

<span data-ttu-id="69d05-120">Agora que definimos a estratégia, precisamos configurar o OmniAuth para usá-la.</span><span class="sxs-lookup"><span data-stu-id="69d05-120">Now that we've defined the strategy, we need to configure OmniAuth to use it.</span></span> <span data-ttu-id="69d05-121">Crie um novo arquivo chamado `omniauth_graph.rb` na `./config/initializers` pasta e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="69d05-121">Create a new file called `omniauth_graph.rb` in the `./config/initializers` folder, and add the following code.</span></span>

```ruby
require 'microsoft_graph_auth'

Rails.application.config.middleware.use OmniAuth::Builder do
  provider :microsoft_graph_auth,
           ENV['AZURE_APP_ID'],
           ENV['AZURE_APP_SECRET'],
           scope: ENV['AZURE_SCOPES']
end
```

<span data-ttu-id="69d05-122">Este código será executado quando o aplicativo for iniciado.</span><span class="sxs-lookup"><span data-stu-id="69d05-122">This code will execute when the app starts.</span></span> <span data-ttu-id="69d05-123">Ele carrega o middleware OmniAuth com o `microsoft_graph_auth` provedor, configurado com as variáveis de ambiente definidas no. `oauth_environment_variables.rb`</span><span class="sxs-lookup"><span data-stu-id="69d05-123">It loads up the OmniAuth middleware with the `microsoft_graph_auth` provider, configured with the environment variables set in `oauth_environment_variables.rb`.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="69d05-124">Implementar logon</span><span class="sxs-lookup"><span data-stu-id="69d05-124">Implement sign-in</span></span>

<span data-ttu-id="69d05-125">Agora que o middleware OmniAuth está configurado, você pode prosseguir para adicionar a entrada ao aplicativo.</span><span class="sxs-lookup"><span data-stu-id="69d05-125">Now that the OmniAuth middleware is configured, you can move on to adding sign-in to the app.</span></span> <span data-ttu-id="69d05-126">Execute o comando a seguir em sua CLI para gerar um controlador para entrar e sair.</span><span class="sxs-lookup"><span data-stu-id="69d05-126">Run the following command in your CLI to generate a controller for sign-in and sign-out.</span></span>

```Shell
rails generate controller Auth
```

<span data-ttu-id="69d05-127">Abra o arquivo `./app/controllers/auth_controller.rb`.</span><span class="sxs-lookup"><span data-stu-id="69d05-127">Open the `./app/controllers/auth_controller.rb` file.</span></span> <span data-ttu-id="69d05-128">Adicione o método a seguir à classe `AuthController`.</span><span class="sxs-lookup"><span data-stu-id="69d05-128">Add the following method to the `AuthController` class.</span></span>

```ruby
def signin
  redirect_to '/auth/microsoft_graph_auth'
end
```

<span data-ttu-id="69d05-129">Todo esse método é redirecionado para a rota que o OmniAuth espera chamar nossa estratégia personalizada.</span><span class="sxs-lookup"><span data-stu-id="69d05-129">All this method does is redirect to the route that OmniAuth expects to invoke our custom strategy.</span></span>

<span data-ttu-id="69d05-130">Em seguida, adicione um método de retorno `AuthController` de chamada à classe.</span><span class="sxs-lookup"><span data-stu-id="69d05-130">Next, add a callback method to the `AuthController` class.</span></span> <span data-ttu-id="69d05-131">Este método será chamado pelo middleware OmniAuth depois que o fluxo OAuth estiver concluído.</span><span class="sxs-lookup"><span data-stu-id="69d05-131">This method will be called by the OmniAuth middleware once the OAuth flow is complete.</span></span>

```ruby
def callback
  # Access the authentication hash for omniauth
  data = request.env['omniauth.auth']

  # Temporary for testing!
  render json: data.to_json
end
```

<span data-ttu-id="69d05-132">Por ora, tudo isso é renderizar o hash fornecido pelo OmniAuth.</span><span class="sxs-lookup"><span data-stu-id="69d05-132">For now all this does is render the hash provided by OmniAuth.</span></span> <span data-ttu-id="69d05-133">Usaremos isso para verificar se a entrada está funcionando antes de prosseguir.</span><span class="sxs-lookup"><span data-stu-id="69d05-133">We'll use this to verify that our sign-in is working before moving on.</span></span> <span data-ttu-id="69d05-134">Antes de testarmos, precisamos adicionar as rotas ao `./config/routes.rb`.</span><span class="sxs-lookup"><span data-stu-id="69d05-134">Before we test, we need to add the routes to `./config/routes.rb`.</span></span>

```ruby
get 'auth/signin'

# Add route for OmniAuth callback
match '/auth/:provider/callback', to: 'auth#callback', via: [:get, :post]
```

<span data-ttu-id="69d05-135">Agora atualize os modos de exibição para `signin` usar a ação.</span><span class="sxs-lookup"><span data-stu-id="69d05-135">Now update the views to use the `signin` action.</span></span> <span data-ttu-id="69d05-136">Abrir `./app/views/layouts/application.html.erb`.</span><span class="sxs-lookup"><span data-stu-id="69d05-136">Open `./app/views/layouts/application.html.erb`.</span></span> <span data-ttu-id="69d05-137">Substitua a linha `<a href="#" class="nav-link">Sign In</a>` pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="69d05-137">Replace the line `<a href="#" class="nav-link">Sign In</a>` with the following.</span></span>

```html
<%= link_to "Sign In", {:controller => :auth, :action => :signin}, :class => "nav-link" %>
```

<span data-ttu-id="69d05-138">Abra o `./app/views/home/index.html.erb` arquivo e substitua a `<a href="#" class="btn btn-primary btn-large">Click here to sign in</a>` linha pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="69d05-138">Open the `./app/views/home/index.html.erb` file and replace the `<a href="#" class="btn btn-primary btn-large">Click here to sign in</a>` line with the following.</span></span>

```html
<%= link_to "Click here to sign in", {:controller => :auth, :action => :signin}, :class => "btn btn-primary btn-large" %>
```

<span data-ttu-id="69d05-139">Inicie o servidor e navegue até `https://localhost:3000`.</span><span class="sxs-lookup"><span data-stu-id="69d05-139">Start the server and browse to `https://localhost:3000`.</span></span> <span data-ttu-id="69d05-140">Clique no botão entrar e você deverá ser redirecionado para `https://login.microsoftonline.com`o.</span><span class="sxs-lookup"><span data-stu-id="69d05-140">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="69d05-141">Faça logon com sua conta da Microsoft e concorde com as permissões solicitadas.</span><span class="sxs-lookup"><span data-stu-id="69d05-141">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="69d05-142">O navegador redireciona para o aplicativo, mostrando o hash gerado pelo OmniAuth.</span><span class="sxs-lookup"><span data-stu-id="69d05-142">The browser redirects to the app, showing the hash generated by OmniAuth.</span></span>

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

## <a name="storing-the-tokens"></a><span data-ttu-id="69d05-143">Armazenar tokens</span><span class="sxs-lookup"><span data-stu-id="69d05-143">Storing the tokens</span></span>

<span data-ttu-id="69d05-144">Agora que você pode obter tokens, é hora de implementar uma maneira de armazená-los no aplicativo.</span><span class="sxs-lookup"><span data-stu-id="69d05-144">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="69d05-145">Como este é um aplicativo de exemplo, por questões de simplicidade, você os armazenará na sessão.</span><span class="sxs-lookup"><span data-stu-id="69d05-145">Since this is a sample app, for simplicity's sake, you'll store them in the session.</span></span> <span data-ttu-id="69d05-146">Um aplicativo real usaria uma solução de armazenamento segura mais confiável, como um banco de dados.</span><span class="sxs-lookup"><span data-stu-id="69d05-146">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

<span data-ttu-id="69d05-147">Abra o arquivo `./app/controllers/application_controller.rb`.</span><span class="sxs-lookup"><span data-stu-id="69d05-147">Open the `./app/controllers/application_controller.rb` file.</span></span> <span data-ttu-id="69d05-148">Você adicionará todos os métodos de gerenciamento de token aqui.</span><span class="sxs-lookup"><span data-stu-id="69d05-148">You'll add all of our token management methods here.</span></span> <span data-ttu-id="69d05-149">Como todos os outros controladores herdam a `ApplicationController` classe, eles poderão usar esses métodos para acessar os tokens.</span><span class="sxs-lookup"><span data-stu-id="69d05-149">Because all of the other controllers inherit the `ApplicationController` class, they'll be able to use these methods to access the tokens.</span></span>

<span data-ttu-id="69d05-150">Adicione o método a seguir à classe `ApplicationController`.</span><span class="sxs-lookup"><span data-stu-id="69d05-150">Add the following method to the `ApplicationController` class.</span></span> <span data-ttu-id="69d05-151">O método utiliza o hash OmniAuth como um parâmetro e extrai os bits relevantes de informações e, em seguida, armazena isso na sessão.</span><span class="sxs-lookup"><span data-stu-id="69d05-151">The method takes the OmniAuth hash as a parameter and extracts the relevant bits of information, then stores that in the session.</span></span>

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

<span data-ttu-id="69d05-152">Agora, adicione funções de assessor para recuperar o nome de usuário, o endereço de email e o token de acesso de fora da sessão.</span><span class="sxs-lookup"><span data-stu-id="69d05-152">Now add accessor functions to retrieve the user name, email address, and access token back out of the session.</span></span>

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

<span data-ttu-id="69d05-153">Por fim, adicione um código que será executado antes que qualquer ação seja processada.</span><span class="sxs-lookup"><span data-stu-id="69d05-153">Finally, add some code that will run before any action is processed.</span></span>

```ruby
before_action :set_user

def set_user
  @user_name = user_name
  @user_email = user_email
end
```

<span data-ttu-id="69d05-154">Este método define as variáveis que o layout (em `application.html.erb`) usa para mostrar as informações do usuário na barra de navegação.</span><span class="sxs-lookup"><span data-stu-id="69d05-154">This method sets the variables that the layout (in `application.html.erb`) uses to show the user's information in the nav bar.</span></span> <span data-ttu-id="69d05-155">Adicionando-o aqui, não é necessário adicionar esse código em cada ação de controlador único.</span><span class="sxs-lookup"><span data-stu-id="69d05-155">By adding it here, you don't have to add this code in every single controller action.</span></span> <span data-ttu-id="69d05-156">No entanto, isso também será executado para ações `AuthController`no, o que não é ideal.</span><span class="sxs-lookup"><span data-stu-id="69d05-156">However, this will also run for actions in the `AuthController`, which isn't optimal.</span></span> <span data-ttu-id="69d05-157">Adicione o seguinte código à `AuthController` classe no `./app/controllers/auth_controller.rb` para ignorar a ação anterior.</span><span class="sxs-lookup"><span data-stu-id="69d05-157">Add the following code to the `AuthController` class in `./app/controllers/auth_controller.rb` to skip the before action.</span></span>

```ruby
skip_before_action :set_user
```

<span data-ttu-id="69d05-158">Em seguida, atualize `callback` a função na `AuthController` classe para armazenar os tokens na sessão e redirecionar de volta para a página principal.</span><span class="sxs-lookup"><span data-stu-id="69d05-158">Then, update the `callback` function in the `AuthController` class to store the tokens in the session and redirect back to the main page.</span></span> <span data-ttu-id="69d05-159">Substitua a função `callback` existente pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="69d05-159">Replace the existing `callback` function with the following.</span></span>

```ruby
def callback
  # Access the authentication hash for omniauth
  data = request.env['omniauth.auth']

  # Save the data in the session
  save_in_session data

  redirect_to root_url
end
```

## <a name="implement-sign-out"></a><span data-ttu-id="69d05-160">Implementar a saída</span><span class="sxs-lookup"><span data-stu-id="69d05-160">Implement sign-out</span></span>

<span data-ttu-id="69d05-161">Antes de testar esse novo recurso, adicione uma maneira de sair. Adicione a ação a seguir à `AuthController` classe.</span><span class="sxs-lookup"><span data-stu-id="69d05-161">Before you test this new feature, add a way to sign out. Add the following action to the `AuthController` class.</span></span>

```ruby
def signout
  reset_session
  redirect_to root_url
end
```

<span data-ttu-id="69d05-162">Adicione esta ação a `./config/routes.rb`.</span><span class="sxs-lookup"><span data-stu-id="69d05-162">Add this action to `./config/routes.rb`.</span></span>

```ruby
get 'auth/signout'
```

<span data-ttu-id="69d05-163">Agora atualize o modo de exibição para `signout` usar a ação.</span><span class="sxs-lookup"><span data-stu-id="69d05-163">Now update the view to use the `signout` action.</span></span> <span data-ttu-id="69d05-164">Abrir `./app/views/layouts/application.html.erb`.</span><span class="sxs-lookup"><span data-stu-id="69d05-164">Open `./app/views/layouts/application.html.erb`.</span></span> <span data-ttu-id="69d05-165">Substitua a linha `<a href="#" class="dropdown-item">Sign Out</a>` por:</span><span class="sxs-lookup"><span data-stu-id="69d05-165">Replace the line `<a href="#" class="dropdown-item">Sign Out</a>` with:</span></span>

```html
<%= link_to "Sign Out", {:controller => :auth, :action => :signout}, :class => "dropdown-item" %>
```

<span data-ttu-id="69d05-166">Reinicie o servidor e vá pelo processo de entrada.</span><span class="sxs-lookup"><span data-stu-id="69d05-166">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="69d05-167">Você deve terminar de volta na Home Page, mas a interface do usuário deve ser alterada para indicar que você está conectado.</span><span class="sxs-lookup"><span data-stu-id="69d05-167">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

![Uma captura de tela da Home Page após entrar](./images/add-aad-auth-01.png)

<span data-ttu-id="69d05-169">Clique no avatar do usuário no canto superior direito para **acessar o link sair.**</span><span class="sxs-lookup"><span data-stu-id="69d05-169">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="69d05-170">Clicar **em sair** redefine a sessão e retorna à Home Page.</span><span class="sxs-lookup"><span data-stu-id="69d05-170">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

![Uma captura de tela do menu suspenso com o link sair](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="69d05-172">Atualizando tokens</span><span class="sxs-lookup"><span data-stu-id="69d05-172">Refreshing tokens</span></span>

<span data-ttu-id="69d05-173">Se você observar atentamente o hash gerado pelo OmniAuth, observará que há dois tokens no hash: `token` e. `refresh_token`</span><span class="sxs-lookup"><span data-stu-id="69d05-173">If you look closely at the hash generated by OmniAuth, you'll notice there are two tokens in the hash: `token` and `refresh_token`.</span></span> <span data-ttu-id="69d05-174">O valor em `token` é o token de acesso, que é enviado no `Authorization` cabeçalho das chamadas de API.</span><span class="sxs-lookup"><span data-stu-id="69d05-174">The value in `token` is the access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="69d05-175">Este é o token que permite que o aplicativo acesse o Microsoft Graph em nome do usuário.</span><span class="sxs-lookup"><span data-stu-id="69d05-175">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="69d05-176">No entanto, esse token é de vida curta.</span><span class="sxs-lookup"><span data-stu-id="69d05-176">However, this token is short-lived.</span></span> <span data-ttu-id="69d05-177">O token expira uma hora após sua emissão.</span><span class="sxs-lookup"><span data-stu-id="69d05-177">The token expires an hour after it is issued.</span></span> <span data-ttu-id="69d05-178">É onde o `refresh_token` valor se torna útil.</span><span class="sxs-lookup"><span data-stu-id="69d05-178">This is where the `refresh_token` value becomes useful.</span></span> <span data-ttu-id="69d05-179">O token de atualização permite que o aplicativo solicite um novo token de acesso sem exigir que o usuário entre novamente.</span><span class="sxs-lookup"><span data-stu-id="69d05-179">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span> <span data-ttu-id="69d05-180">Atualize o código de gerenciamento de token para implementar a atualização de token.</span><span class="sxs-lookup"><span data-stu-id="69d05-180">Update the token management code to implement token refresh.</span></span>

<span data-ttu-id="69d05-181">Abra `./app/controllers/application_controller.rb` e adicione as seguintes `require` instruções na parte superior:</span><span class="sxs-lookup"><span data-stu-id="69d05-181">Open `./app/controllers/application_controller.rb` and add the following `require` statements at the top:</span></span>

```ruby
require 'microsoft_graph_auth'
require 'oauth2'
```

<span data-ttu-id="69d05-182">Em seguida, adicione o método a `ApplicationController` seguir à classe.</span><span class="sxs-lookup"><span data-stu-id="69d05-182">Then add the following method to the `ApplicationController` class.</span></span>

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

<span data-ttu-id="69d05-183">Esse método usa o gem [oauth2](https://github.com/oauth-xx/oauth2) (uma dependência do `omniauth-oauth2` gem) para atualizar os tokens e atualiza a sessão.</span><span class="sxs-lookup"><span data-stu-id="69d05-183">This method uses the [oauth2](https://github.com/oauth-xx/oauth2) gem (a dependency of the `omniauth-oauth2` gem) to refresh the tokens, and updates the session.</span></span>

<span data-ttu-id="69d05-184">Agora coloque esse método a ser usado.</span><span class="sxs-lookup"><span data-stu-id="69d05-184">Now put this method to use.</span></span> <span data-ttu-id="69d05-185">Para fazer isso, torne o `access_token` acessador `ApplicationController` na classe um bit mais inteligente.</span><span class="sxs-lookup"><span data-stu-id="69d05-185">To do that, make the `access_token` accessor in the `ApplicationController` class a bit smarter.</span></span> <span data-ttu-id="69d05-186">Em vez de apenas retornar o token da sessão, ele primeiro verificará se está próximo de expirar.</span><span class="sxs-lookup"><span data-stu-id="69d05-186">Instead of just returning the token from the session, it will first check if it is close to expiration.</span></span> <span data-ttu-id="69d05-187">Se for, ele será atualizado antes de retornar o token.</span><span class="sxs-lookup"><span data-stu-id="69d05-187">If it is, then it will refresh before returning the token.</span></span> <span data-ttu-id="69d05-188">Substitua o método `access_token` atual pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="69d05-188">Replace the current `access_token` method with the following.</span></span>

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
