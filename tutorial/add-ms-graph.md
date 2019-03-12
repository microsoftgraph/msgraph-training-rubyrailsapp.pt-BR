<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="d6af3-101">Neste exercício, você incorporará o Microsoft Graph no aplicativo.</span><span class="sxs-lookup"><span data-stu-id="d6af3-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="d6af3-102">Para este aplicativo, você usará o gem [httparty](https://github.com/jnunemaker/httparty) para fazer chamadas para o Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="d6af3-102">For this application, you will use the [httparty](https://github.com/jnunemaker/httparty) gem to make calls to Microsoft Graph.</span></span>

## <a name="create-a-graph-helper"></a><span data-ttu-id="d6af3-103">Criar um auxiliar de gráfico</span><span class="sxs-lookup"><span data-stu-id="d6af3-103">Create a Graph helper</span></span>

<span data-ttu-id="d6af3-104">Crie um auxiliar para gerenciar todas as suas chamadas de API.</span><span class="sxs-lookup"><span data-stu-id="d6af3-104">Create a helper to manage all of your API calls.</span></span> <span data-ttu-id="d6af3-105">Execute o comando a seguir em sua CLI para gerar o auxiliar.</span><span class="sxs-lookup"><span data-stu-id="d6af3-105">Run the following command in your CLI to generate the helper.</span></span>

```Shell
rails generate helper Graph
```

<span data-ttu-id="d6af3-106">Abra o `./app/helpers/graph_helper.rb` arquivo recém-criado e substitua o conteúdo pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="d6af3-106">Open the newly created `./app/helpers/graph_helper.rb` file and replace the contents with the following.</span></span>

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

<span data-ttu-id="d6af3-107">Reserve um tempo para revisar o que esse código faz.</span><span class="sxs-lookup"><span data-stu-id="d6af3-107">Take a moment to review what this code does.</span></span> <span data-ttu-id="d6af3-108">Ele faz uma solicitação GET simples através do `httparty` gem para o ponto de extremidade solicitado.</span><span class="sxs-lookup"><span data-stu-id="d6af3-108">It makes a simple GET request via the `httparty` gem to the requested endpoint.</span></span> <span data-ttu-id="d6af3-109">Ele envia o token de acesso no `Authorization` cabeçalho e inclui os parâmetros de consulta que são passados.</span><span class="sxs-lookup"><span data-stu-id="d6af3-109">It sends the access token in the `Authorization` header, and it includes any query parameters that are passed.</span></span>

<span data-ttu-id="d6af3-110">Por exemplo, para usar o `make_api_call` método para fazer um get para `https://graph.microsoft.com/v1.0/me?$select=displayName`, você poderia chamá-lo da seguinte maneira:</span><span class="sxs-lookup"><span data-stu-id="d6af3-110">For example, to use the `make_api_call` method to do a GET to `https://graph.microsoft.com/v1.0/me?$select=displayName`, you could call it like so:</span></span>

```ruby
make_api_call `/v1.0/me`, access_token, { '$select': 'displayName' }
```

<span data-ttu-id="d6af3-111">Você criará isso posteriormente à medida que implementar mais recursos do Microsoft Graph no aplicativo.</span><span class="sxs-lookup"><span data-stu-id="d6af3-111">You'll build on this later as you implement more Microsoft Graph features into the app.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="d6af3-112">Obter eventos de calendário do Outlook</span><span class="sxs-lookup"><span data-stu-id="d6af3-112">Get calendar events from Outlook</span></span>

<span data-ttu-id="d6af3-113">Vamos começar adicionando a capacidade de Exibir eventos no calendário do usuário.</span><span class="sxs-lookup"><span data-stu-id="d6af3-113">Let's start by adding the ability to view events on the user's calendar.</span></span> <span data-ttu-id="d6af3-114">Na sua CLI, execute o seguinte comando para adicionar um novo controlador.</span><span class="sxs-lookup"><span data-stu-id="d6af3-114">In your CLI, run the following command to add a new controller.</span></span>

```Shell
rails generate controller Calendar index
```

<span data-ttu-id="d6af3-115">Agora que temos a rota disponível, atualize o link do **calendário** na barra de navegação `./app/view/layouts/application.html.erb` para usá-lo.</span><span class="sxs-lookup"><span data-stu-id="d6af3-115">Now that we have the route available, update the **Calendar** link in the navbar in `./app/view/layouts/application.html.erb` to use it.</span></span> <span data-ttu-id="d6af3-116">Substitua a linha `<a class="nav-link" href="#">Calendar</a>` pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="d6af3-116">Replace the line `<a class="nav-link" href="#">Calendar</a>` with the following.</span></span>

```html
<%= link_to "Calendar", {:controller => :calendar, :action => :index}, class: "nav-link#{' active' if controller.controller_name == 'calendar'}" %>
```

<span data-ttu-id="d6af3-117">Adicione um novo método ao auxiliar de gráfico para [listar os eventos do usuário](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/api/user_list_events).</span><span class="sxs-lookup"><span data-stu-id="d6af3-117">Add a new method to the Graph helper to [list the user's events](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/api/user_list_events).</span></span> <span data-ttu-id="d6af3-118">Abra `./app/helpers/graph_helper.rb` e adicione o método a seguir ao `GraphHelper` módulo.</span><span class="sxs-lookup"><span data-stu-id="d6af3-118">Open `./app/helpers/graph_helper.rb` and add the following method to the `GraphHelper` module.</span></span>

```ruby
def get_calendar_events(token)
  get_events_url = '/v1.0/me/events'

  query = {
    '$select': 'subject,organizer,start,end',
    '$orderby': 'createdDateTime DESC'
  }

  response = make_api_call get_events_url, token, query

  raise response.parsed_response.to_s || "Request returned #{response.code}" unless response.code == 200
  response.parsed_response['value']
end
```

<span data-ttu-id="d6af3-119">Considere o que esse código está fazendo.</span><span class="sxs-lookup"><span data-stu-id="d6af3-119">Consider what this code is doing.</span></span>

- <span data-ttu-id="d6af3-120">A URL que será chamada é `/v1.0/me/events`.</span><span class="sxs-lookup"><span data-stu-id="d6af3-120">The URL that will be called is `/v1.0/me/events`.</span></span>
- <span data-ttu-id="d6af3-121">O `$select` parâmetro limita os campos retornados para cada evento para apenas aqueles que o nosso modo de exibição realmente usará.</span><span class="sxs-lookup"><span data-stu-id="d6af3-121">The `$select` parameter limits the fields returned for each events to just those our view will actually use.</span></span>
- <span data-ttu-id="d6af3-122">O `$orderby` parâmetro classifica os resultados pela data e hora em que foram criados, com o item mais recente em primeiro lugar.</span><span class="sxs-lookup"><span data-stu-id="d6af3-122">The `$orderby` parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>
- <span data-ttu-id="d6af3-123">Para uma resposta bem-sucedida, ela retorna a matriz de itens contidos na `value` chave.</span><span class="sxs-lookup"><span data-stu-id="d6af3-123">For a successful response, it returns the array of items contained in the `value` key.</span></span>

<span data-ttu-id="d6af3-124">Agora você pode testar isso.</span><span class="sxs-lookup"><span data-stu-id="d6af3-124">Now you can test this.</span></span> <span data-ttu-id="d6af3-125">Abra `./app/controllers/calendar_controller.rb` e atualize a `index` ação para chamar este método e renderizar os resultados.</span><span class="sxs-lookup"><span data-stu-id="d6af3-125">Open `./app/controllers/calendar_controller.rb` and update the `index` action to call this method and render the results.</span></span>

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

<span data-ttu-id="d6af3-126">Reiniciar o servidor.</span><span class="sxs-lookup"><span data-stu-id="d6af3-126">Restart the server.</span></span> <span data-ttu-id="d6af3-127">Entre e clique no link **calendário** na barra de navegação.</span><span class="sxs-lookup"><span data-stu-id="d6af3-127">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="d6af3-128">Se tudo funcionar, você deverá ver um despejo JSON de eventos no calendário do usuário.</span><span class="sxs-lookup"><span data-stu-id="d6af3-128">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="d6af3-129">Exibir os resultados</span><span class="sxs-lookup"><span data-stu-id="d6af3-129">Display the results</span></span>

<span data-ttu-id="d6af3-130">Agora você pode adicionar HTML e CSS para exibir os resultados de forma mais amigável.</span><span class="sxs-lookup"><span data-stu-id="d6af3-130">Now you can add HTML and CSS to display the results in a more user-friendly manner.</span></span>

<span data-ttu-id="d6af3-131">Abra `./app/views/calendar/index.html.erb` e substitua seu conteúdo pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="d6af3-131">Open `./app/views/calendar/index.html.erb` and replace its contents with the following.</span></span>

```html
<h1>Calendar</h1>
<table class="table">
  <thead>
    <tr>
      <th scope="col">Organizer</th>
      <th scope="col">Subject</th>
      <th scope="col">Start</th>
      <th scope="col">End</th>
    </tr>
  </thead>
  <tbody>
    <% @events.each do |event| %>
      <tr>
        <td><%= event['organizer']['emailAddress']['name'] %></td>
        <td><%= event['subject'] %></td>
        <td><%= event['start']['dateTime'].to_time(:utc).localtime.strftime('%-m/%-d/%y %l:%M %p') %></td>
        <td><%= event['end']['dateTime'].to_time(:utc).localtime.strftime('%-m/%-d/%y %l:%M %p') %></td>
      </tr>
    <% end %>
  </tbody>
</table>
```

<span data-ttu-id="d6af3-132">Isso executará um loop através de uma coleção de eventos e adicionará uma linha de tabela para cada um.</span><span class="sxs-lookup"><span data-stu-id="d6af3-132">That will loop through a collection of events and add a table row for each one.</span></span> <span data-ttu-id="d6af3-133">Remova a `render json: @events` linha da `index` ação no `./app/controllers/calendar_controller.rb` e o aplicativo agora deve renderizar uma tabela de eventos.</span><span class="sxs-lookup"><span data-stu-id="d6af3-133">Remove the `render json: @events` line from the `index` action in `./app/controllers/calendar_controller.rb` and the app should now render a table of events.</span></span>

![Uma captura de tela da tabela de eventos](./images/add-msgraph-01.png)