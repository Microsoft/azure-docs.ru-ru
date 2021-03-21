---
title: Мониторинг зрения Azure
description: Узнайте, как настроить метрики, оповещения и параметры диагностики Azure зрения с помощью Azure Monitor.
author: chanuengg
ms.author: csugunan
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 12/03/2020
ms.openlocfilehash: 4cb3965d359980856c238cd563ed8b761754660b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101667737"
---
# <a name="azure-purview-metrics-in-azure-monitor"></a>Метрики Azure зрения в Azure Monitor

В этой статье описывается, как настроить метрики, оповещения и параметры диагностики для Azure зрения с помощью Azure Monitor.

## <a name="monitor-azure-purview"></a>Мониторинг Azure зрения

Администраторы зрения Azure могут использовать Azure Monitor для отслеживания операционного состояния учетной записи зрения. Собираются метрики, чтобы предоставить точки данных для мониторинга потенциальных проблем, устранения неполадок и повышения надежности учетной записи зрения. Метрики отправляются в Azure Monitor для событий, происходящих в Azure зрения.

## <a name="aggregated-metrics"></a>Агрегированные метрики

Доступ к метрикам можно получить из портал Azure учетной записи зрения. Доступ к метрикам контролируется назначением роли учетной записи зрения. Чтобы увидеть метрики, пользователи должны быть частью роли "читатель мониторинга" в Azure зрения. Чтобы узнать больше о уровнях доступа к ролям, ознакомьтесь с [разрешениями наблюдения за ролью чтения](../azure-monitor/roles-permissions-security.md#built-in-monitoring-roles) .

Пользователь, создавший учетную запись зрения, автоматически получает разрешения на просмотр метрик. Если кто угодно хочет просмотреть метрики, добавьте их в роль **читатель мониторинга** , выполнив следующие действия.

### <a name="add-a-user-to-the-monitoring-reader-role"></a>Добавление пользователя в роль читателя мониторинга

Чтобы добавить пользователя в роль **читатель мониторинга** , владелец учетной записи зрения или владелец подписки может выполнить следующие действия:

1. Перейдите в [портал Azure](https://portal.azure.com) и найдите имя учетной записи Azure зрения.

2. Выберите **Управление доступом (IAM)** .

   :::image type="content" source="./media/how-to-monitor-with-azure-monitor/access-iam.png" alt-text="Снимок экрана, показывающий, как получить доступ к IAM.":::

3. Выберите **добавить назначение ролей**.

   :::image type="content" source="./media/how-to-monitor-with-azure-monitor/add-role-assignment.png" alt-text="Снимок экрана, показывающий, как добавить назначение ролей.":::

4. Выберите **средство чтения мониторинга** ролей и задайте для параметра назначить доступ **пользователю Azure AD, группе или субъекту-службе**. И назначьте учетной записи AAD доступ к метрикам.  

   :::image type="content" source="./media/how-to-monitor-with-azure-monitor/add-monitoring-reader.png" alt-text="Снимок экрана, показывающий, как добавить роль читателя мониторинга.":::

## <a name="metrics-visualization"></a>Визуализация метрик

Пользователи в роли " **читатель мониторинга** " могут видеть агрегированные метрики и журналы диагностики, отправленные в Azure Monitor. Метрики перечислены в портал Azure соответствующей учетной записи зрения. В портал Azure выберите раздел метрики, чтобы просмотреть список всех доступных метрик.

   :::image type="content" source="./media/how-to-monitor-with-azure-monitor/purview-metrics.png" alt-text="Снимок экрана, показывающий доступный раздел метрик зрения." lightbox="./media/how-to-monitor-with-azure-monitor/purview-metrics.png":::

Пользователи Azure зрения также могут получить доступ к странице метрики непосредственно из центра управления учетной записи Azure зрения. Выберите Azure Monitor на главной странице центра управления зрения, чтобы запустить портал Azure.

   :::image type="content" source="./media/how-to-monitor-with-azure-monitor/launch-metrics-from-management.png" alt-text="Снимок экрана для запуска метрик зрения из центра управления." lightbox="./media/how-to-monitor-with-azure-monitor/launch-metrics-from-management.png":::

### <a name="available-metrics"></a>Доступные метрики

Чтобы ознакомиться с использованием раздела "Метрика" в портал Azure заранее прочтите следующие два документа. Начало [работы с обозревателем метрик](../azure-monitor/essentials/metrics-getting-started.md) и [дополнительными функциями обозревателя метрик](../azure-monitor/essentials/metrics-charts.md).

В следующей таблице содержится список метрик, доступных для изучения в портал Azure.

| Имя метрики | Пространство имен метрики | Тип агрегирования | Описание |
| ------------------- | ------------------- | ------------------- | ----------------- |
| Сканирование отменено | Автоматическая проверка | SUM <br> Count | Статистическая обработка отмененных сканирований источника данных за период времени |
| Сканирование завершено | Автоматическая проверка | SUM <br> Count | Статистическая обработка завершенных просмотров источника данных за период времени |
| Сбой сканирования | Автоматическая проверка | SUM <br> Count | Агрегирование проверок источника данных с ошибками за период времени |
| Затраченное время сканирования | Автоматическая проверка | Min <br> Max <br> SUM <br> Сред. | Совокупное общее время, затраченное на сканирование за период времени |

## <a name="diagnostic-logs-to-azure-storage-account"></a>Журналы диагностики в учетную запись хранения Azure

События необработанных данных телеметрии создаются в Azure Monitor. События можно регистрировать в учетной записи хранения клиента по выбору для дальнейшего анализа. Экспорт журналов выполняется с помощью параметров диагностики для учетной записи зрения на портал Azure.

Выполните действия, чтобы создать параметр диагностики для учетной записи Azure зрения.

1. Создайте новый параметр диагностики для сбора журналов и метрик платформы, следуя этой статье: [Создание параметров диагностики для отправки журналов платформы и метрик в разные места назначения](../azure-monitor/essentials/diagnostic-settings.md). Выберите назначение только в качестве учетной записи хранения Azure.

   :::image type="content" source="./media/how-to-monitor-with-azure-monitor/step-one-diagnostic-setting.png" alt-text="Снимок экрана, показывающий создание журнала диагностики." lightbox="./media/how-to-monitor-with-azure-monitor/step-one-diagnostic-setting.png":::

2. Регистрировать события в учетной записи хранения. Для архивирования журналов диагностики рекомендуется использовать выделенную учетную запись хранения. Чтобы [создать учетную запись хранения](../storage/common/storage-account-create.md?tabs=azure-portal), необходимо выполнить следующие инструкции.

   :::image type="content" source="./media/how-to-monitor-with-azure-monitor/step-two-diagnostic-setting.png" alt-text="Снимок экрана, показывающий Назначение учетной записи хранения для журнала диагностики." lightbox="./media/how-to-monitor-with-azure-monitor/step-two-diagnostic-setting.png":::

Подождите 15 минут, чтобы начать получать журналы в созданной учетной записи хранения. [См. статью хранение данных и схема журналов ресурсов в учетной записи хранения Azure](../azure-monitor/essentials/resource-logs.md#send-to-azure-storage). После настройки журналов диагностики события направляются в учетную запись хранения.

### <a name="scanstatuslogevent"></a>сканстатусложевент

Событие отслеживает жизненный цикл сканирования. Операция просмотра проходит по последовательности состояний, из очереди, выполняется и, наконец, состояние терминала выполнено | Сбой | Отменено. Событие регистрируется для каждого перехода состояния, и схема события будет иметь следующие свойства.

```JSON
{
  "time": "<The UTC time when the event occurred>",
  "properties": {
    "dataSourceName": "<Registered data source friendly name>",
    "dataSourceType": "<Registered data source type>",
    "scanName": "<Scan instance friendly name>",
    "assetsDiscovered": "<If the resultType is succeeded, count of assets discovered in scan run>",
    "assetsClassified": "<If the resultType is succeeded, count of assets classified in scan run>",
    "scanQueueTimeInSeconds": "<If the resultType is succeeded, total seconds the scan instance in queue>",
    "scanTotalRunTimeInSeconds": "<If the resultType is succeeded, total seconds the scan took to run>",
    "runType": "<How the scan is triggered>",
    "errorDetails": "<Scan failure error>",
    "scanResultId": "<Unique GUID for the scan instance>"
  },
  "resourceId": "<The azure resource identifier>",
  "category": "<The diagnostic log category>",
  "operationName": "<The operation that cause the event Possible values for ScanStatusLogEvent category are: 
                    |AdhocScanRun 
                    |TriggeredScanRun 
                    |StatusChangeNotification>",
  "resultType": "Queued – indicates a scan is queued. 
                 Running – indicates a scan entered a running state. 
                 Succeeded – indicates a scan completed successfully. 
                 Failed – indicates a scan failure event. 
                 Cancelled – indicates a scan was cancelled. ",
  "resultSignature": "<Not used for ScanStatusLogEvent category. >",
  "resultDescription": "<This will have an error message if the resultType is Failed. >",
  "durationMs": "<Not used for ScanStatusLogEvent category. >",
  "level": "<The log severity level. Possible values are:
            |Informational
            |Error >",
  "location": "<The location of the Azure Purview account>",
}
```

Пример журнала для экземпляра события показан в следующем разделе.

```JSON
{
  "time": "2020-11-24T20:25:13.022860553Z",
  "properties": {
    "dataSourceName": "AzureDataExplorer-swD",
    "dataSourceType": "AzureDataExplorer",
    "scanName": "Scan-Kzw-shoebox-test",
    "assetsDiscovered": "0",
    "assetsClassified": "0",
    "scanQueueTimeInSeconds": "0",
    "scanTotalRunTimeInSeconds": "0",
    "runType": "Manual",
    "errorDetails": "empty_value",
    "scanResultId": "0dc51a72-4156-40e3-8539-b5728394561f"
  },
  "resourceId": "/SUBSCRIPTIONS/111111111111-111-4EB2/RESOURCEGROUPS/FOOBAR-TEST-RG/PROVIDERS/MICROSOFT.PURVIEW/ACCOUNTS/FOOBAR-HEY-TEST-NEW-MANIFEST-EUS",
  "category": "ScanStatusLogEvent",
  "operationName": "TriggeredScanRun",
  "resultType": "Delayed",
  "resultSignature": "empty_value",
  "resultDescription": "empty_value",
  "durationMs": 0,
  "level": "Informational",
  "location": "eastus",
}
```

## <a name="next-steps"></a>Дальнейшие действия

[Просмотреть Аналитику активов](asset-insights.md)