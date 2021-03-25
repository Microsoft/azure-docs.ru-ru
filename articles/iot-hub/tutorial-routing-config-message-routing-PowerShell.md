---
title: Руководство. Настройка маршрутизации сообщений для Центра Интернета вещей Azure с помощью Azure PowerShell
description: Руководство по настройке маршрутизации сообщений для Центра Интернета вещей Azure с помощью Azure PowerShell. В зависимости от свойств сообщения выполняйте маршрутизацию в учетную запись хранения или очередь служебной шины.
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: tutorial
ms.date: 03/25/2019
ms.author: robinsh
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: be1560bcc03ec7a26f4bc374392c746243cd731a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98624077"
---
# <a name="tutorial-use-azure-powershell-to-configure-iot-hub-message-routing"></a>Руководство по Маршрутизация сообщений Центра Интернета вещей с помощью Azure PowerShell

[!INCLUDE [iot-hub-include-routing-intro](../../includes/iot-hub-include-routing-intro.md)]

[!INCLUDE [iot-hub-include-routing-create-resources](../../includes/iot-hub-include-routing-create-resources.md)]

## <a name="download-the-script-optional"></a>Скачивание скрипта (необязательно)

Для работы со второй частью этого руководства нужно скачать и запустить приложение Visual Studio для отправки сообщений в Центр Интернета вещей. В скачиваемых ресурсах есть папка, которая содержит шаблон Azure Resource Manager и файл параметров, а также скрипты Azure CLI и PowerShell. 

Если вы хотите просмотреть готовый скрипт, скачайте [примеры на C# для Интернета вещей Azure](https://github.com/Azure-Samples/azure-iot-samples-csharp/archive/master.zip). Распакуйте файл master.zip. Скрипт Azure CLI находится в каталоге /iot-hub/Tutorials/Routing/SimulatedDevice/resources/ в виде файла **iothub_routing_psh.ps1**.

## <a name="create-your-resources"></a>Создание нужных ресурсов

Начните с создания ресурсов с помощью PowerShell.

### <a name="use-powershell-to-create-your-base-resources"></a>Для создания базовых ресурсов воспользуйтесь PowerShell

Скопируйте и вставьте приведенный ниже скрипт в Cloud Shell и нажмите клавишу ВВОД. Скрипт будет выполняться построчно. Первый раздел скрипта создает базовые ресурсы для работы с этим руководством, включая учетную запись хранения, Центр Интернета вещей, пространство имен служебной шины и очереди служебной шины. Работая с этим руководством, копируйте и вставляйте все блоки скрипта в Cloud Shell, чтобы запустить его.

Некоторым ресурсам необходимо присвоить уникальные имена, включая имя Центра Интернета вещей и имя учетной записи хранения. Чтобы упростить эту задачу, к именам этих ресурсов добавляется случайное буквенно-цифровое значение *randomValue*. Значение randomValue создается один раз в верхней части скрипта и повсеместно добавляется к соответствующим именам ресурсов в пределах скрипта. Если вы не хотите, чтобы это значение было случайным, можно заменить его пустой строкой или определенным значением. 

> [!IMPORTANT]
> Так как переменные, заданные в исходном скрипте, также будут использоваться скриптом маршрутизации, можно выполнить весь скрипт в том же сеансе Cloud Shell. Если открыть новый сеанс для выполнения скрипта для настройки маршрутизации, у нескольких переменных будут отсутствовать значения. 
>

```azurepowershell-interactive
# This command retrieves the subscription id of the current Azure account.
# This field is used when setting up the routing queries.
$subscriptionID = (Get-AzContext).Subscription.Id

# Concatenate this number onto the resources that have to be globally unique.
# You can set this to "" or to a specific value if you don't want it to be random.
# This retrieves the first 6 digits of a random value.
$randomValue = "$(Get-Random)".Substring(0,6)

# Set the values for the resource names that don't have to be globally unique.
$location = "West US"
$resourceGroup = "ContosoResources"
$iotHubConsumerGroup = "ContosoConsumers"
$containerName = "contosoresults"

# Create the resource group to be used 
#   for all resources for this tutorial.
New-AzResourceGroup -Name $resourceGroup -Location $location

# The IoT hub name must be globally unique, 
#   so add a random value to the end.
$iotHubName = "ContosoTestHub" + $randomValue
Write-Host "IoT hub name is " $iotHubName

# Create the IoT hub.
New-AzIotHub -ResourceGroupName $resourceGroup `
    -Name $iotHubName `
    -SkuName "S1" `
    -Location $location `
    -Units 1 

# Add a consumer group to the IoT hub for the 'events' endpoint.
Add-AzIotHubEventHubConsumerGroup -ResourceGroupName $resourceGroup `
  -Name $iotHubName `
  -EventHubConsumerGroupName $iotHubConsumerGroup `
  -EventHubEndpointName "events"

# The storage account name must be globally unique, so add a random value to the end.
$storageAccountName = "contosostorage" + $randomValue
Write-Host "storage account name is " $storageAccountName

# Create the storage account to be used as a routing destination.
# Save the context for the storage account 
#   to be used when creating a container.
$storageAccount = New-AzStorageAccount -ResourceGroupName $resourceGroup `
    -Name $storageAccountName `
    -Location $location `
    -SkuName Standard_LRS `
    -Kind Storage
# Retrieve the connection string from the context. 
$storageConnectionString = $storageAccount.Context.ConnectionString
Write-Host "storage connection string = " $storageConnectionString 

# Create the container in the storage account.
New-AzStorageContainer -Name $containerName `
    -Context $storageAccount.Context

# The Service Bus namespace must be globally unique,
#   so add a random value to the end.
$serviceBusNamespace = "ContosoSBNamespace" + $randomValue
Write-Host "Service Bus namespace is " $serviceBusNamespace

# Create the Service Bus namespace.
New-AzServiceBusNamespace -ResourceGroupName $resourceGroup `
    -Location $location `
    -Name $serviceBusNamespace 

# The Service Bus queue name must be globally unique,
#  so add a random value to the end.
$serviceBusQueueName  = "ContosoSBQueue" + $randomValue
Write-Host "Service Bus queue name is " $serviceBusQueueName 

# Create the Service Bus queue to be used as a routing destination.
New-AzServiceBusQueue -ResourceGroupName $resourceGroup `
    -Namespace $serviceBusNamespace `
    -Name $serviceBusQueueName  `
    -EnablePartitioning $False 
```

### <a name="create-a-simulated-device"></a>Создайте виртуальное устройство.

[!INCLUDE [iot-hub-include-create-simulated-device-portal](../../includes/iot-hub-include-create-simulated-device-portal.md)]

Теперь, когда базовые ресурсы готовы, можно настроить маршрутизацию сообщений.

## <a name="set-up-message-routing"></a>Настройка маршрутизации сообщений

[!INCLUDE [iot-hub-include-create-routing-description](../../includes/iot-hub-include-create-routing-description.md)]

Чтобы создать конечную точку маршрутизации, выполните команду [Add-AzIotHubRoutingEndpoint](/powershell/module/az.iothub/Add-AzIotHubRoutingEndpoint). Чтобы создать маршрут обмена сообщениями для конечной точки, выполните команду [Add-AzIotHubRoute](/powershell/module/az.iothub/Add-AzIoTHubRoute).

### <a name="route-to-a-storage-account"></a>Маршрутизация в учетную запись хранения 

Сначала настройте конечную точку учетной записи хранения, а затем создайте маршрут сообщений.

[!INCLUDE [iot-hub-include-blob-storage-format](../../includes/iot-hub-include-blob-storage-format.md)]

Это переменные, используемые скриптом, которые необходимо задать в сеансе Cloud Shell:

**resourceGroup**. Это поле встречается дважды. Задайте значения в обоих для своей группы ресурсов.

**name**. В этом поле указывается имя Центра Интернета вещей, к которому будет применяться маршрутизация.

**endpointName**. В этом поле указывается имя, определяющее конечную точку. 

**endpointType**. Здесь указывается тип конечной точки. Это значение должно быть `azurestoragecontainer`, `eventhub`, `servicebusqueue` или `servicebustopic`. В нашем примере используется значение `azurestoragecontainer`.

**subscriptionID**. В этом поле указывается идентификатор вашей учетной записи Azure.

**storageConnectionString**. Это значение извлекается из учетной записи хранения, настроенной в предыдущем скрипте. Оно используется при маршрутизации сообщений для получения доступа к учетной записи хранения.

**containerName**. В этом поле указывается имя контейнера в учетной записи хранения, куда будут записаны данные.

**Encoding**. В этом поле укажите значение `AVRO` или `JSON`. Оно определяет формат хранимых данных. По умолчанию используется AVRO.

**routeName**. В этом поле указывается имя маршрута, который вы настраиваете. 

**condition**. В этом поле указывается запрос для фильтрации сообщений, отправляемых в эту конечную точку. Условие запроса для сообщений, маршрутизируемых в хранилище, — `level="storage"`.

**enabled**. По умолчанию в этом поле задано значение `true`, означающее, что маршрут сообщений будет включен после создания.

Скопируйте и вставьте этот скрипт в окно Cloud Shell.

```powershell
##### ROUTING FOR STORAGE #####

$endpointName = "ContosoStorageEndpoint"
$endpointType = "azurestoragecontainer"
$routeName = "ContosoStorageRoute"
$condition = 'level="storage"'
```

Следующим шагом является создание конечной точки маршрутизации для учетной записи хранения. Также укажите контейнер, в котором будут храниться результаты. Этот контейнер был создан при создании учетной записи хранения.

```powershell
# Create the routing endpoint for storage.
# Specify 'AVRO' or 'JSON' for the encoding of the data.
Add-AzIotHubRoutingEndpoint `
  -ResourceGroupName $resourceGroup `
  -Name $iotHubName `
  -EndpointName $endpointName `
  -EndpointType $endpointType `
  -EndpointResourceGroup $resourceGroup `
  -EndpointSubscriptionId $subscriptionId `
  -ConnectionString $storageConnectionString  `
  -ContainerName $containerName `
  -Encoding AVRO
```

Теперь создайте маршрут сообщений для конечной точки хранилища. Маршрут сообщений определяет, куда отправлять сообщения, соответствующие спецификации запроса.

```powershell
# Create the route for the storage endpoint.
Add-AzIotHubRoute `
   -ResourceGroupName $resourceGroup `
   -Name $iotHubName `
   -RouteName $routeName `
   -Source DeviceMessages `
   -EndpointName $endpointName `
   -Condition $condition `
   -Enabled 
```

### <a name="route-to-a-service-bus-queue"></a>Маршрутизация в очередь служебной шины

Теперь настройте маршрутизацию для очереди служебной шины. Чтобы получить строку подключения для очереди служебной шины, необходимо создать правило авторизации с определенными правами. Следующий скрипт создает правила авторизации для очереди служебной шины `sbauthrule` и определяет правила как `Listen Manage Send`. Определив это правило авторизации, вы сможете использовать его для получения строки подключения для очереди.

```powershell
##### ROUTING FOR SERVICE BUS QUEUE #####

# Create the authorization rule for the Service Bus queue.
New-AzServiceBusAuthorizationRule `
  -ResourceGroupName $resourceGroup `
  -NamespaceName $serviceBusNamespace `
  -Queue $serviceBusQueueName `
  -Name "sbauthrule" `
  -Rights @("Manage","Listen","Send")
```

Теперь с помощью правила авторизации можно получить ключ очереди служебной шины. Это правило авторизации будет использоваться позже для получения строки подключения в скрипте.

```powershell
$sbqkey = Get-AzServiceBusKey `
    -ResourceGroupName $resourceGroup `
    -NamespaceName $serviceBusNamespace `
    -Queue $servicebusQueueName `
    -Name "sbauthrule"
```

Настройте конечную точку маршрутизации и маршрут сообщений для очереди служебной шины. Это переменные, используемые скриптом, которые необходимо задать в сеансе Cloud Shell:

**endpointName**. В этом поле указывается имя, определяющее конечную точку. 

**endpointType**. Здесь указывается тип конечной точки. Это значение должно быть `azurestoragecontainer`, `eventhub`, `servicebusqueue` или `servicebustopic`. В нашем примере используется значение `servicebusqueue`.

**routeName**. В этом поле указывается имя маршрута, который вы настраиваете. 

**condition**. В этом поле указывается запрос для фильтрации сообщений, отправляемых в эту конечную точку. Условие запроса для сообщений, маршрутизируемых в очередь служебной шины, — `level="critical"`.

Ниже представлен скрипт Azure PowerShell для маршрутизации сообщений для очереди служебной шины.

```powershell
$endpointName = "ContosoSBQueueEndpoint"
$endpointType = "servicebusqueue"
$routeName = "ContosoSBQueueRoute"
$condition = 'level="critical"'

# Add the routing endpoint, using the connection string property from the key.
Add-AzIotHubRoutingEndpoint `
  -ResourceGroupName $resourceGroup `
  -Name $iotHubName `
  -EndpointName $endpointName `
  -EndpointType $endpointType `
  -EndpointResourceGroup $resourceGroup `
  -EndpointSubscriptionId $subscriptionId `
  -ConnectionString $sbqkey.PrimaryConnectionString

# Set up the message route for the Service Bus queue endpoint.
Add-AzIotHubRoute `
   -ResourceGroupName $resourceGroup `
   -Name $iotHubName `
   -RouteName $routeName `
   -Source DeviceMessages `
   -EndpointName $endpointName `
   -Condition $condition `
   -Enabled 
```

### <a name="view-message-routing-in-the-portal"></a>Просмотр маршрутизации сообщений на портале

[!INCLUDE [iot-hub-include-view-routing-in-portal](../../includes/iot-hub-include-view-routing-in-portal.md)]

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы настроили ресурсы и маршрутизацию сообщений, переходите к следующему руководству, чтобы узнать, как отправлять сообщения в центр Интернета вещей, маршрутизируя их в разные места назначения. 

> [!div class="nextstepaction"]
> [Часть 2. Просмотр результатов маршрутизации сообщений](tutorial-routing-view-message-routing-results.md)
