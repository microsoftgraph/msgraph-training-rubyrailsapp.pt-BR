<!-- markdownlint-disable MD002 MD041 -->

Neste exercício, você incorporará o Microsoft Graph no aplicativo. Para este aplicativo, você usará o gem [httparty](https://github.com/jnunemaker/httparty) para fazer chamadas para o Microsoft Graph.

## <a name="create-a-graph-helper"></a>Criar um auxiliar de gráfico

Crie um auxiliar para gerenciar todas as suas chamadas de API. Execute o comando a seguir em sua CLI para gerar o auxiliar.

```Shell
rails generate helper Graph
```

Abra o `./app/helpers/graph_helper.rb` arquivo recém-criado e substitua o conteúdo pelo seguinte.

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

Reserve um tempo para revisar o que esse código faz. Ele faz uma solicitação GET simples através do `httparty` gem para o ponto de extremidade solicitado. Ele envia o token de acesso no `Authorization` cabeçalho e inclui os parâmetros de consulta que são passados.

Por exemplo, para usar o `make_api_call` método para fazer um get para `https://graph.microsoft.com/v1.0/me?$select=displayName`, você poderia chamá-lo da seguinte maneira:

```ruby
make_api_call `/v1.0/me`, access_token, { '$select': 'displayName' }
```

Você criará isso posteriormente à medida que implementar mais recursos do Microsoft Graph no aplicativo.

## <a name="get-calendar-events-from-outlook"></a>Obter eventos de calendário do Outlook

Vamos começar adicionando a capacidade de Exibir eventos no calendário do usuário. Na sua CLI, execute o seguinte comando para adicionar um novo controlador.

```Shell
rails generate controller Calendar index
```

Agora que temos a rota disponível, atualize o link do **calendário** na barra de navegação `./app/view/layouts/application.html.erb` para usá-lo. Substitua a linha `<a class="nav-link" href="#">Calendar</a>` pelo seguinte.

```html
<%= link_to "Calendar", {:controller => :calendar, :action => :index}, class: "nav-link#{' active' if controller.controller_name == 'calendar'}" %>
```

Adicione um novo método ao auxiliar de gráfico para [listar os eventos do usuário](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/api/user_list_events). Abra `./app/helpers/graph_helper.rb` e adicione o método a seguir ao `GraphHelper` módulo.

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

Considere o que esse código está fazendo.

- A URL que será chamada é `/v1.0/me/events`.
- O `$select` parâmetro limita os campos retornados para cada evento para apenas aqueles que o nosso modo de exibição realmente usará.
- O `$orderby` parâmetro classifica os resultados pela data e hora em que foram criados, com o item mais recente em primeiro lugar.
- Para uma resposta bem-sucedida, ela retorna a matriz de itens contidos na `value` chave.

Agora você pode testar isso. Abra `./app/controllers/calendar_controller.rb` e atualize a `index` ação para chamar este método e renderizar os resultados.

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

Reiniciar o servidor. Entre e clique no link **calendário** na barra de navegação. Se tudo funcionar, você deverá ver um despejo JSON de eventos no calendário do usuário.

## <a name="display-the-results"></a>Exibir os resultados

Agora você pode adicionar HTML e CSS para exibir os resultados de forma mais amigável.

Abra `./app/views/calendar/index.html.erb` e substitua seu conteúdo pelo seguinte.

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

Isso executará um loop através de uma coleção de eventos e adicionará uma linha de tabela para cada um. Remova a `render json: @events` linha da `index` ação no `./app/controllers/calendar_controller.rb` e o aplicativo agora deve renderizar uma tabela de eventos.

![Uma captura de tela da tabela de eventos](./images/add-msgraph-01.png)
