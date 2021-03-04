---
title: журналов ресурсов Azure.
description: Узнайте, как выполнять потоковую передачу журналов ресурсов Azure в рабочую область Log Analytics в Azure Monitor.
author: bwren
services: azure-monitor
ms.topic: conceptual
ms.date: 07/17/2019
ms.author: bwren
ms.openlocfilehash: cb4f1ecdada68218c104558a85277417641906f6
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102033018"
---
# <a name="azure-resource-logs"></a>журналов ресурсов Azure.
Журналы ресурсов Azure — это [журналы платформ](../essentials/platform-logs-overview.md) , которые позволяют получить представление об операциях, выполненных в рамках ресурса Azure. Содержимое журналов ресурсов зависит от службы и типа ресурса Azure. Журналы ресурсов по умолчанию не собираются. Необходимо создать параметр диагностики для каждого ресурса Azure, чтобы отправлять журналы ресурсов в Log Analytics рабочую область для использования с [журналами Azure Monitor](../logs/data-platform-logs.md), концентраторов событий Azure для пересылки за пределы Azure или в службу хранилища Azure для архивации.

Дополнительные сведения о создании параметров диагностики и [развертывании Azure Monitor в масштабе с помощью политики](../deploy-scale.md) Azure см. в статье [Создание параметров диагностики для отправки журналов и метрик платформы в разные места назначения](../essentials/diagnostic-settings.md) для получения дополнительных сведений об использовании политики Azure для автоматического создания параметров диагностики для каждого создаваемого ресурса Azure.

## <a name="send-to-log-analytics-workspace"></a>Отправка в рабочую область Log Analytics
 Отправляйте журналы ресурсов в рабочую область Log Analytics, чтобы включить функции [журналов Azure Monitor](../logs/data-platform-logs.md) , которые включают в себя следующее:

- Сопоставьте данные журнала ресурсов с другими данными мониторинга, собранными Azure Monitor.
- Объедините записи журнала из нескольких ресурсов Azure, подписок и клиентов в единое место для анализа вместе.
- Используйте запросы журналов для выполнения сложного анализа и получения подробных сведений о данных журнала.
- Используйте оповещения журнала со сложной логикой предупреждений.

[Создайте параметр диагностики](../essentials/diagnostic-settings.md) для отправки журналов ресурсов в рабочую область log Analytics. Эти данные хранятся в таблицах, как описано в разделе [Структура журналов Azure Monitor](../logs/data-platform-logs.md). Таблицы, используемые журналами ресурсов, зависят от типа коллекции, используемой ресурсом.

- Диагностика Azure — все данные, записанные в таблицу _AzureDiagnostics_ .
- Данные конкретного ресурса записываются в отдельную таблицу для каждой категории ресурса.

### <a name="azure-diagnostics-mode"></a>Режим диагностики Azure 
В этом режиме все данные из любого параметра диагностики будут собраны в таблицу _AzureDiagnostics_ . Это устаревший метод, используемый на сегодняшний день большинством служб Azure. Так как несколько типов ресурсов отправляют данные в одну и ту же таблицу, их схема является надмножеством схем всех различных собираемых типов данных.

Рассмотрим следующий пример, где параметры диагностики собираются в одной рабочей области для следующих типов данных:

- Журналы аудита службы 1 (со схемой, состоящей из столбцов A, B и C)  
- Журналы ошибок службы 1 (со схемой, состоящей из столбцов D, E и F)  
- Журналы аудита службы 2 (со схемой, состоящей из столбцов G, H и I)  

Таблица AzureDiagnostics будет выглядеть следующим образом:  

| ResourceProvider    | Категория     | Объект  | B  | C  | D  | E  | F  | G  | H  | I  |
| -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
| Microsoft. Service1 | AuditLogs    | x1 | Y1 | Z1 |    |    |    |    |    |    |
| Microsoft. Service1 | ErrorLogs    |    |    |    | q1 | W1 | e1 |    |    |    |
| Microsoft. S2 | AuditLogs    |    |    |    |    |    |    | J1 | k1 | L1 |
| Microsoft. Service1 | ErrorLogs    |    |    |    | q2 | W2 | e2 |    |    |    |
| Microsoft. S2 | AuditLogs    |    |    |    |    |    |    | J3 | k3 | Индекс |
| Microsoft. Service1 | AuditLogs    | x5 | y5 | z5 |    |    |    |    |    |    |
| ... |

### <a name="resource-specific"></a>Для конкретного ресурса
В этом режиме отдельные таблицы в выбранной рабочей области создаются для каждой категории, выбранной в параметре диагностики. Этот метод рекомендуется, поскольку он значительно упрощает работу с данными в запросах к журналу, обеспечивает лучшую возможность обнаружения схем и их структуры, повышает производительность как задержки приема, так и времени выполнения запросов, а также возможность предоставления прав RBAC Azure для определенной таблицы. Все службы Azure в конечном итоге будут перенесены в режим Resource-Specific. 

Приведенный выше пример привел бы к созданию трех таблиц:
 
- Таблица *Service1AuditLogs* выглядит следующим образом:

    | Поставщик ресурсов | Категория | Объект | B | C |
    | -- | -- | -- | -- | -- |
    | Service1 | AuditLogs | x1 | Y1 | Z1 |
    | Service1 | AuditLogs | x5 | y5 | z5 |
    | ... |

- Таблица *Service1ErrorLogs* выглядит следующим образом:  

    | Поставщик ресурсов | Категория | D | E | F |
    | -- | -- | -- | -- | -- | 
    | Service1 | ErrorLogs |  q1 | W1 | e1 |
    | Service1 | ErrorLogs |  q2 | W2 | e2 |
    | ... |

- Таблица *Service2AuditLogs* выглядит следующим образом:  

    | Поставщик ресурсов | Категория | G | H | I |
    | -- | -- | -- | -- | -- |
    | S2 | AuditLogs | J1 | k1 | L1|
    | S2 | AuditLogs | J3 | k3 | Индекс|
    | ... |



### <a name="select-the-collection-mode"></a>Выбор режима сбора
Большинство ресурсов Azure записывают данные в рабочую область в **службе диагностики Azure** или в **режиме отдельных ресурсов** , не предоставляя выбора. Дополнительные сведения о том, какой режим используется, см. в [документации по каждой службе](./resource-logs-schema.md) . Все службы Azure в конечном итоге будут использовать режим Resource-Specific. В рамках этого перехода некоторые ресурсы позволяют выбрать режим в параметре диагностики. Укажите режим конкретного ресурса для всех новых параметров диагностики, так как это упрощает управление данными и может помочь избежать сложных миграций позднее.
  
   ![Селектор режима параметров диагностики](media/resource-logs/diagnostic-settings-mode-selector.png)

> [!NOTE]
> Пример настройки режима сбора с помощью шаблона Resource Manager см. [в разделе примеры шаблонов диспетчер ресурсов для параметров диагностики в Azure Monitor](./resource-manager-diagnostic-settings.md#diagnostic-setting-for-recovery-services-vault).


Можно изменить существующий параметр диагностики на режим, зависящий от ресурса. В этом случае собранные данные останутся в таблице _AzureDiagnostics_ до тех пор, пока не будут удалены в соответствии с параметром хранения для рабочей области. В выделенной таблице будут собираться новые данные. Используйте оператор [Union](/azure/kusto/query/unionoperator) для запроса данных в обеих таблицах.

Перейдите к блогу об [обновлениях Azure](https://azure.microsoft.com/updates/) , чтобы просмотреть объявления о службах Azure, поддерживающих режим Resource-Specific.

### <a name="column-limit-in-azurediagnostics"></a>Ограничение столбца в AzureDiagnostics
Существует ограничение 500 свойств для любой таблицы в журналах Azure Monitor. После достижения этого ограничения все строки, содержащие данные со свойством, находящимся за пределами первых 500, будут удалены во время приема. Таблица *AzureDiagnostics* особенно подвержена этому ограничению, так как она включает свойства для всех служб Azure, записывающих в нее.

Если вы собираете журналы ресурсов из нескольких служб, _AzureDiagnostics_ может превысить это ограничение, и данные будут пропущены. До тех пор пока все службы Azure не будут поддерживать режим конкретного ресурса, необходимо настроить ресурсы для записи в несколько рабочих областей, чтобы снизить вероятность достижения ограничения в 500 столбцов.

### <a name="azure-data-factory"></a>Фабрика данных Azure
Фабрика данных Azure из-за подробного набора журналов — это служба, которая записывает большое количество столбцов и может привести к превышению предела _AzureDiagnostics_ . Для всех параметров диагностики, настроенных до включения режима для конкретного ресурса, будет создан новый столбец для каждого уникально именованного параметра пользователя в отношении любого действия. Будут созданы дополнительные столбцы из-за подробной природы входных и выходных данных действия.
 
Необходимо как можно скорее перенести журналы, чтобы использовать режим, зависящий от ресурса. Если вы не можете сделать это немедленно, следует изолировать журналы фабрики данных Azure в своей рабочей области, чтобы снизить вероятность того, что эти журналы будут влиять на другие типы журналов, собираемые в ваших рабочих областях.


## <a name="send-to-azure-event-hubs"></a>Отправка в концентраторы событий Azure
Отправляйте журналы ресурсов в концентратор событий, чтобы отправить их за пределы Azure, например в сторонние SIEM или другие решения log Analytics. Журналы ресурсов из концентраторов событий используются в формате JSON с элементом, `records` содержащим записи в каждой полезной нагрузке. Схема зависит от типа ресурса, как описано в разделе [Общие и зависящие от службы схемы для журналов ресурсов Azure](resource-logs-schema.md).

Ниже приведен пример выходных данных из концентраторов событий для журнала ресурсов.

```json
{
    "records": [
        {
            "time": "2019-07-15T18:00:22.6235064Z",
            "workflowId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/JOHNKEMTEST/PROVIDERS/MICROSOFT.LOGIC/WORKFLOWS/JOHNKEMTESTLA",
            "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/JOHNKEMTEST/PROVIDERS/MICROSOFT.LOGIC/WORKFLOWS/JOHNKEMTESTLA/RUNS/08587330013509921957/ACTIONS/SEND_EMAIL",
            "category": "WorkflowRuntime",
            "level": "Error",
            "operationName": "Microsoft.Logic/workflows/workflowActionCompleted",
            "properties": {
                "$schema": "2016-04-01-preview",
                "startTime": "2016-07-15T17:58:55.048482Z",
                "endTime": "2016-07-15T18:00:22.4109204Z",
                "status": "Failed",
                "code": "BadGateway",
                "resource": {
                    "subscriptionId": "00000000-0000-0000-0000-000000000000",
                    "resourceGroupName": "JohnKemTest",
                    "workflowId": "243aac67fe904cf195d4a28297803785",
                    "workflowName": "JohnKemTestLA",
                    "runId": "08587330013509921957",
                    "location": "westus",
                    "actionName": "Send_email"
                },
                "correlation": {
                    "actionTrackingId": "29a9862f-969b-4c70-90c4-dfbdc814e413",
                    "clientTrackingId": "08587330013509921958"
                }
            }
        },
        {
            "time": "2019-07-15T18:01:15.7532989Z",
            "workflowId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/JOHNKEMTEST/PROVIDERS/MICROSOFT.LOGIC/WORKFLOWS/JOHNKEMTESTLA",
            "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/JOHNKEMTEST/PROVIDERS/MICROSOFT.LOGIC/WORKFLOWS/JOHNKEMTESTLA/RUNS/08587330012106702630/ACTIONS/SEND_EMAIL",
            "category": "WorkflowRuntime",
            "level": "Information",
            "operationName": "Microsoft.Logic/workflows/workflowActionStarted",
            "properties": {
                "$schema": "2016-04-01-preview",
                "startTime": "2016-07-15T18:01:15.5828115Z",
                "status": "Running",
                "resource": {
                    "subscriptionId": "00000000-0000-0000-0000-000000000000",
                    "resourceGroupName": "JohnKemTest",
                    "workflowId": "243aac67fe904cf195d4a28297803785",
                    "workflowName": "JohnKemTestLA",
                    "runId": "08587330012106702630",
                    "location": "westus",
                    "actionName": "Send_email"
                },
                "correlation": {
                    "actionTrackingId": "042fb72c-7bd4-439e-89eb-3cf4409d429e",
                    "clientTrackingId": "08587330012106702632"
                }
            }
        }
    ]
}
```

## <a name="send-to-azure-storage"></a>Отправка в службу хранилища Azure
Отправьте журналы ресурсов в службу хранилища Azure, чтобы хранить их для архивирования. После создания параметра диагностики в учетной записи хранения создается контейнер хранилища, как только в одной из включенных категорий журнала будет происходить событие. Большие двоичные объекты в контейнере используют следующее соглашение об именовании:

```
insights-logs-{log category name}/resourceId=/SUBSCRIPTIONS/{subscription ID}/RESOURCEGROUPS/{resource group name}/PROVIDERS/{resource provider name}/{resource type}/{resource name}/y={four-digit numeric year}/m={two-digit numeric month}/d={two-digit numeric day}/h={two-digit 24-hour clock hour}/m=00/PT1H.json
```

Например, большой двоичный объект для группы безопасности сети может иметь имя, аналогичное следующему:

```
insights-logs-networksecuritygrouprulecounter/resourceId=/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/TESTRESOURCEGROUP/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUP/TESTNSG/y=2016/m=08/d=22/h=18/m=00/PT1H.json
```

Каждый большой двоичный объект PT1H.json содержит большой двоичный объект JSON с событиями, произошедшими в течение часа, указанного в URL-адресе этого объекта (например, h=12). В течение заданного часа события добавляются в файл PT1H.json по мере возникновения. Значение минуты (m = 00) всегда равно 00, так как события журнала ресурсов разбиваются на отдельные большие двоичные объекты в час.

В PT1H.jsфайла каждое событие хранится в следующем формате. При этом будет использоваться общая схема верхнего уровня, но она будет уникальной для каждой службы Azure, как описано в статье о [схемах журналов ресурсов](./resource-logs-schema.md).

``` JSON
{"time": "2016-07-01T00:00:37.2040000Z","systemId": "46cdbb41-cb9c-4f3d-a5b4-1d458d827ff1","category": "NetworkSecurityGroupRuleCounter","resourceId": "/SUBSCRIPTIONS/s1id1234-5679-0123-4567-890123456789/RESOURCEGROUPS/TESTRESOURCEGROUP/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/TESTNSG","operationName": "NetworkSecurityGroupCounters","properties": {"vnetResourceGuid": "{12345678-9012-3456-7890-123456789012}","subnetPrefix": "10.3.0.0/24","macAddress": "000123456789","ruleName": "/subscriptions/ s1id1234-5679-0123-4567-890123456789/resourceGroups/testresourcegroup/providers/Microsoft.Network/networkSecurityGroups/testnsg/securityRules/default-allow-rdp","direction": "In","type": "allow","matchedConnections": 1988}}
```

> [!NOTE]
> Журналы платформы записываются в хранилище BLOB-объектов с помощью [строк JSON](http://jsonlines.org/), где каждое событие является линией, а символ новой строки обозначает новое событие. Этот формат был реализован в ноябре 2018. До этой даты журналы были записаны в хранилище BLOB-объектов в виде массива записей JSON, как описано в разделе [Подготовка к изменению формата для Azure Monitor журналов платформы, архивов которых является учетная запись хранения](resource-logs-blob-format.md).


## <a name="next-steps"></a>Дальнейшие действия

* [Дополнительные сведения см. в статье о журналах ресурсов](../essentials/platform-logs-overview.md).
* [Создайте параметры диагностики для отправки журналов платформы и метрик в разные места назначения](../essentials/diagnostic-settings.md).