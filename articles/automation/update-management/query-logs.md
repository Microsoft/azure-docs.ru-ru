---
title: Запрос журналов Управления обновлениями из службы автоматизации Azure
description: В этой статье рассказывается, как запрашивать журналы Управления обновлениями в рабочей области Log Analytics.
services: automation
ms.subservice: update-management
ms.date: 09/24/2020
ms.topic: conceptual
ms.openlocfilehash: 5eb0c7d72896cc9a27907743b1b9c3d5a40614dd
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100592856"
---
# <a name="query-update-management-logs"></a>Запрос на получение журналов Управления обновлениями

Помимо сведений, предоставленных во время развертывания Управления обновлениями, можно выполнять поиск в журналах, хранящихся в рабочей области Log Analytics. Чтобы выполнить поиск в журналах из учетной записи службы автоматизации, выберите **Управление обновлениями** и откройте рабочую область Log Analytics, связанную с развертыванием.

Можно также настроить запросы к журналам или использовать их из других клиентов. См. [документацию по API поиска Log Analytics](https://dev.loganalytics.io/).

## <a name="query-update-records"></a>Запрос записей обновлений

Решение "Управление обновлениями" собирает записи для виртуальных машин Windows и Linux и типов данных, которые приводятся в результатах поиска по журналам. Эти записи описываются в разделах ниже.

### <a name="query-required-updates"></a>Запрос требуемых обновлений

Создается запись с типом `RequiredUpdate`, которая представляет обновления, необходимые для компьютера. У этих записей есть свойства, приведенные в таблице ниже.

| Свойство | Описание |
|----------|-------------|
| Компьютер | Полное доменное имя компьютера. |
| KBID | Идентификатор статьи базы знаний об обновлении Windows. |
| ManagementGroupName | Имя группы управления Operations Manager или рабочей области Log Analytics. |
| Продукт | Продукты, к которым применимо обновление. |
| PublishDate | Дата готовности обновления к скачиванию и установке из Центра обновления Windows. |
| Сервер | | 
| SourceHealthServiceId | Уникальный идентификатор, представляющий идентификатор агента Log Analytics в Windows. |
| SourceSystem | *OperationsManager* |
| TenantId | Уникальный идентификатор, представляющий экземпляр Azure Active Directory вашей организации. |
| TimeGenerated | Дата и время создания записи |
| Тип | *Update* |
| UpdateClassification | Указывает тип применимых обновлений. Для Windows:<br> *Критические обновления*<br> *Обновления для системы безопасности*<br> *Накопительные пакеты обновления*<br> *Пакеты дополнительных компонентов*<br> *Пакеты обновления*<br> *Обновления определений*<br> *Инструменты*<br> *Обновления.* Для Linux:<br> *Критические обновления и обновления для системы безопасности*<br> *Другое* |
| UpdateSeverity | Оценка серьезности уязвимости. Возможны следующие значения.<br> *Критическая*<br> *Важно*<br> *Средняя*<br> *Низкая* |
| UpdateTitle | Название обновления.|

### <a name="query-update-record"></a>Запрос записи обновления

Создается запись с типом `Update`, которая представляет доступные обновления и состояние их установки для компьютера. У этих записей есть свойства, приведенные в таблице ниже.

| Свойство | Описание |
|----------|-------------|
| ApprovalSource | Применяется только к операционной системе Windows. Источник утверждения для записи. Значение — Центр обновления Майкрософт. |
| Approved | Значение true, если запись утверждена, или false в противном случае. |
| Классификация | Классификация утверждения. Значение — "Обновления". |
| Компьютер | Полное доменное имя компьютера. |
| ComputerEnvironment | Среда. Возможные значения: "Azure" или "Не относящаяся к Azure". |
| MSRCBulletinID | Идентификационный номер бюллетеня по безопасности. |
| MSRCSeverity | Оценка серьезности уязвимости. Возможны следующие значения.<br> Critical<br> Важно!<br> Средний<br> Низкий |  
| KBID | Идентификатор статьи базы знаний об обновлении Windows. |
| ManagementGroupName | Имя группы управления Operations Manager или рабочей области Log Analytics. |
| UpdateID | Уникальный идентификатор обновления программного обеспечения. |
| RevisionNumber | Номер определенной редакции обновления. |
| Необязательно | Значение true, если запись является необязательной, или false в противном случае. |
| RebootBehavior | Необходимость в перезагрузке после установки или удаления обновления. |
| _ResourceId | Уникальный идентификатор ресурса, связанного с записью. |
| Тип | Тип записи. Значение — "обновление". |
| VMUUID | Уникальный идентификатор виртуальной машины. |
| MG | Уникальный идентификатор группы управления или рабочей области Log Analytics. |
| TenantId | Уникальный идентификатор, представляющий экземпляр Azure Active Directory вашей организации. |
| SourceSystem | Исходная система записи. Значение равно `OperationsManager`. |
| TimeGenerated | Дата и время создания записи. |
| SourceComputerId | Уникальный идентификатор, представляющий исходный компьютер. |
| Title | Название обновления. |
| PublishedDate (UTC) | Дата готовности обновления к скачиванию и установке из Центра обновления Windows.  |
| UpdateState | Текущее состояние обновления. |
| Продукт | Продукты, к которым применимо обновление. |
| SubscriptionId | Уникальный идентификатор подписки Azure. |
| ResourceGroup | Имя группы ресурсов, к которой принадлежит ресурс. |
| ResourceProvider | Поставщик ресурса. |
| Ресурс | Имя ресурса. |
| ResourceType | Тип ресурса. |

### <a name="query-update-agent-record"></a>Запрос записи агента обновления

Создается запись типа `UpdateAgent`, которая предоставляет сведения об агенте обновления на компьютере. У этих записей есть свойства, приведенные в таблице ниже.

| Свойство | Описание |
|----------|-------------|
| AgeofOldestMissingRequiredUpdate | |
| AutomaticUpdateEnabled | |
| Компьютер | Полное доменное имя компьютера. |
| DaySinceLastUpdateBucket | |
| ManagementGroupName | Имя группы управления Operations Manager или рабочей области Log Analytics. |
| OSVersion | Версия операционной системы. |
| Сервер | |
| SourceHealthServiceId | Уникальный идентификатор, представляющий идентификатор агента Log Analytics в Windows. |
| SourceSystem | Исходная система записи. Значение равно `OperationsManager`. |
| TenantId | Уникальный идентификатор, представляющий экземпляр Azure Active Directory вашей организации. |
| TimeGenerated | Дата и время создания записи. |
| Тип | Тип записи. Значение — "обновление". |
| WindowsUpdateAgentVersion | Версия агента Центра обновления Windows. |
| WSUSServer | Ошибки, помогающие в устранении неполадок с агентом Центра обновления Windows. |

### <a name="query-update-deployment-status-record"></a>Запрос записи о состоянии развертывания обновления

Создается запись типа `UpdateRunProgress`, которая предоставляет состояние запланированного развертывания обновления на компьютере. У этих записей есть свойства, приведенные в таблице ниже.

| Свойство | Описание |
|----------|-------------|
| Компьютер | Полное доменное имя компьютера. |
| ComputerEnvironment | Среда. Значения: "Azure" или "Не относящаяся к Azure". |
| CorrelationId | Уникальный идентификатор выполнения задания runbook для обновления. |
| EndTime | Время завершения процесса синхронизации. *Это свойство в настоящее время не используется. См. TimeGenerated.* |
| ErrorResult | Код ошибки Центра обновления Windows, генерируемый при сбое установки обновления. |
| InstallationStatus | Возможные состояния установки обновления на клиентском компьютере:<br> `NotStarted` — задание еще не активировано.<br> `FailedToStart` — не удается запустить задание на компьютере.<br> `Failed` — задание запущено, но завершилось с исключением.<br> `InProgress` — задание выполняется.<br> `MaintenanceWindowExceeded` — выполнение не завершилось, но период обслуживания истек.<br> `Succeeded` — задание успешно выполнено.<br> `InstallFailed` — не удалось успешно установить обновление.<br> `NotIncluded`<br> `Excluded` |
| KBID | Идентификатор статьи базы знаний об обновлении Windows. |
| ManagementGroupName | Имя группы управления Operations Manager или рабочей области Log Analytics. |
| OSType | Тип операционной системы. Значения: Windows или Linux. |
| Продукт | Продукты, к которым применимо обновление. |
| Ресурс | Имя ресурса. |
| ResourceId | Уникальный идентификатор ресурса, связанного с записью. |
| ResourceProvider | Поставщик ресурса. |
| ResourceType | Тип ресурса. |
| SourceComputerId | Уникальный идентификатор, представляющий исходный компьютер. |
| SourceSystem | Исходная система записи. Значение равно `OperationsManager`. |
| StartTime | Запланированное время установки обновления. *Это свойство в настоящее время не используется. См. TimeGenerated.* |
| SubscriptionId | Уникальный идентификатор подписки Azure. |
| SucceededOnRetry | Значение, указывающее, произошла ли ошибка при первой попытке выполнения обновления и является ли текущая операция повторной попыткой. |
| TimeGenerated | Дата и время создания записи. |
| Title | Название обновления. |
| Тип | Тип обновления. Значение равно `UpdateRunProgress`. |
| UpdateId | Уникальный идентификатор обновления программного обеспечения. |
| VMUUID | Уникальный идентификатор виртуальной машины. |
| ResourceId | Уникальный идентификатор ресурса, связанного с записью. |

### <a name="query-update-summary-record"></a>Запрос сводной записи обновления

Создается запись типа `UpdateSummary`, которая предоставляет сводку обновлений по компьютерам. У этих записей есть свойства, приведенные в таблице ниже.

| Свойство | Описание |
|----------|-------------|
| Компьютер | Полное доменное имя компьютера. |
| ComputerEnvironment | Среда. Значения: "Azure" или "Не относящаяся к Azure". |
| CriticalUpdatesMissing | Число отсутствующих критически важных обновлений, применимых к системе. |
| ManagementGroupName | Имя группы управления Operations Manager или рабочей области Log Analytics. |
| NETRuntimeVersion | Версия .NET Framework, установленная на компьютере с ОС Windows. |
| OldestMissingSecurityUpdateBucket | Описатель самого раннего отсутствующего контейнера безопасности. Возможны следующие значения.<br> "Недавно", если значение меньше 30 дней<br> 30 дней назад<br> 60 дней назад<br> 90 дней назад<br> 120 дней назад<br> 150 дней назад<br> 180 дней назад<br> "Старше", если значение больше 180 дней. |
| OldestMissingSecurityUpdateInDays | Возраст самого раннего обнаруженного обновления, которое не было установлено. |
| OsVersion | Версия операционной системы. |
| OtherUpdatesMissing | Число обнаруженных отсутствующих обновлений. |
| Ресурс | Имя ресурса записи. |
| ResourceGroup | Имя группы ресурсов, которая содержит ресурс. |
| ResourceId | Уникальный идентификатор ресурса, связанного с записью. |
| ResourceProvider | Поставщик ресурса. |
| ResourceType | Тип ресурса. |
| RestartPending | Значение true, если ожидается перезапуск, или false в противном случае. |
| SecurityUpdatesMissing | Число отсутствующих применимых обновлений для системы безопасности.|
| SourceComputerId | Уникальный идентификатор виртуальной машины. |
| SourceSystem | Исходная система записи. Значение равно `OpsManager`. |
| SubscriptionId | Уникальный идентификатор подписки Azure. |
| TimeGenerated | Дата и время создания записи. |
| TotalUpdatesMissing | Общее число отсутствующих применимых обновлений. |
| Тип | Тип записи. Значение равно `UpdateSummary`. |
| VMUUID | Уникальный идентификатор виртуальной машины. |
| WindowsUpdateAgentVersion | Версия агента Центра обновления Windows. |
| WindowsUpdateSetting | Состояние агента Центра обновления Windows. Возможны следующие значения:<br> `Scheduled installation`<br> `Notify before installation`<br> `Error returned from unhealthy WUA agent` |
| WSUSServer | Ошибки, помогающие в устранении неполадок с агентом Центра обновления Windows. |
| _ResourceId | Уникальный идентификатор ресурса, связанного с записью. |

## <a name="sample-queries"></a>Примеры запросов

В следующих разделах приведены примеры запросов журнала для записей обновлений, которые собираются для Управления обновлениями.

### <a name="confirm-that-non-azure-machines-are-enabled-for-update-management"></a>Подтверждение включения Управления обновлениями для виртуальных машин, не относящихся к Azure

Чтобы подтвердить, что подключенные напрямую виртуальные машины взаимодействуют с журналами Azure Monitor, выполните один из приведенных ниже поисков по журналу.

#### <a name="linux"></a>Linux

```loganalytics
Heartbeat
| where OSType == "Linux" | summarize arg_max(TimeGenerated, *) by SourceComputerId | top 500000 by Computer asc | render table
```

#### <a name="windows"></a>Windows

```loganalytics
Heartbeat
| where OSType == "Windows" | summarize arg_max(TimeGenerated, *) by SourceComputerId | top 500000 by Computer asc | render table
```

Чтобы проверить подключение агента к журналам Azure Monitor на компьютере с Windows, ознакомьтесь со следующей информацией:

1. На панели управления откройте **Microsoft Monitoring Agent**. На вкладке **Служба анализа журналов Azure** агент отображает следующее сообщение: **The Microsoft Monitoring Agent has successfully connected to Log Analytics** (Microsoft Monitoring Agent успешно подключен к Log Analytics).

1. Откройте журнал событий Windows. Перейдите к **журналам приложения и служб или Operations Manager** и выполните поиск идентификатора события 3000 или 5002 в исходном **соединителе службы**. Эти события указывают, что компьютер был зарегистрирован для рабочей области Log Analytics и получает конфигурации.

Если агент не может взаимодействовать с журналами Azure Monitor и настроен на взаимодействие с Интернетом через брандмауэр или прокси-сервер, подтвердите корректную настройку брандмауэра или прокси-сервера. Чтобы узнать, как проверить это, ознакомьтесь со статьей [Подключение компьютеров Windows к службе Log Analytics в Azure](../../azure-monitor/agents/agent-windows.md) или [Настройка агента Log Analytics для компьютеров Linux в гибридной среде](../../azure-monitor/vm/quick-collect-linux-computer.md).

> [!NOTE]
> Если при включении Управления обновлениями системы Linux настроены для взаимодействия с прокси-сервером или шлюзом Log Analytics, обновите разрешения `proxy.conf`, чтобы предоставить группе omiuser разрешение на чтение файла. Для этого выполните следующие команды.
>
> `sudo chown omsagent:omiusers /etc/opt/microsoft/omsagent/proxy.conf`
> `sudo chmod 644 /etc/opt/microsoft/omsagent/proxy.conf`

Добавленные агенты Linux отобразят состояние **обновления** после оценки. Этот процесс может занять до 6 часов.

Сведения о том, как подтвердить, что группа управления Operations Manager взаимодействует с журналами Azure Monitor, см. в разделе [Проверка интеграции Operations Manager с журналами Azure Monitor](../../azure-monitor/agents/om-agents.md#validate-operations-manager-integration-with-azure-monitor).

### <a name="single-azure-vm-assessment-queries-windows"></a>Запросы оценки одной виртуальной машины Azure (Windows)

Замените значение VMUUID на GUID запрашиваемой виртуальной машины. Вы можете найти нужный VMUUID, выполнив следующий запрос в журналах Azure Monitor: `Update | where Computer == "<machine name>" | summarize by Computer, VMUUID`

#### <a name="missing-updates-summary"></a>Сводка об отсутствующих обновлениях

```loganalytics
Update
| where TimeGenerated>ago(14h) and OSType!="Linux" and (Optional==false or Classification has "Critical" or Classification has "Security") and VMUUID=~"b08d5afa-1471-4b52-bd95-a44fea6e4ca8"
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, Approved) by Computer, SourceComputerId, UpdateID
| where UpdateState=~"Needed" and Approved!=false
| summarize by UpdateID, Classification
| summarize allUpdatesCount=count(), criticalUpdatesCount=countif(Classification has "Critical"), securityUpdatesCount=countif(Classification has "Security"), otherUpdatesCount=countif(Classification !has "Critical" and Classification !has "Security")
```

#### <a name="missing-updates-list"></a>Список отсутствующих обновлений

```loganalytics
Update
| where TimeGenerated>ago(14h) and OSType!="Linux" and (Optional==false or Classification has "Critical" or Classification has "Security") and VMUUID=~"8bf1ccc6-b6d3-4a0b-a643-23f346dfdf82"
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, Title, KBID, PublishedDate, Approved) by Computer, SourceComputerId, UpdateID
| where UpdateState=~"Needed" and Approved!=false
| project-away UpdateState, Approved, TimeGenerated
| summarize computersCount=dcount(SourceComputerId, 2), displayName=any(Title), publishedDate=min(PublishedDate), ClassificationWeight=max(iff(Classification has "Critical", 4, iff(Classification has "Security", 2, 1))) by id=strcat(UpdateID, "_", KBID), classification=Classification, InformationId=strcat("KB", KBID), InformationUrl=iff(isnotempty(KBID), strcat("https://support.microsoft.com/kb/", KBID), ""), osType=2
| sort by ClassificationWeight desc, computersCount desc, displayName asc
| extend informationLink=(iff(isnotempty(InformationId) and isnotempty(InformationUrl), toobject(strcat('{ "uri": "', InformationUrl, '", "text": "', InformationId, '", "target": "blank" }')), toobject('')))
| project-away ClassificationWeight, InformationId, InformationUrl
```

### <a name="single-azure-vm-assessment-queries-linux"></a>Запросы оценки одной виртуальной машины Azure (Linux)

В некоторых дистрибутивах Linux наблюдается несоответствие [порядка байтов](https://en.wikipedia.org/wiki/Endianness) между значениями VMUUID, которое поступает из Azure Resource Manager и хранится в журналах Azure Monitor. Следующий запрос проверяет совпадение по порядку байтов. Чтобы вернулись правильные результаты, замените значения VMUUID форматом GUID с прямым и обратным порядком. Вы можете найти нужный VMUUID, выполнив следующий запрос в журналах Azure Monitor: `Update | where Computer == "<machine name>"
| summarize by Computer, VMUUID`

#### <a name="missing-updates-summary"></a>Сводка об отсутствующих обновлениях

```loganalytics
Update
| where TimeGenerated>ago(5h) and OSType=="Linux" and (VMUUID=~"625686a0-6d08-4810-aae9-a089e68d4911" or VMUUID=~"a0865662-086d-1048-aae9-a089e68d4911")
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification) by Computer, SourceComputerId, Product, ProductArch
| where UpdateState=~"Needed"
| summarize by Product, ProductArch, Classification
| summarize allUpdatesCount=count(), criticalUpdatesCount=countif(Classification has "Critical"), securityUpdatesCount=countif(Classification has "Security"), otherUpdatesCount=countif(Classification !has "Critical" and Classification !has "Security")
```

#### <a name="missing-updates-list"></a>Список отсутствующих обновлений

```loganalytics
Update
| where TimeGenerated>ago(5h) and OSType=="Linux" and (VMUUID=~"625686a0-6d08-4810-aae9-a089e68d4911" or VMUUID=~"a0865662-086d-1048-aae9-a089e68d4911")
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, BulletinUrl, BulletinID) by Computer, SourceComputerId, Product, ProductArch
| where UpdateState=~"Needed"
| project-away UpdateState, TimeGenerated
| summarize computersCount=dcount(SourceComputerId, 2), ClassificationWeight=max(iff(Classification has "Critical", 4, iff(Classification has "Security", 2, 1))) by id=strcat(Product, "_", ProductArch), displayName=Product, productArch=ProductArch, classification=Classification, InformationId=BulletinID, InformationUrl=tostring(split(BulletinUrl, ";", 0)[0]), osType=1
| sort by ClassificationWeight desc, computersCount desc, displayName asc
| extend informationLink=(iff(isnotempty(InformationId) and isnotempty(InformationUrl), toobject(strcat('{ "uri": "', InformationUrl, '", "text": "', InformationId, '", "target": "blank" }')), toobject('')))
| project-away ClassificationWeight, InformationId, InformationUrl

```

### <a name="multi-vm-assessment-queries"></a>Запросы для оценки нескольких виртуальных машин

#### <a name="computers-summary"></a>Сводка по компьютерам

```loganalytics
Heartbeat
| where TimeGenerated>ago(12h) and OSType=~"Windows" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
| where Solutions has "updates"
| distinct SourceComputerId
| join kind=leftouter
(
    Update
    | where TimeGenerated>ago(14h) and OSType!="Linux"
    | summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Approved, Optional, Classification) by SourceComputerId, UpdateID
    | distinct SourceComputerId, Classification, UpdateState, Approved, Optional
    | summarize WorstMissingUpdateSeverity=max(iff(UpdateState=~"Needed" and (Optional==false or Classification has "Critical" or Classification has "Security") and Approved!=false, iff(Classification has "Critical", 4, iff(Classification has "Security", 2, 1)), 0)) by SourceComputerId
)
on SourceComputerId
| extend WorstMissingUpdateSeverity=coalesce(WorstMissingUpdateSeverity, -1)
| summarize computersBySeverity=count() by WorstMissingUpdateSeverity
| union (Heartbeat
| where TimeGenerated>ago(12h) and OSType=="Linux" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
| where Solutions has "updates"
| distinct SourceComputerId
| join kind=leftouter
(
    Update
    | where TimeGenerated>ago(5h) and OSType=="Linux"
    | summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification) by SourceComputerId, Product, ProductArch
    | distinct SourceComputerId, Classification, UpdateState
    | summarize WorstMissingUpdateSeverity=max(iff(UpdateState=~"Needed", iff(Classification has "Critical", 4, iff(Classification has "Security", 2, 1)), 0)) by SourceComputerId
)
on SourceComputerId
| extend WorstMissingUpdateSeverity=coalesce(WorstMissingUpdateSeverity, -1)
| summarize computersBySeverity=count() by WorstMissingUpdateSeverity)
| summarize assessedComputersCount=sumif(computersBySeverity, WorstMissingUpdateSeverity>-1), notAssessedComputersCount=sumif(computersBySeverity, WorstMissingUpdateSeverity==-1), computersNeedCriticalUpdatesCount=sumif(computersBySeverity, WorstMissingUpdateSeverity==4), computersNeedSecurityUpdatesCount=sumif(computersBySeverity, WorstMissingUpdateSeverity==2), computersNeedOtherUpdatesCount=sumif(computersBySeverity, WorstMissingUpdateSeverity==1), upToDateComputersCount=sumif(computersBySeverity, WorstMissingUpdateSeverity==0)
| summarize assessedComputersCount=sum(assessedComputersCount), computersNeedCriticalUpdatesCount=sum(computersNeedCriticalUpdatesCount),  computersNeedSecurityUpdatesCount=sum(computersNeedSecurityUpdatesCount), computersNeedOtherUpdatesCount=sum(computersNeedOtherUpdatesCount), upToDateComputersCount=sum(upToDateComputersCount), notAssessedComputersCount=sum(notAssessedComputersCount)
| extend allComputersCount=assessedComputersCount+notAssessedComputersCount
```

#### <a name="missing-updates-summary"></a>Сводка об отсутствующих обновлениях

```loganalytics
Update
| where TimeGenerated>ago(5h) and OSType=="Linux" and SourceComputerId in ((Heartbeat
| where TimeGenerated>ago(12h) and OSType=="Linux" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
| where Solutions has "updates"
| distinct SourceComputerId))
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification) by Computer, SourceComputerId, Product, ProductArch
| where UpdateState=~"Needed"
| summarize by Product, ProductArch, Classification
| union (Update
| where TimeGenerated>ago(14h) and OSType!="Linux" and (Optional==false or Classification has "Critical" or Classification has "Security") and SourceComputerId in ((Heartbeat
| where TimeGenerated>ago(12h) and OSType=~"Windows" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
| where Solutions has "updates"
| distinct SourceComputerId))
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, Approved) by Computer, SourceComputerId, UpdateID
| where UpdateState=~"Needed" and Approved!=false
| summarize by UpdateID, Classification )
| summarize allUpdatesCount=count(), criticalUpdatesCount=countif(Classification has "Critical"), securityUpdatesCount=countif(Classification has "Security"), otherUpdatesCount=countif(Classification !has "Critical" and Classification !has "Security")
```

#### <a name="computers-list"></a>Список компьютеров

```loganalytics
Heartbeat
| where TimeGenerated>ago(12h) and OSType=="Linux" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions, Computer, ResourceId, ComputerEnvironment, VMUUID) by SourceComputerId
| where Solutions has "updates"
| extend vmuuId=VMUUID, azureResourceId=ResourceId, osType=1, environment=iff(ComputerEnvironment=~"Azure", 1, 2), scopedToUpdatesSolution=true, lastUpdateAgentSeenTime=""
| join kind=leftouter
(
    Update
    | where TimeGenerated>ago(5h) and OSType=="Linux" and SourceComputerId in ((Heartbeat
    | where TimeGenerated>ago(12h) and OSType=="Linux" and notempty(Computer)
    | summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
    | where Solutions has "updates"
    | distinct SourceComputerId))
    | summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, Product, Computer, ComputerEnvironment) by SourceComputerId, Product, ProductArch
    | summarize Computer=any(Computer), ComputerEnvironment=any(ComputerEnvironment), missingCriticalUpdatesCount=countif(Classification has "Critical" and UpdateState=~"Needed"), missingSecurityUpdatesCount=countif(Classification has "Security" and UpdateState=~"Needed"), missingOtherUpdatesCount=countif(Classification !has "Critical" and Classification !has "Security" and UpdateState=~"Needed"), lastAssessedTime=max(TimeGenerated), lastUpdateAgentSeenTime="" by SourceComputerId
    | extend compliance=iff(missingCriticalUpdatesCount > 0 or missingSecurityUpdatesCount > 0, 2, 1)
    | extend ComplianceOrder=iff(missingCriticalUpdatesCount > 0 or missingSecurityUpdatesCount > 0 or missingOtherUpdatesCount > 0, 1, 3)
)
on SourceComputerId
| project id=SourceComputerId, displayName=Computer, sourceComputerId=SourceComputerId, scopedToUpdatesSolution=true, missingCriticalUpdatesCount=coalesce(missingCriticalUpdatesCount, -1), missingSecurityUpdatesCount=coalesce(missingSecurityUpdatesCount, -1), missingOtherUpdatesCount=coalesce(missingOtherUpdatesCount, -1), compliance=coalesce(compliance, 4), lastAssessedTime, lastUpdateAgentSeenTime, osType=1, environment=iff(ComputerEnvironment=~"Azure", 1, 2), ComplianceOrder=coalesce(ComplianceOrder, 2)
| union(Heartbeat
| where TimeGenerated>ago(12h) and OSType=~"Windows" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions, Computer, ResourceId, ComputerEnvironment, VMUUID) by SourceComputerId
| where Solutions has "updates"
| extend vmuuId=VMUUID, azureResourceId=ResourceId, osType=2, environment=iff(ComputerEnvironment=~"Azure", 1, 2), scopedToUpdatesSolution=true, lastUpdateAgentSeenTime=""
| join kind=leftouter
(
    Update
    | where TimeGenerated>ago(14h) and OSType!="Linux" and SourceComputerId in ((Heartbeat
    | where TimeGenerated>ago(12h) and OSType=~"Windows" and notempty(Computer)
    | summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
    | where Solutions has "updates"
    | distinct SourceComputerId))
    | summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, Title, Optional, Approved, Computer, ComputerEnvironment) by Computer, SourceComputerId, UpdateID
    | summarize Computer=any(Computer), ComputerEnvironment=any(ComputerEnvironment), missingCriticalUpdatesCount=countif(Classification has "Critical" and UpdateState=~"Needed" and Approved!=false), missingSecurityUpdatesCount=countif(Classification has "Security" and UpdateState=~"Needed" and Approved!=false), missingOtherUpdatesCount=countif(Classification !has "Critical" and Classification !has "Security" and UpdateState=~"Needed" and Optional==false and Approved!=false), lastAssessedTime=max(TimeGenerated), lastUpdateAgentSeenTime="" by SourceComputerId
    | extend compliance=iff(missingCriticalUpdatesCount > 0 or missingSecurityUpdatesCount > 0, 2, 1)
    | extend ComplianceOrder=iff(missingCriticalUpdatesCount > 0 or missingSecurityUpdatesCount > 0 or missingOtherUpdatesCount > 0, 1, 3)
)
on SourceComputerId
| project id=SourceComputerId, displayName=Computer, sourceComputerId=SourceComputerId, scopedToUpdatesSolution=true, missingCriticalUpdatesCount=coalesce(missingCriticalUpdatesCount, -1), missingSecurityUpdatesCount=coalesce(missingSecurityUpdatesCount, -1), missingOtherUpdatesCount=coalesce(missingOtherUpdatesCount, -1), compliance=coalesce(compliance, 4), lastAssessedTime, lastUpdateAgentSeenTime, osType=2, environment=iff(ComputerEnvironment=~"Azure", 1, 2), ComplianceOrder=coalesce(ComplianceOrder, 2) )
| order by ComplianceOrder asc, missingCriticalUpdatesCount desc, missingSecurityUpdatesCount desc, missingOtherUpdatesCount desc, displayName asc
| project-away ComplianceOrder
```

#### <a name="missing-updates-list"></a>Список отсутствующих обновлений

```loganalytics
Update
| where TimeGenerated>ago(5h) and OSType=="Linux" and SourceComputerId in ((Heartbeat
| where TimeGenerated>ago(12h) and OSType=="Linux" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
| where Solutions has "updates"
| distinct SourceComputerId))
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, BulletinUrl, BulletinID) by SourceComputerId, Product, ProductArch
| where UpdateState=~"Needed"
| project-away UpdateState, TimeGenerated
| summarize computersCount=dcount(SourceComputerId, 2), ClassificationWeight=max(iff(Classification has "Critical", 4, iff(Classification has "Security", 2, 1))) by id=strcat(Product, "_", ProductArch), displayName=Product, productArch=ProductArch, classification=Classification, InformationId=BulletinID, InformationUrl=tostring(split(BulletinUrl, ";", 0)[0]), osType=1
| union(Update
| where TimeGenerated>ago(14h) and OSType!="Linux" and (Optional==false or Classification has "Critical" or Classification has "Security") and SourceComputerId in ((Heartbeat
| where TimeGenerated>ago(12h) and OSType=~"Windows" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
| where Solutions has "updates"
| distinct SourceComputerId))
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, Title, KBID, PublishedDate, Approved) by Computer, SourceComputerId, UpdateID
| where UpdateState=~"Needed" and Approved!=false
| project-away UpdateState, Approved, TimeGenerated
| summarize computersCount=dcount(SourceComputerId, 2), displayName=any(Title), publishedDate=min(PublishedDate), ClassificationWeight=max(iff(Classification has "Critical", 4, iff(Classification has "Security", 2, 1))) by id=strcat(UpdateID, "_", KBID), classification=Classification, InformationId=strcat("KB", KBID), InformationUrl=iff(isnotempty(KBID), strcat("https://support.microsoft.com/kb/", KBID), ""), osType=2)
| sort by ClassificationWeight desc, computersCount desc, displayName asc
| extend informationLink=(iff(isnotempty(InformationId) and isnotempty(InformationUrl), toobject(strcat('{ "uri": "', InformationUrl, '", "text": "', InformationId, '", "target": "blank" }')), toobject('')))
| project-away ClassificationWeight, InformationId, InformationUrl
```

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о журналах Azure Monitor см. в статье [Журналы Azure Monitor](../../azure-monitor/logs/log-query-overview.md).
* Справку по оповещениям см. в статье [Настройка оповещений](configure-alerts.md).
