---
title: Руководство. Настройка маршрутизации сообщений для Центра Интернета вещей Azure с помощью Azure CLI
description: Руководство по настройке маршрутизации сообщений для Центра Интернета вещей Azure с помощью Azure CLI. В зависимости от свойств сообщения выполняйте маршрутизацию в учетную запись хранения или очередь служебной шины.
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: tutorial
ms.date: 03/25/2019
ms.author: robinsh
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: cf994dfe3d53232ab1e2374fda620dc768127097
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107788895"
---
# <a name="tutorial-use-the-azure-cli-to-configure-iot-hub-message-routing"></a>Руководство по Маршрутизация сообщений Центра Интернета вещей с помощью Azure CLI

[!INCLUDE [iot-hub-include-routing-intro](../../includes/iot-hub-include-routing-intro.md)]

[!INCLUDE [iot-hub-include-routing-create-resources](../../includes/iot-hub-include-routing-create-resources.md)]

## <a name="download-the-script-optional"></a>Скачивание скрипта (необязательно)

Для работы со второй частью этого руководства нужно скачать и запустить приложение Visual Studio для отправки сообщений в Центр Интернета вещей. В скачиваемых ресурсах есть папка, которая содержит шаблон Azure Resource Manager и файл параметров, а также скрипты Azure CLI и PowerShell.

Если вы хотите просмотреть готовый скрипт, скачайте [примеры на C# для Интернета вещей Azure](https://github.com/Azure-Samples/azure-iot-samples-csharp/archive/master.zip). Распакуйте файл master.zip. Скрипт Azure CLI находится в каталоге /iot-hub/Tutorials/Routing/SimulatedDevice/resources/ в виде файла **iothub_routing_cli.azcli**.

## <a name="use-the-azure-cli-to-create-your-resources"></a>Создание ресурсов с помощью Azure CLI

Скопируйте и вставьте приведенный ниже скрипт в Cloud Shell и нажмите клавишу ВВОД. Скрипт будет выполняться построчно. Первый раздел скрипта создает базовые ресурсы для работы с этим руководством, включая учетную запись хранения, Центр Интернета вещей, пространство имен служебной шины и очереди служебной шины. Работая с этим руководством, копируйте и вставляйте все блоки скрипта в Cloud Shell, чтобы запустить его.

> [!TIP]
> Замечание относительно отладки: для удобочитаемости этот скрипт использует символ продолжения (обратная косая черта `\`). Если при выполнении скрипта возникла проблема, убедитесь, что сеанс Cloud Shell работает c использованием `bash` и что после символов обратной косой черты нет пробелов.
> 

Некоторым ресурсам необходимо присвоить уникальные имена, включая имя Центра Интернета вещей и имя учетной записи хранения. Чтобы упростить эту задачу, к именам этих ресурсов добавляется случайное буквенно-цифровое значение *randomValue*. Значение randomValue создается один раз в верхней части скрипта и повсеместно добавляется к соответствующим именам ресурсов в пределах скрипта. Если вы не хотите, чтобы это значение было случайным, можно заменить его пустой строкой или определенным значением. 

> [!IMPORTANT]
> Так как переменные, заданные в исходном скрипте, также будут использоваться скриптом маршрутизации, можно выполнить весь скрипт в том же сеансе Cloud Shell. Если открыть новый сеанс для выполнения скрипта для настройки маршрутизации, у нескольких переменных будут отсутствовать значения.
>

```azurecli-interactive
# This command retrieves the subscription id of the current Azure account. 
# This field is used when setting up the routing queries.
subscriptionID=$(az account show --query id -o tsv)

# Concatenate this number onto the resources that have to be globally unique.
# You can set this to "" or to a specific value if you don't want it to be random.
# This retrieves a random value.
randomValue=$RANDOM

# This command installs the IOT Extension for Azure CLI.
# You only need to install this the first time.
# You need it to create the device identity. 
az extension add --name azure-iot

# Set the values for the resource names that 
#   don't have to be globally unique.
location=westus
resourceGroup=ContosoResources
iotHubConsumerGroup=ContosoConsumers
containerName=contosoresults
iotDeviceName=Contoso-Test-Device

# Create the resource group to be used
#   for all the resources for this tutorial.
az group create --name $resourceGroup \
    --location $location

# The IoT hub name must be globally unique, 
#   so add a random value to the end.
iotHubName=ContosoTestHub$randomValue 
echo "IoT hub name = " $iotHubName

# Create the IoT hub.
az iot hub create --name $iotHubName \
    --resource-group $resourceGroup \
    --sku S1 --location $location

# Add a consumer group to the IoT hub for the 'events' endpoint.
az iot hub consumer-group create --hub-name $iotHubName \
    --name $iotHubConsumerGroup

# The storage account name must be globally unique, 
#   so add a random value to the end.
storageAccountName=contosostorage$randomValue
echo "Storage account name = " $storageAccountName

# Create the storage account to be used as a routing destination.
az storage account create --name $storageAccountName \
    --resource-group $resourceGroup \
    --location $location \
    --sku Standard_LRS

# Get the primary storage account key. 
#    You need this to create the container.
storageAccountKey=$(az storage account keys list \
    --resource-group $resourceGroup \
    --account-name $storageAccountName \
    --query "[0].value" | tr -d '"') 

# See the value of the storage account key.
echo "storage account key = " $storageAccountKey

# Create the container in the storage account. 
az storage container create --name $containerName \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --public-access off

# The Service Bus namespace must be globally unique, 
#   so add a random value to the end.
sbNamespace=ContosoSBNamespace$randomValue
echo "Service Bus namespace = " $sbNamespace

# Create the Service Bus namespace.
az servicebus namespace create --resource-group $resourceGroup \
    --name $sbNamespace \
    --location $location

# The Service Bus queue name must be globally unique, 
#   so add a random value to the end.
sbQueueName=ContosoSBQueue$randomValue
echo "Service Bus queue name = " $sbQueueName

# Create the Service Bus queue to be used as a routing destination.
az servicebus queue create --name $sbQueueName \
    --namespace-name $sbNamespace \
    --resource-group $resourceGroup

# Create the IoT device identity to be used for testing.
az iot hub device-identity create --device-id $iotDeviceName \
    --hub-name $iotHubName

# Retrieve the information about the device identity, then copy the primary key to
#   Notepad. You need this to run the device simulation during the testing phase.
az iot hub device-identity show --device-id $iotDeviceName \
    --hub-name $iotHubName
```

Теперь, когда базовые ресурсы готовы, можно настроить маршрутизацию сообщений.

## <a name="set-up-message-routing"></a>Настройка маршрутизации сообщений

[!INCLUDE [iot-hub-include-create-routing-description](../../includes/iot-hub-include-create-routing-description.md)]

Чтобы создать конечную точку маршрутизации, используйте [az iot hub routing-endpoint create](/cli/azure/iot/hub/routing-endpoint#az_iot_hub_routing_endpoint_create). Чтобы создать маршрут сообщения для конечной точки, используйте [az iot hub route create](/cli/azure/iot/hub/route#az_iot_hub_route_create).

### <a name="route-to-a-storage-account"></a>Маршрутизация в учетную запись хранения

[!INCLUDE [iot-hub-include-blob-storage-format](../../includes/iot-hub-include-blob-storage-format.md)]

Сначала настройте конечную точку учетной записи хранения, а затем настройте маршрут. 

Это переменные, используемые скриптом, которые необходимо задать в сеансе Cloud Shell:

**storageConnectionString**. Это значение извлекается из учетной записи хранения, настроенной в предыдущем скрипте. Оно используется при маршрутизации сообщений для получения доступа к учетной записи хранения.

  **resourceGroup**. Есть два экземпляра группы ресурсов, которые нужно использовать.

**endpoint subscriptionID**. Это поле представляет значение идентификатора подписки Azure для конечной точки. 

**endpointType**. Здесь указывается тип конечной точки. Это значение должно быть `azurestoragecontainer`, `eventhub`, `servicebusqueue` или `servicebustopic`. В нашем примере используется значение `azurestoragecontainer`.

**iotHubName**: Это поле представляет имя приложения логики.

**containerName**. В этом поле указывается имя контейнера в учетной записи хранения, куда будут записаны данные.

**encoding**. Это поле представляет значение `avro` или `json`. Оно описывает формат хранимых данных.

**routeName**. В этом поле указывается имя маршрута, который вы настраиваете. 

**endpointName**. В этом поле указывается имя, определяющее конечную точку. 

**enabled**. По умолчанию в этом поле задано значение `true`, означающее, что маршрут сообщений будет включен после создания.

**condition**. В этом поле указывается запрос для фильтрации сообщений, отправляемых в эту конечную точку. Условие запроса для сообщений, маршрутизируемых в хранилище, — `level="storage"`.

Скопируйте и вставьте этот скрипт в окно Cloud Shell, а затем запустите его.

```azurecli
##### ROUTING FOR STORAGE ##### 

endpointName="ContosoStorageEndpoint"
endpointType="azurestoragecontainer"
routeName="ContosoStorageRoute"
condition='level="storage"'

# Get the connection string for the storage account.
# Adding the "-o tsv" makes it be returned without the default double quotes around it.
storageConnectionString=$(az storage account show-connection-string \
  --name $storageAccountName --query connectionString -o tsv)
```

Следующим шагом является создание конечной точки маршрутизации для учетной записи хранения. Также укажите контейнер, в котором будут храниться результаты. Этот контейнер был создан ранее при создании учетной записи хранения.

```azurecli
# Create the routing endpoint for storage.
az iot hub routing-endpoint create \
  --connection-string $storageConnectionString \
  --endpoint-name $endpointName \
  --endpoint-resource-group $resourceGroup \
  --endpoint-subscription-id $subscriptionID \
  --endpoint-type $endpointType \
  --hub-name $iotHubName \
  --container $containerName \
  --resource-group $resourceGroup \
  --encoding avro
```

Теперь создайте маршрут для конечной точки хранилища. Маршрут сообщений определяет, куда отправлять сообщения, соответствующие спецификации запроса. 

```azurecli
# Create the route for the storage endpoint.
az iot hub route create \
  --name $routeName \
  --hub-name $iotHubName \
  --source devicemessages \
  --resource-group $resourceGroup \
  --endpoint-name $endpointName \
  --enabled \
  --condition $condition
```

### <a name="route-to-a-service-bus-queue"></a>Маршрутизация в очередь служебной шины

Теперь настройте маршрутизацию для очереди служебной шины. Чтобы получить строку подключения для очереди служебной шины, необходимо создать правило авторизации с определенными правами. Следующий скрипт создает правила авторизации для очереди служебной шины `sbauthrule` и определяет правила как `Listen Manage Send`. Определив это правил авторизации, вы сможете использовать его для получения строки подключения для очереди.

```azurecli
# Create the authorization rule for the Service Bus queue.
az servicebus queue authorization-rule create \
  --name "sbauthrule" \
  --namespace-name $sbNamespace \
  --queue-name $sbQueueName \
  --resource-group $resourceGroup \
  --rights Listen Manage Send \
  --subscription $subscriptionID
```

Теперь с помощью правила авторизации можно получить строку подключения для очереди служебной шины.

```azurecli
# Get the Service Bus queue connection string.
# The "-o tsv" ensures it is returned without the default double-quotes.
sbqConnectionString=$(az servicebus queue authorization-rule keys list \
  --name "sbauthrule" \
  --namespace-name $sbNamespace \
  --queue-name $sbQueueName \
  --resource-group $resourceGroup \
  --subscription $subscriptionID \
  --query primaryConnectionString -o tsv)

# Show the Service Bus queue connection string.
echo "service bus queue connection string = " $sbqConnectionString
```

Настройте конечную точку маршрутизации и маршрут сообщений для очереди служебной шины. Это переменные, используемые скриптом, которые необходимо задать в сеансе Cloud Shell:

**endpointName**. В этом поле указывается имя, определяющее конечную точку. 

**endpointType**. Здесь указывается тип конечной точки. Это значение должно быть `azurestoragecontainer`, `eventhub`, `servicebusqueue` или `servicebustopic`. В нашем примере используется значение `servicebusqueue`.

**routeName**. В этом поле указывается имя маршрута, который вы настраиваете. 

**condition**. В этом поле указывается запрос для фильтрации сообщений, отправляемых в эту конечную точку. Условие запроса для сообщений, маршрутизируемых в очередь служебной шины, — `level="critical"`.

Ниже представлен скрипт Azure CLI для конечной точки маршрутизации и маршрута сообщений для очереди служебной шины.

```azurecli
endpointName="ContosoSBQueueEndpoint"
endpointType="ServiceBusQueue"
routeName="ContosoSBQueueRoute"
condition='level="critical"'

# Set up the routing endpoint for the Service Bus queue.
# This uses the Service Bus queue connection string.
az iot hub routing-endpoint create \
  --connection-string $sbqConnectionString \
  --endpoint-name $endpointName \
  --endpoint-resource-group $resourceGroup \
  --endpoint-subscription-id $subscriptionID \
  --endpoint-type $endpointType \
  --hub-name $iotHubName \
  --resource-group $resourceGroup 

# Set up the message route for the Service Bus queue endpoint.
az iot hub route create --name $routeName \
  --hub-name $iotHubName \
  --source-type devicemessages \
  --resource-group $resourceGroup \
  --endpoint-name $endpointName \
  --enabled \
  --condition $condition
  ```

### <a name="view-message-routing-in-the-portal"></a>Просмотр маршрутизации сообщений на портале

[!INCLUDE [iot-hub-include-view-routing-in-portal](../../includes/iot-hub-include-view-routing-in-portal.md)]

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы настроили ресурсы и маршрутизацию сообщений, переходите к следующему руководству, чтобы узнать, как отправлять сообщения в центр Интернета вещей, маршрутизируя их в разные места назначения. 

> [!div class="nextstepaction"]
> [Часть 2. Просмотр результатов маршрутизации сообщений](tutorial-routing-view-message-routing-results.md)
