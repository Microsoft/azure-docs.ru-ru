---
title: Собирайте & анализа журналов ресурсов
description: Записывайте и анализируйте события журнала ресурсов для реестра контейнеров Azure, такие как проверка подлинности, отправка образа и извлечение образа.
ms.topic: article
ms.date: 06/01/2020
ms.openlocfilehash: 952f239e94df6b3b21317985f56d3d7a999813fe
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100572007"
---
# <a name="azure-container-registry-logs-for-diagnostic-evaluation-and-auditing"></a>Журналы реестра контейнеров Azure для диагностической оценки и аудита

В этой статье объясняется, как получить данные журнала для реестра контейнеров Azure с помощью функций [Azure Monitor](../azure-monitor/overview.md). Azure Monitor собирает [журналы ресурсов](../azure-monitor/essentials/platform-logs-overview.md) (ранее называемые *журналами диагностики*) для событий, управляемых пользователем, в реестре. Собирайте и потребляют эти данные в соответствии с потребностями:

* Аудит событий проверки подлинности в реестре для обеспечения безопасности и соответствия требованиям 

* Предоставьте полный журнал действий для артефактов реестра, таких как события извлечения и извлечения, чтобы диагностировать проблемы с реестром. 

Сбор данных журнала ресурсов с помощью Azure Monitor может повлечь дополнительные затраты. См. [Azure Monitor цены](https://azure.microsoft.com/pricing/details/monitor/). 

## <a name="repository-events"></a>События репозитория

В настоящее время регистрируются следующие события уровня репозитория для образов и других артефактов:

* **Push**
* **Опрос**
* **Удалить тег**
* **Удалить** (включая события удаления репозитория)
* **Очистить тег** и **очистить манифест**

> [!NOTE]
> События очистки регистрируются только в том случае, если настроена [Политика хранения](container-registry-retention-policy.md) реестра.

## <a name="registry-resource-logs"></a>Журналы ресурсов реестра

Журналы ресурсов содержат сведения, созданные ресурсами Azure, которые описывают их внутреннюю работу. Для реестра контейнеров Azure журналы содержат события проверки подлинности и уровня репозитория, хранящиеся в следующих таблицах. 

* **Контаинеррегистрилогиневентс**  — события и состояние проверки подлинности реестра, включая входящий идентификатор и IP-адрес.
* **Контаинеррегистрирепоситоревентс** — такие операции, как отправка и Извлечение изображений и других артефактов в репозиториях реестра.
* **Азуреметрикс**  -  [Метрики реестра контейнеров](../azure-monitor/essentials/metrics-supported.md#microsoftcontainerregistryregistries) , такие как агрегированные push-уведомления и счетчики опрашивающей репликации.

Для операций данные журнала включают:
  * Состояние успеха или сбоя
  * Метки времени начала и окончания

Помимо журналов ресурсов, Azure предоставляет [Журнал действий](../azure-monitor/essentials/platform-logs-overview.md), отдельную запись на уровне подписки на события управления Azure, например создание или удаление реестра контейнеров.

## <a name="enable-collection-of-resource-logs"></a>Включение сбора журналов ресурсов

Сбор журналов ресурсов для реестра контейнеров по умолчанию отключен. Явно включите параметры диагностики для каждого реестра, который требуется отслеживать. Сведения о параметрах включения диагностики см. в статье [Создание параметров диагностики для сбора журналов и метрик платформы в Azure](../azure-monitor/essentials/diagnostic-settings.md).

Например, чтобы просмотреть журналы и метрики для реестра контейнеров практически в реальном времени в Azure Monitor, собирайте журналы ресурсов в рабочей области Log Analytics. Чтобы включить этот параметр диагностики с помощью портал Azure, выполните следующие действия.

1. Если у вас еще нет рабочей области, создайте ее с помощью [портал Azure](../azure-monitor/logs/quick-create-workspace.md). Чтобы максимально сокращать задержку в сборе данных, убедитесь, что Рабочая область находится в том **же регионе** , что и реестр контейнеров.
1. На портале выберите реестр и выберите **мониторинг > параметры диагностики > добавить параметр диагностики**.
1. Введите имя параметра и выберите **отправить в log Analytics**.
1. Выберите рабочую область для журналов диагностики реестра.
1. Выберите данные журнала, которые требуется отслеживать, и нажмите кнопку **сохранить**.

На следующем рисунке показано создание параметра диагностики для реестра с помощью портала.

![Включение параметров диагностики](media/container-registry-diagnostics-audit-logs/diagnostic-settings.png)

> [!TIP]
> Собирайте только необходимые данные, стоимость балансировки и потребности мониторинга. Например, если требуется только аудит событий проверки подлинности, выберите только журнал **контаинеррегистрилогиневентс** . 

## <a name="view-data-in-azure-monitor"></a>Просмотр данных в Azure Monitor

После включения сбора журналов диагностики в Log Analytics для отображения данных в Azure Monitor может потребоваться несколько минут. Чтобы просмотреть данные на портале, выберите реестр и выберите **мониторинг > журналы**. Выберите одну из таблиц, содержащих данные для реестра. 

Выполните запросы, чтобы просмотреть данные. Вы можете воспользоваться предоставленными примерами запросов или выполнить собственный запрос. Например, следующий запрос получает последние 24 часа данных из таблицы **контаинеррегистрирепоситоревентс** :

```Kusto
ContainerRegistryRepositoryEvents
| where TimeGenerated > ago(1d) 
```

На следующем рисунке показан пример выходных данных:

![Запрашивание данных журнала](media/container-registry-diagnostics-audit-logs/azure-monitor-query.png)

Руководство по использованию Log Analytics в портал Azure см. в статье Начало [работы с Azure Monitor log Analytics](../azure-monitor/logs/log-analytics-tutorial.md)или использование [демонстрационной среды](https://portal.loganalytics.io/demo)log Analytics. 

Дополнительные сведения о запросах журналов см. [в разделе Общие сведения о запросах журналов в Azure Monitor](../azure-monitor/logs/log-query-overview.md).

## <a name="query-examples"></a>Примеры запросов

### <a name="error-events-from-the-last-hour"></a>События ошибок за последний час

```Kusto
union Event, Syslog // Event table stores Windows event records, Syslog stores Linux records
| where TimeGenerated > ago(1h)
| where EventLevelName == "Error" // EventLevelName is used in the Event (Windows) records
    or SeverityLevel== "err" // SeverityLevel is used in Syslog (Linux) records
```

### <a name="100-most-recent-registry-events"></a>100. Последние события реестра

```Kusto
ContainerRegistryRepositoryEvents
| union ContainerRegistryLoginEvents
| top 100 by TimeGenerated
| project TimeGenerated, LoginServer, OperationName, Identity, Repository, DurationMs, Region , ResultType
```

### <a name="identity-of-user-or-object-that-deleted-repository"></a>Удостоверение пользователя или объекта, который удалил репозиторий

```Kusto
ContainerRegistryRepositoryEvents
| where OperationName contains "Delete"
| project LoginServer, OperationName, Repository, Identity, CallerIpAddress
```

### <a name="identity-of-user-or-object-that-deleted-tag"></a>Идентификатор пользователя или объекта, который удалил тег

```Kusto
ContainerRegistryRepositoryEvents
| where OperationName contains "Untag"
| project LoginServer, OperationName, Repository, Tag, Identity, CallerIpAddress
```

### <a name="repository-level-operation-failures"></a>Сбои операций на уровне репозитория

```kusto
ContainerRegistryRepositoryEvents 
| where ResultDescription contains "40"
| project TimeGenerated, OperationName, Repository, Tag, ResultDescription
```

### <a name="registry-authentication-failures"></a>Ошибки проверки подлинности реестра

```kusto
ContainerRegistryLoginEvents 
| where ResultDescription != "200"
| project TimeGenerated, Identity, CallerIpAddress, ResultDescription
```


## <a name="additional-log-destinations"></a>Дополнительные назначения журналов

Помимо отправки журналов в Log Analytics или в качестве альтернативы распространенным сценарием является выбор учетной записи хранения Azure в качестве назначения журнала. Чтобы архивировать журналы в службе хранилища Azure, создайте учетную запись хранения, прежде чем включать архивацию с помощью параметров диагностики.

Вы также можете выполнять потоковую передачу событий журнала диагностики в [концентратор событий Azure](../event-hubs/event-hubs-about.md). Центры событий способны принимать миллионы событий в секунду, позволяя преобразовать и сохранять их с помощью любого поставщика аналитики в реальном времени. 

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения об использовании [log Analytics](../azure-monitor/logs/log-analytics-tutorial.md) и создании [запросов журналов](../azure-monitor/logs/get-started-queries.md).
* Сведения о журналах платформы, доступных на разных уровнях Azure, см. в статье [Обзор журналов платформы Azure](../azure-monitor/essentials/platform-logs-overview.md) .