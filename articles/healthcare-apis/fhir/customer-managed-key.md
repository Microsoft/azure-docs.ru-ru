---
title: Настройка ключей, управляемых клиентом, для Azure API для FHIR
description: Поддержка функции создания собственного ключа в Azure API для FHIR с помощью Cosmos DB
services: healthcare-apis
author: ginalee-dotcom
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: overview
ms.date: 09/28/2020
ms.author: ginle
ms.openlocfilehash: daa71a6df0ad412823736b3ee094cfd3945af492
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/02/2021
ms.locfileid: "106220855"
---
# <a name="configure-customer-managed-keys-at-rest"></a>Настройка хранимых ключей, управляемых клиентом

При создании учетной записи Azure API для FHIR данные шифруются по умолчанию с помощью ключей, управляемых Майкрософт. Теперь можно добавить второй уровень шифрования данных, используя собственный ключ, который вы выбираете и которым управляете.

В Azure это обычно выполняется с помощью ключа шифрования в Azure Key Vault клиента. SQL Azure, служба хранилища Azure и Cosmos DB — это несколько примеров, предоставляющих эти возможности уже сегодня. Azure API для FHIR использует эту поддержку через Cosmos DB. При создании учетной записи у вас будет возможность указать универсальный код ресурса (URI) ключа Azure Key Vault. Этот ключ будет передан Cosmos DB при подготовке учетной записи базы данных. Когда выполняется запрос FHIR, Cosmos DB получает ключ и использует его для шифрования или расшифровки данных. Чтобы приступить к работе, воспользуйтесь следующими ссылками:

- [Регистрация поставщика ресурсов Azure Cosmos DB в своей подписке Azure](../../cosmos-db/how-to-setup-cmk.md#register-resource-provider) 
- [Настройка экземпляра Azure Key Vault](../../cosmos-db/how-to-setup-cmk.md#configure-your-azure-key-vault-instance)
- [Добавление политики доступа в экземпляр Azure Key Vault](../../cosmos-db/how-to-setup-cmk.md#add-access-policy)
- [Создание нового ключа в Azure Key Vault](../../cosmos-db/how-to-setup-cmk.md#generate-a-key-in-azure-key-vault)

## <a name="using-azure-portal"></a>Использование портала Azure

При создании на портале Azure учетной записи Azure API для FHIR в разделе "Параметры базы данных" на вкладке "Дополнительные параметры" можно увидеть параметр конфигурации "Шифрование данных". По умолчанию выбран параметр "Ключ, управляемый службой". 

Вы также можете выбрать свой ключ из KeyPicker:

:::image type="content" source="media/bring-your-own-key/bring-your-own-key-keypicker.png" alt-text="KeyPicker":::

Или вы можете указать ключ Azure Key Vault, выбрав параметр "Ключ, управляемый клиентом". Здесь можно ввести URI ключа:

:::image type="content" source="media/bring-your-own-key/bring-your-own-key-create.png" alt-text="Создание Azure API для FHIR":::

Для имеющихся учетных записей FHIR можно просмотреть вариант шифрования ключей (ключ, управляемый службой или клиентом) в колонке "База данных", как показано ниже. Параметр конфигурации нельзя изменить после выбора. Однако можно изменить и обновить ключ.

:::image type="content" source="media/bring-your-own-key/bring-your-own-key-database.png" alt-text="База данных":::

Кроме того, можно создать версию указанного ключа, после чего данные будут зашифрованы с помощью новой версии без прерывания работы служб. Вы также можете удалить доступ к ключу, чтобы удалить доступ к данным. Если ключ отключен, запросы будут вызывать ошибку. Если ключ снова включен, запросы будут выполняться успешно.




## <a name="using-azure-powershell"></a>Использование Azure PowerShell

Используя URI ключа Azure Key Vault, можно настроить управляемый клиентом ключ с помощью PowerShell, выполнив следующую команду PowerShell:

```powershell
New-AzHealthcareApisService
    -Name "myService"
    -Kind "fhir-R4"
    -ResourceGroupName "myResourceGroup"
    -Location "westus2"
    -CosmosKeyVaultKeyUri "https://<my-vault>.vault.azure.net/keys/<my-key>"
```

## <a name="using-azure-cli"></a>Использование Azure CLI

Как и в случае с методом с PowerShell, вы можете настроить управляемые клиентом ключи, передав URI ключа Azure Key Vault в параметре `key-vault-key-uri` и выполнив следующую команду CLI: 

```azurecli-interactive
az healthcareapis service create
    --resource-group "myResourceGroup"
    --resource-name "myResourceName"
    --kind "fhir-R4"
    --location "westus2"
    --cosmos-db-configuration key-vault-key-uri="https://<my-vault>.vault.azure.net/keys/<my-key>"

```
## <a name="using-azure-resource-manager-template"></a>Использование шаблона Azure Resource Manager

С помощью URI ключа Azure Key Vault вы можете настроить управляемый клиентом ключ, передав его в свойстве **keyVaultKeyUri** в объекте **properties**.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "services_myService_name": {
            "defaultValue": "myService",
            "type": "String"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.HealthcareApis/services",
            "apiVersion": "2020-03-30",
            "name": "[parameters('services_myService_name')]",
            "location": "westus2",
            "kind": "fhir-R4",
            "properties": {
                "accessPolicies": [],
                "cosmosDbConfiguration": {
                    "offerThroughput": 400,
                    "keyVaultKeyUri": "https://<my-vault>.vault.azure.net/keys/<my-key>"
                },
                "authenticationConfiguration": {
                    "authority": "https://login.microsoftonline.com/72f988bf-86f1-41af-91ab-2d7cd011db47",
                    "audience": "[concat('https://', parameters('services_myService_name'), '.azurehealthcareapis.com')]",
                    "smartProxyEnabled": false
                },
                "corsConfiguration": {
                    "origins": [],
                    "headers": [],
                    "methods": [],
                    "maxAge": 0,
                    "allowCredentials": false
                }
            }
        }
    ]
}
```

Вы также можете развернуть шаблон, используя следующий скрипт PowerShell:

```powershell
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$accountLocation = "West US 2"
$keyVaultKeyUri = "https://<my-vault>.vault.azure.net/keys/<my-key>"

New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile "deploy.json" `
    -accountName $accountName `
    -location $accountLocation `
    -keyVaultKeyUri $keyVaultKeyUri
```

## <a name="next-steps"></a>Следующие шаги

Из этой статьи вы узнали, как настроить управляемые клиентом ключи для хранения с помощью портала Azure, PowerShell, CLI и шаблона ARM. Если у вас есть дополнительные вопросы, ознакомьтесь с разделом вопросов и ответов по Azure Cosmos DB: 
 
>[!div class="nextstepaction"]
>[Cosmos DB: настройка ключей, управляемых клиентом](../../cosmos-db/how-to-setup-cmk.md#frequently-asked-questions)