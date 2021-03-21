---
title: Мониторинг хранилища очередей Azure
description: Узнайте, как отслеживать производительность и доступность хранилища очередей Azure. Мониторинг данных хранилища очередей Azure, изучение конфигурации и анализ данных метрик и журналов.
author: normesta
services: storage
ms.author: normesta
ms.reviewer: fryu
ms.date: 10/26/2020
ms.topic: conceptual
ms.service: storage
ms.subservice: queues
ms.custom: monitoring, devx-track-csharp, devx-track-azurecli
ms.openlocfilehash: 8f49485d00379f5845569976e793f06d56a8967d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102506826"
---
# <a name="monitoring-azure-queue-storage"></a>Мониторинг хранилища очередей Azure

При наличии критически важных приложений и бизнес-процессов, использующих ресурсы Azure, необходимо отслеживать эти ресурсы на предмет их доступности, производительности и функционирования. В этой статье описываются данные мониторинга, созданные хранилищем очередей Azure, и способы использования функций Azure Monitor для анализа оповещений об этих данных.

> [!NOTE]
> Журналы службы хранилища Azure в Azure Monitor предоставляются в общедоступной предварительной версии. Они также доступны для предварительного тестирования во всех регионах общедоступного облака. Эта предварительная версия включает журналы для больших двоичных объектов (в том числе Azure Data Lake Storage 2-го поколения), файлов, очередей и таблиц. Эта функция доступна для всех учетных записей хранения, созданных с помощью модели развертывания Azure Resource Manager. См. раздел [Общие сведения об учетной записи хранения](../common/storage-account-overview.md).

## <a name="monitor-overview"></a>Общие сведения о мониторинге

Страница **Обзор** в портал Azure для каждого ресурса хранилища очередей содержит краткий обзор использования ресурсов, например запросов и почасовой оплаты. Эти сведения полезны, но лишь небольшое количество данных мониторинга доступно. Некоторые из этих данных собираются автоматически и доступны для анализа сразу после создания ресурса. Вы можете включить дополнительные типы сбора данных с помощью определенной конфигурации.

## <a name="what-is-azure-monitor"></a>Общие сведения об Azure Monitor

Хранилище очередей Azure создает данные мониторинга с помощью [Azure Monitor](../../azure-monitor/overview.md), которая представляет собой полную службу мониторинга стека в Azure. Azure Monitor предоставляет полный набор функций для мониторинга ресурсов Azure, а также ресурсов в других облаках и локальных средах.

Начните с [мониторинга ресурсов Azure с Azure Monitor](../../azure-monitor/essentials/monitor-azure-resource.md) , которые описывают следующее:

- Общие сведения об Azure Monitor
- затраты, связанные с мониторингом;
- данные мониторинга, собираемые в Azure;
- настройка сбора данных;
- стандартные средства Azure для анализа данных мониторинга и оповещения.

В следующих разделах этой статьи описываются конкретные данные, собранные из службы хранилища Azure. В примерах показано, как настроить сбор данных и анализировать эти данные с помощью средств Azure.

## <a name="monitoring-data"></a>Данные мониторинга

Хранилище очередей Azure собирает те же данные мониторинга, что и другие ресурсы Azure, которые описаны в разделе [мониторинг данных из ресурсов Azure](../../azure-monitor/essentials/monitor-azure-resource.md#monitoring-data).

Подробные сведения о метриках и журналах, созданных хранилищем очередей Azure, см. в [справочнике по данным мониторинга хранилища очередей Azure](monitor-queue-storage-reference.md) .

Метрики и журналы в Azure Monitor поддерживают только учетные записи хранения Azure Resource Manager. Azure Monitor не поддерживает классические учетные записи хранения. Если вы хотите использовать метрики или журналы в классической учетной записи хранения, необходимо выполнить миграцию в учетную запись хранения Azure Resource Manager. См. статью [Поддерживаемый платформой перенос ресурсов IaaS из классической модели в модель Azure Resource Manager](../../virtual-machines/migration-classic-resource-manager-overview.md).

При необходимости вы можете продолжить использование классических метрик и журналов. На самом деле, классические метрики и журналы доступны параллельно с метриками и журналами в Azure Monitor. Предусмотрена та же поддержка, пока в службе хранилища Azure обслуживаются устаревшие метрики и журналы.

## <a name="collection-and-routing"></a>Сбор и маршрутизация

Метрики платформы и журнал действий собираются автоматически, но их можно направить в другие расположения с помощью параметра диагностики.

Чтобы выполнить сбор журналов ресурсов, необходимо создать параметр диагностики. При создании параметра выберите **очередь** в качестве типа хранилища, для которого необходимо включить журналы. Затем укажите одну из следующих категорий операций, для которых требуется вести журнал.

| Категория | Описание |
|:---|:---|
| **StorageRead** | Операции чтения с объектами. |
| **StorageWrite** | Операции записи в объекты. |
| **StorageDelete** | Удаление операций с объектами. |

## <a name="creating-a-diagnostic-setting"></a>Создание параметра диагностики

Вы можете создать параметр диагностики с помощью портал Azure, PowerShell, Azure CLI или шаблона Azure Resource Manager.

Общие рекомендации см. в статье [Создание параметров диагностики для сбора журналов и метрик платформы в Azure](../../azure-monitor/essentials/diagnostic-settings.md).

> [!NOTE]
> Журналы службы хранилища Azure в Azure Monitor предоставляются в общедоступной предварительной версии. Они также доступны для предварительного тестирования во всех регионах общедоступного облака. Эта предварительная версия включает журналы для больших двоичных объектов (в том числе Azure Data Lake Storage 2-го поколения), файлов, очередей и таблиц. Эта функция доступна для всех учетных записей хранения, созданных с помощью модели развертывания Azure Resource Manager. См. раздел [Общие сведения об учетной записи хранения](../common/storage-account-overview.md).

### <a name="azure-portal"></a>[Портал Azure](#tab/azure-portal)

1. Войдите на портал Azure.

2. Войдите в свою учетную запись хранения.

3. В разделе **мониторинг** щелкните **параметры диагностики (Предварительная версия)**.

   > [!div class="mx-imgBorder"]
   > ![Портал — журналы диагностики](media/monitor-queue-storage/diagnostic-logs-settings-pane.png)

4. Выберите **очередь** в качестве типа хранилища, для которого необходимо включить журналы.

5. Щелкните **Добавить параметр диагностики**.

   > [!div class="mx-imgBorder"]
   > ![Портал — журналы ресурсов — Добавление параметра диагностики](media/monitor-queue-storage/diagnostic-logs-settings-pane-2.png)

   Откроется страница **параметры диагностики** .

   > [!div class="mx-imgBorder"]
   > ![Страница "Журналы ресурсов"](media/monitor-queue-storage/diagnostic-logs-page.png)

6. В поле **имя** страницы введите имя для этого параметра журнала ресурсов. Затем выберите операции, которые требуется регистрировать (операции чтения, записи и удаления), а также место, куда должны отправляться журналы.

#### <a name="archive-logs-to-a-storage-account"></a>Архивация журналов в учетную запись хранения

Если вы решили архивировать журналы в учетную запись хранения, вы оплачиваете объем журналов, отправляемых в учетную запись хранения. Конкретные цены см. в разделе **журналы платформы** на странице [цен на Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/#platform-logs) .

1. Установите флажок **архивировать в учетную запись хранения** , а затем нажмите кнопку **настроить** .

   > [!div class="mx-imgBorder"]
   > ![Архивное хранилище страницы параметров диагностики](media/monitor-queue-storage/diagnostic-logs-settings-pane-archive-storage.png)

2. В раскрывающемся списке **учетная запись хранения** выберите учетную запись хранения, в которую нужно архивировать журналы, нажмите кнопку **ОК** , а затем нажмите кнопку **сохранить** .
 
   [!INCLUDE [no retention policy](../../../includes/azure-storage-logs-retention-policy.md)]

   > [!NOTE]
   > Перед тем как выбрать учетную запись хранения в качестве назначения экспорта, см. раздел [Архивация журналов ресурсов Azure](../../azure-monitor/essentials/resource-logs.md#send-to-azure-storage) , чтобы ознакомиться с предварительными требованиями в учетной записи хранения.

#### <a name="stream-logs-to-azure-event-hubs"></a>Потоковая передача журналов в концентраторы событий Azure

Если вы решили выполнить потоковую передачу журналов в концентратор событий, вы оплачиваете объем журналов, отправляемых в концентратор событий. Конкретные цены см. в разделе **журналы платформы** на странице [цен на Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/#platform-logs) .

1. Установите флажок **потоковая передача в концентратор событий** , а затем нажмите кнопку **настроить** .

2. В области **Выбор концентратора событий** выберите пространство имен, имя и имя политики концентратора событий, в который будут подаваться потоки журналов.

   > [!div class="mx-imgBorder"]
   > ![Концентратор событий страницы параметров диагностики](media/monitor-queue-storage/diagnostic-logs-settings-pane-event-hub.png)

3. Нажмите кнопку " **ОК** ", а затем кнопку " **сохранить** ".

#### <a name="send-logs-to-azure-log-analytics"></a>Отправка журналов в Azure Log Analytics

1. Установите флажок **отправить на log Analytics** , выберите log Analytics рабочую область, а затем нажмите кнопку **сохранить** .

   > [!div class="mx-imgBorder"]
   > ![Страница "параметры диагностики" log Analytics](media/monitor-queue-storage/diagnostic-logs-settings-pane-log-analytics.png)

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

Замените `<storage-service-resource--id>` заполнитель в этом фрагменте идентификатором ресурса очереди. ИД ресурса можно найти на портале Azure, открыв страницу **свойств** учетной записи хранения.

Можно использовать `StorageRead` , `StorageWrite` и `StorageDelete` для значения параметра **Category** .

[!INCLUDE [no retention policy](../../../includes/azure-storage-logs-retention-policy.md)]

Приведем пример:

`Set-AzDiagnosticSetting -ResourceId /subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount/queueServices/default -StorageAccountId /subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/myloggingstorageaccount -Enabled $true -Category StorageWrite,StorageDelete`

Описание каждого параметра см. в разделе [Архивация журналов ресурсов Azure с помощью Azure PowerShell](../../azure-monitor/essentials/resource-logs.md#send-to-azure-storage).

#### <a name="stream-logs-to-an-event-hub"></a>Потоковая передача журналов в концентратор событий

Если вы решили выполнить потоковую передачу журналов в концентратор событий, вы оплачиваете объем журналов, отправляемых в концентратор событий. Конкретные цены см. в разделе **журналы платформы** на странице [цен на Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/#platform-logs) .

Включите журналы с помощью командлета PowerShell [Set-аздиагностиксеттинг](/powershell/module/az.monitor/set-azdiagnosticsetting) с `EventHubAuthorizationRuleId` параметром.

```powershell
Set-AzDiagnosticSetting -ResourceId <storage-service-resource-id> -EventHubAuthorizationRuleId <event-hub-namespace-and-key-name> -Enabled $true -Category <operations-to-log> -RetentionEnabled <retention-bool> -RetentionInDays <number-of-days>
```

Приведем пример:

`Set-AzDiagnosticSetting -ResourceId /subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount/queueServices/default -EventHubAuthorizationRuleId /subscriptions/20884142-a14v3-4234-5450-08b10c09f4/resourceGroups/myresourcegroup/providers/Microsoft.EventHub/namespaces/myeventhubnamespace/authorizationrules/RootManageSharedAccessKey -Enabled $true -Category StorageDelete`

Описание каждого параметра см. [в статье потоковая передача данных в концентраторы событий с помощью командлетов PowerShell](../../azure-monitor/essentials/resource-logs.md#send-to-azure-event-hubs).

#### <a name="send-logs-to-log-analytics"></a>Отправка журналов в Log Analytics

Включите журналы с помощью командлета PowerShell [Set-аздиагностиксеттинг](/powershell/module/az.monitor/set-azdiagnosticsetting) с `WorkspaceId` параметром.

```powershell
Set-AzDiagnosticSetting -ResourceId <storage-service-resource-id> -WorkspaceId <log-analytics-workspace-resource-id> -Enabled $true -Category <operations-to-log> -RetentionEnabled <retention-bool> -RetentionInDays <number-of-days>
```

Приведем пример:

`Set-AzDiagnosticSetting -ResourceId /subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount/queueServices/default -WorkspaceId /subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.OperationalInsights/workspaces/my-analytic-workspace -Enabled $true -Category StorageDelete`

Дополнительные сведения см. [в статье потоковая передача журналов ресурсов Azure в log Analytics рабочую область в Azure Monitor](../../azure-monitor/essentials/resource-logs.md#send-to-log-analytics-workspace).

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Сначала откройте [Azure Cloud Shell](../../cloud-shell/overview.md)или, если вы [установили Azure CLI](/cli/azure/install-azure-cli) локально, Откройте консольное приложение командной строки, например PowerShell.

2. Если ваше удостоверение связано с более чем одной подпиской, установите активную подписку на подписку учетной записи хранения, для которой требуется включить журналы.

   ```azurecli-interactive
   az account set --subscription <subscription-id>
   ```

   Замените значение заполнителя `<subscription-id>` идентификатором своей подписки.

#### <a name="archive-logs-to-a-storage-account"></a>Архивация журналов в учетную запись хранения

Если вы решили архивировать журналы в учетную запись хранения, вы оплачиваете объем журналов, отправляемых в учетную запись хранения. Конкретные цены см. в разделе **журналы платформы** на странице [цен на Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/#platform-logs) .

Включите журналы с помощью [`az monitor diagnostic-settings create`](/cli/azure/monitor/diagnostic-settings#az-monitor-diagnostic-settings-create) команды.

```azurecli-interactive
az monitor diagnostic-settings create --name <setting-name> --storage-account <storage-account-name> --resource <storage-service-resource-id> --resource-group <resource-group> --logs '[{"category": <operations>, "enabled": true}]'
```

Замените `<storage-service-resource--id>` заполнитель в этом фрагменте идентификатором ресурса очереди. ИД ресурса можно найти на портале Azure, открыв страницу **свойств** учетной записи хранения.

Можно использовать `StorageRead` , `StorageWrite` и `StorageDelete` для значения `category` параметра.

[!INCLUDE [no retention policy](../../../includes/azure-storage-logs-retention-policy.md)]

Приведем пример:

`az monitor diagnostic-settings create --name setting1 --storage-account mystorageaccount --resource /subscriptions/938841be-a40c-4bf4-9210-08bcf06c09f9/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/myloggingstorageaccount/queueServices/default --resource-group myresourcegroup --logs '[{"category": StorageWrite, "enabled": true}]'`

Описание каждого параметра см. [в разделе архивирование журналов ресурсов с помощью Azure CLI](../../azure-monitor/essentials/resource-logs.md#send-to-azure-storage).

#### <a name="stream-logs-to-an-event-hub"></a>Потоковая передача журналов в концентратор событий

Если вы решили выполнить потоковую передачу журналов в концентратор событий, вы оплачиваете объем журналов, отправляемых в концентратор событий. Конкретные цены см. в разделе **журналы платформы** на странице [цен на Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/#platform-logs) .

Включите журналы с помощью [`az monitor diagnostic-settings create`](/cli/azure/monitor/diagnostic-settings#az-monitor-diagnostic-settings-create) команды.

```azurecli-interactive
az monitor diagnostic-settings create --name <setting-name> --event-hub <event-hub-name> --event-hub-rule <event-hub-namespace-and-key-name> --resource <storage-account-resource-id> --logs '[{"category": <operations>, "enabled": true "retentionPolicy": {"days": <number-days>, "enabled": <retention-bool}}]'
```

Приведем пример:

`az monitor diagnostic-settings create --name setting1 --event-hub myeventhub --event-hub-rule /subscriptions/938841be-a40c-4bf4-9210-08bcf06c09f9/resourceGroups/myresourcegroup/providers/Microsoft.EventHub/namespaces/myeventhubnamespace/authorizationrules/RootManageSharedAccessKey --resource /subscriptions/938841be-a40c-4bf4-9210-08bcf06c09f9/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/myloggingstorageaccount/queueServices/default --logs '[{"category": StorageDelete, "enabled": true }]'`

Описание каждого параметра см. [в статье потоковая передача данных в концентраторы событий с помощью Azure CLI](../../azure-monitor/essentials/resource-logs.md#send-to-azure-event-hubs).

#### <a name="send-logs-to-log-analytics"></a>Отправка журналов в Log Analytics

Включите журналы с помощью [`az monitor diagnostic-settings create`](/cli/azure/monitor/diagnostic-settings#az-monitor-diagnostic-settings-create) команды.

```azurecli-interactive
az monitor diagnostic-settings create --name <setting-name> --workspace <log-analytics-workspace-resource-id> --resource <storage-account-resource-id> --logs '[{"category": <category name>, "enabled": true "retentionPolicy": {"days": <days>, "enabled": <retention-bool}}]'
```

Приведем пример:

`az monitor diagnostic-settings create --name setting1 --workspace /subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.OperationalInsights/workspaces/my-analytic-workspace --resource /subscriptions/938841be-a40c-4bf4-9210-08bcf06c09f9/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/myloggingstorageaccount/queueServices/default --logs '[{"category": StorageDelete, "enabled": true ]'`

 Дополнительные сведения см. [в статье потоковая передача журналов ресурсов Azure в log Analytics рабочую область в Azure Monitor](../../azure-monitor/essentials/resource-logs.md#send-to-log-analytics-workspace).

# <a name="template"></a>[Шаблон](#tab/template)

Чтобы просмотреть шаблон Azure Resource Manager, который создает параметр диагностики, см. раздел [параметр диагностики для службы хранилища Azure](../../azure-monitor/essentials/resource-manager-diagnostic-settings.md#diagnostic-setting-for-azure-storage).

---

## <a name="analyzing-metrics"></a>Анализ метрик

Вы можете анализировать метрики для службы хранилища Azure с метриками из других служб Azure с помощью обозреватель метрик Azure. Откройте обозреватель метрик, выбрав **Метрики** в меню **Azure Monitor**. Подробные сведения об использовании этого средства см. на странице [Начало работы с обозревателем метрик Azure](../../azure-monitor/essentials/metrics-getting-started.md).

В этом примере показано, как просмотреть **транзакции** на уровне учетной записи.

![Снимок экрана осуществления доступа к метрикам на портале Azure](./media/monitor-queue-storage/access-metrics-portal.png)

Для метрик с поддержкой измерений можно выполнить фильтрацию по нужному значению измерения. В этом примере объясняется, как просмотреть **транзакции** на уровне учетной записи для определенной операции, выбрав значения для измерения **Имя API**.

![Снимок экрана осуществления доступа к метрикам с поддержкой измерений на портале Azure](./media/monitor-queue-storage/access-metrics-portal-with-dimension.png)

Полный список измерений, поддерживаемых службой хранилища Azure, см. в разделе [Измерения метрик](monitor-queue-storage-reference.md#metrics-dimensions).

Метрики для хранилища очередей Azure находятся в следующих пространствах имен:

- Microsoft.Storage/storageAccounts
- Microsoft.Storage/storageAccounts/queueServices

Список всех метрик поддержки Azure Monitor, включая хранилище очередей Azure, см. в статье [Azure Monitor поддерживаемые метрики](../../azure-monitor/essentials/metrics-supported.md).

### <a name="accessing-metrics"></a>Доступ к метрикам

> [!TIP]
> Чтобы просмотреть примеры Azure CLI или .NET, выберите соответствующую вкладку ниже.

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

#### <a name="list-the-metric-definition"></a>Отображение определения метрики

Вы можете составить список определения метрик учетной записи хранения или службы хранилища очередей. Используйте командлет [Get-AzMetricDefinition](/powershell/module/az.monitor/get-azmetricdefinition).

В этом примере замените `<resource-ID>` заполнитель идентификатором ресурса всей учетной записи хранения или идентификатором ресурса очереди. Эти ИД ресурса можно найти на странице **свойств** учетной записи хранения на портале Azure.

```powershell
   $resourceId = "<resource-ID>"
   Get-AzMetricDefinition -ResourceId $resourceId
```

#### <a name="reading-metric-values"></a>Считывание значений метрик

Вы можете читать значения метрик уровня учетной записи хранения или службы хранилища очередей. Используйте командлет [Get-AzMetric](/powershell/module/az.monitor/get-azmetric).

```powershell
   $resourceId = "<resource-ID>"
   Get-AzMetric -ResourceId $resourceId -MetricNames "UsedCapacity" -TimeGrain 01:00:00
```

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

#### <a name="list-the-account-level-metric-definition"></a>Отображение определения метрик на уровне учетной записи

Вы можете составить список определения метрик учетной записи хранения или службы хранилища очередей. Используйте [`az monitor metrics list-definitions`](/cli/azure/monitor/metrics#az-monitor-metrics-list-definitions) команду.

В этом примере замените `<resource-ID>` заполнитель идентификатором ресурса всей учетной записи хранения или идентификатором ресурса очереди. Эти ИД ресурса можно найти на странице **свойств** учетной записи хранения на портале Azure.

```azurecli-interactive
   az monitor metrics list-definitions --resource <resource-ID>
```

#### <a name="read-account-level-metric-values"></a>Считывание значений метрик на уровне учетной записи

Вы можете прочитать значения метрик вашей учетной записи хранения или службы хранилища очередей. Используйте [`az monitor metrics list`](/cli/azure/monitor/metrics#az-monitor-metrics-list) команду.

```azurecli-interactive
   az monitor metrics list --resource <resource-ID> --metric "UsedCapacity" --interval PT1H
```

### <a name="net"></a>[.NET](#tab/azure-portal)

Azure Monitor предоставляет [пакет SDK для .NET](https://www.nuget.org/packages/microsoft.azure.management.monitor/) для считывания определения и значений метрик. [Пример кода](https://azure.microsoft.com/resources/samples/monitor-dotnet-metrics-api/) демонстрирует использование пакета SDK с различными параметрами. Необходимо использовать `0.18.0-preview` или более позднюю версию для метрик хранилища.

В этих примерах замените `<resource-ID>` заполнитель идентификатором ресурса всей учетной записи хранения или очереди. Эти ИД ресурса можно найти на странице **свойств** учетной записи хранения на портале Azure.

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
        // Resource ID for queue storage
        var resourceId = "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}/queueServices/default";
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

### <a name="template"></a>[Шаблон](#tab/template)

Недоступно

---

## <a name="analyzing-logs"></a>анализ журналов;

Доступ к журналам ресурсов можно получить либо в виде очереди в учетной записи хранения, либо в виде данных события, либо с помощью запросов Log Analytics.

Подробные справочные сведения о полях, которые отображаются в этих журналах, см. в разделе [Справочник по данным мониторинга хранилища очередей Azure](monitor-queue-storage-reference.md).

> [!NOTE]
> Журналы службы хранилища Azure в Azure Monitor предоставляются в общедоступной предварительной версии. Они также доступны для предварительного тестирования во всех регионах общедоступного облака. Эта предварительная версия включает журналы для больших двоичных объектов (в том числе Azure Data Lake Storage 2-го поколения), файлов, очередей, таблиц, учетных записей хранения класса Premium общего назначения версии 1 и учетных записей хранения общего назначения версии 2. Классические учетные записи хранения не поддерживаются.

Записи журнала создаются только при получении запроса к конечной точке службы. Например, если учетная запись хранения имеет действия в своей конечной точке очереди, но отсутствует в ее таблице или конечных точках BLOB-объектов, создаются только журналы, относящиеся к хранилищу очередей. Журналы службы хранилища Azure содержат подробные сведения об успешных и неудачных запросах к службе хранилища. Эта информация может использоваться для мониторинга отдельных запросов и диагностики неполадок в службе хранилища. Запросы вносятся в журнал наилучшим возможным образом.

### <a name="log-authenticated-requests"></a>Ведение журналов запросов, прошедших аутентификацию

Регистрируются запросы, прошедшие аутентификацию, следующих типов:

- Успешные запросы.
- Неудачные запросы, в том числе из-за ошибок, связанных с временем ожидания, регулированием, сетью, авторизацией и др.
- Запросы, в которых используется подписанный URL-адрес (SAS) или OAuth, в том числе неудачные и успешные запросы.
- Запросы к данным аналитики (классические данные журнала в контейнере **$logs** и данные метрик класса в таблицах **$metric**).

Запросы, выполняемые самой службой хранилища очередей, например создание или удаление журнала, не регистрируются. Полный список регистрируемых данных приведен на страницах об [операциях с протоколированием и сообщениях о состоянии службы хранилища](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages) и [формате журналов службы хранилища](monitor-queue-storage-reference.md).

### <a name="log-anonymous-requests"></a>Ведение журналов анонимных запросов

Регистрируются анонимные запросы следующих типов:

- Успешные запросы.
- ошибки сервера;
- Ошибки времени ожидания для клиента и сервера.
- Неудачные `GET` запросы с кодом ошибки 304 ( `Not Modified` )

Остальные неудачные анонимные запросы не регистрируются. Полный список регистрируемых данных приведен на страницах об [операциях с протоколированием и сообщениях о состоянии службы хранилища](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages) и [формате журналов службы хранилища](monitor-queue-storage-reference.md).

### <a name="accessing-logs-in-a-storage-account"></a>Доступ к журналам в учетной записи хранения

Журналы отображаются как большие двоичные объекты, хранящиеся в контейнере в целевой учетной записи хранения. Данные собираются и хранятся внутри одного большого двоичного объекта в качестве полезных данных JSON с разделителем-строкой. Имя большого двоичного объекта соответствует следующему соглашению об именовании.

`https://<destination-storage-account>.blob.core.windows.net/insights-logs-<storage-operation>/resourceId=/subscriptions/<subscription-ID>/resourceGroups/<resource-group-name>/providers/Microsoft.Storage/storageAccounts/<source-storage-account>/queueServices/default/y=<year>/m=<month>/d=<day>/h=<hour>/m=<minute>/PT1H.json`

Ниже приведен пример:

`https://mylogstorageaccount.blob.core.windows.net/insights-logs-storagewrite/resourceId=/subscriptions/`<br>`208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount/queueServices/default/y=2019/m=07/d=30/h=23/m=12/PT1H.json`

### <a name="accessing-logs-in-an-event-hub"></a>Доступ к журналам в концентраторе событий

Журналы, отправленные в концентратор событий, не сохраняются в виде файла, однако вы можете проверить, получил ли концентратор событий данные журнала. В портал Azure перейдите к концентратору событий и убедитесь, что значение `incoming requests` счетчика больше нуля.

![Журналы аудита](media/monitor-queue-storage/event-hub-log.png)

Вы можете получать доступ к данным журнала, отправляемым в концентратор событий, и считывать их, используя сведения о безопасности, а также средства мониторинга и управления событиями. Дополнительные сведения см. на [этой странице](../../azure-monitor/essentials/stream-monitoring-data-event-hubs.md#partner-tools-with-azure-monitor-integration).

### <a name="accessing-logs-in-a-log-analytics-workspace"></a>Доступ к журналам в рабочей области Log Analytics

Доступ к журналам, отправляемым в рабочую область Log Analytics, можно получить с помощью запросов журналов Azure Monitor.

Дополнительные сведения см. в статье [Начало работы с Log Analytics в Azure Monitor](../../azure-monitor/logs/log-analytics-tutorial.md).

Данные хранятся в `StorageQueueLogs` таблице.

#### <a name="sample-kusto-queries"></a>Примеры запросов Kusto

Ниже приведены некоторые запросы, которые можно ввести на панели **поиска журналов** , чтобы упростить мониторинг очередей. Эти запросы поддерживают [новый язык](../../azure-monitor/logs/log-query-overview.md).

> [!IMPORTANT]
> При выборе **журналов** в меню группы ресурсов учетной записи хранения log Analytics открывается с областью запроса, заданной для текущей группы ресурсов. Это означает, что запросы журнала будут содержать только данные из этой группы ресурсов. Если требуется выполнить запрос, который включает данные из других ресурсов или данных из других служб Azure, выберите **журналы** в меню **Azure Monitor** . Подробные сведения см. в статье [Область запросов журнала и временной диапазон в Azure Monitor Log Analytics](../../azure-monitor/logs/scope.md).

Используйте следующие запросы для мониторинга учетных записей хранения Azure:

- Для отображения 10 наиболее распространенных ошибок за последние три дня.

    ```Kusto
    StorageQueueLogs
    | where TimeGenerated > ago(3d) and StatusText !contains "Success"
    | summarize count() by StatusText
    | top 10 by count_ desc
    ```

- Для отображения основных 10 операций, вызвавших наибольшее количество ошибок за последние три дня.

    ```Kusto
    StorageQueueLogs
    | where TimeGenerated > ago(3d) and StatusText !contains "Success"
    | summarize count() by OperationName
    | top 10 by count_ desc
    ```

- Для отображения основных 10 операций с наибольшей сквозной задержкой за последние три дня.

    ```Kusto
    StorageQueueLogs
    | where TimeGenerated > ago(3d)
    | top 10 by DurationMs desc
    | project TimeGenerated, OperationName, DurationMs, ServerLatencyMs, ClientLatencyMs = DurationMs - ServerLatencyMs
    ```

- Для отображения всех операций, вызвавших ошибки регулирования на стороне сервера за последние три дня.

    ```Kusto
    StorageQueueLogs
    | where TimeGenerated > ago(3d) and StatusText contains "ServerBusy"
    | project TimeGenerated, OperationName, StatusCode, StatusText
    ```

- Для отображения всех запросов с анонимным доступом за последние три дня.

    ```Kusto
    StorageBlobLogs
    | where TimeGenerated > ago(3d) and AuthenticationType == "Anonymous"
    | project TimeGenerated, OperationName, AuthenticationType, Uri
    ```

- Для создания круговой диаграммы операций, используемых за последние три дня.

    ```Kusto
    StorageQueueLogs
    | where TimeGenerated > ago(3d)
    | summarize count() by OperationName
    | sort by count_ desc
    | render piechart
    ```

## <a name="faq"></a>ВОПРОСЫ И ОТВЕТЫ

**Поддерживаются ли метрики службы хранилища Azure для управляемых дисков или неуправляемых дисков?**

Нет. Расчетные экземпляры поддерживают метрики на дисках. Дополнительные сведения см. [в разделе метрики дисков для управляемых и неуправляемых дисков](https://azure.microsoft.com/blog/per-disk-metrics-managed-disks/).

## <a name="next-steps"></a>Дальнейшие действия

- Справочные сведения о журналах и метриках, созданных хранилищем очередей Azure, см. в статье [Справочник по данным мониторинга хранилища очередей Azure](monitor-queue-storage-reference.md).
- Подробные сведения о мониторинге ресурсов Azure см. на странице [Мониторинг ресурсов Azure с помощью Azure Monitor](../../azure-monitor/essentials/monitor-azure-resource.md).
- Дополнительные сведения о миграции метрик см. на [этой странице](../common/storage-metrics-migration.md).
