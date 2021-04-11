---
title: Руководство. Использование обогащений сообщений в Центре Интернета вещей Azure
description: В этом руководстве описывается, как использовать обогащения для сообщений Центра Интернета вещей Azure.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: tutorial
ms.date: 12/20/2019
ms.author: robinsh
ms.custom: mqtt, devx-track-azurecli, devx-track-csharp
ms.openlocfilehash: bd047419275d98049f883bb88a83708a9dfdd7d3
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106066897"
---
# <a name="tutorial-use-azure-iot-hub-message-enrichments"></a>Руководство. Использование обогащений сообщений в Центре Интернета вещей Azure

*Обогащение сообщений* означает возможность Центра Интернета вещей Azure *помечать* сообщения, добавляя к ним дополнительные сведения, перед их отправкой в целевую конечную точку. Одной из причин для использования обогащений сообщений является включение данных, с помощью которых можно упростить последующую обработку. Например, обогащение сообщений телеметрии устройства с включением тега двойника устройства позволяет уменьшить нагрузку для клиентов путем отправки вызовов API двойнику устройства для получения этой информации. Дополнительные сведения см. в статье [Общие сведения об обогащениях сообщений](iot-hub-message-enrichments-overview.md).

В этом руководстве приведены два способа создания и настройки ресурсов, необходимых для тестирования обогащений сообщений в центре Интернета вещей. Такие ресурсы включают одну учетную запись хранения с двумя контейнерами учетной записи хранения. Один контейнер содержит обогащенные сообщения, а другой — исходные сообщения. Кроме того, центр Интернета вещей принимает сообщения и перенаправляет их в соответствующий контейнер учетной записи хранения с учетом того, обогащены они или нет.

* Первый способ заключается в использовании Azure CLI для создания ресурсов и настройки маршрутизации сообщений. Затем вы определите обогащения вручную на [портале Azure](https://portal.azure.com).

* Второй способ заключается в использовании шаблона ARM для создания ресурсов *и* конфигураций для маршрутизации и обогащений сообщений.

После создания конфигураций для маршрутизации и обогащения сообщений вы воспользуетесь приложением для отправки сообщений в центр Интернета вещей. Центр затем направит их в оба контейнера учетной записи хранения. Обогащаются только те сообщения, которые отправляются в конечную точку для контейнера **enriched** учетной записи хранения.

Ниже приведены задачи, которые вы выполните при работе с этим руководством:

**Использование обогащений сообщений в Центре Интернета вещей**
> [!div class="checklist"]
> * Первый способ: создание ресурсов и настройка маршрутизации сообщений с помощью Azure CLI. Вы можете настроить обогащение сообщений вручную с помощью [портала Azure](https://portal.azure.com).
> * Второй способ: создание ресурсов и настройка маршрутизации и обогащений сообщений с помощью шаблона ARM. 
> * Запуск приложения, которое имитирует устройство Интернета вещей, отправляющее сообщения в центр.
> * Просмотр результатов и проверка того, что обогащения сообщений работают правильно.

## <a name="prerequisites"></a>Предварительные требования

- У вас должна быть подписка Azure. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

- Установить [Visual Studio](https://www.visualstudio.com/).

- Убедитесь, что в брандмауэре открыт порт 8883. Пример устройства в этом руководстве использует протокол MQTT, который передает данные через порт 8883. В некоторых корпоративных и академических сетях этот порт может быть заблокирован. Дополнительные сведения и способы устранения этой проблемы см. в разделе о [подключении к Центру Интернета вещей по протоколу MQTT](iot-hub-mqtt-support.md#connecting-to-iot-hub).

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

## <a name="retrieve-the-iot-c-samples-repository"></a>Извлечение репозитория с примерами C# для Интернета вещей

Скачайте [примеры C# для Интернета вещей](https://github.com/Azure-Samples/azure-iot-samples-csharp/archive/master.zip) с GitHub и распакуйте их. Этот репозиторий содержит несколько приложений, скриптов и шаблонов ARM. Ниже приведены те из них, которые будут использоваться в этом руководстве:

* Для способа с выполнением вручную доступен скрипт CLI, который используется для создания ресурсов. Этот скрипт находится здесь: /azure-iot-samples-csharp/iot-hub/Tutorials/Routing/SimulatedDevice/resources/iothub_msgenrichment_cli.azcli. Он создает ресурсы и настраивает маршрутизацию сообщений. После выполнения скрипта создайте обогащения сообщений вручную с помощью [портала Azure](https://portal.azure.com).
* Для способа с автоматическим выполнением доступен шаблон ARM. Он находится здесь: /azure-iot-samples-csharp/iot-hub/Tutorials/Routing/SimulatedDevice/resources/template_msgenrichments.json. Этот шаблон создает ресурсы, настраивает маршрутизацию сообщений, а затем настраивает обогащения сообщений.
* Третье приложение, которое вы будете использовать, — это приложение имитации устройства, которое используется для отправки сообщений в центр Интернета вещей и тестирования обогащений сообщений.

## <a name="manually-set-up-and-configure-by-using-the-azure-cli"></a>Настройка вручную с помощью Azure CLI

Кроме создания необходимых ресурсов, скрипт Azure CLI также настраивает два маршрута к конечным точкам, которые являются отдельными контейнерами учетной записи хранения. Подробные сведения о настройке маршрутизации сообщений см. в [руководстве по маршрутизации](tutorial-routing.md). После настройки ресурсов воспользуйтесь [порталом Azure](https://portal.azure.com) для настройки обогащений сообщений для каждой конечной точки. Затем перейдите к тестированию.

> [!NOTE]
> Все сообщения направляются в обе конечные точки, но обогащаться будут только те сообщения, которые передаются в конечную точку с настроенными обогащениями сообщений.
>

Вы можете использовать приведенный ниже скрипт или открыть скрипт в папке /resources скачанного репозитория. Скрипт выполняет следующие задачи.

* Создайте Центр Интернета вещей.
* Создайте учетную запись хранения.
* Создает два контейнера в учетной записи хранения. Один контейнер предназначен для обогащенных сообщений, а другой — для сообщений, которые не были обогащены.
* Настраивает маршрутизацию для двух разных учетных записей хранения:
    * Создает конечную точку для каждого контейнера учетной записи хранения.
    * Создает маршрут к каждой из конечных точек контейнера учетной записи хранения.

Некоторым ресурсам необходимо присвоить уникальные имена, включая имя центра Интернета вещей и имя учетной записи хранения. Чтобы упростить выполнение скрипта, к именам этих ресурсов добавляется случайное буквенно-цифровое значение *randomValue*. Случайное значение создается один раз в начале скрипта. Оно добавляется к именам ресурсов по мере необходимости во время выполнения скрипта. Если вы не хотите, чтобы это значение было случайным, можно заменить его пустой строкой или определенным значением.

Если вы еще этого не сделали, откройте окно Azure [Cloud Shell](https://shell.azure.com) и убедитесь, что там используется Bash. Откройте скрипт в распакованном файле репозитория, нажмите клавиши CTRL+A, чтобы выбрать все содержимое, и нажмите CTRL+C, чтобы скопировать его. Кроме того, вы можете скопировать следующий скрипт CLI или открыть его непосредственно в Cloud Shell. Вставьте скрипт в окно Cloud Shell, щелкнув командную строку правой кнопкой мыши и выбрав **Вставить**. Скрипт выполняет инструкции поочередно. После завершения выполнения скрипта нажмите **ВВОД**, чтобы убедиться, что он выполнил последнюю команду. В следующем блоке кода приведен используемый скрипт с поясняющими комментариями.

Ниже приведены ресурсы, создаваемые скриптом. *Enriched* означает, что ресурс предназначен для сообщений с обогащениями. *Original* означает, что ресурс предназначен для сообщений без обогащений.

| Имя | Значение |
|-----|-----|
| resourceGroup | ContosoResourcesMsgEn |
| container name | оригинальный  |
| container name | enriched  |
| IoT device name | Contoso-Test-Device |
| Имя Центра Интернета вещей | ContosoTestHubMsgEn |
| storage Account Name | contosostorage |
| endpoint Name 1 | ContosoStorageEndpointOriginal |
| endpoint Name 2 | ContosoStorageEndpointEnriched|
| route Name 1 | ContosoStorageRouteOriginal |
| route Name 2 | ContosoStorageRouteEnriched |

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
location=westus2
resourceGroup=ContosoResourcesMsgEn
containerName1=original
containerName2=enriched
iotDeviceName=Contoso-Test-Device

# Create the resource group to be used
#   for all the resources for this tutorial.
az group create --name $resourceGroup \
    --location $location

# The IoT hub name must be globally unique,
#   so add a random value to the end.
iotHubName=ContosoTestHubMsgEn$randomValue
echo "IoT hub name = " $iotHubName

# Create the IoT hub.
az iot hub create --name $iotHubName \
    --resource-group $resourceGroup \
    --sku S1 --location $location

# You need a storage account that will have two containers
#   -- one for the original messages and
#   one for the enriched messages.
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
#    You need this to create the containers.
storageAccountKey=$(az storage account keys list \
    --resource-group $resourceGroup \
    --account-name $storageAccountName \
    --query "[0].value" | tr -d '"')

# See the value of the storage account key.
echo "storage account key = " $storageAccountKey

# Create the containers in the storage account.
az storage container create --name $containerName1 \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --public-access off

az storage container create --name $containerName2 \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --public-access off

# Create the IoT device identity to be used for testing.
az iot hub device-identity create --device-id $iotDeviceName \
    --hub-name $iotHubName

# Retrieve the information about the device identity, then copy the primary key to
#   Notepad. You need this to run the device simulation during the testing phase.
# If you are using Cloud Shell, you can scroll the window back up to retrieve this value.
az iot hub device-identity show --device-id $iotDeviceName \
    --hub-name $iotHubName

##### ROUTING FOR STORAGE #####

# You're going to have two routes and two endpoints.
# One route points to the first container ("original") in the storage account
#   and includes the original messages.
# The other points to the second container ("enriched") in the same storage account
#   and includes the enriched versions of the messages.

endpointType="azurestoragecontainer"
endpointName1="ContosoStorageEndpointOriginal"
endpointName2="ContosoStorageEndpointEnriched"
routeName1="ContosoStorageRouteOriginal"
routeName2="ContosoStorageRouteEnriched"

# for both endpoints, retrieve the messages going to storage
condition='level="storage"'

# Get the connection string for the storage account.
# Adding the "-o tsv" makes it be returned without the default double quotes around it.
storageConnectionString=$(az storage account show-connection-string \
  --name $storageAccountName --query connectionString -o tsv)

# Create the routing endpoints and routes.
# Set the encoding format to either avro or json.

# This is the endpoint for the first container, for endpoint messages that are not enriched.
az iot hub routing-endpoint create \
  --connection-string $storageConnectionString \
  --endpoint-name $endpointName1 \
  --endpoint-resource-group $resourceGroup \
  --endpoint-subscription-id $subscriptionID \
  --endpoint-type $endpointType \
  --hub-name $iotHubName \
  --container $containerName1 \
  --resource-group $resourceGroup \
  --encoding json

# This is the endpoint for the second container, for endpoint messages that are enriched.
az iot hub routing-endpoint create \
  --connection-string $storageConnectionString \
  --endpoint-name $endpointName2 \
  --endpoint-resource-group $resourceGroup \
  --endpoint-subscription-id $subscriptionID \
  --endpoint-type $endpointType \
  --hub-name $iotHubName \
  --container $containerName2 \
  --resource-group $resourceGroup \
  --encoding json

# This is the route for messages that are not enriched.
# Create the route for the first storage endpoint.
az iot hub route create \
  --name $routeName1 \
  --hub-name $iotHubName \
  --source devicemessages \
  --resource-group $resourceGroup \
  --endpoint-name $endpointName1 \
  --enabled \
  --condition $condition

# This is the route for messages that are enriched.
# Create the route for the second storage endpoint.
az iot hub route create \
  --name $routeName2 \
  --hub-name $iotHubName \
  --source devicemessages \
  --resource-group $resourceGroup \
  --endpoint-name $endpointName2 \
  --enabled \
  --condition $condition
```

На этом этапе все ресурсы и маршрутизация сообщений настроены. Вы можете просмотреть конфигурацию маршрутизации сообщений на портале и настроить обогащения для сообщений, передаваемых в контейнер **enriched** учетной записи хранения.

### <a name="manually-configure-the-message-enrichments-by-using-the-azure-portal"></a>Настройка обогащений сообщений вручную с помощью портала Azure

1. Перейдите в центр Интернета вещей, выбрав **Группы ресурсов**. Затем выберите группу ресурсов, настроенную для этого руководства (**ContosoResourcesMsgEn**). Найдите в списке центр Интернета вещей и выберите его. Выберите **Маршрутизация сообщений** для центра Интернета вещей.

   ![Выбор маршрутизации сообщений](./media/tutorial-message-enrichments/select-iot-hub.png)

   В области маршрутизации сообщений доступно три вкладки **Маршруты**, **Пользовательские конечные точки** и **Обогащение сообщений**. Откройте первые две вкладки, чтобы просмотреть конфигурацию, настроенную скриптом. На третьей вкладке добавьте обогащения сообщений. Мы выполним обогащение сообщений, направляемых в конечную точку для контейнера **enriched** учетной записи хранения. Введите имя и значение, а затем выберите конечную точку **ContosoStorageEndpointEnriched** в раскрывающемся списке. Ниже приведен пример того, как настроить обогащение, которое добавляет имя центра Интернета вещей к сообщению:

   ![Добавление первого обогащения](./media/tutorial-message-enrichments/add-message-enrichments.png)

2. Добавьте эти значения в список для конечной точки ContosoStorageEndpointEnriched.

   | Ключ | Значение | Конечная точка (раскрывающийся список) |
   | ---- | ----- | -------------------------|
   | myIotHub | $iothubname | AzureStorageContainers > ContosoStorageEndpointEnriched |
   | DeviceLocation | $twin.tags.location | AzureStorageContainers > ContosoStorageEndpointEnriched |
   |customerID | 6ce345b8-1e4a-411e-9398-d34587459a3a | AzureStorageContainers > ContosoStorageEndpointEnriched |

   > [!NOTE]
   > Если для устройства отсутствует двойник, введенное здесь значение будет помечено как строка для значения в обогащениях сообщений. Чтобы просмотреть сведения о двойнике устройства, перейдите к своему центру на портале и выберите **Устройства Интернета вещей**. Выберите свое устройство и щелкните **Двойник устройства** вверху страницы.
   >
   > Вы можете изменить сведения о двойнике, чтобы добавить теги (например, с данными о расположении), и задать для них конкретное значение. См. [общие сведения о двойниках устройств и их использовании в Центре Интернета вещей](iot-hub-devguide-device-twins.md).

3. По завершении ваша область должна выглядеть примерно вот так:

   ![Таблица со всеми добавленными обогащениями](./media/tutorial-message-enrichments/all-message-enrichments.png)

4. Выберите **Применить**, чтобы сохранить изменения. Перейдите к разделу [Протестировать обогащения сообщений](#test-message-enrichments).

## <a name="create-and-configure-by-using-a-resource-manager-template"></a>Создание и настройка с помощью шаблона ARM
Вы можете воспользоваться шаблоном ARM, чтобы создать и настроить ресурсы, маршрутизацию сообщений и обогащения сообщений.

1. Войдите на портал Azure. Выберите **Создать ресурс**, чтобы отобразить поле поиска. Введите *шаблон развертывания* и выполните поиск шаблона. В области результатов выберите **Развертывание шаблона (развертывание с помощью настраиваемого шаблона)** .

   ![Развертывание шаблона на портале Azure](./media/tutorial-message-enrichments/template-select-deployment.png)

1. Выберите **Создать** в области **Развертывание шаблона**.

1. В области **Настраиваемое развертывание** выберите **Создать собственный шаблон в редакторе**.

1. В области **Изменение шаблона** выберите **Загрузить файл**. Отобразится Проводник Windows. Найдите файл **template_messageenrichments.json** в распакованном файле репозитория в папке **/iot-hub/Tutorials/Routing/SimulatedDevice/resources**. 

   ![Выбор шаблона на локальном компьютере](./media/tutorial-message-enrichments/template-select.png)

1. Выберите **Открыть**, чтобы загрузить файл шаблона с локального компьютера. Он будет загружен и отобразится в области редактирования.

   Этот шаблон настроен для использования глобально уникального имени центра Интернета вещей и имени учетной записи хранения путем добавления случайного значения в конец имен по умолчанию. Поэтому мы вы можете использовать шаблон без внесения в него изменений.

   Ниже приведены ресурсы, создаваемые при загрузке шаблона. **Enriched** означает, что ресурс предназначен для сообщений с обогащениями. **Original** означает, что ресурс предназначен для сообщений без обогащений. Это те же значения, которые используются в скрипте Azure CLI.

   | Имя | Значение |
   |-----|-----|
   | resourceGroup | ContosoResourcesMsgEn |
   | container name | оригинальный  |
   | container name | enriched  |
   | IoT device name | Contoso-Test-Device |
   | Имя Центра Интернета вещей | ContosoTestHubMsgEn |
   | storage Account Name | contosostorage |
   | endpoint Name 1 | ContosoStorageEndpointOriginal |
   | endpoint Name 2 | ContosoStorageEndpointEnriched|
   | route Name 1 | ContosoStorageRouteOriginal |
   | route Name 2 | ContosoStorageRouteEnriched |

1. Выберите **Сохранить**. Отобразится область **Настраиваемое развертывание** со всеми параметрами, используемыми шаблоном. Вам нужно только задать поле **Группа ресурсов**. Создайте новый ресурс или выберите его в раскрывающемся списке.

   Ниже приведена верхняя половина области **Настраиваемое развертывание**. Вы можете видеть, где вводятся данные о группе ресурсов.

   ![Верхняя половина области "Настраиваемое развертывание"](./media/tutorial-message-enrichments/template-deployment-top.png)

1. Ниже приведена нижняя половина области **Настраиваемое развертывание**. Вы можете видеть другие параметры и условия. 

   ![Нижняя половина области "Настраиваемое развертывание"](./media/tutorial-message-enrichments/template-deployment-bottom.png)

1. Установите флажок, чтобы принять условия. Затем выберите **Приобрести**, чтобы продолжить развертывание шаблона.

1. Дождитесь полного развертывания шаблона. Щелкните значок колокольчика в верхней части экрана, чтобы проверить ход выполнения. После его завершения перейдите к разделу [Тестирование обогащений сообщений](#test-message-enrichments).

## <a name="test-message-enrichments"></a>Тестирование обогащений сообщений

Чтобы просмотреть обогащения сообщений, выберите **Группы ресурсов**. Затем выберите группу ресурсов, используемую при работе с этим руководством. Выберите центр Интернета вещей в списке ресурсов и перейдите в раздел **Сообщения**. Отобразится конфигурация маршрутизации сообщений и настроенные обогащения.

Теперь, когда обогащения сообщений настроены для конечной точки, выполните приложение имитированного устройства, чтобы отправить сообщения в центр Интернета вещей. Центр настроен с параметрами, которые выполняют следующие задачи:

* Сообщения, направляемые в конечную точку службы хранилища ContosoStorageEndpointOriginal, не будут обогащаться и будут сохраняться в контейнере учетной записи хранения `original`.

* Сообщения, направляемые в конечную точку службы хранилища ContosoStorageEndpointEnriched, будут обогащаться и будут сохраняться в контейнере учетной записи хранения `enriched`.

Приложение имитированного устройства — это одно из приложений из скачанного распакованного файла. Приложение отправляет сообщения для каждого из методов маршрутизации сообщений, приведенных в [руководстве по маршрутизации](tutorial-routing.md), который включает службу хранилища Azure.

Дважды щелкните файл решения **IoT_SimulatedDevice.sln**, чтобы открыть код в Visual Studio, и откройте файл **Program.cs**. Замените имя центра Интернета вещей маркером `{your hub name}`. Формат имени узла центра Интернета вещей **{имя центра}.azure-devices.net**. В этом руководстве используется имя узла центра ContosoTestHubMsgEn.azure-devices.net. Затем замените ключ устройства, сохраненный ранее при выполнении скрипта для создания ресурсов, маркером `{your device key}`.

Если у вас нет ключа устройства, его можно получить на портале. После входа перейдите в раздел **Группы ресурсов**, выберите группу ресурсов, а затем выберите центр Интернета вещей. Найдите в разделе **Устройства Интернета вещей** свое тестовое устройство и выберите его. Выберите значок копирования рядом с элементом **Первичный ключ**, чтобы скопировать его в буфер обмена.

   ```csharp
        private readonly static string s_myDeviceId = "Contoso-Test-Device";
        private readonly static string s_iotHubUri = "ContosoTestHubMsgEn.azure-devices.net";
        // This is the primary key for the device. This is in the portal.
        // Find your IoT hub in the portal > IoT devices > select your device > copy the key.
        private readonly static string s_deviceKey = "{your device key}";
   ```

### <a name="run-and-test"></a>Запуск и тестирование

Запустите консольное приложение на несколько минут. Отправляемые сообщения будут отображаться на экране консоли приложения.

Приложение отправляет новое сообщение с устройства в облако к Центру IoT каждую секунду. Сообщение содержит JSON-сериализованный объект с идентификатором устройства, температурой, влажностью и уровнем сообщений, значение которого по умолчанию `normal`. Он случайным образом назначает уровень `critical` или `storage`, что приводит к маршрутизации сообщения в учетную запись хранения или конечную точку по умолчанию. Сообщения, отправленные в контейнер **enriched** в учетной записи хранения, будут обогащены.

После отправки нескольких сообщений учетной записи хранения просмотрите данные.

1. Выберите элемент **Группы ресурсов**. Найдите свою группу ресурсов, **ContosoResourcesMsgEn**, и выберите ее.

2. Выберите свою учетную запись хранения — **contosostorage**. Затем выберите **Обозреватель службы хранилища (предварительная версия)** в области слева.

   ![Выбор Обозревателя службы хранилища](./media/tutorial-message-enrichments/select-storage-explorer.png)

   Выберите **Контейнеры BLOB-объектов**, чтобы просмотреть два доступные для использования контейнера.

   ![Просмотр контейнеров в учетной записи хранения](./media/tutorial-message-enrichments/show-blob-containers.png)

Сообщения в контейнере с именем **enriched** содержат включенные обогащения. Сообщения в контейнере с именем **original** не имеют обогащений. Раскройте содержимое контейнера до нижнего уровня и откройте последний файл сообщения. Затем сделайте то же самое для другого контейнера, чтобы убедиться в том, что в его сообщения не добавлены обогащения.

Если просмотреть обогащенные сообщения, можно увидеть поле my IoT Hub с именем центра, расположением и идентификатором клиентам, например:

```json
{"EnqueuedTimeUtc":"2019-05-10T06:06:32.7220000Z","Properties":{"level":"storage","my IoT Hub":"contosotesthubmsgen3276","devicelocation":"$twin.tags.location","customerID":"6ce345b8-1e4a-411e-9398-d34587459a3a"},"SystemProperties":{"connectionDeviceId":"Contoso-Test-Device","connectionAuthMethod":"{\"scope\":\"device\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}","connectionDeviceGenerationId":"636930642531278483","enqueuedTime":"2019-05-10T06:06:32.7220000Z"},"Body":"eyJkZXZpY2VJZCI6IkNvbnRvc28tVGVzdC1EZXZpY2UiLCJ0ZW1wZXJhdHVyZSI6MjkuMjMyMDE2ODQ4MDQyNjE1LCJodW1pZGl0eSI6NjQuMzA1MzQ5NjkyODQ0NDg3LCJwb2ludEluZm8iOiJUaGlzIGlzIGEgc3RvcmFnZSBtZXNzYWdlLiJ9"}
```

Ниже приведено необогащенное сообщение. Обратите внимание, что поля my IoT Hub, devicelocation и customerID отсутствуют здесь, так как они добавляются обогащениями. Эта конечная точка не имеет обогащений.

```json
{"EnqueuedTimeUtc":"2019-05-10T06:06:32.7220000Z","Properties":{"level":"storage"},"SystemProperties":{"connectionDeviceId":"Contoso-Test-Device","connectionAuthMethod":"{\"scope\":\"device\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}","connectionDeviceGenerationId":"636930642531278483","enqueuedTime":"2019-05-10T06:06:32.7220000Z"},"Body":"eyJkZXZpY2VJZCI6IkNvbnRvc28tVGVzdC1EZXZpY2UiLCJ0ZW1wZXJhdHVyZSI6MjkuMjMyMDE2ODQ4MDQyNjE1LCJodW1pZGl0eSI6NjQuMzA1MzQ5NjkyODQ0NDg3LCJwb2ludEluZm8iOiJUaGlzIGlzIGEgc3RvcmFnZSBtZXNzYWdlLiJ9"}
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы удалить все ресурсы, которые были созданы в этом руководстве, удалите группу ресурсов. При этом будут также удалены все ресурсы, содержащиеся в группе. В этом случае происходит удаление центра Интернета вещей, учетной записи хранения и самой группы ресурсов.

### <a name="use-the-azure-cli-to-clean-up-resources"></a>Очистка ресурсов с помощью Azure CLI

Чтобы удалить группу ресурсов, используйте команду [az group delete](/cli/azure/group#az-group-delete). Помните, что `$resourceGroup` было присвоено значение **ContosoResourcesMsgEn** в начале работы с этим руководством.

```azurecli-interactive
az group delete --name $resourceGroup
```

## <a name="next-steps"></a>Следующие шаги

Из этого руководства вы узнали, как настроить и протестировать добавление обогащений сообщений в сообщения Центра Интернета вещей с помощью следующих действий:

**Использование обогащений сообщений в Центре Интернета вещей**

> [!div class="checklist"]
> * Первый способ: создание ресурсов и настройка маршрутизации сообщений с помощью Azure CLI. Вы можете настроить обогащение сообщений вручную с помощью [портала Azure](https://portal.azure.com).
> * Второй способ: создание ресурсов и настройка маршрутизации и обогащений сообщений с помощью шаблона ARM.
> * Запуск приложения, которое имитирует устройство Интернета вещей, отправляющее сообщения в центр.
> * Просмотр результатов и проверка того, что обогащения сообщений работают правильно.

Дополнительные сведения об обогащении сообщений см. в статье [Обзор обогащений сообщений](iot-hub-message-enrichments-overview.md).

Дополнительные сведения о маршрутизации сообщений см. в следующих статьях:

> [!div class="nextstepaction"]
> [Использование маршрутизации сообщений в Центре Интернета вещей для отправки с устройства в облако в разные конечные точки](iot-hub-devguide-messages-d2c.md)

> [!div class="nextstepaction"]
> [Учебник. Маршрутизация в Центре Интернета вещей](tutorial-routing.md)