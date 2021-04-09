---
title: Учебник. Мониторинг опубликованных API-интерфейсов в службе "Управление API Azure" | Документация Майкрософт
description: Следуйте инструкциям из этого учебника, чтобы узнать, как использовать метрики, оповещения, журналы действий и журналы ресурсов для мониторинга интерфейсов API в службе "Управление API Azure".
services: api-management
author: vladvino
ms.service: api-management
ms.custom: mvc
ms.topic: tutorial
ms.date: 10/14/2020
ms.author: apimpm
ms.openlocfilehash: 1cb902c4b59193c46dbeca47bb355f0695a0f2c7
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100572798"
---
# <a name="tutorial-monitor-published-apis"></a>Руководство по Мониторинг опубликованных API-интерфейсов

С помощью Azure Monitor можно визуализировать, запрашивать, маршрутизировать, архивировать метрики или журналы, полученные от службы "Управление API Azure", а также реагировать на них.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * просмотр метрик API; 
> * настройка правила оповещения; 
> * Просмотр журналов действий
> * включение и просмотр журналов ресурсов.

Кроме того, с помощью встроенной в службу "Управление API" [аналитики](howto-use-analytics.md) можно отслеживать использование и производительность API-интерфейсов.

## <a name="prerequisites"></a>Предварительные требования

+ Ознакомьтесь с [терминологией службы управления API в Azure](api-management-terminology.md).
+ Выполните задачи в кратком руководстве по [созданию экземпляра службы управления API Azure](get-started-create-service-instance.md).
+ Также выполните инструкции из руководства [Импорт и публикация первого API](import-and-publish.md).

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="view-metrics-of-your-apis"></a>Просмотр метрик API

Служба "Управление API" каждую минуту передает [метрики](../azure-monitor/essentials/data-platform-metrics.md), позволяя отслеживать состояние и работоспособность API-интерфейсов практически в реальном времени. Ниже приведены две самые часто используемые метрики. Список всех доступных метрик см. в разделе со списком [поддерживаемых метрик](../azure-monitor/essentials/metrics-supported.md#microsoftapimanagementservice).

* **Емкость**. Позволяет принимать решения о необходимости повышения или понижения уровня служб Управления API. Метрика отправляется каждую минуту и отражает емкость шлюза в момент создания отчета. Значение метрики колеблется в диапазоне от 0 до 100. Оно вычисляется на основе ресурсов шлюза, таких как загрузка центрального процессора и использование памяти.
* **Запросы**. Помогает анализировать трафик API, проходящий через службы Управления API. Метрика отправляется каждую минуту и сообщает о количестве запросов к шлюзу с включением таких измерений, как коды ответа, расположение, имя узла и ошибки. 

> [!IMPORTANT]
> Следующие метрики не поддерживаются с мая 2019 г. и будут удалены в августе 2023 г.: Total Gateway Requests (общее количество запросов к шлюзу), Successful Gateway Requests (количество выполненных запросов к шлюзу), Unauthorized Gateway Requests (количество неавторизованных запросов к шлюзу), Failed Gateway Requests (количество неудачных запросов к шлюзу), Other Gateway Requests (количество других запросов к шлюзу). Используйте метрику Requests, которая имеет аналогичную функциональность.

:::image type="content" source="media/api-management-howto-use-azure-monitor/apim-monitor-metrics.png" alt-text="Снимок экрана: метрики на странице обзора службы &quot;Управление API&quot;":::

Для доступа к метрике сделайте следующее:

1. Перейдите к экземпляру Управления API на [портале Azure](https://portal.azure.com). На странице **Обзор** просмотрите основные метрики для API-интерфейсов.
1. Чтобы подробно изучить метрики, в меню внизу страницы выберите **Метрики**.

    :::image type="content" source="media/api-management-howto-use-azure-monitor/api-management-metrics-blade.png" alt-text="Снимок экрана: элемент &quot;Метрики&quot; в меню &quot;Мониторинг&quot;":::

1. В раскрывающемся списке выберите метрики, которые вас интересуют. Например, **Requests**. 
1. Просмотрите диаграмму с общим числом вызовов API.
1. Диаграмму можно отфильтровать с помощью измерений метрики **запросов**. Например, выберите **Добавить фильтр**, а затем — **Backend Response Code** (Категория кода отклика серверной части) и введите 500 в качестве значения. Теперь на диаграмме показано количество запросов, завершившихся сбоем в серверной части API.   

## <a name="set-up-an-alert-rule"></a>Настройка правила оповещения 

Вы можете получать [оповещения](../azure-monitor/alerts/alerts-metric-overview.md) на основе метрик и журналов действий. Azure Monitor позволяет [настроить действие](../azure-monitor/alerts/alerts-metric.md), выполняемое при активации оповещения:

* Отправка уведомления по электронной почте.
* Вызов webhook.
* Вызов приложения логики Azure.

Чтобы настроить пример правила генерации оповещений на основе метрики запроса, выполните следующие действия.

1. Перейдите к экземпляру Управления API на [портале Azure](https://portal.azure.com).
1. В меню внизу страницы выберите пункт **Оповещения**.

    :::image type="content" source="media/api-management-howto-use-azure-monitor/alert-menu-item.png" alt-text="Снимок экрана: параметр &quot;Оповещения&quot; в меню &quot;Мониторинг&quot;":::

1. Выберите **+ Новое правило генерации оповещений**.
1. В окне **Создать правило генерации оповещений** щелкните **Выбрать условие**.
1. В окне **Настройка логики сигналов** выполните следующие действия:
    1. Выберите **Метрики** для параметра **Тип сигнала**.
    1. Выберите **Запросы** для параметра **Имя сигнала**.
    1. В разделе **Split by dimensions** (Разделение по измерениям) в поле **Имя измерения** выберите **Gateway Response Code Category** (Категория кода отклика шлюза).
    1. В поле **Значения измерения** для таких ошибок клиента, как неавторизованные или недопустимые запросы, выберите значение **4xx**.
    1. В разделе **Логика оповещений** укажите пороговое значение, после которого будет активироваться оповещение, и нажмите кнопку **Готово**.

    :::image type="content" source="media/api-management-howto-use-azure-monitor/threshold.png" alt-text="Снимок экрана окон настройки логики сигналов":::

1. Создайте группу действий или выберите имеющуюся. В следующем примере создается группа действий. Пользователю admin@contoso.com будет отправлено электронное письмо с уведомлением. 

    :::image type="content" source="media/api-management-howto-use-azure-monitor/action-details.png" alt-text="Снимок экрана: уведомления для новой группы действий":::

1. Введите имя и описание правила генерации оповещений и выберите уровень серьезности. 
1. Выберите **Создать правило генерации оповещений**.
1. Теперь проверьте правило генерации оповещений, вызвав Conference API без ключа API. Пример:

    ```bash
    curl GET https://apim-hello-world.azure-api.net/conference/speakers HTTP/1.1 
    ```

    По результатам ознакомительного периода будет активировано оповещение и отправлено сообщение электронной почты по адресу admin@contoso.com. 

    Кроме того, оповещения отображаются на странице **Оповещения** для экземпляра Управления API.

    :::image type="content" source="media/api-management-howto-use-azure-monitor/portal-alerts.png" alt-text="Снимок экрана оповещений на портале":::

## <a name="activity-logs"></a>Журналы действий

Журналы действий позволяют подробно проанализировать операции, выполненные в службах управления API. С помощью журналов изменений можно ответить на вопросы "что? кто? когда?" о любой операции записи (PUT, POST, DELETE) в службах управления API.

> [!NOTE]
> Журналы действий не содержат операций чтения (GET) и операций, выполненных на портале Azure или с помощью исходных API управления.

Вы можете получить доступ к журналам действий в службе управления API или получить доступ к журналам всех ресурсов Azure в Azure Monitor. 

:::image type="content" source="media/api-management-howto-use-azure-monitor/api-management-activity-logs.png" alt-text="Снимок экрана журнала действий на портале":::

Для просмотра журнала действий выполните следующие действия:

1. Перейдите к экземпляру Управления API на [портале Azure](https://portal.azure.com).

1. Выберите **Журнал действий**.

    :::image type="content" source="media/api-management-howto-use-azure-monitor/api-management-activity-logs-blade.png" alt-text="Снимок экрана: элемент &quot;Журнал действий&quot; в меню &quot;Мониторинг&quot;":::
1. Выберите требуемую область фильтрации и щелкните **Применить**.

## <a name="resource-logs"></a>Журналы ресурсов

Журналы ресурсов предоставляют подробные сведения об операциях и ошибках, которые важны для аудита, а также для устранения неполадок. Журналы ресурсов отличаются от журналов действий. Журнал действий позволяет подробно проанализировать операции, выполненные с ресурсами Azure. Журналы ресурсов дают представление об операциях, выполняемых ресурсом.

Чтобы настроить журналы ресурсов, выполните приведенные ниже действия.

1. Перейдите к экземпляру Управления API на [портале Azure](https://portal.azure.com).
2. Выберите **Параметры диагностики**.

    :::image type="content" source="media/api-management-howto-use-azure-monitor/api-management-diagnostic-logs-blade.png" alt-text="Снимок экрана: элемент &quot;Параметры диагностики&quot; в меню &quot;Мониторинг&quot;":::

1. Щелкните команду **Добавить параметр диагностики**.
1. Выберите журналы или метрики для сбора.

   Вы можете архивировать журналы ресурсов и метрики в учетную запись хранения, передать их потоком в концентратор событий или отправить в рабочую область Log Analytics. 

Дополнительные сведения см. в статье [Создание параметров диагностики для отправки журналов платформы и метрик в различные места назначения](../azure-monitor/essentials/diagnostic-settings.md).

## <a name="view-diagnostic-data-in-azure-monitor"></a>Просмотр диагностических данных в Azure Monitor

Если включить сбор GatewayLogs или метрик в рабочей области Log Analytics, то для отображения данных в Azure Monitor может потребоваться несколько минут. Для просмотра данных выполните следующие действия:

1. Перейдите к экземпляру Управления API на [портале Azure](https://portal.azure.com).
1. В меню внизу страницы выберите пункт **Журналы**.

    :::image type="content" source="media/api-management-howto-use-azure-monitor/logs-menu-item.png" alt-text="Снимок экрана: элемент &quot;Журналы&quot; в меню &quot;Мониторинг&quot;":::

Выполните запросы, чтобы просмотреть данные. Вы можете воспользоваться предоставленными [примерами запросов](../azure-monitor/logs/example-queries.md) или выполнить собственный запрос. Например, следующий запрос получает данные из таблицы GatewayLogs за последние 24 часа:

```kusto
ApiManagementGatewayLogs
| where TimeGenerated > ago(1d) 
```

Дополнительные сведения об использовании журналов ресурсов для Управления API см. в следующих статьях:

* [Начало работы со службой Log Analytics в Azure Monitor](../azure-monitor/logs/log-analytics-tutorial.md). Вы также можете воспользоваться [демонстрационной средой Log Analytics](https://portal.loganalytics.io/demo).

* [Общие сведения о запросах к журналам в Azure Monitor](../azure-monitor/logs/log-query-overview.md).

Следующий код JSON указывает пример записи в GatewayLogs для успешного запроса API. Дополнительные сведения см. в [справочнике по схеме](gateway-log-schema-reference.md). 

```json
{
    "Level": 4,
    "isRequestSuccess": true,
    "time": "2020-10-14T17:xx:xx.xx",
    "operationName": "Microsoft.ApiManagement/GatewayLogs",
    "category": "GatewayLogs",
    "durationMs": 152,
    "callerIpAddress": "xx.xx.xxx.xx",
    "correlationId": "3f06647e-xxxx-xxxx-xxxx-530eb9f15261",
    "location": "East US",
    "properties": {
        "method": "GET",
        "url": "https://apim-hello-world.azure-api.net/conference/speakers",
        "backendResponseCode": 200,
        "responseCode": 200,
        "responseSize": 41583,
        "cache": "none",
        "backendTime": 87,
        "requestSize": 526,
        "apiId": "demo-conference-api",
        "operationId": "GetSpeakers",
        "apimSubscriptionId": "master",
        "clientTime": 65,
        "clientProtocol": "HTTP/1.1",
        "backendProtocol": "HTTP/1.1",
        "apiRevision": "1",
        "clientTlsVersion": "1.2",
        "backendMethod": "GET",
        "backendUrl": "https://conferenceapi.azurewebsites.net/speakers"
    },
    "resourceId": "/SUBSCRIPTIONS/<subscription ID>/RESOURCEGROUPS/<resource group>/PROVIDERS/MICROSOFT.APIMANAGEMENT/SERVICE/APIM-HELLO-WORLD"
}
```

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как выполнять следующие задачи:

> [!div class="checklist"]
> * просмотр метрик API;
> * настройка правила оповещения; 
> * Просмотр журналов действий
> * включение и просмотр журналов ресурсов.


Перейдите к следующему руководству:

> [!div class="nextstepaction"]
> [Трассировка вызовов](api-management-howto-api-inspector.md)