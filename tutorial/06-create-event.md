<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="39297-101">Nesta seção, você adicionará a capacidade de criar eventos no calendário do usuário.</span><span class="sxs-lookup"><span data-stu-id="39297-101">In this section you will add the ability to create events on the user's calendar.</span></span>

1. <span data-ttu-id="39297-102">Open **./app/helpers/graph_helper. rb** e adicione o método a seguir à classe do **gráfico** .</span><span class="sxs-lookup"><span data-stu-id="39297-102">Open **./app/helpers/graph_helper.rb** and add the following method to the **Graph** class.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="CreateEventSnippet":::

1. <span data-ttu-id="39297-103">Abra **./app/controllers/calendar_controller** e adicione a rota a seguir à classe **CalendarController** .</span><span class="sxs-lookup"><span data-stu-id="39297-103">Open **./app/controllers/calendar_controller** and add the following route to the **CalendarController** class.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/calendar_controller.rb" id="CreateEventRouteSnippet":::

1. <span data-ttu-id="39297-104">Abra **./config/routes.rb** e adicione a nova rota.</span><span class="sxs-lookup"><span data-stu-id="39297-104">Open **./config/routes.rb** and add the new route.</span></span>

    ```ruby
    post 'calendar/new', :to => 'calendar#create'
    ```

1. <span data-ttu-id="39297-105">Abra **./app/views/calendar/new.html. erb** e substitua seu conteúdo pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="39297-105">Open **./app/views/calendar/new.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/new.html.erb" id="NewEventFormSnippet":::

1. <span data-ttu-id="39297-106">Salve suas alterações e atualize o aplicativo.</span><span class="sxs-lookup"><span data-stu-id="39297-106">Save your changes and refresh the app.</span></span> <span data-ttu-id="39297-107">Na página **calendário** , selecione o botão **novo evento** .</span><span class="sxs-lookup"><span data-stu-id="39297-107">On the **Calendar** page, select the **New event** button.</span></span> <span data-ttu-id="39297-108">Preencha o formulário e selecione **criar** para criar um novo evento.</span><span class="sxs-lookup"><span data-stu-id="39297-108">Fill in the form and select **Create** to create a new event.</span></span>

    ![Uma captura de tela do novo formulário de evento](images/create-event-01.png)
