<!-- markdownlint-disable MD002 MD041 -->

Neste exercício, você incorporará o Microsoft Graph no aplicativo. Para este aplicativo, você usará o gem [httparty](https://github.com/jnunemaker/httparty) para fazer chamadas para o Microsoft Graph.

## <a name="create-a-graph-helper"></a>Criar um auxiliar de gráfico

1. Crie um auxiliar para gerenciar todas as suas chamadas de API. Execute o comando a seguir em sua CLI para gerar o auxiliar.

    ```Shell
    rails generate helper Graph
    ```

1. Abra **./app/helpers/graph_helper. rb** e substitua o conteúdo pelo seguinte.

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

1. Na sua CLI, execute o seguinte comando para adicionar um novo controlador.

    ```Shell
    rails generate controller Calendar index
    ```

1. Adicione a nova rota a **./config/routes.rb**.

    ```ruby
    get 'calendar', to: 'calendar#index'
    ```

1. Adicione um novo método ao auxiliar de gráfico para [listar os eventos do usuário](/graph/api/user-list-events?view=graph-rest-1.0). Open **./app/helpers/graph_helper. rb** e adicione o método a seguir ao `GraphHelper` módulo.

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="GetCalendarSnippet":::

    Considere o que esse código está fazendo.

    - A URL que será chamada é `/v1.0/me/events`.
    - O `$select` parâmetro limita os campos retornados para cada evento para apenas aqueles que o nosso modo de exibição realmente usará.
    - O `$orderby` parâmetro classifica os resultados pela data e hora em que foram criados, com o item mais recente em primeiro lugar.
    - Para uma resposta bem-sucedida, ela retorna a matriz de itens contidos na `value` chave.

1. Abra **./app/controllers/calendar_controller. rb** e substitua todo o seu conteúdo pelo seguinte.

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

1. Reiniciar o servidor. Entre e clique no link **calendário** na barra de navegação. Se tudo funcionar, você deverá ver um despejo JSON de eventos no calendário do usuário.

## <a name="display-the-results"></a>Exibir os resultados

Agora você pode adicionar HTML para exibir os resultados de forma mais amigável.

1. Abra **./app/views/Calendar/index.html.erb** e substitua seu conteúdo pelo seguinte.

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/index.html.erb" id="CalendarSnippet":::

    Isso executará um loop através de uma coleção de eventos e adicionará uma linha de tabela para cada um.

1. Remova a `render json: @events` linha da `index` ação em **./app/Controllers/calendar_controller. rb**.

1. Atualize a página e o aplicativo agora deve renderizar uma tabela de eventos.

    ![Uma captura de tela da tabela de eventos](./images/add-msgraph-01.png)
