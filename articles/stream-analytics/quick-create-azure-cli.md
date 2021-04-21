---
title: Краткое руководство. Создание задания Azure Stream Analytics с помощью Azure CLI
description: В этом кратком руководстве показано, как с помощью Azure CLI создать задание Azure Stream Analytics.
services: stream-analytics
ms.service: stream-analytics
author: sidramadoss
ms.author: sidram
ms.reviewer: jasonh
ms.workload: big-data
ms.topic: quickstart
ms.custom: mvc, devx-track-azurecli
ms.date: 07/01/2020
ms.openlocfilehash: 58dccf56cd493782a422b0ddf0386e31d4d87daf
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107765995"
---
# <a name="quickstart-create-an-azure-stream-analytics-job-using-the-azure-cli"></a>Краткое руководство. Создание задания Azure Stream Analytics с помощью Azure CLI

В этом кратком руководстве объясняется, как с помощью Azure CLI определить задание Stream Analytics, которое фильтрует сообщения датчиков в реальном времени со значениями температуры выше 27. В рамках задания Stream Analytics будут считываться данные из Центра Интернета вещей, затем данные будут преобразовываться и записываться в контейнер в хранилище BLOB-объектов. Входные данные, используемые в этом кратком руководстве, создаются онлайн-симулятором Raspberry Pi.

## <a name="before-you-begin"></a>Перед началом

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Создайте группу ресурсов. Все ресурсы Azure должны развертываться в группе ресурсов. Группы ресурсов позволяют организовать соответствующие ресурсы Azure и управлять ими.

   Для работы с этим кратким руководством создайте группу ресурсов с именем *streamanalyticsrg* в регионе *eastus* с помощью следующей команды [az group create](/cli/azure/group#az_group_create):

   ```azurecli
   az group create --name streamanalyticsrg --location eastus
   ```

## <a name="prepare-the-input-data"></a>Подготовка входных данных

Прежде чем определить задание Stream Analytics, подготовьте данные, которые используются в качестве входных данных для этого задания.

В следующих блоках кода Azure CLI выполняются команды для подготовки входных данных, необходимых для задания. Просмотрите разделы, чтобы понять код.

1. Создайте Центр Интернета вещей с помощью команды [az iot hub create](../iot-hub/iot-hub-create-using-cli.md#create-an-iot-hub). Этот пример создает Центр Интернета вещей с именем **MyASAIoTHub**. Так как имена Центров Интернета вещей являются уникальными, необходимо придумать собственное имя Центра Интернета вещей. Задайте для номера SKU значение F1, чтобы использовать уровень "Бесплатный", если он доступен с вашей подпиской. В противном случае выберите следующий самый низкий уровень.

    ```azurecli
    az iot hub create --name "MyASAIoTHub" --resource-group streamanalyticsrg --sku S1
    ```

    После создания Центра Интернета вещей получите его строку подключения с помощью команды [az iot hub show-connection-string](/cli/azure/iot/hub). Скопируйте всю строку подключения и сохраните ее, чтобы использовать во время добавления Центра Интернета вещей в качестве источника входных данных в задание Stream Analytics.

    ```azurecli
    az iot hub show-connection-string --hub-name "MyASAIoTHub"
    ```

2. Добавьте устройство к Центру Интернета вещей с помощью команды [az iothub device-identity create](../iot-hub/quickstart-send-telemetry-c.md#register-a-device). В этом примере создается устройство с именем **MyASAIoTDevice**.

    ```azurecli
    az iot hub device-identity create --hub-name "MyASAIoTHub" --device-id "MyASAIoTDevice"
    ```

3. Получите строку подключения устройства с помощью команды [az iot hub device-identity show-connection-string](/cli/azure/ext/azure-iot/iot/hub/device-identity#ext-azure-iot-az-iot-hub-device-identity-show-connection-string). Скопируйте всю строку подключения и сохраните ее для использования при создании симулятора Raspberry Pi.

    ```azurecli
    az iot hub device-identity show-connection-string --hub-name "MyASAIoTHub" --device-id "MyASAIoTDevice" --output table
    ```

    **Пример выходных данных.**

    ```output
    HostName=MyASAIoTHub.azure-devices.net;DeviceId=MyASAIoTDevice;SharedAccessKey=a2mnUsg52+NIgYudxYYUNXI67r0JmNubmfVafojG8=
    ```

## <a name="create-a-blob-storage-account"></a>Создание учетной записи хранения больших двоичных объектов

В следующих блоках кода Azure CLI создается учетная запись Хранилища BLOB-объектов, которая используется для вывода данных задания. Просмотрите разделы, чтобы понять код.

1. Создайте учетную запись хранения общего назначения с помощью команды [az storage account create](/cli/azure/storage/account). Эта учетная запись хранения может использоваться для всех четырех служб: больших двоичных объектов, файлов, таблиц и очередей.

   Не забудьте заменить значения заполнителей в угловых скобках собственными значениями.

   ```azurecli
   az storage account create \
       --name <storage-account> \
       --resource-group streamanalyticsrg \
       --location eastus \
       --sku Standard_ZRS \
       --encryption-services blob
   ```

2. Получите ключ своей учетной записи хранения с помощью команды [az storage account keys list](/cli/azure/storage/account/keys). Сохраните этот ключ для использования на следующем шаге.

   ```azurecli
   az storage account keys list -g streamanalyticsrg -n <storage-account>
   ```

3. Создайте контейнер для хранения больших двоичных объектов с помощью команды [az storage container create](/cli/azure/storage/container). Вы можете использовать ключ учетной записи хранения, чтобы авторизовать операцию создания контейнера. Дополнительные сведения об авторизации операций с данными с помощью Azure CLI см. в статье [Авторизация доступа к данным большого двоичного объекта или очереди с помощью Azure CLI](../storage/blobs/authorize-data-operations-cli.md).

   ```azurecli
   az storage container create \
       --account-name <storage-account> \
       --name sample-container \
       --account-key <key>
       --auth-mode key
   ```

## <a name="create-a-stream-analytics-job"></a>Создание задания Stream Analytics

В следующем блоке кода Azure CLI создается задание Stream Analytics. Просмотрите разделы, чтобы понять, как работает код.

1. Создайте задание Stream Analytics с помощью команды [az stream-analytics job create](/cli/azure/ext/stream-analytics/stream-analytics/job#ext-stream-analytics-az-stream-analytics-job-create).

```azurecli
az stream-analytics job create \
    --resource-group streamanalyticsrg 
    --name streamanalyticsjob \
    --location eastus \
    --output-error-policy "Drop" \
    --events-outoforder-policy "Drop" \
    --events-outoforder-max-delay 5 \
    --events-late-arrival-max-delay 16 \     
    --data-locale "en-US"
```

## <a name="configure-input-to-the-job"></a>Настройка входных данных для задания

Добавьте входные данные к заданию с помощью командлета [az stream-analytics input](/cli/azure/ext/stream-analytics/stream-analytics/input#ext-stream-analytics-az-stream-analytics-input-create). Этот командлет принимает имя задания, имя входных данных задания, имя группы ресурсов и определение входных данных задания в качестве параметров. Определение входных данных задания представляет собой JSON-файл со свойствами, необходимыми для настройки входных данных задания. В этом примере показано, как создать Центр Интернета вещей в качестве источника входных данных.

На вашем локальном компьютере создайте файл с именем `datasource.json` и добавьте в него следующие данные JSON. Не забудьте заменить значение для параметра `sharedAccessPolicyKey` частью `SharedAccessKey` строки подключения Центра Интернета вещей, сохраненной в предыдущем разделе.

```json
{
    "type": "Microsoft.Devices/IotHubs",
    "properties": {
        "iotHubNamespace": "iothub",
        "sharedAccessPolicyName": "iothubowner",
        "sharedAccessPolicyKey": "sharedAccessPolicyKey=",
        "consumerGroupName": "sdkconsumergroup",
        "endpoint": "messages/events"
    }
}
```

На вашем локальном компьютере создайте файл с именем `serialization.json` и добавьте в него следующие данные JSON.

```json
{
     "type": "Json",
     "properties": {
         "encoding": "UTF8"
     }
}
```

Затем выполните командлет `az stream-analytics input create`. Обязательно замените значение переменной `datasource` на путь, по которому вы сохранили файл JSON с определением входных данных задания, а значение переменной `serialization` — на путь, по которому вы сохранили файл сериализации в формате JSON.

```azurecli
az stream-analytics input create \
    --resource-group streamanalyticsrg 
    --job-name streamanalyticsjob \
    --name asaiotinput \
    --type Stream \
    --datasource datasource.json \
    --serialization serialization.json
```

## <a name="configure-output-to-the-job"></a>Настройка выходных данных для задания

Добавьте выходные данные к заданию с помощью командлета [az stream-analytics output create](/cli/azure/ext/stream-analytics/stream-analytics/output#ext-stream-analytics-az-stream-analytics-output-create). Этот командлет принимает имя задания, имя выходных данных задания, имя группы ресурсов и определение выходных данных задания в качестве параметров. Определение выходных данных задания представляет собой JSON-файл со свойствами, необходимыми для настройки выходных данных задания. В этом примере в качестве выходных данных используется хранилище BLOB-объектов.

На вашем локальном компьютере создайте файл с именем `datasink.json` и добавьте в него следующие данные JSON. Не забудьте заменить значение `accountKey` ключом доступа к учетной записи хранения. Это значение, хранящееся в $storageAccountKey.

```json
{
    "type": "Microsoft.Storage/Blob",
    "properties": {
        "storageAccounts": [
            {
                "accountName": "<storage-account>",
                "accountKey": "accountKey=="
            }
        ],
        "container": "state",
        "pathPattern": "{date}/{time}",
        "dateFormat": "yyyy/MM/dd",
        "timeFormat": "HH"
    }
}
```

Затем выполните командлет `az stream-analytics output`. Обязательно замените значение переменной `datasource` на путь, по которому вы сохранили файл JSON с определением выходных данных задания, а значение переменной `serialization` — на путь, по которому вы сохранили файл сериализации в формате JSON.

```azurecli
az stream-analytics output create \
    --resource-group streamanalyticsrg \
    --job-name streamanalyticsjob \
    --name asabloboutput \
    --datasource datasink.json \
    --serialization serialization.json
```

## <a name="define-the-transformation-query"></a>Определение запроса преобразования

Добавьте преобразование в задание с помощью командлета [az stream-analytics transformation create](/cli/azure/ext/stream-analytics/stream-analytics/transformation#ext-stream-analytics-az-stream-analytics-transformation-create). Этот командлет принимает имя задания, имя преобразования задания, имя группы ресурсов и определение преобразования задания в качестве параметров. 

Запустите командлет `az stream-analytics transformation create`.

```azurecli
az stream-analytics transformation create \
    --resource-group streamanalyticsrg \
    --job-name streamanalyticsjob \
    --name Transformation \
    --streaming-units "6" \
    --transformation-query "SELECT * INTO asabloboutput FROM asaiotinput HAVING Temperature > 27"
```
## <a name="run-the-iot-simulator"></a>Запуск IoT-симулятора

1. Откройте [онлайн-симулятор Raspberry Pi для Интернета вещей Azure](https://azure-samples.github.io/raspberry-pi-web-simulator/).

2. Замените значения заполнителя в 15-й строке строкой подключения устройства Центра Интернета вещей, которую вы сохранили при работе с предыдущим разделом.

3. Нажмите кнопку **Запустить**. В выходных данных должны присутствовать показания датчика и сообщения, отправляемые в Центр Интернета вещей.

    ![Онлайн-симулятор Raspberry Pi для Интернета вещей Azure.](./media/stream-analytics-quick-create-powershell/ras-pi-connection-string.png)

## <a name="start-the-stream-analytics-job-and-check-the-output"></a>Запуск задания Stream Analytics и просмотр выходных данных

Запустите задание с помощью командлета [az stream-analytics job start](/cli/azure/ext/stream-analytics/stream-analytics/job#ext-stream-analytics-az-stream-analytics-job-start). Этот командлет принимает имя задания, имя группы ресурсов, режим начала вывода задания и время запуска в качестве параметров. `OutputStartMode` принимает значения `JobStartTime`, `CustomTime` или `LastOutputEventTime`.

После запуска следующего командлета, если задание запустится, будет возвращено значение `True` в качестве выходных данных. В контейнере хранилища будет создана выходная папка с преобразованными данными.

```azurecli
az stream-analytics job start \
    --resource-group streamanalyticsrg \
    --name streamanalyticsjob \
    --output-start-mode JobStartTime
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Ставшие ненужными группу ресурсов, задание потоковой передачи и все связанные ресурсы можно удалить. При удалении задания будет прекращена тарификация за единицы потоковой передачи, потребляемые заданием. Если вы планируете использовать задание в дальнейшем, остановите его и не выполняйте процедуру его удаления. Если вам больше не нужно это задание, удалите все ресурсы, созданные в ходе работы с этим кратким руководством, выполнив следующий командлет.

```azurecli
az group delete \
    --name streamanalyticsrg \
    --no-wait
```

## <a name="next-steps"></a>Дальнейшие действия

С помощью инструкций из этого краткого руководства вы развернули простое задание Stream Analytics, используя Azure CLI. Вы также можете развертывать задания Stream Analytics, используя [портал Azure](stream-analytics-quick-create-portal.md) и [Visual Studio](stream-analytics-quick-create-vs.md).

Дополнительные сведения о настройке других источников входных данных и обнаружении в режиме реального времени см. в следующей статье:

> [!div class="nextstepaction"]
> [Приступая к работе с Azure Stream Analytics: выявление мошенничества в режиме реального времени](stream-analytics-real-time-fraud-detection.md)