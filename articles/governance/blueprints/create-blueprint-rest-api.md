---
title: Краткое руководство. Создание схемы с помощью REST API
description: В рамках этого краткого руководства вы создадите, определите и развернете артефакты с помощью REST API.
ms.date: 01/27/2021
ms.topic: quickstart
ms.openlocfilehash: ceea54957e52f3b33d2c3fb2af190f15e6c94ec3
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105558981"
---
# <a name="quickstart-define-and-assign-an-azure-blueprint-with-rest-api"></a>Краткое руководство. Определение и назначение схемы Azure с помощью REST API

Зная, как создавать и назначать схемы, вы сможете определять типичные шаблоны для разработки многоразовых и быстро развертываемых конфигураций на основе шаблонов Azure Resource Manager (шаблонов ARM), политик, правил безопасности и т. п. В этом руководстве вы научитесь использовать службу Azure Blueprint для выполнения некоторых общих задач, связанных с созданием, публикацией и назначением схемы в вашей организации, таких как:

## <a name="prerequisites"></a>Предварительные требования

- Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free), прежде чем начинать работу.
- Зарегистрируйте поставщик ресурсов `Microsoft.Blueprint`. См. инструкции в статье [Поставщики и типы ресурсов Azure](../../azure-resource-manager/management/resource-providers-and-types.md).

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="getting-started-with-rest-api"></a>Начало работы с REST API

Если у вас нет опыта работы с интерфейсом REST API, сначала ознакомьтесь со [справочником по REST API Azure](/rest/api/azure/), чтобы получить общее представление о нем, в частности об URI запроса и тексте запроса. В данной статье эти понятия используются в указаниях по работе со схемами Azure, поэтому предполагается, что у вас есть соответствующие практические навыки. Начинающим рекомендуется применять такие средства, как [ARMClient](https://github.com/projectkudu/ARMClient), обеспечивающие автоматическую авторизацию.

Спецификации по Azure Blueprints см. в статье о [REST API для Azure Blueprints](/rest/api/blueprints/).

### <a name="rest-api-and-powershell"></a>REST API и PowerShell

Если у вас еще нет средства для выполнения вызовов REST API, рекомендуем использовать среду PowerShell. Ниже приведен пример заголовка для проверки подлинности в Azure. Создайте заголовок проверки подлинности, иногда называемый **токеном носителя**, и укажите универсальный код ресурса (URI) REST API, к которому нужно подключиться, а также параметры или **текст запроса**.

```azurepowershell-interactive
# Log in first with Connect-AzAccount if not using Cloud Shell

$azContext = Get-AzContext
$azProfile = [Microsoft.Azure.Commands.Common.Authentication.Abstractions.AzureRmProfileProvider]::Instance.Profile
$profileClient = New-Object -TypeName Microsoft.Azure.Commands.ResourceManager.Common.RMProfileClient -ArgumentList ($azProfile)
$token = $profileClient.AcquireAccessToken($azContext.Subscription.TenantId)
$authHeader = @{
    'Content-Type'='application/json'
    'Authorization'='Bearer ' + $token.AccessToken
}

# Invoke the REST API
$restUri = 'https://management.azure.com/subscriptions/{subscriptionId}?api-version=2020-01-01'
$response = Invoke-RestMethod -Uri $restUri -Method Get -Headers $authHeader
```

Замените `{subscriptionId}` в значении приведенной выше переменной **$restUri**, чтобы получить сведения о своей подписке. Переменная $response содержит результат выполнения командлета `Invoke-RestMethod`, который может быть проанализирован с помощью таких командлетов, как [ConvertFrom-Json](/powershell/module/microsoft.powershell.utility/convertfrom-json). Если конечная точка службы REST API требует **текста запроса**, укажите переменную в формате JSON в качестве значения параметра `-Body` командлета `Invoke-RestMethod`.

## <a name="create-a-blueprint"></a>Создание схемы

Первым этапом при определении стандартной модели соответствия требованиям является формирование схемы на основе доступных ресурсов. Мы создадим схему с именем MyBlueprint для настройки роли и назначения политик для подписки. Затем мы добавим группу ресурсов, шаблон ARM и назначение ролей группе ресурсов.

> [!NOTE]
> При использовании REST API сначала создается объект _схемы_. Для каждого _артефакта_ с параметрами, который необходимо добавить, необходимо заранее определить параметры в исходной _схеме_.

В каждом универсальном коде ресурса (URI) REST API есть переменные, которые необходимо заменить собственными значениями:

- `{YourMG}` — замените это значение идентификатором своей группы управления.
- `{subscriptionId}` — замените это значение идентификатором своей подписки.

> [!NOTE]
> Схемы также можно создавать на уровне подписки. Пример см. в статье [ о создании и обновлении схем](/rest/api/blueprints/blueprints/createorupdate#subscriptionblueprint).

1. Создайте исходный объект _схемы_. **Текст запроса** содержит свойства схемы, группы ресурсов, которые необходимо создать, и все параметры уровня схемы. Эти параметры задаются при назначении и используются артефактами, которые будут добавлены в дальнейшем.

   - Универсальный код ресурса (URI) REST API

     ```http
     PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint?api-version=2018-11-01-preview
     ```

   - Текст запроса

     ```json
     {
         "properties": {
             "description": "This blueprint sets tag policy and role assignment on the subscription, creates a ResourceGroup, and deploys a resource template and role assignment to that ResourceGroup.",
             "targetScope": "subscription",
             "parameters": {
                 "storageAccountType": {
                     "type": "string",
                     "metadata": {
                         "displayName": "storage account type.",
                         "description": null
                     }
                 },
                 "tagName": {
                     "type": "string",
                     "metadata": {
                         "displayName": "The name of the tag to provide the policy assignment.",
                         "description": null
                     }
                 },
                 "tagValue": {
                     "type": "string",
                     "metadata": {
                         "displayName": "The value of the tag to provide the policy assignment.",
                         "description": null
                     }
                 },
                 "contributors": {
                     "type": "array",
                     "metadata": {
                         "description": "List of AAD object IDs that is assigned Contributor role at the subscription"
                     }
                 },
                 "owners": {
                     "type": "array",
                     "metadata": {
                         "description": "List of AAD object IDs that is assigned Owner role at the resource group"
                     }
                 }
             },
             "resourceGroups": {
                 "storageRG": {
                     "description": "Contains the resource template deployment and a role assignment."
                 }
             }
         }
     }
     ```

1. Добавьте назначение ролей в подписку. В **тексте запроса** определяется _тип_ артефакта, свойства соответствуют идентификатору определения роли, а удостоверения субъектов передаются в виде массива значений. В приведенном ниже примере удостоверения субъектов, которым предоставляется указанная роль, передаются в параметре, задаваемом во время назначения схемы. В этом примере используется встроенная роль _Участник_ с глобальным уникальным идентификатором `b24988ac-6180-42a0-ab88-20f7382dd24c`.

   - Универсальный код ресурса (URI) REST API

     ```http
     PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/artifacts/roleContributor?api-version=2018-11-01-preview
     ```

   - Текст запроса

     ```json
     {
         "kind": "roleAssignment",
         "properties": {
             "roleDefinitionId": "/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c",
             "principalIds": "[parameters('contributors')]"
         }
     }
     ```

1. Добавьте назначение политики в подписку. В **тексте запроса** определяется _тип_ артефакта и свойства, соответствующие определению политики или инициативы, а для назначения политики настраивается использование определенных параметров схемы, настраиваемых во время назначения схемы. В этом примере используется встроенная политика "_Задать тег и его значение по умолчанию для групп ресурсов_" с глобальным уникальным идентификатором `49c88fc8-6fd1-46fd-a676-f12d1d3a4c71`.

   - Универсальный код ресурса (URI) REST API

     ```http
     PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/artifacts/policyTags?api-version=2018-11-01-preview
     ```

   - Текст запроса

     ```json
     {
         "kind": "policyAssignment",
         "properties": {
             "description": "Apply tag and its default value to resource groups",
             "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/49c88fc8-6fd1-46fd-a676-f12d1d3a4c71",
             "parameters": {
                 "tagName": {
                     "value": "[parameters('tagName')]"
                 },
                 "tagValue": {
                     "value": "[parameters('tagValue')]"
                 }
             }
         }
     }
     ```

1. Добавьте в подписку еще одно назначение политики для тега хранилища (повторно используя параметр _storageAccountType_). Этот дополнительный артефакт назначения политики демонстрирует, что определенный в схеме параметр может использоваться несколькими артефактами. В этом примере **storageAccountType** используется для настройки тега группы ресурсов. Это значение предоставляет сведения об учетной записи хранения, которая будет создана на следующем шаге. В этом примере используется встроенная политика "_Задать тег и его значение по умолчанию для групп ресурсов_" с глобальным уникальным идентификатором `49c88fc8-6fd1-46fd-a676-f12d1d3a4c71`.

   - Универсальный код ресурса (URI) REST API

     ```http
     PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/artifacts/policyStorageTags?api-version=2018-11-01-preview
     ```

   - Текст запроса

     ```json
     {
         "kind": "policyAssignment",
         "properties": {
             "description": "Apply storage tag and the parameter also used by the template to resource groups",
             "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/49c88fc8-6fd1-46fd-a676-f12d1d3a4c71",
             "parameters": {
                 "tagName": {
                     "value": "StorageType"
                 },
                 "tagValue": {
                     "value": "[parameters('storageAccountType')]"
                 }
             }
         }
     }
     ```

1. Добавьте шаблон в группу ресурсов. **Текст запроса** шаблона ARM содержит обычный компонент JSON шаблона и определяет целевую группу ресурсов с помощью **properties.resourceGroup**. Этот шаблон также повторно использует параметры схемы **storageAccountType**, **tagName** и **tagValue**, передавая каждый из них в шаблон. Чтобы параметры схемы были доступны шаблону, определяется структура **properties.parameters**. В коде JSON шаблона эта пара "ключ — значение" используется для внедрения значения. Имена параметров схемы и шаблона могут совпадать, но здесь они различаются, чтобы можно было наглядно увидеть, как каждый из них передается из схемы в артефакт шаблона.

   - Универсальный код ресурса (URI) REST API

     ```http
     PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/artifacts/templateStorage?api-version=2018-11-01-preview
     ```

   - Текст запроса

     ```json
     {
         "kind": "template",
         "properties": {
             "template": {
                 "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                 "contentVersion": "1.0.0.0",
                 "parameters": {
                     "storageAccountTypeFromBP": {
                         "type": "string",
                         "defaultValue": "Standard_LRS",
                         "allowedValues": [
                             "Standard_LRS",
                             "Standard_GRS",
                             "Standard_ZRS",
                             "Premium_LRS"
                         ],
                         "metadata": {
                             "description": "Storage Account type"
                         }
                     },
                     "tagNameFromBP": {
                         "type": "string",
                         "defaultValue": "NotSet",
                         "metadata": {
                             "description": "Tag name from blueprint"
                         }
                     },
                     "tagValueFromBP": {
                         "type": "string",
                         "defaultValue": "NotSet",
                         "metadata": {
                             "description": "Tag value from blueprint"
                         }
                     }
                 },
                 "variables": {
                     "storageAccountName": "[concat(uniquestring(resourceGroup().id), 'standardsa')]"
                 },
                 "resources": [{
                     "type": "Microsoft.Storage/storageAccounts",
                     "name": "[variables('storageAccountName')]",
                     "apiVersion": "2016-01-01",
                     "tags": {
                        "[parameters('tagNameFromBP')]": "[parameters('tagValueFromBP')]"
                     },
                     "location": "[resourceGroups('storageRG').location]",
                     "sku": {
                         "name": "[parameters('storageAccountTypeFromBP')]"
                     },
                     "kind": "Storage",
                     "properties": {}
                 }],
                 "outputs": {
                     "storageAccountSku": {
                         "type": "string",
                         "value": "[variables('storageAccountName')]"
                     }
                 }
             },
             "resourceGroup": "storageRG",
             "parameters": {
                 "storageAccountTypeFromBP": {
                     "value": "[parameters('storageAccountType')]"
                 },
                 "tagNameFromBP": {
                     "value": "[parameters('tagName')]"
                 },
                 "tagValueFromBP": {
                     "value": "[parameters('tagValue')]"
                 }
             }
         }
     }
     ```

1. Добавьте назначение ролей в группу ресурсов. Так же как и в предыдущем фрагменте назначения ролей, в приведенном ниже примере используется идентификатор для роли **Владелец**, но из схемы для него передается другой параметр. В этом примере используется встроенная роль _Владелец_ с глобальным уникальным идентификатором `8e3af657-a8ff-443c-a75c-2fe8c4bcb635`.

   - Универсальный код ресурса (URI) REST API

     ```http
     PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/artifacts/roleOwner?api-version=2018-11-01-preview
     ```

   - Текст запроса

     ```json
     {
         "kind": "roleAssignment",
         "properties": {
             "resourceGroup": "storageRG",
             "roleDefinitionId": "/providers/Microsoft.Authorization/roleDefinitions/8e3af657-a8ff-443c-a75c-2fe8c4bcb635",
             "principalIds": "[parameters('owners')]"
         }
     }
     ```

## <a name="publish-a-blueprint"></a>Публикация схемы

Теперь, когда артефакты добавлены в схему, пора ее опубликовать. Публикация схемы позволяет назначать ее подписке.

- Универсальный код ресурса (URI) REST API

  ```http
  PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/versions/{BlueprintVersion}?api-version=2018-11-01-preview
  ```

Значение `{BlueprintVersion}` представляет собой строку, состоящую из букв, цифр и дефисов (без пробелов или других специальных символов), содержащую не более 20 символов. Оно должно быть уникальным и информативным, например **v20180622-135541**.

## <a name="assign-a-blueprint"></a>Назначение схемы

После публикации схемы с помощью REST API ее можно назначить подписке. Назначьте созданную схему одной из подписок в иерархии группы управления. Если схема сохранена в подписке, схему можно будет назначить только этой подписке. В **тексте запроса** указываются назначаемая схема, имена и расположения всех групп ресурсов в определении схемы, а также все параметры, определенные в схеме и используемые одним или несколькими связанными артефактами.

В каждом универсальном коде ресурса (URI) REST API есть переменные, которые необходимо заменить собственными значениями:

- `{tenantId}` — замените своим идентификатором клиента.
- `{YourMG}` — замените это значение идентификатором своей группы управления.
- `{subscriptionId}` — замените это значение идентификатором своей подписки.

1. Предоставьте субъекту-службе Azure Blueprint роль **владельца** в целевой подписке. Идентификатор AppId является статическим (`f71766dc-90d9-4b7d-bd9d-4499c4331c3f`), но идентификаторы субъекта-службы у клиентов различны. Подробные сведения для своего клиента можно запросить с помощью приведенного ниже REST API. Он использует [API Graph Azure Active Directory](/graph/migrate-azure-ad-graph-planning-checklist), схема авторизации которого отличается.

   - Универсальный код ресурса (URI) REST API

     ```http
     GET https://graph.windows.net/{tenantId}/servicePrincipals?api-version=1.6&$filter=appId eq 'f71766dc-90d9-4b7d-bd9d-4499c4331c3f'
     ```

1. Запустите развертывание схемы, назначив ее подписке. Так как в параметрах **contributors** и **owners** должны быть переданы массивы идентификаторов objectId субъектов, которым должно быть предоставлено назначение ролей, используйте [API Graph Azure Active Directory](/graph/migrate-azure-ad-graph-planning-checklist) для сбора идентификаторов objectId, которые вы затем будете использовать в **тексте запроса** для собственных пользователей, групп или субъектов служб.

   - Универсальный код ресурса (URI) REST API

     ```http
     PUT https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Blueprint/blueprintAssignments/assignMyBlueprint?api-version=2018-11-01-preview
     ```

   - Текст запроса

     ```json
     {
         "properties": {
             "blueprintId": "/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint",
             "resourceGroups": {
                 "storageRG": {
                     "name": "StorageAccount",
                     "location": "eastus2"
                 }
             },
             "parameters": {
                 "storageAccountType": {
                     "value": "Standard_GRS"
                 },
                 "tagName": {
                     "value": "CostCenter"
                 },
                 "tagValue": {
                     "value": "ContosoIT"
                 },
                 "contributors": {
                     "value": [
                         "7be2f100-3af5-4c15-bcb7-27ee43784a1f",
                         "38833b56-194d-420b-90ce-cff578296714"
                     ]
                 },
                 "owners": {
                     "value": [
                         "44254d2b-a0c7-405f-959c-f829ee31c2e7",
                         "316deb5f-7187-4512-9dd4-21e7798b0ef9"
                     ]
                 }
             }
         },
         "identity": {
             "type": "systemAssigned"
         },
         "location": "westus"
     }
     ```

   - Управляемое удостоверение, назначаемое пользователем

     Для назначения схемы можно также использовать [управляемое удостоверение, назначаемое пользователем](../../active-directory/managed-identities-azure-resources/overview.md).
     В этом случае раздел **identity** в тексте запроса нужно изменить следующим образом. Замените `{yourRG}` и `{userIdentity}` именем вашей группы ресурсов и именем управляемого удостоверения, назначаемого пользователем, соответственно.

     ```json
     "identity": {
         "type": "userAssigned",
         "tenantId": "{tenantId}",
         "userAssignedIdentities": {
             "/subscriptions/{subscriptionId}/resourceGroups/{yourRG}/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{userIdentity}": {}
         }
     },
     ```

     **Управляемое удостоверение, назначаемое пользователем**, может быть в любой подписке и группе ресурсов, к которым у пользователя, назначающего схему, есть разрешения на доступ.

     > [!IMPORTANT]
     > Azure Blueprints не администрирует управляемое удостоверение, назначаемое пользователем. Назначение соответствующих ролей и разрешений выполняют пользователи. В противном случае назначение схемы завершится ошибкой.

## <a name="clean-up-resources"></a>Очистка ресурсов

### <a name="unassign-a-blueprint"></a>Отмена назначения схемы

Вы можете удалить схему из подписки. Удаление часто выполняется, когда ресурсы артефакта больше не требуются. При удалении схемы назначенные артефакты оставляются. Чтобы отменить назначение схемы, используйте следующую операцию REST API:

- Универсальный код ресурса (URI) REST API

  ```http
  DELETE https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Blueprint/blueprintAssignments/assignMyBlueprint?api-version=2018-11-01-preview
  ```

### <a name="delete-a-blueprint"></a>Удаление схемы

Чтобы удалить саму схему, используйте следующую операцию REST API:

- Универсальный код ресурса (URI) REST API

  ```http
  DELETE https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint?api-version=2018-11-01-preview
  ```

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого краткого руководства вы создали, назначили и удалили схему с помощью REST API. Чтобы узнать больше о схемах Azure, перейдите к статье о жизненном цикле схемы.

> [!div class="nextstepaction"]
> [Ознакомьтесь со сведениями о жизненном цикле схем](./concepts/lifecycle.md)