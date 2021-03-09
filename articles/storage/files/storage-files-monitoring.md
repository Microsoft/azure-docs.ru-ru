---
title: Мониторинг файлов Azure | Документация Майкрософт
description: Узнайте, как отслеживать производительность и доступность файлов Azure. Мониторинг данных файлов Azure, изучение конфигурации и анализ данных метрик и журналов.
author: normesta
services: storage
ms.service: storage
ms.subservice: files
ms.topic: conceptual
ms.date: 3/02/2021
ms.author: normesta
ms.reviewer: fryu
ms.custom: monitoring, devx-track-csharp, devx-track-azurecli
ms.openlocfilehash: 0612984afe71c3ae497d16968d2470668cc60ca7
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102504837"
---
# <a name="monitoring-azure-files"></a>Мониторинг файлов Azure

При наличии критически важных приложений и бизнес-процессов, использующих ресурсы Azure, необходимо отслеживать эти ресурсы на предмет их доступности, производительности и функционирования. В этой статье описываются данные мониторинга, создаваемые службой файлов Azure, и способы использования функций Azure Monitor для анализа оповещений об этих данных.

## <a name="monitor-overview"></a>Общие сведения о мониторинге

На странице **Обзор** в портал Azure для каждого ресурса службы файлов Azure содержатся краткие сведения об использовании ресурсов, таких как запросы и почасовая оплата. Эти сведения полезны, но лишь небольшое количество данных мониторинга доступно. Некоторые из этих данных собираются автоматически и доступны для анализа сразу после создания ресурса. Вы можете включить дополнительные типы сбора данных с помощью определенной конфигурации.

## <a name="what-is-azure-monitor"></a>Общие сведения об Azure Monitor
Служба "файлы Azure" создает данные мониторинга с помощью [Azure Monitor](../../azure-monitor/overview.md), которая представляет собой полную службу мониторинга стека в Azure. Azure Monitor предоставляет полный набор функций для мониторинга ресурсов Azure и ресурсов в других облаках, а также в локальной среде. 

Начните с статьи [мониторинг ресурсов Azure с помощью Azure Monitor](../../azure-monitor/essentials/monitor-azure-resource.md), который описывает следующее:

- Общие сведения об Azure Monitor
- затраты, связанные с мониторингом;
- данные мониторинга, собираемые в Azure;
- настройка сбора данных;
- стандартные средства Azure для анализа данных мониторинга и оповещения.

Следующие разделы посвящены этой статье с помощью описания конкретных данных, собранных из службы файлов Azure. В примерах показано, как настроить сбор данных и анализировать эти данные с помощью средств Azure.

## <a name="monitoring-data"></a>Данные мониторинга

Служба файлов Azure собирает те же данные мониторинга, что и другие ресурсы Azure, которые описаны в разделе [мониторинг данных из ресурсов Azure](../../azure-monitor/essentials/monitor-azure-resource.md#monitoring-data). 

Подробные сведения о метриках и журналах, созданных службой файлов Azure, см. в [справочнике по данным мониторинга файлов Azure](storage-files-monitoring-reference.md) .

Метрики и журналы в Azure Monitor поддерживают только учетные записи хранения Azure Resource Manager. Azure Monitor не поддерживает классические учетные записи хранения. Если вы хотите использовать метрики или журналы в классической учетной записи хранения, необходимо выполнить миграцию в учетную запись хранения Azure Resource Manager. См. статью [Поддерживаемый платформой перенос ресурсов IaaS из классической модели в модель Azure Resource Manager](../../virtual-machines/migration-classic-resource-manager-overview.md).

## <a name="collection-and-routing"></a>Сбор и маршрутизация

Метрики платформы и журнал действий собираются автоматически, но их можно направить в другие расположения с помощью параметра диагностики. 

Чтобы выполнить сбор журналов ресурсов, необходимо создать параметр диагностики. При создании параметра выберите **файл** в качестве типа хранилища, для которого необходимо включить журналы. Затем укажите одну из следующих категорий операций, для которых требуется вести журнал. 

| Категория | Описание |
|:---|:---|
| StorageRead | Операции чтения с объектами. |
| StorageWrite | Операции записи в объекты. |
| StorageDelete | Удаление операций с объектами. |

Чтобы получить список регистрируемых операций SMB и RESTFUL, см. раздел [операции с зарегистрированными хранилищами и сообщения о состоянии](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages) и [Справочник по данным мониторинга службы файлов Azure](storage-files-monitoring-reference.md).

## <a name="creating-a-diagnostic-setting"></a>Создание параметра диагностики

Вы можете создать параметр диагностики с помощью портал Azure, PowerShell, Azure CLI или шаблона Azure Resource Manager.

> [!NOTE]
> Журналы службы хранилища Azure в Azure Monitor предоставляются в общедоступной предварительной версии. Они также доступны для предварительного тестирования во всех регионах общедоступного облака. Эта предварительная версия включает журналы для больших двоичных объектов (в том числе Azure Data Lake Storage 2-го поколения), файлов, очередей и таблиц. Эта функция доступна для всех учетных записей хранения, созданных с помощью модели развертывания Azure Resource Manager. См. раздел [Общие сведения об учетной записи хранения](../common/storage-account-overview.md).

Общие рекомендации см. в статье [Создание параметров диагностики для сбора журналов и метрик платформы в Azure](../../azure-monitor/essentials/diagnostic-settings.md).

### <a name="azure-portal"></a>[Портал Azure](#tab/azure-portal)

1. Войдите на портал Azure.

2. Войдите в свою учетную запись хранения.

3. В разделе **мониторинг** щелкните **параметры диагностики (Предварительная версия)**.

   > [!div class="mx-imgBorder"]
   > ![Портал — журналы диагностики](media/storage-files-monitoring/diagnostic-logs-settings-pane.png)   

4. Выберите **файл** в качестве типа хранилища, для которого необходимо включить журналы.

5. Щелкните **Добавить параметр диагностики**.

   > [!div class="mx-imgBorder"]
   > ![Портал — журналы ресурсов — Добавление параметра диагностики](media/storage-files-monitoring/diagnostic-logs-settings-pane-2.png)

   Откроется страница **параметры диагностики** .

   > [!div class="mx-imgBorder"]
   > ![Страница "Журналы ресурсов"](media/storage-files-monitoring/diagnostic-logs-page.png)

6. В поле **имя** страницы введите имя для этого параметра журнала ресурсов. Затем выберите операции, которые требуется регистрировать (операции чтения, записи и удаления), а также место, куда должны отправляться журналы.

#### <a name="archive-logs-to-a-storage-account"></a>Архивация журналов в учетную запись хранения

Если вы решили архивировать журналы в учетную запись хранения, вы оплачиваете объем журналов, отправляемых в учетную запись хранения. Конкретные цены см. в разделе **журналы платформы** на странице [цен на Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/#platform-logs) .

1. Установите флажок **архивировать в учетную запись хранения** и нажмите кнопку **настроить** .

   > [!div class="mx-imgBorder"]   
   > ![Архивное хранилище страницы параметров диагностики](media/storage-files-monitoring/diagnostic-logs-settings-pane-archive-storage.png)

2. В раскрывающемся списке **учетная запись хранения** выберите учетную запись хранения, в которую нужно архивировать журналы, нажмите кнопку **ОК** , а затем нажмите кнопку **сохранить** .

   [!INCLUDE [no retention policy](../../../includes/azure-storage-logs-retention-policy.md)]

   > [!NOTE]
   > Перед тем как выбрать учетную запись хранения в качестве назначения экспорта, см. раздел [Архивация журналов ресурсов Azure](../../azure-monitor/essentials/resource-logs.md#send-to-azure-storage) , чтобы ознакомиться с предварительными требованиями в учетной записи хранения.

#### <a name="stream-logs-to-azure-event-hubs"></a>Потоковая передача журналов в концентраторы событий Azure

Если вы решили выполнить потоковую передачу журналов в концентратор событий, вы оплачиваете объем журналов, отправляемых в концентратор событий. Конкретные цены см. в разделе **журналы платформы** на странице [цен на Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/#platform-logs) .

1. Установите флажок **поток в концентратор событий** и нажмите кнопку **настроить** .

2. В области **Выбор концентратора событий** выберите пространство имен, имя и имя политики концентратора событий, в который будут подаваться потоки журналов. 

   > [!div class="mx-imgBorder"]
   > ![Концентратор событий страницы параметров диагностики](media/storage-files-monitoring/diagnostic-logs-settings-pane-event-hub.png)

3. Нажмите кнопку " **ОК** ", а затем кнопку " **сохранить** ".

#### <a name="send-logs-to-azure-log-analytics"></a>Отправка журналов в Azure Log Analytics

1. Установите флажок **отправить log Analytics** , выберите рабочую область log Analytics, а затем нажмите кнопку **сохранить** .

   > [!div class="mx-imgBorder"]   
   > ![Страница "параметры диагностики" log Analytics](media/storage-files-monitoring/diagnostic-logs-settings-pane-log-analytics.png)

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Откройте командное окно Windows PowerShell и войдите в подписку Azure с помощью `Connect-AzAccount` команды. Затем следуйте инструкциям на экране.

   ```powershell
   Connect-AzAccount
   ```

2. Настройте активную подписку для подписки учетной записи хранения, для которой требуется включить ведение журнала.

   ```powershell
   Set-AzContext -SubscriptionId <subscription-id>
   ```

#### <a name="archive-logs-to-a-storage-account"></a>Архивация журналов в учетную запись хранения

Если вы решили архивировать журналы в учетную запись хранения, вы оплачиваете объем журналов, отправляемых в учетную запись хранения. Конкретные цены см. в разделе **журналы платформы** на странице [цен на Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/#platform-logs) .

Включите журналы с помощью командлета PowerShell [Set-аздиагностиксеттинг](/powershell/module/az.monitor/set-azdiagnosticsetting) вместе с `StorageAccountId` параметром.

```powershell
Set-AzDiagnosticSetting -ResourceId <storage-service-resource-id> -StorageAccountId <storage-account-resource-id> -Enabled $true -Category <operations-to-log> 
```

Замените `<storage-service-resource--id>` заполнитель в этом фрагменте идентификатором ресурса службы файлов Azure. ИД ресурса можно найти на портале Azure, открыв страницу **свойств** учетной записи хранения.

Можно использовать `StorageRead` , `StorageWrite` и `StorageDelete` для значения параметра **Category** .

[!INCLUDE [no retention policy](../../../includes/azure-storage-logs-retention-policy.md)]

Ниже приведен пример:

`Set-AzDiagnosticSetting -ResourceId /subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount/fileServices/default -StorageAccountId /subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/myloggingstorageaccount -Enabled $true -Category StorageWrite,StorageDelete`

Описание каждого параметра см. в разделе [Архивация журналов ресурсов Azure с помощью Azure PowerShell](../../azure-monitor/essentials/resource-logs.md#send-to-azure-storage).

#### <a name="stream-logs-to-an-event-hub"></a>Потоковая передача журналов в концентратор событий

Если вы решили выполнить потоковую передачу журналов в концентратор событий, вы оплачиваете объем журналов, отправляемых в концентратор событий. Конкретные цены см. в разделе **журналы платформы** на странице [цен на Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/#platform-logs) .

Включите журналы с помощью командлета PowerShell [Set-аздиагностиксеттинг](/powershell/module/az.monitor/set-azdiagnosticsetting) с `EventHubAuthorizationRuleId` параметром.

```powershell
Set-AzDiagnosticSetting -ResourceId <storage-service-resource-id> -EventHubAuthorizationRuleId <event-hub-namespace-and-key-name> -Enabled $true -Category <operations-to-log> -RetentionEnabled <retention-bool> -RetentionInDays <number-of-days>
```

Ниже приведен пример:

`Set-AzDiagnosticSetting -ResourceId /subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount/fileServices/default -EventHubAuthorizationRuleId /subscriptions/20884142-a14v3-4234-5450-08b10c09f4/resourceGroups/myresourcegroup/providers/Microsoft.EventHub/namespaces/myeventhubnamespace/authorizationrules/RootManageSharedAccessKey -Enabled $true -Category StorageDelete`

Описание каждого параметра см. в статье [потоковая передача данных в концентраторы событий с помощью командлетов PowerShell](../../azure-monitor/essentials/resource-logs.md#send-to-azure-event-hubs).

#### <a name="send-logs-to-log-analytics"></a>Отправка журналов в Log Analytics

Включите журналы с помощью командлета PowerShell [Set-аздиагностиксеттинг](/powershell/module/az.monitor/set-azdiagnosticsetting) с `WorkspaceId` параметром.

```powershell
Set-AzDiagnosticSetting -ResourceId <storage-service-resource-id> -WorkspaceId <log-analytics-workspace-resource-id> -Enabled $true -Category <operations-to-log> -RetentionEnabled <retention-bool> -RetentionInDays <number-of-days>
```

Ниже приведен пример:

`Set-AzDiagnosticSetting -ResourceId /subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount/fileServices/default -WorkspaceId /subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.OperationalInsights/workspaces/my-analytic-workspace -Enabled $true -Category StorageDelete`

Дополнительные сведения см. [в статье потоковая передача журналов ресурсов Azure в log Analytics рабочую область в Azure Monitor](../../azure-monitor/essentials/resource-logs.md#send-to-log-analytics-workspace).

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Сначала откройте [Azure Cloud Shell](../../cloud-shell/overview.md) или, если вы [установили](/cli/azure/install-azure-cli) Azure CLI локально, откройте командное консольное приложение (например, Windows PowerShell).

2. Если ваше удостоверение связано с более чем одной подпиской, установите активную подписку на подписку учетной записи хранения, для которой требуется включить журналы.

   ```azurecli-interactive
   az account set --subscription <subscription-id>
   ```

   Замените значение заполнителя `<subscription-id>` идентификатором своей подписки.

#### <a name="archive-logs-to-a-storage-account"></a>Архивация журналов в учетную запись хранения

Если вы решили архивировать журналы в учетную запись хранения, вы оплачиваете объем журналов, отправляемых в учетную запись хранения. Конкретные цены см. в разделе **журналы платформы** на странице [цен на Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/#platform-logs) .

Включите журналы с помощью команды [AZ Monitor диагностики-Settings Create](/cli/azure/monitor/diagnostic-settings#az-monitor-diagnostic-settings-create) .

```azurecli-interactive
az monitor diagnostic-settings create --name <setting-name> --storage-account <storage-account-name> --resource <storage-service-resource-id> --resource-group <resource-group> --logs '[{"category": <operations>, "enabled": true}]'
```

Замените `<storage-service-resource--id>` заполнитель в этом фрагменте на службу хранилища BLOB-объектов с идентификатором ресурса. ИД ресурса можно найти на портале Azure, открыв страницу **свойств** учетной записи хранения.

Можно использовать `StorageRead` , `StorageWrite` и `StorageDelete` для значения параметра **Category** .

[!INCLUDE [no retention policy](../../../includes/azure-storage-logs-retention-policy.md)]

Ниже приведен пример:

`az monitor diagnostic-settings create --name setting1 --storage-account mystorageaccount --resource /subscriptions/938841be-a40c-4bf4-9210-08bcf06c09f9/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/myloggingstorageaccount/fileServices/default --resource-group myresourcegroup --logs '[{"category": StorageWrite, "enabled": true}]'`

Описание каждого параметра см. в разделе [Архивные журналы ресурсов с помощью Azure CLI](../../azure-monitor/essentials/resource-logs.md#send-to-azure-storage).

#### <a name="stream-logs-to-an-event-hub"></a>Потоковая передача журналов в концентратор событий

Если вы решили выполнить потоковую передачу журналов в концентратор событий, вы оплачиваете объем журналов, отправляемых в концентратор событий. Конкретные цены см. в разделе **журналы платформы** на странице [цен на Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/#platform-logs) .

Включите журналы с помощью команды [AZ Monitor диагностики-Settings Create](/cli/azure/monitor/diagnostic-settings#az-monitor-diagnostic-settings-create) .

```azurecli-interactive
az monitor diagnostic-settings create --name <setting-name> --event-hub <event-hub-name> --event-hub-rule <event-hub-namespace-and-key-name> --resource <storage-account-resource-id> --logs '[{"category": <operations>, "enabled": true "retentionPolicy": {"days": <number-days>, "enabled": <retention-bool}}]'
```

Ниже приведен пример:

`az monitor diagnostic-settings create --name setting1 --event-hub myeventhub --event-hub-rule /subscriptions/938841be-a40c-4bf4-9210-08bcf06c09f9/resourceGroups/myresourcegroup/providers/Microsoft.EventHub/namespaces/myeventhubnamespace/authorizationrules/RootManageSharedAccessKey --resource /subscriptions/938841be-a40c-4bf4-9210-08bcf06c09f9/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/myloggingstorageaccount/fileServices/default --logs '[{"category": StorageDelete, "enabled": true }]'`

Описание каждого параметра см. в статье [потоковая передача данных в концентраторы событий через Azure CLI](../../azure-monitor/essentials/resource-logs.md#send-to-azure-event-hubs).

#### <a name="send-logs-to-log-analytics"></a>Отправка журналов в Log Analytics

Включите журналы с помощью команды [AZ Monitor диагностики-Settings Create](/cli/azure/monitor/diagnostic-settings#az-monitor-diagnostic-settings-create) .

```azurecli-interactive
az monitor diagnostic-settings create --name <setting-name> --workspace <log-analytics-workspace-resource-id> --resource <storage-account-resource-id> --logs '[{"category": <category name>, "enabled": true "retentionPolicy": {"days": <days>, "enabled": <retention-bool}}]'
```

Ниже приведен пример:

`az monitor diagnostic-settings create --name setting1 --workspace /subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.OperationalInsights/workspaces/my-analytic-workspace --resource /subscriptions/938841be-a40c-4bf4-9210-08bcf06c09f9/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/myloggingstorageaccount/fileServices/default --logs '[{"category": StorageDelete, "enabled": true ]'`

 Дополнительные сведения см. [в статье потоковая передача журналов ресурсов Azure в log Analytics рабочую область в Azure Monitor](../../azure-monitor/essentials/resource-logs.md#send-to-log-analytics-workspace).

### <a name="template"></a>[Шаблон](#tab/template)

Чтобы просмотреть шаблон Azure Resource Manager, который создает параметр диагностики, см. раздел [параметр диагностики для службы хранилища Azure](../../azure-monitor/essentials/resource-manager-diagnostic-settings.md#diagnostic-setting-for-azure-storage).

---

## <a name="analyzing-metrics"></a>Анализ метрик

Вы можете анализировать метрики для службы хранилища Azure с помощью метрик из других служб Azure, используя обозреватель метрик. Откройте обозреватель метрик, выбрав **Метрики** в меню **Azure Monitor**. Подробные сведения об использовании этого средства см. на странице [Начало работы с обозревателем метрик Azure](../../azure-monitor/essentials/metrics-getting-started.md). 

Для метрик с поддержкой измерений можно выполнить фильтрацию по нужному значению измерения.  Полный список измерений, поддерживаемых службой хранилища Azure, см. в разделе [Измерения метрик](storage-files-monitoring-reference.md#metrics-dimensions). Метрики для службы файлов Azure находятся в следующих пространствах имен: 

- Microsoft.Storage/storageAccounts
- Microsoft.Storage/storageAccounts/fileServices

Список всех метрик поддержки Azure Monitor, включая службы файлов Azure, см. в статье [Azure Monitor поддерживаемые метрики](../../azure-monitor/essentials/metrics-supported.md#microsoftstoragestorageaccountsfileservices).

### <a name="accessing-metrics"></a>Доступ к метрикам

> [!TIP]
> Чтобы просмотреть примеры Azure CLI или .NET, выберите соответствующую вкладку ниже.

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

#### <a name="list-the-metric-definition"></a>Отображение определения метрики

Вы можете составить список определений метрик вашей учетной записи хранения или службы файлов Azure. Используйте командлет [Get-AzMetricDefinition](/powershell/module/az.monitor/get-azmetricdefinition).

В этом примере замените `<resource-ID>` заполнитель идентификатором ресурса всей учетной записи хранения или идентификатором ресурса службы файлов Azure.  Эти ИД ресурса можно найти на странице **свойств** учетной записи хранения на портале Azure.

```powershell
   $resourceId = "<resource-ID>"
   Get-AzMetricDefinition -ResourceId $resourceId
```

#### <a name="reading-metric-values"></a>Считывание значений метрик

Вы можете читать значения метрик на уровне учетной записи хранения или службы файлов Azure. Используйте командлет [Get-AzMetric](/powershell/module/Az.Monitor/Get-AzMetric).

```powershell
   $resourceId = "<resource-ID>"
   Get-AzMetric -ResourceId $resourceId -MetricNames "UsedCapacity" -TimeGrain 01:00:00
```

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

#### <a name="list-the-account-level-metric-definition"></a>Отображение определения метрик на уровне учетной записи

Вы можете составить список определений метрик вашей учетной записи хранения или службы файлов Azure. Используйте команду [az monitor metrics list-definitions](/cli/azure/monitor/metrics#az-monitor-metrics-list-definitions).
 
В этом примере замените `<resource-ID>` заполнитель идентификатором ресурса всей учетной записи хранения или идентификатором ресурса службы файлов Azure. Эти ИД ресурса можно найти на странице **свойств** учетной записи хранения на портале Azure.

```azurecli-interactive
   az monitor metrics list-definitions --resource <resource-ID>
```

#### <a name="read-account-level-metric-values"></a>Считывание значений метрик на уровне учетной записи

Вы можете прочитать значения метрик вашей учетной записи хранения или службы файлов Azure. Используйте команду [az monitor metrics list](/cli/azure/monitor/metrics#az-monitor-metrics-list).

```azurecli-interactive
   az monitor metrics list --resource <resource-ID> --metric "UsedCapacity" --interval PT1H
```

### <a name="net"></a>[.NET](#tab/azure-portal)

Azure Monitor предоставляет [пакет SDK для .NET](https://www.nuget.org/packages/Microsoft.Azure.Management.Monitor/) для считывания определения и значений метрик. [Пример кода](https://azure.microsoft.com/resources/samples/monitor-dotnet-metrics-api/) демонстрирует использование пакета SDK с различными параметрами. Необходимо использовать `0.18.0-preview` или более позднюю версию для метрик хранилища.
 
В этих примерах замените `<resource-ID>` заполнитель идентификатором ресурса всей учетной записи хранения или службой файлов Azure. Эти ИД ресурса можно найти на странице **свойств** учетной записи хранения на портале Azure.

Замените переменную `<subscription-ID>` на ИД своей подписки. Инструкции по получению значений для `<tenant-ID>`, `<application-ID>` и `<AccessKey>` см. на странице [Создание приложения Azure Active Directory и субъекта-службы с доступом к ресурсам с помощью портала](../../active-directory/develop/howto-create-service-principal-portal.md). 

#### <a name="list-the-account-level-metric-definition"></a>Отображение определения метрик на уровне учетной записи

В приведенном ниже примере показано, как получить список определений метрик на уровне учетной записи.

```csharp
    public static async Task ListStorageMetricDefinition()
    {
        var resourceId = "<resource-ID>";
        var subscriptionId = "<subscription-ID>";
        var tenantId = "<tenant-ID>";
        var applicationId = "<application-ID>";
        var accessKey = "<AccessKey>";


        MonitorManagementClient readOnlyClient = AuthenticateWithReadOnlyClient(tenantId, applicationId, accessKey, subscriptionId).Result;
        IEnumerable<MetricDefinition> metricDefinitions = await readOnlyClient.MetricDefinitions.ListAsync(resourceUri: resourceId, cancellationToken: new CancellationToken());

        foreach (var metricDefinition in metricDefinitions)
        {
            // Enumrate metric definition:
            //    Id
            //    ResourceId
            //    Name
            //    Unit
            //    MetricAvailabilities
            //    PrimaryAggregationType
            //    Dimensions
            //    IsDimensionRequired
        }
    }

```

#### <a name="reading-account-level-metric-values"></a>Чтение значений метрик уровня учетной записи

В приведенном ниже примере показано, как считать данные `UsedCapacity` на уровне учетной записи.

```csharp
    public static async Task ReadStorageMetricValue()
    {
        var resourceId = "<resource-ID>";
        var subscriptionId = "<subscription-ID>";
        var tenantId = "<tenant-ID>";
        var applicationId = "<application-ID>";
        var accessKey = "<AccessKey>";

        MonitorClient readOnlyClient = AuthenticateWithReadOnlyClient(tenantId, applicationId, accessKey, subscriptionId).Result;

        Microsoft.Azure.Management.Monitor.Models.Response Response;

        string startDate = DateTime.Now.AddHours(-3).ToUniversalTime().ToString("o");
        string endDate = DateTime.Now.ToUniversalTime().ToString("o");
        string timeSpan = startDate + "/" + endDate;

        Response = await readOnlyClient.Metrics.ListAsync(
            resourceUri: resourceId,
            timespan: timeSpan,
            interval: System.TimeSpan.FromHours(1),
            metricnames: "UsedCapacity",

            aggregation: "Average",
            resultType: ResultType.Data,
            cancellationToken: CancellationToken.None);

        foreach (var metric in Response.Value)
        {
            // Enumrate metric value
            //    Id
            //    Name
            //    Type
            //    Unit
            //    Timeseries
            //        - Data
            //        - Metadatavalues
        }
    }

```

#### <a name="reading-multidimensional-metric-values"></a>Считывание значений многомерных метрик

Для многомерных метрик необходимо определить фильтры по метаданным, если требуется считать данные метрик для конкретных значений измерений.

В приведенном ниже примере показано, как считать данные в метрике, поддерживающей многомерные значения.

```csharp
    public static async Task ReadStorageMetricValueTest()
    {
        // Resource ID for Azure Files
        var resourceId = "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}/fileServices/default";
        var subscriptionId = "<subscription-ID}";
        // How to identify Tenant ID, Application ID and Access Key: https://azure.microsoft.com/documentation/articles/resource-group-create-service-principal-portal/
        var tenantId = "<tenant-ID>";
        var applicationId = "<application-ID>";
        var accessKey = "<AccessKey>";

        MonitorManagementClient readOnlyClient = AuthenticateWithReadOnlyClient(tenantId, applicationId, accessKey, subscriptionId).Result;

        Microsoft.Azure.Management.Monitor.Models.Response Response;

        string startDate = DateTime.Now.AddHours(-3).ToUniversalTime().ToString("o");
        string endDate = DateTime.Now.ToUniversalTime().ToString("o");
        string timeSpan = startDate + "/" + endDate;
        // It's applicable to define meta data filter when a metric support dimension
        // More conditions can be added with the 'or' and 'and' operators, example: BlobType eq 'BlockBlob' or BlobType eq 'PageBlob'
        ODataQuery<MetadataValue> odataFilterMetrics = new ODataQuery<MetadataValue>(
            string.Format("BlobType eq '{0}'", "BlockBlob"));

        Response = readOnlyClient.Metrics.List(
                        resourceUri: resourceId,
                        timespan: timeSpan,
                        interval: System.TimeSpan.FromHours(1),
                        metricnames: "BlobCapacity",
                        odataQuery: odataFilterMetrics,
                        aggregation: "Average",
                        resultType: ResultType.Data);

        foreach (var metric in Response.Value)
        {
            //Enumrate metric value
            //    Id
            //    Name
            //    Type
            //    Unit
            //    Timeseries
            //        - Data
            //        - Metadatavalues
        }
    }

```

# <a name="template"></a>[Шаблон](#tab/template)

Недоступно

---

## <a name="analyzing-logs"></a>анализ журналов;

Доступ к журналам ресурсов можно получить с помощью большого двоичного объекта в учетной записи хранения, данных событий или запросов Log Analytic.

Чтобы получить список регистрируемых операций SMB и RESTFUL, см. раздел [операции с зарегистрированными хранилищами и сообщения о состоянии](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages) и [Справочник по данным мониторинга службы файлов Azure](storage-files-monitoring-reference.md).

> [!NOTE]
> Журналы службы хранилища Azure в Azure Monitor предоставляются в общедоступной предварительной версии. Они также доступны для предварительного тестирования во всех регионах общедоступного облака. Эта предварительная версия включает журналы для больших двоичных объектов (в том числе Azure Data Lake Storage 2-го поколения), файлов, очередей, таблиц, учетных записей хранения класса Premium общего назначения версии 1 и учетных записей хранения общего назначения версии 2. Классические учетные записи хранения не поддерживаются.

Записи журнала создаются только при получении запроса к конечной точке службы. Например, если учетная запись хранения имеет действие в конечной точке файла, но не находится в конечных точках таблицы или очереди, то создаются только журналы, относящиеся к службе файлов Azure. Журналы службы хранилища Azure содержат подробные сведения об успешных и неудачных запросах к службе хранилища. Эта информация может использоваться для мониторинга отдельных запросов и диагностики неполадок в службе хранилища. Запросы вносятся в журнал наилучшим возможным образом.

### <a name="log-authenticated-requests"></a>Ведение журналов запросов, прошедших аутентификацию

 Регистрируются запросы, прошедшие аутентификацию, следующих типов:

- Успешные запросы.
- Неудачные запросы, в том числе из-за ошибок, связанных со временем ожидания, регулированием, сетью, авторизацией и т. п.
- Запросы, использующие Kerberos, NTLM или подписанный URL-адрес (SAS), включая неудачные и успешные запросы
- Запросы к данным аналитики (классические данные журнала в контейнере **$logs** и классические данные метрик в таблицах **$Metric** )

Запросы, выполняемые самой службой файлов Azure, например создание или удаление журнала, не регистрируются. Полный список заносимых запросов SMB и RESTFUL см. в разделе операции с [зарегистрированным хранилищем и сообщения о состоянии](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages) и [Справочник по данным мониторинга службы файлов Azure](storage-files-monitoring-reference.md).

### <a name="accessing-logs-in-a-storage-account"></a>Доступ к журналам в учетной записи хранения

Журналы отображаются как большие двоичные объекты, хранящиеся в контейнере в целевой учетной записи хранения. Данные собираются и хранятся внутри одного большого двоичного объекта в качестве полезных данных JSON с разделителем-строкой. Имя большого двоичного объекта соответствует следующему соглашению об именовании.

`https://<destination-storage-account>.blob.core.windows.net/insights-logs-<storage-operation>/resourceId=/subscriptions/<subscription-ID>/resourceGroups/<resource-group-name>/providers/Microsoft.Storage/storageAccounts/<source-storage-account>/fileServices/default/y=<year>/m=<month>/d=<day>/h=<hour>/m=<minute>/PT1H.json`

Ниже приведен пример:

`https://mylogstorageaccount.blob.core.windows.net/insights-logs-storagewrite/resourceId=/subscriptions/`<br>`208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount/fileServices/default/y=2019/m=07/d=30/h=23/m=12/PT1H.json`

### <a name="accessing-logs-in-an-event-hub"></a>Доступ к журналам в концентраторе событий

Журналы, отправленные в концентратор событий, не сохраняются в виде файла, однако вы можете проверить, получил ли концентратор событий данные журнала. На портале Azure перейдите в концентратор событий и убедитесь, что количество **входящих сообщений** больше нуля. 

![Журналы аудита](media/storage-files-monitoring/event-hub-log.png)

Вы можете получать доступ к данным журнала, отправляемым в концентратор событий, и считывать их, используя сведения о безопасности, а также средства мониторинга и управления событиями. Дополнительные сведения см. на [этой странице](../../azure-monitor/essentials/stream-monitoring-data-event-hubs.md).

### <a name="accessing-logs-in-a-log-analytics-workspace"></a>Доступ к журналам в рабочей области Log Analytics

Доступ к журналам, отправляемым в рабочую область Log Analytics, можно получить с помощью запросов журналов Azure Monitor. Данные хранятся в таблице **сторажефилелогс** . 

Дополнительные сведения см. в разделе [log Analytics учебник](../../azure-monitor/logs/log-analytics-tutorial.md).

#### <a name="sample-kusto-queries"></a>Примеры запросов Kusto

Ниже приведены некоторые запросы, которые можно ввести на панели **поиска журналов** , чтобы упростить мониторинг файлов Azure. Эти запросы поддерживают [новый язык](../../azure-monitor/logs/log-query-overview.md).

> [!IMPORTANT]
> При выборе **журналов** в меню группы ресурсов учетной записи хранения log Analytics открывается с областью запроса, заданной для текущей группы ресурсов. Это означает, что запросы журнала будут содержать только данные из этой группы ресурсов. Если требуется выполнить запрос, который включает данные из других ресурсов или данных из других служб Azure, выберите **журналы** в меню **Azure Monitor** . Подробные сведения см. в статье [Область запросов журнала и временной диапазон в Azure Monitor Log Analytics](../../azure-monitor/logs/scope.md).

Используйте эти запросы для мониторинга файловых ресурсов Azure:

- Просмотр ошибок SMB за последнюю неделю

```Kusto
StorageFileLogs
| where Protocol == "SMB" and TimeGenerated >= ago(7d) and StatusCode contains "-"
| sort by StatusCode
```
- Создание круговой диаграммы операций SMB за последнюю неделю

```Kusto
StorageFileLogs
| where Protocol == "SMB" and TimeGenerated >= ago(7d) 
| summarize count() by OperationName
| sort by count_ desc
| render piechart
```

- Просмотр ошибок RESTFUL за последнюю неделю

```Kusto
StorageFileLogs
| where Protocol == "HTTPS" and TimeGenerated >= ago(7d) and StatusText !contains "Success"
| sort by StatusText asc
```

- Создание круговой диаграммы операций RESTFUL за последнюю неделю

```Kusto
StorageFileLogs
| where Protocol == "HTTPS" and TimeGenerated >= ago(7d) 
| summarize count() by OperationName
| sort by count_ desc
| render piechart
```

Список имен и описаний столбцов для службы файлов Azure см. в разделе [сторажефилелогс](/azure/azure-monitor/reference/tables/storagefilelogs).

Дополнительные сведения о написании запросов см. в разделе [log Analytics учебник](../../azure-monitor/logs/log-analytics-tutorial.md).

## <a name="alerts"></a>видны узлы

Оповещения Azure Monitor заблаговременно уведомляют вас при обнаружении важных условий в данных мониторинга. Они позволяют выявлять и устранять проблемы в системе до того, как ваши клиенты заметят их. Оповещения можно настроить для [метрик](../../azure-monitor/alerts/alerts-metric-overview.md), [журналов](../../azure-monitor/alerts/alerts-unified-log.md) и [журнала действий](../../azure-monitor/alerts/activity-log-alerts.md). 

В следующей таблице перечислены некоторые примеры сценариев для мониторинга и правильная метрика, используемая для оповещения.

| Сценарий | Метрика, используемая для оповещения |
|-|-|
| Файловый ресурс регулируется. | Метрика: транзакции<br>Имя измерения: тип ответа <br>Имя измерения: общая папка (только для файлового ресурса Premium) |
| Размер общей папки составляет 80% емкости. | Метрика: емкость файла<br>Имя измерения: общая папка (только для файлового ресурса Premium) |
| Исходящий файловый ресурс превысил 500 гиб в один день. | Метрика: исходящий трафик<br>Имя измерения: общая папка (только для файлового ресурса Premium) |

### <a name="how-to-create-alerts-for-azure-files"></a>Создание оповещений для службы файлов Azure

1. Перейдите к своей **учетной записи хранения** в **портал Azure**. 

2. Щелкните **оповещения** , а затем — **+ новое правило генерации оповещений**.

3. Щелкните **изменить ресурс**, выберите **тип файлового ресурса** и нажмите кнопку **Готово**. 

4. Нажмите кнопку **Добавить условие** и укажите следующие сведения для оповещения. 

    - **Метрика**
    - **Имя измерения**
    - **Логика оповещений**

5. Щелкните **Добавить группы действий** и добавьте в предупреждение группу действий (электронная почта, SMS и т. д.), выбрав существующую группу действий или создав новую группу действий.

6. Введите сведения о **предупреждении** , такие как имя, **Описание** и **серьезность** **правила оповещения**.

7. Щелкните **создать правило генерации оповещений** , чтобы создать оповещение.

> [!NOTE]  
> Если вы создаете предупреждение и оно слишком шумным, измените пороговое значение и логику оповещений.

### <a name="how-to-create-an-alert-if-a-file-share-is-throttled"></a>Создание оповещения в случае регулирования общей папки

1. Перейдите к своей **учетной записи хранения** в **портал Azure**.
2. В разделе **мониторинг** щелкните **оповещения**, а затем — **+ новое правило генерации оповещений**.
3. Щелкните **изменить ресурс**, выберите **тип файлового ресурса** для учетной записи хранения и нажмите кнопку **Готово**. Например, если имя учетной записи хранения — `contoso` , выберите `contoso/file` ресурс.
4. Нажмите кнопку **Добавить условие** , чтобы добавить условие.
5. Вы увидите список сигналов, поддерживаемых для учетной записи хранения, и выберите метрику **транзакции** .
6. В колонке **Настройка логики сигнала** щелкните раскрывающийся список **имя измерения** и выберите **тип ответа**.
7. Щелкните раскрывающийся список **значения измерений** и выберите соответствующие типы ответов для общей папки.

    Для стандартных файловых ресурсов выберите следующие типы ответов:

    - сукцессвиссроттлинг
    - ClientThrottlingError

    Для общих файловых ресурсов уровня "Премиум" выберите следующие типы ответов:

    - сукцессвисшаригресссроттлинг
    - сукцессвисшареингресссроттлинг
    - сукцессвисшареиопссроттлинг
    - клиентшаригресссроттлинжеррор
    - клиентшареингресссроттлинжеррор
    - клиентшареиопссроттлинжеррор

   > [!NOTE]
   > Если типы ответа не указаны в раскрывающемся списке **значения измерений** , это означает, что ресурс не был отрегулирован. Чтобы добавить значения измерений, рядом с раскрывающимся списком **значения измерений** выберите **Добавить настраиваемое значение**, введите тип ответе (например, **Сукцессвиссроттлинг**), нажмите кнопку **ОК**, а затем повторите эти действия, чтобы добавить все применимые типы ответов для общей папки.

8. Для **файловых ресурсов** уровня "Премиум" щелкните раскрывающийся список **имя измерения** и выберите **файловый ресурс**. Для **стандартных файловых ресурсов** перейдите к **шагу #10**.

   > [!NOTE]
   > Если общая папка является стандартным общим ресурсом, то в измерении **общей папки** не будут перечислены общие папки, так как метрики для общих папок недоступны для стандартных файловых ресурсов. Предупреждения регулирования для стандартных файловых ресурсов будут активированы, если какая-либо общая папка в учетной записи хранения регулируется, а предупреждение не определит, какой файловый ресурс был отрегулирован. Так как метрики для общего ресурса недоступны для стандартных файловых ресурсов, рекомендуется использовать одну общую папку для каждой учетной записи хранения.

9. Щелкните раскрывающийся список **значения измерений** и выберите Общие папки, по которым необходимо создать оповещение.
10. Определите **Параметры оповещения** (пороговое значение, оператор, гранулярность агрегирования и частоту оценки) и нажмите кнопку **Готово**.

    > [!TIP]
    > При использовании статического порога диаграмма метрик может помочь определить разумное пороговое значение, если общая папка в данный момент регулируется. Если используется динамическое пороговое значение, на диаграмме метрики будут отображаться рассчитанные пороговые значения на основе последних данных.

11. Нажмите кнопку **Добавить группы действий** , чтобы добавить **группу действий** (электронную почту, SMS и т. д.) в предупреждение, выбрав существующую группу действий или создав новую группу действий.
12. Введите сведения о **предупреждении** , такие как имя, **Описание** и **серьезность** **правила оповещения**.
13. Щелкните **создать правило генерации оповещений** , чтобы создать оповещение.

### <a name="how-to-create-an-alert-if-the-azure-file-share-size-is-80-of-capacity"></a>Как создать оповещение, если размер общей папки Azure составляет 80% емкости

1. Перейдите к своей **учетной записи хранения** в **портал Azure**.
2. В разделе **мониторинг** щелкните **оповещения** , а затем — **+ новое правило генерации оповещений**.
3. Щелкните **изменить ресурс**, выберите **тип файлового ресурса** для учетной записи хранения и нажмите кнопку **Готово**. Например, если имя учетной записи хранения — `contoso` , выберите `contoso/file` ресурс.
4. Нажмите кнопку **Добавить условие** , чтобы добавить условие.
5. Вы увидите список сигналов, поддерживаемых для учетной записи хранения, и выберите метрику **емкости файлов** .
6. Для **файловых ресурсов** уровня "Премиум" щелкните раскрывающийся список **имя измерения** и выберите **файловый ресурс**. Для **стандартных файловых ресурсов** перейдите к **шагу #8**.

   > [!NOTE]
   > Если общая папка является стандартным общим ресурсом, то в измерении **общей папки** не будут перечислены общие папки, так как метрики для общих папок недоступны для стандартных файловых ресурсов. Оповещения для стандартных файловых ресурсов основаны на всех общих файловых ресурсах в учетной записи хранения. Так как метрики для общего ресурса недоступны для стандартных файловых ресурсов, рекомендуется использовать одну общую папку для каждой учетной записи хранения.

7. Щелкните раскрывающийся список **значения измерений** и выберите Общие папки, по которым необходимо создать оповещение.
8. Введите **пороговое значение** в байтах. Например, если файловый ресурс имеет размер 100 Тиб и вы хотите получать предупреждение, если размер общей папки составляет 80% емкости, то пороговое значение в байтах — 87960930222080.
9. Определите остальные **Параметры оповещения** (гранулярность статистической обработки и частоту оценки) и нажмите кнопку **Готово**.
10. Нажмите кнопку **Добавить группы действий** , чтобы добавить **группу действий** (электронную почту, SMS и т. д.) в предупреждение, выбрав существующую группу действий или создав новую группу действий.
11. Введите сведения о **предупреждении** , такие как имя, **Описание** и **серьезность** **правила оповещения**.
12. Щелкните **создать правило генерации оповещений** , чтобы создать оповещение.

### <a name="how-to-create-an-alert-if-the-azure-file-share-egress-has-exceeded-500-gib-in-a-day"></a>Как создать оповещение, если исходящий файловый ресурс Azure превысил 500 гиб в день

1. Перейдите к своей **учетной записи хранения** в **портал Azure**.
2. В разделе Мониторинг щелкните **оповещения** , а затем — **+ новое правило генерации оповещений**.
3. Щелкните **изменить ресурс**, выберите **тип файлового ресурса** для учетной записи хранения и нажмите кнопку **Готово**. Например, если имя учетной записи хранения — Contoso, выберите ресурс Contoso/File.
4. Нажмите кнопку **Добавить условие** , чтобы добавить условие.
5. Вы увидите список сигналов, поддерживаемых для учетной записи хранения, и выберите метрику исходящего **трафика** .
6. Для **файловых ресурсов** уровня "Премиум" щелкните раскрывающийся список **имя измерения** и выберите **файловый ресурс**. Для **стандартных файловых ресурсов** перейдите к **шагу #8**.

   > [!NOTE]
   > Если общая папка является стандартным общим ресурсом, то в измерении **общей папки** не будут перечислены общие папки, так как метрики для общих папок недоступны для стандартных файловых ресурсов. Оповещения для стандартных файловых ресурсов основаны на всех общих файловых ресурсах в учетной записи хранения. Так как метрики для общего ресурса недоступны для стандартных файловых ресурсов, рекомендуется использовать одну общую папку для каждой учетной записи хранения.

7. Щелкните раскрывающийся список **значения измерений** и выберите Общие папки, по которым необходимо создать оповещение.
8. Введите значение порога **536870912000** байт. 
9. Щелкните раскрывающийся список **детализации статистической обработки** и выберите **24 часа**.
10. Выберите **частоту оценки** и **нажмите кнопку Готово**.
11. Нажмите кнопку **Добавить группы действий** , чтобы добавить **группу действий** (электронную почту, SMS и т. д.) в предупреждение, выбрав существующую группу действий или создав новую группу действий.
12. Введите сведения о **предупреждении** , такие как имя, **Описание** и **серьезность** **правила оповещения**.
13. Щелкните **создать правило генерации оповещений** , чтобы создать оповещение.

## <a name="next-steps"></a>Дальнейшие действия

- [Справочник по данным мониторинга файлов Azure](storage-files-monitoring-reference.md)
- [Мониторинг ресурсов Azure с помощью Azure Monitor](../../azure-monitor/essentials/monitor-azure-resource.md)
- [Миграция метрик службы хранилища Azure](../common/storage-metrics-migration.md)
- [Планирование развертывания Файлов Azure](./storage-files-planning.md)
- [Как развернуть службу файлов Azure](./storage-how-to-create-file-share.md)
- [Устранение неполадок в работе службы "Файлы Azure" в Windows](./storage-troubleshoot-windows-file-connection-problems.md)
- [Устранение неполадок в работе службы "Файлы Azure" в Linux](./storage-troubleshoot-linux-file-connection-problems.md)
