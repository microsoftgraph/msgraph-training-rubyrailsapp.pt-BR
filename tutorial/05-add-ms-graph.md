<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="37c8a-101">Neste exercício, você incorporará o Microsoft Graph no aplicativo.</span><span class="sxs-lookup"><span data-stu-id="37c8a-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="37c8a-102">Para este aplicativo, você usará o gem [httparty](https://github.com/jnunemaker/httparty) para fazer chamadas para o Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="37c8a-102">For this application, you will use the [httparty](https://github.com/jnunemaker/httparty) gem to make calls to Microsoft Graph.</span></span>

## <a name="create-a-graph-helper"></a><span data-ttu-id="37c8a-103">Criar um auxiliar de gráfico</span><span class="sxs-lookup"><span data-stu-id="37c8a-103">Create a Graph helper</span></span>

1. <span data-ttu-id="37c8a-104">Crie um auxiliar para gerenciar todas as suas chamadas de API.</span><span class="sxs-lookup"><span data-stu-id="37c8a-104">Create a helper to manage all of your API calls.</span></span> <span data-ttu-id="37c8a-105">Execute o comando a seguir em sua CLI para gerar o auxiliar.</span><span class="sxs-lookup"><span data-stu-id="37c8a-105">Run the following command in your CLI to generate the helper.</span></span>

    ```Shell
    rails generate helper Graph
    ```

1. <span data-ttu-id="37c8a-106">Abra **./app/helpers/graph_helper. rb** e substitua o conteúdo pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="37c8a-106">Open **./app/helpers/graph_helper.rb** and replace the contents with the following.</span></span>

    ```ruby
    require 'httparty'

    # Graph API helper methods
    module GraphHelper
      GRAPH_HOST = 'https://graph.microsoft.com'.freeze

      def make_api_call(endpoint, token, params = nil)
        headers = {
          Authorization: "Bearer #{token}"
        }

        query = params || {}

        HTTParty.get "#{GRAPH_HOST}#{endpoint}",
                     headers: headers,
                     query: query
      end
    end
    ```

<span data-ttu-id="37c8a-107">Reserve um tempo para revisar o que esse código faz.</span><span class="sxs-lookup"><span data-stu-id="37c8a-107">Take a moment to review what this code does.</span></span> <span data-ttu-id="37c8a-108">Ele faz uma solicitação GET simples através do `httparty` gem para o ponto de extremidade solicitado.</span><span class="sxs-lookup"><span data-stu-id="37c8a-108">It makes a simple GET request via the `httparty` gem to the requested endpoint.</span></span> <span data-ttu-id="37c8a-109">Ele envia o token de acesso no `Authorization` cabeçalho e inclui os parâmetros de consulta que são passados.</span><span class="sxs-lookup"><span data-stu-id="37c8a-109">It sends the access token in the `Authorization` header, and it includes any query parameters that are passed.</span></span>

<span data-ttu-id="37c8a-110">Por exemplo, para usar o `make_api_call` método para fazer um get para `https://graph.microsoft.com/v1.0/me?$select=displayName`, você poderia chamá-lo da seguinte maneira:</span><span class="sxs-lookup"><span data-stu-id="37c8a-110">For example, to use the `make_api_call` method to do a GET to `https://graph.microsoft.com/v1.0/me?$select=displayName`, you could call it like so:</span></span>

```ruby
make_api_call `/v1.0/me`, access_token, { '$select': 'displayName' }
```

<span data-ttu-id="37c8a-111">Você criará isso posteriormente à medida que implementar mais recursos do Microsoft Graph no aplicativo.</span><span class="sxs-lookup"><span data-stu-id="37c8a-111">You'll build on this later as you implement more Microsoft Graph features into the app.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="37c8a-112">Obter eventos de calendário do Outlook</span><span class="sxs-lookup"><span data-stu-id="37c8a-112">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="37c8a-113">Na sua CLI, execute o seguinte comando para adicionar um novo controlador.</span><span class="sxs-lookup"><span data-stu-id="37c8a-113">In your CLI, run the following command to add a new controller.</span></span>

    ```Shell
    rails generate controller Calendar index
    ```

1. <span data-ttu-id="37c8a-114">Adicione a nova rota a **./config/routes.rb**.</span><span class="sxs-lookup"><span data-stu-id="37c8a-114">Add the new route to **./config/routes.rb**.</span></span>

    ```ruby
    get 'calendar', to: 'calendar#index'
    ```

1. <span data-ttu-id="37c8a-115">Adicione um novo método ao auxiliar de gráfico para [listar os eventos do usuário](/graph/api/user-list-events?view=graph-rest-1.0).</span><span class="sxs-lookup"><span data-stu-id="37c8a-115">Add a new method to the Graph helper to [list the user's events](/graph/api/user-list-events?view=graph-rest-1.0).</span></span> <span data-ttu-id="37c8a-116">Open **./app/helpers/graph_helper. rb** e adicione o método a seguir ao `GraphHelper` módulo.</span><span class="sxs-lookup"><span data-stu-id="37c8a-116">Open **./app/helpers/graph_helper.rb** and add the following method to the `GraphHelper` module.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="GetCalendarSnippet":::

    <span data-ttu-id="37c8a-117">Considere o que esse código está fazendo.</span><span class="sxs-lookup"><span data-stu-id="37c8a-117">Consider what this code is doing.</span></span>

    - <span data-ttu-id="37c8a-118">A URL que será chamada é `/v1.0/me/events`.</span><span class="sxs-lookup"><span data-stu-id="37c8a-118">The URL that will be called is `/v1.0/me/events`.</span></span>
    - <span data-ttu-id="37c8a-119">O `$select` parâmetro limita os campos retornados para cada evento para apenas aqueles que o nosso modo de exibição realmente usará.</span><span class="sxs-lookup"><span data-stu-id="37c8a-119">The `$select` parameter limits the fields returned for each events to just those our view will actually use.</span></span>
    - <span data-ttu-id="37c8a-120">O `$orderby` parâmetro classifica os resultados pela data e hora em que foram criados, com o item mais recente em primeiro lugar.</span><span class="sxs-lookup"><span data-stu-id="37c8a-120">The `$orderby` parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>
    - <span data-ttu-id="37c8a-121">Para uma resposta bem-sucedida, ela retorna a matriz de itens contidos na `value` chave.</span><span class="sxs-lookup"><span data-stu-id="37c8a-121">For a successful response, it returns the array of items contained in the `value` key.</span></span>

1. <span data-ttu-id="37c8a-122">Abra **./app/controllers/calendar_controller. rb** e substitua todo o seu conteúdo pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="37c8a-122">Open **./app/controllers/calendar_controller.rb** and replace its entire contents with the following.</span></span>

    ```ruby
    # Calendar controller
    class CalendarController < ApplicationController
      include GraphHelper

      def index
        @events = get_calendar_events access_token || []
        render json: @events
      rescue RuntimeError => e
        @errors = [
          {
            message: 'Microsoft Graph returned an error getting events.',
            debug: e
          }
        ]
      end
    end
    ```

1. <span data-ttu-id="37c8a-123">Reiniciar o servidor.</span><span class="sxs-lookup"><span data-stu-id="37c8a-123">Restart the server.</span></span> <span data-ttu-id="37c8a-124">Entre e clique no link **calendário** na barra de navegação.</span><span class="sxs-lookup"><span data-stu-id="37c8a-124">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="37c8a-125">Se tudo funcionar, você deverá ver um despejo JSON de eventos no calendário do usuário.</span><span class="sxs-lookup"><span data-stu-id="37c8a-125">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="37c8a-126">Exibir os resultados</span><span class="sxs-lookup"><span data-stu-id="37c8a-126">Display the results</span></span>

<span data-ttu-id="37c8a-127">Agora você pode adicionar HTML para exibir os resultados de forma mais amigável.</span><span class="sxs-lookup"><span data-stu-id="37c8a-127">Now you can add HTML to display the results in a more user-friendly manner.</span></span>

1. <span data-ttu-id="37c8a-128">Abra **./app/views/Calendar/index.html.erb** e substitua seu conteúdo pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="37c8a-128">Open **./app/views/calendar/index.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/index.html.erb" id="CalendarSnippet":::

    <span data-ttu-id="37c8a-129">Isso executará um loop através de uma coleção de eventos e adicionará uma linha de tabela para cada um.</span><span class="sxs-lookup"><span data-stu-id="37c8a-129">That will loop through a collection of events and add a table row for each one.</span></span>

1. <span data-ttu-id="37c8a-130">Remova a `render json: @events` linha da `index` ação em **./app/Controllers/calendar_controller. rb**.</span><span class="sxs-lookup"><span data-stu-id="37c8a-130">Remove the `render json: @events` line from the `index` action in **./app/controllers/calendar_controller.rb**.</span></span>

1. <span data-ttu-id="37c8a-131">Atualize a página e o aplicativo agora deve renderizar uma tabela de eventos.</span><span class="sxs-lookup"><span data-stu-id="37c8a-131">Refresh the page and the app should now render a table of events.</span></span>

    ![Uma captura de tela da tabela de eventos](./images/add-msgraph-01.png)
