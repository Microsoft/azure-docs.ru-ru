---
title: Разработка для Azure NetApp Files с REST API с помощью PowerShell | Документация Майкрософт
description: Описывает, как приступить к работе с Azure NetApp Files REST API с помощью PowerShell.
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
ms.openlocfilehash: 2e169bb4f7be8b52657d2caf8f05643875a8348c
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102180383"
---
# <a name="develop-for-azure-netapp-files-with-rest-api-using-powershell"></a>Разработка для Azure NetApp Files с REST API с помощью PowerShell

REST API для службы Azure NetApp Files определяет операции HTTP в ресурсах, таких как учетная запись NetApp, пул емкости, тома и моментальные снимки. Эта статья поможет вам приступить к работе с Azure NetApp Files REST API с помощью PowerShell.

## <a name="azure-netapp-files-rest-api-specification"></a>Спецификация REST API Azure NetApp Files

Спецификация REST API для Azure NetApp Files опубликована с помощью [GitHub](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/netapp/resource-manager):

`https://github.com/Azure/azure-rest-api-specs/tree/master/specification/netapp/resource-manager`


## <a name="access-the-azure-netapp-files-rest-api"></a>Доступ к REST API Azure NetApp Files  

1. [Установите Azure CLI](/cli/azure/install-azure-cli), если это еще не сделано.
2. Создайте субъект-службу в Azure Active Directory (Azure AD).
   1. Убедитесь, что у вас [достаточно разрешений](../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app).

   2. Выполните следующую команду в Azure CLI:  

      ```azurepowershell
      $RBAC_SP = az ad sp create-for-rbac --name <YOURSPNAMEGOESHERE> | ConvertFrom-Json         
      ```

      Чтобы отобразить сведения о субъекте-службе, введите `$RBAC_SP` и нажмите клавишу ВВОД.

      ```output
      appId       : appID displays here
      displayName : displayName
      name        : http://SP_Name
      password    : super secret password
      tenant      : your tenant shows here
      ```
        
      Выходные данные сохраняются в объекте переменной `$RBAC_SP` . Мы будем использовать `$RBAC_SP.appId` значения, `$RBAC_SP.password` и `$RBAC_SP.tenant` .

3. Запросите маркер доступа OAuth.

    В примерах в этой статье используется PowerShell. Можно также использовать различные инструменты API, например [Postman](https://www.getpostman.com/), [Insomnia](https://insomnia.rest/) и [Paw](https://paw.cloud/).  

    Используя `$RBAC_SP` переменную, мы теперь получаем токен носителя. 
    
    ```azurepowershell
    $body = "grant_type=client_credentials&client_id=$($RBAC_SP.appId)&client_secret=$($RBAC_SP.password)&resource=https://management.azure.com/"
    $BearerToken = Invoke-RestMethod -Method Post -body $body -Uri https://login.microsoftonline.com/$($RBAC_SP.tenant)/oauth2/token
    ```
    Выходные данные предоставляют объект токена носителя. Маркер доступа можно увидеть, введя `$BearerToken.access_token` . Он имеет следующий вид.

    ```output
    eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Im5iQ3dXMTF3M1hrQi14VWFYd0tSU0xqTUhHUSIsImtpZCI6Im5iQ3dXMTF3M1hrQi14VWFYd0tSU0xqTUhHUSJ9
    ```

    Отображенный маркер действителен в течение 3600 секунд. После этого нужно запросить новый маркер. Токен сохраняется в переменной и будет использоваться на следующем шаге.

4. Создайте объект `headers`:

    ```azurepowershell  
    $headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
    $headers.Add("Authorization", "Bearer $($BearerToken.access_token)")
    $headers.Add("Content-Type", "application/json")
    ```

5. Отправьте тестовый вызов и включите маркер, чтобы проверить доступ к REST API:

    ```azurepowershell
    $SubId = (Get-AzureRmContext).Subscription.Id 
    Invoke-RestMethod -Method Get -Headers $headers -Uri https://management.azure.com/subscriptions/$SubId/providers/Microsoft.Web/sites?api-version=2019-11-01
    ```

## <a name="examples-using-the-api"></a>Примеры в работе с API  

В этой статье для запросов используется приведенный ниже URL-адрес. Этот URL-адрес указывает на корень пространства имен Azure NetApp Files. 

`https://management.azure.com/subscriptions/$SUBID/resourceGroups/$RESOURCEGROUP/providers/Microsoft.NetApp/netAppAccounts?api-version=2019-11-01`

Необходимо назначить значения переменных перед выполнением следующих примеров со своими собственными значениями. Доступ к переменным PowerShell осуществляется с помощью команды `$variablename` .
Переменные PowerShell назначаются с помощью `$variablename = “value”` .

```azurepowershell
$Region = “westus2" 
$ResourceGroup = “MYTest-RG-63" 
$ANFvnetname = “NetAppFilesVnet-63"
$ANFvnetCIDR = “10.63.64.0/18"
$ANFsubnet = “NetAppFilesSubnet-63"
$ANFsubnetCIDR = “10.63.120.0/28"
$ANFsubnetID = “/subscriptions/$SubID/resourceGroups/$ResourceGroup/providers/Microsoft.Network/virtualNetworks/$ANFvnetname/subnets/$ANFSubnet"
$ANFAccount = “TestoftheAPI"
$ANFCapacityPool = “ANFTestPool"
$ANFServicelevel = “Standard"
$ANFVolume = “ANFTestVolume"
$ANFVolumeShareName = “Share-TEST"
$ANFVolumesize = 100GB
$ANFSnapshot = “ANFTestSnapshot"
```

### <a name="put-request-examples"></a>Примеры запросов PUT

Используйте запрос PUT, чтобы создать новые объекты в Azure NetApp Files, как показано в приведенных ниже примерах. Текст запроса размещения включает в себя данные в формате JSON для изменений. Она должна быть включена в команду PowerShell в виде текста или ссылка на файл. Чтобы сослаться на текст в виде файла, сохраните пример JSON в файл и добавьте `-body (Get-Content @<filename>)` в команду PowerShell.

```azurepowershell
    #create a NetApp account  
    $body = "{ 
        `"name`": `"$ANFAccount`", 
        `"type`": `"Microsoft.NetApp/netAppAccounts`", 
        `"location`": `"$Region`", 
        `"properties`": { 
            `"name`": `"$ANFAccount`" 
        }
    } "

    $api_version = "2020-02-01"
    Invoke-RestMethod -Method 'PUT' -Headers $headers -Body $body "https://management.azure.com/subscriptions/$SubID/resourceGroups/$ResourceGroup/providers/Microsoft.NetApp/netAppAccounts/$ANFAccount`?api-version=$api_version"  
```

```azurepowershell
    #create a capacity pool  
    $body = "{
      `"location`": `"$Region`",
      `"properties`": {
        `"size`": " + 4TB + ",
        `"serviceLevel`": `"Standard`"
      }
    }"
    $api_version = "2020-02-01"
    Invoke-RestMethod -Method 'PUT' -Headers $headers -Body $body "https://management.azure.com/subscriptions/$SubID/resourceGroups/$ResourceGroup/providers/Microsoft.NetApp/netAppAccounts/$ANFAccount/capacityPools/$ANFCapacityPool`?api-version=$api_version" 
```

```azurepowershell
    #create a volume  
    $body = "{
        `"name`": `"$ANFVolume`",
        `"type`": `"Microsoft.NetApp/netAppAccounts/capacityPools/volumes`",
        `"location`": `"$Region`",
        `"properties`": {
            `"serviceLevel`": `"$ANFServicelevel`",
            `"usageThreshold`": `"$ANFVolumesize`",
            `"creationToken`": `"$ANFVolumeShareName`",
            `"snapshotId`": `"`",
            `"subnetId`": `"$ANFSubnetID`"
        }
    }"
    $api_version = "2020-02-01"
    Invoke-RestMethod -Method 'PUT' -Headers $headers -Body $body "https://management.azure.com/subscriptions/$SubID/resourceGroups/$ResourceGroup/providers/Microsoft.NetApp/netAppAccounts/$ANFAccount/capacityPools/$ANFCapacityPool/volumes/$ANFVolume`?api-version=$api_version" 
```

```azurepowershell
    #create a volume snapshot
    $body = "{
        `"name`": `"$ANFAccount/$ANFCapacityPool/$ANFVolume/$ANFSnapshot`",
        `"type`": `"Microsoft.NetApp/netAppAccounts/capacityPools/Volumes/Snapshots`",
        `"location`": `"$Region`",
        `"properties`": {
            `"name`": `"$ANFSnapshot`",
            `"fileSystemId`": `"$FileSystemID`"
        }
    }"
    $api_version = '2020-02-01'
    Invoke-RestMethod -Method 'PUT' -Headers $headers -Body $body "https://management.azure.com/subscriptions/$SubID/resourceGroups/$ResourceGroup/providers/Microsoft.NetApp/netAppAccounts/$ANFAccount/capacityPools/$ANFCapacityPool/volumes/$ANFVolume/Snapshots/$ANFSnapshot`?api-version=$api_version"
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
            "subnetId": "/subscriptions/$SUBID/resourceGroups/$RESOURCEGROUP/providers/Microsoft.Network/virtualNetworks/VNETGOESHERE/subnets/MYDELEGATEDSUBNET.sn"
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

### <a name="get-request-examples"></a>Примеры запросов GET

Если ресурс не существует, возникает ошибка. Используйте запрос GET, чтобы запросить объекты Azure NetApp Files в подписке, как показано в приведенных ниже примерах:

```azurepowershell
#get NetApp accounts 
Invoke-RestMethod -Method Get -Headers $headers -Uri https://management.azure.com/subscriptions/$SUBID/resourceGroups/$ResourceGroup/providers/Microsoft.NetApp/netAppAccounts?api-version=2019-11-01 | ConvertTo-Json
```

```azurepowershell
#get capacity pools for NetApp account 
Invoke-RestMethod -Method Get -Headers $headers -Uri https://management.azure.com/subscriptions/$SUBID/resourceGroups/$ResourceGroup/providers/Microsoft.NetApp/netAppAccounts/$ANFACCOUNT/capacityPools?api-version=2019-11-01 | ConvertTo-Json
```

```azurepowershell
#get volumes in NetApp account & capacity pool 
Invoke-RestMethod -Method Get -Headers $headers -Uri https://management.azure.com/subscriptions/$SUBID/resourceGroups/$ResourceGroup/providers/Microsoft.NetApp/netAppAccounts/$ANFACCOUNT/capacityPools/$ANFCAPACITYPOOL/volumes?api-version=2019-11-01 | ConvertTo-Json
```

```azurepowershell
#get snapshots for a volume 
Invoke-RestMethod -Method Get -Headers $headers -Uri https://management.azure.com/subscriptions/$SUBID/resourceGroups/$ResourceGroup/providers/Microsoft.NetApp/netAppAccounts/$ANFACCOUNT/capacityPools/$ANFCAPACITYPOOL/volumes/$ANFVOLUME/snapshots?api-version=2019-11-01 | ConvertTo-Json
```

### <a name="complete-powershell-scripts"></a>Завершение сценариев PowerShell
В этом разделе приведены примеры сценариев для PowerShell.

```azurepowershell
    <#
    Disclaimer 
    The sample scripts are not supported under any Microsoft standard support program or service. The sample scripts are provided AS IS without warranty of any kind. Microsoft further disclaims all implied warranties including, without limitation, any implied warranties of merchantability or of fitness for a particular purpose. The entire risk arising out of the use or performance of the sample scripts and documentation remains with you. In no event shall Microsoft, its authors, or anyone else involved in the creation, production, or delivery of the scripts be liable for any damages whatsoever (including, without limitation, damages for loss of business profits, business interruption, loss of business information, or other pecuniary loss) arising out of the use of or inability to use the sample scripts or documentation, even if Microsoft has been advised of the possibility of such damages.
    #>
    
    $Region = "westus2"
    $SubID = (Get-AzureRmContext).Subscription.Id
    $MyRandomID =  Get-Random -Minimum 100 -Maximum 999
    $ResourceGroup = "MYTest-RG-" + $MyRandomID
    $ANFAccount = "ANFTestAccount-$Region-" + $MyRandomID
    $ANFCapacityPool =  "ANFTestPool-$Region-" + $MyRandomID
    $ANFVolume = "ANFTestVolume-$Region-" + $MyRandomID
    $ANFVolumesize = 100GB
    $ANFVolumeShareName = "Share-" + $MyRandomID
    $ANFSnapshot = "ANFTestSnapshot-$Region-" + $MyRandomID
    $ANFServicelevel = "Standard"
    $Octet2 = Get-Random -Minimum 1 -Maximum 254
    $ANFvnetname = "NetAppFilesVnet-$Octet2-$Region-" + $MyRandomID
    $ANFvnetCIDR = "10.$Octet2.64.0/18"
    $ANFsubnet = "NetAppFilesSubnet-$Octet2-$Region-" + $MyRandomID
    $ANFsubnetCIDR = "10.$Octet2.120.0/28"
    $vmsubnet = "VM-subnet-$Octet2-$Region-" + $MyRandomID
    $vmsubnetCIDR = "10.$Octet2.121.0/24"
    
    $BearerToken = (az account get-access-token | ConvertFrom-Json).accessToken
    
    $headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
    $headers.Add("Authorization", "Bearer $BearerToken")
    $headers.Add("Content-Type", "application/json")
    
    " ** Creating Resource Group $ResourceGroup *************"
    $api_version = "2020-01-01"
    $body = "    {
      `"location`": `"$Region`",
      `"name`": `"$ResourceGroup`"
    },"
    $response = Invoke-RestMethod -Method 'PUT' -Headers $headers -Body $body -Uri "https://management.azure.com/subscriptions/$SubID/resourcegroups/$ResourceGroup`?api-version=$api_version" 
    While ($response.properties.provisioningState -notmatch "Succeeded") {
       $response.properties.provisioningState
       sleep 5
       $response = Invoke-RestMethod -Method ‘GET’ -Headers $headers -Uri "https://management.azure.com/subscriptions/$SubID/resourceGroups/$ResourceGroup`?api-version=$api_version" 
       } 
    
    " ** Creating Virtual Network $ANFvnetname *************"
    $body = "{
      `"properties`": {
        `"addressSpace`": {
          `"addressPrefixes`": [
            `"$ANFvnetCIDR`"
          ]
        },
        `"subnets`": [
          {
            `"name`": `"$ANFsubnet`",
            `"properties`": {
              `"addressPrefix`": `"$ANFsubnetCIDR`",
              `"delegations`": [
                {
                  `"name`": `"Microsoft.NetApp`",
                  `"properties`": {
                    `"serviceName`": `"Microsoft.NetApp/volumes`"
                  }
                }
              ]
            }
          },
         {
            `"name`": `"$vmsubnet`",
            `"properties`": {
              `"addressPrefix`": `"$vmsubnetCIDR`"
                }
         }
        ]
      },
      `"location`": `"$Region`"
    }"
    
    $api_version = "2020-03-01"
    $response = Invoke-RestMethod -Method 'PUT' -Headers $headers -Body $body -Uri "https://management.azure.com/subscriptions/$SubID/resourceGroups/$ResourceGroup/providers/Microsoft.Network/virtualNetworks/$ANFvnetname`?api-version=$api_version" 
    While ($response.properties.provisioningState -notmatch "Succeeded") {
           sleep 5
           $response = Invoke-RestMethod -Method ‘GET’ -Headers $headers -Uri "https://management.azure.com/subscriptions/$SubID/resourceGroups/$ResourceGroup/providers/Microsoft.Network/virtualNetworks/$ANFvnetname`?api-version=$api_version" 
           $response.properties.provisioningState
           }  
    $ANFsubnetID = $response.properties.subnets.id[0]
    
    " ** Creating ANF Account $ANFAccount *************"
    $body = "{ 
        `"name`": `"$ANFAccount`", 
        `"type`": `"Microsoft.NetApp/netAppAccounts`", 
        `"location`": `"$Region`", 
        `"properties`": { 
            `"name`": `"$ANFAccount`" 
        }
    } "
    $api_version = "2020-02-01"
    $response = Invoke-RestMethod -Method 'PUT' -Headers $headers -Body $body -Uri "https://management.azure.com/subscriptions/$SubID/resourceGroups/$ResourceGroup/providers/Microsoft.NetApp/netAppAccounts/$ANFAccount`?api-version=$api_version" 
    $response.properties.provisioningState
    While ($response.properties.provisioningState -notmatch "Succeeded") {
       $response.properties.provisioningState
       sleep 5
       $response = Invoke-RestMethod -Method ‘GET’ -Headers $headers -Uri "https://management.azure.com/subscriptions/$SubID/resourceGroups/$ResourceGroup/providers/Microsoft.NetApp/netAppAccounts/$ANFAccount`?api-version=$api_version" 
       }  
    
    " ** Creating Capacity Pool $ANFCapacityPool *************"
    $body = "{
      `"location`": `"$Region`",
      `"properties`": {
        `"size`": " + 4TB + ",
        `"serviceLevel`": `"Standard`"
      }
    }"
    $api_version = "2020-02-01"
    $response = Invoke-RestMethod -Method 'PUT' -Headers $headers -Body $body -Uri "https://management.azure.com/subscriptions/$SubID/resourceGroups/$ResourceGroup/providers/Microsoft.NetApp/netAppAccounts/$ANFAccount/capacityPools/$ANFCapacityPool`?api-version=$api_version" 
    $response.properties.provisioningState
    While ($response.properties.provisioningState -notmatch "Succeeded") {
       $response.properties.provisioningState
       sleep 5
       $response = Invoke-RestMethod -Method ‘GET’ -Headers $headers -Uri "https://management.azure.com/subscriptions/$SubID/resourceGroups/$ResourceGroup/providers/Microsoft.NetApp/netAppAccounts/$ANFAccount`?api-version=$api_version" 
       }  
    
    " ** Creating ANF Volume $ANFVolume *************"
    $body = "{
        `"name`": `"$ANFVolume`",
        `"type`": `"Microsoft.NetApp/netAppAccounts/capacityPools/volumes`",
        `"location`": `"$Region`",
        `"properties`": {
            `"serviceLevel`": `"$ANFServicelevel`",
            `"usageThreshold`": `"$ANFVolumesize`",
            `"creationToken`": `"$ANFVolumeShareName`",
            `"snapshotId`": `"`",
            `"subnetId`": `"$ANFSubnetID`"
        }
    }"
    $api_version = "2020-02-01"
    $response = Invoke-RestMethod -Method 'PUT' -Headers $headers -Body $body -Uri "https://management.azure.com/subscriptions/$SubID/resourceGroups/$ResourceGroup/providers/Microsoft.NetApp/netAppAccounts/$ANFAccount/capacityPools/$ANFCapacityPool/volumes/$ANFVolume`?api-version=$api_version" 
    While ($response.properties.provisioningState -notmatch "Succeeded") {
       $response.properties.provisioningState
       sleep 15
       $response = Invoke-RestMethod -Method ‘GET’ -Headers $headers -Uri "https://management.azure.com/subscriptions/$SubID/resourceGroups/$ResourceGroup/providers/Microsoft.NetApp/netAppAccounts/$ANFAccount/capacityPools/$ANFCapacityPool/volumes/$ANFVolume`?api-version=$api_version" 
       } 
    $Volume = $response 
    $FileSystemID = $response.properties.fileSystemId
    
    " ** Creating ANF Volume Snapshot $ANFSnapshot *************"
    $body = "{
        `"name`": `"$ANFAccount/$ANFCapacityPool/$ANFVolume/$ANFSnapshot`",
        `"type`": `"Microsoft.NetApp/netAppAccounts/capacityPools/Volumes/Snapshots`",
        `"location`": `"$Region`",
        `"properties`": {
            `"name`": `"$ANFSnapshot`",
            `"fileSystemId`": `"$FileSystemID`"
        }
    }"
    $api_version = '2020-02-01'
    $response = Invoke-RestMethod -Method 'PUT' -Headers $headers -Body $body -Uri "https://management.azure.com/subscriptions/$SubID/resourceGroups/$ResourceGroup/providers/Microsoft.NetApp/netAppAccounts/$ANFAccount/capacityPools/$ANFCapacityPool/volumes/$ANFVolume/Snapshots/$ANFSnapshot`?api-version=$api_version" 
    While ($response.properties.provisioningState -notmatch "Succeeded") {
       $response.properties.provisioningState
       sleep 5
       $response = Invoke-RestMethod -Method ‘GET’ -Headers $headers -Uri "https://management.azure.com/subscriptions/$SubID/resourceGroups/$ResourceGroup/providers/Microsoft.NetApp/netAppAccounts/$ANFAccount/capacityPools/$ANFCapacityPool/volumes/$ANFVolume/Snapshots/$ANFSnapshot`?api-version=$api_version" 
       }  
```

## <a name="next-steps"></a>Дальнейшие действия

[См. статью о REST API Azure NetApp Files](/rest/api/netapp/)