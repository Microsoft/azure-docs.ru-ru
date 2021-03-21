---
title: Настройка Приватного канала Azure для учетной записи Azure Cosmos
description: Узнайте, как настроить Приватный канал Azure для доступа к учетной записи Azure Cosmos с помощью частного IP-адреса в виртуальной сети.
author: ThomasWeiss
ms.service: cosmos-db
ms.topic: how-to
ms.date: 03/02/2021
ms.author: thweiss
ms.custom: devx-track-azurecli
ms.openlocfilehash: d21943c90e1f77bd4a43cdfd27b183df018f6cc7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101690674"
---
# <a name="configure-azure-private-link-for-an-azure-cosmos-account"></a>Настройка Приватного канала Azure для учетной записи Azure Cosmos
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Приватный канал Azure позволяет подключиться к учетной записи Azure Cosmos с помощью частной конечной точки. Частная конечная точка — это набор частных IP-адресов в подсети в виртуальной сети. Затем можно ограничить доступ к учетной записи Azure Cosmos только через частные IP-адреса. Если Приватный канал используется совместно с ограниченными политиками NSG, это снижает риск утечки данных. Дополнительные сведения о частных конечных точках см. в статье [Приватный канал Azure](../private-link/private-link-overview.md).

> [!NOTE]
> Частная ссылка не мешает разрешать конечные точки Cosmos Azure в общедоступной службе DNS. Фильтрация входящих запросов происходит на уровне приложения, а не на уровне транспорта или сети.

Приватный канал позволяет пользователям получить доступ к учетной записи Azure Cosmos из виртуальной сети или из любой одноранговой виртуальной сети. Ресурсы, сопоставленные с Приватным каналом, также доступны локально по частному пирингу через VPN или Azure ExpressRoute.

Вы можете подключиться к учетной записи Azure Cosmos, настроенной с помощью Приватного канала, используя метод автоматического или ручного утверждения. Дополнительные сведения см. в разделе [Рабочий процесс утверждения](../private-link/private-endpoint-overview.md#access-to-a-private-link-resource-using-approval-workflow) в документации по Приватному каналу.

В этой статье описывается, как настроить частные конечные точки для хранилища транзакций Azure Cosmos DB. Предполагается, что вы используете метод автоматического утверждения. Если вы используете аналитическое хранилище, см. статью [частные конечные точки в статье по аналитическому хранилищу](analytical-store-private-endpoints.md) .

## <a name="create-a-private-endpoint-by-using-the-azure-portal"></a>Создание частной конечной точки с помощью портала Azure

Чтобы с помощью портала Azure создать частную конечную точку для существующей учетной записи Azure Cosmos, выполните следующие действия.

1. В области **Все ресурсы** выберите учетную запись Azure Cosmos.

1. В списке параметров выберите **Подключения к частной конечной точке**, а затем выберите **Частная конечная точка**.

   :::image type="content" source="./media/how-to-configure-private-endpoints/create-private-endpoint-portal.png" alt-text="Выбор параметров для создания частной конечной точки на портале Azure":::

1. В области **Создание частной конечной точки — Основные** введите или выберите следующие параметры.

    | Параметр | Значение |
    | ------- | ----- |
    | **Сведения о проекте** | |
    | Подписка | Выберите свою подписку. |
    | Группа ресурсов | Выберите группу ресурсов.|
    | **Сведения об экземпляре** |  |
    | Имя | Введите любое имя для частной конечной точки. Если это имя уже используется, создайте уникальное имя. |
    |Регион| Выберите регион, в котором требуется развернуть Приватный канал. Создайте частную конечную точку в том же расположении, где находится виртуальная сеть.|
    |||
1. По завершении выберите **Next: Ресурс**.
1. В окне **Создание частной конечной точки — Ресурс** введите или выберите следующую информацию:

    | Параметр | Значение |
    | ------- | ----- |
    |Метод подключения  | Выберите **Подключиться к ресурсу Azure в моем каталоге**. <br/><br/> Затем можно выбрать один из ресурсов для настройки Приватного канала. Или можно подключиться к чужому ресурсу, используя предоставленный вам ИД или псевдоним ресурса.|
    | Подписка| Выберите свою подписку. |
    | Тип ресурса | Выберите **Microsoft.AzureCosmosDB/databaseAccounts**. |
    | Ресурс |Выберите свою учетную запись Azure Cosmos. |
    |Целевой подресурс |Выберите тип API Azure Cosmos DB, который нужно сопоставить. По умолчанию имеется только один вариант для API-интерфейсов SQL, MongoDB и Cassandra. Для API-интерфейсов Gremlin и Table можно также выбрать **SQL**, так как они совместимы с API SQL. |
    |||

1. По завершении выберите **Next: Конфигурация**.
1. В окне **Создание частной конечной точки — Конфигурация** введите или выберите следующую информацию.

    | Параметр | Значение |
    | ------- | ----- |
    |**Сеть**| |
    | Виртуальная сеть| Выберите свою виртуальную сеть. |
    | Подсеть | Выберите подсеть. |
    |**Интеграция с частной зоной DNS**||
    |Интеграция с частной зоной DNS |Выберите **Да**. <br><br/> Для частного подключения к частной конечной точке требуется запись DNS. Рекомендуем интегрировать частную конечную точку с частной зоной DNS. Вы также можете использовать собственные DNS-серверы или создать записи DNS с использованием файлов узлов на своих виртуальных машинах. |
    |Частная зона DNS |Выберите **privatelink.documents.azure.com**. <br><br/> Частная зона DNS определяется автоматически. Ее нельзя изменить на портале Azure.|
    |||

1. Выберите **Review + create** (Просмотреть и создать). На странице **Просмотр и создание** Azure проверит вашу конфигурацию.
1. При появлении сообщения **Проверка пройдена** нажмите кнопку **Создать**.

После утверждения Приватного канала для учетной записи Azure Cosmos на портале Azure параметр **Все сети** в области **Брандмауэр и виртуальные сети** будет недоступен.

В следующей таблице показано сопоставление между различными типами API учетной записи Azure Cosmos, поддерживаемыми подресурсами, а также соответствующими именами частных зон. Вы также можете получить доступ к учетным записям API-интерфейсов Gremlin и Table через API SQL, поэтому для них существует две записи.

|Тип API-интерфейса учетной записи Azure Cosmos  |Поддерживаемые подресурсы (или идентификаторы групп) |Имя частной зоны  |
|---------|---------|---------|
|SQL    |   SQL      | privatelink.documents.azure.com   |
|Cassandra    | Cassandra        |  privatelink.cassandra.cosmos.azure.com    |
|Mongo   |  MongoDB       |  privatelink.mongo.cosmos.azure.com    |
|Gremlin     | Gremlin        |  privatelink.gremlin.cosmos.azure.com   |
|Gremlin     |  SQL       |  privatelink.documents.azure.com    |
|Таблица    |    Таблица     |   privatelink.table.cosmos.azure.com    |
|Таблица     |   SQL      |  privatelink.documents.azure.com    |

### <a name="fetch-the-private-ip-addresses"></a>Получение частных IP-адресов

После подготовки частной конечной точки можно запросить IP-адреса. Чтобы просмотреть IP-адреса на портале Azure, выполните следующие действия.

1. Щелкните **Все ресурсы**.
1. Найдите частную конечную точку, созданную ранее. В данном случае это **cdbPrivateEndpoint3**.
1. Перейдите на вкладку **Обзор**, чтобы просмотреть параметры DNS и IP-адреса.

:::image type="content" source="./media/how-to-configure-private-endpoints/private-ip-addresses-portal.png" alt-text="Частные IP-адреса на портале Azure":::

Для каждой частной конечной точки создается несколько IP-адресов.

* Один для глобальной (независимой от региона) конечной точки учетной записи Azure Cosmos.
* По одному для каждого региона, в котором развернута учетная запись Azure Cosmos.

## <a name="create-a-private-endpoint-by-using-azure-powershell"></a>Создание частной конечной точки с помощью Azure PowerShell

Выполните следующий сценарий PowerShell, чтобы создать частную конечную точку с именем "MyPrivateEndpoint" для существующей учетной записи Azure Cosmos. Замените значения переменных на значения для вашей среды.

```azurepowershell-interactive
$SubscriptionId = "<your Azure subscription ID>"
# Resource group where the Azure Cosmos account and virtual network resources are located
$ResourceGroupName = "myResourceGroup"
# Name of the Azure Cosmos account
$CosmosDbAccountName = "mycosmosaccount"

# API type of the Azure Cosmos account: Sql, MongoDB, Cassandra, Gremlin, or Table
$CosmosDbApiType = "Sql"
# Name of the existing virtual network
$VNetName = "myVnet"
# Name of the target subnet in the virtual network
$SubnetName = "mySubnet"
# Name of the private endpoint to create
$PrivateEndpointName = "MyPrivateEndpoint"
# Location where the private endpoint can be created. The private endpoint should be created in the same location where your subnet or the virtual network exists
$Location = "westcentralus"

$cosmosDbResourceId = "/subscriptions/$($SubscriptionId)/resourceGroups/$($ResourceGroupName)/providers/Microsoft.DocumentDB/databaseAccounts/$($CosmosDbAccountName)"

$privateEndpointConnection = New-AzPrivateLinkServiceConnection -Name "myConnectionPS" -PrivateLinkServiceId $cosmosDbResourceId -GroupId $CosmosDbApiType
 
$virtualNetwork = Get-AzVirtualNetwork -ResourceGroupName  $ResourceGroupName -Name $VNetName  
 
$subnet = $virtualNetwork | Select -ExpandProperty subnets | Where-Object  {$_.Name -eq $SubnetName}  
 
$privateEndpoint = New-AzPrivateEndpoint -ResourceGroupName $ResourceGroupName -Name $PrivateEndpointName -Location "westcentralus" -Subnet  $subnet -PrivateLinkServiceConnection $privateEndpointConnection
```

### <a name="integrate-the-private-endpoint-with-a-private-dns-zone"></a>Интеграция частной конечной точки с частной зоной DNS

После создания частной конечной точки ее можно интегрировать с частной зоной DNS с помощью следующего сценария PowerShell.

```azurepowershell-interactive
Import-Module Az.PrivateDns
$zoneName = "privatelink.documents.azure.com"
$zone = New-AzPrivateDnsZone -ResourceGroupName $ResourceGroupName `
  -Name $zoneName

$link  = New-AzPrivateDnsVirtualNetworkLink -ResourceGroupName $ResourceGroupName `
  -ZoneName $zoneName `
  -Name "myzonelink" `
  -VirtualNetworkId $virtualNetwork.Id  
 
$pe = Get-AzPrivateEndpoint -Name $PrivateEndpointName `
  -ResourceGroupName $ResourceGroupName

$networkInterface = Get-AzResource -ResourceId $pe.NetworkInterfaces[0].Id `
  -ApiVersion "2019-04-01"
 
foreach ($ipconfig in $networkInterface.properties.ipConfigurations) { 
foreach ($fqdn in $ipconfig.properties.privateLinkConnectionProperties.fqdns) { 
Write-Host "$($ipconfig.properties.privateIPAddress) $($fqdn)"  
$recordName = $fqdn.split('.',2)[0] 
$dnsZone = $fqdn.split('.',2)[1] 
New-AzPrivateDnsRecordSet -Name $recordName `
  -RecordType A -ZoneName $zoneName  `
  -ResourceGroupName $ResourceGroupName -Ttl 600 `
  -PrivateDnsRecords (New-AzPrivateDnsRecordConfig `
  -IPv4Address $ipconfig.properties.privateIPAddress)  
}
}
```

### <a name="fetch-the-private-ip-addresses"></a>Получение частных IP-адресов

После подготовки частной конечной точки можно запросить IP-адреса и сопоставление полного доменного имени с помощью следующего сценария PowerShell.

```azurepowershell-interactive
$pe = Get-AzPrivateEndpoint -Name MyPrivateEndpoint -ResourceGroupName myResourceGroup
$networkInterface = Get-AzNetworkInterface -ResourceId $pe.NetworkInterfaces[0].Id
foreach ($IPConfiguration in $networkInterface.IpConfigurations)
{
    Write-Host $IPConfiguration.PrivateIpAddress ":" $IPConfiguration.PrivateLinkConnectionProperties.Fqdns
}
```

## <a name="create-a-private-endpoint-by-using-azure-cli"></a>Создание частной конечной точки с помощью Azure CLI

Выполните следующий сценарий Azure CLI, чтобы создать частную конечную точку с именем "myPrivateEndpoint" для существующей учетной записи Azure Cosmos. Замените значения переменных на значения для вашей среды.

```azurecli-interactive
# Resource group where the Azure Cosmos account and virtual network resources are located
ResourceGroupName="myResourceGroup"

# Subscription ID where the Azure Cosmos account and virtual network resources are located
SubscriptionId="<your Azure subscription ID>"

# Name of the existing Azure Cosmos account
CosmosDbAccountName="mycosmosaccount"

# API type of your Azure Cosmos account: Sql, MongoDB, Cassandra, Gremlin, or Table
CosmosDbApiType="Sql"

# Name of the virtual network to create
VNetName="myVnet"

# Name of the subnet to create
SubnetName="mySubnet"

# Name of the private endpoint to create
PrivateEndpointName="myPrivateEndpoint"

# Name of the private endpoint connection to create
PrivateConnectionName="myConnection"

az network vnet create \
 --name $VNetName \
 --resource-group $ResourceGroupName \
 --subnet-name $SubnetName

az network vnet subnet update \
 --name $SubnetName \
 --resource-group $ResourceGroupName \
 --vnet-name $VNetName \
 --disable-private-endpoint-network-policies true

az network private-endpoint create \
    --name $PrivateEndpointName \
    --resource-group $ResourceGroupName \
    --vnet-name $VNetName  \
    --subnet $SubnetName \
    --private-connection-resource-id "/subscriptions/$SubscriptionId/resourceGroups/$ResourceGroupName/providers/Microsoft.DocumentDB/databaseAccounts/$CosmosDbAccountName" \
    --group-ids $CosmosDbApiType \
    --connection-name $PrivateConnectionName
```

### <a name="integrate-the-private-endpoint-with-a-private-dns-zone"></a>Интеграция частной конечной точки с частной зоной DNS

После создания частной конечной точки ее можно интегрировать с частной зоной DNS с помощью следующего сценария Azure CLI.

```azurecli-interactive
zoneName="privatelink.documents.azure.com"

az network private-dns zone create --resource-group $ResourceGroupName \
   --name  $zoneName

az network private-dns link vnet create --resource-group $ResourceGroupName \
   --zone-name  $zoneName\
   --name myzonelink \
   --virtual-network $VNetName \
   --registration-enabled false 

#Query for the network interface ID  
networkInterfaceId=$(az network private-endpoint show --name $PrivateEndpointName --resource-group $ResourceGroupName --query 'networkInterfaces[0].id' -o tsv)
 
# Copy the content for privateIPAddress and FQDN matching the Azure Cosmos account 
az resource show --ids $networkInterfaceId --api-version 2019-04-01 -o json 
 
#Create DNS records 
az network private-dns record-set a create --name recordSet1 --zone-name privatelink.documents.azure.com --resource-group $ResourceGroupName
az network private-dns record-set a add-record --record-set-name recordSet2 --zone-name privatelink.documents.azure.com --resource-group $ResourceGroupName -a <Private IP Address>
```

## <a name="create-a-private-endpoint-by-using-a-resource-manager-template"></a>Создание частной конечной точки с помощью шаблона Resource Manager

Можно настроить Приватный канал, создав частную конечную точку в подсети виртуальной сети. Это достигается с помощью шаблона Azure Resource Manager.

Используйте следующий код для создания шаблона Resource Manager с именем "PrivateEndpoint_template.json". Этот шаблон используется для создания частной конечной точки для имеющейся учетной записи API Azure Cosmos SQL в существующей виртуальной сети.

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
          "type": "string",
          "defaultValue": "[resourceGroup().location]",
          "metadata": {
            "description": "Location for all resources."
          }
        },
        "privateEndpointName": {
            "type": "string"
        },
        "resourceId": {
            "type": "string"
        },
        "groupId": {
            "type": "string"
        },
        "subnetId": {
            "type": "string"
        }
    },
    "resources": [
        {
            "name": "[parameters('privateEndpointName')]",
            "type": "Microsoft.Network/privateEndpoints",
            "apiVersion": "2019-04-01",
            "location": "[parameters('location')]",
            "properties": {
                "subnet": {
                    "id": "[parameters('subnetId')]"
                },
                "privateLinkServiceConnections": [
                    {
                        "name": "MyConnection",
                        "properties": {
                            "privateLinkServiceId": "[parameters('resourceId')]",
                            "groupIds": [ "[parameters('groupId')]" ],
                            "requestMessage": ""
                        }
                    }
                ]
            }
        }
    ],
    "outputs": {
        "privateEndpointNetworkInterface": {
          "type": "string",
          "value": "[reference(concat('Microsoft.Network/privateEndpoints/', parameters('privateEndpointName'))).networkInterfaces[0].id]"
        }
    }
}
```

**Настройка файла параметров для шаблона**

Создайте файл параметров для шаблона и назовите его "PrivateEndpoint_parameters.json". Добавьте следующий код в файл параметров.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "privateEndpointName": {
            "value": ""
        },
        "resourceId": {
            "value": ""
        },
        "groupId": {
            "value": ""
        },
        "subnetId": {
            "value": ""
        }
    }
}
```

**Развертывание шаблона с помощью сценария PowerShell**

Создайте сценарий PowerShell со следующим содержимым. Перед запуском сценария замените идентификатор подписки, имя группы ресурсов и другие значения переменных на значения вашей среды.

```azurepowershell-interactive
### This script creates a private endpoint for an existing Azure Cosmos account in an existing virtual network

## Step 1: Fill in these details. Replace the variable values with the details for your environment.
$SubscriptionId = "<your Azure subscription ID>"
# Resource group where the Azure Cosmos account and virtual network resources are located
$ResourceGroupName = "myResourceGroup"
# Name of the Azure Cosmos account
$CosmosDbAccountName = "mycosmosaccount"
# API type of the Azure Cosmos account. It can be one of the following: "Sql", "MongoDB", "Cassandra", "Gremlin", "Table"
$CosmosDbApiType = "Sql"
# Name of the existing virtual network
$VNetName = "myVnet"
# Name of the target subnet in the virtual network
$SubnetName = "mySubnet"
# Name of the private endpoint to create
$PrivateEndpointName = "myPrivateEndpoint"

$cosmosDbResourceId = "/subscriptions/$($SubscriptionId)/resourceGroups/$($ResourceGroupName)/providers/Microsoft.DocumentDB/databaseAccounts/$($CosmosDbAccountName)"
$VNetResourceId = "/subscriptions/$($SubscriptionId)/resourceGroups/$($ResourceGroupName)/providers/Microsoft.Network/virtualNetworks/$($VNetName)"
$SubnetResourceId = "$($VNetResourceId)/subnets/$($SubnetName)"
$PrivateEndpointTemplateFilePath = "PrivateEndpoint_template.json"
$PrivateEndpointParametersFilePath = "PrivateEndpoint_parameters.json"

## Step 2: Sign in to your Azure account and select the target subscription.
Login-AzAccount
Select-AzSubscription -SubscriptionId $subscriptionId

## Step 3: Make sure private endpoint network policies are disabled in the subnet.
$VirtualNetwork= Get-AzVirtualNetwork -Name "$VNetName" -ResourceGroupName "$ResourceGroupName"
($virtualNetwork | Select -ExpandProperty subnets | Where-Object  {$_.Name -eq "$SubnetName"} ).PrivateEndpointNetworkPolicies = "Disabled"
$virtualNetwork | Set-AzVirtualNetwork

## Step 4: Create the private endpoint.
Write-Output "Deploying private endpoint on $($resourceGroupName)"
$deploymentOutput = New-AzResourceGroupDeployment -Name "PrivateCosmosDbEndpointDeployment" `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile $PrivateEndpointTemplateFilePath `
    -TemplateParameterFile $PrivateEndpointParametersFilePath `
    -SubnetId $SubnetResourceId `
    -ResourceId $CosmosDbResourceId `
    -GroupId $CosmosDbApiType `
    -PrivateEndpointName $PrivateEndpointName

$deploymentOutput
```

В сценарии PowerShell переменная `GroupId` может содержать только одно значение. Это значение является типом API учетной записи. Допустимые значения: `Sql`, `MongoDB`, `Cassandra`, `Gremlin` и `Table`. Некоторые типы учетных записей Azure Cosmos доступны через несколько интерфейсов API. Пример:

* Доступ к учетной записи API Gremlin можно получить из учетных записей API Gremlin и SQL.
* Доступ к учетной записи API Table можно получить из учетных записей API Table и SQL.

Для этих учетных записей необходимо создать одну частную конечную точку для каждого типа API. Соответствующий тип API указывается в массиве `GroupId`.

После успешного развертывания шаблона должны отобразиться выходные данные, аналогичные приведенным ниже. Если частные конечные точки настроены правильно, параметр `provisioningState` должен иметь значение `Succeeded`.

:::image type="content" source="./media/how-to-configure-private-endpoints/resource-manager-template-deployment-output.png" alt-text="Выходные данные развертывания шаблона Resource Manager":::

После развертывания шаблона частные IP-адреса резервируются в подсети. Правило брандмауэра учетной записи Azure Cosmos настраивается для приема подключений только от частной конечной точки.

### <a name="integrate-the-private-endpoint-with-a-private-dns-zone"></a>Интеграция частной конечной точки с частной зоной DNS

Используйте следующий код для создания шаблона Resource Manager с именем "PrivateZone_template.json". Этот шаблон используется для создания частной зоны DNS для имеющейся учетной записи API Azure Cosmos SQL в существующей виртуальной сети.

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "privateZoneName": {
            "type": "string"
        },
        "VNetId": {
            "type": "string"
        }        
    },
    "resources": [
        {
            "name": "[parameters('privateZoneName')]",
            "type": "Microsoft.Network/privateDnsZones",
            "apiVersion": "2018-09-01",
            "location": "global",
            "properties": {                
            }
        },
        {
            "type": "Microsoft.Network/privateDnsZones/virtualNetworkLinks",
            "apiVersion": "2018-09-01",
            "name": "[concat(parameters('privateZoneName'), '/myvnetlink')]",
            "location": "global",
            "dependsOn": [
                "[resourceId('Microsoft.Network/privateDnsZones', parameters('privateZoneName'))]"
            ],
            "properties": {
                "registrationEnabled": false,
                "virtualNetwork": {
                    "id": "[parameters('VNetId')]"
                }
            }
        }        
    ]
}
```

Используйте следующий код для создания шаблона Resource Manager с именем "PrivateZoneRecords_template.json".

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "DNSRecordName": {
            "type": "string"
        },
        "IPAddress": {
            "type":"string"
        }        
    },
    "resources": [
         {
            "type": "Microsoft.Network/privateDnsZones/A",
            "apiVersion": "2018-09-01",
            "name": "[parameters('DNSRecordName')]",
            "properties": {
                "ttl": 300,
                "aRecords": [
                    {
                        "ipv4Address": "[parameters('IPAddress')]"
                    }
                ]
            }
        }    
    ]
}
```

**Настройка файла параметров для шаблона**

Создайте для шаблона следующие два файла параметров. Создайте файл "PrivateZone_parameters.json" на новый код:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "privateZoneName": {
            "value": ""
        },
        "VNetId": {
            "value": ""
        }
    }
}
```

Создайте файл "PrivateZoneRecords_parameters.json" на новый код:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "DNSRecordName": {
            "value": ""
        },
        "IPAddress": {
            "type":"object"
        }
    }
}
```

**Развертывание шаблона с помощью сценария PowerShell**

Создайте сценарий PowerShell со следующим содержимым. Перед запуском сценария замените идентификатор подписки, имя группы ресурсов и другие значения переменных на значения вашей среды.

```azurepowershell-interactive
### This script:
### - creates a private zone
### - creates a private endpoint for an existing Cosmos DB account in an existing VNet
### - maps the private endpoint to the private zone

## Step 1: Fill in these details. Replace the variable values with the details for your environment.
$SubscriptionId = "<your Azure subscription ID>"
# Resource group where the Azure Cosmos account and virtual network resources are located
$ResourceGroupName = "myResourceGroup"
# Name of the Azure Cosmos account
$CosmosDbAccountName = "mycosmosaccount"
# API type of the Azure Cosmos account. It can be one of the following: "Sql", "MongoDB", "Cassandra", "Gremlin", "Table"
$CosmosDbApiType = "Sql"
# Name of the existing virtual network
$VNetName = "myVnet"
# Name of the target subnet in the virtual network
$SubnetName = "mySubnet"
# Name of the private zone to create
$PrivateZoneName = "myPrivateZone.documents.azure.com"
# Name of the private endpoint to create
$PrivateEndpointName = "myPrivateEndpoint"

$cosmosDbResourceId = "/subscriptions/$($SubscriptionId)/resourceGroups/$($ResourceGroupName)/providers/Microsoft.DocumentDB/databaseAccounts/$($CosmosDbAccountName)"
$VNetResourceId = "/subscriptions/$($SubscriptionId)/resourceGroups/$($ResourceGroupName)/providers/Microsoft.Network/virtualNetworks/$($VNetName)"
$SubnetResourceId = "$($VNetResourceId)/subnets/$($SubnetName)"
$PrivateZoneTemplateFilePath = "PrivateZone_template.json"
$PrivateZoneParametersFilePath = "PrivateZone_parameters.json"
$PrivateZoneRecordsTemplateFilePath = "PrivateZoneRecords_template.json"
$PrivateZoneRecordsParametersFilePath = "PrivateZoneRecords_parameters.json"
$PrivateEndpointTemplateFilePath = "PrivateEndpoint_template.json"
$PrivateEndpointParametersFilePath = "PrivateEndpoint_parameters.json"

## Step 2: Login your Azure account and select the target subscription
Login-AzAccount 
Select-AzSubscription -SubscriptionId $subscriptionId

## Step 3: Make sure private endpoint network policies are disabled in the subnet
$VirtualNetwork= Get-AzVirtualNetwork -Name "$VNetName" -ResourceGroupName "$ResourceGroupName"
($virtualNetwork | Select -ExpandProperty subnets | Where-Object  {$_.Name -eq "$SubnetName"} ).PrivateEndpointNetworkPolicies = "Disabled"
$virtualNetwork | Set-AzVirtualNetwork

## Step 4: Create the private zone
New-AzResourceGroupDeployment -Name "PrivateZoneDeployment" `
    -ResourceGroupName $ResourceGroupName `
    -TemplateFile $PrivateZoneTemplateFilePath `
    -TemplateParameterFile $PrivateZoneParametersFilePath `
    -PrivateZoneName $PrivateZoneName `
    -VNetId $VNetResourceId

## Step 5: Create the private endpoint
Write-Output "Deploying private endpoint on $($resourceGroupName)"
$deploymentOutput = New-AzResourceGroupDeployment -Name "PrivateCosmosDbEndpointDeployment" `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile $PrivateEndpointTemplateFilePath `
    -TemplateParameterFile $PrivateEndpointParametersFilePath `
    -SubnetId $SubnetResourceId `
    -ResourceId $CosmosDbResourceId `
    -GroupId $CosmosDbApiType `
    -PrivateEndpointName $PrivateEndpointName
$deploymentOutput

## Step 6: Map the private endpoint to the private zone
$networkInterface = Get-AzResource -ResourceId $deploymentOutput.Outputs.privateEndpointNetworkInterface.Value -ApiVersion "2019-04-01"
foreach ($ipconfig in $networkInterface.properties.ipConfigurations) {
    foreach ($fqdn in $ipconfig.properties.privateLinkConnectionProperties.fqdns) {
        $recordName = $fqdn.split('.',2)[0]
        $dnsZone = $fqdn.split('.',2)[1]
        Write-Output "Deploying PrivateEndpoint DNS Record $($PrivateZoneName)/$($recordName) Template on $($resourceGroupName)"
        New-AzResourceGroupDeployment -Name "PrivateEndpointDNSDeployment" `
            -ResourceGroupName $ResourceGroupName `
            -TemplateFile $PrivateZoneRecordsTemplateFilePath `
            -TemplateParameterFile $PrivateZoneRecordsParametersFilePath `
            -DNSRecordName "$($PrivateZoneName)/$($RecordName)" `
            -IPAddress $ipconfig.properties.privateIPAddress
    }
}
```

## <a name="configure-custom-dns"></a>Настройка пользовательской службы DNS

Необходимо использовать частную зону DNS в подсети, в которой была создана частная конечная точка. Настройте конечные точки таким образом, чтобы каждый частный IP-адрес был сопоставлен с записью DNS. (См. значение свойства `fqdns` в ответе, показанном выше.)

При создании частной конечной точки ее можно интегрировать с частной зоной DNS в Azure. Если вместо этого вы решили использовать пользовательскую зону DNS, необходимо настроить ее, чтобы добавить записи DNS для всех частных IP-адресов, зарезервированных для частной конечной точки.

> [!IMPORTANT]
> Это разрешение DNS для запросов, которые определяют, проходят ли эти запросы к частным конечным точкам, или принимают стандартный открытый маршрут. Убедитесь, что локальная служба DNS правильно ссылается на частный IP-адрес, сопоставленный с частной конечной точкой.

## <a name="private-link-combined-with-firewall-rules"></a>Использование Приватного канала совместно с правилами брандмауэра

При использовании Приватного канала совместно с правилами брандмауэра возможны следующие ситуации и результаты.

* Если не настраивать никакие правила брандмауэра, по умолчанию доступ к учетной записи Azure Cosmos будет иметь весь трафик.

* Если вы настраиваете общедоступный трафик или конечную точку службы и создаете частные конечные точки, то для разных типов входящего трафика применяется соответствующий тип правила брандмауэра. Если частная конечная точка настроена в подсети, где также настроена конечная точка службы:
  * трафик к учетной записи базы данных, сопоставленной с частной конечной точкой, направляется через частную конечную точку.
  * трафик к другим учетным записям базы данных из подсети направляется через конечную точку службы.

* Если вы не настраиваете никакого общего трафика или конечной точки службы, но создаете частные конечные точки, учетная запись Azure Cosmos будет доступна только через частные конечные точки. Если не настроить общедоступный трафик или конечную точку службы после того, как все утвержденные частные конечные точки будут отклонены или удалены, учетная запись будет открыта для всей сети, если для Публикнетворкакцесс не задано значение Disabled (см. раздел ниже).

## <a name="blocking-public-network-access-during-account-creation"></a>Блокировка доступа из общедоступной сети во время создания учетной записи

Как описано в предыдущем разделе и если не заданы конкретные правила брандмауэра, добавление частной конечной точки делает учетную запись Azure Cosmos доступной только для частных конечных точек. Это означает, что учетная запись Azure Cosmos после создания, но перед добавлением закрытой конечной точки будет доступна из общедоступного трафика. Чтобы запретить доступ из общедоступной сети еще до создания частных конечных точек, можно во время создания учетной записи установить флаг `publicNetworkAccess` в значение `Disabled`. Обратите внимание, что этот флаг имеет приоритет над любым правилом IP или виртуальной сети. весь трафик общедоступного и виртуальной сети блокируется, если для флага задано значение `Disabled` , даже если исходный IP-адрес или виртуальная сеть разрешены в конфигурации брандмауэра.

Пример использования этого флага см. в [шаблоне Azure Resource Manager](https://azure.microsoft.com/resources/templates/101-cosmosdb-private-endpoint/).

## <a name="adding-private-endpoints-to-an-existing-cosmos-account-with-no-downtime"></a>Добавление частных конечных точек к существующей учетной записи Cosmos без простоя

По умолчанию Добавление частной конечной точки к существующей учетной записи приводит к короткому простою примерно через 5 минут. Чтобы избежать этого простоя, следуйте приведенным ниже инструкциям.

1. Добавьте правила IP или виртуальной сети в конфигурацию брандмауэра, чтобы явно разрешить подключения клиентов.
1. Подождите 10 минут и убедитесь, что обновление конфигурации установлено.
1. Настройте новую закрытую конечную точку.
1. Удалите правила брандмауэра, установленные на шаге 1.

## <a name="port-range-when-using-direct-mode"></a>Диапазон портов при использовании прямого режима

Если вы используете закрытую связь с учетной записью Azure Cosmos через подключение в режиме прямого подключения, необходимо убедиться, что весь диапазон TCP-портов (0-65535) открыт.

## <a name="update-a-private-endpoint-when-you-add-or-remove-a-region"></a>Обновление частной конечной точки при добавлении или удалении региона

Если вы не используете частную группу зон DNS, добавление или удаление регионов в учетной записи Azure Cosmos требует добавления или удаления записей DNS для этой учетной записи. После добавления или удаления регионов можно обновить частную зону DNS подсети, чтобы отразить добавленные или удаленные записи DNS и соответствующие им частные IP-адреса.

Предположим, например, что вы развертываете учетную запись Azure Cosmos в трех регионах: "Западная часть США", "Центральная часть США" и "Западная Европа". При создании частной конечной точки для вашей учетной записи в подсети зарезервированы четыре частных IP-адреса. По одному IP-адресу для каждого из трех регионов и один IP-адрес для глобальной конечной точки, не зависящей от региона.

Позже вы могли добавить в учетную запись Azure Cosmos новый регион (например, "Восточная часть США"). После добавления нового региона необходимо добавить соответствующую запись DNS в вашу частную зону DNS или пользовательскую службу DNS.

Аналогичные действия необходимы и при удалении региона. После удаления региона необходимо удалить соответствующую запись DNS из вашей частной зоны DNS или пользовательской службы DNS.

## <a name="current-limitations"></a>Текущие ограничения

При использовании Приватного канала с учетной записью Azure Cosmos действуют следующие ограничения.

* В одной учетной записи Azure Cosmos не может быть более 200 частных конечных точек.

* Если вы используете закрытую связь с учетной записью Azure Cosmos через подключение в прямом режиме, можно использовать только протокол TCP. Протокол HTTP в настоящее время не поддерживается.

* При использовании API Azure Cosmos DB для учетных записей MongoDB частная конечная точка поддерживается только для учетных записей на серверах версии 3.6 (то есть для учетных записей, использующих конечную точку в формате `*.mongo.cosmos.azure.com`). Приватный канал не поддерживается для учетных записей на серверах версии 3.2 (то есть для учетных записей, использующих конечную точку в формате `*.documents.azure.com`). Чтобы использовать Приватный канал, необходимо перенести старые учетные записи на новую версию.

* Если вы используете API Azure Cosmos DB для учетной записи MongoDB с частной ссылкой, инструменты и библиотеки должны поддерживать идентификацию имени службы (SNI) или передавать `appName` параметр из строки подключения для правильного подключения. Некоторые устаревшие средства и библиотеки могут быть несовместимы с возможностью использования закрытой ссылки.

* Для создания автоматически утверждаемых частных конечных точек администратору сети необходимо предоставить по крайней мере разрешение `Microsoft.DocumentDB/databaseAccounts/PrivateEndpointConnectionsApproval/action` в области учетной записи Azure Cosmos.

### <a name="limitations-to-private-dns-zone-integration"></a>Ограничения на интеграцию частной зоны DNS

Если вы не используете частную группу зон DNS, записи DNS в частной зоне DNS не удаляются автоматически при удалении частной конечной точки или при удалении региона из учетной записи Azure Cosmos. Необходимо вручную удалить записи DNS перед:

* добавлением новой частной конечной точки, связанной с этой частной зоной DNS;
* добавлением нового региона в любую учетную запись базы данных с частными конечными точками, связанными с этой частной зоной DNS.

Если не очистить записи DNS, могут возникнуть непредвиденные проблемы с плоскостью данных. Эти проблемы включают сбои данных в регионах, добавленных после удаления частной конечной точки или удаления региона.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о средствах безопасности Azure Cosmos DB см. в следующих статьях.

* Сведения о настройке брандмауэра для Azure Cosmos DB см. в статье о [поддержке брандмауэра](how-to-configure-firewall.md).

* Сведения о настройке конечной точки службы виртуальной сети для учетной записи Azure Cosmos см. в статье [Настройка доступа из виртуальных сетей](how-to-configure-vnet-service-endpoint.md).

* Дополнительные сведения о Приватных каналах см. в документации по [Приватным каналам Azure](../private-link/private-link-overview.md).