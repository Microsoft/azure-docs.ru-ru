---
title: Log Analytics экспорт данных рабочей области в Azure Monitor (Предварительная версия)
description: Log Analytics экспорт данных позволяет непрерывно экспортировать данные выбранных таблиц из рабочей области Log Analytics в учетную запись хранения Azure или концентратор событий Azure по мере их сбора.
ms.topic: conceptual
ms.custom: references_regions, devx-track-azurecli
author: bwren
ms.author: bwren
ms.date: 02/07/2021
ms.openlocfilehash: f0bbe02576323342376ad155878d575c6403cf70
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102048817"
---
# <a name="log-analytics-workspace-data-export-in-azure-monitor-preview"></a>Log Analytics экспорт данных рабочей области в Azure Monitor (Предварительная версия)
Log Analytics экспорт данных рабочей области в Azure Monitor позволяет непрерывно экспортировать данные из выбранных таблиц в Log Analytics рабочей области в учетную запись хранения Azure или концентратор событий Azure по мере их сбора. Эта статья содержит сведения об этой функции и действиях по настройке экспорта данных в рабочих областях.

## <a name="overview"></a>Обзор
После настройки экспорта данных для рабочей области Log Analytics все новые данные, отправляемые в выбранные таблицы в рабочей области, автоматически экспортируются в учетную запись хранения в почасовом добавочном добавлении BLOB-объектов или концентратора событий в ближайшем режиме реального времени.

![Общие сведения о экспорте данных](media/logs-data-export/data-export-overview.png)

Все данные из включенных таблиц экспортируются без фильтра. Например, при настройке правила экспорта данных для таблицы *SecurityEvent* все данные, отправленные в таблицу *SecurityEvent* , экспортируются начиная с момента настройки.


## <a name="other-export-options"></a>Другие параметры экспорта
Log Analytics экспорт данных рабочей области непрерывно экспортирует данные из рабочей области Log Analytics. Далее приведены другие варианты экспорта данных для определенных сценариев.

- Запланированный экспорт из запроса журнала с помощью приложения логики. Это похоже на функцию экспорта данных, но позволяет отсылать отфильтрованные или агрегированные данные в службу хранилища Azure. При использовании этого метода в соответствии с [ограничениями запросов журнала](../service-limits.md#log-analytics-workspaces)можно ознакомиться с разделом [Архивация данных из log Analytics рабочей области в службу хранилища Azure с помощью приложения логики](logs-export-logic-app.md).
- Один раз экспортируйте на локальный компьютер с помощью скрипта PowerShell. См. раздел [Invoke-азоператионалинсигхтскуерекспорт](https://www.powershellgallery.com/packages/Invoke-AzOperationalInsightsQueryExport).


## <a name="limitations"></a>Ограничения

- Конфигурацию можно выполнить с помощью запросов CLI или RESTFUL в настоящее время. Портал Azure или PowerShell еще не поддерживаются.
- ```--export-all-tables```Параметр в интерфейсе командной строки и интерфейсе RESTful не поддерживается и будет удален. Необходимо явно указать список таблиц в правилах экспорта.
- Поддерживаемые таблицы в настоящее время ограничиваются указанными в разделе [Поддерживаемые таблицы](#supported-tables) ниже. Например, в настоящее время пользовательские таблицы журналов не поддерживаются.
- Если правило экспорта данных включает неподдерживаемую таблицу, операция будет выполнена, но данные для этой таблицы не будут экспортированы, пока не будет поддерживаться таблица. 
- Если правило экспорта данных включает таблицу, которая не существует, она завершится ошибкой ```Table <tableName> does not exist in the workspace``` .
- Рабочая область Log Analytics может находиться в любом регионе, за исключением следующих:
  - регионы Azure для государственных организаций;
  - Западная Япония
  - Юго-восток Бразилии
  - Восточная Норвегия;
  - Северная часть ОАЭ;
- В рабочей области можно создать два правила экспорта — одно правило для концентратора событий и одно правило для учетной записи хранения.
- Целевая учетная запись хранения или концентратор событий должны находиться в том же регионе, что и Рабочая область Log Analytics.
- Имена экспортируемых таблиц не могут содержать более 60 символов для учетной записи хранения и не более 47 символов в концентраторе событий. Таблицы с более длинными именами не будут экспортированы.
- Поддержка добавочных BLOB-объектов для Azure Data Lake Storage теперь [ограничена общедоступной предварительной версией](https://azure.microsoft.com/updates/append-blob-support-for-azure-data-lake-storage-preview/)

## <a name="data-completeness"></a>Полнота данных
Экспорт данных продолжит попытки отправки данных в течение 30 минут, когда назначение недоступно. Если он по-прежнему недоступен через 30 минут, данные будут удалены до тех пор, пока назначение не станет доступным.

## <a name="cost"></a>Стоимость
В настоящее время дополнительная плата за функцию экспорта данных не взимается. Цены на экспорт данных будут объявлены в будущем и появится уведомление, предоставленное до начала выставления счетов. Если вы решили продолжить использование экспорта данных после периода уведомления, плата будет взиматься по соответствующей ставке.

## <a name="export-destinations"></a>Экспорт назначений

### <a name="storage-account"></a>Учетная запись хранения
Данные отправляются в учетные записи хранения, так как они достигают Azure Monitor и хранятся в каждый час добавочных BLOB-объектов. Конфигурация экспорта данных создает контейнер для каждой таблицы в учетной записи хранения с именем *, за которым следует имя таблицы* . Например, таблица *SecurityEvent* будет отправлена в контейнер с именем *AM-SecurityEvent*.

Путь к BLOB-объекту учетной записи хранения — *воркспацересаурцеид =/субскриптионс/субскриптион-ИД/ресаурцеграупс/ \<resource-group\> /провидерс/Микрософт.оператионалинсигхтс/воркспацес/ \<workspace\> /y = \<four-digit numeric year\> /m = \<two-digit numeric month\> /d = \<two-digit numeric day\> /h = \<two-digit 24-hour clock hour\> /m = 00/PT1H.json*. Так как добавление больших двоичных объектов ограничено 50 000 записью в хранилище, количество экспортируемых больших двоичных объектов может расширяться, если число добавлений велико. Шаблон именования для больших двоичных объектов в таком случае будет PT1H_ #. JSON, где # — добавочное число больших двоичных объектов.

Формат данных учетной записи хранения — [строки JSON](../essentials/resource-logs-blob-format.md). Это означает, что каждая запись отделяется символом новой строки, без массива внешних записей и без запятых между записями JSON. 

[![Образец данных хранилища](media/logs-data-export/storage-data.png)](media/logs-data-export/storage-data.png#lightbox)

Log Analytics экспорт данных может записывать Добавление больших двоичных объектов в неизменяемые учетные записи хранения, если для политик хранения на основе времени включен параметр *алловпротектедаппендвритес* . Это позволяет записывать новые блоки в добавочный большой двоичный объект, сохраняя неизменную защиту и соответствие. См. раздел [разрешение записи BLOB-объектов с защищенным добавлением](../../storage/blobs/storage-blob-immutable-storage.md#allow-protected-append-blobs-writes).

> [!NOTE]
> Поддержка добавочных BLOB-объектов для хранилища Azure Data Lake теперь доступна в предварительной версии во всех регионах Azure. Прежде чем создавать правило экспорта для Azure Data Lake хранилища, [Зарегистрируйтесь в ограниченной общедоступной предварительной версии](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR4mEEwKhLjlBjU3ziDwLH-pURDk2NjMzUTVEVzU5UU1XUlRXSTlHSlkxQS4u) . Экспорт не будет действовать без этой регистрации.

### <a name="event-hub"></a>концентратор событий;
Данные отправляются в концентратор событий практически в реальном времени по мере достижения Azure Monitor. Концентратор событий создается для каждого экспортируемого типа данных *с именем, за которым* следует имя таблицы. Например, таблица *SecurityEvent* будет отправлена в концентратор событий с именем *AM-SecurityEvent*. Если вы хотите, чтобы экспортированные данные достигли определенного концентратора событий или если у вас есть таблица с именем, длина которого превышает ограничение в 47 символов, можно указать собственное имя концентратора событий и экспортировать все данные для определенных таблиц.

> [!IMPORTANT]
> [Количество поддерживаемых концентраторов событий на пространство имен равно 10](../../event-hubs/event-hubs-quotas.md#common-limits-for-all-tiers). При экспорте более 10 таблиц укажите собственное имя концентратора событий, чтобы экспортировать все таблицы в этот концентратор событий. 

Рекомендации:
1. SKU концентратора событий "базовый" поддерживает более низкий [предел](../../event-hubs/event-hubs-quotas.md#basic-vs-standard-tiers) размера событий, и некоторые журналы в рабочей области могут превыситься и быть удалены. В качестве назначения экспорта рекомендуется использовать концентратор событий "Стандартный" или "выделенный".
2. Объем экспортированных данных часто увеличивается со временем, а масштаб концентратора событий должен быть увеличен для обработки больших скоростей передачи и предотвращения ситуаций регулирования и задержки данных. Для автоматического масштабирования и увеличения количества единиц пропускной способности и удовлетворения потребностей в использовании следует использовать функцию автоматического расширения концентраторов событий. Дополнительные сведения см. в статье [Автоматическое масштабирование единиц пропускной способности концентраторов событий Azure](../../event-hubs/event-hubs-auto-inflate.md) .

## <a name="prerequisites"></a>Предварительные требования
Ниже приведены предварительные требования, которые необходимо выполнить перед настройкой Log Analytics экспорта данных.

- Учетная запись хранения и концентратор событий уже должны быть созданы и должны находиться в том же регионе, что и Рабочая область Log Analytics. Если вам нужно реплицировать данные в другие учетные записи хранения, можно использовать любые [параметры избыточности хранилища Azure](../../storage/common/storage-redundancy.md).  
- Учетная запись хранения должна быть StorageV1 или StorageV2. Классическое хранилище не поддерживается  
- Если вы настроили учетную запись хранения, чтобы разрешить доступ из выбранных сетей, необходимо добавить исключение в параметры учетной записи хранения, чтобы разрешить Azure Monitor запись в хранилище.

## <a name="enable-data-export"></a>Включить экспорт данных
Для включения Log Analytics экспорта данных необходимо выполнить следующие действия. Дополнительные сведения о каждой из них см. в следующих разделах.

- Регистрация поставщика ресурсов.
- Разрешите Доверенные службы Майкрософт.
- Создайте одно или несколько правил экспорта данных, определяющих экспортируемые таблицы и их назначение.

### <a name="register-resource-provider"></a>Регистрация поставщика ресурсов
Для включения Log Analytics экспорта данных необходимо зарегистрировать следующего поставщика ресурсов Azure в подписке. 

- Microsoft.Insights

Возможно, этот поставщик ресурсов уже зарегистрирован для большинства пользователей Azure Monitor. Чтобы проверить, перейдите к разделу **подписки** в портал Azure. Выберите свою подписку, а затем щелкните **поставщики ресурсов** в разделе **Параметры** меню. Нахождение **Microsoft. Insights**. Если его состояние **зарегистрировано**, оно уже зарегистрировано. В противном случае нажмите кнопку **зарегистрировать** , чтобы зарегистрировать его.

Можно также использовать любой из доступных методов для регистрации поставщика ресурсов, как описано в разделе [поставщики и типы ресурсов Azure](../../azure-resource-manager/management/resource-providers-and-types.md). Ниже приведен пример команды с помощью PowerShell:

```PowerShell
Register-AzResourceProvider -ProviderNamespace Microsoft.insights
```

### <a name="allow-trusted-microsoft-services"></a>Разрешить доверенные службы Майкрософт
Если вы настроили учетную запись хранения, чтобы разрешить доступ из выбранных сетей, необходимо добавить исключение, чтобы разрешить Azure Monitor запись в учетную запись. В учетной записи хранения в **брандмауэрах и виртуальных сетях** выберите **разрешить доверенным службам Майкрософт доступ к этой учетной записи хранения**.

[![Брандмауэры учетной записи хранения и виртуальные сети](media/logs-data-export/storage-account-vnet.png)](media/logs-data-export/storage-account-vnet.png#lightbox)


### <a name="create-or-update-data-export-rule"></a>Создать или обновить правило экспорта данных
Правило экспорта данных определяет экспортируемые данные для набора таблиц в одном месте назначения. Для каждого назначения можно создать одно правило.


# <a name="azure-portal"></a>[Портал Azure](#tab/portal)

Н/Д

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Недоступно

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Используйте следующую команду CLI для просмотра таблиц в рабочей области. Он поможет скопировать нужные таблицы и включить их в правило экспорта данных.

```azurecli
az monitor log-analytics workspace table list --resource-group resourceGroupName --workspace-name workspaceName --query [].name --output table
```

Используйте следующую команду, чтобы создать правило экспорта данных для учетной записи хранения с помощью интерфейса командной строки.

```azurecli
$storageAccountResourceId = '/subscriptions/subscription-id/resourceGroups/resource-group-name/providers/Microsoft.Storage/storageAccounts/storage-account-name'
az monitor log-analytics workspace data-export create --resource-group resourceGroupName --workspace-name workspaceName --name ruleName --tables SecurityEvent Heartbeat --destination $storageAccountResourceId
```

Используйте следующую команду, чтобы создать правило экспорта данных для концентратора событий с помощью интерфейса командной строки. Для каждой таблицы создается отдельный концентратор событий.

```azurecli
$eventHubsNamespacesResourceId = '/subscriptions/subscription-id/resourceGroups/resource-group-name/providers/Microsoft.EventHub/namespaces/namespaces-name'
az monitor log-analytics workspace data-export create --resource-group resourceGroupName --workspace-name workspaceName --name ruleName --tables SecurityEvent Heartbeat --destination $eventHubsNamespacesResourceId
```

Используйте следующую команду, чтобы создать правило экспорта данных для определенного концентратора событий с помощью интерфейса командной строки. Все таблицы экспортируются в указанное имя концентратора событий. 

```azurecli
$eventHubResourceId = '/subscriptions/subscription-id/resourceGroups/resource-group-name/providers/Microsoft.EventHub/namespaces/namespaces-name/eventHubName/eventhub-name'
az monitor log-analytics workspace data-export create --resource-group resourceGroupName --workspace-name workspaceName --name ruleName --tables SecurityEvent Heartbeat --destination $eventHubResourceId
```

# <a name="rest"></a>[REST](#tab/rest)

Используйте следующий запрос, чтобы создать правило экспорта данных с помощью REST API. Запрос должен использовать авторизацию токена носителя и тип содержимого Application/JSON.

```rest
PUT https://management.azure.com/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/Microsoft.operationalInsights/workspaces/<workspace-name>/dataexports/<data-export-name>?api-version=2020-08-01
```

В тексте запроса указывается назначение таблиц. Ниже приведен пример основного текста для запроса на запись хранения.

```json
{
    "properties": {
        "destination": {
            "resourceId": "/subscriptions/subscription-id/resourcegroups/resource-group-name/providers/Microsoft.Storage/storageAccounts/storage-account-name"
        },
        "tablenames": [
            "table1",
            "table2" 
        ],
        "enable": true
    }
}
```

Ниже приведен пример основного текста для запроса RESTFUL для концентратора событий.

```json
{
    "properties": {
        "destination": {
            "resourceId": "/subscriptions/subscription-id/resourcegroups/resource-group-name/providers/Microsoft.EventHub/namespaces/eventhub-namespaces-name"
        },
        "tablenames": [
            "table1",
            "table2"
        ],
        "enable": true
    }
}
```

Ниже приведен пример основного текста для запроса оставшейся части концентратора событий, в котором указано имя концентратора событий. В этом случае все экспортированные данные отправляются в этот концентратор событий.

```json
{
    "properties": {
        "destination": {
            "resourceId": "/subscriptions/subscription-id/resourcegroups/resource-group-name/providers/Microsoft.EventHub/namespaces/eventhub-namespaces-name",
            "metaData": {
                "EventHubName": "eventhub-name"
        },
        "tablenames": [
            "table1",
            "table2"
        ],
        "enable": true
    }
  }
}
```

# <a name="template"></a>[Шаблон](#tab/json)

Используйте следующую команду, чтобы создать правило экспорта данных для учетной записи хранения с помощью шаблона.

```
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "workspaceName": {
            "defaultValue": "workspace-name",
            "type": "String"
        },
        "workspaceLocation": {
            "defaultValue": "workspace-region",
            "type": "string"
        },
        "storageAccountRuleName": {
            "defaultValue": "storage-account-rule-name",
            "type": "string"
        },
        "storageAccountResourceId": {
            "defaultValue": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/resource-group-name/providers/Microsoft.Storage/storageAccounts/storage-account-name",
            "type": "String"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "microsoft.operationalinsights/workspaces",
            "apiVersion": "2020-08-01",
            "name": "[parameters('workspaceName')]",
            "location": "[parameters('workspaceLocation')]",
            "resources": [
                {
                  "type": "microsoft.operationalinsights/workspaces/dataexports",
                  "apiVersion": "2020-08-01",
                  "name": "[concat(parameters('workspaceName'), '/' , parameters('storageAccountRuleName'))]",
                  "dependsOn": [
                      "[resourceId('microsoft.operationalinsights/workspaces', parameters('workspaceName'))]"
                  ],
                  "properties": {
                      "destination": {
                          "resourceId": "[parameters('storageAccountResourceId')]"
                      },
                      "tableNames": [
                          "Heartbeat",
                          "InsightsMetrics",
                          "VMConnection",
                          "Usage"
                      ],
                      "enable": true
                  }
              }
            ]
        }
    ]
}
```

Используйте следующую команду, чтобы создать правило экспорта данных для концентратора событий с помощью шаблона. Для каждой таблицы создается отдельный концентратор событий.

```
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "workspaceName": {
            "defaultValue": "workspace-name",
            "type": "String"
        },
        "workspaceLocation": {
            "defaultValue": "workspace-region",
            "type": "string"
        },
        "eventhubRuleName": {
            "defaultValue": "event-hub-rule-name",
            "type": "string"
        },
        "namespacesResourceId": {
            "defaultValue": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/resource-group-name/providers/microsoft.eventhub/namespaces/namespaces-name",
            "type": "String"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "microsoft.operationalinsights/workspaces",
            "apiVersion": "2020-08-01",
            "name": "[parameters('workspaceName')]",
            "location": "[parameters('workspaceLocation')]",
            "resources": [
              {
                  "type": "microsoft.operationalinsights/workspaces/dataexports",
                  "apiVersion": "2020-08-01",
                  "name": "[concat(parameters('workspaceName'), '/', parameters('eventhubRuleName'))]",
                  "dependsOn": [
                      "[resourceId('microsoft.operationalinsights/workspaces', parameters('workspaceName'))]"
                  ],
                  "properties": {
                      "destination": {
                          "resourceId": "[parameters('namespacesResourceId')]"
                      },
                      "tableNames": [
                          "Usage",
                          "Heartbeat"
                      ],
                      "enable": true
                  }
              }
            ]
        }
    ]
}
```

Используйте следующую команду, чтобы создать правило экспорта данных для определенного концентратора событий с помощью шаблона. Все таблицы экспортируются в указанное имя концентратора событий.

```
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "workspaceName": {
            "defaultValue": "workspace-name",
            "type": "String"
        },
        "workspaceLocation": {
            "defaultValue": "workspace-region",
            "type": "string"
        },
        "eventhubRuleName": {
            "defaultValue": "event-hub-rule-name",
            "type": "string"
        },
        "namespacesResourceId": {
            "defaultValue": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/resource-group-name/providers/microsoft.eventhub/namespaces/namespaces-name",
            "type": "String"
        },
        "eventhubName": {
            "defaultValue": "event-hub-name",
            "type": "string"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "microsoft.operationalinsights/workspaces",
            "apiVersion": "2020-08-01",
            "name": "[parameters('workspaceName')]",
            "location": "[parameters('workspaceLocation')]",
            "resources": [
              {
                  "type": "microsoft.operationalinsights/workspaces/dataexports",
                  "apiVersion": "2020-08-01",
                  "name": "[concat(parameters('workspaceName'), '/', parameters('eventhubRuleName'))]",
                  "dependsOn": [
                      "[resourceId('microsoft.operationalinsights/workspaces', parameters('workspaceName'))]"
                  ],
                  "properties": {
                      "destination": {
                          "resourceId": "[parameters('namespacesResourceId')]",
                          "metaData": {
                              "eventHubName": "[parameters('eventhubName')]"
                          }
                      },
                      "tableNames": [
                          "Usage",
                          "Heartbeat"
                      ],
                      "enable": true
                  }
              }
            ]
        }
    ]
}
```

---

## <a name="view-data-export-rule-configuration"></a>Просмотр конфигурации правила экспорта данных

# <a name="azure-portal"></a>[Портал Azure](#tab/portal)

Н/Д

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Недоступно

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Используйте следующую команду, чтобы просмотреть конфигурацию правила экспорта данных с помощью интерфейса командной строки.

```azurecli
az monitor log-analytics workspace data-export show --resource-group resourceGroupName --workspace-name workspaceName --name ruleName
```

# <a name="rest"></a>[REST](#tab/rest)

Используйте следующий запрос, чтобы просмотреть конфигурацию правила экспорта данных с помощью REST API. Запрос должен использовать авторизацию токена носителя.

```rest
GET https://management.azure.com/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/Microsoft.operationalInsights/workspaces/<workspace-name>/dataexports/<data-export-name>?api-version=2020-08-01
```

# <a name="template"></a>[Шаблон](#tab/json)

Н/Д

---

## <a name="disable-an-export-rule"></a>Отключение правила экспорта

# <a name="azure-portal"></a>[Портал Azure](#tab/portal)

Н/Д

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Недоступно

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Можно отключить правила экспорта, чтобы вы могли отменить экспорт, когда не нужно хранить данные в течение определенного периода, например при выполнении тестирования. Используйте следующую команду, чтобы отключить правило экспорта данных с помощью интерфейса командной строки.

```azurecli
az monitor log-analytics workspace data-export update --resource-group resourceGroupName --workspace-name workspaceName --name ruleName --enable false
```

# <a name="rest"></a>[REST](#tab/rest)

Можно отключить правила экспорта, чтобы вы могли отменить экспорт, когда не нужно хранить данные в течение определенного периода, например при выполнении тестирования. Используйте следующий запрос, чтобы отключить правило экспорта данных с помощью REST API. Запрос должен использовать авторизацию токена носителя.

```rest
PUT https://management.azure.com/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/Microsoft.operationalInsights/workspaces/<workspace-name>/dataexports/<data-export-name>?api-version=2020-08-01
Authorization: Bearer <token>
Content-type: application/json

{
    "properties": {
        "destination": {
            "resourceId": "/subscriptions/subscription-id/resourcegroups/resource-group-name/providers/Microsoft.Storage/storageAccounts/storage-account-name"
        },
        "tablenames": [
"table1",
    "table2" 
        ],
        "enable": false
    }
}
```

# <a name="template"></a>[Шаблон](#tab/json)

Можно отключить правила экспорта, чтобы вы могли отменить экспорт, когда не нужно хранить данные в течение определенного периода, например при выполнении тестирования. Задайте ```"enable": false``` в шаблоне значение, чтобы отключить экспорт данных.

---

## <a name="delete-an-export-rule"></a>Удаление правила экспорта

# <a name="azure-portal"></a>[Портал Azure](#tab/portal)

Н/Д

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Недоступно

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Используйте следующую команду, чтобы удалить правило экспорта данных с помощью интерфейса командной строки.

```azurecli
az monitor log-analytics workspace data-export delete --resource-group resourceGroupName --workspace-name workspaceName --name ruleName
```

# <a name="rest"></a>[REST](#tab/rest)

Используйте следующий запрос, чтобы удалить правило экспорта данных с помощью REST API. Запрос должен использовать авторизацию токена носителя.

```rest
DELETE https://management.azure.com/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/Microsoft.operationalInsights/workspaces/<workspace-name>/dataexports/<data-export-name>?api-version=2020-08-01
```

# <a name="template"></a>[Шаблон](#tab/json)

Н/Д

---

## <a name="view-all-data-export-rules-in-a-workspace"></a>Просмотр всех правил экспорта данных в рабочей области

# <a name="azure-portal"></a>[Портал Azure](#tab/portal)

Н/Д

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Недоступно

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Используйте следующую команду, чтобы просмотреть все правила экспорта данных в рабочей области с помощью интерфейса командной строки.

```azurecli
az monitor log-analytics workspace data-export list --resource-group resourceGroupName --workspace-name workspaceName
```

# <a name="rest"></a>[REST](#tab/rest)

Используйте следующий запрос, чтобы просмотреть все правила экспорта данных в рабочей области с помощью REST API. Запрос должен использовать авторизацию токена носителя.

```rest
GET https://management.azure.com/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/Microsoft.operationalInsights/workspaces/<workspace-name>/dataexports?api-version=2020-08-01
```

# <a name="template"></a>[Шаблон](#tab/json)

Н/Д

---

## <a name="unsupported-tables"></a>Неподдерживаемые таблицы
Если правило экспорта данных включает неподдерживаемую таблицу, то конфигурация будет выполнена, но данные для этой таблицы не будут экспортированы. Если таблица в дальнейшем поддерживается, в это время будут экспортированы ее данные.

Если правило экспорта данных включает таблицу, которая не существует, она завершится ошибкой "Таблица <tableName> не существует в рабочей области".


## <a name="supported-tables"></a>Поддерживаемые таблицы
Поддерживаемые таблицы в настоящее время ограничены указанными ниже. Все данные из таблицы будут экспортированы, если не указаны ограничения. Этот список будет обновлен по мере добавления поддержки дополнительных таблиц.


| Таблица | Ограничения |
|:---|:---|
| AADDomainServicesAccountLogon |  |
| ааддомаинсервицесаккаунтманажемент |  |
| AADDomainServicesDirectoryServiceAccess |  |
| ааддомаинсервицеслогонлогофф |  |
| ааддомаинсервицесполицичанже |  |
| AADDomainServicesPrivilegeUse |  |
| AADManagedIdentitySignInLogs |  |
| AADNonInteractiveUserSignInLogs |  |
| AADProvisioningLogs |  |
| AADServicePrincipalSignInLogs |  |
| ABSBotRequests |  |
| ACSBillingUsage |  |
| ACSSMSIncomingOperations |  |
| ADAssessmentRecommendation |  |
| адфактивитирун |  |
| адфпипелинерун |  |
| адфтригжеррун |  |
| ADReplicationResult |  |
| адсекуритяссессментрекоммендатион |  |
| ADTDigitalTwinsOperation |  |
| ADTEventRoutesOperation |  |
| ADTModelsOperation |  |
| ADTQueryOperation |  |
| ADXCommand |  |
| ADXQuery |  |
| AegDeliveryFailureLogs |  |
| AegPublishFailureLogs |  |
| Оповещение |  |
| амлонлининдпоинтконсолелог |  |
| апиманажементгатевайлогс |  |
| аппцентереррор |  |
| AppPlatformSystemLogs |  |
| AppServiceAppLogs |  |
| AppServiceAuditLogs |  |
| AppServiceConsoleLogs |  |
| AppServiceFileAuditLogs |  |
| AppServiceHTTPLogs |  |
| AppServicePlatformLogs |  |
| AuditLogs |  |
| аутоскаливалуатионслог |  |
| аутоскалескалеактионслог |  |
| AWSCloudTrail |  |
| азуреассессментрекоммендатион |  |
| AzureDevOpsAuditing |  |
| BehaviorAnalytics |  |
| BlockchainApplicationLog |  |
| BlockchainProxyLog |  |
| CommonSecurityLog |  |
| ComputerGroup |  |
| ConfigurationData | Частичная поддержка — некоторые данные принимаются через внутренние службы, которые не поддерживаются для экспорта. Эта часть в настоящее время отсутствует в экспорте. |
| ContainerImageInventory |  |
| ContainerInventory |  |
| ContainerLog |  |
| ContainerNodeInventory |  |
| ContainerServiceLog |  |
| кореазуребаккуп |  |
| датабрикксаккаунтс |  |
| датабрикксклустерс |  |
| датабрикксдбфс |  |
| датабрикксинстанцепулс |  |
| датабрикксжобс |  |
| датабриккснотебук |  |
| датабриккссекретс |  |
| датабриккссклпермиссионс |  |
| датабриккссш |  |
| датабрикксворкспаце |  |
| DnsEvents |  |
| DnsInventory |  |
| Dynamics365Activity |  |
| Событие | Частичная поддержка — некоторые данные из этой таблицы принимаются через учетную запись хранения. Эта часть в настоящее время отсутствует в экспорте. |
| ExchangeAssessmentRecommendation |  |
| FailedIngestion |  |
| FunctionAppLogs |  |
| Пульс |  |
| HuntingBookmark |  |
| InsightsMetrics | Частичная поддержка — некоторые данные принимаются через внутренние службы, которые не поддерживаются для экспорта. Эта часть в настоящее время отсутствует в экспорте. |
| интунеаудитлогс |  |
| IntuneDevices |  |
| IntuneOperationalLogs |  |
| KubeEvents |  |
| KubeHealth |  |
| KubeMonAgentEvents |  |
| KubeNodeInventory |  |
| KubePodInventory |  |
| KubeServices |  |
| LAQueryLogs |  |
| McasShadowItReporting |  |
| MicrosoftAzureBastionAuditLogs |  |
| MicrosoftDataShareReceivedSnapshotLog |  |
| MicrosoftDataShareSentSnapshotLog |  |
| MicrosoftHealthcareApisAuditLogs |  |
| NWConnectionMonitorPathResult |  |
| NWConnectionMonitorTestResult |  |
| OfficeActivity | Частичная поддержка — часть данных, принимаемых через веб-перехватчики из Office 365 в LA. Эта часть в настоящее время отсутствует в экспорте. |
| Операция | Частичная поддержка — некоторые данные принимаются через внутренние службы, которые не поддерживаются для экспорта. Эта часть в настоящее время отсутствует в экспорте. |
| Perf | Частичная поддержка — в настоящее время поддерживаются только данные о производительности Windows. В настоящее время в экспорте отсутствуют данные о производительности Linux. |
| повербидатасетстенант |  |
| повербидатасетсворкспаце |  |
| PowerBIDatasetsWorkspacePreview |  |
| сккмассессментрекоммендатион |  |
| скомассессментрекоммендатион |  |
| SecurityAlert |  |
| SecurityBaseline |  |
| SecurityBaselineSummary |  |
| секуритидетектион |  |
| SecurityEvent | Частичная поддержка — некоторые данные из этой таблицы принимаются через учетную запись хранения. Эта часть в настоящее время отсутствует в экспорте. |
| SecurityIncident |  |
| секуритиотравевент |  |
| SecurityNestedRecommendation |  |
| секуритирекоммендатион |  |
| сфбассессментрекоммендатион |  |
| сфбонлинеассессментрекоммендатион |  |
| шарепоинтонлинеассессментрекоммендатион |  |
| SignalRServiceDiagnosticLogs |  |
| SigninLogs |  |
| спассессментрекоммендатион |  |
| SQLAssessmentRecommendation |  |
| SucceededIngestion |  |
| SynapseBigDataPoolApplicationsEnded |  |
| SynapseBuiltinSqlPoolRequestsEnded |  |
| SynapseGatewayApiRequests |  |
| синапсеинтегратионактивитирунс |  |
| синапсеинтегратионпипелинерунс |  |
| синапсеинтегратионтригжеррунс |  |
| SynapseRbacOperations |  |
| SynapseSqlPoolDmsWorkers |  |
| SynapseSqlPoolExecRequests |  |
| SynapseSqlPoolRequestSteps |  |
| SynapseSqlPoolSqlRequests |  |
| SynapseSqlPoolWaits |  |
| Системный журнал | Частичная поддержка — некоторые данные из этой таблицы принимаются через учетную запись хранения. Эта часть в настоящее время отсутствует в экспорте. |
| ThreatIntelligenceIndicator |  |
| Update | Частичная поддержка — некоторые данные принимаются через внутренние службы, которые не поддерживаются для экспорта. Эта часть в настоящее время отсутствует в экспорте. |
| UpdateRunProgress |  |
| UpdateSummary |  |
| Использование |  |
| Watchlist |  |
| WindowsEvent |  |
| WindowsFirewall |  |
| Данные передачи | Частичная поддержка — некоторые данные принимаются через внутренние службы, которые не поддерживаются для экспорта. Эта часть в настоящее время отсутствует в экспорте. |
| WVDCheckpoints |  |
| WVDConnections |  |
| WVDErrors |  |
| WVDFeeds |  |
| WVDManagement |  |


## <a name="next-steps"></a>Дальнейшие действия

- [Запросите экспортированные данные из обозреватель данных Azure](../logs/azure-data-explorer-query-storage.md).
