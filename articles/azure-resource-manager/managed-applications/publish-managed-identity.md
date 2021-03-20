---
title: Управляемое приложение с управляемым удостоверением
description: Настройка управляемого приложения с управляемым удостоверением для связывания с существующими ресурсами, управления ресурсами Azure и предоставления операционной идентификации для журнала действий.
ms.topic: conceptual
ms.author: jobreen
author: jjbfour
ms.date: 05/13/2019
ms.openlocfilehash: 277faa2d47df9fddd1762d90d9aa2fb5bf00d4df
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "82508141"
---
# <a name="azure-managed-application-with-managed-identity"></a>Управляемое удостоверение приложения Azure с помощью управляемого удостоверения

> [!NOTE]
> Поддержка управляемого удостоверения для управляемых приложений сейчас доступна в предварительной версии. Для использования управляемого удостоверения используйте версию API 2018-09-01-Preview.

Узнайте, как настроить управляемое приложение для включения управляемого удостоверения. Управляемое удостоверение можно использовать, чтобы позволить клиенту предоставлять управляемому приложению доступ к дополнительным существующим ресурсам. Удостоверения управляются платформой Azure, и для них не нужно подготавливать или изменять секреты. Дополнительные сведения об управляемых удостоверениях в Azure Active Directory (AAD) см. в статье [управляемые удостоверения для ресурсов Azure](../../active-directory/managed-identities-azure-resources/overview.md).

Приложению можно предоставить два типа удостоверений:

- **Назначаемое системой удостоверение** привязывается к приложению и удаляется при удалении приложения. Приложение может иметь только одно назначаемое системой удостоверение.
- **Назначаемое пользователем удостоверение** — это изолированный ресурс Azure, который можно назначить приложению. Приложение может иметь несколько назначаемых пользователем удостоверений.

## <a name="how-to-use-managed-identity"></a>Как использовать управляемое удостоверение

Управляемое удостоверение позволяет выполнять многие сценарии для управляемых приложений. Ниже приведены некоторые распространенные сценарии, которые можно решить:

- Развертывание управляемого приложения, связанного с существующими ресурсами Azure. Примером является развертывание виртуальной машины Azure в управляемом приложении, подключенном к [существующему сетевому интерфейсу](../../virtual-network/virtual-network-network-interface-vm.md).
- Предоставление управляемому приложению и издателю доступа к ресурсам Azure за пределами **управляемой группы ресурсов**.
- Предоставление операционной идентификации управляемых приложений для журнала действий и других служб в Azure.

## <a name="adding-managed-identity"></a>Добавление управляемого удостоверения

Для создания управляемого приложения с управляемым удостоверением требуется задать дополнительное свойство для ресурса Azure. В следующем примере показан пример свойства **Identity** :

```json
{
"identity": {
    "type": "SystemAssigned, UserAssigned",
    "userAssignedIdentities": {
        "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.ManagedIdentity/userassignedidentites/myuserassignedidentity": {}
    }
}
```

Существует два стандартных способа создания управляемого приложения с помощью **Identity**: [CreateUIDefinition.jsв](./create-uidefinition-overview.md) [шаблонах и Azure Resource Manager](../templates/template-syntax.md). Для простых сценариев однократного создания CreateUIDefinition следует использовать, чтобы включить управляемое удостоверение, так как оно обеспечивает более широкие возможности. Однако при работе с расширенными или сложными системами, требующими автоматизированного или множественного развертывания управляемых приложений, можно использовать шаблоны.

### <a name="using-createuidefinition"></a>Использование CreateUIDefinition

Управляемое приложение можно настроить с помощью управляемого удостоверения с помощью [CreateUIDefinition.jsв](./create-uidefinition-overview.md). В [разделе Outputs](./create-uidefinition-overview.md#outputs) `managedIdentity` можно использовать ключ для переопределения свойства Identity шаблона управляемого приложения. Пример ниже включит в управляемом приложении удостоверение, **назначенное системой** . Более сложные объекты Identity можно формировать с помощью элементов CreateUIDefinition, чтобы задать потребителю входные данные. Эти входные данные можно использовать для создания управляемых приложений с **удостоверением, назначенным пользователем**.

```json
"outputs": {
    "managedIdentity": { "Type": "SystemAssigned" }
}
```

#### <a name="when-to-use-createuidefinition-for-managed-identity"></a>Когда следует использовать CreateUIDefinition для управляемого удостоверения

Ниже приведены рекомендации по использованию CreateUIDefinition для включения управляемого удостоверения в управляемых приложениях.

- Создание управляемого приложения проходит через портал Azure или Marketplace.
- Для управляемого удостоверения требуются сложные входные данные потребителя.
- Для создания управляемого приложения требуется управляемое удостоверение.

#### <a name="managed-identity-createuidefinition-control"></a>Управляемый элемент управления CreateUIDefinition Identity

CreateUIDefinition поддерживает встроенный [управляемый элемент управления идентификацией](./microsoft-managedidentity-identityselector.md).

```json
{
  "$schema": "https://schema.management.azure.com/schemas/0.1.2-preview/CreateUIDefinition.MultiVm.json#",
  "handler": "Microsoft.Azure.CreateUIDef",
  "version": "0.0.1-preview",
  "parameters": {
    "basics": [],
    "steps": [
      {
        "name": "applicationSettings",
        "label": "Application Settings",
        "subLabel": {
          "preValidation": "Configure your application settings",
          "postValidation": "Done"
        },
        "bladeTitle": "Application Settings",
        "elements": [
          {
            "name": "appName",
            "type": "Microsoft.Common.TextBox",
            "label": "Managed application Name",
            "toolTip": "Managed application instance name",
            "visible": true
          },
          {
            "name": "appIdentity",
            "type": "Microsoft.ManagedIdentity.IdentitySelector",
            "label": "Managed Identity Configuration",
            "toolTip": {
              "systemAssignedIdentity": "Enable system assigned identity to grant the managed application access to additional existing resources.",
              "userAssignedIdentity": "Add user assigned identities to grant the managed application access to additional existing resources."
            },
            "defaultValue": {
              "systemAssignedIdentity": "Off"
            },
            "options": {
              "hideSystemAssignedIdentity": false,
              "hideUserAssignedIdentity": false,
              "readOnlySystemAssignedIdentity": false
            },
            "visible": true
          }
        ]
      }
    ],
    "outputs": {
      "applicationResourceName": "[steps('applicationSettings').appName]",
      "location": "[location()]",
      "managedIdentity": "[steps('applicationSettings').appIdentity]"
    }
  }
}
```

![CreateUIDefinition управляемых удостоверений](./media/publish-managed-identity/msi-cuid.png)

### <a name="using-azure-resource-manager-templates"></a>Использование шаблонов Azure Resource Manager

> [!NOTE]
> Шаблоны управляемых приложений Marketplace автоматически создаются для клиентов, которые проходят через портал Azure создания.
> В этих сценариях для `managedIdentity` включения удостоверения необходимо использовать выходной ключ в CreateUIDefinition.

Управляемое удостоверение можно также включить с помощью шаблонов Azure Resource Manager. Пример ниже включит в управляемом приложении удостоверение, **назначенное системой** . Более сложные объекты Identity можно формировать с помощью Azure Resource Manager параметров шаблона для предоставления входных данных. Эти входные данные можно использовать для создания управляемых приложений с **удостоверением, назначенным пользователем**.

#### <a name="when-to-use-azure-resource-manager-templates-for-managed-identity"></a>Использование шаблонов Azure Resource Manager для управляемого удостоверения

Ниже приведены рекомендации по использованию шаблонов Azure Resource Manager для включения управляемого удостоверения в управляемых приложениях.

- Управляемые приложения можно развертывать программно на основе шаблона.
- Пользовательские назначения ролей для управляемого удостоверения необходимы для инициализации управляемого приложения.
- Управляемому приложению не требуется процесс создания портал Azure и Marketplace.

#### <a name="systemassigned-template"></a>Шаблон SystemAssigned

Базовый шаблон Azure Resource Manager, который развертывает управляемое приложение с удостоверением, **назначенным системой** .

```json
"resources": [
    {
        "type": "Microsoft.Solutions/applications",
        "name": "[parameters('applicationName')]",
        "apiVersion": "2018-09-01-preview",
        "location": "[parameters('location')]",
        "identity": {
            "type": "SystemAssigned"
        },
        "properties": {
            "ManagedResourceGroupId": "[parameters('managedByResourceGroupId')]",
            "parameters": { }
        }
    }
]
```

### <a name="userassigned-template"></a>Шаблон Усерассигнед

Базовый шаблон Azure Resource Manager, который развертывает управляемое приложение с **удостоверением, назначенным пользователем**.

```json
"resources": [
    {
      "type": "Microsoft.ManagedIdentity/userAssignedIdentities",
      "name": "[parameters('managedIdentityName')]",
      "apiVersion": "2018-11-30",
      "location": "[parameters('location')]"
    },
    {
        "type": "Microsoft.Solutions/applications",
        "name": "[parameters('applicationName')]",
        "apiVersion": "2018-09-01-preview",
        "location": "[parameters('location')]",
        "identity": {
            "type": "UserAssigned",
            "userAssignedIdentities": {
                "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/',parameters('managedIdentityName'))]": {}
            }
        },
        "properties": {
            "ManagedResourceGroupId": "[parameters('managedByResourceGroupId')]",
            "parameters": { }
        }
    }
]
```

## <a name="granting-access-to-azure-resources"></a>Предоставление доступа к ресурсам Azure

После того как управляемому приложению будет предоставлено удостоверение, ему может быть предоставлен доступ к существующим ресурсам Azure. Этот процесс можно выполнить с помощью интерфейса управления доступом (IAM) в портал Azure. Для добавления назначения ролей можно выполнить поиск имени управляемого приложения или **удостоверения, назначенного пользователю** .

![Добавление назначения ролей для управляемого приложения](./media/publish-managed-identity/identity-role-assignment.png)

## <a name="linking-existing-azure-resources"></a>Связывание существующих ресурсов Azure

> [!NOTE]
> Перед развертыванием управляемого приложения необходимо [настроить](../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md) **удостоверение, назначенное пользователем** . Кроме того, развертывание связанных ресурсов для управляемых приложений поддерживается только для типа **Marketplace** .

Управляемое удостоверение также можно использовать для развертывания управляемого приложения, которое требует доступа к существующим ресурсам во время его развертывания. Когда управляемое приложение подготавливается клиентом, можно добавить **пользовательские удостоверения** , чтобы предоставить дополнительные разрешения на развертывание **mainTemplate** .

### <a name="authoring-the-createuidefinition-with-a-linked-resource"></a>Создание CreateUIDefinition с помощью связанного ресурса

При связывании развертывания управляемого приложения с существующими ресурсами необходимо предоставить как существующий ресурс Azure, так и **назначенное пользователем удостоверение** с соответствующим назначением ролей для этого ресурса.

 Пример CreateUIDefinition, для которого требуется два входа: идентификатор ресурса сетевого интерфейса и идентификатор ресурса удостоверения, назначенный пользователем.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/0.1.2-preview/CreateUIDefinition.MultiVm.json#",
    "handler": "Microsoft.Compute.MultiVm",
    "version": "0.1.2-preview",
    "parameters": {
        "basics": [
            {}
        ],
        "steps": [
            {
                "name": "managedApplicationSetting",
                "label": "Managed Application Settings",
                "subLabel": {
                    "preValidation": "Managed Application Settings",
                    "postValidation": "Done"
                },
                "bladeTitle": "Managed Application Settings",
                "elements": [
                    {
                        "name": "networkInterfaceId",
                        "type": "Microsoft.Common.TextBox",
                        "label": "network interface resource id",
                        "defaultValue": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Network/networkInterfaces/existingnetworkinterface",
                        "toolTip": "Must represent the identity as an Azure Resource Manager resource identifer format ex. /subscriptions/sub1/resourcegroups/myGroup/providers/Microsoft.Network/networkInterfaces/networkinterface1",
                        "visible": true
                    },
                    {
                        "name": "userAssignedId",
                        "type": "Microsoft.Common.TextBox",
                        "label": "user assigned identity resource id",
                        "defaultValue": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.ManagedIdentity/userassignedidentites/myuserassignedidentity",
                        "toolTip": "Must represent the identity as an Azure Resource Manager resource identifer format ex. /subscriptions/sub1/resourcegroups/myGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/identity1",
                        "visible": true
                    }
                ]
            }
        ],
        "outputs": {
            "existingNetworkInterfaceId": "[steps('managedApplicationSetting').networkInterfaceId]",
            "managedIdentity": "[parse(concat('{\"Type\":\"UserAssigned\",\"UserAssignedIdentities\":{',string(steps('managedApplicationSetting').userAssignedId),':{}}}'))]"
        }
    }
}
```

Эта CreateUIDefinition.jsна создание пользовательского интерфейса с двумя полями. Первое поле позволяет пользователю ввести идентификатор ресурса Azure для ресурса, связанного с развертыванием управляемого приложения. Во-вторых, потребитель вводит идентификатор ресурса Azure, **назначенный пользователю** , который имеет доступ к связанному ресурсу Azure. Созданный интерфейс выглядит следующим образом:

![Пример CreateUIDefinition с двумя входными данными: идентификатор ресурса сетевого интерфейса и идентификатор ресурса удостоверения, назначенный пользователем.](./media/publish-managed-identity/network-interface-cuid.png)

### <a name="authoring-the-maintemplate-with-a-linked-resource"></a>Создание mainTemplate с помощью связанного ресурса

Кроме обновления CreateUIDefinition, основной шаблон также необходимо обновить, чтобы принять переданный идентификатор связанного ресурса. Основной шаблон можно обновить, чтобы принять новые выходные данные, добавив новый параметр. Поскольку `managedIdentity` выходные данные переопределяют значение в созданном шаблоне управляемого приложения, оно не передается в основной шаблон и не должно включаться в раздел Parameters.

Пример основного шаблона, который задает сетевой профиль для существующего сетевого интерфейса, предоставляемого CreateUIDefinition.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
      "existingNetworkInterfaceId": { "type": "string" }
    },
    "variables": {
    },
    "resources": [
        {
            "apiVersion": "2016-04-30-preview",
            "type": "Microsoft.Compute/virtualMachines",
            "name": "myLinkedResourceVM",
            "location": "[resourceGroup().location]",
            "properties": {
                …,
                "networkProfile": {
                    "networkInterfaces": [
                        {
                            "id": "[parameters('existingNetworkInterfaceId')]"
                        }
                    ]
                }
            }
        }
    ]
}
```

### <a name="consuming-the-managed-application-with-a-linked-resource"></a>Использование управляемого приложения со связанным ресурсом

После создания пакета управляемого приложения управляемое приложение можно использовать с помощью портал Azure. Прежде чем его можно будет использовать, необходимо выполнить несколько необходимых действий.

- Необходимо создать экземпляр необходимого связанного ресурса Azure.
- Необходимо создать **назначаемое пользователем удостоверение** и назначить ему [назначение ролей](../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md) для связанного ресурса.
- Существующий идентификатор связанного ресурса и **назначенный пользователем идентификатор удостоверения** предоставляются CreateUIDefinition.

## <a name="accessing-the-managed-identity-token"></a>Доступ к маркеру управляемого удостоверения

Теперь доступ к маркеру управляемого приложения можно получить через `listTokens` API клиента издателя. Пример запроса может выглядеть следующим образом:

``` HTTP
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Solutions/applications/{applicationName}/listTokens?api-version=2018-09-01-preview HTTP/1.1

{
    "authorizationAudience": "https://management.azure.com/",
    "userAssignedIdentities": [
        "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{userAssignedIdentityName}"
    ]
}
```

Параметры текста запроса:

Параметр | Обязательно | Описание
---|---|---
аусоризатионаудиенце | *Нет* | URI идентификатора приложения целевого ресурса. Кроме того, это `aud` утверждение (аудитория) выданного маркера. Значение по умолчанию — " https://management.azure.com/ "
userAssignedIdentities | *Нет* | Список назначаемых пользователем управляемых удостоверений для получения маркера. Если он не указан, `listTokens` возвращает маркер для управляемого системой удостоверения.


Пример ответа может выглядеть следующим образом:

``` HTTP
HTTP/1.1 200 OK
Content-Type: application/json

{
    "value": [
        {
            "access_token": "eyJ0eXAi…",
            "expires_in": "2…",
            "expires_on": "1557…",
            "not_before": "1557…",
            "authorizationAudience": "https://management.azure.com/",
            "resourceId": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Solutions/applications/{applicationName}",
            "token_type": "Bearer"
        }
    ]
}
```

Ответ будет содержать массив токенов в `value` свойстве:

Параметр | Описание
---|---
access_token | Запрашиваемый маркер доступа.
expires_in | Количество секунд, в течение которых маркер доступа будет действителен.
expires_on | Период времени после истечения срока действия маркера доступа. Представляется в виде числа секунд от эпохи.
not_before | Интервал времени, когда вступает в силу маркер доступа. Представляется в виде числа секунд от эпохи.
аусоризатионаудиенце | Маркер доступа, запрошенный `aud` (аудиторией). Это то же самое, что было указано в `listTokens` запросе.
resourceId | Идентификатор ресурса Azure для выданного маркера. Это либо идентификатор управляемого приложения, либо идентификатор удостоверения, назначенный пользователем.
token_type | Тип маркера.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Настройка управляемого приложения с помощью настраиваемого поставщика](../custom-providers/overview.md)
