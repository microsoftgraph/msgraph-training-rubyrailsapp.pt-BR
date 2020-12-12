<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="b8926-101">Neste exercício, você incorporará o Microsoft Graph no aplicativo.</span><span class="sxs-lookup"><span data-stu-id="b8926-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="b8926-102">Para este aplicativo, você usará o gem [httparty](https://github.com/jnunemaker/httparty) para fazer chamadas para o Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="b8926-102">For this application, you will use the [httparty](https://github.com/jnunemaker/httparty) gem to make calls to Microsoft Graph.</span></span>

## <a name="create-a-graph-helper"></a><span data-ttu-id="b8926-103">Criar um auxiliar de gráfico</span><span class="sxs-lookup"><span data-stu-id="b8926-103">Create a Graph helper</span></span>

1. <span data-ttu-id="b8926-104">Crie um auxiliar para gerenciar todas as suas chamadas de API.</span><span class="sxs-lookup"><span data-stu-id="b8926-104">Create a helper to manage all of your API calls.</span></span> <span data-ttu-id="b8926-105">Execute o comando a seguir em sua CLI para gerar o auxiliar.</span><span class="sxs-lookup"><span data-stu-id="b8926-105">Run the following command in your CLI to generate the helper.</span></span>

    ```Shell
    rails generate helper Graph
    ```

1. <span data-ttu-id="b8926-106">Abra **./app/helpers/graph_helper. rb** e substitua o conteúdo pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="b8926-106">Open **./app/helpers/graph_helper.rb** and replace the contents with the following.</span></span>

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

<span data-ttu-id="b8926-107">Reserve um tempo para revisar o que esse código faz.</span><span class="sxs-lookup"><span data-stu-id="b8926-107">Take a moment to review what this code does.</span></span> <span data-ttu-id="b8926-108">Ele faz uma solicitação GET ou POST simples através do `httparty` gem para o ponto de extremidade solicitado.</span><span class="sxs-lookup"><span data-stu-id="b8926-108">It makes a simple GET or POST request via the `httparty` gem to the requested endpoint.</span></span> <span data-ttu-id="b8926-109">Ele envia o token de acesso no `Authorization` cabeçalho e inclui os parâmetros de consulta que são passados.</span><span class="sxs-lookup"><span data-stu-id="b8926-109">It sends the access token in the `Authorization` header, and it includes any query parameters that are passed.</span></span>

<span data-ttu-id="b8926-110">Por exemplo, para usar o `make_api_call` método para fazer um get para `https://graph.microsoft.com/v1.0/me?$select=displayName` , você poderia chamá-lo da seguinte maneira:</span><span class="sxs-lookup"><span data-stu-id="b8926-110">For example, to use the `make_api_call` method to do a GET to `https://graph.microsoft.com/v1.0/me?$select=displayName`, you could call it like so:</span></span>

```ruby
make_api_call 'GET', '/v1.0/me', access_token, { '$select': 'displayName' }
```

<span data-ttu-id="b8926-111">Você criará isso posteriormente à medida que implementar mais recursos do Microsoft Graph no aplicativo.</span><span class="sxs-lookup"><span data-stu-id="b8926-111">You'll build on this later as you implement more Microsoft Graph features into the app.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="b8926-112">Obter eventos de calendário do Outlook</span><span class="sxs-lookup"><span data-stu-id="b8926-112">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="b8926-113">Na sua CLI, execute o seguinte comando para adicionar um novo controlador.</span><span class="sxs-lookup"><span data-stu-id="b8926-113">In your CLI, run the following command to add a new controller.</span></span>

    ```Shell
    rails generate controller Calendar index new
    ```

1. <span data-ttu-id="b8926-114">Adicione a nova rota a **./config/routes.rb**.</span><span class="sxs-lookup"><span data-stu-id="b8926-114">Add the new route to **./config/routes.rb**.</span></span>

    ```ruby
    get 'calendar', :to => 'calendar#index'
    ```

1. <span data-ttu-id="b8926-115">Adicione um novo método ao auxiliar de gráfico para [obter um modo de exibição de calendário](https://docs.microsoft.com/graph/api/calendar-list-calendarview?view=graph-rest-1.0).</span><span class="sxs-lookup"><span data-stu-id="b8926-115">Add a new method to the Graph helper to [get a calendar view](https://docs.microsoft.com/graph/api/calendar-list-calendarview?view=graph-rest-1.0).</span></span> <span data-ttu-id="b8926-116">Open **./app/helpers/graph_helper. rb** e adicione o método a seguir ao `GraphHelper` módulo.</span><span class="sxs-lookup"><span data-stu-id="b8926-116">Open **./app/helpers/graph_helper.rb** and add the following method to the `GraphHelper` module.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="GetCalendarSnippet":::

    <span data-ttu-id="b8926-117">Considere o que esse código está fazendo.</span><span class="sxs-lookup"><span data-stu-id="b8926-117">Consider what this code is doing.</span></span>

    - <span data-ttu-id="b8926-118">A URL que será chamada é `/v1.0/me/calendarview` .</span><span class="sxs-lookup"><span data-stu-id="b8926-118">The URL that will be called is `/v1.0/me/calendarview`.</span></span>
        - <span data-ttu-id="b8926-119">O `Prefer: outlook.timezone` cabeçalho faz com que as horas de início e de término nos resultados sejam ajustadas para o fuso horário do usuário.</span><span class="sxs-lookup"><span data-stu-id="b8926-119">The `Prefer: outlook.timezone` header causes the start and end times in the results to be adjusted to the user's time zone.</span></span>
        - <span data-ttu-id="b8926-120">Os `startDateTime` `endDateTime` parâmetros e definem o início e o fim do modo de exibição.</span><span class="sxs-lookup"><span data-stu-id="b8926-120">The `startDateTime` and `endDateTime` parameters set the start and end of the view.</span></span>
        - <span data-ttu-id="b8926-121">O `$select` parâmetro limita os campos retornados para cada evento para apenas aqueles que o modo de exibição realmente usará.</span><span class="sxs-lookup"><span data-stu-id="b8926-121">The `$select` parameter limits the fields returned for each events to just those the view will actually use.</span></span>
        - <span data-ttu-id="b8926-122">O `$orderby` parâmetro classifica os resultados por hora de início.</span><span class="sxs-lookup"><span data-stu-id="b8926-122">The `$orderby` parameter sorts the results by start time.</span></span>
        - <span data-ttu-id="b8926-123">O `$top` parâmetro limita os resultados a 50 eventos.</span><span class="sxs-lookup"><span data-stu-id="b8926-123">The `$top` parameter limits the results to 50 events.</span></span>
    - <span data-ttu-id="b8926-124">Para uma resposta bem-sucedida, ela retorna a matriz de itens contidos na `value` chave.</span><span class="sxs-lookup"><span data-stu-id="b8926-124">For a successful response, it returns the array of items contained in the `value` key.</span></span>

1. <span data-ttu-id="b8926-125">Adicione um novo método ao auxiliar de gráfico para pesquisar um [identificador de fuso horário da IANA](https://www.iana.org/time-zones) com base em um nome de fuso horário do Windows.</span><span class="sxs-lookup"><span data-stu-id="b8926-125">Add a new method to the Graph helper to lookup an [IANA time zone identifier](https://www.iana.org/time-zones) based on a Windows time zone name.</span></span> <span data-ttu-id="b8926-126">Isso é necessário porque o Microsoft Graph pode retornar fusos horários como nomes de fuso horário do Windows e a classe **DateTime** do Ruby requer identificadores de fuso horário da IANA.</span><span class="sxs-lookup"><span data-stu-id="b8926-126">This is necessary because Microsoft Graph can return time zones as Windows time zone names, and the Ruby **DateTime** class requires IANA time zone identifiers.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="ZoneMappingSnippet":::

1. <span data-ttu-id="b8926-127">Abra **./app/controllers/calendar_controller. rb** e substitua todo o seu conteúdo pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="b8926-127">Open **./app/controllers/calendar_controller.rb** and replace its entire contents with the following.</span></span>

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

1. <span data-ttu-id="b8926-128">Reiniciar o servidor.</span><span class="sxs-lookup"><span data-stu-id="b8926-128">Restart the server.</span></span> <span data-ttu-id="b8926-129">Entre e clique no link **calendário** na barra de navegação.</span><span class="sxs-lookup"><span data-stu-id="b8926-129">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="b8926-130">Se tudo funcionar, você deverá ver um despejo JSON de eventos no calendário do usuário.</span><span class="sxs-lookup"><span data-stu-id="b8926-130">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="b8926-131">Exibir os resultados</span><span class="sxs-lookup"><span data-stu-id="b8926-131">Display the results</span></span>

<span data-ttu-id="b8926-132">Agora você pode adicionar HTML para exibir os resultados de forma mais amigável.</span><span class="sxs-lookup"><span data-stu-id="b8926-132">Now you can add HTML to display the results in a more user-friendly manner.</span></span>

1. <span data-ttu-id="b8926-133">Abra **./app/views/calendar/index.html. erb** e substitua seu conteúdo pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="b8926-133">Open **./app/views/calendar/index.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/index.html.erb" id="CalendarSnippet":::

    <span data-ttu-id="b8926-134">Isso executará um loop através de uma coleção de eventos e adicionará uma linha de tabela para cada um.</span><span class="sxs-lookup"><span data-stu-id="b8926-134">That will loop through a collection of events and add a table row for each one.</span></span>

1. <span data-ttu-id="b8926-135">Remova a `render json: @events` linha da `index` ação em **./app/Controllers/calendar_controller. rb**.</span><span class="sxs-lookup"><span data-stu-id="b8926-135">Remove the `render json: @events` line from the `index` action in **./app/controllers/calendar_controller.rb**.</span></span>

1. <span data-ttu-id="b8926-136">Atualize a página e o aplicativo agora deve renderizar uma tabela de eventos.</span><span class="sxs-lookup"><span data-stu-id="b8926-136">Refresh the page and the app should now render a table of events.</span></span>

    ![Uma captura de tela da tabela de eventos](./images/add-msgraph-01.png)
