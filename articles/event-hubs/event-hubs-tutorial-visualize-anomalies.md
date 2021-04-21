---
title: Центры событий Azure. Визуализации аномальных данных в событиях реального времени
description: Руководство по визуализации аномалий данных в событиях реального времени, отправляемых в Центры событий Microsoft Azure
ms.topic: tutorial
ms.date: 06/23/2020
ms.openlocfilehash: bb7d8da2498005b8b2e1183a836d9385f3d31e5c
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107783761"
---
# <a name="tutorial-visualize-data-anomalies-in-real-time-events-sent-to-azure-event-hubs"></a>Руководство по визуализации аномальных данных в событиях реального времени, отправляемых в Центры событий Azure

Центры событий Azure позволяют вам использовать Azure Stream Analytics для проверки входящих данных и извлечения аномалий, которые затем можно визуализировать в Power BI. Предположим, что у вас есть тысячи устройств, непрерывно отправляющих в реальном времени данные в концентратор событий и генерирующих миллионы событий в секунду. Как вы проверите такой большой объем данных на наличие аномалий или ошибок? Что если устройства, к примеру, отправляют транзакции с кредитными картами, и необходимо отслеживать все случаи, когда в течение 5-секундного интервала выполняется несколько транзакций из разных стран и регионов? Это может произойти, когда кто-то крадет кредитные карты и использует их для покупки товаров по всему миру в одно и то же время. 

В этом руководстве мы смоделируем эту ситуацию. Мы запустим приложение, которое создает и отправляет транзакции с кредитными картами в концентратор событий. Затем мы в реальном времени считаем поток данных в решении Azure Stream Analytics, которое отделит допустимые транзакции от недопустимых. После этого мы визуально выделим транзакции, отмеченные как недопустимые, в Power BI.

В этом руководстве описано следующее:
> [!div class="checklist"]
> * Создание пространства имен в Центрах событий
> * Создание концентратора событий
> * Запуск приложения, которое отправляет транзакции с кредитными картами
> * Настройка задания Stream Analytics для обработки этих транзакций
> * Настройка визуализации Power BI для отображения результатов

Для работы с этим руководством вам потребуется подписка Azure. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись][], прежде чем начать работу.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Установить [Visual Studio](https://www.visualstudio.com/). 
- Вам понадобится учетная запись Power BI для анализа выходных данных из задания Stream Analytics. Доступна [бесплатная пробная версия Power BI](https://app.powerbi.com/signupredirect?pbi_source=web).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="set-up-resources"></a>Настройка ресурсов

Для этого руководства вам понадобится пространство имен Центров событий и концентратор событий. Создать эти ресурсы можно в интерфейсе командной строки (CLI) Azure или с помощью Azure PowerShell. Используйте те же группу ресурсов и расположение для всех ресурсов. Затем в конце, удалив группу ресурсов, можно удалить все данные за один шаг.

В следующих разделах описано, как выполнить эти необходимые шаги. Следуя инструкциям по командной строке *или* PowerShell, выполните следующие действия:

1. Создайте [группу ресурсов](../azure-resource-manager/management/overview.md). 

2. Создайте пространство имен Центров событий. 

3. Создайте концентратор событий.

> [!NOTE]
> Каждый сценарий содержит набор переменных, который потребуется позже в этом руководстве. В него входит имя группы ресурсов ($resourceGroup), пространство имен концентратора событий ( **$eventHubNamespace**) и имя концентратора событий ( **$eventHubName**). В дальнейшем эти переменные приведены в статье со знаком доллара ($), чтобы было понятно, что они заданы в сценарии.

<!-- some day they will approve the tab control; 
  When that happens, put CLI and PSH in tabs. -->

### <a name="set-up-your-resources-using-azure-cli"></a>Настройка ресурсов с помощью Azure CLI

Скопируйте и вставьте этот сценарий в Cloud Shell. Предполагая, что вы уже вошли в систему, Cloud Shell запустит сценарий, выполняя одну строку за другой.

К переменным, которые должны быть глобально уникальными, добавляется `$RANDOM`. При запуске сценария и назначении переменных в конец фиксированной строки добавляется сгенерированная случайным образом числовая строка, чтобы сделать значение уникальным.

```azurecli-interactive
# Set the values for location and resource group name.
location=westus
resourceGroup=ContosoResourcesEH

# Create the resource group to be used
#   for all the resources for this tutorial.
az group create --name $resourceGroup \
    --location $location

# The Event Hubs namespace name must be globally unique, so add a random number to the end.
eventHubNamespace=ContosoEHNamespace$RANDOM
echo "Event Hub Namespace = " $eventHubNamespace

# Create the Event Hubs namespace.
az eventhubs namespace create --resource-group $resourceGroup \
   --name $eventHubNamespace \
   --location $location \
   --sku Standard

# The event hub name must be globally unique, so add a random number to the end.
eventHubName=ContosoEHhub$RANDOM
echo "event hub name = " $eventHubName

# Create the event hub.
az eventhubs eventhub create --resource-group $resourceGroup \
    --namespace-name $eventHubNamespace \
    --name $eventHubName \
    --message-retention 3 \
    --partition-count 2

# Get the connection string that authenticates the app with the Event Hubs service.
connectionString=$(az eventhubs namespace authorization-rule keys list \
   --resource-group $resourceGroup \
   --namespace-name $eventHubNamespace \
   --name RootManageSharedAccessKey \
   --query primaryConnectionString \
   --output tsv)
echo "Connection string = " $connectionString 
```

### <a name="set-up-your-resources-using-azure-powershell"></a>Настройка ресурсов с помощью Azure PowerShell

Скопируйте и вставьте этот сценарий в Cloud Shell. Предполагая, что вы уже вошли в систему, Cloud Shell запустит сценарий, выполняя одну строку за другой.

К переменным, которые должны быть глобально уникальными, добавляется `$(Get-Random)`. При запуске сценария и назначении переменных в конец фиксированной строки добавляется сгенерированная случайным образом числовая строка, чтобы сделать значение уникальным.

```azurepowershell-interactive
# Log in to Azure account.
Login-AzAccount

# Set the values for the location and resource group.
$location = "West US"
$resourceGroup = "ContosoResourcesEH"

# Create the resource group to be used  
#   for all resources for this tutorial.
New-AzResourceGroup -Name $resourceGroup -Location $location

# The Event Hubs namespace name must be globally unique, so add a random number to the end.
$eventHubNamespace = "contosoEHNamespace$(Get-Random)"
Write-Host "Event Hub Namespace is " $eventHubNamespace

# The event hub name must be globally unique, so add a random number to the end.
$eventHubName = "contosoEHhub$(Get-Random)"
Write-Host "Event hub Name is " $eventHubName

# Create the Event Hubs namespace.
New-AzEventHubNamespace -ResourceGroupName $resourceGroup `
     -NamespaceName $eventHubNamespace `
     -Location $location

# Create the event hub.
$yourEventHub = New-AzEventHub -ResourceGroupName $resourceGroup `
    -NamespaceName $eventHubNamespace `
    -Name $eventHubName `
    -MessageRetentionInDays 3 `
    -PartitionCount 2

# Get the event hub key, and retrieve the connection string from that object.
# You need this to run the app that sends test messages to the event hub.
$eventHubKey = Get-AzEventHubKey -ResourceGroupName $resourceGroup `
    -Namespace $eventHubNamespace `
    -AuthorizationRuleName RootManageSharedAccessKey

# Save this value somewhere local for later use.
Write-Host "Connection string is " $eventHubKey.PrimaryConnectionString
```

## <a name="run-app-to-produce-test-event-data"></a>Запуск приложения для формирования тестовых данных событий

[Примеры для Центров событий в GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet) включают приложение Детектора аномалий, которое генерирует тестовые данные. Оно имитирует работу с кредитными картами, записывая в концентратор событий транзакции, включая время от времени несколько транзакций по одной и той же карте в нескольких местах одновременно, таким образом, чтобы они отмечались как аномалии. Чтобы запустить это приложение, сделайте следующее: 

1. Скачайте [примеры Центров событий Azure](https://github.com/Azure/azure-event-hubs/archive/master.zip) из GitHub и распакуйте их на локальной системе.
2. Перейдите к папке **\azure-event-hubs-master\samples\DotNet\\** . 
3. Перейдите к папке **Azure.Messaging.EventHubs\AnomalyDetector\\** и дважды щелкните **AnomalyDetector.sln**, чтобы открыть решение в Visual Studio. 

    Чтобы использовать старую версию примера, для которой применяется старый пакет Microsoft.Azure.EventHubs, откройте решение из папки **Microsoft.Azure.EventHubs\AnomalyDetector**. 
3. Откройте файл Program.cs и замените **строку подключения Центров событий** строкой подключения, сохраненной при запуске сценария. 
4. Замените **имя концентратора событий** именем вашего концентратора. Нажмите клавишу F5 для запуска приложения. Оно начнет отправлять события в концентратор, пока не отправит 1000 событий. В некоторых случаях для извлечения данных приложение должно быть в запущенном состоянии. Эти случаи указаны в следующих инструкциях, где это необходимо.

## <a name="set-up-azure-stream-analytics"></a>Настройка Azure Stream Analytics

Теперь мы можем начать передачу данных в концентратор событий. Чтобы использовать эти данные в визуализации Power BI, сначала создайте задание Stream Analytics для получения данных, которые затем будут передаваться в визуализацию.

### <a name="create-the-stream-analytics-job"></a>Создание задания Stream Analytics

1. На портале Azure щелкните **Создать ресурс**. Введите **stream analytics** в поле поиска и нажмите клавишу **ВВОД**. Выберите **Задание Stream Analytics**. В панели задания Stream Analytics щелкните **Создать**. 

2. Введите для задания следующие данные.

   **Имя задания.** **contosoEHjob**. Это поле содержит имя задания, и оно должно быть глобально уникальным.

   **Подписка**: Выберите свою подписку.

   **Группа ресурсов.** используйте ту же группу ресурсов, которую использует концентратор событий (**ContosoResourcesEH**).

   **Расположение.** используйте то же расположение, которое используется в сценарии установки, т. е. **West US** (западная часть США).

   ![Снимок экрана с созданием задания Stream Analytics.](./media/event-hubs-tutorial-visualize-anomalies/stream-analytics-add-job.png)

    Для остальных полей оставьте значения по умолчанию. Нажмите кнопку **Создать**. 

### <a name="add-an-input-to-the-stream-analytics-job"></a>Добавление входных данных в задание Stream Analytics

Если вам необходимо быстро перейти к панели **задания Stream Analytics** на портале, щелкните **Группа ресурсов** и выберите свою группу (**ContosoResourcesEH**). Это действие отобразит все ресурсы в группе, и вы сможете выбрать свое задание Stream Analytics. 

Входные данные для задания Stream Analytics представляют собой транзакции с кредитными картами из концентратора событий.

> [!NOTE]
> Значения переменных, имена которых начинаются со знака доллара ($), заданы в сценариях запуска из предыдущих разделов. Используйте здесь те же значения при указании этих полей, то есть пространства имен Центров событий и имени концентратора событий.

1. В разделе **Топология задания** щелкните **Входные данные**.

2. В панели **Входные данные** щелкните **Добавить потоковый вход** и выберите "Центры событий". На появившемся экране заполните следующие поля.

   **Входной псевдоним.** **contosoinputs**. Это поле содержит имя входного потока, используемого при определении запроса для данных.

   **Подписка**: Выберите свою подписку.

   **Пространство имен Центров событий**: выберите пространство имен концентратора событий ($**eventHubNamespace**). 

   **Имя концентратора событий**: щелкните **Использовать существующий** и выберите свой концентратор событий ($**eventHubName**).

   **Имя политики концентратора событий**. Выберите **RootManageSharedAccessKey**.

   **Группа потребителей концентратора событий**: оставьте это поле пустым, чтобы использовать группу потребителей по умолчанию.

   Для остальных полей оставьте значения по умолчанию.

   ![Снимок экрана с добавлением входного потока для задания Stream Analytics.](./media/event-hubs-tutorial-visualize-anomalies/stream-analytics-inputs.png)

5. Выберите команду **Сохранить**.

### <a name="add-an-output-to-the-stream-analytics-job"></a>Добавление выходных данных в задание Stream Analytics

1. В разделе **Топология задания** щелкните **Выходные данные**. Это поле содержит имя выходного потока, используемого при определении запроса для данных.

2. На панели **выходных данных** щелкните **Добавить** и выберите **Power BI**. На появившемся экране заполните следующие поля.

   **Выходной псевдоним.** **contosooutputs**. Уникальный псевдоним для выходных данных. 

   **Имя набора данных.** **contosoehdataset**. Имя набора данных для использования в Power BI. 

   **Имя таблицы.** **contosoehtable**. Имя таблицы для использования в Power BI. 

   Для остальных полей оставьте значения по умолчанию.

   ![Снимок экрана с настройкой выходных данных для задания Stream Analytics.](./media/event-hubs-tutorial-visualize-anomalies/stream-analytics-outputs.png)

3. Щелкните **Авторизовать** и войдите в учетную запись Power BI.

4. Для остальных полей оставьте значения по умолчанию.

5. Выберите команду **Сохранить**.

### <a name="configure-the-query-of-the-stream-analytics-job"></a>Настройка запроса задания Stream Analytics

Этот запрос используется для получения данных, которые в итоге отправляются на визуализацию в Power BI. В запросе используются псевдонимы **contosoinputs** и **contosooutputs**, которые мы задали ранее при настройке задания. Запрос извлекает транзакции с кредитными картами, которые он считает мошенническими, то есть в случаях, когда транзакции с одним номером кредитной карты происходят в разных местах в течение одного пятисекундного интервала.

1. В разделе **Топология задания** щелкните **Запрос**.

2. Замените запрос следующим: 

   ```SQL
   /* criteria for fraud:
      credit card purchases with the same card
      in different locations within 5 seconds
   */
   SELECT System.Timestamp AS WindowEnd, 
     COUNT(*) as FraudulentUses      
   INTO contosooutputs
   FROM contosoinputs CS1 TIMESTAMP BY [Timestamp]
       JOIN contosoinputs CS2 TIMESTAMP BY [Timestamp]
       /* where the credit card # is the same */
       ON CS1.CreditCardId = CS2.CreditCardId
       /* and time between the two is between 0 and 5 seconds */
       AND DATEDIFF(second, CS1, CS2) BETWEEN 0 AND 5
       /* where the location is different */
   WHERE CS1.Location != CS2.Location
   GROUP BY TumblingWindow(Duration(second, 1))
   ```

4. Выберите команду **Сохранить**.

### <a name="test-the-query-for-the-stream-analytics-job"></a>Тестирование запроса для задания Stream Analytics 

1. Запустите приложение для обнаружения аномалий, чтобы отправлять в концентратор событий данные, пока вы настраиваете и запускаете тест. 

2. В панели запроса щелкните точки рядом с входными данными **contosoinputs** и выберите **Образец данных с входа**.

3. Укажите, что необходимы данные за три минуты, и нажмите кнопку **ОК**. Дождитесь уведомления о выборке данных.

4. Щелкните **Тест** и убедитесь, что вы получаете результаты. Результаты отобразятся в разделе **Результаты** в нижней панели справа сразу под запросом.

5. Закройте панель запроса.

### <a name="run-the-stream-analytics-job"></a>Выполнение задания Stream Analytics

В задании Stream Analytics щелкните **Запуск**, **Сейчас** и снова **Запуск**. После успешного запуска состояние задания **Остановлено** изменится на **Выполняется**.

## <a name="set-up-the-power-bi-visualizations"></a>Настройка визуализаций Power BI

1. Запустите приложение для обнаружения аномалий, чтобы отправлять данные в концентратор событий, пока вы настраиваете визуализацию Power BI. Возможно, потребуется запустить приложение несколько раз, так как при каждом запуске оно генерирует только 1000 транзакций.

2. Выполните вход в учетную запись [Power BI](https://powerbi.microsoft.com/).

3. Перейдите в раздел **Моя рабочая область**.

4. Нажмите кнопку **Наборы данных**.

   Вы должны увидеть набор данных, указанный при создании выходных данных для задания Stream Analytics (**contosoehdataset**). Вывод набора данных в первый раз может занять 5–10 минут.

5. Щелкните **Панели мониторинга**, затем **Создать** и выберите **Панель мониторинга**.

   ![Снимок экрана с кнопками "Панели мониторинга" и "Создать".](./media/event-hubs-tutorial-visualize-anomalies/power-bi-add-dashboard.png)

6. Укажите имя панели мониторинга и нажмите **Создать**. Введите **Аномалии с кредитными картами**.

   ![Снимок экрана с указанием имени панели мониторинга.](./media/event-hubs-tutorial-visualize-anomalies/power-bi-dashboard-name.png)

7. На странице панели мониторинга щелкните **Добавить плитку**, затем в разделе **Данные в режиме реального времени** выберите **Пользовательские данные потоковой передачи** и щелкните **Далее**.

   ![Снимок экрана с указанием источника для плитки.](./media/event-hubs-tutorial-visualize-anomalies/power-bi-add-card-real-time-data.png)

8. Выберите свой набор данных (**contosoehdataset**) и щелкните **Далее**.

   ![Снимок экрана с указанием набора данных.](./media/event-hubs-tutorial-visualize-anomalies/power-bi-dashboard-select-dataset.png)

9. В качестве типа визуализации выберите **Карточка**. В разделе **Поля** щелкните **Добавить значение** и выберите `fraudulentuses`.

   ![Снимок экрана с указанием типа визуализации и полей.](./media/event-hubs-tutorial-visualize-anomalies/power-bi-add-card-tile.png)

   Щелкните **Далее**.

10. Задайте заголовок **Случаи мошенничества** и подзаголовок **Сумма за последние несколько минут**. Нажмите кнопку **Применить**. В результате плитка появится на панели мониторинга.

    ![Снимок экрана с указанием заголовка и подзаголовка для плитки панели мониторинга.](./media/event-hubs-tutorial-visualize-anomalies/power-bi-tile-details.png)

    > [!IMPORTANT]
    > Когда вы запускаете пример приложения и передаете поток данных в концентратор событий, цифра на этой плитке быстро меняется (каждую секунду). Это происходит потому, что запрос Stream Analytics фактически обновляет значение **каждую секунду**. Установите для запроса отображение "переворачивающегося" окна через 3 минуты, чтобы просматривать сумму за последние несколько минут. 
11. Добавьте еще одну визуализацию. Повторите первые несколько шагов:

    * Щелкните **Добавить плитку**.
    * Выберите **Пользовательские данные потоковой передачи**. 
    * Щелкните **Далее**.
    * Выберите свой набор данных и щелкните **Далее**. 

12. В поле **Тип визуализации** выберите **График**.

13. В разделе **Оси** щелкните **Добавить значение** и выберите `windowend`. 

14. В разделе **Значения** щелкните **Добавить значение** и выберите `fraudulentuses`.

15. В качестве **отображаемого интервала времени** задайте последние пять минут. Щелкните **Далее**.

16. В качестве заголовка плитки укажите **Случаи мошенничества за период**, подзаголовок оставьте пустым, затем нажмите кнопку **Применить**. Вы вернетесь к панели мониторинга.

17. Вновь запустите приложение для обнаружения аномалий, чтобы отправить данные в концентратор событий. Вы увидите на плитке **Случаев мошеннического использования** изменения по мере того, как она будет анализировать данные и данные будут отображаться на графике. 

    ![Снимок экрана, показывающий результаты в Power BI](./media/event-hubs-tutorial-visualize-anomalies/power-bi-results.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите удалить все созданные ресурсы, удалите данные визуализации Power BI, а затемгруппу ресурсов. Удаление группы ресурсов приведет к удалению всех ресурсов, которые она содержит. В нашем случае будут удалены концентратор событий, пространство имен концентратора событий, задание Stream Analytics и сама группа ресурсов. 

### <a name="clean-up-resources-in-the-power-bi-visualization"></a>Очистка ресурсов в визуализации Power BI

Войдите в учетную запись Power BI. Перейдите в раздел **Моя рабочая область**. В строке с именем вашей панели мониторинга щелкните значок корзины. Затем перейдите к **наборам данных** и щелкните значок корзины для удаления набора (**contosoehdataset**).

### <a name="clean-up-resources-using-azure-cli"></a>Очистка ресурсов с помощью Azure CLI

Чтобы удалить группу ресурсов, используйте команду [az group delete](/cli/azure/group#az_group_delete).

```azurecli-interactive
az group delete --name $resourceGroup
```

### <a name="clean-up-resources-using-powershell"></a>Очистка ресурсов с помощью PowerShell

Чтобы удалить группу ресурсов, используйте команду [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup).

```azurepowershell-interactive
Remove-AzResourceGroup -Name $resourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как выполнять следующие задачи:
> [!div class="checklist"]
> * Создание пространства имен в Центрах событий
> * Создание концентратора событий
> * Запуск приложения, которое имитирует события и отправляет их в концентратор событий
> * Настройка задания Stream Analytics для обработки событий, отправляемых в концентратор
> * Настройка визуализации Power BI для отображения результатов

Перейдите к следующей статье, чтобы узнать больше о Центрах событий Azure.

> [!div class="nextstepaction"]
> [Приступая к отправке событий в Центры событий Azure на платформе .NET Standard](event-hubs-dotnet-standard-getstarted-send.md)

[создайте бесплатную учетную запись]: https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio
