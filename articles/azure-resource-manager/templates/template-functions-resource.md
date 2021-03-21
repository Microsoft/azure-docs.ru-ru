---
title: Функции шаблонов — ресурсы
description: Описывает функции, используемые в шаблоне Azure Resource Manager (шаблон ARM) для получения значений о ресурсах.
ms.topic: conceptual
ms.date: 02/10/2021
ms.openlocfilehash: da85308e7d214f198b29b40bc380a4d33947c865
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100364568"
---
# <a name="resource-functions-for-arm-templates"></a>Функции ресурсов для шаблонов ARM

Диспетчер ресурсов предоставляет следующие функции для получения значений ресурсов в шаблоне Azure Resource Manager (шаблон ARM):

* [extensionResourceId](#extensionresourceid)
* [list*](#list)
* [пиккзонес](#pickzones)
* [providers](#providers)
* [reference](#reference)
* [resourceGroup](#resourcegroup)
* [resourceId](#resourceid)
* [subscription](#subscription)
* [subscriptionResourceId](#subscriptionresourceid)
* [tenantResourceId](#tenantresourceid)

Получение значений параметров, переменных или текущего развертывания описано в разделе [Функции для параметров развертывания](template-functions-deployment.md).

[!INCLUDE [Bicep preview](../../../includes/resource-manager-bicep-preview.md)]

## <a name="extensionresourceid"></a>extensionResourceId

`extensionResourceId(resourceId, resourceType, resourceName1, [resourceName2], ...)`

Возвращает идентификатор [ресурса расширения](../management/extension-resource-types.md), который является типом ресурса, применяемого к другому ресурсу для добавления к его возможностям.

### <a name="parameters"></a>Параметры

| Параметр | Обязательно | Тип | Описание |
|:--- |:--- |:--- |:--- |
| resourceId |Да |строка |Идентификатор ресурса, к которому применяется ресурс расширения. |
| тип_ресурса |Да |строка |Тип ресурса, включая пространство имен поставщика ресурсов. |
| имя_ресурса1 |Да |строка |Имя ресурса. |
| имя_ресурса2 |Нет |строка |Следующий сегмент имени ресурса, если он необходим. |

Продолжайте добавлять имена ресурсов в качестве параметров, если тип ресурса включает больше сегментов.

### <a name="return-value"></a>Возвращаемое значение

Базовый формат идентификатора ресурса, возвращаемого этой функцией:

```json
{scope}/providers/{extensionResourceProviderNamespace}/{extensionResourceType}/{extensionResourceName}
```

Сегмент области зависит от расширяемого ресурса.

Когда ресурс расширения применяется к **ресурсу**, идентификатор ресурса возвращается в следующем формате:

```json
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/{baseResourceProviderNamespace}/{baseResourceType}/{baseResourceName}/providers/{extensionResourceProviderNamespace}/{extensionResourceType}/{extensionResourceName}
```

Когда ресурс расширения применяется к **группе ресурсов**, используется следующий формат:

```json
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/{extensionResourceProviderNamespace}/{extensionResourceType}/{extensionResourceName}
```

Когда ресурс расширения применяется к **подписке**, используется следующий формат:

```json
/subscriptions/{subscriptionId}/providers/{extensionResourceProviderNamespace}/{extensionResourceType}/{extensionResourceName}
```

Когда ресурс расширения применяется к **группе управления**, используется следующий формат:

```json
/providers/Microsoft.Management/managementGroups/{managementGroupName}/providers/{extensionResourceProviderNamespace}/{extensionResourceType}/{extensionResourceName}
```

### <a name="extensionresourceid-example"></a>Пример extensionResourceId

В следующем примере возвращается идентификатор ресурса для блокировки группы ресурсов.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "lockName": {
      "type": "string"
    }
  },
  "variables": {},
  "resources": [],
  "outputs": {
    "lockResourceId": {
      "type": "string",
      "value": "[extensionResourceId(resourceGroup().Id , 'Microsoft.Authorization/locks', parameters('lockName'))]"
    }
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param lockName string

output lockResourceId string = extensionResourceId(resourceGroup().Id, 'Microsoft.Authorization/locks', lockName)
```

---

Определение пользовательской политики, развернутое в группе управления, реализуется как ресурс расширения. Чтобы создать и назначить политику, разверните следующий шаблон в группе управления.

# <a name="json"></a>[JSON](#tab/json)

:::code language="json" source="~/quickstart-templates/managementgroup-deployments/mg-policy/azuredeploy.json":::

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param targetMG string
param allowedLocations array = [
  'australiaeast'
  'australiasoutheast'
  'australiacentral'
]

var mgScope = tenantResourceId('Microsoft.Management/managementGroups', targetMG)
var policyDefinition = 'LocationRestriction'

resource myDefinition 'Microsoft.Authorization/policyDefinitions@2019-09-01' = {
  name: policyDefinition
  properties: {
    policyType: 'Custom'
    mode: 'All'
    parameters: {}
    policyRule: {
      'if': {
        'not': {
          'field': 'location'
          'in': allowedLocations
        }
      }
      'then': {
        'effect': 'deny'
      }
    }
  }
}

resource myAssignment 'Microsoft.Authorization/policyAssignments@2019-09-01' = {
  name: 'location-lock'
  properties: {
    scope: mgScope
    policyDefinitionId: extensionResourceId(mgScope, 'Microsoft.Authorization/policyDefinitions', policyDefinition)
  }
}
```

---

Встроенные определения политик — это ресурсы уровня клиента. Пример развертывания встроенного определения политики см. в разделе [тенантресаурцеид](#tenantresourceid).

<a id="listkeys"></a>
<a id="list"></a>

## <a name="list"></a>list*

`list{Value}(resourceName or resourceIdentifier, apiVersion, functionValues)`

Синтаксис для этой функции зависит от имени из списка операций. Каждая реализация возвращает значения для типа ресурса, который поддерживает операцию list. Имя операции должно начинаться с `list` и может иметь суффикс. Некоторые распространенные случаи использования: `list` ,, `listKeys` `listKeyValue` и `listSecrets` .

### <a name="parameters"></a>Параметры

| Параметр | Обязательно | Тип | Описание |
|:--- |:--- |:--- |:--- |
| имя_ресурса или идентификатор_ресурса |Да |строка |Уникальный идентификатор ресурса. |
| версия_API |Да |строка |Версия API для состояния среды выполнения ресурса. Как правило, указывается в формате **гггг-мм-дд**. |
| functionValues |Нет |объект | Объект, содержащий значения для функции. Предоставляйте этот объект только для функций, которые поддерживают прием объекта с параметрами, например **listAccountSas** в учетной записи хранения. В этой статье показан пример передачи значения функции. |

### <a name="valid-uses"></a>Допустимые варианты использования

Функции List можно использовать в свойствах определения ресурса. Не используйте функцию List, которая предоставляет конфиденциальную информацию в разделе Outputs шаблона. Выходные значения хранятся в журнале развертывания и могут быть получены злоумышленником.

При использовании с [итерацией свойства](copy-properties.md) можно использовать функции list для `input`, так как выражение назначается свойству ресурса. Их нельзя использовать с `count`, так как count должен быть определен до разрешения функции list.

### <a name="implementations"></a>Варианты реализации решения

Следующая таблица содержит возможные способы использования list*.

| Тип ресурса | Имя функции |
| ------------- | ------------- |
| Microsoft. надстройки/Суппортпровидерс | листсуппортпланинфо |
| Microsoft.AnalysisServices/servers | [listGatewayStatus](/rest/api/analysisservices/servers/listgatewaystatus) |
| Microsoft. ApiManagement/Service/Аусоризатионсерверс | [листсекретс](/rest/api/apimanagement/2019-12-01/authorizationserver/listsecrets) |
| Microsoft. ApiManagement/служба/шлюзы | [listKeys](/rest/api/apimanagement/2019-12-01/gateway/listkeys) |
| Microsoft. ApiManagement/Service/Идентитипровидерс | [листсекретс](/rest/api/apimanagement/2019-12-01/identityprovider/listsecrets) |
| Microsoft. ApiManagement/Service/Намедвалуес | [Значение списка](/rest/api/apimanagement/2019-12-01/namedvalue/listvalue) |
| Microsoft. ApiManagement/Service/Опенидконнектпровидерс | [листсекретс](/rest/api/apimanagement/2019-12-01/openidconnectprovider/listsecrets) |
| Microsoft. ApiManagement/Service/Subscriptions | [листсекретс](/rest/api/apimanagement/2019-12-01/subscription/listsecrets) |
| Microsoft.AppConfiguration/configurationStores | [ListKeys](/rest/api/appconfiguration/configurationstores/listkeys) |
| Microsoft.AppPlatform/Spring | [листтесткэйс](/rest/api/azurespringcloud/services/listtestkeys) |
| Microsoft.Automation/automationAccounts | [listKeys](/rest/api/automation/keys/listbyautomationaccount) |
| Microsoft.Batch/batchAccounts | [listkeys](/rest/api/batchmanagement/batchaccount/getkeys) |
| Microsoft.BatchAI/workspaces/experiments/jobs | [listoutputfiles](/rest/api/batchai/jobs/listoutputfiles) |
| Microsoft.Blockchain/blockchainMembers | [listApiKeys](/rest/api/blockchain/2019-06-01-preview/blockchainmembers/listapikeys) |
| Microsoft.Blockchain/blockchainMembers/transactionNodes | [listApiKeys](/rest/api/blockchain/2019-06-01-preview/transactionnodes/listapikeys) |
| Microsoft. Ботсервице/Ботсервицес/каналы | [листчаннелвискэйс](https://github.com/Azure/azure-rest-api-specs/blob/master/specification/botservice/resource-manager/Microsoft.BotService/stable/2020-06-02/botservice.json#L553) |
| Microsoft.Cache/redis | [listKeys](/rest/api/redis/redis/listkeys) |
| Microsoft.CognitiveServices/accounts | [listKeys](/rest/api/cognitiveservices/accountmanagement/accounts/listkeys) |
| Microsoft.ContainerRegistry/registries | [listBuildSourceUploadUrl](/rest/api/containerregistry/registries%20(tasks)/getbuildsourceuploadurl) |
| Microsoft.ContainerRegistry/registries | [listCredentials](/rest/api/containerregistry/registries/listcredentials) |
| Microsoft.ContainerRegistry/registries | [listUsages](/rest/api/containerregistry/registries/listusages) |
| Microsoft. ContainerRegistry/реестров/ажентпулс | листкуеуестатус |
| Microsoft. ContainerRegistry/реестров/Буилдтаскс | листсаурцерепоситорипропертиес |
| Microsoft. ContainerRegistry/registrys/Буилдтаскс/шаги | листбуилдаргументс |
| Microsoft. ContainerRegistry/реестров/таскрунс | listDetails |
| Microsoft.ContainerRegistry/registries/webhooks | [listEvents](/rest/api/containerregistry/webhooks/listevents) |
| Microsoft.ContainerRegistry/registries/runs | [listLogSasUrl](/rest/api/containerregistry/runs/getlogsasurl) |
| Microsoft.ContainerRegistry/registries/tasks | [listDetails](/rest/api/containerregistry/tasks/getdetails) |
| Microsoft.ContainerService/managedClusters | [listClusterAdminCredential](/rest/api/aks/managedclusters/listclusteradmincredentials) |
| Microsoft.ContainerService/managedClusters | [листклустермониторингусеркредентиал](/rest/api/aks/managedclusters/listclustermonitoringusercredentials) |
| Microsoft.ContainerService/managedClusters | [listClusterUserCredential](/rest/api/aks/managedclusters/listclusterusercredentials) |
| Microsoft.ContainerService/managedClusters/accessProfiles | [listCredential](/rest/api/aks/managedclusters/getaccessprofile) |
| Microsoft.DataBox/jobs | listCredentials |
| Microsoft.DataFactory/datafactories/gateways | listauthkeys |
| Microsoft.DataFactory/factories/integrationruntimes | [listauthkeys](/rest/api/datafactory/integrationruntimes/listauthkeys) |
| Microsoft.DataLakeAnalytics/accounts/storageAccounts/Containers | [listSasTokens](/rest/api/datalakeanalytics/storageaccounts/listsastokens) |
| Microsoft.DataShare/accounts/shares | [listSynchronizations](/rest/api/datashare/shares/listsynchronizations) |
| Microsoft.DataShare/accounts/shareSubscriptions | [listSourceShareSynchronizationSettings](/rest/api/datashare/sharesubscriptions/listsourcesharesynchronizationsettings) |
| Microsoft.DataShare/accounts/shareSubscriptions | [listSynchronizationDetails](/rest/api/datashare/sharesubscriptions/listsynchronizationdetails) |
| Microsoft.DataShare/accounts/shareSubscriptions | [listSynchronizations](/rest/api/datashare/sharesubscriptions/listsynchronizations) |
| Microsoft.Devices/iotHubs | [listkeys](/rest/api/iothub/iothubresource/listkeys) |
| Microsoft.Devices/iotHubs/iotHubKeys | [listkeys](/rest/api/iothub/iothubresource/getkeysforkeyname) |
| Microsoft.Devices/provisioningServices/keys | [listkeys](/rest/api/iot-dps/iotdpsresource/listkeysforkeyname) |
| Microsoft.Devices/provisioningServices | [listkeys](/rest/api/iot-dps/iotdpsresource/listkeys) |
| Microsoft.DevTestLab/labs | [ListVhds](/rest/api/dtl/labs/listvhds) |
| Microsoft.DevTestLab/labs/schedules | [ListApplicable](/rest/api/dtl/schedules/listapplicable) |
| Microsoft.DevTestLab/labs/users/serviceFabrics | [ListApplicableSchedules](/rest/api/dtl/servicefabrics/listapplicableschedules) |
| Microsoft.DevTestLab/labs/virtualMachines | [ListApplicableSchedules](/rest/api/dtl/virtualmachines/listapplicableschedules) |
| Microsoft.DocumentDB/databaseAccounts | [listConnectionStrings](/rest/api/cosmos-db-resource-provider/2020-06-01-preview/databaseaccounts/listconnectionstrings) |
| Microsoft.DocumentDB/databaseAccounts | [listKeys](/rest/api/cosmos-db-resource-provider/2020-06-01-preview/databaseaccounts/listkeys) |
| Microsoft.DocУментдб/databaseAccounts/Нотебукворкспацес | [листконнектионинфо](/rest/api/cosmos-db-resource-provider/2020-06-01/notebookworkspaces/listconnectioninfo) |
| Microsoft.DomainRegistration | [listDomainRecommendations](/rest/api/appservice/domains/listrecommendations) |
| Microsoft.DomainRegistration/topLevelDomains | [listAgreements](/rest/api/appservice/topleveldomains/listagreements) |
| Microsoft.EventGrid/domains | [listKeys](/rest/api/eventgrid/version2020-06-01/domains/listsharedaccesskeys) |
| Microsoft.EventGrid/topics | [listKeys](/rest/api/eventgrid/version2020-06-01/topics/listsharedaccesskeys) |
| Microsoft.EventHub/namespaces/authorizationRules | [listkeys](/rest/api/eventhub) |
| Microsoft.EventHub/namespaces/disasterRecoveryConfigs/authorizationRules | [listkeys](/rest/api/eventhub) |
| Microsoft.EventHub/namespaces/eventhubs/authorizationRules | [listkeys](/rest/api/eventhub) |
| Microsoft.ImportExport/jobs | [listBitLockerKeys](/rest/api/storageimportexport/bitlockerkeys/list) |
| Microsoft.Kusto/Clusters/Databases | [ListPrincipals](/rest/api/azurerekusto/databases/listprincipals) |
| Microsoft.LabServices/users | [ListEnvironments](/rest/api/labservices/globalusers/listenvironments) |
| Microsoft.LabServices/users | [ListLabs](/rest/api/labservices/globalusers/listlabs) |
| Microsoft.Logic/integrationAccounts/agreements | [listContentCallbackUrl](/rest/api/logic/agreements/listcontentcallbackurl) |
| Microsoft.Logic/integrationAccounts/assemblies | [listContentCallbackUrl](/rest/api/logic/integrationaccountassemblies/listcontentcallbackurl) |
| Microsoft.Logic/integrationAccounts | [listCallbackUrl](/rest/api/logic/integrationaccounts/getcallbackurl) |
| Microsoft.Logic/integrationAccounts | [listKeyVaultKeys](/rest/api/logic/integrationaccounts/listkeyvaultkeys) |
| Microsoft.Logic/integrationAccounts/maps | [listContentCallbackUrl](/rest/api/logic/maps/listcontentcallbackurl) |
| Microsoft.Logic/integrationAccounts/partners | [listContentCallbackUrl](/rest/api/logic/partners/listcontentcallbackurl) |
| Microsoft.Logic/integrationAccounts/schemas | [listContentCallbackUrl](/rest/api/logic/schemas/listcontentcallbackurl) |
| Microsoft.Logic/workflows | [listCallbackUrl](/rest/api/logic/workflows/listcallbackurl) |
| Microsoft.Logic/workflows | [listSwagger](/rest/api/logic/workflows/listswagger) |
| Microsoft.Logic/workflows/runs/actions | [listExpressionTraces](/rest/api/logic/workflowrunactions/listexpressiontraces) |
| Microsoft.Logic/workflows/runs/actions/repetitions | [listExpressionTraces](/rest/api/logic/workflowrunactionrepetitions/listexpressiontraces) |
| Microsoft.Logic/workflows/triggers | [listCallbackUrl](/rest/api/logic/workflowtriggers/listcallbackurl) |
| Microsoft.Logic/workflows/versions/triggers | [listCallbackUrl](/rest/api/logic/workflowversions/listcallbackurl) |
| Microsoft.MachineLearning/webServices | [listkeys](/rest/api/machinelearning/webservices/listkeys) |
| Microsoft.MachineLearning/Workspaces | listworkspacekeys |
| Microsoft.MachineLearningServices/workspaces/computes | [listKeys](/rest/api/azureml/workspacesandcomputes/machinelearningcompute/listkeys) |
| Microsoft.MachineLearningServices/workspaces/computes | [listNodes](/rest/api/azureml/workspacesandcomputes/machinelearningcompute/listnodes) |
| Microsoft.MachineLearningServices/workspaces | [listKeys](/rest/api/azureml/workspacesandcomputes/workspaces/listkeys) |
| Microsoft.Maps/accounts | [listKeys](/rest/api/maps-management/accounts/listkeys) |
| Microsoft.Media/mediaservices/assets | [listContainerSas](/rest/api/media/assets/listcontainersas) |
| Microsoft.Media/mediaservices/assets | [listStreamingLocators](/rest/api/media/assets/liststreaminglocators) |
| Microsoft.Media/mediaservices/streamingLocators | [listContentKeys](/rest/api/media/streaminglocators/listcontentkeys) |
| Microsoft.Media/mediaservices/streamingLocators | [listPaths](/rest/api/media/streaminglocators/listpaths) |
| Microsoft.Network/applicationSecurityGroups | listIpConfigurations |
| Microsoft.NotificationHubs/Namespaces/authorizationRules | [listkeys](/rest/api/notificationhubs/namespaces/listkeys) |
| Microsoft.NotificationHubs/Namespaces/NotificationHubs/authorizationRules | [listkeys](/rest/api/notificationhubs/notificationhubs/listkeys) |
| Microsoft.OperationalInsights/workspaces | [list](/rest/api/loganalytics/workspaces/list) |
| Microsoft.OperationalInsights/workspaces | listKeys |
| Microsoft.PolicyInsights/remediations | [listDeployments](/rest/api/policy-insights/remediations/listdeploymentsatresourcegroup) |
| Microsoft. Редхатопеншифт/Опеншифтклустерс | [listCredentials](/rest/api/openshift/openshiftclusters/listcredentials) |
| Microsoft.Relay/namespaces/authorizationRules | [listkeys](/rest/api/relay/namespaces/listkeys) |
| Microsoft.Relay/namespaces/disasterRecoveryConfigs/authorizationRules | listkeys |
| Microsoft.Relay/namespaces/HybridConnections/authorizationRules | [listkeys](/rest/api/relay/hybridconnections/listkeys) |
| Microsoft.Relay/namespaces/WcfRelays/authorizationRules | [listkeys](/rest/api/relay/wcfrelays/listkeys) |
| Microsoft.Search/searchServices | [listAdminKeys](/rest/api/searchmanagement/adminkeys/get) |
| Microsoft.Search/searchServices | [listQueryKeys](/rest/api/searchmanagement/querykeys/listbysearchservice) |
| Microsoft.ServiceBus/namespaces/authorizationRules | [listkeys](/rest/api/servicebus/stable/namespaces%20-%20authorization%20rules/listkeys) |
| Microsoft.ServiceBus/namespaces/disasterRecoveryConfigs/authorizationRules | [listkeys](/rest/api/servicebus/stable/disasterrecoveryconfigs/listkeys) |
| Microsoft.ServiceBus/namespaces/queues/authorizationRules | [listkeys](/rest/api/servicebus/stable/queues%20-%20authorization%20rules/listkeys) |
| Microsoft.ServiceBus/namespaces/topics/authorizationRules | [listkeys](/rest/api/servicebus/stable/topics%20–%20authorization%20rules/listkeys) |
| Microsoft.SignalRService/SignalR | [listkeys](/rest/api/signalr/signalr/listkeys) |
| Microsoft.Storage/storageAccounts | [listAccountSas](/rest/api/storagerp/storageaccounts/listaccountsas) |
| Microsoft.Storage/storageAccounts | [listkeys](/rest/api/storagerp/storageaccounts/listkeys) |
| Microsoft.Storage/storageAccounts | [listServiceSas](/rest/api/storagerp/storageaccounts/listservicesas) |
| Microsoft.StorSimple/managers/devices | [listFailoverSets](/rest/api/storsimple/devices/listfailoversets) |
| Microsoft.StorSimple/managers/devices | [listFailoverTargets](/rest/api/storsimple/devices/listfailovertargets) |
| Microsoft.StorSimple/managers | [listActivationKey](/rest/api/storsimple/managers/getactivationkey) |
| Microsoft.StorSimple/managers | [listPublicEncryptionKey](/rest/api/storsimple/managers/getpublicencryptionkey) |
| Microsoft. синапсе/workspaces/Интегратионрунтимес | [листаускэйс](/rest/api/synapse/integrationruntimeauthkeys/list) |
| Microsoft.Web/connectionGateways | ListStatus |
| microsoft.web/connections | listconsentlinks |
| Microsoft.Web/customApis | listWsdlInterfaces |
| microsoft.web/locations | listwsdlinterfaces |
| microsoft.web/apimanagementaccounts/apis/connections | listconnectionkeys |
| microsoft.web/apimanagementaccounts/apis/connections | listsecrets |
| microsoft.web/sites/backups | [list](/rest/api/appservice/webapps/listbackups) |
| Microsoft.Web/sites/config | [list](/rest/api/appservice/webapps/listconfigurations) |
| microsoft.web/sites/functions | [listkeys](/rest/api/appservice/webapps/listfunctionkeys)
| microsoft.web/sites/functions | [listsecrets](/rest/api/appservice/webapps/listfunctionsecrets) |
| microsoft.web/sites/hybridconnectionnamespaces/relays | [listkeys](/rest/api/appservice/appserviceplans/listhybridconnectionkeys) |
| microsoft.web/sites | [listsyncfunctiontriggerstatus](/rest/api/appservice/webapps/listsyncfunctiontriggers) |
| microsoft.web/sites/slots/functions | [listsecrets](/rest/api/appservice/webapps/listfunctionsecretsslot) |
| microsoft.web/sites/slots/backups | [list](/rest/api/appservice/webapps/listbackupsslot) |
| Microsoft.Web/sites/slots/config | [list](/rest/api/appservice/webapps/listconfigurationsslot) |
| microsoft.web/sites/slots/functions | [listsecrets](/rest/api/appservice/webapps/listfunctionsecretsslot) |

Чтобы определить, какие типы ресурсов поддерживают операцию list, можно использовать следующие варианты:

* Просмотрите [операции REST API](/rest/api/) для поставщика ресурсов и найдите операции list. Например, в учетных записях хранения есть [операция listKeys](/rest/api/storagerp/storageaccounts).
* Воспользуйтесь командлетом PowerShell [Get-AzProviderOperation](/powershell/module/az.resources/get-azprovideroperation). В следующем примере извлекаются все операции list для учетных записей хранения:

  ```powershell
  Get-AzProviderOperation -OperationSearchString "Microsoft.Storage/*" | where {$_.Operation -like "*list*"} | FT Operation
  ```

* Используйте следующую команду Azure CLI, чтобы отфильтровать только операции list:

  ```azurecli
  az provider operation show --namespace Microsoft.Storage --query "resourceTypes[?name=='storageAccounts'].operations[].name | [?contains(@, 'list')]"
  ```

### <a name="return-value"></a>Возвращаемое значение

Возвращаемый объект зависит от используемой функции списка. Например, listKeys для учетной записи хранения возвращает следующий формат.

```json
{
  "keys": [
    {
      "keyName": "key1",
      "permissions": "Full",
      "value": "{value}"
    },
    {
      "keyName": "key2",
      "permissions": "Full",
      "value": "{value}"
    }
  ]
}
```

Другие функции list возвращают данные в других форматах. Чтобы просмотреть формат функции, включите ее в раздел outputs, как показано в примере шаблона.

### <a name="remarks"></a>Remarks

Укажите ресурс с помощью его имени или [функции resourceId](#resourceid). Если эта функция списка задана в том же шаблоне, с помощью которого выполняется развертывание ресурса, на который указывает ссылка, следует использовать имя ресурса.

При использовании функции **list** в ресурсе, который развернут условно, функция вычисляется, даже если ресурс не развернут. Если функция **list** ссылается на несуществующий ресурс, возникает ошибка. Используйте функцию **if**, чтобы убедиться, что функция вычисляется только при развернутом ресурсе. Пример шаблона, который использует функции if и list с условно развернутым ресурсом, см. в описании [функции if](template-functions-logical.md#if).

### <a name="list-example"></a>Пример функции list

В следующем примере используется listKeys при задании значения для [скриптов развертывания](deployment-script-template.md).

# <a name="json"></a>[JSON](#tab/json)

```json
"storageAccountSettings": {
  "storageAccountName": "[variables('storageAccountName')]",
  "storageAccountKey": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName')), '2019-06-01').keys[0].value]"
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
storageAccountSettings: {
  storageAccountName: storageAccountName
  storageAccountKey: listKeys(resourceId('Microsoft.Storage/storageAccounts', storageAccountName), '2019-06-01').keys[0].value
}

```

---

В следующем примере показана функция List, которая принимает параметр. В этом случае функция — **листаккаунтсас**. Передайте объект для времени окончания срока действия. Время окончания срока действия должно быть в будущем.

# <a name="json"></a>[JSON](#tab/json)

```json
"parameters": {
  "accountSasProperties": {
    "type": "object",
    "defaultValue": {
      "signedServices": "b",
      "signedPermission": "r",
      "signedExpiry": "2020-08-20T11:00:00Z",
      "signedResourceTypes": "s"
    }
  }
},
...
"sasToken": "[listAccountSas(parameters('storagename'), '2018-02-01', parameters('accountSasProperties')).accountSasToken]"
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param accountSasProperties object {
  default: {
    signedServices: 'b'
    signedPermission: 'r'
    signedExpiry: '2020-08-20T11:00:00Z'
    signedResourceTypes: 's'
  }
}
...
sasToken: listAccountSas(storagename, '2018-02-01', accountSasProperties).accountSasToken
```

---

## <a name="pickzones"></a>пиккзонес

`pickZones(providerNamespace, resourceType, location, [numberOfZones], [offset])`

Определяет, поддерживает ли тип ресурса зоны для региона.

### <a name="parameters"></a>Параметры

| Параметр | Обязательно | Тип | Описание |
|:--- |:--- |:--- |:--- |
| пространство_имен_поставщика | Да | строка | Пространство имен поставщика ресурсов для типа ресурса, для которого проверяется поддержка зоны. |
| тип_ресурса | Да | строка | Тип ресурса для проверки поддержки зоны. |
| location | Да | строка | Регион, в котором проверяется поддержка зоны. |
| нумберофзонес | Нет | Целое число | Число возвращаемых логических зон. Значение по умолчанию — 1. Число должно быть целым положительным числом от 1 до 3.  Используйте 1 для ресурсов с одной зоной. Для ресурсов с несколькими зонами значение должно быть меньше или равно числу поддерживаемых зон. |
| offset | Нет | Целое число | Смещение от начальной логической зоны. Функция возвращает ошибку, если смещение плюс Нумберофзонес превышает число поддерживаемых зон. |

### <a name="return-value"></a>Возвращаемое значение

Массив с поддерживаемыми зонами. При использовании значений по умолчанию для offset и Нумберофзонес тип ресурса и регион, поддерживающий зоны, возвращают следующий массив:

```json
[
    "1"
]
```

Если `numberOfZones` параметр имеет значение 3, он возвращает:

```json
[
    "1",
    "2",
    "3"
]
```

Если тип или регион ресурса не поддерживает зоны, возвращается пустой массив.

```json
[
]
```

### <a name="pickzones-example"></a>Пример Пиккзонес

В следующем шаблоне показаны три результата использования функции Пиккзонес.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "functions": [],
  "variables": {},
  "resources": [],
  "outputs": {
    "supported": {
      "type": "array",
      "value": "[pickZones('Microsoft.Compute', 'virtualMachines', 'westus2')]"
    },
    "notSupportedRegion": {
      "type": "array",
      "value": "[pickZones('Microsoft.Compute', 'virtualMachines', 'northcentralus')]"
    },
    "notSupportedType": {
      "type": "array",
      "value": "[pickZones('Microsoft.Cdn', 'profiles', 'westus2')]"
    }
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
output supported array = pickZones('Microsoft.Compute', 'virtualMachines', 'westus2')
output notSupportedRegion array = pickZones('Microsoft.Compute', 'virtualMachines', 'northcentralus')
output notSupportedType array = pickZones('Microsoft.Cdn', 'profiles', 'westus2')
```

---

Выходные данные из предыдущих примеров возвращают три массива.

| Имя | Type | Значение |
| ---- | ---- | ----- |
| Поддерживается | array | ["1"] |
| нотсуппортедрегион | array | [] |
| нотсуппортедтипе | array | [] |

Вы можете использовать ответ от Пиккзонес, чтобы определить, следует ли предоставить значение NULL для зон или назначить виртуальные машины разным зонам. В следующем примере задается значение для зоны на основе доступности зон.

# <a name="json"></a>[JSON](#tab/json)

```json
"zones": {
  "value": "[if(not(empty(pickZones('Microsoft.Compute', 'virtualMachines', 'westus2'))), string(add(mod(copyIndex(),3),1)), json('null'))]"
},
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

> [!NOTE]
> Циклы и copyIndex () еще не реализованы.  См. раздел [Loops](https://github.com/Azure/bicep/blob/main/docs/spec/loops.md).

---

## <a name="providers"></a>providers

`providers(providerNamespace, [resourceType])`

Возвращает сведения о поставщике ресурсов и поддерживаемых типах ресурсов. Если тип ресурса не указан, функция возвращает все типы, поддерживаемые для поставщика ресурсов.

### <a name="parameters"></a>Параметры

| Параметр | Обязательно | Тип | Описание |
|:--- |:--- |:--- |:--- |
| пространство_имен_поставщика |Да |строка |Пространство имен поставщика. |
| тип_ресурса |Нет |строка |Тип ресурса в указанном пространстве имен. |

### <a name="return-value"></a>Возвращаемое значение

Все поддерживаемые типы возвращаются в следующем формате:

```json
{
  "resourceType": "{name of resource type}",
  "locations": [ all supported locations ],
  "apiVersions": [ all supported API versions ]
}
```

Упорядочение массива возвращаемых значений не гарантируется.

### <a name="providers-example"></a>Пример функции provider

В следующем [примере шаблона](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/providers.json) показано, как использовать функцию provider:

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "providerNamespace": {
      "type": "string"
    },
    "resourceType": {
      "type": "string"
    }
  },
  "resources": [],
  "outputs": {
    "providerOutput": {
      "type": "object",
      "value": "[providers(parameters('providerNamespace'), parameters('resourceType'))]"
    }
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param providerNamespace string
param resourceType string

output providerOutput array = providers(providerNamespace, resourceType)
```

---

Для поставщика ресурсов **Microsoft.Web** и типа ресурса **sites** приведенный выше пример возвращает объект в таком формате:

```json
{
  "resourceType": "sites",
  "locations": [
    "South Central US",
    "North Europe",
    "West Europe",
    "Southeast Asia",
    ...
  ],
  "apiVersions": [
    "2016-08-01",
    "2016-03-01",
    "2015-08-01-preview",
    "2015-08-01",
    ...
  ]
}
```

## <a name="reference"></a>reference

`reference(resourceName or resourceIdentifier, [apiVersion], ['Full'])`

Возвращает объект, представляющий состояние среды выполнения ресурса.

### <a name="parameters"></a>Параметры

| Параметр | Обязательно | Тип | Описание |
|:--- |:--- |:--- |:--- |
| имя_ресурса или идентификатор_ресурса |Да |строка |Имя или уникальный идентификатор ресурса. При указании ссылки на ресурс в текущем шаблоне укажите в качестве параметра только имя ресурса. При указании ссылки на ранее развернутый ресурс или если имя ресурса неоднозначно, укажите идентификатор ресурса. |
| версия_API |Нет |строка |Версия API для указанного ресурса. **Этот параметр является обязательным, если ресурс не подготовлен в рамках того же шаблона.** Как правило, указывается в формате **гггг-мм-дд**. Действительные версии API своего ресурса можно посмотреть в [справочнике по шаблонам](/azure/templates/). |
| Full |Нет |строка |Значение, указывающее, следует ли возвращать полный объект ресурса. Если вы не укажете `'Full'`, возвращается только объект свойств ресурса. Полный объект включает такие значения, как идентификатор ресурса и расположение. |

### <a name="return-value"></a>Возвращаемое значение

Каждый тип ресурса возвращает для функции reference разные свойства. Функция не возвращает данные в едином предварительно заданном формате. Кроме того, возвращаемое значение отличается в зависимости от значения аргумента `'Full'`. Чтобы просмотреть свойства для типа ресурса, возвратите объект в разделе outputs, как показано в примере.

### <a name="remarks"></a>Remarks

Эталонная функция извлекает состояние среды выполнения ранее развернутого ресурса или ресурса, развернутого в текущем шаблоне. В этой статье приведены примеры для обоих сценариев.

Обычно функция **reference** используется, чтобы получить определенное значение из объекта, например универсальный код ресурса (URI) конечной точки большого двоичного объекта или полное доменное имя.

# <a name="json"></a>[JSON](#tab/json)

```json
"outputs": {
  "BlobUri": {
    "type": "string",
    "value": "[reference(resourceId('Microsoft.Storage/storageAccounts', parameters('storageAccountName'))).primaryEndpoints.blob]"
  },
  "FQDN": {
    "type": "string",
    "value": "[reference(resourceId('Microsoft.Network/publicIPAddresses', parameters('ipAddressName'))).dnsSettings.fqdn]"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
output BlobUri string = reference(resourceId('Microsoft.Storage/storageAccounts', storageAccountName)).primaryEndpoints.blob
output FQDN string = reference(resourceId('Microsoft.Network/publicIPAddresses', ipAddressName)).dnsSettings.fqdn
```

---

Используйте `'Full'`, если вам нужны значения ресурсов, которые не являются частью схемы свойств. Например, чтобы задать политику доступа к хранилищу ключей, получите свойства идентификатора для виртуальной машины.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "type": "Microsoft.KeyVault/vaults",
  "apiVersion": "2019-09-01",
  "name": "vaultName",
  "properties": {
    "tenantId": "[subscription().tenantId]",
    "accessPolicies": [
      {
        "tenantId": "[reference(resourceId('Microsoft.Compute/virtualMachines', variables('vmName')), '2019-03-01', 'Full').identity.tenantId]",
        "objectId": "[reference(resourceId('Microsoft.Compute/virtualMachines', variables('vmName')), '2019-03-01', 'Full').identity.principalId]",
        "permissions": {
          "keys": [
            "all"
          ],
          "secrets": [
            "all"
          ]
        }
      }
    ],
    ...
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
resource myVault 'Microsoft.KeyVault/vaults@2019-09-01' = {
  name: 'vaultName'
  properties: {
    tenantId: subscription().tenantId
    accessPolicies: [
      {
        'tenantId': reference(resourceId('Microsoft.Compute/virtualMachines', vmName), '2019-03-01', 'Full').identity.tenantId
        'objectId': reference(resourceId('Microsoft.Compute/virtualMachines', vmName), '2019-03-01', 'Full').identity.principalId
        'permissions': {
          'keys': [
            'all'
          ]
          'secrets': [
            'all'
          ]
        }
      }
    ]
  }
}
```

---

### <a name="valid-uses"></a>Допустимые варианты использования

Эталонную функцию можно использовать только в свойствах определения ресурса и в разделе выходных данных шаблона или развертывания. При использовании с [итерацией свойства](copy-properties.md) можно использовать функцию reference для `input`, так как выражение назначается свойству ресурса.

Нельзя использовать функцию reference для задания значения свойства `count` в цикле копирования. Можно использовать для задания других свойств в цикле. Функция reference заблокирована для свойства count, так как это свойство должно быть определено до разрешения функции reference.

Чтобы использовать функцию Reference или любую функцию List * в разделе Outputs вложенного шаблона, необходимо задать для параметра значение,  ```expressionEvaluationOptions``` чтобы использовать [внутреннюю оценку области](linked-templates.md#expression-evaluation-scope-in-nested-templates) , или использовать ссылку вместо вложенного шаблона.

При использовании функции **reference** в ресурсе, который развернут условно, функция вычисляется, даже если ресурс не развернут.  Если функция **reference** ссылается на несуществующий ресурс, возникает ошибка. Используйте функцию **if**, чтобы убедиться, что функция вычисляется только при развернутом ресурсе. Пример шаблона, который использует функции if и reference с условно развернутым ресурсом, см. в описании [функции if](template-functions-logical.md#if).

### <a name="implicit-dependency"></a>Неявная зависимость

С помощью функции reference вы прямо объявляете, что один ресурс зависит от другого, если ресурс, на который указывает ссылка, предоставляется в том же шаблоне и вы ссылаетесь, используя его имя (а не идентификатор). При этом свойство dependsOn использовать не нужно. Расчет функции выполняется только после развертывания ресурса, на который указывает ссылка.

### <a name="resource-name-or-identifier"></a>Имя или идентификатор ресурса

При ссылке на ресурс, развернутый в том же шаблоне, укажите имя ресурса.

# <a name="json"></a>[JSON](#tab/json)

```json
"value": "[reference(parameters('storageAccountName'))]"
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
value: reference(storageAccountName)
```

---

При ссылке на ресурс, который не развернут в том же шаблоне, укажите идентификатор ресурса и `apiVersion`.

# <a name="json"></a>[JSON](#tab/json)

```json
"value": "[reference(resourceId(parameters('storageResourceGroup'), 'Microsoft.Storage/storageAccounts', parameters('storageAccountName')), '2018-07-01')]"
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
value: reference(resourceId(storageResourceGroup, 'Microsoft.Storage/storageAccounts', storageAccountName), '2018-07-01')]"
```

---

Чтобы избежать неоднозначности в отношении ресурса, на который вы ссылаетесь, можно указать полный идентификатор ресурса.

# <a name="json"></a>[JSON](#tab/json)

```json
"value": "[reference(resourceId('Microsoft.Network/publicIPAddresses', parameters('ipAddressName')))]"
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
value: reference(resourceId('Microsoft.Network/publicIPAddresses', ipAddressName))
```

---

При создании полной ссылки на ресурс порядок объединения сегментов из типа и имени представляет собой не только использование этих двух вариантов. Вместо этого после пространства имен используйте пары *типа и имени*, начиная от наименее подходящей к наиболее подходящей.

**{пространство_имен_поставщика_ресурсов}/{тип_родительского_ресурса}/{имя_родительского_ресурса}[/{тип_дочернего_ресурса}/{имя_дочернего_ресурса}]**

Пример:

`Microsoft.Compute/virtualMachines/myVM/extensions/myExt` — правильно, `Microsoft.Compute/virtualMachines/extensions/myVM/myExt` — неправильно.

Чтобы упростить создание любого идентификатора ресурса, вместо функции `concat()` используйте функции `resourceId()`, описанные в этом документе.

### <a name="get-managed-identity"></a>Получение управляемого удостоверения

[Управляемые удостоверения для ресурсов Azure](../../active-directory/managed-identities-azure-resources/overview.md) являются [типами ресурсов расширения](../management/extension-resource-types.md), которые создаются неявно для некоторых ресурсов. Так как управляемое удостоверение не определено явным образом в шаблоне, необходимо указать ссылку на ресурс, к которому применяется удостоверение. Используйте `Full`, чтобы получить все свойства, включая явно созданное удостоверение.

Шаблон:

`"[reference(resourceId(<resource-provider-namespace>, <resource-name>, <API-version>, 'Full').Identity.propertyName]"`

Например, чтобы получить идентификатор субъекта для управляемого удостоверения, применяемого к виртуальной машине, используйте:

# <a name="json"></a>[JSON](#tab/json)

```json
"[reference(resourceId('Microsoft.Compute/virtualMachines', variables('vmName')),'2019-12-01', 'Full').identity.principalId]",
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
reference(resourceId('Microsoft.Compute/virtualMachines', vmName),'2019-12-01', 'Full').identity.principalId
```

---

Чтобы получить идентификатор клиента для управляемого удостоверения, применяемого к масштабируемому набору виртуальных машин, используйте:

# <a name="json"></a>[JSON](#tab/json)

```json
"[reference(resourceId('Microsoft.Compute/virtualMachineScaleSets',  variables('vmNodeType0Name')), 2019-12-01, 'Full').Identity.tenantId]"
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
reference(resourceId('Microsoft.Compute/virtualMachineScaleSets',  vmNodeType0Name), 2019-12-01, 'Full').Identity.tenantId
```

---

### <a name="reference-example"></a>Пример ссылки

Следующий [пример шаблона](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/referencewithstorage.json) развертывает ресурс и ссылается на этот ресурс.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageAccountName": {
      "type": "string"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2016-12-01",
      "name": "[parameters('storageAccountName')]",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage",
      "tags": {},
      "properties": {
      }
    }
  ],
  "outputs": {
      "referenceOutput": {
        "type": "object",
        "value": "[reference(parameters('storageAccountName'))]"
      },
      "fullReferenceOutput": {
        "type": "object",
        "value": "[reference(parameters('storageAccountName'), '2016-12-01', 'Full')]"
      }
    }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param storageAccountName string

resource myStorage 'Microsoft.Storage/storageAccounts@2016-12-01' = {
  name: storageAccountName
  location: resourceGroup().location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'Storage'
  tags: {}
  properties: {}
}

output referenceOutput object = reference(storageAccountName)
output fullReferenceOutput object = reference(storageAccountName, '2016-12-01', 'Full')
```

---

В предыдущем примере возвращаются два объекта. Объект свойств имеет следующий формат:

```json
{
   "creationTime": "2017-10-09T18:55:40.5863736Z",
   "primaryEndpoints": {
     "blob": "https://examplestorage.blob.core.windows.net/",
     "file": "https://examplestorage.file.core.windows.net/",
     "queue": "https://examplestorage.queue.core.windows.net/",
     "table": "https://examplestorage.table.core.windows.net/"
   },
   "primaryLocation": "southcentralus",
   "provisioningState": "Succeeded",
   "statusOfPrimary": "available",
   "supportsHttpsTrafficOnly": false
}
```

Полный объект имеет следующий формат:

```json
{
  "apiVersion":"2016-12-01",
  "location":"southcentralus",
  "sku": {
    "name":"Standard_LRS",
    "tier":"Standard"
  },
  "tags":{},
  "kind":"Storage",
  "properties": {
    "creationTime":"2017-10-09T18:55:40.5863736Z",
    "primaryEndpoints": {
      "blob":"https://examplestorage.blob.core.windows.net/",
      "file":"https://examplestorage.file.core.windows.net/",
      "queue":"https://examplestorage.queue.core.windows.net/",
      "table":"https://examplestorage.table.core.windows.net/"
    },
    "primaryLocation":"southcentralus",
    "provisioningState":"Succeeded",
    "statusOfPrimary":"available",
    "supportsHttpsTrafficOnly":false
  },
  "subscriptionId":"<subscription-id>",
  "resourceGroupName":"functionexamplegroup",
  "resourceId":"Microsoft.Storage/storageAccounts/examplestorage",
  "referenceApiVersion":"2016-12-01",
  "condition":true,
  "isConditionTrue":true,
  "isTemplateResource":false,
  "isAction":false,
  "provisioningOperation":"Read"
}
```

Следующий [пример шаблона](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/reference.json) ссылается на учетную запись хранения, которая не развертывается в этом шаблоне. Учетная запись хранения уже имеется в той же подписке.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageResourceGroup": {
      "type": "string"
    },
    "storageAccountName": {
      "type": "string"
    }
  },
  "resources": [],
  "outputs": {
    "ExistingStorage": {
      "type": "object",
      "value": "[reference(resourceId(parameters('storageResourceGroup'), 'Microsoft.Storage/storageAccounts', parameters('storageAccountName')), '2018-07-01')]"
    }
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param storageResourceGroup string
param storageAccountName string

output ExistingStorage object = reference(resourceId(storageAccountName), 'Microsoft.Storage/storageAccounts', storageAccountName), '2018-07-01')
```

---

## <a name="resourcegroup"></a>resourceGroup

`resourceGroup()`

Возвращает объект, который представляет текущую группу ресурсов.

### <a name="return-value"></a>Возвращаемое значение

Возвращаемый объект имеет следующий формат:

```json
{
  "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}",
  "name": "{resourceGroupName}",
  "type":"Microsoft.Resources/resourceGroups",
  "location": "{resourceGroupLocation}",
  "managedBy": "{identifier-of-managing-resource}",
  "tags": {
  },
  "properties": {
    "provisioningState": "{status}"
  }
}
```

Свойство **managedBy** возвращается только для групп ресурсов, которые содержат ресурсы, управляемые другой службой. Для управляемых приложений, Databricks и AKS значением этого свойства является идентификатор управляющего ресурса.

### <a name="remarks"></a>Remarks

Функцию `resourceGroup()` нельзя использовать в шаблоне, который [развернут на уровне подписки](deploy-to-subscription.md). Ее можно использовать только в шаблонах, развернутых в группе ресурсов. Функцию `resourceGroup()` можно использовать в [связанном или вложенном шаблоне (с внутренней областью)](linked-templates.md), предназначенном для группы ресурсов, даже если родительский шаблон развертывается в подписке. В этом сценарии связанный или вложенный шаблон развертывается на уровне группы ресурсов. Дополнительные сведения о нацеливании на группу ресурсов в развертывании уровня подписки см. в разделе [Развертывание ресурсов Azure в нескольких подписках или группах ресурсов](./deploy-to-resource-group.md).

Как правило, функция resourceGroup используется для создания ресурсов в одном расположении с группой ресурсов. В следующем примере расположение группы ресурсов используется в качестве значения параметра по умолчанию.

# <a name="json"></a>[JSON](#tab/json)

```json
"parameters": {
  "location": {
    "type": "string",
    "defaultValue": "[resourceGroup().location]"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param location string = resourceGroup().location
```

---

Кроме того, функцию resourceGroup можно использовать для применения к ресурсу тегов из группы ресурсов. Дополнительные сведения см. в разделе [Применение тегов из группы ресурсов](../management/tag-resources.md#apply-tags-from-resource-group).

При использовании вложенных шаблонов для развертывания в нескольких группах ресурсов можно указать область для вычисления функции resourceGroup. Дополнительные сведения см. в разделе [Развертывание ресурсов Azure в нескольких подписках или группах ресурсов](./deploy-to-resource-group.md).

### <a name="resource-group-example"></a>Пример группы ресурсов

Следующий [пример шаблона](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/resourcegroup.json) возвращает свойства группы ресурсов.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [],
  "outputs": {
    "resourceGroupOutput": {
      "type" : "object",
      "value": "[resourceGroup()]"
    }
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
output resourceGroupOutput object = resourceGroup()
```

---

В предыдущем примере возвращается объект в следующем формате:

```json
{
  "id": "/subscriptions/{subscription-id}/resourceGroups/examplegroup",
  "name": "examplegroup",
  "type":"Microsoft.Resources/resourceGroups",
  "location": "southcentralus",
  "properties": {
    "provisioningState": "Succeeded"
  }
}
```

## <a name="resourceid"></a>resourceId

`resourceId([subscriptionId], [resourceGroupName], resourceType, resourceName1, [resourceName2], ...)`

Возвращает уникальный идентификатор ресурса. Используйте эту функцию в том случае, когда имя ресурса является неоднозначным или не было предоставлено в пределах того же шаблона. Формат возвращаемого идентификатора зависит от того, происходит ли развертывание в области действия группы ресурсов, подписки, группы управления или клиента.

### <a name="parameters"></a>Параметры

| Параметр | Обязательно | Тип | Описание |
|:--- |:--- |:--- |:--- |
| subscriptionId |Нет |строка (в формате GUID) |Значение по умолчанию — текущая подписка. Укажите это значение, если нужно получить ресурс из другой подписки. Это значение предоставляется только при развертывании в области действия группы ресурсов или подписки. |
| имя_группы_ресурсов |Нет |строка |Значение по умолчанию — текущая группа ресурсов. Укажите это значение, если нужно получить ресурс из другой группы ресурсов. Это значение предоставляется только при развертывании в области действия группы ресурсов. |
| тип_ресурса |Да |строка |Тип ресурса, включая пространство имен поставщика ресурсов. |
| имя_ресурса1 |Да |строка |Имя ресурса. |
| имя_ресурса2 |Нет |строка |Следующий сегмент имени ресурса, если он необходим. |

Продолжайте добавлять имена ресурсов в качестве параметров, если тип ресурса включает больше сегментов.

### <a name="return-value"></a>Возвращаемое значение

При развертывании шаблона в области группы ресурсов идентификатор ресурса возвращается в следующем формате:

```json
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/{resourceProviderNamespace}/{resourceType}/{resourceName}
```

Для других областей развертывания можно использовать функцию resourceId, но формат идентификатора изменится.

Если при развертывании в подписке используется resourceId, идентификатор ресурса возвращается в следующем формате:

```json
/subscriptions/{subscriptionId}/providers/{resourceProviderNamespace}/{resourceType}/{resourceName}
```

Если вы используете resourceId при развертывании в группе управления или клиенте, идентификатор ресурса возвращается в следующем формате:

```json
/providers/{resourceProviderNamespace}/{resourceType}/{resourceName}
```

Во избежание путаницы рекомендуется не использовать resourceId при работе с ресурсами, развернутыми в подписке, группе управления или клиенте. Вместо этого используйте функцию ID, предназначенную для области.

Для [ресурсов уровня подписки](deploy-to-subscription.md)используйте функцию [субскриптионресаурцеид](#subscriptionresourceid) .

Для [ресурсов уровня группы управления](deploy-to-management-group.md)используйте функцию [екстенсионресаурцеид](#extensionresourceid) для ссылки на ресурс, реализованный как расширение группы управления. Например, определения настраиваемой политики, развернутые в группе управления, являются расширениями группы управления. Используйте функцию [тенантресаурцеид](#tenantresourceid) для ссылки на ресурсы, развернутые в клиенте, но доступные в группе управления. Например, встроенные определения политик реализуются как ресурсы уровня клиента.

Для [ресурсов на уровне клиента](deploy-to-tenant.md)используйте функцию [тенантресаурцеид](#tenantresourceid) . Используйте Тенантресаурцеид для встроенных определений политик, так как они реализуются на уровне клиента.

### <a name="remarks"></a>Remarks

Количество предоставляемых параметров зависит от того, является ли ресурс родительским или дочерним, а также находится ли ресурс в той же подписке или группе ресурсов.

Чтобы получить идентификатор ресурса для родительского ресурса в той же подписке и группе ресурсов, укажите тип и имя ресурса.

# <a name="json"></a>[JSON](#tab/json)

```json
"[resourceId('Microsoft.ServiceBus/namespaces', 'namespace1')]"
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
resourceId('Microsoft.ServiceBus/namespaces', 'namespace1')
```

---

При получении идентификатора ресурса для дочернего ресурса обратите внимание на количество сегментов в типе ресурса. Укажите имя ресурса для каждого сегмента типа ресурса. Имя сегмента соответствует ресурсу, который существует для этой части иерархии.

# <a name="json"></a>[JSON](#tab/json)

```json
"[resourceId('Microsoft.ServiceBus/namespaces/queues/authorizationRules', 'namespace1', 'queue1', 'auth1')]"
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
resourceId('Microsoft.ServiceBus/namespaces/queues/authorizationRules', 'namespace1', 'queue1', 'auth1')
```

---

Чтобы получить идентификатор ресурса в той же подписке, но в другой группе ресурсов, укажите имя группы ресурсов.

# <a name="json"></a>[JSON](#tab/json)

```json
"[resourceId('otherResourceGroup', 'Microsoft.Storage/storageAccounts', 'examplestorage')]"
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
resourceId('otherResourceGroup', 'Microsoft.Storage/storageAccounts', 'examplestorage')
```

---

Чтобы получить идентификатор ресурса в другой подписке и группе ресурсов, укажите идентификатор подписки и имя группы ресурсов.

# <a name="json"></a>[JSON](#tab/json)

```json
"[resourceId('xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx', 'otherResourceGroup', 'Microsoft.Storage/storageAccounts','examplestorage')]"
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
resourceId('xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx', 'otherResourceGroup', 'Microsoft.Storage/storageAccounts','examplestorage')
```

---

Эта функция часто необходима при использовании учетной записи хранения или виртуальной сети в альтернативной группе ресурсов. В следующем примере показано, как ресурс из внешней группы ресурсов можно легко использовать:

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "type": "string"
    },
    "virtualNetworkName": {
      "type": "string"
    },
    "virtualNetworkResourceGroup": {
      "type": "string"
    },
    "subnet1Name": {
      "type": "string"
    },
    "nicName": {
      "type": "string"
    }
  },
  "variables": {
    "subnet1Ref": "[resourceId(parameters('virtualNetworkResourceGroup'), 'Microsoft.Network/virtualNetworks/subnets', parameters('virtualNetworkName'), parameters('subnet1Name'))]"
  },
  "resources": [
    {
      "type": "Microsoft.Network/networkInterfaces",
      "apiVersion": "2015-05-01-preview",
      "name": "[parameters('nicName')]",
      "location": "[parameters('location')]",
      "properties": {
        "ipConfigurations": [
          {
            "name": "ipconfig1",
            "properties": {
              "privateIPAllocationMethod": "Dynamic",
              "subnet": {
                "id": "[variables('subnet1Ref')]"
              }
            }
          }
        ]
      }
    }
  ]
}
```
# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param location string
param virtualNetworkName string
param virtualNetworkResourceGroup string
param subnet1Name string
param nicName string

var subnet1Ref = resourceId('virtualNetworkResourceGroup', 'Microsoft.Network/virtualNetworks/subnets', 'virtualNetworkName', 'subnet1Name')

resource myInterface 'Microsoft.Network/networkInterfaces@2015-05-01-preview' = {
  name: nicName
  location: location
  properties: {
    ipConfigurations: [
      {
        name: 'ipconfig1'
        properties: {
          privateIPAllocationMethod: 'Dynamic'
          subnet: {
            id: subnet1Ref
          }
        }
      }
    ]
  }
}
```

---


### <a name="resource-id-example"></a>Пример идентификатора ресурса

Следующий [пример шаблона](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/resourceid.json) возвращает идентификатор ресурса для учетной записи хранения в группе ресурсов:

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [],
  "outputs": {
    "sameRGOutput": {
      "type": "string",
      "value": "[resourceId('Microsoft.Storage/storageAccounts','examplestorage')]"
    },
    "differentRGOutput": {
      "type": "string",
      "value": "[resourceId('otherResourceGroup', 'Microsoft.Storage/storageAccounts','examplestorage')]"
    },
    "differentSubOutput": {
      "type": "string",
      "value": "[resourceId('11111111-1111-1111-1111-111111111111', 'otherResourceGroup', 'Microsoft.Storage/storageAccounts','examplestorage')]"
    },
    "nestedResourceOutput": {
      "type": "string",
      "value": "[resourceId('Microsoft.SQL/servers/databases', 'serverName', 'databaseName')]"
    }
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
output sameRGOutput string = resourceId('Microsoft.Storage/storageAccounts','examplestorage')
output differentRGOutput string = resourceId('otherResourceGroup', 'Microsoft.Storage/storageAccounts','examplestorage')
output differentSubOutput string = resourceId('11111111-1111-1111-1111-111111111111', 'otherResourceGroup', 'Microsoft.Storage/storageAccounts','examplestorage')
output nestedResourceOutput string = resourceId('Microsoft.SQL/servers/databases', 'serverName', 'databaseName')
```

---

Выходные данные из предыдущего примера со значениями по умолчанию:

| Имя | Type | Значение |
| ---- | ---- | ----- |
| sameRGOutput | Строка | /subscriptions/{ИД_текущей_подписки}/resourceGroups/examplegroup/providers/Microsoft.Storage/storageAccounts/examplestorage |
| differentRGOutput | Строка | /subscriptions/{ИД_текущей_подписки}/resourceGroups/otherResourceGroup/providers/Microsoft.Storage/storageAccounts/examplestorage |
| differentSubOutput | Строка | /subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/otherResourceGroup/providers/Microsoft.Storage/storageAccounts/examplestorage |
| nestedResourceOutput | Строка | /subscriptions/{ИД_текущей_подписки}/resourceGroups/examplegroup/providers/Microsoft.SQL/servers/serverName/databases/databaseName |

## <a name="subscription"></a>Подписка

`subscription()`

Возвращает сведения о подписке для текущего развертывания.

### <a name="return-value"></a>Возвращаемое значение

Функция возвращает значение в следующем формате:

```json
{
  "id": "/subscriptions/{subscription-id}",
  "subscriptionId": "{subscription-id}",
  "tenantId": "{tenant-id}",
  "displayName": "{name-of-subscription}"
}
```

### <a name="remarks"></a>Remarks

При использовании вложенных шаблонов для развертывания в нескольких подписках можно указать область для вычисления функции subscription. Дополнительные сведения см. в разделе [Развертывание ресурсов Azure в нескольких подписках или группах ресурсов](./deploy-to-resource-group.md).

### <a name="subscription-example"></a>Пример подписки

В следующем [примере шаблона](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/subscription.json) показана функция subscription, вызываемая в разделе выходных данных.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [],
  "outputs": {
    "subscriptionOutput": {
      "value": "[subscription()]",
      "type" : "object"
    }
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
output subscriptionOutput object = subscription()
```

---

## <a name="subscriptionresourceid"></a>subscriptionResourceId

`subscriptionResourceId([subscriptionId], resourceType, resourceName1, [resourceName2], ...)`

Возвращает уникальный идентификатор ресурса, развернутого на уровне подписки.

### <a name="parameters"></a>Параметры

| Параметр | Обязательно | Тип | Описание |
|:--- |:--- |:--- |:--- |
| subscriptionId |Нет |строка (в формате GUID) |Значение по умолчанию — текущая подписка. Укажите это значение, если нужно получить ресурс из другой подписки. |
| тип_ресурса |Да |строка |Тип ресурса, включая пространство имен поставщика ресурсов. |
| имя_ресурса1 |Да |строка |Имя ресурса. |
| имя_ресурса2 |Нет |строка |Следующий сегмент имени ресурса, если он необходим. |

Продолжайте добавлять имена ресурсов в качестве параметров, если тип ресурса включает больше сегментов.

### <a name="return-value"></a>Возвращаемое значение

Идентификатор возвращается в следующем формате:

```json
/subscriptions/{subscriptionId}/providers/{resourceProviderNamespace}/{resourceType}/{resourceName}
```

### <a name="remarks"></a>Remarks

Используйте эту функцию, чтобы получить идентификатор ресурса для ресурсов, [развернутых на уровне подписки](deploy-to-subscription.md), а не на уровне группы ресурсов. Возвращаемый идентификатор отличается от значения, возвращаемого функцией [resourceId](#resourceid), тем, что он не включает значение группы ресурсов.

### <a name="subscriptionresourceid-example"></a>Пример subscriptionResourceID

Следующий шаблон назначает встроенную роль. Его можно развернуть либо в группе ресурсов, либо в подписке. Для получения идентификатора ресурса для встроенных ролей в нем используется функция subscriptionResourceId.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "principalId": {
      "type": "string",
      "metadata": {
        "description": "The principal to assign the role to"
      }
    },
    "builtInRoleType": {
      "type": "string",
      "allowedValues": [
        "Owner",
        "Contributor",
        "Reader"
      ],
      "metadata": {
        "description": "Built-in role to assign"
      }
    },
    "roleNameGuid": {
      "type": "string",
      "defaultValue": "[newGuid()]",
      "metadata": {
        "description": "A new GUID used to identify the role assignment"
      }
    }
  },
  "variables": {
    "Owner": "[subscriptionResourceId('Microsoft.Authorization/roleDefinitions', '8e3af657-a8ff-443c-a75c-2fe8c4bcb635')]",
    "Contributor": "[subscriptionResourceId('Microsoft.Authorization/roleDefinitions', 'b24988ac-6180-42a0-ab88-20f7382dd24c')]",
    "Reader": "[subscriptionResourceId('Microsoft.Authorization/roleDefinitions', 'acdd72a7-3385-48ef-bd42-f606fba81ae7')]"
  },
  "resources": [
    {
      "type": "Microsoft.Authorization/roleAssignments",
      "apiVersion": "2018-09-01-preview",
      "name": "[parameters('roleNameGuid')]",
      "properties": {
        "roleDefinitionId": "[variables(parameters('builtInRoleType'))]",
        "principalId": "[parameters('principalId')]"
      }
    }
  ]
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param principalId string {
  metadata: {
    'description': 'principalId'
  }
}
param builtInRoleType string {
  'allowed': [
    'Owner'
    'Contributor'
    'Reader'
  ]
  'metadata': {
      'description': 'Built-in role to assign'
  }
}
param roleNameGuid string {
  default: newGuid()
  metadata: {
    'description': 'A new GUID used to identify the role assignment'
  }
}

var roleDefinitionId = {
  Owner: {
    id: subscriptionResourceId('Microsoft.Authorization/roleDefinitions', '8e3af657-a8ff-443c-a75c-2fe8c4bcb635')
  }
  Contributor: {
    id: subscriptionResourceId('Microsoft.Authorization/roleDefinitions', 'b24988ac-6180-42a0-ab88-20f7382dd24c')
  }
  Reader: {
    id: subscriptionResourceId('Microsoft.Authorization/roleDefinitions', 'acdd72a7-3385-48ef-bd42-f606fba81ae7')
  }
}

resource myRoleAssignment 'Microsoft.Authorization/roleAssignments@2018-09-01-preview' = {
  name: roleNameGuid
  properties: {
    roleDefinitionId: roleDefinitionId[builtInRoleType].id
    principalId: principalId
  }
}
```

---

## <a name="tenantresourceid"></a>tenantResourceId

`tenantResourceId(resourceType, resourceName1, [resourceName2], ...)`

Возвращает уникальный идентификатор ресурса, развернутого на уровне клиента.

### <a name="parameters"></a>Параметры

| Параметр | Обязательно | Тип | Описание |
|:--- |:--- |:--- |:--- |
| тип_ресурса |Да |строка |Тип ресурса, включая пространство имен поставщика ресурсов. |
| имя_ресурса1 |Да |строка |Имя ресурса. |
| имя_ресурса2 |Нет |строка |Следующий сегмент имени ресурса, если он необходим. |

Продолжайте добавлять имена ресурсов в качестве параметров, если тип ресурса включает больше сегментов.

### <a name="return-value"></a>Возвращаемое значение

Идентификатор возвращается в следующем формате:

```json
/providers/{resourceProviderNamespace}/{resourceType}/{resourceName}
```

### <a name="remarks"></a>Remarks

Эта функция используется для получения идентификатора ресурса, развернутого в клиенте. Возвращенный идентификатор отличается от значений, возвращаемых другими функциями получения идентификаторов, тем, что он не включает значения группы ресурсов и подписки.

### <a name="tenantresourceid-example"></a>Пример Тенантресаурцеид

Встроенные определения политик — это ресурсы уровня клиента. Чтобы развернуть назначение политики, ссылающееся на определение встроенной политики, используйте функцию Тенантресаурцеид.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "policyDefinitionID": {
      "type": "string",
      "defaultValue": "0a914e76-4921-4c19-b460-a2d36003525a",
      "metadata": {
        "description": "Specifies the ID of the policy definition or policy set definition being assigned."
      }
    },
    "policyAssignmentName": {
      "type": "string",
      "defaultValue": "[guid(parameters('policyDefinitionID'), resourceGroup().name)]",
      "metadata": {
        "description": "Specifies the name of the policy assignment, can be used defined or an idempotent name as the defaultValue provides."
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.Authorization/policyAssignments",
      "name": "[parameters('policyAssignmentName')]",
      "apiVersion": "2019-09-01",
      "properties": {
        "scope": "[subscriptionResourceId('Microsoft.Resources/resourceGroups', resourceGroup().name)]",
        "policyDefinitionId": "[tenantResourceId('Microsoft.Authorization/policyDefinitions', parameters('policyDefinitionID'))]"
      }
    }
  ]
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param policyDefinitionID string{
  default: '0a914e76-4921-4c19-b460-a2d36003525a'
  metadata: {
    'description': 'Specifies the ID of the policy definition or policy set definition being assigned.'
  }
}

param policyAssignmentName string {
  default: guid(policyDefinitionID, resourceGroup().name)
  metadata: {
    'description': 'Specifies the name of the policy assignment, can be used defined or an idempotent name as the defaultValue provides.'
  }
}

resource myPolicyAssignment 'Microsoft.Authorization/policyAssignments@2019-09-01' = {
  name: policyAssignmentName
  properties: {
    scope: subscriptionResourceId('Microsoft.Resources/resourceGroups', resourceGroup().name)
    policyDefinitionId: tenantResourceId('Microsoft.Authorization/policyDefinitions', policyDefinitionID)
  }
}
```

---

## <a name="next-steps"></a>Дальнейшие действия

* Описание разделов в шаблоне ARM см. [в разделе Общие сведения о структуре и синтаксисе шаблонов ARM](template-syntax.md).
* Сведения о слиянии нескольких шаблонов см. в разделе [Использование связанных и вложенных шаблонов при развертывании ресурсов Azure](linked-templates.md).
* Сведения о итерации указанного числа раз при создании типа ресурса см. в разделе [Итерация ресурсов в шаблонах ARM](copy-resources.md).
* Чтобы узнать, как развернуть созданный вами шаблон, см. статью [развертывание ресурсов с помощью шаблонов ARM и Azure PowerShell](deploy-powershell.md).
