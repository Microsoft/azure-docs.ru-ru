---
title: Разработка для Azure NetApp Files с помощью REST API | Документация Майкрософт
description: REST API для службы Azure NetApp Files определяет операции HTTP для таких ресурсов, как учетная запись NetApp, пул емкости, тома и моментальные снимки.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 06/02/2020
ms.author: b-juche
ms.openlocfilehash: c5993dc1dc645319e272ab310a97bc3ff8ac495d
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102174246"
---
# <a name="develop-for-azure-netapp-files-with-rest-api"></a>Разработка для Azure NetApp Files с помощью REST API 

REST API для службы Azure NetApp Files определяет операции HTTP в ресурсах, таких как учетная запись NetApp, пул емкости, тома и моментальные снимки. В этой статье описано, как приступить к работе с помощью REST API Azure NetApp Files.

## <a name="azure-netapp-files-rest-api-specification"></a>Спецификация REST API Azure NetApp Files

Спецификация REST API для Azure NetApp Files опубликована с помощью [GitHub](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/netapp/resource-manager):

`https://github.com/Azure/azure-rest-api-specs/tree/master/specification/netapp/resource-manager`


## <a name="access-the-azure-netapp-files-rest-api"></a>Доступ к REST API Azure NetApp Files  

1. [Установите Azure CLI](/cli/azure/install-azure-cli), если это еще не сделано.
2. Создайте субъект-службу в Azure Active Directory (Azure AD).
   1. Убедитесь, что у вас [достаточно разрешений](../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app).

   2. Выполните следующую команду в Azure CLI: 
    
        ```azurecli
        az ad sp create-for-rbac --name $YOURSPNAMEGOESHERE
        ```

      Результат команды должен быть примерно таким:  

        ```output
        { 
            "appId": "appIDgoeshere", 
            "displayName": "APPNAME", 
            "name": "http://APPNAME", 
            "password": "supersecretpassword", 
            "tenant": "tenantIDgoeshere" 
        } 
        ```

      Сохраните результат команды.  Вам понадобятся значения `appId`, `password` и `tenant`. 

3. Запросите маркер доступа OAuth.

    В примерах этой статьи используется cURL. Можно также использовать различные инструменты API, например [Postman](https://www.getpostman.com/), [Insomnia](https://insomnia.rest/) и [Paw](https://paw.cloud/).  

    Замените переменные в следующем примере результатами команды из шага 2 выше. 
    
    ```azurecli
    curl -X POST -d 'grant_type=client_credentials&client_id=[APP_ID]&client_secret=[PASSWORD]&resource=https%3A%2F%2Fmanagement.azure.com%2F' https://login.microsoftonline.com/[TENANT_ID]/oauth2/token
    ```

    Результат будет содержать примерно такой маркер доступа:

    `eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Im5iQ3dXMTF3M1hrQi14VWFYd0tSU0xqTUhHUSIsImtpZCI6Im5iQ3dXMTF3M1hrQi14VWFYd0tSU0xqTUhHUSJ9`

    Отображенный маркер действителен в течение 3600 секунд. После этого нужно запросить новый маркер. 
    Сохраните маркер в текстовый редактор.  Он понадобится вам на следующем шаге.

4. Отправьте тестовый вызов и включите маркер, чтобы проверить доступ к REST API:

    ```azurecli
    curl -X GET -H "Authorization: Bearer [TOKEN]" -H "Content-Type: application/json" https://management.azure.com/subscriptions/[SUBSCRIPTION_ID]/providers/Microsoft.Web/sites?api-version=2019-11-01
    ```

## <a name="examples-using-the-api"></a>Примеры в работе с API  

В этой статье для запросов используется приведенный ниже URL-адрес. Этот URL-адрес указывает на корень пространства имен Azure NetApp Files. 

`https://management.azure.com/subscriptions/SUBIDGOESHERE/resourceGroups/RESOURCEGROUPGOESHERE/providers/Microsoft.NetApp/netAppAccounts?api-version=2019-11-01`

В следующих примерах замените значения `SUBIDGOESHERE` и `RESOURCEGROUPGOESHERE` собственными значениями. 

### <a name="get-request-examples"></a>Примеры запросов GET

Используйте запрос GET, чтобы запросить объекты Azure NetApp Files в подписке, как показано в приведенных ниже примерах: 

```azurecli
#get NetApp accounts 
curl -X GET -H "Authorization: Bearer TOKENGOESHERE" -H "Content-Type: application/json" https://management.azure.com/subscriptions/SUBIDGOESHERE/resourceGroups/RESOURCEGROUPGOESHERE/providers/Microsoft.NetApp/netAppAccounts?api-version=2019-11-01
```

```azurecli
#get capacity pools for NetApp account 
curl -X GET -H "Authorization: Bearer TOKENGOESHERE" -H "Content-Type: application/json" https://management.azure.com/subscriptions/SUBIDGOESHERE/resourceGroups/RESOURCEGROUPGOESHERE/providers/Microsoft.NetApp/netAppAccounts/NETAPPACCOUNTGOESHERE/capacityPools?api-version=2019-11-01
```

```azurecli
#get volumes in NetApp account & capacity pool 
curl -X GET -H "Authorization: Bearer TOKENGOESHERE" -H "Content-Type: application/json" https://management.azure.com/subscriptions/SUBIDGOESHERE/resourceGroups/RESOURCEGROUPGOESHERE/providers/Microsoft.NetApp/netAppAccounts/NETAPPACCOUNTGOESHERE/capacityPools/CAPACITYPOOLGOESHERE/volumes?api-version=2019-11-01
```

```azurecli
#get snapshots for a volume 
curl -X GET -H "Authorization: Bearer TOKENGOESHERE" -H "Content-Type: application/json" https://management.azure.com/subscriptions/SUBIDGOESHERE/resourceGroups/RESOURCEGROUPGOESHERE/providers/Microsoft.NetApp/netAppAccounts/NETAPPACCOUNTGOESHERE/capacityPools/CAPACITYPOOLGOESHERE/volumes/VOLUMEGOESHERE/snapshots?api-version=2019-11-01
```

### <a name="put-request-examples"></a>Примеры запросов PUT

Используйте запрос PUT, чтобы создать новые объекты в Azure NetApp Files, как показано в приведенных ниже примерах. Тело запроса размещения может включать в себя данные в формате JSON для изменений. Она должна быть включена в команду в виде текста или ссылок в виде файла. Чтобы сослаться на текст как на файл, сохраните пример JSON в файл и добавьте `-d @<filename>` команду в фигурную скобку.

```azurecli
#create a NetApp account  
curl -d @<filename> -X PUT -H "Authorization: Bearer TOKENGOESHERE" -H "Content-Type: application/json" https://management.azure.com/subscriptions/SUBIDGOESHERE/resourceGroups/RESOURCEGROUPGOESHERE/providers/Microsoft.NetApp/netAppAccounts/NETAPPACCOUNTGOESHERE?api-version=2019-11-01
```

```azurecli
#create a capacity pool  
curl -d @<filename> -X PUT -H "Authorization: Bearer TOKENGOESHERE" -H "Content-Type: application/json" https://management.azure.com/subscriptions/SUBIDGOESHERE/resourceGroups/RESOURCEGROUPGOESHERE/providers/Microsoft.NetApp/netAppAccounts/NETAPPACCOUNTGOESHERE/capacityPools/CAPACITYPOOLGOESHERE?api-version=2019-11-01
```

```azurecli
#create a volume  
curl -d @<filename> -X PUT -H "Authorization: Bearer TOKENGOESHERE" -H "Content-Type: application/json" https://management.azure.com/subscriptions/SUBIDGOESHERE/resourceGroups/RESOURCEGROUPGOESHERE/providers/Microsoft.NetApp/netAppAccounts/NETAPPACCOUNTGOESHERE/capacityPools/CAPACITYPOOLGOESHERE/volumes/MYNEWVOLUME?api-version=2019-11-01
```

```azurecli
 #create a volume snapshot  
curl -d @<filename> -X PUT -H "Authorization: Bearer TOKENGOESHERE" -H "Content-Type: application/json" https://management.azure.com/subscriptions/SUBIDGOESHERE/resourceGroups/RESOURCEGROUPGOESHERE/providers/Microsoft.NetApp/netAppAccounts/NETAPPACCOUNTGOESHERE/capacityPools/CAPACITYPOOLGOESHERE/volumes/MYNEWVOLUME/Snapshots/SNAPNAME?api-version=2019-11-01
```

### <a name="json-examples"></a>Примеры JSON

В приведенном ниже примере показано, как создать учетную запись NetApp.

```json
{ 
    "name": "MYNETAPPACCOUNT", 
    "type": "Microsoft.NetApp/netAppAccounts", 
    "location": "westus2", 
    "properties": { 
        "name": "MYNETAPPACCOUNT" 
    }
} 
```

В приведенном ниже примере показано, как создать пул емкости. 

```json
{
    "name": "MYNETAPPACCOUNT/POOLNAME",
    "type": "Microsoft.NetApp/netAppAccounts/capacityPools",
    "location": "westus2",
    "properties": {
        "name": "POOLNAME",
        "size": "4398046511104",
        "serviceLevel": "Premium"
    }
}
```

В следующем примере показано, как создать новый том. (Протоколом по умолчанию для тома является NFSV3.) 

```json
{
    "name": "MYNEWVOLUME",
    "type": "Microsoft.NetApp/netAppAccounts/capacityPools/volumes",
    "location": "westus2",
    "properties": {
        "serviceLevel": "Premium",
        "usageThreshold": "322122547200",
        "creationToken": "MY-FILEPATH",
        "snapshotId": "",
        "subnetId": "/subscriptions/SUBIDGOESHERE/resourceGroups/RESOURCEGROUPGOESHERE/providers/Microsoft.Network/virtualNetworks/VNETGOESHERE/subnets/MYDELEGATEDSUBNET.sn"
         }
}
```

В приведенном ниже примере показано, как сделать моментальный снимок тома. 

```json
{
    "name": "apitest2/apiPool01/apiVol01/snap02",
    "type": "Microsoft.NetApp/netAppAccounts/capacityPools/Volumes/Snapshots",
    "location": "westus2",
    "properties": {
         "name": "snap02",
        "fileSystemId": "0168704a-bbec-da81-2c29-503825fe7420"
    }
}
```

> [!NOTE] 
> Для создания моментального снимка необходимо указать `fileSystemId`.  Вы можете получить значение `fileSystemId` с помощью запроса GET к тому. 

## <a name="next-steps"></a>Дальнейшие действия

[См. статью о REST API Azure NetApp Files](/rest/api/netapp/)