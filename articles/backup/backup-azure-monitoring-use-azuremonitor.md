---
title: Мониторинг Azure Backup с помощью Azure Monitor
description: Вы можете отслеживать рабочие нагрузки Azure Backup и создавать пользовательские оповещения с помощью Azure Monitor.
ms.topic: conceptual
ms.date: 06/04/2019
ms.assetid: 01169af5-7eb0-4cb0-bbdb-c58ac71bf48b
ms.openlocfilehash: 1800771bfff0afbcec8440383536734246ea8f5c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100580737"
---
# <a name="monitor-at-scale-by-using-azure-monitor"></a>Мониторинг в требуемом масштабе с помощью Azure Monitor

Azure Backup предоставляет [встроенные возможности мониторинга и оповещения](backup-azure-monitoring-built-in-monitor.md) в хранилище Служб восстановления. Эти возможности доступны без какой бы то ни было дополнительной инфраструктуры управления. Но эта встроенная служба ограничена в следующих сценариях.

- Мониторинг данных из нескольких хранилищ Служб восстановления в разных подписках.
- Предпочтительный канал уведомлений *отличается* от электронной почты.
- Пользователям требуются оповещения для других сценариев.
- Вы хотите просмотреть сведения из локального компонента, такого как System Center Data Protection Manager в Azure, которые не отображаются в разделе [**Задания архивации**](backup-azure-monitoring-built-in-monitor.md#backup-jobs-in-recovery-services-vault) или [**Оповещения, связанные с архивацией**](backup-azure-monitoring-built-in-monitor.md#backup-alerts-in-recovery-services-vault) портала.

## <a name="using-log-analytics-workspace"></a>Использование рабочей области Log Analytics

### <a name="create-alerts-by-using-log-analytics"></a>Создание оповещений с помощью Log Analytics

В Azure Monitor можно создавать собственные оповещения в рабочей области Log Analytics. В этой рабочей области для выбора предпочтительного механизма уведомления используются *группы действий Azure*.

> [!IMPORTANT]
> Сведения о затратах на создание этого запроса см. в разделе [Цены Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).

Откройте раздел **Журналы** в рабочей области Log Analytics и создайте запрос для собственных журналов. При выборе элемента **Новое правило генерации оповещений** открывается страница создания оповещений Azure Monitor, как показано на рисунке ниже.

![Создание оповещения в рабочей области Log Analytics](media/backup-azure-monitoring-laworkspace/custom-alert.png)

Здесь ресурс уже помечен как рабочая область Log Analytics, и предоставляется интеграция с группой действий.

![Страница создания оповещений Log Analytics](media/backup-azure-monitoring-laworkspace/inkedla-azurebackup-createalert.jpg)

#### <a name="alert-condition"></a>Условие оповещения

Определяющей характеристикой предупреждения является его условие активации. Выберите **Условие**, чтобы автоматически загрузить запрос Kusto на странице **Журналы**, как показано на рисунке ниже. Здесь можно изменить условие в соответствии со своими потребностями. Дополнительные сведения см. в разделе [Примеры запросов Kusto](#sample-kusto-queries).

![Настройка условия оповещения](media/backup-azure-monitoring-laworkspace/la-azurebackup-alertlogic.png)

При необходимости запрос Kusto можно изменить. Выберите пороговое значение, период и частоту. Пороговое значение определяет, когда будет вызвано предупреждение. Период — это окно времени, в котором выполняется запрос. Например, если пороговое значение больше 0, период составляет 5 минут, а частота составляет 5 минут, то правило выполняет запрос каждые 5 минут, просматривая предыдущие 5 минут. Если число результатов больше 0, вы получаете уведомления через выбранную группу действий.

> [!NOTE]
> Чтобы запустить правило генерации оповещений один раз в день во всех событиях и журналах, созданных в указанный день, измените значение "period" и "Frequency" на 1440, то есть 24 часа.

#### <a name="alert-action-groups"></a>Группы действий оповещений

Используйте группу действий для указания канала уведомлений. Чтобы просмотреть доступные механизмы уведомления, в разделе **Группы действий** выберите **Создать**.

![Доступные механизмы уведомления в окне "Добавление группы действий"](media/backup-azure-monitoring-laworkspace/LA-AzureBackup-ActionGroup.png)

Вы можете удовлетворить все требования к предупреждениям и мониторингу прямо из Log Analytics или использовать Log Analytics в дополнение ко встроенным уведомлениям.

Дополнительные сведения см. в статьях [Создание и просмотр оповещений журнала, а также управление ими с помощью Azure Monitor](../azure-monitor/alerts/alerts-log.md) и [Создание групп действий и управление ими на портале Azure](../azure-monitor/alerts/action-groups.md).

### <a name="sample-kusto-queries"></a>Примеры запросов Kusto

Графы по умолчанию предоставляют вам запросы Kusto для основных сценариев, на основе которых можно создавать оповещения. Кроме того, можно изменить запросы, чтобы получить данные, для которых вы хотите получать оповещения. Вставьте приведенные ниже примеры запросов Kusto на странице **Журналы**, а затем создайте предупреждения для запросов.

- Все успешные задания резервного копирования

    ````Kusto
    AddonAzureBackupJobs
    | where JobOperation=="Backup"
    | summarize arg_max(TimeGenerated,*) by JobUniqueId
    | where JobStatus=="Completed"
    ````

- Все неудачные задания резервного копирования

    ````Kusto
    AddonAzureBackupJobs
    | where JobOperation=="Backup"
    | summarize arg_max(TimeGenerated,*) by JobUniqueId
    | where JobStatus=="Failed"
    ````

- Все успешные задания резервного копирования виртуальных машин Azure

    ````Kusto
    AddonAzureBackupJobs
    | where JobOperation=="Backup"
    | summarize arg_max(TimeGenerated,*) by JobUniqueId
    | where JobStatus=="Completed"
    | join kind=inner
    (
        CoreAzureBackup
        | where OperationName == "BackupItem"
        | where BackupItemType=="VM" and BackupManagementType=="IaaSVM"
        | distinct BackupItemUniqueId, BackupItemFriendlyName
    )
    on BackupItemUniqueId
    ````

- Все успешные задания резервного копирования журналов SQL

    ````Kusto
    AddonAzureBackupJobs
    | where JobOperation=="Backup" and JobOperationSubType=="Log"
    | summarize arg_max(TimeGenerated,*) by JobUniqueId
    | where JobStatus=="Completed"
    | join kind=inner
    (
        CoreAzureBackup
        | where OperationName == "BackupItem"
        | where BackupItemType=="SQLDataBase" and BackupManagementType=="AzureWorkload"
        | distinct BackupItemUniqueId, BackupItemFriendlyName
    )
    on BackupItemUniqueId
    ````

- Все успешные задания агента Azure Backup

    ````Kusto
    AddonAzureBackupJobs
    | where JobOperation=="Backup"
    | summarize arg_max(TimeGenerated,*) by JobUniqueId
    | where JobStatus=="Completed"
    | join kind=inner
    (
        CoreAzureBackup
        | where OperationName == "BackupItem"
        | where BackupItemType=="FileFolder" and BackupManagementType=="MAB"
        | distinct BackupItemUniqueId, BackupItemFriendlyName
    )
    on BackupItemUniqueId
    ````

- Используемое хранилище резервных копий на элемент резервного копирования

    ````Kusto
    CoreAzureBackup
    //Get all Backup Items
    | where OperationName == "BackupItem"
    //Get distinct Backup Items
    | distinct BackupItemUniqueId, BackupItemFriendlyName
    | join kind=leftouter
    (AddonAzureBackupStorage
    | where OperationName == "StorageAssociation"
    //Get latest record for each Backup Item
    | summarize arg_max(TimeGenerated, *) by BackupItemUniqueId
    | project BackupItemUniqueId , StorageConsumedInMBs)
    on BackupItemUniqueId
    | project BackupItemUniqueId , BackupItemFriendlyName , StorageConsumedInMBs
    | sort by StorageConsumedInMBs desc
    ````

### <a name="diagnostic-data-update-frequency"></a>Частота обновления данных диагностики

Данные диагностики из хранилища передаются в рабочую область Log Analytics с некоторой задержкой. Каждое событие поступает в рабочую область Log Analytics через *20–30 минут* после его отправки из хранилища Служб восстановления. Ниже приведены дополнительные сведения об этой задержке.

- Во всех решениях встроенные оповещения службы резервного копирования передаются сразу после их создания. Поэтому они обычно отображаются в рабочей области Log Analytics через 20–30 минут.
- Во всех решениях задания резервного копирования по запросу и задания восстановления передаются сразу после их *завершения*.
- Во всех решениях, кроме резервного копирования SQL, запланированные задания резервного копирования передаются сразу после их *завершения*.
- Для резервного копирования SQL, так как резервное копирование журналов может происходить каждые 15 минут, сведения обо всех завершенных запланированных заданиях резервного копирования, включая журналы, помещаются в пакеты и передаются каждые 6 часов.
- Во всех решениях другие сведения, такие как элемент резервного копирования, политика, точки восстановления, хранилище и т. п., передаются по меньшей мере *один раз в день.*
- Изменение конфигурации резервного копирования (например, смена или изменение политики) активирует принудительную передачу всех связанных сведений резервного копирования.

## <a name="using-the-recovery-services-vaults-activity-logs"></a>Использование журналов действий хранилища Служб восстановления

> [!CAUTION]
> Следующие действия применимы только к *резервным копиям виртуальных машин Azure.* Эти действия невозможно использовать для таких решений, как агент Azure Backup, резервные копии SQL в Azure или Файлы Azure.

Вы также можете использовать журналы действий для получения уведомлений о таких событиях, как успешное резервное копирование. Чтобы начать работу, выполните следующие действия.

1. Войдите на портал Azure.
2. Откройте соответствующее хранилище Служб восстановления.
3. В свойствах хранилища откройте раздел **Журнал действий**.

Определение соответствующего журнала и создание оповещения

1. Убедитесь, что вы получаете журналы действий для успешного резервного копирования, применив фильтры, показанные на рисунке ниже. При необходимости измените значение **Временной диапазон**, чтобы просмотреть записи.

   ![Фильтрация для поиска журналов действий для резервных копий виртуальных машин Azure](media/backup-azure-monitoring-laworkspace/activitylogs-azurebackup-vmbackups.png)

2. Выберите имя операции, чтобы просмотреть соответствующие сведения.
3. Выберите **Новое правило генерации оповещений**, чтобы открыть страницу **Создание правила**.
4. Создайте оповещение, выполнив действия из раздела [Создание, просмотр и управление оповещениями журнала действий с помощью Azure Monitor](../azure-monitor/alerts/alerts-activity-log.md).

   ![Новое правило генерации оповещений](media/backup-azure-monitoring-laworkspace/new-alert-rule.png)

Здесь ресурсом является само хранилище Служб восстановления. Повторите те же действия для всех хранилищ, в которых хотите получать уведомления с помощью журналов действий. Условие не будет иметь порог, период или частоту, так как это оповещение основано на событиях. Оповещение выдается после создания соответствующего журнала действий.

## <a name="using-log-analytics-to-monitor-at-scale"></a>Использование Log Analytics для мониторинга в требуемом масштабе

Вы можете просмотреть все оповещения, созданные на основе журналов действий и рабочих областей Log Analytics в Azure Monitor. Просто откройте панель **Оповещения** слева.

Хотя вы можете получать уведомления с помощью журналов действий, для мониторинга в требуемом масштабе мы настоятельно рекомендуем использовать Log Analytics, а не журналы действий. Для этого есть следующие причины.

- **Ограниченные сценарии**: уведомления, созданные с помощью журналов действий, применяются только к резервным копиям виртуальных машин Azure. Уведомления должны быть настроены для каждого хранилища Служб восстановления.
- **Соответствие определения**: запланированное действие резервного копирования не соответствует последнему определению журналов действий. Вместо этого оно соответствует [журналам ресурсов](../azure-monitor/essentials/resource-logs.md#send-to-log-analytics-workspace). Такое согласование приводит к непредвиденным последствиям при изменении данных, передаваемых по каналу журнала действий.
- **Проблемы с каналом журнала действий**: в хранилищах Служб восстановления журналы действий, передаваемые из Azure Backup, следуют новой модели. К сожалению, это изменение затрагивает создание журналов действий в Azure для государственных организаций, Azure для Германии и Azure для Китая от 21Vianet. Если пользователи этих облачных служб создают или настраивают оповещения из журналов действий в Azure Monitor, такие оповещения не активируются. Кроме того, во всех общедоступных регионах Azure, когда пользователь [собирает журналы действий Служб восстановления в рабочую область Log Analytics](../azure-monitor/essentials/activity-log.md), эти журналы не отображаются.

Используйте рабочую область Log Analytics для мониторинга и использования оповещений в требуемом масштабе для всех рабочих нагрузок, защищенных Azure Backup.

## <a name="next-steps"></a>Дальнейшие действия

Сведения о создании пользовательских запросов см. в разделе [Модель данных Log Analytics](backup-azure-reports-data-model.md).
