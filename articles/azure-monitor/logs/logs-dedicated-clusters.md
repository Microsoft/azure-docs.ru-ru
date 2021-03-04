---
title: Azure Monitor журналов выделенных кластеров
description: Клиенты, получившие более 1 ТБ данных мониторинга в день, могут использовать выделенные, а не общие кластеры.
ms.topic: conceptual
author: rboucher
ms.author: robb
ms.date: 09/16/2020
ms.openlocfilehash: 1fbb8f82366ee961d10ce8a6bc098128bad6555a
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102041014"
---
# <a name="azure-monitor-logs-dedicated-clusters"></a>Azure Monitor журналов выделенных кластеров

Azure Monitor журналов выделенные кластеры — это вариант развертывания, который включает расширенные возможности для Azure Monitor журналов клиентов. Клиенты с выделенными кластерами могут выбрать рабочие области, которые будут размещены в этих кластерах.

Ниже перечислены возможности, требующие использования выделенных кластеров.

- **[Ключи, управляемые клиентом](../logs/customer-managed-keys.md)** — шифруют данные кластера с помощью ключей, предоставляемых и контролируемых клиентом.
- **[Защищенное хранилище](../logs/customer-managed-keys.md#customer-lockbox-preview)** — клиенты могут контролировать запросы на доступ к данным в службе поддержки Майкрософт.
- **[Двойное шифрование](../../storage/common/storage-service-encryption.md#doubly-encrypt-data-with-infrastructure-encryption)** защищает от ситуации, когда один из алгоритмов или ключей шифрования может быть скомпрометирован. В этом случае дополнительный уровень шифрования продолжит защищать ваши данные.
- **[Несколько рабочих областей](../logs/cross-workspace-query.md)** . Если клиент использует более одной рабочей области для рабочей среды, может быть целесообразно использовать выделенный кластер. Запросы между рабочими областями будут выполняться быстрее, если все рабочие области находятся в одном кластере. Кроме того, использование выделенного кластера может быть более экономичным, так как назначенные уровни резервирования емкости принимают во внимание все приемы кластера и применяются ко всем его рабочим областям, даже если некоторые из них невелики и не имеют скидки на резервирование емкости.

Для выделенных кластеров клиенты должны использовать емкость не менее 1 ТБ приема данных в день. Миграция в выделенный кластер проста. Нет потерь данных или прерываний службы. 

## <a name="management"></a>Управление 

Управление выделенными кластерами осуществляется через ресурс Azure, который представляет Azure Monitor кластеры журналов. Все операции выполняются с этим ресурсом с помощью PowerShell или REST API.

После создания кластера его можно настроить и связать с ним рабочие области. Если Рабочая область связана с кластером, новые данные, отправляемые в рабочую область, находятся в кластере. Только рабочие области, наявляющиеся в том же регионе, что и кластер, могут быть связаны с кластером. Рабочие области могут быть в отличие от кластера с некоторыми ограничениями. Дополнительные сведения об этих ограничениях приведены в этой статье. 

Данные, принимаемые в выделенные кластеры, шифруются дважды — один раз на уровне службы с помощью ключей, управляемых корпорацией Майкрософт или [управляемым клиентом ключом](../logs/customer-managed-keys.md), и один раз на уровне инфраструктуры с использованием двух разных алгоритмов шифрования и двух разных ключей. [Двойное шифрование](../../storage/common/storage-service-encryption.md#doubly-encrypt-data-with-infrastructure-encryption) защищает от ситуации, когда один из алгоритмов или ключей шифрования может быть скомпрометирован. В этом случае дополнительный уровень шифрования продолжит защищать ваши данные. Выделенный кластер также позволяет защищать данные с помощью элемента управления [защищенным хранилищем](../logs/customer-managed-keys.md#customer-lockbox-preview) .

Для всех операций на уровне кластера требуется `Microsoft.OperationalInsights/clusters/write` разрешение Action в кластере. Это разрешение можно предоставить с помощью владельца или участника, который содержит `*/write` действие, или с помощью роли участника log Analytics, которая содержит `Microsoft.OperationalInsights/*` действие. Дополнительные сведения о Log Analytics разрешений см. [в разделе Управление доступом к данным и рабочим областям журнала в Azure Monitor](./manage-access.md). 


## <a name="cluster-pricing-model"></a>Ценовая модель кластера

В Log Analytics выделенных кластерах используется модель ценообразования резервирования емкости, которая не менее 1000 ГБ в день. При превышении уровня использования резервирования будет взиматься плата по мере использования.  Сведения о ценах на резервирование емкости доступны на [странице цен на Azure Monitor]( https://azure.microsoft.com/pricing/details/monitor/).  

Уровень резервирования емкости кластера настраивается программным путем с Azure Resource Manager с помощью `Capacity` параметра в разделе `Sku` . `Capacity` указывается в ГБ и может иметь значения 1000 ГБ/день или более с шагом приращения 100 ГБ/день.

Существует два режима выставления счетов за использование в кластере. Они могут быть заданы `billingType` параметром при настройке кластера. 

1. **Кластер**. в данном случае (по умолчанию) выставление счетов за принимаемые данные выполняется на уровне кластера. Полученные количества данных из каждой рабочей области, связанной с кластером, суммируются для вычисления ежедневного счета за кластер. 

2. **Рабочие области**. затраты на резервирование ресурсов для кластера настраиваются пропорционально рабочим областям в кластере (после учета для каждого узла из [центра безопасности Azure](../../security-center/index.yml) для каждой рабочей области).

Если ваша рабочая область использует ценовую категорию "устаревшие" на узел, если она связана с кластером, плата будет взиматься с учетом данных, полученных в отношении резервирования емкости кластера и более недоступных для каждого узла. Распределение данных по узлам из центра безопасности Azure будет по прежнему применено.

Дополнительные сведения о выставлении счетов за Log Analytics выделенные кластеры доступны [здесь]( https://docs.microsoft.com/azure/azure-monitor/platform/manage-cost-storage#log-analytics-dedicated-clusters).

## <a name="asynchronous-operations-and-status-check"></a>Асинхронные операции и проверка состояния

Некоторые этапы настройки выполняются асинхронно, так как они не могут быть завершены быстро. В ответе может быть указано одно из следующих состояний: "выполняется", "обновление", "удаление", "успешно" или "сбой", включая код ошибки. При использовании функции остальное ответ изначально возвращает код состояния HTTP 202 (принято) и заголовок со свойством Azure-AsyncOperation:

```JSON
"Azure-AsyncOperation": "https://management.azure.com/subscriptions/subscription-id/providers/Microsoft.OperationalInsights/locations/region-name/operationStatuses/operation-id?api-version=2020-08-01"
```

Вы можете проверить состояние асинхронной операции, отправив запрос GET значению заголовка Azure-AsyncOperation:

```rst
GET https://management.azure.com/subscriptions/subscription-id/providers/microsoft.operationalInsights/locations/region-name/operationstatuses/operation-id?api-version=2020-08-01
Authorization: Bearer <token>
```

## <a name="creating-a-cluster"></a>Создание кластера

Сначала создайте ресурсы кластера, чтобы начать создание выделенного кластера.

Необходимо указать следующие свойства:

- **Имя_кластера**: используется для административных целей. Пользователи не предоставляются по этому имени.
- **ResourceGroupName**. как и для любого ресурса Azure, кластеры принадлежат к группе ресурсов. Мы рекомендуем использовать центральную группу ресурсов ИТ, так как кластеры обычно являются общими для многих команд в Организации. Дополнительные рекомендации по проектированию см. [в разделе Разработка развертывания журналов Azure Monitor](../logs/design-logs-deployment.md)
- **Расположение**: кластер находится в определенном регионе Azure. Только рабочие области, расположенные в этом регионе, могут быть связаны с этим кластером.
- **СкукапаЦити**: при создании ресурса *кластера* необходимо указать уровень *резервирования емкости* (SKU). Уровень *резервирования емкости* может быть в диапазоне от 1 000 гб до 3 000 ГБ в день. При необходимости его можно обновить, выполнив шаги, описанные в 100. Если требуется уровень резервирования емкости более 3 000 ГБ в день, свяжитесь с нами по адресу LAIngestionRate@microsoft.com . Дополнительные сведения о расходах на кластер см. в статье [Управление затратами для log Analytics кластеров](./manage-cost-storage.md#log-analytics-dedicated-clusters) .

После создания ресурса *кластера* можно изменить дополнительные свойства, такие как *SKU*, * кэйваултпропертиес или *биллингтипе*. Дополнительные сведения см. ниже.

У вас может быть до 2 активных кластеров на одну подписку для каждого региона. Если кластер удален, он по-прежнему резервируется в течение 14 дней. Можно использовать до 4 зарезервированных кластеров на одну подписку в каждом регионе (активная или недавно удаленная).

> [!WARNING]
> Создание кластера вызывает выделение ресурсов и подготовку. Выполнение этой операции может занять несколько часов. Рекомендуется выполнять его асинхронно.

Учетная запись пользователя, которая создает кластеры, должна иметь стандартное разрешение на создание ресурсов Azure: `Microsoft.Resources/deployments/*` и разрешение на запись в кластере `Microsoft.OperationalInsights/clusters/write` , используя назначения ролей для этого конкретного действия или `Microsoft.OperationalInsights/*` `*/write` .

### <a name="create"></a>Создание 

**PowerShell**

```powershell
New-AzOperationalInsightsCluster -ResourceGroupName {resource-group-name} -ClusterName {cluster-name} -Location {region-name} -SkuCapacity {daily-ingestion-gigabyte} -AsJob

# Check when the job is done
Get-Job -Command "New-AzOperationalInsightsCluster*" | Format-List -Property *
```

**REST**

*Call* 
```rst
PUT https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.OperationalInsights/clusters/<cluster-name>?api-version=2020-08-01
Authorization: Bearer <token>
Content-type: application/json

{
  "identity": {
    "type": "systemAssigned"
    },
  "sku": {
    "name": "capacityReservation",
    "Capacity": 1000
    },
  "properties": {
    "billingType": "cluster",
    },
  "location": "<region-name>",
}
```

*Ответ*

Должно быть 202 (принято) и заголовком.

### <a name="check-cluster-provisioning-status"></a>Проверка состояния подготовки кластера

Подготовка кластера Log Analytics занимает некоторое время. Проверить состояние подготовки можно несколькими способами.

- Выполните команду PowerShell Get-AzOperationalInsightsCluster с именем группы ресурсов и проверьте свойство ProvisioningState. Значение *провисионингаккаунт* во время подготовки и *успешного* завершения.
  ```powershell
  New-AzOperationalInsightsCluster -ResourceGroupName {resource-group-name} 
  ```

- Скопируйте значение URL-адреса Azure-AsyncOperation из ответа и выполните проверку состояния асинхронных операций.

- Отправьте запрос GET на ресурс *Кластер* и просмотрите значение *provisioningState*. Значение *провисионингаккаунт* во время подготовки и *успешного* завершения.

   ```rst
   GET https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.OperationalInsights/clusters/<cluster-name>?api-version=2020-08-01
   Authorization: Bearer <token>
   ```

   **Ответ**

   ```json
   {
     "identity": {
       "type": "SystemAssigned",
       "tenantId": "tenant-id",
       "principalId": "principal-id"
       },
     "sku": {
       "name": "capacityReservation",
       "capacity": 1000,
       "lastSkuUpdate": "Sun, 22 Mar 2020 15:39:29 GMT"
       },
     "properties": {
       "provisioningState": "ProvisioningAccount",
       "billingType": "cluster",
       "clusterId": "cluster-id"
       },
     "id": "/subscriptions/subscription-id/resourceGroups/resource-group-name/providers/Microsoft.OperationalInsights/clusters/cluster-name",
     "name": "cluster-name",
     "type": "Microsoft.OperationalInsights/clusters",
     "location": "region-name"
   }
   ```

Идентификатор GUID *principalId* создается управляемой службой удостоверений для ресурса *кластера* .

## <a name="link-a-workspace-to-cluster"></a>Связывание рабочей области с кластером

Если Рабочая область связана с выделенным кластером, новые данные, полученные в рабочей области, направляются в новый кластер, а существующие данные остаются в существующем кластере. Если выделенный кластер шифруется с помощью управляемых клиентом ключей (CMK), с ключом шифруются только новые данные. Система является абстрактным различием от пользователей, и пользователи просто запрашивают рабочую область как обычно, а система выполняет запросы между кластерами в серверной части.

Кластер может быть связан с рабочими областями до 100. Связанные рабочие области находятся в том же регионе, что и кластер. Чтобы обеспечить защиту серверной части системы и избежать фрагментации данных, Рабочая область не может быть связана с кластером более чем дважды в месяц.

Для выполнения операции связывания необходимо иметь разрешения на запись в рабочую область и ресурс *кластера* :

- В рабочей области: *Microsoft. OperationalInsights/рабочие области/запись*
- В ресурсе *кластера* : *Microsoft. OperationalInsights/Clusters/Write*

В отличие от аспектов выставления счетов, связанная Рабочая область хранит собственные параметры, такие как продолжительность хранения данных.
Рабочая область и кластер могут находиться в разных подписках. Рабочая область и кластер могут находиться в разных клиентах, если Azure Лигхсаусе используется для их параллельного отображения с одним клиентом.

При любой операции кластера связывание рабочей области может выполняться только после завершения подготовки кластера Log Analytics.

> [!WARNING]
> Чтобы связать рабочую область с кластером, необходимо синхронизировать несколько серверных компонентов и гарантировать расконсервации кэша. Выполнение этой операции может занять до двух часов. Мы рекомендуем запускать его асинхронно.


**PowerShell**

Используйте следующую команду PowerShell для связи с кластером:

```powershell
# Find cluster resource ID
$clusterResourceId = (Get-AzOperationalInsightsCluster -ResourceGroupName {resource-group-name} -ClusterName {cluster-name}).id

# Link the workspace to the cluster
Set-AzOperationalInsightsLinkedService -ResourceGroupName {resource-group-name} -WorkspaceName {workspace-name} -LinkedServiceName cluster -WriteAccessResourceId $clusterResourceId -AsJob

# Check when the job is done
Get-Job -Command "Set-AzOperationalInsightsLinkedService" | Format-List -Property *
```


**REST**

Используйте следующий вызов функции RESTFUL, чтобы связать с кластером:

*Отправить*

```rst
PUT https://management.azure.com/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/microsoft.operationalinsights/workspaces/<workspace-name>/linkedservices/cluster?api-version=2020-08-01 
Authorization: Bearer <token>
Content-type: application/json

{
  "properties": {
    "WriteAccessResourceId": "/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/microsoft.operationalinsights/clusters/<cluster-name>"
    }
}
```

*Ответ*

202 (принято) и заголовок.

### <a name="check-workspace-link-status"></a>Проверить состояние ссылки на рабочую область
  
Если вы используете ключи, управляемые клиентом, полученные данные будут храниться в зашифрованном виде с помощью управляемого ключа после операции связывания, что может занять до 90 минут. 

Проверить состояние сопоставления рабочей области можно двумя способами.

- Скопируйте значение URL-адреса Azure-AsyncOperation из ответа и выполните проверку состояния асинхронных операций.

- Выполните операцию Get в рабочей области и проверьте, есть ли в ответе свойство *клустерресаурцеид* в разделе *Features*.

**CLI**

```azurecli
az monitor log-analytics cluster show --resource-group "resource-group-name" --name "cluster-name"
```

**PowerShell**

```powershell
Get-AzOperationalInsightsWorkspace -ResourceGroupName "resource-group-name" -Name "workspace-name"
```

**REST**

*Call*

```rest
GET https://management.azure.com/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/microsoft.operationalinsights/workspaces/<workspace-name>?api-version=2020-08-01
Authorization: Bearer <token>
```

*Ответ*

```json
{
  "properties": {
    "source": "Azure",
    "customerId": "workspace-name",
    "provisioningState": "Succeeded",
    "sku": {
      "name": "pricing-tier-name",
      "lastSkuUpdate": "Tue, 28 Jan 2020 12:26:30 GMT"
    },
    "retentionInDays": 31,
    "features": {
      "legacy": 0,
      "searchVersion": 1,
      "enableLogAccessUsingOnlyResourcePermissions": true,
      "clusterResourceId": "/subscriptions/subscription-id/resourceGroups/resource-group-name/providers/Microsoft.OperationalInsights/clusters/cluster-name"
    },
    "workspaceCapping": {
      "dailyQuotaGb": -1.0,
      "quotaNextResetTime": "Tue, 28 Jan 2020 14:00:00 GMT",
      "dataIngestionStatus": "RespectQuota"
    }
  },
  "id": "/subscriptions/subscription-id/resourcegroups/resource-group-name/providers/microsoft.operationalinsights/workspaces/workspace-name",
  "name": "workspace-name",
  "type": "Microsoft.OperationalInsights/workspaces",
  "location": "region-name"
}
```

## <a name="change-cluster-properties"></a>Изменение свойств кластера

После создания ресурса *кластера* и его полной подготовки можно изменить дополнительные свойства на уровне кластера с помощью PowerShell или REST API. Кроме свойств, доступных во время создания кластера, дополнительные свойства можно задать только после подготовки кластера.

- **кэйваултпропертиес** — обновляет ключ в Azure Key Vault. См. раздел [Обновление кластера с подробными сведениями об идентификаторе ключа](../logs/customer-managed-keys.md#update-cluster-with-key-identifier-details). Он содержит следующие параметры: *KeyVaultUri*, *keyName*, *кэйверсион*. 
- **биллингтипе** — свойство *биллингтипе* определяет атрибуты выставления счетов для ресурса *кластера* и его данных:
  - **Кластер** (по умолчанию). затраты на резервирование емкости кластера приводятся к ресурсу *кластера* .
  - **Рабочие области** . затраты на резервирование ресурсов для кластера настраиваются пропорционально рабочим областям кластера, при этом ресурс *кластера* выставляет счет за использование, если общее количество принимаемых данных за день находится под резервированием емкости. Дополнительные сведения о модели ценообразования кластера см. в разделе [log Analytics выделенные кластеры](./manage-cost-storage.md#log-analytics-dedicated-clusters) . 

> [!NOTE]
> Свойство *биллингтипе* не поддерживается в PowerShell.

### <a name="get-all-clusters-in-resource-group"></a>Получение всех кластеров в группе ресурсов
  
**CLI**

```azurecli
az monitor log-analytics cluster list --resource-group "resource-group-name"
```

**PowerShell**

```powershell
Get-AzOperationalInsightsCluster -ResourceGroupName "resource-group-name"
```

**REST**

*Call*

  ```rst
  GET https://management.azure.com/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/Microsoft.OperationalInsights/clusters?api-version=2020-08-01
  Authorization: Bearer <token>
  ```

*Ответ*
  
  ```json
  {
    "value": [
      {
        "identity": {
          "type": "SystemAssigned",
          "tenantId": "tenant-id",
          "principalId": "principal-Id"
        },
        "sku": {
          "name": "capacityReservation",
          "capacity": 1000,
          "lastSkuUpdate": "Sun, 22 Mar 2020 15:39:29 GMT"
          },
        "properties": {
           "keyVaultProperties": {
              "keyVaultUri": "https://key-vault-name.vault.azure.net",
              "keyName": "key-name",
              "keyVersion": "current-version"
              },
          "provisioningState": "Succeeded",
          "billingType": "cluster",
          "clusterId": "cluster-id"
        },
        "id": "/subscriptions/subscription-id/resourcegroups/resource-group-name/providers/microsoft.operationalinsights/workspaces/workspace-name",
        "name": "cluster-name",
        "type": "Microsoft.OperationalInsights/clusters",
        "location": "region-name"
      }
    ]
  }
  ```

### <a name="get-all-clusters-in-subscription"></a>Получить все кластеры в подписке

**CLI**

```azurecli
az monitor log-analytics cluster list
```

**PowerShell**

```powershell
Get-AzOperationalInsightsCluster
```

**REST**

*Call*

```rst
GET https://management.azure.com/subscriptions/<subscription-id>/providers/Microsoft.OperationalInsights/clusters?api-version=2020-08-01
Authorization: Bearer <token>
```
    
*Ответ*
    
То же, что и для "кластеров в группе ресурсов", но в области подписки.



### <a name="update-capacity-reservation-in-cluster"></a>Обновление резервирования емкости в кластере

Когда объем данных в связанных рабочих областях изменяется с течением времени и необходимо соответствующим образом обновить уровень резервирования емкости. Емкость указывается в единицах ГБ и может иметь значения 1000 ГБ/день или более с шагом в 100 ГБ/день. Обратите внимание, что вам не нужно указывать полный текст запроса на оставшуюся часть, но должен быть указан номер SKU.

**CLI**

```azurecli
az monitor log-analytics cluster update --name "cluster-name" --resource-group "resource-group-name" --sku-capacity 1000
```

**PowerShell**

```powershell
Update-AzOperationalInsightsCluster -ResourceGroupName "resource-group-name" -ClusterName "cluster-name" -SkuCapacity 1000
```

**REST**

*Call*

  ```rst
  PATCH https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.OperationalInsights/clusters/<cluster-name>?api-version=2020-08-01
  Authorization: Bearer <token>
  Content-type: application/json

  {
    "sku": {
      "name": "capacityReservation",
      "Capacity": 2000
    }
  }
  ```

### <a name="update-billingtype-in-cluster"></a>Обновление Биллингтипе в кластере

Свойство *биллингтипе* определяет атрибуты выставления счетов для кластера и его данных:
- *кластер* (по умолчанию) — выставление счетов относится к подписке, в которой размещен ваш ресурс "Кластер";
- *рабочие области* — выставление счетов относится к подпискам, в которых пропорционально размещаются ваши рабочие области.

**REST**

*Call*

  ```rst
  PATCH https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.OperationalInsights/clusters/<cluster-name>?api-version=2020-08-01
  Authorization: Bearer <token>
  Content-type: application/json

  {
    "properties": {
      "billingType": "cluster",
      }  
  }
  ```

### <a name="unlink-a-workspace-from-cluster"></a>Отмена связи рабочей области с кластером

Вы можете удалить связь рабочей области с кластером. После разрыва связи с рабочей областью из кластера новые данные, связанные с этой рабочей областью, не отправляются в выделенный кластер. Кроме того, выставление счетов за рабочую область больше не выполняется через кластер. Старые данные несвязанной рабочей области могут остаться в кластере. Если эти данные шифруются с помощью ключей, управляемых клиентом (CMK), то сохраняются Key Vault секреты. Система абстрагирует это изменение от Log Analytics пользователей. Пользователи могут просто запросить рабочую область, как обычно. Система выполняет запросы между кластерами в серверной части по мере необходимости без указания пользователям.  

> [!WARNING] 
> Для конкретной рабочей области в течение месяца существует ограничение в две операции связывания. Примите во внимание время, чтобы учесть и спланировать отсоединение действий соответствующим образом.

**CLI**

```azurecli
az monitor log-analytics workspace linked-service delete --resource-group "resource-group-name" --workspace-name "MyWorkspace" --name cluster
```

**PowerShell**

Используйте следующую команду PowerShell для разрыва связи рабочей области с кластером:

```powershell
# Unlink a workspace from cluster
Remove-AzOperationalInsightsLinkedService -ResourceGroupName {resource-group-name} -WorkspaceName {workspace-name} -LinkedServiceName cluster
```

### <a name="delete-cluster"></a>Удаление кластера

Выделенный ресурс кластера можно удалить. Необходимо разорвать связь всех рабочих областей с кластером, прежде чем удалять их. Чтобы выполнить эту операцию, требуются разрешения на "запись" для ресурса *Кластер*. 

После удаления ресурса кластера физический кластер переходит в процесс очистки и удаления. При удалении кластера удаляются все данные, хранящиеся в кластере. Данные могут быть из рабочих областей, которые были связаны с кластером в прошлом.

Ресурс *Кластер*, удаленный за последние 14 дней, находится в состоянии обратимого удаления, и его можно восстановить вместе с его данными. Так как все рабочие области не были связаны с *кластерным* ресурсом с удалением ресурса *кластера* , необходимо повторно связать рабочие области после восстановления. Пользователь не может выполнить операцию восстановления, обратившись к каналу Майкрософт или поддержку запросов на восстановление.

В течение 14 дней после удаления имя ресурса кластера зарезервировано и не может использоваться другими ресурсами.

> [!WARNING] 
> Существует ограничение в три кластера на подписку. В этом разделе учитываются как активные, так и обратимо удаленные кластеры. Клиенты не должны создавать повторно выполняемые процедуры, создающие и удаляющие кластеры. Он оказывает значительное влияние на Log Analytics серверных систем.

**PowerShell**

Для удаления кластера используйте следующую команду PowerShell:

  ```powershell
  Remove-AzOperationalInsightsCluster -ResourceGroupName "resource-group-name" -ClusterName "cluster-name"
  ```

**REST**

Для удаления кластера используйте следующий вызов функции RESTFUL:

  ```rst
  DELETE https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.OperationalInsights/clusters/<cluster-name>?api-version=2020-08-01
  Authorization: Bearer <token>
  ```

  **Ответ**

  200 ОК

## <a name="limits-and-constraints"></a>Ограничения

- Максимальное число активных кластеров на регион и подписку равно 2

- Максимальное число зарезервированных кластеров (активных или недавно удаленных) для каждого региона и подписки равно 4. 

- Максимальное число связанных рабочих областей в кластере — 1000

- Вы можете связать рабочую область с кластером, а затем удалить ее связь. Число операций ссылок на рабочую область в конкретной рабочей области ограничено 2 в течение 30 дней.

- Перемещение кластера в другую группу ресурсов или подписку сейчас не поддерживается.

- В настоящее время защищенное хранилище недоступно в Китае. 

- [Двойное шифрование](../../storage/common/storage-service-encryption.md#doubly-encrypt-data-with-infrastructure-encryption) настраивается автоматически для кластеров, созданных с 2020 октября в поддерживаемых регионах. Вы можете проверить, настроено ли для кластера двойное шифрование, отправив запрос GET в кластер и отслеживая, что это `isDoubleEncryptionEnabled` значение `true` для кластеров с включенным двойным шифрованием. 
  - При создании кластера и получении ошибки "<имя региона> не поддерживает двойное шифрование для кластеров". Вы по-прежнему можете создать кластер без двойного шифрования, добавив `"properties": {"isDoubleEncryptionEnabled": false}` в текст запроса.
  - Параметр двойного шифрования нельзя изменить после создания кластера.

## <a name="troubleshooting"></a>Устранение неполадок

- Если при создании кластера возникает ошибка конфликта, возможно, кластер был удален за последние 14 дней и он находится в состоянии обратимого удаления. Имя кластера остается зарезервированным в течение периода обратимого удаления, и вы не можете создать новый кластер с таким именем. Имя освобождается после периода обратимого удаления, когда кластер окончательно удаляется.

- Если вы обновляете кластер, пока кластер находится в состоянии подготовки или обновления, обновление завершится ошибкой.

- Некоторые операции являются длительными и могут занять некоторое время — это создание кластера, обновление ключа кластера и удаление кластера. Проверить состояние операции можно двумя способами.
  - При использовании функции "ОСТАВШАЯся" скопируйте значение Azure-AsyncOperation URL-адрес из ответа и выполните [проверку состояния асинхронных операций](#asynchronous-operations-and-status-check).
  - Отправьте запрос GET в кластер или рабочую область и просмотрите ответ. Например, несвязанная Рабочая область не будет иметь *клустерресаурцеид* в разделе " *компоненты*".

- Ссылка на рабочую область кластера завершится ошибкой, если она связана с другим кластером.

- Сообщения об ошибках
  
  Создание кластера:
  -  400--недопустимое имя кластера. Имя кластера может содержать символы a – z, A – Z, 0-9 и длину 3-63.
  -  400--текст запроса имеет значение null или имеет неправильный формат.
  -  400--недопустимое имя SKU. Задайте для номера SKU имя КапаЦитиресерватион.
  -  400--Емкость предоставлена, но SKU не КапаЦитиресерватион. Задайте для номера SKU имя КапаЦитиресерватион.
  -  400--отсутствует емкость в номере SKU. Задайте для параметра емкость значение 1000 или выше на шаге 100 (ГБ).
  -  400--Емкость SKU не входит в диапазон. Должно быть не менее 1000 и не больше максимальной допустимой емкости, доступной в разделе "использование и оценочная стоимость" в рабочей области.
  -  400--Емкость заблокирована в течение 30 дней. Уменьшение емкости разрешено через 30 дней после обновления.
  -  400--номер SKU не задан. Задайте для номера SKU значение КапаЦитиресерватион и Capacity, равное 1000 или выше, на шаге 100 (ГБ).
  -  400--Identity имеет значение null или пусто. Задайте удостоверение с типом systemAssigned.
  -  400--Кэйваултпропертиес устанавливаются при создании. Обновление Кэйваултпропертиес после создания кластера.
  -  400--невозможно выполнить операцию сейчас. Асинхронная операция находится в состоянии, отличном от "завершено". Перед выполнением операции обновления кластер должен завершить свою операцию.

  Обновление кластера
  -  400--кластер находится в состоянии удаления. Асинхронная операция выполняется. Перед выполнением операции обновления кластер должен завершить свою операцию.
  -  400--Кэйваултпропертиес не является пустым, но имеет неправильный формат. См. [раздел об обновлении идентификатора ключа](../logs/customer-managed-keys.md#update-cluster-with-key-identifier-details).
  -  400--не удалось проверить ключ в Key Vault. Возможно, из-за отсутствия разрешений или если ключ не существует. Убедитесь, что [Политика ключа и доступа задана](../logs/customer-managed-keys.md#grant-key-vault-permissions) в Key Vault.
  -  400-ключ не может быть восстановлен. Для Key Vault должно быть задано обратимое удаление и защита от удаления. См. [документацию по Key Vault](../../key-vault/general/soft-delete-overview.md)
  -  400--невозможно выполнить операцию сейчас. Дождитесь завершения асинхронной операции и повторите попытку.
  -  400--кластер находится в состоянии удаления. Дождитесь завершения асинхронной операции и повторите попытку.

  Получение кластера:
    -  404--кластер не найден, возможно, кластер удален. Если вы попытаетесь создать кластер с этим именем и получить конфликт, кластер будет обратимо удален в течение 14 дней. Вы можете обратиться в службу поддержки, чтобы восстановить ее, или использовать другое имя для создания нового кластера. 

  Удаление кластера
    -  409--не удается удалить кластер в состоянии подготовки. Дождитесь завершения асинхронной операции и повторите попытку.

  Ссылка на рабочую область:
  -  404--Рабочая область не найдена. Указанная Рабочая область не существует или была удалена.
  -  409--ссылка на рабочую область или отмена связи в процессе.
  -  400--кластер не найден, указанный кластер не существует или был удален. Если вы попытаетесь создать кластер с этим именем и получить конфликт, кластер будет обратимо удален в течение 14 дней. Вы можете обратиться в службу поддержки, чтобы восстановить ее.

  Отмена связи рабочей области:
  -  404--Рабочая область не найдена. Указанная Рабочая область не существует или была удалена.
  -  409--ссылка на рабочую область или отмена связи в процессе.

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о [log Analytics выставлении счетов за выделенный кластер](./manage-cost-storage.md#log-analytics-dedicated-clusters)
- Сведения о [правильном проектировании рабочих областей log Analytics](../logs/design-logs-deployment.md)