<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="63658-101">Neste exercício, você incorporará o microsoft Graph no aplicativo.</span><span class="sxs-lookup"><span data-stu-id="63658-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="63658-102">Para este aplicativo, você usará a [gema httparty](https://github.com/jnunemaker/httparty) para fazer chamadas para o Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="63658-102">For this application, you will use the [httparty](https://github.com/jnunemaker/httparty) gem to make calls to Microsoft Graph.</span></span>

## <a name="create-a-graph-helper"></a><span data-ttu-id="63658-103">Criar um Graph auxiliar</span><span class="sxs-lookup"><span data-stu-id="63658-103">Create a Graph helper</span></span>

1. <span data-ttu-id="63658-104">Crie um auxiliar para gerenciar todas as chamadas da API.</span><span class="sxs-lookup"><span data-stu-id="63658-104">Create a helper to manage all of your API calls.</span></span> <span data-ttu-id="63658-105">Execute o seguinte comando em sua CLI para gerar o auxiliar.</span><span class="sxs-lookup"><span data-stu-id="63658-105">Run the following command in your CLI to generate the helper.</span></span>

    ```Shell
    rails generate helper Graph
    ```

1. <span data-ttu-id="63658-106">Abra **./app/helpers/graph_helper.rb** e substitua o conteúdo pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="63658-106">Open **./app/helpers/graph_helper.rb** and replace the contents with the following.</span></span>

    ```ruby
    require 'httparty'

    # Graph API helper methods
    module GraphHelper
      GRAPH_HOST = 'https://graph.microsoft.com'.freeze

      def make_api_call(method, endpoint, token, headers = nil, params = nil, payload = nil)
        headers ||= {}
        headers[:Authorization] = "Bearer #{token}"
        headers[:Accept] = 'application/json'

        params ||= {}

        case method.upcase
        when 'GET'
          HTTParty.get "#{GRAPH_HOST}#{endpoint}",
                       :headers => headers,
                       :query => params
        when 'POST'
          headers['Content-Type'] = 'application/json'
          HTTParty.post "#{GRAPH_HOST}#{endpoint}",
                        :headers => headers,
                        :query => params,
                        :body => payload ? payload.to_json : nil
        else
          raise "HTTP method #{method.upcase} not implemented"
        end
      end
    end
    ```

<span data-ttu-id="63658-107">Aproveite um momento para revisar o que esse código faz.</span><span class="sxs-lookup"><span data-stu-id="63658-107">Take a moment to review what this code does.</span></span> <span data-ttu-id="63658-108">Ele faz uma solicitação GET ou POST simples por meio da `httparty` gema para o ponto de extremidade solicitado.</span><span class="sxs-lookup"><span data-stu-id="63658-108">It makes a simple GET or POST request via the `httparty` gem to the requested endpoint.</span></span> <span data-ttu-id="63658-109">Ele envia o token de acesso no header e inclui todos os parâmetros de consulta `Authorization` que são passados.</span><span class="sxs-lookup"><span data-stu-id="63658-109">It sends the access token in the `Authorization` header, and it includes any query parameters that are passed.</span></span>

<span data-ttu-id="63658-110">Por exemplo, para usar o `make_api_call` método para fazer um GET para , você poderia `https://graph.microsoft.com/v1.0/me?$select=displayName` chamá-lo assim:</span><span class="sxs-lookup"><span data-stu-id="63658-110">For example, to use the `make_api_call` method to do a GET to `https://graph.microsoft.com/v1.0/me?$select=displayName`, you could call it like so:</span></span>

```ruby
make_api_call 'GET', '/v1.0/me', access_token, {}, { '$select': 'displayName' }
```

<span data-ttu-id="63658-111">Você criará isso mais tarde à medida que implementar mais recursos Graph Microsoft no aplicativo.</span><span class="sxs-lookup"><span data-stu-id="63658-111">You'll build on this later as you implement more Microsoft Graph features into the app.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="63658-112">Obtenha eventos de calendário do Outlook</span><span class="sxs-lookup"><span data-stu-id="63658-112">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="63658-113">Em sua CLI, execute o seguinte comando para adicionar um novo controlador.</span><span class="sxs-lookup"><span data-stu-id="63658-113">In your CLI, run the following command to add a new controller.</span></span>

    ```Shell
    rails generate controller Calendar index new
    ```

1. <span data-ttu-id="63658-114">Adicione a nova rota **a ./config/routes.rb**.</span><span class="sxs-lookup"><span data-stu-id="63658-114">Add the new route to **./config/routes.rb**.</span></span>

    ```ruby
    get 'calendar', :to => 'calendar#index'
    ```

1. <span data-ttu-id="63658-115">Adicione um novo método ao Graph auxiliar para [obter um modo de exibição de calendário](https://docs.microsoft.com/graph/api/calendar-list-calendarview?view=graph-rest-1.0).</span><span class="sxs-lookup"><span data-stu-id="63658-115">Add a new method to the Graph helper to [get a calendar view](https://docs.microsoft.com/graph/api/calendar-list-calendarview?view=graph-rest-1.0).</span></span> <span data-ttu-id="63658-116">Abra **./app/helpers/graph_helper.rb e** adicione o método a seguir ao `GraphHelper` módulo.</span><span class="sxs-lookup"><span data-stu-id="63658-116">Open **./app/helpers/graph_helper.rb** and add the following method to the `GraphHelper` module.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="GetCalendarSnippet":::

    <span data-ttu-id="63658-117">Considere o que este código está fazendo.</span><span class="sxs-lookup"><span data-stu-id="63658-117">Consider what this code is doing.</span></span>

    - <span data-ttu-id="63658-118">O URL que será chamado é `/v1.0/me/calendarview`.</span><span class="sxs-lookup"><span data-stu-id="63658-118">The URL that will be called is `/v1.0/me/calendarview`.</span></span>
        - <span data-ttu-id="63658-119">O header faz com que os horários de início e término nos resultados sejam ajustados ao `Prefer: outlook.timezone` fuso horário do usuário.</span><span class="sxs-lookup"><span data-stu-id="63658-119">The `Prefer: outlook.timezone` header causes the start and end times in the results to be adjusted to the user's time zone.</span></span>
        - <span data-ttu-id="63658-120">Os `startDateTime` `endDateTime` parâmetros e definirão o início e o fim do exibição.</span><span class="sxs-lookup"><span data-stu-id="63658-120">The `startDateTime` and `endDateTime` parameters set the start and end of the view.</span></span>
        - <span data-ttu-id="63658-121">O `$select` parâmetro limita os campos retornados para cada evento para apenas aqueles que o exibição realmente usará.</span><span class="sxs-lookup"><span data-stu-id="63658-121">The `$select` parameter limits the fields returned for each events to just those the view will actually use.</span></span>
        - <span data-ttu-id="63658-122">O `$orderby` parâmetro classifica os resultados por hora de início.</span><span class="sxs-lookup"><span data-stu-id="63658-122">The `$orderby` parameter sorts the results by start time.</span></span>
        - <span data-ttu-id="63658-123">O `$top` parâmetro limita os resultados a 50 eventos.</span><span class="sxs-lookup"><span data-stu-id="63658-123">The `$top` parameter limits the results to 50 events.</span></span>
    - <span data-ttu-id="63658-124">Para uma resposta bem-sucedida, ele retorna a matriz de itens contidos na `value` chave.</span><span class="sxs-lookup"><span data-stu-id="63658-124">For a successful response, it returns the array of items contained in the `value` key.</span></span>

1. <span data-ttu-id="63658-125">Adicione um novo método ao Graph auxiliar para procurar um identificador de fuso horário [IANA](https://www.iana.org/time-zones) com base em um Windows de fuso horário.</span><span class="sxs-lookup"><span data-stu-id="63658-125">Add a new method to the Graph helper to lookup an [IANA time zone identifier](https://www.iana.org/time-zones) based on a Windows time zone name.</span></span> <span data-ttu-id="63658-126">Isso é necessário porque a Microsoft Graph pode retornar fusos horário como Windows nomes de fuso horário, e a classe Ruby **DateTime** requer identificadores de fuso horário IANA.</span><span class="sxs-lookup"><span data-stu-id="63658-126">This is necessary because Microsoft Graph can return time zones as Windows time zone names, and the Ruby **DateTime** class requires IANA time zone identifiers.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="ZoneMappingSnippet":::

1. <span data-ttu-id="63658-127">Abra **./app/controllers/calendar_controller.rb** e substitua todo o conteúdo pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="63658-127">Open **./app/controllers/calendar_controller.rb** and replace its entire contents with the following.</span></span>

    ```ruby
    # Calendar controller
    class CalendarController < ApplicationController
      include GraphHelper

      def index
        # Get the IANA identifier of the user's time zone
        time_zone = get_iana_from_windows(user_timezone)

        # Calculate the start and end of week in the user's time zone
        start_datetime = Date.today.beginning_of_week(:sunday).in_time_zone(time_zone).to_time
        end_datetime = start_datetime.advance(:days => 7)

        @events = get_calendar_view access_token, start_datetime, end_datetime, user_timezone || []
        render json: @events
      rescue RuntimeError => e
        @errors = [
          {
            :message => 'Microsoft Graph returned an error getting events.',
            :debug => e
          }
        ]
      end
    end
    ```

1. <span data-ttu-id="63658-128">Reiniciar o servidor.</span><span class="sxs-lookup"><span data-stu-id="63658-128">Restart the server.</span></span> <span data-ttu-id="63658-129">Entre e clique no link **Calendário** na barra de nav.</span><span class="sxs-lookup"><span data-stu-id="63658-129">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="63658-130">Se tudo funcionar, você deverá ver um despejo JSON de eventos no calendário do usuário.</span><span class="sxs-lookup"><span data-stu-id="63658-130">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="63658-131">Exibir os resultados</span><span class="sxs-lookup"><span data-stu-id="63658-131">Display the results</span></span>

<span data-ttu-id="63658-132">Agora você pode adicionar HTML para exibir os resultados de uma maneira mais amigável.</span><span class="sxs-lookup"><span data-stu-id="63658-132">Now you can add HTML to display the results in a more user-friendly manner.</span></span>

1. <span data-ttu-id="63658-133">Abra **./app/views/calendar/index.html.erb e** substitua seu conteúdo pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="63658-133">Open **./app/views/calendar/index.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/index.html.erb" id="CalendarSnippet":::

    <span data-ttu-id="63658-134">Isso percorrerá uma coleção de eventos e adicionará uma linha de tabela para cada um.</span><span class="sxs-lookup"><span data-stu-id="63658-134">That will loop through a collection of events and add a table row for each one.</span></span>

1. <span data-ttu-id="63658-135">Remova a `render json: @events` linha da ação em `index` **./app/controllers/calendar_controller.rb**.</span><span class="sxs-lookup"><span data-stu-id="63658-135">Remove the `render json: @events` line from the `index` action in **./app/controllers/calendar_controller.rb**.</span></span>

1. <span data-ttu-id="63658-136">Atualize a página e o aplicativo deve renderizar uma tabela de eventos.</span><span class="sxs-lookup"><span data-stu-id="63658-136">Refresh the page and the app should now render a table of events.</span></span>

    ![Uma captura de tela da tabela de eventos](./images/add-msgraph-01.png)
