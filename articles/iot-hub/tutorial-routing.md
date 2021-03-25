---
title: Руководство. Настройка маршрутизации сообщений для Центра Интернета вещей Azure с помощью Azure CLI
description: Руководство по настройке маршрутизации сообщений для Центра Интернета вещей Azure с помощью Azure CLI и портала Azure.
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: tutorial
ms.date: 03/12/2019
ms.author: robinsh
ms.custom:
- mvc
- 'Role: Cloud Development'
- 'Role: Data Analytics'
- devx-track-azurecli
ms.openlocfilehash: d7a10aac82286616273f7283bc3fba74f2c58f35
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98625220"
---
# <a name="tutorial-use-the-azure-cli-and-azure-portal-to-configure-iot-hub-message-routing"></a>Руководство по Использование Azure CLI и портала Azure для настройки маршрутизации сообщений Центра Интернета вещей

[!INCLUDE [iot-hub-include-routing-intro](../../includes/iot-hub-include-routing-intro.md)]

[!INCLUDE [iot-hub-include-routing-create-resources](../../includes/iot-hub-include-routing-create-resources.md)]

## <a name="use-the-azure-cli-to-create-the-base-resources"></a>Использование Azure CLI для создания базовых ресурсов

В этом руководстве с помощью Azure CLI создаются базовые ресурсы, а затем показано, как [портал Azure](https://portal.azure.com) используется для настройки маршрутизации сообщений и настройки виртуального устройства для тестирования.

Скопируйте и вставьте приведенный ниже скрипт в Cloud Shell и нажмите клавишу ВВОД. Скрипт будет выполняться построчно. Скрипт создает базовые ресурсы для работы с этим руководством, включая учетную запись хранения, Центр Интернета вещей, пространство имен служебной шины и очереди служебной шины.

Некоторым ресурсам необходимо присвоить уникальные имена, включая имя Центра Интернета вещей и имя учетной записи хранения. Чтобы упростить эту задачу, к именам этих ресурсов добавляется случайное буквенно-цифровое значение *randomValue*. Значение randomValue создается один раз в верхней части скрипта и повсеместно добавляется к соответствующим именам ресурсов в пределах скрипта. Если вы не хотите, чтобы это значение было случайным, можно заменить его пустой строкой или определенным значением.

> [!TIP]
> Замечание относительно отладки: для удобочитаемости этот скрипт использует символ продолжения (обратная косая черта `\`). Если при выполнении скрипта возникла проблема, убедитесь, что сеанс Cloud Shell работает c использованием `bash` и что после символов обратной косой черты нет пробелов.
>

```azurecli-interactive
# This retrieves the subscription id of the account 
#   in which you're logged in.
# This field is used to set up the routing queries.
subscriptionID=$(az account show --query id)

# Concatenate this number onto the resources that have to be globally unique.
# You can set this to "" or to a specific value if you don't want it to be random.
# This retrieves a random value.
randomValue=$RANDOM

# Set the values for the resource names that 
#   don't have to be globally unique.
location=westus
resourceGroup=ContosoResources
iotHubConsumerGroup=ContosoConsumers
containerName=contosoresults

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

```

Теперь, когда базовые ресурсы готовы, можно настроить маршрутизацию сообщений на [портале Azure](https://portal.azure.com).

## <a name="set-up-message-routing"></a>Настройка маршрутизации сообщений

[!INCLUDE [iot-hub-include-create-routing-description](../../includes/iot-hub-include-create-routing-description.md)]

### <a name="route-to-a-storage-account"></a>Маршрутизация в учетную запись хранения

Теперь настройте маршрутизацию для учетной записи хранения. Перейдите в панель маршрутизации сообщений, а затем добавьте маршрут. При добавлении маршрута определите новую конечную точку для этого маршрута. После настройки сообщения, свойство **level** (уровень) которых имеет значение **storage** (хранение), автоматически записываются в учетную запись хранения. 

[!INCLUDE [iot-hub-include-blob-storage-format](../../includes/iot-hub-include-blob-storage-format.md)]

1. На [портале Azure](https://portal.azure.com) щелкните **Группы ресурсов** и выберите группу ресурсов. В этом руководстве используется **ContosoResources**.

2. Выберите центр Интернета вещей в списке ресурсов. В этом руководстве используется **ContosoTestHub**.

3. Выберите **Маршрутизация сообщений**. В области **Маршрутизация сообщений** выберите +**Добавить**. В области **Добавление маршрута** выберите **+Добавить конечную точку** рядом с полем "Конечная точка", чтобы появились поддерживаемые конечные точки, как показано на следующем рисунке:

   ![Добавление конечной точки для маршрута](./media/tutorial-routing/message-routing-add-a-route-with-storage-endpoint-ver2.png)

4. Выберите **Хранилище**. Затем вы увидите панель **Добавление конечной точки хранилища**.

   ![Добавление конечной точки](./media/tutorial-routing/message-routing-add-storage-endpoint-ver2.png)

5. Введите имя для конечной точки. В этом руководстве используется **ContosoStorageEndpoint**.

6. Щелкните **Выберите контейнер**. Появится список ваших учетных записей хранения. Выберите ту, которую настраивали на этапе подготовки. В этом руководстве используется **contosostorage**. Появится список контейнеров в этой учетной записи хранения. **Выберите** контейнер, настроенный на этапе подготовки. В этом руководстве используется **contosoresults**. Вы вернетесь в область **Добавление конечной точки хранилища**, где можно просмотреть заданные настройки.

7. Укажите кодировку — AVRO или JSON. При работе с этим руководством можно использовать значения по умолчанию для остальных полей. Это поле будет неактивно, если для выбранного региона не поддерживается кодировка JSON.

   > [!NOTE]
   > Вы можете задать **формат имени файла больших двоичных объектов**. Значение по умолчанию — `{iothub}/{partition}/{YYYY}/{MM}/{DD}/{HH}/{mm}`. Формат должен содержать элементы {iothub}, {partition}, {YYYY}, {MM}, {DD}, {HH} и {mm} в любом порядке.
   >
   > Например, если вы используете формат имени файла больших двоичных объектов по умолчанию и имя центра — ContosoTestHub, а дата и временя — 30 октября 2018 г 10:56, имя большого двоичного объекта будет выглядеть так: `ContosoTestHub/0/2018/10/30/10/56`.
   > 
   > Большие двоичные объекты, записываются в формате Avro.
   >

8. Нажмите кнопку **Создать**, чтобы создать конечную точку хранилища и добавить ее в маршрут. Вернитесь в панель **Добавление маршрута**.

9. Теперь заполните остальные сведения запроса маршрутизации. Этот запрос задает условия для отправки сообщений в контейнер хранилища, который вы только что добавили в качестве конечной точки. Заполните поля на экране.

   **Name** (Имя). Введите имя для вашего запроса маршрутизации. В этом руководстве используется **ContosoStorageRoute**.

   **Конечная точка**. Здесь отображается только что настроенная конечная точка.

   **Источник данных**: В раскрывающемся списке выберите **Сообщения телеметрии устройства**.

   **Включить маршрут.** Убедитесь, что это поле содержит значение `enabled`.
   
   **Маршрутизация запроса.** Введите `level="storage"` как строку запроса.

   ![Создание запроса маршрутизации для учетной записи хранения](./media/tutorial-routing/message-routing-finish-route-storage-ep.png)  

   Щелкните **Сохранить**. После завершения откроется панель маршрутизации сообщений, где можно просмотреть новый запрос маршрутизации для хранения. Закрыв панель маршрутов, возвратитесь на страницу группы ресурсов.

### <a name="route-to-a-service-bus-queue"></a>Маршрутизация в очередь служебной шины

Теперь настройте маршрутизацию для очереди служебной шины. Перейдите в панель маршрутизации сообщений, а затем добавьте маршрут. При добавлении маршрута определите новую конечную точку для этого маршрута. После такой настройки сообщения, свойство **level** (уровень) которых имеет значение **critical** (критический), записываются в очередь служебной шины, которая запускает приложение логики. Затем это приложение логики отправляет электронное сообщение с информацией.

1. На странице группы "Ресурс" выберите центр Интернета вещей, а затем **Маршрутизация сообщений**.

2. В области **Маршрутизация сообщений** выберите +**Добавить**.

3. В области **Добавление маршрута** нажмите кнопку +**Добавить** рядом с полем "Конечная точка". Выберите **Очередь служебной шины**. Появится панель **Добавление конечной точки служебной шины**.

   ![Добавление конечной точки служебной шины](./media/tutorial-routing/message-routing-add-sbqueue-ep.png)

4. Заполните следующие поля.

   **Имя конечной точки.** Введите имя для конечной точки. В этом руководстве используется **ContosoSBQueueEndpoint**.
   
   **Пространство имен служебной шины.** В раскрывающемся списке выберите пространство имен служебной шины, которая настраивается на этапах подготовки. В этом руководстве используется **ContosoSBNamespace**.

   **Очередь служебной шины.** В раскрывающемся списке выберите очередь служебной шины. В этом руководстве используется **contososbqueue**.

5. Нажмите кнопку **Создать**, чтобы добавить конечную точку очереди служебной шины. Вернитесь в панель **Добавление маршрута**.

6. Теперь заполните остальные сведения запроса маршрутизации. Этот запрос задает условия для отправки сообщений в очередь служебной шины, которую вы только что добавили в качестве конечной точки. Заполните поля на экране. 

   **Name** (Имя). Введите имя для вашего запроса маршрутизации. В этом руководстве используется **ContosoSBQueueRoute**. 

   **Конечная точка**. Здесь отображается только что настроенная конечная точка.

   **Источник данных**: В раскрывающемся списке выберите **Сообщения телеметрии устройства**.

   **Маршрутизация запроса.** Введите `level="critical"` как строку запроса. 

   ![Создание запроса маршрутизации для очереди служебной шины](./media/tutorial-routing/message-routing-finish-route-sbq-ep.png)

7. Щелкните **Сохранить**. Как показано здесь, при возвращении в область маршрутов видно оба новых маршрута.

   ![Настроенные маршруты](./media/tutorial-routing/message-routing-show-both-routes.png)

8. Чтобы увидеть настроенные вами пользовательские конечные точки, перейдите на вкладку **Пользовательские конечные точки**.

   ![Настроенная пользовательская конечная точка](./media/tutorial-routing/message-routing-show-custom-endpoints.png)

9. Закройте панель маршрутизации сообщений и вернитесь в панель группы ресурсов.

## <a name="create-a-simulated-device"></a>Создайте виртуальное устройство.

[!INCLUDE [iot-hub-include-create-simulated-device-portal](../../includes/iot-hub-include-create-simulated-device-portal.md)]

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы настроили ресурсы и маршрутизацию сообщений, переходите к следующему руководству, чтобы узнать, как отправлять сообщения в центр Интернета вещей, маршрутизируя их в разные места назначения. 

> [!div class="nextstepaction"]
> [Часть 2. Просмотр результатов маршрутизации сообщений](tutorial-routing-view-message-routing-results.md)