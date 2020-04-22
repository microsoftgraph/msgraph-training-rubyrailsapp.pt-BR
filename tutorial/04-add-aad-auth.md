<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="4a409-101">Neste exercício, você estenderá o aplicativo do exercício anterior para oferecer suporte à autenticação com o Azure AD.</span><span class="sxs-lookup"><span data-stu-id="4a409-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="4a409-102">Isso é necessário para obter o token de acesso OAuth necessário para chamar o Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="4a409-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="4a409-103">Nesta etapa, você integrará o gem [omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) ao aplicativo e criará uma estratégia de omniauth personalizada.</span><span class="sxs-lookup"><span data-stu-id="4a409-103">In this step you will integrate the [omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) gem into the application, and create a custom OmniAuth strategy.</span></span>

1. <span data-ttu-id="4a409-104">Crie um arquivo separado para manter a ID do aplicativo e o segredo.</span><span class="sxs-lookup"><span data-stu-id="4a409-104">Create a separate file to hold your app ID and secret.</span></span> <span data-ttu-id="4a409-105">Crie um novo arquivo chamado `oauth_environment_variables.rb` na pasta **./config** e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="4a409-105">Create a new file called `oauth_environment_variables.rb` in the **./config** folder, and add the following code.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/config/oauth_environment_variables.rb.example":::

1. <span data-ttu-id="4a409-106">Substitua `YOUR_APP_ID_HERE` pela ID do aplicativo do portal de registro do aplicativo e substitua `YOUR_APP_SECRET_HERE` pela senha gerada.</span><span class="sxs-lookup"><span data-stu-id="4a409-106">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_SECRET_HERE` with the password you generated.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="4a409-107">Se você estiver usando o controle de origem como o Git, agora seria uma boa hora para excluir `oauth_environment_variables.rb` o arquivo do controle de origem para evitar vazar inadvertidamente sua ID de aplicativo e sua senha.</span><span class="sxs-lookup"><span data-stu-id="4a409-107">If you're using source control such as git, now would be a good time to exclude the `oauth_environment_variables.rb` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

1. <span data-ttu-id="4a409-108">Abra **./config/Environment.rb** e adicione o código a seguir antes `Rails.application.initialize!` da linha.</span><span class="sxs-lookup"><span data-stu-id="4a409-108">Open **./config/environment.rb** and add the following code before the `Rails.application.initialize!` line.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/config/environment.rb" id="LoadOAuthSettingsSnippet" highlight="4-6":::

## <a name="setup-omniauth"></a><span data-ttu-id="4a409-109">Configurar OmniAuth</span><span class="sxs-lookup"><span data-stu-id="4a409-109">Setup OmniAuth</span></span>

<span data-ttu-id="4a409-110">Você já instalou o `omniauth-oauth2` Gem, mas para fazê-lo funcionar com os pontos de extremidade do Azure OAuth, é necessário [criar uma estratégia de OAuth2](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy).</span><span class="sxs-lookup"><span data-stu-id="4a409-110">You've already installed the `omniauth-oauth2` gem, but in order to make it work with the Azure OAuth endpoints, you need to [create an OAuth2 strategy](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy).</span></span> <span data-ttu-id="4a409-111">Esta é uma classe Ruby que define os parâmetros para fazer solicitações OAuth para o provedor do Azure.</span><span class="sxs-lookup"><span data-stu-id="4a409-111">This is a Ruby class that defines the parameters for making OAuth requests to the Azure provider.</span></span>

1. <span data-ttu-id="4a409-112">Crie um novo arquivo chamado `microsoft_graph_auth.rb` na pasta **./lib**' \* \* e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="4a409-112">Create a new file called `microsoft_graph_auth.rb` in the **./lib**\`\*\* folder, and add the following code.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/lib/microsoft_graph_auth.rb" id="AuthStrategySnippet":::

    <span data-ttu-id="4a409-113">Reserve um tempo para revisar o que esse código faz.</span><span class="sxs-lookup"><span data-stu-id="4a409-113">Take a moment to review what this code does.</span></span>

    - <span data-ttu-id="4a409-114">Ele define o `client_options` para especificar os pontos de extremidade da plataforma de identidade da Microsoft.</span><span class="sxs-lookup"><span data-stu-id="4a409-114">It sets the `client_options` to specify the Microsoft identity platform endpoints.</span></span>
    - <span data-ttu-id="4a409-115">Especifica que o `scope` parâmetro deve ser enviado durante a fase de autorização.</span><span class="sxs-lookup"><span data-stu-id="4a409-115">It specifies that the `scope` parameter should be sent during the authorize phase.</span></span>
    - <span data-ttu-id="4a409-116">Ele mapeia a `id` Propriedade do usuário como a ID exclusiva para o usuário.</span><span class="sxs-lookup"><span data-stu-id="4a409-116">It maps the `id` property of the user as the unique ID for the user.</span></span>
    - <span data-ttu-id="4a409-117">Ele usa o token de acesso para recuperar o perfil do usuário do Microsoft Graph para preencher o `raw_info` hash.</span><span class="sxs-lookup"><span data-stu-id="4a409-117">It uses the access token to retrieve the user's profile from Microsoft Graph to fill in the `raw_info` hash.</span></span>
    - <span data-ttu-id="4a409-118">Ele substitui a URL de retorno de chamada para garantir que ela corresponda ao retorno de chamada registrado no portal de registro do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="4a409-118">It overrides the callback URL to ensure that it matches the registered callback in the app registration portal.</span></span>

1. <span data-ttu-id="4a409-119">Crie um novo arquivo chamado `omniauth_graph.rb` na pasta **./config/Initializers** e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="4a409-119">Create a new file called `omniauth_graph.rb` in the **./config/initializers** folder, and add the following code.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/omniauth_graph.rb" id="ConfigureOmniAuthSnippet":::

    <span data-ttu-id="4a409-120">Este código será executado quando o aplicativo for iniciado.</span><span class="sxs-lookup"><span data-stu-id="4a409-120">This code will execute when the app starts.</span></span> <span data-ttu-id="4a409-121">Ele carrega o middleware OmniAuth com o `microsoft_graph_auth` provedor, configurado com as variáveis de ambiente definidas em **oauth_environment_variables. rb**.</span><span class="sxs-lookup"><span data-stu-id="4a409-121">It loads up the OmniAuth middleware with the `microsoft_graph_auth` provider, configured with the environment variables set in **oauth_environment_variables.rb**.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="4a409-122">Implementar logon</span><span class="sxs-lookup"><span data-stu-id="4a409-122">Implement sign-in</span></span>

<span data-ttu-id="4a409-123">Agora que o middleware OmniAuth está configurado, você pode prosseguir para adicionar a entrada ao aplicativo.</span><span class="sxs-lookup"><span data-stu-id="4a409-123">Now that the OmniAuth middleware is configured, you can move on to adding sign-in to the app.</span></span>

1. <span data-ttu-id="4a409-124">Execute o comando a seguir em sua CLI para gerar um controlador para entrar e sair.</span><span class="sxs-lookup"><span data-stu-id="4a409-124">Run the following command in your CLI to generate a controller for sign-in and sign-out.</span></span>

    ```Shell
    rails generate controller Auth
    ```

1. <span data-ttu-id="4a409-125">Open **./app/controllers/auth_controller. rb**.</span><span class="sxs-lookup"><span data-stu-id="4a409-125">Open **./app/controllers/auth_controller.rb**.</span></span> <span data-ttu-id="4a409-126">Adicione um método de retorno de `AuthController` chamada à classe.</span><span class="sxs-lookup"><span data-stu-id="4a409-126">Add a callback method to the `AuthController` class.</span></span> <span data-ttu-id="4a409-127">Este método será chamado pelo middleware OmniAuth depois que o fluxo OAuth estiver concluído.</span><span class="sxs-lookup"><span data-stu-id="4a409-127">This method will be called by the OmniAuth middleware once the OAuth flow is complete.</span></span>

    ```ruby
    def callback
      # Access the authentication hash for omniauth
      data = request.env['omniauth.auth']

      # Temporary for testing!
      render json: data.to_json
    end
    ```

    <span data-ttu-id="4a409-128">Por ora, tudo isso é renderizar o hash fornecido pelo OmniAuth.</span><span class="sxs-lookup"><span data-stu-id="4a409-128">For now all this does is render the hash provided by OmniAuth.</span></span> <span data-ttu-id="4a409-129">Você o usará para verificar se a entrada está funcionando antes de prosseguir.</span><span class="sxs-lookup"><span data-stu-id="4a409-129">You'll use this to verify that sign-in is working before moving on.</span></span>

1. <span data-ttu-id="4a409-130">Adicione as rotas a **./config/routes.rb**.</span><span class="sxs-lookup"><span data-stu-id="4a409-130">Add the routes to **./config/routes.rb**.</span></span>

    ```ruby
    # Add route for OmniAuth callback
    match '/auth/:provider/callback', to: 'auth#callback', via: [:get, :post]
    ```

1. <span data-ttu-id="4a409-131">Inicie o servidor e navegue até `https://localhost:3000`.</span><span class="sxs-lookup"><span data-stu-id="4a409-131">Start the server and browse to `https://localhost:3000`.</span></span> <span data-ttu-id="4a409-132">Clique no botão entrar e você deverá ser redirecionado para `https://login.microsoftonline.com`o.</span><span class="sxs-lookup"><span data-stu-id="4a409-132">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="4a409-133">Faça logon com sua conta da Microsoft e concorde com as permissões solicitadas.</span><span class="sxs-lookup"><span data-stu-id="4a409-133">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="4a409-134">O navegador redireciona para o aplicativo, mostrando o hash gerado pelo OmniAuth.</span><span class="sxs-lookup"><span data-stu-id="4a409-134">The browser redirects to the app, showing the hash generated by OmniAuth.</span></span>

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

## <a name="storing-the-tokens"></a><span data-ttu-id="4a409-135">Armazenar tokens</span><span class="sxs-lookup"><span data-stu-id="4a409-135">Storing the tokens</span></span>

<span data-ttu-id="4a409-136">Agora que você pode obter tokens, é hora de implementar uma maneira de armazená-los no aplicativo.</span><span class="sxs-lookup"><span data-stu-id="4a409-136">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="4a409-137">Como este é um aplicativo de exemplo, por questões de simplicidade, você os armazenará na sessão.</span><span class="sxs-lookup"><span data-stu-id="4a409-137">Since this is a sample app, for simplicity's sake, you'll store them in the session.</span></span> <span data-ttu-id="4a409-138">Um aplicativo real usaria uma solução de armazenamento segura mais confiável, como um banco de dados.</span><span class="sxs-lookup"><span data-stu-id="4a409-138">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

1. <span data-ttu-id="4a409-139">Open **./app/controllers/application_controller. rb**.</span><span class="sxs-lookup"><span data-stu-id="4a409-139">Open **./app/controllers/application_controller.rb**.</span></span> <span data-ttu-id="4a409-140">Adicione o método a seguir à classe `ApplicationController`.</span><span class="sxs-lookup"><span data-stu-id="4a409-140">Add the following method to the `ApplicationController` class.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="SaveInSessionSnippet":::

    <span data-ttu-id="4a409-141">O método utiliza o hash OmniAuth como um parâmetro e extrai os bits relevantes de informações e, em seguida, armazena isso na sessão.</span><span class="sxs-lookup"><span data-stu-id="4a409-141">The method takes the OmniAuth hash as a parameter and extracts the relevant bits of information, then stores that in the session.</span></span>

1. <span data-ttu-id="4a409-142">Adicione funções de assessor à `ApplicationController` classe para recuperar o nome de usuário, o endereço de email e o token de acesso de volta da sessão.</span><span class="sxs-lookup"><span data-stu-id="4a409-142">Add accessor functions to the `ApplicationController` class to retrieve the user name, email address, and access token back out of the session.</span></span>

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

1. <span data-ttu-id="4a409-143">Adicione o código a seguir à `ApplicationController` classe que será executada antes que qualquer ação seja processada.</span><span class="sxs-lookup"><span data-stu-id="4a409-143">Add the following code to the `ApplicationController` class that will run before any action is processed.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="BeforeActionSnippet":::

    <span data-ttu-id="4a409-144">Este método define as variáveis que o layout (em **Application. html. erb**) usa para mostrar as informações do usuário na barra de navegação.</span><span class="sxs-lookup"><span data-stu-id="4a409-144">This method sets the variables that the layout (in **application.html.erb**) uses to show the user's information in the nav bar.</span></span> <span data-ttu-id="4a409-145">Adicionando-o aqui, não é necessário adicionar esse código em cada ação de controlador único.</span><span class="sxs-lookup"><span data-stu-id="4a409-145">By adding it here, you don't have to add this code in every single controller action.</span></span> <span data-ttu-id="4a409-146">No entanto, isso também será executado para ações `AuthController`no, o que não é ideal.</span><span class="sxs-lookup"><span data-stu-id="4a409-146">However, this will also run for actions in the `AuthController`, which isn't optimal.</span></span>

1. <span data-ttu-id="4a409-147">Adicione o código a seguir à `AuthController` classe em **./app/Controllers/auth_controller. rb** para ignorar a ação anterior.</span><span class="sxs-lookup"><span data-stu-id="4a409-147">Add the following code to the `AuthController` class in **./app/controllers/auth_controller.rb** to skip the before action.</span></span>

    ```ruby
    skip_before_action :set_user
    ```

1. <span data-ttu-id="4a409-148">Atualize a `callback` função na `AuthController` classe para armazenar os tokens na sessão e redirecionar de volta para a página principal.</span><span class="sxs-lookup"><span data-stu-id="4a409-148">Update the `callback` function in the `AuthController` class to store the tokens in the session and redirect back to the main page.</span></span> <span data-ttu-id="4a409-149">Substitua a função `callback` existente pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="4a409-149">Replace the existing `callback` function with the following.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/auth_controller.rb" id="CallbackSnippet":::

## <a name="implement-sign-out"></a><span data-ttu-id="4a409-150">Implementar a saída</span><span class="sxs-lookup"><span data-stu-id="4a409-150">Implement sign-out</span></span>

<span data-ttu-id="4a409-151">Antes de testar esse novo recurso, adicione uma maneira de sair.</span><span class="sxs-lookup"><span data-stu-id="4a409-151">Before you test this new feature, add a way to sign out.</span></span>

1. <span data-ttu-id="4a409-152">Adicione a ação a seguir à `AuthController` classe.</span><span class="sxs-lookup"><span data-stu-id="4a409-152">Add the following action to the `AuthController` class.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/auth_controller.rb" id="SignOutSnippet":::

1. <span data-ttu-id="4a409-153">Adicione esta ação a **./config/routes.rb**.</span><span class="sxs-lookup"><span data-stu-id="4a409-153">Add this action to **./config/routes.rb**.</span></span>

    ```ruby
    get 'auth/signout'
    ```

1. <span data-ttu-id="4a409-154">Reinicie o servidor e vá pelo processo de entrada.</span><span class="sxs-lookup"><span data-stu-id="4a409-154">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="4a409-155">Você deve terminar de volta na Home Page, mas a interface do usuário deve ser alterada para indicar que você está conectado.</span><span class="sxs-lookup"><span data-stu-id="4a409-155">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![Uma captura de tela da Home Page após entrar](./images/add-aad-auth-01.png)

1. <span data-ttu-id="4a409-157">Clique no avatar do usuário no canto superior direito para **acessar o link sair.**</span><span class="sxs-lookup"><span data-stu-id="4a409-157">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="4a409-158">Clicar **em sair** redefine a sessão e retorna à Home Page.</span><span class="sxs-lookup"><span data-stu-id="4a409-158">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![Uma captura de tela do menu suspenso com o link sair](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="4a409-160">Atualizando tokens</span><span class="sxs-lookup"><span data-stu-id="4a409-160">Refreshing tokens</span></span>

<span data-ttu-id="4a409-161">Se você observar atentamente o hash gerado pelo OmniAuth, observará que há dois tokens no hash: `token` e. `refresh_token`</span><span class="sxs-lookup"><span data-stu-id="4a409-161">If you look closely at the hash generated by OmniAuth, you'll notice there are two tokens in the hash: `token` and `refresh_token`.</span></span> <span data-ttu-id="4a409-162">O valor em `token` é o token de acesso, que é enviado no `Authorization` cabeçalho das chamadas de API.</span><span class="sxs-lookup"><span data-stu-id="4a409-162">The value in `token` is the access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="4a409-163">Este é o token que permite que o aplicativo acesse o Microsoft Graph em nome do usuário.</span><span class="sxs-lookup"><span data-stu-id="4a409-163">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="4a409-164">No entanto, esse token é de vida curta.</span><span class="sxs-lookup"><span data-stu-id="4a409-164">However, this token is short-lived.</span></span> <span data-ttu-id="4a409-165">O token expira uma hora após sua emissão.</span><span class="sxs-lookup"><span data-stu-id="4a409-165">The token expires an hour after it is issued.</span></span> <span data-ttu-id="4a409-166">É onde o `refresh_token` valor se torna útil.</span><span class="sxs-lookup"><span data-stu-id="4a409-166">This is where the `refresh_token` value becomes useful.</span></span> <span data-ttu-id="4a409-167">O token de atualização permite que o aplicativo solicite um novo token de acesso sem exigir que o usuário entre novamente.</span><span class="sxs-lookup"><span data-stu-id="4a409-167">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span> <span data-ttu-id="4a409-168">Atualize o código de gerenciamento de token para implementar a atualização de token.</span><span class="sxs-lookup"><span data-stu-id="4a409-168">Update the token management code to implement token refresh.</span></span>

1. <span data-ttu-id="4a409-169">Abra **./app/controllers/application_controller. rb** e adicione as seguintes `require` instruções na parte superior:</span><span class="sxs-lookup"><span data-stu-id="4a409-169">Open **./app/controllers/application_controller.rb** and add the following `require` statements at the top:</span></span>

    ```ruby
    require 'microsoft_graph_auth'
    require 'oauth2'
    ```

1. <span data-ttu-id="4a409-170">Adicione o método a seguir à classe `ApplicationController`.</span><span class="sxs-lookup"><span data-stu-id="4a409-170">Add the following method to the `ApplicationController` class.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="RefreshTokensSnippet":::

    <span data-ttu-id="4a409-171">Esse método usa o gem [oauth2](https://github.com/oauth-xx/oauth2) (uma dependência do `omniauth-oauth2` gem) para atualizar os tokens e atualiza a sessão.</span><span class="sxs-lookup"><span data-stu-id="4a409-171">This method uses the [oauth2](https://github.com/oauth-xx/oauth2) gem (a dependency of the `omniauth-oauth2` gem) to refresh the tokens, and updates the session.</span></span>

1. <span data-ttu-id="4a409-172">Substitua o método `access_token` atual pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="4a409-172">Replace the current `access_token` method with the following.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="AccessTokenSnippet":::

    <span data-ttu-id="4a409-173">Em vez de apenas retornar o token da sessão, ele primeiro verificará se está próximo de expirar.</span><span class="sxs-lookup"><span data-stu-id="4a409-173">Instead of just returning the token from the session, it will first check if it is close to expiration.</span></span> <span data-ttu-id="4a409-174">Se for, ele será atualizado antes de retornar o token.</span><span class="sxs-lookup"><span data-stu-id="4a409-174">If it is, then it will refresh before returning the token.</span></span>
