---
title: Публикации устойчивых функций в службе "Сетка событий Azure" (предварительная версия)
description: Узнайте, как настроить автоматическую публикацию в службе "Сетка событий Azure" для устойчивых функций.
ms.topic: conceptual
ms.date: 04/25/2020
ms.openlocfilehash: 44df100a5c794abf918a09dea0f94d30ddf916d3
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102175963"
---
# <a name="durable-functions-publishing-to-azure-event-grid-preview"></a>Публикации устойчивых функций в службе "Сетка событий Azure" (предварительная версия)

В этой статье показано, как настроить Устойчивые функции для публикации событий жизненного цикла оркестрации (например, события создания, завершения и сбоя) в пользовательском [разделе Сетки событий Azure](../../event-grid/overview.md).

Ниже приведены некоторые сценарии, где эта функция полезна:

* **Сценарии DevOps, такие как "синие" или "зеленые" развертывания**. Вам может потребоваться определить, выполняются ли какие-то задачи, перед реализацией [стратегии параллельного развертывания](durable-functions-versioning.md#side-by-side-deployments).

* **Поддержка расширенного мониторинга и диагностики**. Вы можете отслеживать сведения о состоянии оркестрации во внешнем хранилище, оптимизированном для запросов, например, базы данных Azure SQL или Azure Cosmos DB.

* **Продолжительное фоновое действие**. При использовании Устойчивых функций для продолжительного фонового действия эта функция позволяет вам узнать текущее состояние.

## <a name="prerequisites"></a>Предварительные требования

* Установите [Microsoft.Azure.WebJobs.Extensions.DurableTask](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.DurableTask) в проекте Устойчивых функций.
* Установите [эмулятор службы хранилища Azure](../../storage/common/storage-use-emulator.md) (только для Windows) или используйте существующую учетную запись хранения Azure.
* Установите [Azure CLI](/cli/azure/) или используйте [Azure Cloud Shell](../../cloud-shell/overview.md).

## <a name="create-a-custom-event-grid-topic"></a>Создание пользовательского раздела сетки событий

Создайте раздел сетки событий для отправки событий из устойчивых функций. Ниже показано, как создать раздел с помощью Azure CLI. Раздел также можно создать с помощью [PowerShell](../../event-grid/custom-event-quickstart-powershell.md) или на [портале Azure](../../event-grid/custom-event-quickstart-portal.md).

### <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте группу ресурсов с помощью команды `az group create`. Сейчас служба "Сетка событий Azure" поддерживает не все регионы. Сведения о поддерживаемых регионах службы "Сетка событий Azure" см. в [этой статье](../../event-grid/overview.md).

```azurecli
az group create --name eventResourceGroup --location westus2
```

### <a name="create-a-custom-topic"></a>Создание пользовательской темы

Раздел сетки событий содержит определяемую пользователем конечную точку, в которой можно размещать свое событие. Замените `<topic_name>` уникальным именем для вашей темы. Имя раздела должно быть уникальным, так как оно становится записью службы доменных имен (DNS).

```azurecli
az eventgrid topic create --name <topic_name> -l westus2 -g eventResourceGroup
```

## <a name="get-the-endpoint-and-key"></a>Получение конечной точки и ключа

Получите конечную точку раздела. Замените `<topic_name>` на выбранное имя.

```azurecli
az eventgrid topic show --name <topic_name> -g eventResourceGroup --query "endpoint" --output tsv
```

Получите ключ раздела. Замените `<topic_name>` на выбранное имя.

```azurecli
az eventgrid topic key list --name <topic_name> -g eventResourceGroup --query "key1" --output tsv
```

Теперь можно отправлять события в раздел.

## <a name="configure-event-grid-publishing"></a>Настройка публикации в службе "Сетка событий"

Найдите файл `host.json` в вашем проекте устойчивых функций.

### <a name="durable-functions-1x"></a>Устойчивые функции 1.x

Добавьте `eventGridTopicEndpoint` и `eventGridKeySettingName` в свойство `durableTask`.

```json
{
  "durableTask": {
    "eventGridTopicEndpoint": "https://<topic_name>.westus2-1.eventgrid.azure.net/api/events",
    "eventGridKeySettingName": "EventGridKey"
  }
}
```

### <a name="durable-functions-2x"></a>Устойчивые функции 2.x

Добавьте раздел `notifications` в свойство `durableTask` файла, заменив `<topic_name>` выбранным вами именем. Если свойства `durableTask` или `extensions` не существуют, создайте их, как показано в следующем примере:

```json
{
  "version": "2.0",
  "extensions": {
    "durableTask": {
      "notifications": {
        "eventGrid": {
          "topicEndpoint": "https://<topic_name>.westus2-1.eventgrid.azure.net/api/events",
          "keySettingName": "EventGridKey"
        }
      }
    }
  }
}
```

Возможные свойства конфигурации службы "Сетка событий Azure" можно найти в документации [host.json documentation](../functions-host-json.md#durabletask). После настройки файла `host.json` приложение-функция будет отправлять события жизненного цикла в службу "Сетки событий". Это действие выполняется при запуске приложения-функции как локально, так и в Azure.

Установите параметр приложения для ключа раздела в приложении-функции и `local.settings.json`. Следующий код JSON является примером `local.settings.json` для локальной отладки. Замените `<topic_key>` на ключ раздела.  

```json
{
    "IsEncrypted": false,
    "Values": {
        "AzureWebJobsStorage": "UseDevelopmentStorage=true",
        "AzureWebJobsDashboard": "UseDevelopmentStorage=true",
        "EventGridKey": "<topic_key>"
    }
}
```

Если вы используете [эмулятор службы хранилища](../../storage/common/storage-use-emulator.md) (только для Windows), убедитесь, что он работает. Рекомендуется запустить команду `AzureStorageEmulator.exe clear all` перед выполнением.

Если вы используете существующую учетную запись службы хранилища Azure, замените `UseDevelopmentStorage=true` в `local.settings.json` на ее строку подключения.

## <a name="create-functions-that-listen-for-events"></a>Создание функций, которые прослушивают события

С помощью портала Azure создайте другое приложение-функцию для прослушивания событий, опубликованных вашим приложением Устойчивых функций. Рекомендуется разместить это приложение функцию в том же регионе, где находится раздел службы "Сетка событий".

### <a name="create-an-event-grid-trigger-function"></a>Создание функции триггера сетки событий

1. В приложении-функции выберите **Функции**, а затем выберите **+ Добавить**. 

   :::image type="content" source="./media/durable-functions-event-publishing/function-add-function.png" alt-text="Добавьте функцию на портале Azure." border="true":::

1. Выполните поиск по фразе **Сетка событий**. Затем выберите шаблон **триггера службы "Сетка событий Azure"** . 

    :::image type="content" source="./media/durable-functions-event-publishing/function-select-event-grid-trigger.png" alt-text="Выберите шаблон триггера Сетки событий на портале Azure." border="true":::

1. Присвойте новому триггеру имя. Затем выберите элемент **Создать функцию**.

    :::image type="content" source="./media/durable-functions-event-publishing/function-name-event-grid-trigger.png" alt-text="Присвойте триггеру Сетки событий имя на портале Azure." border="true":::


    Будет создана функция со следующим кодом:

    # <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

    ```csharp
    #r "Newtonsoft.Json"
    using Newtonsoft.Json;
    using Newtonsoft.Json.Linq;
    using Microsoft.Extensions.Logging;

    public static void Run(JObject eventGridEvent, ILogger log)
    {
        log.LogInformation(eventGridEvent.ToString(Formatting.Indented));
    }
    ```

   # <a name="javascript"></a>[JavaScript](#tab/javascript)

   ```javascript
   module.exports = async function(context, eventGridEvent) {
       context.log(typeof eventGridEvent);
       context.log(eventGridEvent);
   }
   ```

---

### <a name="add-an-event-grid-subscription"></a>Добавление подписки в службе "Сетка событий"

Теперь вы можете добавить подписку на Сетку событий для созданного вами раздела. Дополнительные сведения см. в статье [Основные понятия в службе "Сетка событий Azure"](../../event-grid/concepts.md).

1. В новой функции выберите элемент **Интеграция** а затем выберите параметр **триггера Сетки событий (eventGridEvent)** . 

    :::image type="content" source="./media/durable-functions-event-publishing/eventgrid-trigger-link.png" alt-text="Выберите ссылку на триггер Сетки событий." border="true":::

1. Выберите **Create Event Grid Description** (Создать описание Сетки событий).

    :::image type="content" source="./media/durable-functions-event-publishing/create-event-grid-subscription.png" alt-text="Создайте подписку на Сетку событий." border="true":::

1. Присвойте имя подписке на события и выберите типа раздела из списка **разделов Сетки событий** 

1. Выберите подписку. Затем выберите для раздела Сетки событий созданную вами группу ресурсов и ресурс. 

1. Нажмите кнопку **создания**.

    :::image type="content" source="./media/durable-functions-event-publishing/event-grid-subscription-details.png" alt-text="Создайте подписку в службе &quot;Сетка событий&quot;." border="true":::

Теперь все готово для получения событий жизненного цикла.

## <a name="run-durable-functions-app-to-send-the-events"></a>Запуск приложения Устойчивых функций для отправки событий

В проекте Устойчивых функций, который вы настроили ранее, запустите отладку на локальном компьютере и начните оркестрацию. Приложение публикует события жизненного цикла Устойчивых функций в службе "Сетка событий". Убедитесь, что служба "Сетка событий" активирует созданную вами функцию прослушивателя. Для этого просмотрите журналы на портале Azure.

```
2019-04-20T09:28:21.041 [Info] Function started (Id=3301c3ef-625f-40ce-ad4c-9ba2916b162d)
2019-04-20T09:28:21.104 [Info] {
    "id": "054fe385-c017-4ce3-b38a-052ac970c39d",
    "subject": "durable/orchestrator/Running",
    "data": {
        "hubName": "DurableFunctionsHub",
        "functionName": "Sample",
        "instanceId": "055d045b1c8a415b94f7671d8df693a6",
        "reason": "",
        "runtimeStatus": "Running"
    },
    "eventType": "orchestratorEvent",
    "eventTime": "2019-04-20T09:28:19.6492068Z",
    "dataVersion": "1.0",
    "metadataVersion": "1",
    "topic": "/subscriptions/<your_subscription_id>/resourceGroups/eventResourceGroup/providers/Microsoft.EventGrid/topics/durableTopic"
}

2019-04-20T09:28:21.104 [Info] Function completed (Success, Id=3301c3ef-625f-40ce-ad4c-9ba2916b162d, Duration=65ms)
2019-04-20T09:28:37.098 [Info] Function started (Id=36fadea5-198b-4345-bb8e-2837febb89a2)
2019-04-20T09:28:37.098 [Info] {
    "id": "8cf17246-fa9c-4dad-b32a-5a868104f17b",
    "subject": "durable/orchestrator/Completed",
    "data": {
        "hubName": "DurableFunctionsHub",
        "functionName": "Sample",
        "instanceId": "055d045b1c8a415b94f7671d8df693a6",
        "reason": "",
        "runtimeStatus": "Completed"
    },
    "eventType": "orchestratorEvent",
    "eventTime": "2019-04-20T09:28:36.5061317Z",
    "dataVersion": "1.0",
    "metadataVersion": "1",
    "topic": "/subscriptions/<your_subscription_id>/resourceGroups/eventResourceGroup/providers/Microsoft.EventGrid/topics/durableTopic"
}
2019-04-20T09:28:37.098 [Info] Function completed (Success, Id=36fadea5-198b-4345-bb8e-2837febb89a2, Duration=0ms)
```

## <a name="event-schema"></a>Схема событий

В приведенном ниже списке описана схема событий жизненного цикла:

* **`id`** : Уникальный идентификатор события Сетки событий.
* **`subject`** : Путь к субъекту событий. `durable/orchestrator/{orchestrationRuntimeStatus}`. `{orchestrationRuntimeStatus}` будет `Running`, `Completed`, `Failed` и `Terminated`.  
* **`data`** : Определенные параметры Устойчивых функций.
  * **`hubName`** : Имя [TaskHub](durable-functions-task-hubs.md).
  * **`functionName`** : Имя функции оркестратора.
  * **`instanceId`** : Идентификатор экземпляра Устойчивых функций.
  * **`reason`** : Дополнительные данные, связанные с событием отслеживания. Дополнительные сведения см. в статье [Диагностика в устойчивых функциях (Функции Azure)](durable-functions-diagnostics.md).
  * **`runtimeStatus`** : Состояние среды выполнения оркестрации. "Running", "Completed", "Failed", "Canceled".
* **`eventType`** : "orchestratorEvent"
* **`eventTime`** : Время события (UTC).
* **`dataVersion`** : Версия схемы события жизненного цикла.
* **`metadataVersion`** :  Версия метаданных.
* **`topic`** : Ресурс раздела Сетки событий.

## <a name="how-to-test-locally"></a>Локальное тестирование

Сведения о локальном тестировании см. в статье [Локальная отладка триггера Сетки событий для функций Azure](../functions-debug-event-grid-trigger-local.md).

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Управление экземплярами в устойчивых функциях (Функции Azure)](durable-functions-instance-management.md)

> [!div class="nextstepaction"]
> [Управление версиями в устойчивых функциях (Функции Azure)](durable-functions-versioning.md)
