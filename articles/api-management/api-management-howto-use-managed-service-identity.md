---
title: Использование управляемых удостоверений в службе управления API Azure | Документация Майкрософт
description: Узнайте, как создавать назначенные системой и назначенные пользователем удостоверения в управлении API с помощью портал Azure, PowerShell и шаблона диспетчер ресурсов.
services: api-management
documentationcenter: ''
author: miaojiang
manager: anneta
editor: ''
ms.service: api-management
ms.workload: integration
ms.topic: article
ms.date: 03/09/2021
ms.author: apimpm
ms.openlocfilehash: 98237efae89e7d88dd23cb7e8fc9f7e9f05bca70
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102521549"
---
# <a name="use-managed-identities-in-azure-api-management"></a>Использование управляемых удостоверений в службе управления API Azure

В этой статье показано, как создать управляемое удостоверение для экземпляра службы управления API Azure и как получить доступ к другим ресурсам. Управляемое удостоверение, созданное Azure Active Directory (Azure AD), позволяет экземпляру службы управления API легко и безопасно получать доступ к другим ресурсам, защищенным Azure AD, таким как Azure Key Vault. Azure управляет этим удостоверением, поэтому вам не нужно подготавливать или поворачивать какие бы то ни было секреты. Дополнительные сведения об управляемых удостоверениях см. в статье [что такое управляемые удостоверения для ресурсов Azure?](../active-directory/managed-identities-azure-resources/overview.md).

Экземпляру управления API можно предоставить два типа удостоверений.

- *Назначенное системой удостоверение* связано со службой и удаляется, если служба удалена. Служба может иметь только одно назначенное системой удостоверение.
- *Назначаемое пользователем удостоверение* — это автономный ресурс Azure, который можно назначить службе. Служба может иметь несколько назначенных пользователю удостоверений.

## <a name="create-a-system-assigned-managed-identity"></a>Создание управляемого удостоверения, назначенного системой

### <a name="azure-portal"></a>Портал Azure

Чтобы настроить управляемое удостоверение в портал Azure, сначала необходимо создать экземпляр управления API, а затем включить эту функцию.

1. Создайте экземпляр управления API на портале как обычно. Перейдите к нему на портале.
2. Выберите **управляемые удостоверения**.
3. На вкладке **назначенная система** переключите **состояние** **на вкл**. Щелкните **Сохранить**.

    :::image type="content" source="./media/api-management-msi/enable-system-msi.png" alt-text="Параметры для включения управляемого удостоверения, назначенного системой" border="true":::

### <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Следующие шаги описывают создание экземпляра управления API и назначение ему удостоверения с помощью Azure PowerShell. 

1. При необходимости установите Azure PowerShell с помощью инструкций, приведенных в разделе инструкции по [Azure PowerShell](/powershell/azure/install-az-ps). Затем выполните команду `Connect-AzAccount` , чтобы создать подключение к Azure.

2. Используйте следующий код для создания экземпляра. Дополнительные примеры использования Azure PowerShell с экземпляром управления API см. в разделе [примеры PowerShell для управления API](powershell-samples.md).

    ```azurepowershell-interactive
    # Create a resource group.
    New-AzResourceGroup -Name $resourceGroupName -Location $location

    # Create an API Management Consumption Sku service.
    New-AzApiManagement -ResourceGroupName $resourceGroupName -Name consumptionskuservice -Location $location -Sku Consumption -Organization contoso -AdminEmail contoso@contoso.com -SystemAssignedIdentity
    ```

3. Обновите существующий экземпляр, чтобы создать удостоверение:

    ```azurepowershell-interactive
    # Get an API Management instance
    $apimService = Get-AzApiManagement -ResourceGroupName $resourceGroupName -Name $apiManagementName

    # Update an API Management instance
    Set-AzApiManagement -InputObject $apimService -SystemAssignedIdentity
    ```

### <a name="azure-resource-manager-template"></a>Шаблон Azure Resource Manager

Чтобы создать экземпляр службы управления API с удостоверением, добавьте в определение ресурса следующее свойство:

```json
"identity" : {
    "type" : "SystemAssigned"
}
```

Это свойство указывает платформе Azure, что требуется создать удостоверение для экземпляра службы управления API и управлять им.

Например, полный шаблон Azure Resource Manager может выглядеть так:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    "contentVersion": "0.9.0.0",
    "resources": [{
        "apiVersion": "2019-01-01",
        "name": "contoso",
        "type": "Microsoft.ApiManagement/service",
        "location": "[resourceGroup().location]",
        "tags": {},
        "sku": {
            "name": "Developer",
            "capacity": "1"
        },
        "properties": {
            "publisherEmail": "admin@contoso.com",
            "publisherName": "Contoso"
        },
        "identity": {
            "type": "systemAssigned"
        }
    }]
}
```

При создании экземпляра он имеет следующие дополнительные свойства:

```json
"identity": {
    "type": "SystemAssigned",
    "tenantId": "<TENANTID>",
    "principalId": "<PRINCIPALID>"
}
```

`tenantId`Свойство определяет, к какому клиенту Azure AD принадлежит удостоверение. `principalId`Свойство является уникальным идентификатором для нового удостоверения экземпляра. В Azure AD субъект-служба имеет то же имя, которое вы присвоили экземпляру службы управления API.

> [!NOTE]
> Экземпляр управления API может одновременно иметь как назначенные системой, так и назначенные пользователем удостоверения. В этом случае свойство будет `type` иметь значение `SystemAssigned,UserAssigned` .

## <a name="supported-scenarios-using-system-assigned-identity"></a>Поддерживаемые сценарии с использованием удостоверения, назначенного системой

### <a name="obtain-a-custom-tlsssl-certificate-for-the-api-management-instance-from-azure-key-vault"></a><a name="use-ssl-tls-certificate-from-azure-key-vault"></a>Получите настраиваемый сертификат TLS/SSL для экземпляра управления API из Azure Key Vault
Для получения настраиваемых сертификатов TLS/SSL, хранящихся в Azure Key Vault, можно использовать назначенное системой удостоверение экземпляра управления API. Затем эти сертификаты можно назначить пользовательским доменам в экземпляре управления API. Придерживайтесь следующих рекомендаций:

- Тип содержимого секрета должен быть *Application/x-PKCS12*.
- Используйте секретную конечную точку сертификата Key Vault, которая содержит секрет.

> [!Important]
> Если вы не предложит версию объекта сертификата, управление API автоматически получит более новую версию сертификата в течение четырех часов после обновления в Key Vault.

В следующем примере показан шаблон Azure Resource Manager, который содержит следующее:

1. Создание экземпляра управления API с управляемым удостоверением.
2. Обновление политик доступа экземпляра Azure Key Vault и разрешение экземпляру службы управления API получать секреты из экземпляра Azure Key Vault.
3. Настройка имени личного домена для экземпляра службы управления API с помощью сертификата из экземпляра Key Vault.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "publisherEmail": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "The email address of the owner of the service"
            }
        },
        "publisherName": {
            "type": "string",
            "defaultValue": "Contoso",
            "minLength": 1,
            "metadata": {
                "description": "The name of the owner of the service"
            }
        },
        "sku": {
            "type": "string",
            "allowedValues": ["Developer",
            "Standard",
            "Premium"],
            "defaultValue": "Developer",
            "metadata": {
                "description": "The pricing tier of this API Management instance"
            }
        },
        "skuCount": {
            "type": "int",
            "defaultValue": 1,
            "metadata": {
                "description": "The instance size of this API Management instance."
            }
        },
        "keyVaultName": {
            "type": "string",
            "metadata": {
                "description": "Name of the vault"
            }
        },
        "proxyCustomHostname1": {
            "type": "string",
            "metadata": {
                "description": "Proxy Custom hostname."
            }
        },
        "keyVaultIdToCertificate": {
            "type": "string",
            "metadata": {
                "description": "Reference to the Key Vault certificate. https://contoso.vault.azure.net/secrets/contosogatewaycertificate."
            }
        }
    },
    "variables": {
        "apiManagementServiceName": "[concat('apiservice', uniqueString(resourceGroup().id))]",
        "apimServiceIdentityResourceId": "[concat(resourceId('Microsoft.ApiManagement/service', variables('apiManagementServiceName')),'/providers/Microsoft.ManagedIdentity/Identities/default')]"
    },
    "resources": [{
        "apiVersion": "2019-01-01",
        "name": "[variables('apiManagementServiceName')]",
        "type": "Microsoft.ApiManagement/service",
        "location": "[resourceGroup().location]",
        "tags": {
        },
        "sku": {
            "name": "[parameters('sku')]",
            "capacity": "[parameters('skuCount')]"
        },
        "properties": {
            "publisherEmail": "[parameters('publisherEmail')]",
            "publisherName": "[parameters('publisherName')]"
        },
        "identity": {
            "type": "systemAssigned"
        }
    },
    {
        "type": "Microsoft.KeyVault/vaults/accessPolicies",
        "name": "[concat(parameters('keyVaultName'), '/add')]",
        "apiVersion": "2015-06-01",
        "dependsOn": [
            "[resourceId('Microsoft.ApiManagement/service', variables('apiManagementServiceName'))]"
        ],
        "properties": {
            "accessPolicies": [{
                "tenantId": "[reference(variables('apimServiceIdentityResourceId'), '2015-08-31-PREVIEW').tenantId]",
                "objectId": "[reference(variables('apimServiceIdentityResourceId'), '2015-08-31-PREVIEW').principalId]",
                "permissions": {
                    "secrets": ["get"]
                }
            }]
        }
    },
    {
        "apiVersion": "2017-05-10",
        "name": "apimWithKeyVault",
        "type": "Microsoft.Resources/deployments",
        "dependsOn": [
        "[resourceId('Microsoft.ApiManagement/service', variables('apiManagementServiceName'))]"
        ],
        "properties": {
            "mode": "incremental",
            "templateLink": {
                "uri": "https://raw.githubusercontent.com/solankisamir/arm-templates/master/basicapim.keyvault.json",
                "contentVersion": "1.0.0.0"
            },
            "parameters": {
                "publisherEmail": { "value": "[parameters('publisherEmail')]"},
                "publisherName": { "value": "[parameters('publisherName')]"},
                "sku": { "value": "[parameters('sku')]"},
                "skuCount": { "value": "[parameters('skuCount')]"},
                "proxyCustomHostname1": {"value" : "[parameters('proxyCustomHostname1')]"},
                "keyVaultIdToCertificate": {"value" : "[parameters('keyVaultIdToCertificate')]"}
            }
        }
    }]
}
```

### <a name="authenticate-to-the-back-end-by-using-an-api-management-identity"></a>Проверка подлинности в серверной части с помощью удостоверения управления API

Вы можете использовать назначенное системой удостоверение для проверки подлинности в серверной части с помощью политики [аутентификации, управляемой удостоверениями](api-management-authentication-policies.md#ManagedIdentity) .

### <a name="connect-to-azure-resources-behind-ip-firewall-using-system-assigned-managed-identity"></a><a name="apim-as-trusted-service"></a>Подключение к ресурсам Azure за брандмауэром IP-адресов с помощью управляемого удостоверения, назначенного системой


Управление API — это надежная служба Майкрософт для следующих ресурсов. Это позволяет службе подключаться к следующим ресурсам за брандмауэром. После явного назначения соответствующей роли Azure [управляемому удостоверению, назначенному системой](../active-directory/managed-identities-azure-resources/overview.md) для этого экземпляра ресурса, область доступа для экземпляра соответствует роли Azure, назначенной управляемому удостоверению.


|Служба Azure | Ссылка|
|---|---|
|Хранилище Azure | [Доверенный доступ к Azure — хранилище](../storage/common/storage-network-security.md?tabs=azure-portal#trusted-access-based-on-system-assigned-managed-identity)|
|Служебная шина Azure | [Доверенный доступ к Azure-Service-Bus](../service-bus-messaging/service-bus-ip-filtering.md#trusted-microsoft-services)|
|концентратору событий Azure | [Надежные — доступ к Azure-Event-концентратору](../event-hubs/event-hubs-ip-filtering.md#trusted-microsoft-services)|


## <a name="create-a-user-assigned-managed-identity"></a>Создание управляемого удостоверения, назначаемого пользователем

> [!NOTE]
> Экземпляр управления API можно связать с управляемыми идентификаторами, имеющими до 10.

### <a name="azure-portal"></a>Портал Azure

Чтобы настроить управляемое удостоверение на портале, сначала необходимо создать экземпляр управления API, а затем включить эту функцию.

1. Создайте экземпляр управления API на портале как обычно. Перейдите к нему на портале.
2. Выберите **управляемые удостоверения**.
3. На вкладке **Назначенные пользователи** выберите **Добавить**.
4. Найдите созданное ранее удостоверение и выберите его. Выберите **Добавить**.

   :::image type="content" source="./media/api-management-msi/enable-user-assigned-msi.png" alt-text="Параметры для включения управляемого удостоверения, назначенного пользователем" border="true":::

### <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Следующие шаги описывают создание экземпляра управления API и назначение ему удостоверения с помощью Azure PowerShell. 

1. При необходимости установите Azure PowerShell, следуя инструкциям в разделе инструкции по [Azure PowerShell](/powershell/azure/install-az-ps). Затем выполните команду `Connect-AzAccount` , чтобы создать подключение к Azure.

2. Используйте следующий код для создания экземпляра. Дополнительные примеры использования Azure PowerShell с экземпляром управления API см. в разделе [примеры PowerShell для управления API](powershell-samples.md).

    ```azurepowershell-interactive
    # Create a resource group.
    New-AzResourceGroup -Name $resourceGroupName -Location $location

    # Create a user-assigned identity. This requires installation of the "Az.ManagedServiceIdentity" module.
    $userAssignedIdentity = New-AzUserAssignedIdentity -Name $userAssignedIdentityName -ResourceGroupName $resourceGroupName

    # Create an API Management Consumption Sku service.
    $userIdentities = @($userAssignedIdentity.Id)

    New-AzApiManagement -ResourceGroupName $resourceGroupName -Location $location -Name $apiManagementName -Organization contoso -AdminEmail admin@contoso.com -Sku Consumption -UserAssignedIdentity $userIdentities
    ```

3. Обновление существующей службы для назначения удостоверения службе:

    ```azurepowershell-interactive
    # Get an API Management instance
    $apimService = Get-AzApiManagement -ResourceGroupName $resourceGroupName -Name $apiManagementName

    # Create a user-assigned identity. This requires installation of the "Az.ManagedServiceIdentity" module.
    $userAssignedIdentity = New-AzUserAssignedIdentity -Name $userAssignedIdentityName -ResourceGroupName $resourceGroupName

    # Update an API Management instance
    $userIdentities = @($userAssignedIdentity.Id)
    Set-AzApiManagement -InputObject $apimService -UserAssignedIdentity $userIdentities
    ```

### <a name="azure-resource-manager-template"></a>Шаблон Azure Resource Manager

Чтобы создать экземпляр службы управления API с удостоверением, добавьте в определение ресурса следующее свойство:

```json
"identity": {
    "type": "UserAssigned",
    "userAssignedIdentities": {
        "<RESOURCEID>": {}
    }
}
```

Добавление назначенного пользователем типа указывает Azure использовать назначенное пользователем удостоверение, указанное для вашего экземпляра.

Например, полный шаблон Azure Resource Manager может выглядеть так:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    "contentVersion": "0.9.0.0",
    "resources": [{
        "apiVersion": "2019-12-01",
        "name": "contoso",
        "type": "Microsoft.ApiManagement/service",
        "location": "[resourceGroup().location]",
        "tags": {},
        "sku": {
            "name": "Developer",
            "capacity": "1"
        },
        "properties": {
            "publisherEmail": "admin@contoso.com",
            "publisherName": "Contoso"
        },
        "identity": {
            "type": "UserAssigned",
             "userAssignedIdentities": {
                "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', variables('identityName'))]": {}
             }
        },
         "dependsOn": [       
          "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', variables('identityName'))]"
        ]
    }]
}
```

При создании службы он имеет следующие дополнительные свойства:

```json
"identity": {
    "type": "UserAssigned",
    "userAssignedIdentities": {
        "<RESOURCEID>": {
            "principalId": "<PRINCIPALID>",
            "clientId": "<CLIENTID>"
        }
    }
}
```

`principalId`Свойство — это уникальный идентификатор для удостоверения, используемого для администрирования Azure AD. `clientId`Свойство является уникальным идентификатором для нового удостоверения приложения, которое используется для указания того, какое удостоверение использовать во время вызовов среды выполнения.

> [!NOTE]
> Экземпляр управления API может одновременно иметь как назначенные системой, так и назначенные пользователем удостоверения. В этом случае свойство будет `type` иметь значение `SystemAssigned,UserAssigned` .

## <a name="supported-scenarios-using-user-assigned-managed-identity"></a>Поддерживаемые сценарии с использованием управляемого удостоверения, назначенного пользователем

### <a name="obtain-a-custom-tlsssl-certificate-for-the-api-management-instance-from-azure-key-vault"></a><a name="use-ssl-tls-certificate-from-azure-key-vault-ua"></a>Получите настраиваемый сертификат TLS/SSL для экземпляра управления API из Azure Key Vault
Для установления отношений доверия между экземпляром управления API и KeyVault можно использовать любое назначенное пользователем удостоверение. Это отношение доверия можно использовать для получения настраиваемых сертификатов TLS/SSL, хранящихся в Azure Key Vault. Затем эти сертификаты можно назначить пользовательским доменам в экземпляре управления API. 

Придерживайтесь следующих рекомендаций:

- Тип содержимого секрета должен быть *Application/x-PKCS12*.
- Используйте секретную конечную точку сертификата Key Vault, которая содержит секрет.

> [!Important]
> Если вы не предложит версию объекта сертификата, управление API автоматически получит более новую версию сертификата в течение четырех часов после обновления в Key Vault.

Полный шаблон см. в разделе [Управление API с помощью SSL на основе KeyVault с использованием удостоверения, назначенного пользователем](https://github.com/Azure/azure-quickstart-templates/blob/master/101-api-management-key-vault-create/azuredeploy.json).

В этом шаблоне будут развернуты перечисленные ниже компоненты.

* Служба "Управление API Azure"
* Удостоверение, назначенное пользователем, управляемым Azure
* Azure KeyVault для хранения сертификата SSL/TLS

Чтобы выполнить развертывание автоматически, нажмите следующую кнопку.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-api-management-key-vault-create%2Fazuredeploy.json)

### <a name="authenticate-to-the-back-end-by-using-a-user-assigned-identity"></a>Проверка подлинности в серверной части с помощью назначенного пользователем удостоверения

Вы можете использовать назначенное пользователем удостоверение для проверки подлинности в серверной части с помощью политики [аутентификации, управляемой удостоверениями](api-management-authentication-policies.md#ManagedIdentity) .

## <a name="remove-an-identity"></a><a name="remove"></a>Удаление удостоверения

Вы можете удалить назначенное системой удостоверение, отключив эту функцию на портале или шаблоне Azure Resource Manager так же, как он был создан. Назначаемые пользователем удостоверения можно удалить по отдельности. Чтобы удалить все удостоверения, задайте для параметра Тип удостоверения значение `"None"` .

Если назначаемое системой удостоверение удаляется таким способом, то оно также удаляется и из Azure AD. Назначенные системой удостоверения также автоматически удаляются из Azure AD при удалении экземпляра службы управления API.

Чтобы удалить все удостоверения с помощью шаблона Azure Resource Manager, обновите этот раздел:

```json
"identity": {
    "type": "None"
}
```

> [!Important]
> Если экземпляр управления API настроен с использованием пользовательского SSL-сертификата из Key Vault и вы пытаетесь отключить управляемое удостоверение, запрос завершится ошибкой.
>
> Вы можете разблокировать себя, переключив сертификат Azure Key Vault во встроенный закодированный сертификат, а затем отключив управляемое удостоверение. Дополнительные сведения см. в разделе [Настройка имени домена](configure-custom-domain.md).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об управляемых удостоверениях для ресурсов Azure:

* [Что такое управляемые удостоверения для ресурсов Azure?](../active-directory/managed-identities-azure-resources/overview.md)
* [Шаблоны диспетчера ресурсов Azure](https://github.com/Azure/azure-quickstart-templates)
* [Проверка подлинности с помощью управляемого удостоверения в политике](./api-management-authentication-policies.md#ManagedIdentity)
