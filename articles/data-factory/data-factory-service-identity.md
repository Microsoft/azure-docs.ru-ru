---
title: Управляемое удостоверение для Фабрики данных
description: Сведения об управляемом удостоверении для фабрики данных Azure.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 03/25/2021
ms.author: jingwang
ms.openlocfilehash: 65512f8e46b5545929a798392ac5f19ddeab39ed
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105562466"
---
# <a name="managed-identity-for-data-factory"></a>Управляемое удостоверение для Фабрики данных

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Эта статья поможет вам понять, что такое управляемое удостоверение для фабрики данных (прежнее название — Управляемое удостоверение службы/MSI) и как она работает.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="overview"></a>Обзор

При создании фабрики данных можно создать управляемое удостоверение вместе с созданием фабрики. Управляемое удостоверение является управляемым приложением, зарегистрированным в Azure Active Directory, и представляет эту конкретную фабрику данных.

Управляемое удостоверение для фабрики данных предоставляет следующие возможности:

- [Храните учетные данные в Azure Key Vault](store-credentials-in-key-vault.md). в этом случае для Azure Key Vault проверки подлинности используется управляемое удостоверение фабрики данных.
- Доступ к хранилищам данных или расчетам с использованием проверки подлинности управляемого удостоверения, включая хранилище BLOB-объектов Azure, обозреватель данных Azure, Azure Data Lake Storage 1-го поколения, Azure Data Lake Storage 2-го поколения, базу данных SQL Azure, Azure SQL Управляемый экземпляр, Azure синапсе Analytics, остальное, действие "действия с модулями данных", веб-действия и многое другое. Дополнительные сведения см. в статьях о соединителях и действиях.

## <a name="generate-managed-identity"></a>Создать управляемое удостоверение

Управляемое удостоверение для фабрики данных создается следующим образом:

- При создании фабрики данных с помощью **портал Azure или PowerShell** управляемое удостоверение всегда будет создаваться автоматически.
- При создании фабрики данных с помощью **пакета SDK** управляемое удостоверение создается только в том случае, если в объекте фабрики будет указано "Identity = New FactoryIdentity ()" для создания. Пример см. в инструкциях по [созданию фабрики данных из краткого руководства по .NET](quickstart-create-data-factory-dot-net.md#create-a-data-factory).
- При создании фабрики данных с помощью **REST API** управляемое удостоверение будет создано только при указании раздела Identity в тексте запроса. Пример см. в инструкциях по [созданию фабрики данных из краткого руководства по REST](quickstart-create-data-factory-rest-api.md#create-a-data-factory).

Если у фабрики данных нет управляемого удостоверения, связанного после [получения управляемой](#retrieve-managed-identity) инструкции по идентификации, вы можете явно создать ее, обновив фабрику данных с помощью инициатора удостоверений программным путем.

- [Создание управляемого удостоверения с помощью PowerShell](#generate-managed-identity-using-powershell)
- [Создание управляемого удостоверения с помощью REST API](#generate-managed-identity-using-rest-api)
- [Создание управляемого удостоверения с помощью шаблона Azure Resource Manager](#generate-managed-identity-using-an-azure-resource-manager-template)
- [Создание управляемого удостоверения с помощью пакета SDK](#generate-managed-identity-using-sdk)

>[!NOTE]
>- Невозможно изменить управляемое удостоверение. Обновление фабрики данных, у которой уже есть управляемое удостоверение, не оказывает никакого влияния, управляемое удостоверение остается неизменным.
>- Если вы обновляете фабрику данных, которая уже имеет управляемое удостоверение, не указывая параметр "Identity" в объекте фабрики или не указывая раздел "Identity" в теле запроса, возникнет ошибка.
>- При удалении фабрики данных связанное управляемое удостоверение будет удалено вместе с.

### <a name="generate-managed-identity-using-powershell"></a>Создание управляемого удостоверения с помощью PowerShell

Вызовите команду **Set-AzDataFactoryV2** , после чего отобразятся только что созданные поля Identity:

```powershell
PS C:\WINDOWS\system32> Set-AzDataFactoryV2 -ResourceGroupName <resourceGroupName> -Name <dataFactoryName> -Location <region>

DataFactoryName   : ADFV2DemoFactory
DataFactoryId     : /subscriptions/<subsID>/resourceGroups/<resourceGroupName>/providers/Microsoft.DataFactory/factories/ADFV2DemoFactory
ResourceGroupName : <resourceGroupName>
Location          : East US
Tags              : {}
Identity          : Microsoft.Azure.Management.DataFactory.Models.FactoryIdentity
ProvisioningState : Succeeded
```

### <a name="generate-managed-identity-using-rest-api"></a>Создание управляемого удостоверения с помощью REST API

Вызовите API с разделом identity в тексте запроса, как в примере ниже.

```
PATCH https://management.azure.com/subscriptions/<subsID>/resourceGroups/<resourceGroupName>/providers/Microsoft.DataFactory/factories/<data factory name>?api-version=2018-06-01
```

**Текст запроса**: add "identity": { "type": "SystemAssigned" }.

```json
{
    "name": "<dataFactoryName>",
    "location": "<region>",
    "properties": {},
    "identity": {
        "type": "SystemAssigned"
    }
}
```

**Ответ**. управляемое удостоверение создается автоматически, а раздел "Identity" заполняется соответствующим образом.

```json
{
    "name": "<dataFactoryName>",
    "tags": {},
    "properties": {
        "provisioningState": "Succeeded",
        "loggingStorageAccountKey": "**********",
        "createTime": "2017-09-26T04:10:01.1135678Z",
        "version": "2018-06-01"
    },
    "identity": {
        "type": "SystemAssigned",
        "principalId": "765ad4ab-XXXX-XXXX-XXXX-51ed985819dc",
        "tenantId": "72f988bf-XXXX-XXXX-XXXX-2d7cd011db47"
    },
    "id": "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.DataFactory/factories/ADFV2DemoFactory",
    "type": "Microsoft.DataFactory/factories",
    "location": "<region>"
}
```

### <a name="generate-managed-identity-using-an-azure-resource-manager-template"></a>Создание управляемого удостоверения с помощью шаблона Azure Resource Manager

**Шаблон:** add "identity": { "type": "SystemAssigned" }.

```json
{
    "contentVersion": "1.0.0.0",
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "resources": [{
        "name": "<dataFactoryName>",
        "apiVersion": "2018-06-01",
        "type": "Microsoft.DataFactory/factories",
        "location": "<region>",
        "identity": {
            "type": "SystemAssigned"
        }
    }]
}
```

### <a name="generate-managed-identity-using-sdk"></a>Создание управляемого удостоверения с помощью пакета SDK

Вызовите функцию create_or_update фабрики данных с Identity=new FactoryIdentity(). Пример кода с использованием .NET:

```csharp
Factory dataFactory = new Factory
{
    Location = <region>,
    Identity = new FactoryIdentity()
};
client.Factories.CreateOrUpdate(resourceGroup, dataFactoryName, dataFactory);
```

## <a name="retrieve-managed-identity"></a>Получение управляемого удостоверения

Управляемое удостоверение можно получить из портал Azure или программно. В разделах ниже приведено несколько примеров.

>[!TIP]
> Если управляемое удостоверение не отображается, [Создайте управляемое удостоверение](#generate-managed-identity) , обновив фабрику.

### <a name="retrieve-managed-identity-using-azure-portal"></a>Получение управляемого удостоверения с помощью портал Azure

Вы можете найти сведения об управляемом удостоверении из портал Azure-> свойств фабрики данных >.

- Идентификатор объекта управляемого идентификатора
- Управляемый клиент удостоверений

При создании связанной службы будут также отображаться сведения об управляемом удостоверении, поддерживающем проверку подлинности управляемого удостоверения, например, большой двоичный объект Azure, Azure Data Lake Storage, Azure Key Vault и т. д.

При предоставлении разрешения на вкладке Управление доступом (IAM) ресурса Azure — > добавить назначение ролей — > назначить доступ к-> выберите фабрика данных в разделе Назначенная системой управляемая идентификация — > выберите по имени фабрики. чтобы найти это удостоверение, в общем случае можно использовать идентификатор объекта или имя фабрики данных (в качестве управляемого имени удостоверения). Если необходимо получить идентификатор приложения управляемого удостоверения, можно использовать PowerShell.

### <a name="retrieve-managed-identity-using-powershell"></a>Получение управляемого удостоверения с помощью PowerShell

Идентификатор управляемого участника удостоверений и идентификатор клиента будут возвращены при получении определенной фабрики данных, как показано ниже. Используйте **PrincipalId** для предоставления доступа:

```powershell
PS C:\WINDOWS\system32> (Get-AzDataFactoryV2 -ResourceGroupName <resourceGroupName> -Name <dataFactoryName>).Identity

PrincipalId                          TenantId
-----------                          --------
765ad4ab-XXXX-XXXX-XXXX-51ed985819dc 72f988bf-XXXX-XXXX-XXXX-2d7cd011db47
```

Идентификатор приложения можно получить, скопировав выше ИД участника, а затем выполните следующую команду Azure Active Directory с ИДЕНТИФИКАТОРом субъекта в качестве параметра.

```powershell
PS C:\WINDOWS\system32> Get-AzADServicePrincipal -ObjectId 765ad4ab-XXXX-XXXX-XXXX-51ed985819dc

ServicePrincipalNames : {76f668b3-XXXX-XXXX-XXXX-1b3348c75e02, https://identity.azure.net/P86P8g6nt1QxfPJx22om8MOooMf/Ag0Qf/nnREppHkU=}
ApplicationId         : 76f668b3-XXXX-XXXX-XXXX-1b3348c75e02
DisplayName           : ADFV2DemoFactory
Id                    : 765ad4ab-XXXX-XXXX-XXXX-51ed985819dc
Type                  : ServicePrincipal
```

### <a name="retrieve-managed-identity-using-rest-api"></a>Получение управляемого удостоверения с помощью REST API

Идентификатор управляемого участника удостоверений и идентификатор клиента будут возвращены при получении определенной фабрики данных, как показано ниже.

Вызовите следующий API в запросе:

```
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DataFactory/factories/{factoryName}?api-version=2018-06-01
```

**Ответ**. в следующем примере вы получите ответ, как показано ниже. Раздел "Identity" заполняется соответствующим образом.

```json
{
    "name":"<dataFactoryName>",
    "identity":{
        "type":"SystemAssigned",
        "principalId":"554cff9e-XXXX-XXXX-XXXX-90c7d9ff2ead",
        "tenantId":"72f988bf-XXXX-XXXX-XXXX-2d7cd011db47"
    },
    "id":"/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.DataFactory/factories/<dataFactoryName>",
    "type":"Microsoft.DataFactory/factories",
    "properties":{
        "provisioningState":"Succeeded",
        "createTime":"2020-02-12T02:22:50.2384387Z",
        "version":"2018-06-01",
        "factoryStatistics":{
            "totalResourceCount":0,
            "maxAllowedResourceCount":0,
            "factorySizeInGbUnits":0,
            "maxAllowedFactorySizeInGbUnits":0
        }
    },
    "eTag":"\"03006b40-XXXX-XXXX-XXXX-5e43617a0000\"",
    "location":"<region>",
    "tags":{

    }
}
```

> [!TIP] 
> Чтобы получить управляемое удостоверение из шаблона ARM, добавьте раздел **Outputs** в JSON ARM:

```json
{
    "outputs":{
        "managedIdentityObjectId":{
            "type":"string",
            "value":"[reference(resourceId('Microsoft.DataFactory/factories', parameters('<dataFactoryName>')), '2018-06-01', 'Full').identity.principalId]"
        }
    }
}
```

## <a name="next-steps"></a>Дальнейшие действия
В следующих разделах представлены сведения о том, когда и как использовать управляемое удостоверение фабрики данных.

- [Хранение учетных данных в Azure Key Vault](store-credentials-in-key-vault.md)
- [Копирование данных в Azure Data Lake Storage Gen1 и из него с помощью фабрики данных Azure](connector-azure-data-lake-store.md)

Дополнительные сведения об управляемых удостоверениях для ресурсов Azure, на которых основан управляемый идентификатор фабрики данных, см. в статье [управляемые удостоверения для ресурсов Azure](../active-directory/managed-identities-azure-resources/overview.md) .