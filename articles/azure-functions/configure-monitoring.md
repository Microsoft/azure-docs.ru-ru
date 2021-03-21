---
title: Настройка мониторинга для функций Azure
description: Узнайте, как подключить приложение функции к Application Insights для мониторинга и настройки сбора данных.
ms.date: 8/31/2020
ms.topic: how-to
ms.custom: contperf-fy21q2
ms.openlocfilehash: 5007009d9aabf9a1c1c6e1d5c2f286c0ba25b340
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99493759"
---
# <a name="how-to-configure-monitoring-for-azure-functions"></a>Настройка мониторинга для функций Azure

Функции Azure интегрируются с Application Insights, что позволяет лучше отслеживать приложения функций. Application Insights, компонент Azure Monitor, — это расширяемая служба управления производительностью приложений (APM), которая собирает данные, создаваемые приложением-функцией, включая сведения, записываемые приложением в журналы. Application Insights интеграция обычно включается при создании приложения-функции. Если в приложении не задан ключ инструментирования, необходимо сначала [включить интеграцию с Application Insights](#enable-application-insights-integration). 

Вы можете использовать Application Insights без какой бы то ни было настройки конфигурации. Конфигурация по умолчанию может привести к созданию больших объемов данных. Если вы используете подписку Azure для Visual Studio, объем данных Application Insights может достигнуть установленного верхнего предела. Дополнительные сведения о Application Insights затратах см. в статье [Управление использованием и затратами для Application Insights](../azure-monitor/app/pricing.md).

Далее в этой статье вы узнаете, как настроить данные, отправляемые функциями в Application Insights. Для приложения-функции ведение журнала настраивается в файле [host.json]. 

> [!NOTE]
> Можно использовать специально настроенные параметры приложения для представления конкретных параметров в host.jsфайле для конкретной среды. Это позволяет эффективно изменять host.jsпараметров без необходимости повторной публикации host.jsфайла в проекте. Дополнительные сведения см. в разделе [Override host.json Values](functions-host-json.md#override-hostjson-values).

## <a name="configure-categories"></a>Настройка категорий

В средстве ведения журнала Функций Azure предусмотрена *категория* для каждого журнала. Категория указывает, какая часть кода среды выполнения или кода функции записывала данные в этот журнал. Категории отличаются от версий 1. x и более поздних версий. В следующей таблице описаны основные категории журналов, создаваемых средой выполнения. 

# <a name="v2x"></a>[v2. x +](#tab/v2)

| Category | Таблица | Описание |
| ----- | ----- | ----- |
| **`Function.<YOUR_FUNCTION_NAME>`** | **сборки**| Данные зависимостей автоматически собираются для некоторых служб. Для успешного выполнения эти журналы находятся на `Information` уровне. Дополнительные сведения см. в разделе [зависимости](functions-monitoring.md#dependencies). Исключения записываются на `Error` уровне. Среда выполнения также создает `Warning` журналы уровней, например, когда сообщения очереди отправляются в [очередь подозрительных](functions-bindings-storage-queue-trigger.md#poison-messages)сообщений. | 
| **`Function.<YOUR_FUNCTION_NAME>`** | **customMetrics**<br/>**customEvents** | Пакеты SDK для C# и JavaScript позволяют собираются пользовательские метрики и заносят в журнал пользовательские события. Дополнительные сведения см. в разделе [пользовательские данные телеметрии](functions-monitoring.md#custom-telemetry-data).|
| **`Function.<YOUR_FUNCTION_NAME>`** | **traces**| Включает в себя запущенные и завершенные журналы функций для конкретных запусков функций. Для успешного выполнения эти журналы находятся на `Information` уровне. Исключения записываются на `Error` уровне. Среда выполнения также создает `Warning` журналы уровней, например, когда сообщения очереди отправляются в [очередь подозрительных](functions-bindings-storage-queue-trigger.md#poison-messages)сообщений. | 
| **`Function.<YOUR_FUNCTION_NAME>.User`** | **traces**| Созданные пользователем журналы, которые могут иметь любой уровень ведения журнала. Дополнительные сведения о записи в журналы из функций см. в разделе [запись в журналы](functions-monitoring.md#writing-to-logs). | 
| **`Host.Aggregator`** | **customMetrics** | Эти журналы, созданные во время выполнения, предоставляют счетчики и средние значения вызовов функций за [Настраиваемый](#configure-the-aggregator) период времени. По умолчанию используется период 30 секунд или 1000 результатов в зависимости от того, что из этого наступит раньше. Например, здесь вы найдете число запусков, долю успешных попыток и длительность выполнения. Все эти журналы записываются на уровне `Information`. При фильтрации на уровне `Warning` или выше эти данные отображаться не будут. |
| **`Host.Results`** | **requests** | Эти журналы, созданные во время выполнения, указывают на успешное выполнение или сбой функции. Все эти журналы записываются на уровне `Information`. При фильтрации на уровне `Warning` или выше эти данные отображаться не будут. |
| **`Microsoft`** | **traces** | Полностью определенная категория журнала, отражающая компонент среды выполнения .NET, вызываемый узлом.  |
| **`Worker`** | **traces** | Журналы, созданные на языке рабочего процесса для языков non-.NET. Журналы рабочих ролей языка также могут быть зарегистрированы в `Microsoft.*` категории, например `Microsoft.Azure.WebJobs.Script.Workers.Rpc.RpcFunctionInvocationDispatcher` . Эти журналы записываются на `Information` уровне.|

> [!NOTE]
> Для функций библиотеки классов .NET в этих категориях предполагается, что вы используете, `ILogger` а не `ILogger<T>` . Дополнительные сведения см. в [документации по функциям ILogger](functions-dotnet-class-library.md#ilogger). 

# <a name="v1x"></a>[Версия 1.x](#tab/v1)

| Category | Таблица | Описание |
| ----- | ----- | ----- |
| **`Function`** | **traces**| Созданные пользователем журналы, которые могут иметь любой уровень ведения журнала. Дополнительные сведения о записи в журналы из функций см. в разделе [запись в журналы](functions-monitoring.md#writing-to-logs). | 
| **`Host.Aggregator`** | **customMetrics** | Эти журналы, созданные во время выполнения, предоставляют счетчики и средние значения вызовов функций за [Настраиваемый](#configure-the-aggregator) период времени. По умолчанию используется период 30 секунд или 1000 результатов в зависимости от того, что из этого наступит раньше. Например, здесь вы найдете число запусков, долю успешных попыток и длительность выполнения. Все эти журналы записываются на уровне `Information`. При фильтрации на уровне `Warning` или выше эти данные отображаться не будут. |
| **`Host.Executor`** | **traces** | Включает **функции, запущенные** и **работающие** с журналами, для выполнения конкретных функций. Успешные выполнения записываются в эти журналы на уровне `Information`. Исключения записываются на уровне `Error`. Среда выполнения также создает `Warning` журналы уровней, например, когда сообщения очереди отправляются в [очередь подозрительных](functions-bindings-storage-queue-trigger.md#poison-messages)сообщений.  |
| **`Host.Results`** | **requests** | Эти журналы, созданные во время выполнения, указывают на успешное выполнение или сбой функции. Все эти журналы записываются на уровне `Information`. При фильтрации на уровне `Warning` или выше эти данные отображаться не будут. |

---

Столбец **таблицы** указывает, в какой таблице в Application Insights записывается журнал. 

## <a name="configure-log-levels"></a>Настройка уровней ведения журнала

[!INCLUDE [functions-log-levels](../../includes/functions-log-levels.md)]

В каждой категории вы можете указать минимальный уровень ведения журнала для отправки данных. host.jsпараметров зависят от [версии среды выполнения функций](functions-versions.md). 

В приведенном ниже примере определяется ведение журнала на основе следующих правил.

+ Для журналов `Host.Results` или `Function` , события регистрируются только на `Error` уровне или более высоком. 
+ Для журналов регистрируются `Host.Aggregator` все созданные метрики ( `Trace` ).
+ Для всех остальных журналов, в том числе журналов пользователей, `Information` событий уровня только для журнала и выше.

# <a name="v2x"></a>[v2. x +](#tab/v2)

```json
{
  "logging": {
    "fileLoggingMode": "always",
    "logLevel": {
      "default": "Information",
      "Host.Results": "Error",
      "Function": "Error",
      "Host.Aggregator": "Trace"
    }
  }
}
```

# <a name="v1x"></a>[Версия 1.x](#tab/v1) 

```json
{
  "logger": {
    "categoryFilter": {
      "defaultLevel": "Information",
      "categoryLevels": {
        "Host.Results": "Error",
        "Function": "Error",
        "Host.Aggregator": "Trace"
      }
    }
  }
}
```

---

Если [host.js] включает в себя несколько журналов, начинающихся с одной и той же строки, сначала сопоставляются более определенные журналы. Рассмотрим следующий пример, который регистрирует все в среде выполнения, за исключением `Host.Aggregator` `Error` уровня:

# <a name="v2x"></a>[v2. x +](#tab/v2)

```json
{
  "logging": {
    "fileLoggingMode": "always",
    "logLevel": {
      "default": "Information",
      "Host": "Error",
      "Function": "Error",
      "Host.Aggregator": "Information"
    }
  }
}
```

# <a name="v1x"></a>[Версия 1.x](#tab/v1) 

```json
{
  "logger": {
    "categoryFilter": {
      "defaultLevel": "Information",
      "categoryLevels": {
        "Host": "Error",
        "Function": "Error",
        "Host.Aggregator": "Information"
      }
    }
  }
}
```

---

Можно использовать параметр уровня ведения журнала, `None` чтобы запретить запись журналов для категории. 

## <a name="configure-the-aggregator"></a>Настройка агрегатора

Как отмечалось в предыдущем разделе, среда выполнения собирает данные о выполнении функции за определенный период времени. По умолчанию используется период в 30 секунд или 1000 запусков в зависимости от того, что из этого наступит раньше. Этот параметр можно настроить в файле [host.json].  Ниже приведен пример:

```json
{
    "aggregator": {
      "batchSize": 1000,
      "flushTimeout": "00:00:30"
    }
}
```

## <a name="configure-sampling"></a>Настройка выборки

В Application Insights есть функция [выборки](../azure-monitor/app/sampling.md), которая позволяет избежать создания слишком большого объема данных телеметрии для завершенных выполнений в периоды пиковой нагрузки. Если скорость входящих выполнений превышает заданное пороговое значение, служба Application Insights будет случайным образом игнорировать часть входящих выполнений. Максимальное количество выполнений в секунду по умолчанию — 20 (пять в версии 1.x). Вы можете настроить выборку в файле [host.json](./functions-host-json.md#applicationinsights).  Ниже приведен пример:

# <a name="v2x"></a>[v2. x +](#tab/v2)

```json
{
  "logging": {
    "applicationInsights": {
      "samplingSettings": {
        "isEnabled": true,
        "maxTelemetryItemsPerSecond" : 20,
        "excludedTypes": "Request;Exception"
      }
    }
  }
}
```


Вы можете исключить определенные типы данных телеметрии из выборки. В этом примере данные типа `Request` и `Exception` исключены из выборки. Это гарантирует, что *все* выполнения функций (запросы) и исключения записываются в журнал, а другие типы телеметрии остаются в процессе выборки. 

# <a name="v1x"></a>[Версия 1.x](#tab/v1)  

```json
{
  "applicationInsights": {
    "sampling": {
      "isEnabled": true,
      "maxTelemetryItemsPerSecond" : 5
    }
  }
}
```
---

Дополнительные сведения см. [в разделе Выборка в Application Insights](../azure-monitor/app/sampling.md).

## <a name="configure-scale-controller-logs"></a>Настройка журналов контроллера масштабирования

_Эта функция доступна в виде предварительной версии._ 

Чтобы лучше понять, какие решения выполняет контроллер масштабирования для приложения-функции, можно создать журналы порождаемого [контроллера масштабирования функций Azure](./event-driven-scaling.md#runtime-scaling) либо Application Insights, либо в хранилище BLOB-объектов.

Чтобы включить эту функцию, добавьте параметр приложения с именем `SCALE_CONTROLLER_LOGGING_ENABLED` в параметры приложения функции. Значение этого параметра должно иметь формат в `<DESTINATION>:<VERBOSITY>` зависимости от следующих значений:

[!INCLUDE [functions-scale-controller-logging](../../includes/functions-scale-controller-logging.md)]

Например, следующая команда Azure CLI включает подробное ведение журнала из контроллера масштабирования для Application Insights:

```azurecli-interactive
az functionapp config appsettings set --name <FUNCTION_APP_NAME> \
--resource-group <RESOURCE_GROUP_NAME> \
--settings SCALE_CONTROLLER_LOGGING_ENABLED=AppInsights:Verbose
```

В этом примере замените `<FUNCTION_APP_NAME>` и `<RESOURCE_GROUP_NAME>` именем приложения функции и именем группы ресурсов соответственно. 

Следующая команда Azure CLI отключает ведение журнала, устанавливая уровень детализации `None` :

```azurecli-interactive
az functionapp config appsettings set --name <FUNCTION_APP_NAME> \
--resource-group <RESOURCE_GROUP_NAME> \
--settings SCALE_CONTROLLER_LOGGING_ENABLED=AppInsights:None
```

Можно также отключить ведение журнала, удалив `SCALE_CONTROLLER_LOGGING_ENABLED` параметр с помощью следующей команды Azure CLI:

```azurecli-interactive
az functionapp config appsettings delete --name <FUNCTION_APP_NAME> \
--resource-group <RESOURCE_GROUP_NAME> \
--setting-names SCALE_CONTROLLER_LOGGING_ENABLED
```

Если ведение журнала контроллера масштабирования включено, теперь вы можете [запрашивать журналы контроллера масштабирования](analyze-telemetry-data.md#query-scale-controller-logs). 

## <a name="enable-application-insights-integration"></a>Включение интеграции с Application Insights

Для отправки данных в Application Insights приложению-функции требуется ключ инструментирования для ресурса Application Insights. В параметре приложения должен быть ключ с именем **APPINSIGHTS_INSTRUMENTATIONKEY**.

При создании приложения-функции на [портале Azure](./functions-get-started.md) из командной строки с помощью [Azure Functions Core Tools](./create-first-function-cli-csharp.md) или в [Visual Studio Code](./create-first-function-vs-code-csharp.md) интеграция с Application Insights включена по умолчанию. Ресурс Application Insights имеет то же имя, что и приложение-функция, и создается в том же или ближайшем регионе.

### <a name="new-function-app-in-the-portal"></a>Создание приложения-функции на портале Azure

Для проверки создаваемого ресурса Application Insights выберите его, чтобы развернуть окно **Application Insights**. Вы можете задать **новое имя ресурса** или выбрать другое **расположение** в [географическом регионе Azure](https://azure.microsoft.com/global-infrastructure/geographies/), где будут храниться данные.

![Включение Application Insights при создании приложения-функции](media/functions-monitoring/enable-ai-new-function-app.png)

При выборе команды **Создать** создается ресурс Application Insights с приложением-функцией, в параметрах приложения которого задано `APPINSIGHTS_INSTRUMENTATIONKEY`. Все готово к работе.

<a id="manually-connect-an-app-insights-resource"></a>
### <a name="add-to-an-existing-function-app"></a>Добавление имеющегося приложения-функции 

Если ресурс Application Insights не был создан с помощью приложения функции, выполните следующие действия, чтобы создать ресурс. Затем ключ инструментирования из этого ресурса можно добавить в качестве [параметра приложения](functions-how-to-use-azure-function-app-settings.md#settings) в приложение-функцию.

1. В [портал Azure](https://portal.azure.com)найдите и выберите **приложение функции**, а затем выберите приложение функции. 

1. Щелкните баннер **Application Insights не настроено** в верхней части окна. Если вы не видите этот баннер, возможно, для приложения уже включено Application Insights.

    :::image type="content" source="media/configure-monitoring/enable-application-insights.png" alt-text="Включение Application Insights с помощью портала":::

1. Разверните пункт **Изменить ресурс** и создайте ресурс Application Insights с помощью параметров, указанных в следующей таблице.  

    | Параметр      | Рекомендуемое значение  | Описание                                        |
    | ------------ |  ------- | -------------------------------------------------- |
    | **Новое название ресурса** | Уникальное имя приложения | Проще всего использовать имя приложения-функции, которое должно быть уникальным в вашей подписке. | 
    | **Расположение** | Западная Европа | Если возможно, используйте [регион](https://azure.microsoft.com/regions/) приложения-функции или ближайший от него. |

    :::image type="content" source="media/configure-monitoring/ai-general.png" alt-text="Создание ресурса Application Insights":::

1. Нажмите кнопку **Применить**. 

   Ресурс Application Insights создается в той же группе ресурсов и подписке, что и приложение-функция. После создания ресурса закройте окно Application Insights.

1. В приложении-функции выберите **Конфигурация** в разделе **Параметры** и выберите **Параметры приложения**. Если появится параметр `APPINSIGHTS_INSTRUMENTATIONKEY`, это означает, что для приложения-функции в Azure включена интеграция с Application Insights. Если по какой либо причине эта настройка не существует, добавьте ее с помощью ключа инструментирования Application Insights в качестве значения.

> [!NOTE]
> Ранние версии решения "Функции" использовали встроенный мониторинг, который больше не рекомендуется. При включении интеграции Application Insights для такого приложения-функции необходимо также [отключить встроенное ведение журналов](#disable-built-in-logging).  

## <a name="disable-built-in-logging"></a>Отключение встроенного ведения журнала

При включении Application Insights отключите встроенное ведение журнала, которое использует службу хранилища Azure. Встроенное ведение журнала полезно для тестирования с легкими рабочими нагрузками, но не предназначено для использования в рабочей среде с высокой нагрузкой. Для мониторинга рабочей среды рекомендуем использовать Application Insights. Если в рабочей среде используется встроенное ведение журнала, записи журнала могут быть неполными из-за регулирования службы хранилища Azure.

Чтобы отключить встроенное ведение журнала, удалите параметр приложения `AzureWebJobsDashboard`. Дополнительные сведения о том, как удалять параметры приложения на портале Azure, см. в разделе **Параметры приложения** статьи [Управление приложением-функцией на портале Azure](functions-how-to-use-azure-function-app-settings.md#settings). Перед удалением параметра приложения убедитесь, что имеющиеся функции в том же приложении-функции не используют этот параметр для триггеров или привязок службы хранилища Azure.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о мониторинге см. в следующих статьях:

+ [Мониторинг Функций Azure](functions-monitoring.md)
+ [Анализ данных телеметрии функций Azure в Application Insights](analyze-telemetry-data.md)
+ [Application Insights](/azure/application-insights/)


[host.json]: functions-host-json.md
