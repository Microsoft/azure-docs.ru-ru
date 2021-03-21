---
title: Экспорт данных из IoT Central Azure (прежние версии) | Документация Майкрософт
description: Экспорт данных из приложения IoT Central Azure в концентраторы событий Azure, служебную шину Azure и хранилище BLOB-объектов Azure
services: iot-central
author: viv-liu
ms.author: viviali
ms.date: 06/25/2020
ms.topic: how-to
ms.service: iot-central
ms.openlocfilehash: d4c099d29a843b4c354ffb218887dc7ffab51771
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98065445"
---
# <a name="export-iot-data-to-cloud-destinations-using-data-export-legacy"></a>Экспорт данных IoT в облачные назначения с помощью экспорта данных (прежние версии)

> [!Note]
> В этой статье описаны функции экспорта устаревших данных в IoT Central.
>
> - Сведения о последних функциях экспорта данных см. в статье [Экспорт данных Интернета вещей в облачные назначения с помощью экспорта данных](./howto-export-data.md).
> - Дополнительные сведения о различиях между функциями предварительной версии экспорта данных и экспортом устаревших данных см. в [таблице сравнения](./howto-export-data.md#comparison-of-legacy-data-export-and-data-export).

В этой статье описывается, как использовать функцию экспорта данных в Azure IoT Central. Эта функция позволяет непрерывно экспортировать данные в **концентраторы событий Azure**, **служебную шину Azure** или экземпляры **хранилища BLOB-объектов Azure** . Экспорт данных использует формат JSON и может включать данные телеметрии, сведения об устройстве и сведения о шаблонах устройств. Используйте экспортированные данные для:

- Подробные сведения о теплом пути и аналитика. Этот параметр включает в себя запуск настраиваемых правил в Azure Stream Analytics, запуск пользовательских рабочих процессов в Azure Logic Apps или их передачу с помощью функций Azure для преобразования.
- Аналитика холодного пути, например моделей обучения в Машинное обучение Azure или долгосрочном анализе тенденций в Microsoft Power BI.

> [!Note]
> При включении экспорта данных вы получаете только данные из этого момента. В настоящее время невозможно получить данные за время отключения экспорта данных. Чтобы хранить более исторические данные, включите экспорт данных на раннем этапе.

## <a name="prerequisites"></a>Предварительные условия

Необходимо быть администратором в IoT Central приложении или иметь разрешения на экспорт данных.

## <a name="set-up-export-destination"></a>Настройка места назначения экспорта

Назначение экспорта должно существовать до настройки экспорта данных.

### <a name="create-event-hubs-namespace"></a>Создание пространства имен Центров событий

Если у вас нет пространства имен концентраторов событий для экспорта, выполните следующие действия.

1. Создайте [пространство имен Центров событий на портале Azure](https://ms.portal.azure.com/#create/Microsoft.EventHub). Дополнительные сведения см. в [документации по Центрам событий Azure](../../event-hubs/event-hubs-create.md).

2. Выберите подписку. Данные можно экспортировать в другие подписки, которые не входят в ту же подписку, что и приложение IoT Central. В этом случае вы подключаетесь, используя строку подключения.

3. Создайте концентратор событий в пространстве имен в Центрах событий. Перейдите к пространству имен и выберите **+ Концентратор событий** в верхней части, чтобы создать экземпляр концентратора событий.

### <a name="create-service-bus-namespace"></a>Создание пространства имен Служебной шины

Если у вас нет пространства имен служебной шины для экспорта, выполните следующие действия.

1. Создайте [новое пространство имен служебной шины в портал Azure](https://ms.portal.azure.com/#create/Microsoft.ServiceBus.1.0.5). Дополнительные сведения см. в [документации по Служебной шине Azure](../../service-bus-messaging/service-bus-create-namespace-portal.md).
2. Выберите подписку. Данные можно экспортировать в другие подписки, которые не входят в ту же подписку, что и приложение IoT Central. В этом случае вы подключаетесь, используя строку подключения.

3. Чтобы создать очередь или раздел для экспорта, перейдите к пространству имен служебной шины и выберите **+ очередь** или **+ раздел**.

Если в качестве назначения экспорта выбрана служебная шина, то очереди и разделы не должны включать сеансы или обнаружение повторяющихся записей. Если любая из этих функций включена, некоторые сообщения не будут поступать в очередь или раздел.

### <a name="create-storage-account"></a>Создание учетной записи хранения

Если у вас нет существующей учетной записи хранения Azure для экспорта, выполните следующие действия.

1. Создайте [учетную запись хранения на портале Azure](https://ms.portal.azure.com/#create/Microsoft.StorageAccount-ARM). Вы можете узнать больше о создании новых [учетных записей хранения BLOB-объектов Azure](../../storage/blobs/storage-quickstart-blobs-portal.md) или [учетных записей хранения Azure Data Lake Storage v2](../../storage/common/storage-account-create.md). Экспорт данных может записывать данные только в учетные записи хранения, поддерживающие блочные BLOB-объекты. В следующем списке приведены известные совместимые типы учетных записей хранения.

    |Уровень производительности|Тип учетной записи|
    |-|-|
    |Стандартный|общего назначения v2|
    |Стандартный|общего назначения v1|
    |Стандартный|Хранилище BLOB-объектов|
    |Premium|Блочное хранилище BLOB-объектов|

2. Создайте контейнер в учетной записи хранения. Войдите в свою учетную запись хранения. Выберите **Обзор BLOB-объектов** в разделе **Служба BLOB-объектов**. Выберите **+ Контейнер** в верхней части экрана, чтобы создать контейнер.

## <a name="set-up-data-export"></a>Настройка экспорта данных

Теперь, когда у вас есть место назначения для экспорта данных, выполните следующие действия, чтобы настроить экспорт данных.

1. Войдите в приложение IoT Central.

2. На левой панели выберите **Экспорт данных**.

    > [!Tip]
    > Если вы не видите элемент **Экспорт данных** в левой области, у вас нет разрешений на настройку экспорта данных в приложении. Обратитесь к администратору, чтобы настроить экспорт данных.

3. Нажмите кнопку **+Создать**. Выберите одно из **хранилищ BLOB-объектов Azure**, **концентраторов событий** Azure, **очередь служебной шины Azure** или **раздел служебной шины Azure** в качестве назначения для экспорта. Максимальное число экспортов на приложение равно пяти.

4. Введите имя для экспорта. В раскрывающемся списке выберите **пространство имен** или **введите строку подключения**.

    - Учетные записи хранения, пространства имен концентраторов событий и пространства имен служебной шины отображаются только в той же подписке, что и приложение IoT Central. Если вы хотите экспортировать в назначение за пределами этой подписки, выберите **Ввод строки подключения** и см. шаг 6.
    - Для приложений, созданных с помощью бесплатного тарифного плана, единственным способом настройки экспорта данных является строка подключения. Для приложений, иссвященных бесплатному тарифному плану, не назначена подписка Azure.

    ![Создать новый концентратор событий](media/howto-export-data-legacy/export-event-hub.png)

5. В раскрывающемся списке выберите концентратор событий, очередь, раздел или контейнер.

6. (Необязательно.) Если вы выбрали **Enter a connection string** (Ввести строку подключения), открывается новое окно, в которое нужно вставить строку подключения. Чтобы получить строку подключения, сделайте следующее:

    - Концентраторы событий или служебная шина, перейдите к пространству имен в портал Azure:
        - Чтобы использовать строку подключения для всего пространства имен, сделайте следующее:
            1. В разделе **Параметры** выберите **политики общего доступа** .
            2. Создайте новый ключ или выберите существующий ключ с разрешениями на **отправку** .
            3. Скопируйте основную или дополнительную строку подключения.
        - Чтобы использовать строку подключения для определенного экземпляра концентратора событий или очереди или раздела служебной шины, перейдите в раздел **сущности > концентраторы событий** или **сущности > очереди** или **сущности > разделов**. Выберите конкретный экземпляр и выполните те же действия, что и для получения строки подключения.
    - Учетная запись хранения, перейдите к учетной записи хранения в портал Azure:
        - Поддерживаются только строки подключения для всей учетной записи хранения. Строки подключения, областью действия которых является один контейнер, не поддерживаются.
          1. В разделе **Параметры** выберите **ключи доступа** .
          2. Скопируйте строку подключения key1 или строку подключения Key2.

    Вставьте строку подключения. Введите **имя контейнера** в экземпляре или с учетом регистра.

7. В разделе **данные для экспорта** выберите типы данных для экспорта, задав для параметра Тип значение **вкл**.

8. Чтобы включить экспорт данных, **Убедитесь, что** **включен переключатель Включить** . Щелкните **Сохранить**.

9. Через несколько минут данные отобразятся в выбранном месте назначения.

## <a name="export-contents-and-format"></a>Экспорт содержимого и формата

Экспортированные данные телеметрии содержат все сообщения, отправленные в IoT Central, а не только сами значения телеметрии. Данные экспортированных устройств содержат изменения свойств и метаданных всех устройств, а экспортированные шаблоны устройств содержат изменения во всех шаблонах устройств.

Для концентраторов событий и служебной шины данные экспортируются практически в реальном времени. Данные находятся в `body` свойстве и имеют формат JSON. Примеры см. ниже.

Для хранилища BLOB-объектов данные экспортируются один раз в минуту, при этом каждый файл содержит пакет изменений с момента последнего экспортированного файла. Экспортированные данные помещаются в три папки в формате JSON. Пути по умолчанию в вашей учетной записи хранения:

- Данные телеметрии: _{Container}/{АПП-ИД}/телеметри/{ИИИИ}/{мм}/{дд}/{ХХ}/{мм}/{филенаме}_
- Устройства: _{Container}/{АПП-ИД}/девицес/{ИИИИ}/{мм}/{дд}/{ХХ}/{мм}/{филенаме}_
- Шаблоны устройств: _{Container}/{АПП-ИД}/девицетемплатес/{ИИИИ}/{мм}/{дд}/{ХХ}/{мм}/{филенаме}_

Чтобы просмотреть экспортированные файлы в портал Azure, перейдите к файлу и перейдите на вкладку **изменение большого двоичного объекта** .

## <a name="telemetry"></a>Телеметрия

Для концентраторов событий и служебной шины IoT Central быстро экспортирует новое сообщение после получения сообщения с устройства. Каждое экспортированное сообщение содержит полное сообщение, отправленное устройством в свойстве body в формате JSON.

Для хранилища BLOB-объектов сообщения помещаются в пакет и экспортируются один раз в минуту. Экспортируемые файлы используют тот же формат, что и файлы сообщений, экспортированные [службой маршрутизации сообщений центра Интернета вещей](../../iot-hub/tutorial-routing.md) в хранилище BLOB-объектов.

> [!NOTE]
> Для хранилища BLOB-объектов убедитесь, что устройства отправляют сообщения с `contentType: application/JSON` и `contentEncoding:utf-8` (или `utf-16` `utf-32` ). Пример см. в документации по центру [Интернета вещей](../../iot-hub/iot-hub-devguide-routing-query-syntax.md#message-routing-query-based-on-message-body) .

Устройство, отправившего данные телеметрии, представлено ИДЕНТИФИКАТОРом устройства (см. следующие разделы). Чтобы получить имена устройств, экспортируйте данные устройства и сопоставьте каждое сообщение с помощью **коннектиондевицеид** , который соответствует идентификатору **deviceId** сообщения устройства.

В следующем примере показано сообщение, полученное из концентратора событий или из очереди или раздела служебной шины:

```json
{
  "temp":81.129693132351775,
  "humid":59.488071477541247,
  "EventProcessedUtcTime":"2020-04-07T09:41:15.2877981Z",
  "PartitionId":0,
  "EventEnqueuedUtcTime":"2020-04-07T09:38:32.7380000Z"
}
```

Это сообщение не содержит идентификатор устройства отправляющего устройства.

Чтобы получить идентификатор устройства из данных сообщения в Azure Stream Analytics запросе, используйте функцию [жетметадатапропертивалуе](/stream-analytics-query/getmetadatapropertyvalue) . Пример см. в статье [расширение возможностей azure IOT Central с помощью настраиваемых правил, использующих Stream Analytics, функции Azure и SendGrid](./howto-create-custom-rules.md).

Чтобы получить идентификатор устройства в Azure Databricks или Apache Spark рабочей области, используйте [системпропертиес](https://github.com/Azure/azure-event-hubs-spark/blob/master/docs/structured-streaming-eventhubs-integration.md). Пример см. в разделе [расширение возможностей Azure IOT Central с настраиваемой аналитикой с помощью Azure Databricks](./howto-create-custom-analytics.md).

В следующем примере показана запись, экспортированная в хранилище BLOB-объектов.

```json
{
  "EnqueuedTimeUtc":"2019-09-26T17:46:09.8870000Z",
  "Properties":{

  },
  "SystemProperties":{
    "connectionDeviceId":"<deviceid>",
    "connectionAuthMethod":"{\"scope\":\"device\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}",
    "connectionDeviceGenerationId":"637051167384630591",
    "contentType":"application/json",
    "contentEncoding":"utf-8",
    "enqueuedTime":"2019-09-26T17:46:09.8870000Z"
  },
  "Body":{
    "temp":49.91322758395974,
    "humid":49.61214852573155,
    "pm25":25.87332214661367
  }
}
```

## <a name="devices"></a>Устройства

Каждое сообщение или запись в моментальном снимке представляет одно или несколько изменений устройства, свойства устройства и облака с момента последнего экспортированного сообщения. Сообщение включает в себя:

- идентификатор (`id`) устройства в IoT Central;
- имя (`displayName`) устройства;
- Идентификатор шаблона устройства в `instanceOf`
- `simulated` флаг, true, если устройство является имитируемым
- `provisioned` флаг, true, если устройство подготовлено
- `approved` флаг, true, если устройство утверждено для отправки данных
- Значения свойств
- `properties` Включение значений свойств устройств и облака

Удаленные устройства не экспортируются. В настоящее время в сообщениях экспорта нет никаких индикаторов касательно удаленных устройств.

Для концентраторов событий и служебной шины IoT Central отправляет сообщения, содержащие данные устройства, в концентратор событий или в очередь или раздел служебной шины практически в реальном времени.

Для хранилища BLOB-объектов новый моментальный снимок, содержащий все изменения с момента последнего записанного, экспортируется один раз в минуту.

В следующем примере сообщения отображаются сведения об устройствах и свойствах в концентраторе событий или в очереди или разделе служебной шины:

```json
{
  "body":{
    "id": "<device Id>",
    "etag": "<etag>",
    "displayName": "Sensor 1",
    "instanceOf": "<device template Id>",
    "simulated": false,
    "provisioned": true,
    "approved": true,
    "properties": {
        "sensorComponent": {
            "setTemp": "30",
            "fwVersion": "2.0.1",
            "status": { "first": "first", "second": "second" },
            "$metadata": {
                "setTemp": {
                    "desiredValue": "30",
                    "desiredVersion": 3,
                    "desiredTimestamp": "2020-02-01T17:15:08.9284049Z",
                    "ackVersion": 3
                },
                "fwVersion": { "ackVersion": 3 },
                "status": {
                    "desiredValue": {
                        "first": "first",
                        "second": "second"
                    },
                    "desiredVersion": 2,
                    "desiredTimestamp": "2020-02-01T17:15:08.9284049Z",
                    "ackVersion": 2
                }
            },
            
        }
    },
    "installDate": { "installDate": "2020-02-01" }
},
  "annotations":{
    "iotcentral-message-source":"devices",
    "x-opt-partition-key":"<partitionKey>",
    "x-opt-sequence-number":39740,
    "x-opt-offset":"<offset>",
    "x-opt-enqueued-time":1539274959654
  },
  "partitionKey":"<partitionKey>",
  "sequenceNumber":39740,
  "enqueuedTimeUtc":"2020-02-01T18:14:49.3820326Z",
  "offset":"<offset>"
}
```

Этот моментальный снимок представляет собой пример сообщения, показывающего устройства и свойства данных в хранилище BLOB-объектов. Экспортируемые файлы содержат одну строку для каждой записи.

```json
{
  "id": "<device Id>",
  "etag": "<etag>",
  "displayName": "Sensor 1",
  "instanceOf": "<device template Id>",
  "simulated": false,
  "provisioned": true,
  "approved": true,
  "properties": {
      "sensorComponent": {
          "setTemp": "30",
          "fwVersion": "2.0.1",
          "status": { "first": "first", "second": "second" },
          "$metadata": {
              "setTemp": {
                  "desiredValue": "30",
                  "desiredVersion": 3,
                  "desiredTimestamp": "2020-02-01T17:15:08.9284049Z",
                  "ackVersion": 3
              },
              "fwVersion": { "ackVersion": 3 },
              "status": {
                  "desiredValue": {
                      "first": "first",
                      "second": "second"
                  },
                  "desiredVersion": 2,
                  "desiredTimestamp": "2020-02-01T17:15:08.9284049Z",
                  "ackVersion": 2
              }
          },
          
      }
  },
  "installDate": { "installDate": "2020-02-01" }
}
```

## <a name="device-templates"></a>Шаблоны устройств

Каждое сообщение или запись моментального снимка представляет одно или несколько изменений шаблона опубликованного устройства с момента последнего экспортированного сообщения. Данные, отправляемые в каждое сообщение или запись, включают:

- `id` шаблона устройства, соответствующего указанному `instanceOf` выше потоку устройств
- версию (`displayName`) шаблона устройства;
- Устройство `capabilityModel` , включая его `interfaces` определения, и данные телеметрии, свойства и команды
- `cloudProperties` вирус
- Переопределения и начальные значения, встроенные в `capabilityModel`

Удаленные шаблоны устройств не экспортируются. В настоящее время в сообщениях экспорта нет никаких индикаторов касательно удаленных шаблонов устройств.

Для концентраторов событий и служебной шины IoT Central отправляет сообщения, содержащие данные шаблона устройства, в концентратор событий или в очередь или раздел служебной шины практически в реальном времени.

Для хранилища BLOB-объектов новый моментальный снимок, содержащий все изменения с момента последнего записанного, экспортируется один раз в минуту.

В этом примере показано сообщение с данными о шаблонах устройств в концентраторе событий или в очереди или разделе служебной шины:

```json
{
  "body":{
      "id": "<device template id>",
      "etag": "<etag>",
      "types": ["DeviceModel"],
      "displayName": "Sensor template",
      "capabilityModel": {
          "@id": "<capability model id>",
          "@type": ["CapabilityModel"],
          "contents": [],
          "implements": [
              {
                  "@id": "<component Id>",
                  "@type": ["InterfaceInstance"],
                  "name": "sensorComponent",
                  "schema": {
                      "@id": "<interface Id>",
                      "@type": ["Interface"],
                      "displayName": "Sensor interface",
                      "contents": [
                          {
                              "@id": "<id>",
                              "@type": ["Telemetry"],
                              "displayName": "Humidity",
                              "name": "humidity",
                              "schema": "double"
                          },
                          {
                              "@id": "<id>",
                              "@type": ["Telemetry", "SemanticType/Event"],
                              "displayName": "Error event",
                              "name": "error",
                              "schema": "integer"
                          },
                          {
                              "@id": "<id>",
                              "@type": ["Property"],
                              "displayName": "Set temperature",
                              "name": "setTemp",
                              "writable": true,
                              "schema": "integer",
                              "unit": "Units/Temperature/fahrenheit",
                              "initialValue": "30"
                          },
                          {
                              "@id": "<id>",
                              "@type": ["Property"],
                              "displayName": "Firmware version read only",
                              "name": "fwversion",
                              "schema": "string"
                          },
                          {
                              "@id": "<id>",
                              "@type": ["Property"],
                              "displayName": "Display status",
                              "name": "status",
                              "writable": true,
                              "schema": {
                                  "@id": "urn:testInterface:status:obj:ka8iw8wka:1",
                                  "@type": ["Object"]
                              }
                          },
                          {
                              "@id": "<id>",
                              "@type": ["Command"],
                              "request": {
                                  "@id": "<id>",
                                  "@type": ["SchemaField"],
                                  "displayName": "Configuration",
                                  "name": "config",
                                  "schema": "string"
                              },
                              "response": {
                                  "@id": "<id>",
                                  "@type": ["SchemaField"],
                                  "displayName": "Response",
                                  "name": "response",
                                  "schema": "string"
                              },
                              "displayName": "Configure sensor",
                              "name": "sensorConfig"
                          }
                      ]
                  }
              }
          ],
          "displayName": "Sensor capability model"
      },
      "solutionModel": {
          "@id": "<id>",
          "@type": ["SolutionModel"],
          "cloudProperties": [
              {
                  "@id": "<id>",
                  "@type": ["CloudProperty"],
                  "displayName": "Install date",
                  "name": "installDate",
                  "schema": "dateTime",
                  "valueDetail": {
                      "@id": "<id>",
                      "@type": ["ValueDetail/DateTimeValueDetail"]
                  }
              }
          ]
      }
  },
    "annotations":{
      "iotcentral-message-source":"deviceTemplates",
      "x-opt-partition-key":"<partitionKey>",
      "x-opt-sequence-number":25315,
      "x-opt-offset":"<offset>",
      "x-opt-enqueued-time":1539274985085
    },
    "partitionKey":"<partitionKey>",
    "sequenceNumber":25315,
    "enqueuedTimeUtc":"2019-10-02T16:23:05.085Z",
    "offset":"<offset>"
  }
}
```

Этот пример снимка показывает сообщение, содержащее данные об устройстве и свойствах в хранилище BLOB-объектов. Экспортируемые файлы содержат одну строку для каждой записи.

```json
{
      "id": "<device template id>",
      "etag": "<etag>",
      "types": ["DeviceModel"],
      "displayName": "Sensor template",
      "capabilityModel": {
          "@id": "<capability model id>",
          "@type": ["CapabilityModel"],
          "contents": [],
          "implements": [
              {
                  "@id": "<component Id>",
                  "@type": ["InterfaceInstance"],
                  "name": "Sensor component",
                  "schema": {
                      "@id": "<interface Id>",
                      "@type": ["Interface"],
                      "displayName": "Sensor interface",
                      "contents": [
                          {
                              "@id": "<id>",
                              "@type": ["Telemetry"],
                              "displayName": "Humidity",
                              "name": "humidity",
                              "schema": "double"
                          },
                          {
                              "@id": "<id>",
                              "@type": ["Telemetry", "SemanticType/Event"],
                              "displayName": "Error event",
                              "name": "error",
                              "schema": "integer"
                          },
                          {
                              "@id": "<id>",
                              "@type": ["Property"],
                              "displayName": "Set temperature",
                              "name": "setTemp",
                              "writable": true,
                              "schema": "integer",
                              "unit": "Units/Temperature/fahrenheit",
                              "initialValue": "30"
                          },
                          {
                              "@id": "<id>",
                              "@type": ["Property"],
                              "displayName": "Firmware version read only",
                              "name": "fwversion",
                              "schema": "string"
                          },
                          {
                              "@id": "<id>",
                              "@type": ["Property"],
                              "displayName": "Display status",
                              "name": "status",
                              "writable": true,
                              "schema": {
                                  "@id": "urn:testInterface:status:obj:ka8iw8wka:1",
                                  "@type": ["Object"]
                              }
                          },
                          {
                              "@id": "<id>",
                              "@type": ["Command"],
                              "request": {
                                  "@id": "<id>",
                                  "@type": ["SchemaField"],
                                  "displayName": "Configuration",
                                  "name": "config",
                                  "schema": "string"
                              },
                              "response": {
                                  "@id": "<id>",
                                  "@type": ["SchemaField"],
                                  "displayName": "Response",
                                  "name": "response",
                                  "schema": "string"
                              },
                              "displayName": "Configure sensor",
                              "name": "sensorconfig"
                          }
                      ]
                  }
              }
          ],
          "displayName": "Sensor capability model"
      },
      "solutionModel": {
          "@id": "<id>",
          "@type": ["SolutionModel"],
          "cloudProperties": [
              {
                  "@id": "<id>",
                  "@type": ["CloudProperty"],
                  "displayName": "Install date",
                  "name": "installDate",
                  "schema": "dateTime",
                  "valueDetail": {
                      "@id": "<id>",
                      "@type": ["ValueDetail/DateTimeValueDetail"]
                  }
              }
          ]
      }
  }
```

## <a name="data-format-change-notice"></a>Уведомление об изменении формата данных

> [!Note]
> Это изменение не влияет на формат данных потока телеметрии. Затрагиваются только потоки данных для устройств и шаблонов устройств.

При наличии в предварительной версии приложения экспорта данных с включенными потоками *устройств* и *шаблонов устройств* обновите экспорт на **30 июня 2020**. Это требование относится к экспорту в хранилище BLOB-объектов Azure, концентраторам событий Azure и служебной шине Azure.

Начиная с 3 февраля 2020, все новые операции экспорта в приложениях с включенными устройствами и шаблонами устройств будут иметь формат данных, описанный выше. Все экспортируемые объекты, созданные до этой даты, остаются в старом формате до 30 июня 2020, при этом экспорт автоматически переносится в новый формат данных. Новый формат данных соответствует [устройству](/rest/api/iotcentral/devices/get), [свойству устройства](/rest/api/iotcentral/devices/getproperties), [свойству облака устройства](/rest/api/iotcentral/devices/getcloudproperties)и объектам [шаблона устройства](/rest/api/iotcentral/devicetemplates/get) в общедоступном API IOT Central.

Для **устройств** существенными различиями между старым форматом данных и новым форматом данных являются:
- `@id` для устройства удалено `deviceId` переименовывается в `id` 
- `provisioned` добавлен флаг, описывающий состояние подготовки устройства.
- `approved` добавлен флаг, описывающий состояние утверждения устройства.
- `properties` включая свойства устройства и облака, сопоставляет сущности в общедоступном API

Для **шаблонов устройств** существенными различиями между старым форматом данных и новым форматом данных являются:

- `@id` для шаблона устройства переименовывается в `id`
- `@type` для шаблона устройства переименовывается в `types` , а теперь является массивом

### <a name="devices-format-deprecated-as-of-3-february-2020"></a>Устройства (формат не рекомендуется к 3 февраля 2020)

```json
{
  "@id":"<id-value>",
  "@type":"Device",
  "displayName":"Airbox",
  "data":{
    "$cloudProperties":{
        "Color":"blue"
    },
    "EnvironmentalSensor":{
      "thsensormodel":{
        "reported":{
          "value":"Neque quia et voluptatem veritatis assumenda consequuntur quod.",
          "$lastUpdatedTimestamp":"2019-09-30T20:35:43.8478978Z"
        }
      },
      "pm25sensormodel":{
        "reported":{
          "value":"Aut alias odio.",
          "$lastUpdatedTimestamp":"2019-09-30T20:35:43.8478978Z"
        }
      }
    },
    "urn_azureiot_DeviceManagement_DeviceInformation":{
      "totalStorage":{
        "reported":{
          "value":27900.9730905171,
          "$lastUpdatedTimestamp":"2019-09-30T20:35:43.8478978Z"
        }
      },
      "totalMemory":{
        "reported":{
          "value":4667.82916715811,
          "$lastUpdatedTimestamp":"2019-09-30T20:35:43.8478978Z"
        }
      }
    }
  },
  "instanceOf":"<template-id>",
  "deviceId":"<device-id>",
  "simulated":true
}
```

### <a name="device-templates-format-deprecated-as-of-3-february-2020"></a>Шаблоны устройств (формат не рекомендуется к 3 февраля 2020)

```json
{
  "@id":"<template-id>",
  "@type":"DeviceModelDefinition",
  "displayName":"Airbox",
  "capabilityModel":{
    "@id":"<id>",
    "@type":"CapabilityModel",
    "implements":[
      {
        "@id":"<id>",
        "@type":"InterfaceInstance",
        "name":"EnvironmentalSensor",
        "schema":{
          "@id":"<id>",
          "@type":"Interface",
          "comment":"Requires temperature and humidity sensors.",
          "description":"Provides functionality to report temperature, humidity. Provides telemetry, commands and read-write properties",
          "displayName":"Environmental Sensor",
          "contents":[
            {
              "@id":"<id>",
              "@type":"Telemetry",
              "description":"Current temperature on the device",
              "displayName":"Temperature",
              "name":"temp",
              "schema":"double",
              "unit":"Units/Temperature/celsius",
              "valueDetail":{
                "@id":"<id>",
                "@type":"ValueDetail/NumberValueDetail",
                "minValue":{
                  "@value":"50"
                }
              },
              "visualizationDetail":{
                "@id":"<id>",
                "@type":"VisualizationDetail"
              }
            },
            {
              "@id":"<id>",
              "@type":"Telemetry",
              "description":"Current humidity on the device",
              "displayName":"Humidity",
              "name":"humid",
              "schema":"integer"
            },
            {
              "@id":"<id>",
              "@type":"Telemetry",
              "description":"Current PM2.5 on the device",
              "displayName":"PM2.5",
              "name":"pm25",
              "schema":"integer"
            },
            {
              "@id":"<id>",
              "@type":"Property",
              "description":"T&H Sensor Model Name",
              "displayName":"T&H Sensor Model",
              "name":"thsensormodel",
              "schema":"string"
            },
            {
              "@id":"<id>",
              "@type":"Property",
              "description":"PM2.5 Sensor Model Name",
              "displayName":"PM2.5 Sensor Model",
              "name":"pm25sensormodel",
              "schema":"string"
            }
          ]
        }
      },
      {
        "@id":"<id>",
        "@type":"InterfaceInstance",
        "name":"urn_azureiot_DeviceManagement_DeviceInformation",
        "schema":{
          "@id":"<id>",
          "@type":"Interface",
          "displayName":"Device information",
          "contents":[
            {
              "@id":"<id>",
              "@type":"Property",
              "comment":"Total available storage on the device in kilobytes. Ex. 20480000 kilobytes.",
              "displayName":"Total storage",
              "name":"totalStorage",
              "displayUnit":"kilobytes",
              "schema":"long"
            },
            {
              "@id":"<id>",
              "@type":"Property",
              "comment":"Total available memory on the device in kilobytes. Ex. 256000 kilobytes.",
              "displayName":"Total memory",
              "name":"totalMemory",
              "displayUnit":"kilobytes",
              "schema":"long"
            }
          ]
        }
      }
    ],
    "displayName":"AAEONAirbox52"
  },
  "solutionModel":{
    "@id":"<id>",
    "@type":"SolutionModel",
    "cloudProperties":[
      {
        "@id":"<id>",
        "@type":"CloudProperty",
        "displayName":"Color",
        "name":"Color",
        "schema":"string",
        "valueDetail":{
          "@id":"<id>",
          "@type":"ValueDetail/StringValueDetail"
        },
        "visualizationDetail":{
          "@id":"<id>",
          "@type":"VisualizationDetail"
        }
      }
    ]
  }
}
```

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы узнали, как экспортировать данные в концентраторы событий Azure, служебную шину Azure и хранилище BLOB-объектов Azure, перейдите к следующему шагу:

> [!div class="nextstepaction"]
> [Как запустить пользовательскую аналитику с модулями](./howto-create-custom-analytics.md)