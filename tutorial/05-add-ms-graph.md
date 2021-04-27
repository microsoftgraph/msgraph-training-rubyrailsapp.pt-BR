<!-- markdownlint-disable MD002 MD041 -->

Neste exercício, você incorporará o microsoft Graph no aplicativo. Para este aplicativo, você usará a [gema httparty](https://github.com/jnunemaker/httparty) para fazer chamadas para o Microsoft Graph.

## <a name="create-a-graph-helper"></a>Criar um Graph auxiliar

1. Crie um auxiliar para gerenciar todas as chamadas da API. Execute o seguinte comando em sua CLI para gerar o auxiliar.

    ```Shell
    rails generate helper Graph
    ```

1. Abra **./app/helpers/graph_helper.rb** e substitua o conteúdo pelo seguinte.

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

Aproveite um momento para revisar o que esse código faz. Ele faz uma solicitação GET ou POST simples por meio da `httparty` gema para o ponto de extremidade solicitado. Ele envia o token de acesso no header e inclui todos os parâmetros de consulta `Authorization` que são passados.

Por exemplo, para usar o `make_api_call` método para fazer um GET para , você poderia `https://graph.microsoft.com/v1.0/me?$select=displayName` chamá-lo assim:

```ruby
make_api_call 'GET', '/v1.0/me', access_token, {}, { '$select': 'displayName' }
```

Você criará isso mais tarde à medida que implementar mais recursos Graph Microsoft no aplicativo.

## <a name="get-calendar-events-from-outlook"></a>Obtenha eventos de calendário do Outlook

1. Em sua CLI, execute o seguinte comando para adicionar um novo controlador.

    ```Shell
    rails generate controller Calendar index new
    ```

1. Adicione a nova rota **a ./config/routes.rb**.

    ```ruby
    get 'calendar', :to => 'calendar#index'
    ```

1. Adicione um novo método ao Graph auxiliar para [obter um modo de exibição de calendário](https://docs.microsoft.com/graph/api/calendar-list-calendarview?view=graph-rest-1.0). Abra **./app/helpers/graph_helper.rb e** adicione o método a seguir ao `GraphHelper` módulo.

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="GetCalendarSnippet":::

    Considere o que este código está fazendo.

    - O URL que será chamado é `/v1.0/me/calendarview`.
        - O header faz com que os horários de início e término nos resultados sejam ajustados ao `Prefer: outlook.timezone` fuso horário do usuário.
        - Os `startDateTime` `endDateTime` parâmetros e definirão o início e o fim do exibição.
        - O `$select` parâmetro limita os campos retornados para cada evento para apenas aqueles que o exibição realmente usará.
        - O `$orderby` parâmetro classifica os resultados por hora de início.
        - O `$top` parâmetro limita os resultados a 50 eventos.
    - Para uma resposta bem-sucedida, ele retorna a matriz de itens contidos na `value` chave.

1. Adicione um novo método ao Graph auxiliar para procurar um identificador de fuso horário [IANA](https://www.iana.org/time-zones) com base em um Windows de fuso horário. Isso é necessário porque a Microsoft Graph pode retornar fusos horário como Windows nomes de fuso horário, e a classe Ruby **DateTime** requer identificadores de fuso horário IANA.

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="ZoneMappingSnippet":::

1. Abra **./app/controllers/calendar_controller.rb** e substitua todo o conteúdo pelo seguinte.

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

1. Reiniciar o servidor. Entre e clique no link **Calendário** na barra de nav. Se tudo funcionar, você deverá ver um despejo JSON de eventos no calendário do usuário.

## <a name="display-the-results"></a>Exibir os resultados

Agora você pode adicionar HTML para exibir os resultados de uma maneira mais amigável.

1. Abra **./app/views/calendar/index.html.erb e** substitua seu conteúdo pelo seguinte.

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/index.html.erb" id="CalendarSnippet":::

    Isso percorrerá uma coleção de eventos e adicionará uma linha de tabela para cada um.

1. Remova a `render json: @events` linha da ação em `index` **./app/controllers/calendar_controller.rb**.

1. Atualize a página e o aplicativo deve renderizar uma tabela de eventos.

    ![Uma captura de tela da tabela de eventos](./images/add-msgraph-01.png)
