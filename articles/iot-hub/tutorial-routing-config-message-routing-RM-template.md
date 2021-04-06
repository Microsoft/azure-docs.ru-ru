---
title: Учебник. Настройка маршрутизации сообщений для Центра Интернета вещей с помощью шаблона ARM
description: Учебник. Настройка маршрутизации сообщений для Центра Интернета вещей с помощью шаблона ARM
author: robinsh
manager: philmeagit st
ms.service: iot-hub
services: iot-hub
ms.topic: tutorial
ms.date: 03/25/2019
ms.author: robinsh
ms.custom: mvc
ms.openlocfilehash: 53217340b0d91f3de77e5e0d8c0a82e30599d6ed
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98621433"
---
# <a name="tutorial-use-an-azure-resource-manager-template-to-configure-iot-hub-message-routing"></a>Руководство по Применение шаблона Azure Resource Manager для настройки маршрутизации сообщений в Центре Интернета вещей

[!INCLUDE [iot-hub-include-routing-intro](../../includes/iot-hub-include-routing-intro.md)]

[!INCLUDE [iot-hub-include-routing-create-resources](../../includes/iot-hub-include-routing-create-resources.md)]

## <a name="message-routing"></a>Маршрутизация сообщений

[!INCLUDE [iot-hub-include-create-routing-description](../../includes/iot-hub-include-create-routing-description.md)]

## <a name="download-the-template-and-parameters-file"></a>Скачивание файла шаблона и параметров

Для работы со второй частью этого руководства нужно скачать и запустить приложение Visual Studio для отправки сообщений в Центр Интернета вещей. В скачиваемых ресурсах присутствует папка, которая содержит шаблон Azure Resource Manager и файл параметров, а также скрипты Azure CLI и PowerShell.

Скачайте файл с [примерами для Azure IoT на C#](https://github.com/Azure-Samples/azure-iot-samples-csharp/archive/master.zip) прямо сейчас. Распакуйте файл master.zip. Шаблон Resource Manager и файл параметров размещаются в папке /iot-hub/Tutorials/Routing/SimulatedDevice/resources/ с именами **template_iothub.json** и **template_iothub_parameters.json**.

## <a name="create-your-resources"></a>Создание нужных ресурсов

С помощью шаблона Azure Resource Manager (RM) вы создадите все нужные ресурсы. Скрипты Azure CLI и PowerShell можно выполнять по несколько строк за раз. Шаблон RM развертывается за один шаг. В этой статье каждый этап вынесен в отдельный раздел, чтобы упростить понимание. Также мы продемонстрируем, как развернуть шаблон и создать виртуальное устройство для тестирования. После развертывания шаблона вы сможете проверить на портале конфигурацию для маршрутизации сообщений.

Некоторым ресурсам необходимо присвоить уникальные имена, включая имя Центра Интернета вещей и имя учетной записи хранения. Чтобы упростить именование ресурсов, имена дополняются случайным буквенно-цифровым префиксом, который генерируется на основе текущей даты и времени. 

Взглянув на шаблон, вы быстро найдете место, где настраиваются переменные для этих ресурсов, которые принимают переданный параметр и объединяют случайное значение (*randomValue*) со значением принятого параметра. 

В следующем разделе описано использование параметров.

### <a name="parameters"></a>Параметры

Большинство из этих параметров имеют значения по умолчанию. Те из них, названия которых заканчиваются на **_in**, соединяются с *randomValue*, чтобы сделать их глобально уникальными. 

**randomValue**: это значение создается на основе текущей даты и времени при развертывании шаблона. Это поле не включено в файл параметров, так как создается непосредственно в шаблоне.

**subscriptionId**: этому полю присваивается значение идентификатора подписки, в которой вы развертываете шаблон. Это поле не включено в файл параметров, так как создается автоматически.

**IoTHubName_in**: это поле содержит базовое имя Центра Интернета вещей, которое объединяется с randomValue для получения глобально уникального значения.

**location**: это поле обозначает регион Azure, в который выполняется развертывание, например "westus".

**consumer_group**: это поле обозначает группу потребителей, которая настроена для получения сообщений, поступающих через конечную точку маршрутизации. Она используется для фильтрации результатов в Azure Stream Analytics. Например, есть полный поток, в который поступают все сообщения. А если в поступающих данных consumer_group имеет значение **Contoso**, то вы можете настроить поток Azure Stream Analytics (и отчет Power BI), отображающий только такие записи. Это поле используется в части 2 этого руководства.

**sku_name**: это поле позволяет масштабировать Центр Интернета вещей. Здесь должно быть указано значение S1 или выше. Уровень "Бесплатный" для этого руководства не подходит, поскольку он не поддерживает несколько конечных точек.

**sku_units**: это поле применяется в сочетании со **sku_name** и обозначает допустимое число используемых единиц Центра Интернета вещей.

**d2c_partitions**: это поле обозначает число секций, используемых для потока событий.

**storageAccountName_in**: это поле содержит имя учетной записи хранения, которая будет создана. Сообщения направляются в контейнер этой учетной записи хранения. Это поле объединяется с randomValue, чтобы получить глобально уникальное значение.

**storageContainerName**: это поле содержит имя контейнера, в котором сохраняются сообщения, отправляемые учетной записи хранения.

**storage_endpoint**: это поле содержит имя конечной точки в учетной записи хранения, которая используется для маршрутизации сообщений.

**service_bus_namespace_in**: это поле содержит имя создаваемого пространства имен служебной шины. Это значение объединяется с randomValue, чтобы получить глобально уникальное значение.

**service_bus_queue_in**: это поле содержит имя очереди служебной шины, которая используется для маршрутизации сообщений. Это значение объединяется с randomValue, чтобы получить глобально уникальное значение.

**AuthRules_sb_queue**: это поле содержит правила авторизации для очереди служебной шины, которые используются для получения строки подключения к этой очереди.

### <a name="variables"></a>Переменные

Эти значения используются в шаблоне и почти все являются производными от параметров.

**queueAuthorizationRuleResourceId**: это поле содержит идентификатор ресурса для правила авторизации очереди служебной шины. Этот идентификатор ресурса, в свою очередь, используется для получения строки подключения к очереди.

**iotHubName**: это поле содержит имя Центра Интернета вещей с присоединенным значением randomValue. 

**storageAccountName**: это поле содержит имя учетной записи хранения с присоединенным значением randomValue. 

**service_bus_namespace**: это поле содержит пространство имен с присоединенным значением randomValue.

**service_bus_queue**: это поле содержит имя очереди служебной шины с присоединенным значением randomValue.

**sbVersion**: используемая версия API Служебной шины. В нашем примере это "2017-04-01".

### <a name="resources-storage-account-and-container"></a>Ресурсы: учетная запись хранения и контейнер

Первой создается учетная запись хранения с контейнером для получения сообщений. Контейнер — это ресурс в учетной записи хранения. В нем есть предложение `dependsOn` для учетной записи хранения, которое требует создать учетную запись хранения перед созданием контейнера.

Полностью этот раздел выглядит так:

```json
{
    "type": "Microsoft.Storage/storageAccounts",
    "name": "[variables('storageAccountName')]",
    "apiVersion": "2018-07-01",
    "location": "[parameters('location')]",
    "sku": {
        "name": "Standard_LRS",
        "tier": "Standard"
    },
    "kind": "Storage",
    "properties": {},
    "resources": [
        {
        "type": "blobServices/containers",
        "apiVersion": "2018-07-01",
        "name": "[concat('default/', parameters('storageContainerName'))]",
        "properties": {
            "publicAccess": "None"
            } ,
        "dependsOn": [
            "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
            ]
        }
    ]
}
```

### <a name="resources-service-bus-namespace-and-queue"></a>Ресурсы: пространство имен и очередь служебной шины

Вторым создается пространство имен и очередь служебной шины, в которую будут направляться сообщения. Для нее установлен номер SKU "Стандартный". Версия API извлекается из переменных. Также здесь настроена активация пространства имен служебной шины при развертывании этого раздела (status:Active). 

```json
{
    "type": "Microsoft.ServiceBus/namespaces",
    "comments": "The Sku should be 'Standard' for this tutorial.",
    "sku": {
        "name": "Standard",
        "tier": "Standard"
    },
    "name": "[variables('service_bus_namespace')]",
    "apiVersion": "[variables('sbVersion')]",
    "location": "[parameters('location')]",
    "properties": {
        "provisioningState": "Succeeded",
        "metricId": "[concat('a4295411-5eff-4f81-b77e-276ab1ccda12:', variables('service_bus_namespace'))]",
        "serviceBusEndpoint": "[concat('https://', variables('service_bus_namespace'),'.servicebus.windows.net:443/')]",
        "status": "Active"
    },
    "dependsOn": []
}
```

Этот раздел создает очередь служебной шины. В этой части скрипта присутствует предложение `dependsOn`, которое требует создать пространство имен перед созданием очереди.

```json
{
    "type": "Microsoft.ServiceBus/namespaces/queues",
    "name": "[concat(variables('service_bus_namespace'), '/', variables('service_bus_queue'))]",
    "apiVersion": "[variables('sbVersion')]",
    "location": "[parameters('location')]",
    "scale": null,
    "properties": {},
    "dependsOn": [
        "[resourceId('Microsoft.ServiceBus/namespaces', variables('service_bus_namespace'))]"
    ]
}
```

### <a name="resources-iot-hub-and-message-routing"></a>Ресурсы: Центр Интернета вещей и маршрутизация сообщений

Теперь, когда созданы учетная запись хранения и очередь служебной шины, можно создать Центр Интернета вещей, который будет направлять в них сообщения. В шаблоне RM используется предложение `dependsOn`, чтобы центр не создавался раньше ресурсов служебной шины и учетной записи хранения. 

Так выглядит первая часть раздела для Центра Интернета вещей. Эта часть шаблона настраивает все зависимости, начиная со значений свойств.

```json
{
    "apiVersion": "2018-04-01",
    "type": "Microsoft.Devices/IotHubs",
    "name": "[variables('IoTHubName')]",
    "location": "[parameters('location')]",
    "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]",
        "[resourceId('Microsoft.ServiceBus/namespaces', variables('service_bus_namespace'))]",
        "[resourceId('Microsoft.ServiceBus/namespaces/queues', variables('service_bus_namespace'), variables('service_bus_queue'))]"
    ],
    "properties": {
        "eventHubEndpoints": {}
            "events": {
                "retentionTimeInDays": 1,
                "partitionCount": "[parameters('d2c_partitions')]"
                }
            },
```

Следующий раздел задает конфигурацию для маршрутизации сообщений в Центре Интернета вещей. Сначала определяются конечные точки. Эта часть шаблона настраивает конечные точки маршрутизации для очереди служебной шины и учетной записи хранения, в том числе строки подключения к ним.

Чтобы создать строку подключения к очереди, из строки параметра извлекается значение queueAuthorizationRulesResourcedId. Чтобы создать строку подключения к учетной записи хранения, нужно извлечь первичный ключ хранилища и подставить его в формат строки подключения.

Кроме того, в конфигурации конечных точек задается формат больших двоичных объектов (`AVRO` или `JSON`).

[!INCLUDE [iot-hub-include-blob-storage-format](../../includes/iot-hub-include-blob-storage-format.md)]

 ```json
"routing": {
    "endpoints": {
        "serviceBusQueues": [
        {
            "connectionString": "[Concat('Endpoint=sb://',variables('service_bus_namespace'),'.servicebus.windows.net/;SharedAccessKeyName=',parameters('AuthRules_sb_queue'),';SharedAccessKey=',listkeys(variables('queueAuthorizationRuleResourceId'),variables('sbVersion')).primaryKey,';EntityPath=',variables('service_bus_queue'))]",
            "name": "[parameters('service_bus_queue_endpoint')]",
            "subscriptionId": "[parameters('subscriptionId')]", 
            "resourceGroup": "[resourceGroup().Name]"
        }
        ],
        "serviceBusTopics": [],
        "eventHubs": [],
        "storageContainers": [
            {
                "connectionString": 
                "[Concat('DefaultEndpointsProtocol=https;AccountName=',variables('storageAccountName'),';AccountKey=',listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName')), providers('Microsoft.Storage', 'storageAccounts').apiVersions[0]).keys[0].value)]",
                "containerName": "[parameters('storageContainerName')]",
                "fileNameFormat": "{iothub}/{partition}/{YYYY}/{MM}/{DD}/{HH}/{mm}",
                "batchFrequencyInSeconds": 100,
                "maxChunkSizeInBytes": 104857600,
                "encoding": "avro",
                "name": "[parameters('storage_endpoint')]",
                "subscriptionId": "[parameters('subscriptionId')]",
                "resourceGroup": "[resourceGroup().Name]"
            }
        ]
    },
```

В следующем разделе указаны маршруты к конечным точкам для сообщений. Для каждой конечной точки настраивается отдельный маршрут: один для очереди служебной шины и один для контейнера учетной записи хранения.

Не забывайте, что для сообщений, направляемых в хранилище, используется условие запроса `level="storage"`, а для направляемых в очередь служебной шины — `level="critical"`.

```json
"routes": [
    {
        "name": "contosoStorageRoute",
        "source": "DeviceMessages",
        "condition": "level=\"storage\"",
        "endpointNames": [
            "[parameters('storage_endpoint')]"
            ],
        "isEnabled": true
    },
    {
        "name": "contosoSBQueueRoute",
        "source": "DeviceMessages",
        "condition": "level=\"critical\"",
        "endpointNames": [
            "[parameters('service_bus_queue_endpoint')]"
            ],
        "isEnabled": true
    }
],
```

Этот код JSON содержит оставшуюся часть раздела для Центра Интернета вещей со значениями по умолчанию и номером SKU для центра.

```json
            "fallbackRoute": {
                "name": "$fallback",
                "source": "DeviceMessages",
                "condition": "true",
                "endpointNames": [
                    "events"
                ],
                "isEnabled": true
            }
        },
        "storageEndpoints": {
            "$default": {
                "sasTtlAsIso8601": "PT1H",
                "connectionString": "",
                "containerName": ""
            }
        },
        "messagingEndpoints": {
            "fileNotifications": {
                "lockDurationAsIso8601": "PT1M",
                "ttlAsIso8601": "PT1H",
                "maxDeliveryCount": 10
            }
        },
        "enableFileUploadNotifications": false,
        "cloudToDevice": {
            "maxDeliveryCount": 10,
            "defaultTtlAsIso8601": "PT1H",
            "feedback": {
                "lockDurationAsIso8601": "PT1M",
                "ttlAsIso8601": "PT1H",
                "maxDeliveryCount": 10
            }
        }
    },
    "sku": {
        "name": "[parameters('sku_name')]",
        "capacity": "[parameters('sku_units')]"
    }
}
```

### <a name="resources-service-bus-queue-authorization-rules"></a>Ресурсы: правила авторизации для очереди служебной шины

Правило авторизации для очереди служебной шины используется для получения строки подключения к очереди служебной шины. В нем указано предложение `dependsOn`, чтобы правило не создавалось раньше пространства имен и очереди служебной шины.

```json
{
    "type": "Microsoft.ServiceBus/namespaces/queues/authorizationRules",
    "name": "[concat(variables('service_bus_namespace'), '/', variables('service_bus_queue'), '/', parameters('AuthRules_sb_queue'))]",
    "apiVersion": "[variables('sbVersion')]",
    "location": "[parameters('location')]",
    "scale": null,
    "properties": {
        "rights": [
            "Send"
        ]
    },
    "dependsOn": [
        "[resourceId('Microsoft.ServiceBus/namespaces', variables('service_bus_namespace'))]",
        "[resourceId('Microsoft.ServiceBus/namespaces/queues', variables('service_bus_namespace'), variables('service_bus_queue'))]"
    ]
},
```

### <a name="resources-consumer-group"></a>Ресурсы: Группа потребителей

В этом разделе вы создадите группу потребителей для данных Центра Интернета вещей, которая будет использоваться для Azure Stream Analytics во второй части этого руководства.

```json
{
    "type": "Microsoft.Devices/IotHubs/eventHubEndpoints/ConsumerGroups",
    "name": "[concat(variables('iotHubName'), '/events/',parameters('consumer_group'))]",
    "apiVersion": "2018-04-01",
    "dependsOn": [
        "[concat('Microsoft.Devices/IotHubs/', variables('iotHubName'))]"
    ]
}
```

### <a name="resources-outputs"></a>Ресурсы: Выходные данные

Если вы хотите вернуть в скрипт развертывания значение для отображения, используйте раздел выходных данных. Эта часть шаблона возвращает строку подключения к очереди служебной шины. Возвращать значение необязательно, и здесь мы настроили его только для демонстрации того, как правильно возвращать результаты вызывающему скрипту.

```json
"outputs": {
    "sbq_connectionString": {
      "type": "string",
      "value": "[Concat('Endpoint=sb://',variables('service_bus_namespace'),'.servicebus.windows.net/;SharedAccessKeyName=',parameters('AuthRules_sb_queue'),';SharedAccessKey=',listkeys(variables('queueAuthorizationRuleResourceId'),variables('sbVersion')).primaryKey,';EntityPath=',variables('service_bus_queue'))]"
    }
  }
```

## <a name="deploy-the-rm-template"></a>Развертывание шаблона RM

Чтобы развернуть шаблон в Azure, отправьте сам шаблон и файл параметров в Azure Cloud Shell, а затем выполните скрипт развертывания шаблона. Откройте Azure Cloud Shell и выполните вход. В этом примере используется PowerShell.

Чтобы передать файлы, щелкните значок **отправки и скачивания файлов** на панели меню, затем выберите действие "Отправить".

![Снимок экрана, на котором выделен значок отправки и скачивания файлов.](media/tutorial-routing-config-message-routing-RM-template/CloudShell_upload_files.png)

В открывшемся окне проводника найдите на локальном диске нужные файлы и выберите их, а затем щелкните **Открыть**.

Когда отправка файлов завершится, отобразится диалоговое окно с результатами, примерно как на следующем изображении.

![Меню Cloud Shell с выделенным пунктом для отправки и скачивания результатов](media/tutorial-routing-config-message-routing-RM-template/CloudShell_upload_results.png)

Файлы передаются в общую папку, которую использует ваш экземпляр Cloud Shell. 

Запустите скрипт, чтобы выполнить развертывание. Последняя строка этого скрипта извлекает переменную, которая была создана для возврата значения. Она содержит строку подключения к очереди служебной шины.

Скрипт задает и использует следующие переменные:

**$RGName** — имя группы ресурсов, в которой нужно развернуть шаблон. Это поле создается перед развертыванием шаблона.

**$location** — расположение Azure, которое будет выбрано для шаблона, например "westus".

**deploymentname** — присвоенное развертыванию имя для получения возвращаемого значения переменной.

Вот полный текст этого скрипта PowerShell. Скопируйте этот код PowerShell и вставьте его в окно Cloud Shell, а затем нажмите клавишу ВВОД, чтобы запустить его.

```powershell
$RGName="ContosoResources"
$location = "westus"
$deploymentname="contoso-routing"

# Remove the resource group if it already exists. 
#Remove-AzResourceGroup -name $RGName 
# Create the resource group.
New-AzResourceGroup -name $RGName -Location $location 

# Set a path to the parameter file. 
$parameterFile = "$HOME/template_iothub_parameters.json"
$templateFile = "$HOME/template_iothub.json"

# Deploy the template.
New-AzResourceGroupDeployment `
    -Name $deploymentname `
    -ResourceGroupName $RGName `
    -TemplateParameterFile $parameterFile `
    -TemplateFile $templateFile `
    -verbose

# Get the returning value of the connection string.
(Get-AzResourceGroupDeployment -ResourceGroupName $RGName -Name $deploymentname).Outputs.sbq_connectionString.value
```

Если в скрипте будет обнаружена ошибка, измените его текст локально, затем повторно отправьте его в Cloud Shell и запустите. Когда скрипт будет успешно выполнен, переходите к следующему шагу.

## <a name="create-simulated-device"></a>Создание виртуального устройства

[!INCLUDE [iot-hub-include-create-simulated-device-portal](../../includes/iot-hub-include-create-simulated-device-portal.md)]

## <a name="view-message-routing-in-the-portal"></a>Просмотр маршрутизации сообщений на портале

[!INCLUDE [iot-hub-include-view-routing-in-portal](../../includes/iot-hub-include-view-routing-in-portal.md)]

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда у вас настроены все ресурсы и маршруты для сообщений, переходите к следующему учебнику, где вы научитесь обрабатывать и отображать сведения о перенаправленных сообщениях.

> [!div class="nextstepaction"]
> [Часть 2. Просмотр результатов маршрутизации сообщений](tutorial-routing-view-message-routing-results.md)
