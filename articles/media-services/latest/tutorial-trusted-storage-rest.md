---
title: Доверенное хранилище Служб мультимедиа Azure
description: Из этого руководства вы узнаете, как включить доверенное хранилище для Служб мультимедиа Azure, использовать управляемые удостоверения для доверенного хранилища и предоставить службам Azure доступ к учетной записи хранения при использовании брандмауэра или VPN.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: tutorial
ms.date: 2/8/2021
ms.openlocfilehash: ee57a743d28dcb80eeff5e2060ec9fff436e888c
ms.sourcegitcommit: edc7dc50c4f5550d9776a4c42167a872032a4151
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105963314"
---
# <a name="tutorial-media-services-trusted-storage"></a>Руководство. Доверенное хранилище Служб мультимедиа

В этом учебнике вы узнаете следующее:

> [!div class="checklist"]
> - как включить доверенное хранилище для Служб мультимедиа Azure;
> - как использовать управляемые удостоверения для доверенного хранилища;
> - как предоставить службам Azure доступ к учетной записи хранения при использовании таких средств управления доступом к сети, как брандмауэр или VPN.

С помощью API 2020-05-01, можно включить доверенное хранилище, связав управляемое удостоверение с учетной записью Служб мультимедиа.

>[!NOTE]
>Доверенное хранилище доступно только в API и в настоящее время не включено на портале Azure.

Службы мультимедиа могут автоматически получать доступ к учетной записи хранения, используя системную аутентификацию. Службы мультимедиа проверяют, находятся ли учетная запись Служб мультимедиа и учетная запись хранения в одной подписке. Кроме того, проверяется, получил ли пользователь, добавляющий связь, доступ к учетной записи хранения с использованием RBAC в Azure Resource Manager.

Но если вы хотите обеспечить защиту учетной записи хранения и включить доверенное хранилище с помощью механизма управления доступом к сети, необходима аутентификация с применением [управляемых удостоверений](concept-managed-identities.md). Так Службы мультимедиа смогут обращаться к учетной записи хранения, настроенной с ограничением брандмауэра или виртуальной сети посредством доступа к доверенному хранилищу.

## <a name="overview"></a>Обзор

> [!IMPORTANT]
> Используйте API 2020-05-01 для всех запросов к Службам мультимедиа.

Ниже приведены общие инструкции по созданию доверенного хранилища для Служб мультимедиа:

1. Создайте группу ресурсов.
1. Создайте учетную запись хранения.
1. Опрашивайте учетную запись хранения, пока она не будет готова. Когда учетная запись хранения будет готова, запрос вернет идентификатор субъекта-службы.
1. Найдите идентификатор роли *Участник данных BLOB-объектов хранилища*.
1. Вызовите поставщик авторизации и добавьте назначение роли.
1. Обновите учетную запись Служб мультимедиа для аутентификации в учетной записи хранения с помощью управляемого удостоверения.
1. Удалите ресурсы, если вы не хотите продолжать их использовать и оплачивать.

<!-- Link to storage role contributor role access differences information -->

## <a name="prerequisites"></a>Предварительные условия

Чтобы приступить к работе, необходима подписка Azure.  Если у вас еще нет подписки Azure, создайте [бесплатную пробную учетную запись](https://azure.microsoft.com/free/).

### <a name="get-your-tenant-id-and-subscription-id"></a>Получение идентификаторов клиента и подписки

Если вы не знаете, как получить идентификаторы клиента и подписки, см. [эту статью](how-to-set-azure-tenant.md), чтобы [найти идентификатор клиента](how-to-set-azure-tenant.md).

### <a name="create-a-service-principal-and-secret"></a>Создание субъекта-службы и секрета

Если вы не знаете, как создать субъект-службу и секрет, см. статью [Получение учетных данных для доступа к API Служб мультимедиа](access-api-howto.md).

## <a name="use-a-rest-client"></a>Использование клиента REST

Этот скрипт предназначен для использования с клиентом REST, например, с клиентом, доступным в расширениях Visual Studio Code.  Адаптируйте свою среду разработки.

## <a name="set-initial-variables"></a>Установка начальных переменных

Эта часть скрипта предназначена для использования в клиенте REST. Переменные можно по-разному использовать в среде разработки.

```rest
### AAD details
@tenantId = your tenant ID
@servicePrincipalId = the service principal ID
@servicePrincipalSecret = the service principal secret

### AAD resources
@armResource = https%3A%2F%2Fmanagement.core.windows.net%2F
@graphResource = https%3A%2F%2Fgraph.windows.net%2F
@storageResource = https%3A%2F%2Fstorage.azure.com%2F

### Service endpoints
@armEndpoint = management.azure.com
@graphEndpoint = graph.windows.net
@aadEndpoint = login.microsoftonline.com

### ARM details
@subscription = your subscription id
@resourceGroup = the resource group you'll be creating
@storageName = the name of the storage you'll be creating
@accountName = the name of the account you'll be creating
@resourceLocation = East US (or the location that works best for your region)
```

## <a name="get-a-token-for-azure-resource-manager"></a>Получение маркера для Azure Resource Manager

```rest
// @name getArmToken
POST https://{{aadEndpoint}}/{{tenantId}}/oauth2/token
Accept: application/json
Content-Type: application/x-www-form-urlencoded

resource={{armResource}}&client_id={{servicePrincipalId}}&client_secret={{servicePrincipalSecret}}&grant_type=client_credentials
```

## <a name="get-a-token-for-the-graph-api"></a>Получение маркера для API Graph

Эта часть скрипта предназначена для использования в клиенте REST. Переменные можно по-разному использовать в среде разработки.

```rest
// @name getGraphToken
POST https://{{aadEndpoint}}/{{tenantId}}/oauth2/token
Accept: application/json
Content-Type: application/x-www-form-urlencoded

resource={{graphResource}}&client_id={{servicePrincipalId}}&client_secret={{servicePrincipalSecret}}&grant_type=client_credentials
```

## <a name="get-the-service-principal-details"></a>Получение сведений о субъекте-службе

```rest
// @name getServicePrincipals
GET https://{{graphEndpoint}}/{{tenantId}}/servicePrincipals?$filter=appId%20eq%20'{{servicePrincipalId}}'&api-version=1.6
x-ms-client-request-id: cae3e4f7-17a0-476a-a05a-0dab934ba959
Authorization:  Bearer {{getGraphToken.response.body.access_token}}
```

## <a name="store-the-service-principal-id"></a>Хранение идентификатора субъекта-службы

```rest
@servicePrincipalObjectId = {{getServicePrincipals.response.body.value[0].objectId}}
```

## <a name="create-a-resource-group"></a>Создание группы ресурсов

```rest
// @name createResourceGroup
PUT https://{{armEndpoint}}/subscriptions/{{subscription}}/resourceGroups/{{resourceGroup}}
    ?api-version=2016-09-01
Authorization: Bearer {{getArmToken.response.body.access_token}}
Content-Type: application/json; charset=utf-8

{
    "location": "{{resourceLocation}}"
}
```

## <a name="create-storage-account"></a>Создание учетной записи хранения

```rest
// @name createStorageAccount
PUT https://{{armEndpoint}}/subscriptions/{{subscription}}/resourceGroups/{{resourceGroup}}/providers/Microsoft.Storage/storageAccounts/{{storageName}}
    ?api-version=2019-06-01
Authorization: Bearer {{getArmToken.response.body.access_token}}
Content-Type: application/json; charset=utf-8

{
    "sku": {
    "name": "Standard_GRS"
    },
    "kind": "StorageV2",
    "location": "{{resourceLocation}}",
    "properties": {
    }
}
```

## <a name="get-the-storage-account-status"></a>Получение сведений о состоянии учетной записи хранения

Подготовка учетной записи хранения занимает некоторое время, поэтому этот запрос уточняет ее состояние.  Повторяйте этот запрос, пока учетная запись хранения не будет готова.

```rest
// @name getStorageAccountStatus
GET {{createStorageAccount.response.headers.Location}}
Authorization: Bearer {{getArmToken.response.body.access_token}}
```

## <a name="get-the-storage-account-details"></a>Получение сведений об учетной записи хранения

Когда учетная запись хранения будет готова, получите сведения о ее свойствах.

```rest
// @name getStorageAccount
GET https://{{armEndpoint}}/subscriptions/{{subscription}}/resourceGroups/{{resourceGroup}}/providers/Microsoft.Storage/storageAccounts/{{storageName}}
    ?api-version=2019-06-01
Authorization: Bearer {{getArmToken.response.body.access_token}}
```

## <a name="get-a-token-for-arm"></a>Получение маркера для ARM

```rest
// @name getStorageToken
POST https://{{aadEndpoint}}/{{tenantId}}/oauth2/token
Accept: application/json
Content-Type: application/x-www-form-urlencoded

resource={{storageResource}}&client_id={{servicePrincipalId}}&client_secret={{servicePrincipalSecret}}&grant_type=client_credentials
```

## <a name="create-a-media-services-account-with-a-system-assigned-managed-identity"></a>Создание учетной записи Служб мультимедиа с управляемым удостоверением, назначенным системой

```rest
// @name createMediaServicesAccount
PUT https://{{armEndpoint}}/subscriptions/{{subscription}}/resourceGroups/{{resourceGroup}}/providers/Microsoft.Media/mediaservices/{{accountName}}?api-version=2020-05-01
Authorization: Bearer {{getArmToken.response.body.access_token}}
Content-Type: application/json; charset=utf-8

{
  "identity": {
      "type": "SystemAssigned"
  },
  "properties": {
    "storageAccounts": [
      {
        "id": "{{getStorageAccountStatus.response.body.id}}"
      }
    ],
    "encryption": {
      "type": "SystemKey"
    }
  },
  "location": "{{resourceLocation}}"
}
```

## <a name="get-the-storage-storage-blob-data-role-definition"></a>Получение определения роли данных большого двоичного объекта хранилища

```rest
// @name getStorageBlobDataContributorRoleDefinition
GET https://management.azure.com/subscriptions/{{subscription}}/resourceGroups/{{resourceGroup}}/providers/Microsoft.Storage/storageAccounts/{{storageName}}/providers/Microsoft.Authorization/roleDefinitions?$filter=roleName%20eq%20%27Storage%20Blob%20Data%20Contributor%27&api-version=2015-07-01
Authorization: Bearer {{getArmToken.response.body.access_token}}
```

### <a name="set-the-storage-role-assignment"></a>Установка назначения роли хранилища

Назначение роли говорит о том, что субъекту-службе для учетной записи Служб мультимедиа назначена роль хранилища *Участник хранилища для больших двоичных объектов*.  Это может занять некоторое время. Важно дождаться завершения процесса, иначе учетная запись Служб мультимедиа не будет правильно настроена.

```rest
PUT https://management.azure.com/subscriptions/{{subscription}}/resourceGroups/{{resourceGroup}}/providers/Microsoft.Storage/storageAccounts/{{storageName}}/providers/Microsoft.Authorization/roleAssignments/{{$guid}}?api-version=2020-04-01-preview
Authorization: Bearer {{getArmToken.response.body.access_token}}
Content-Type: application/json; charset=utf-8

{
  "properties": {
    "roleDefinitionId": "/subscriptions/{{subscription}}/resourceGroups/{{resourceGroup}}/providers/Microsoft.Storage/storageAccounts/{{storageName}}/providers/Microsoft.Authorization/roleDefinitions/{{getStorageBlobDataContributorRoleDefinition.response.body.value[0].name}}",
    "principalId": "{{createMediaServicesAccount.response.body.identity.principalId}}"
  }
}
```

## <a name="give-managed-identity-bypass-access-to-the-storage-account"></a>Предоставление управляемому удостоверению обходного доступа к учетной записи хранения

Это действие позволяет перевести доступ с использования удостоверения, управляемого системой, на использование управляемого удостоверения. Таким образом учетная запись хранения может получить доступ к учетной записи хранения через брандмауэр, так как службы Azure могут получить доступ к учетной записи хранения независимо от правил доступа по IP-адресам (ACL).

Как и ранее, дождитесь назначения роли в учетной записи хранения, иначе учетная запись Служб мультимедиа будет настроена неправильно.

```rest
// @name setStorageAccountFirewall
PUT https://{{armEndpoint}}/subscriptions/{{subscription}}/resourceGroups/{{resourceGroup}}/providers/Microsoft.Storage/storageAccounts/{{storageName}}
    ?api-version=2019-06-01
Authorization: Bearer {{getArmToken.response.body.access_token}}
Content-Type: application/json; charset=utf-8

{
    "sku": {
    "name": "Standard_GRS"
    },
    "kind": "StorageV2",
    "location": "{{resourceLocation}}",
    "properties": {
      "minimumTlsVersion": "TLS1_2",
      "networkAcls": {
        "bypass": "AzureServices",
        "virtualNetworkRules": [],
        "ipRules": [],
        "defaultAction": "Deny"
      }
    }
}
```

## <a name="update-the-media-services-account-to-use-the-managed-identity"></a>Настройка учетной записи Служб мультимедиа для использования управляемого удостоверения

Возможно, потребуется повторить этот запрос несколько раз, так как распространение назначения роли хранилища может занять некоторое время.

```rest
// @name updateMediaServicesAccountWithManagedStorageAuth
PUT https://{{armEndpoint}}/subscriptions/{{subscription}}/resourceGroups/{{resourceGroup}}/providers/Microsoft.Media/mediaservices/{{accountName}}?api-version=2020-05-01
Authorization: Bearer {{getArmToken.response.body.access_token}}
Content-Type: application/json; charset=utf-8

{
  "identity": {
      "type": "SystemAssigned"
  },
  "properties": {
    "storageAccounts": [
      {
        "id": "{{getStorageAccountStatus.response.body.id}}"
      }
    ],
    "storageAuthentication": "ManagedIdentity",
    "encryption": {
      "type": "SystemKey"
    }
  },
  "location": "{{resourceLocation}}"
}
```

## <a name="test-access"></a>Проверка доступа

Проверьте доступ, создав ресурс в учетной записи хранения.

```rest
// @name createAsset
PUT https://{{armEndpoint}}/subscriptions/{{subscription}}/resourceGroups/{{resourceGroup}}/providers/Microsoft.Media/mediaservices/{{accountName}}/assets/testasset{{index}}withoutmi?api-version=2018-07-01
Authorization: Bearer {{getArmToken.response.body.access_token}}
Content-Type: application/json; charset=utf-8

{
}
```

## <a name="delete-resources"></a>Удаление ресурсов

Если вы не хотите сохранить и продолжать оплачивать созданные ресурсы, удалите их.

```rest
### Clean up the Storage account
DELETE https://{{armEndpoint}}/subscriptions/{{subscription}}/resourceGroups/{{resourceGroup}}/providers/Microsoft.Storage/storageAccounts/{{storageName}}
    ?api-version=2019-06-01
Authorization: Bearer {{getArmToken.response.body.access_token}}

### Clean up the Media Services account
DELETE https://{{armEndpoint}}/subscriptions/{{subscription}}/resourceGroups/{{resourceGroup}}/providers/Microsoft.Media/mediaservices/{{accountName}}?api-version=2020-05-01
Authorization: Bearer {{getArmToken.response.body.access_token}}

### Clean up the Media Services account
GET https://{{armEndpoint}}/subscriptions/{{subscription}}/resourceGroups/{{resourceGroup}}/providers/Microsoft.Media/mediaservices/{{accountName}}?api-version=2020-05-01
Authorization: Bearer {{getArmToken.response.body.access_token}}
```

## <a name="next-steps"></a>Дальнейшие действия

[Как создать ресурс](asset-create-asset-how-to.md)
