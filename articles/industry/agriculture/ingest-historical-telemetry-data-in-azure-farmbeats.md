---
title: Прием архивных данных телеметрии
description: В этой статье описывается прием исторических данных телеметрии.
author: uhabiba04
ms.topic: article
ms.date: 11/04/2019
ms.author: v-ummehabiba
ms.custom: has-adal-ref
ms.openlocfilehash: e4182f3718422be3a0aa99c980d33026df4ce9af
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102173755"
---
# <a name="ingest-historical-telemetry-data"></a>Прием архивных данных телеметрии

В этой статье описывается прием исторических данных датчика в Azure Фармбеатс.

Получение исторических данных из ресурсов Интернет вещей (IoT), таких как устройства и датчики, является распространенным сценарием в Фармбеатс. Вы создаете метаданные для устройств и датчиков, а затем принимаете исторические данные в Фармбеатс в каноническом формате.

## <a name="before-you-begin"></a>Перед началом

Прежде чем продолжить работу с этой статьей, убедитесь, что вы установили Фармбеатс и собрали исторические данные с устройств IoT. Также необходимо включить Партнерский доступ, как упоминалось на следующих шагах.

## <a name="enable-partner-access"></a>Включить доступ партнеров

Необходимо включить интеграцию партнера с экземпляром Azure Фармбеатс. На этом шаге создается клиент, имеющий доступ к экземпляру Azure Фармбеатс в качестве партнера по устройствам, и предоставляются следующие значения, необходимые для последующих шагов:

- Конечная точка API: это URL-адрес Датахуб, например https:// \<datahub> . azurewebsites.NET
- Tenant ID
- Идентификатор клиента
- Секрет клиента
- Строка подключения EventHub

Выполните следующие действия:

> [!NOTE]
> Чтобы выполнить следующие действия, необходимо быть администратором.

1. Выполните вход в https://portal.azure.com/.

2. **Если вы используете Фармбеатс версии 1.2.7 или более поздней, пропустите шаги a, b и c и перейдите к шагу 3.** Чтобы проверить версию Фармбеатс, выберите значок **Параметры** в правом верхнем углу пользовательского интерфейса фармбеатс.

      а.  Переход к **Azure Active Directory**  >  **Регистрация приложений**

      b. Выберите **регистрацию приложения** , созданную в рамках развертывания фармбеатс. Имя будет совпадать с именем Фармбеатс датахуб.

      c. Выберите **открыть API** > выберите **добавить клиентское приложение** и введите **04B07795-8ddb-461a-bbEE-02f9e1bf7b46** и установите флажок **авторизовать область**. Это предоставит доступ к Azure CLI (Cloud Shell), чтобы выполнить следующие действия.

3. Откройте Cloud Shell. Этот параметр доступен на панели инструментов в правом верхнем углу портал Azure.

    ![Панель инструментов портал Azure](./media/get-drone-imagery-from-drone-partner/navigation-bar-1.png)

4. Убедитесь, что для среды задано значение **PowerShell**. По умолчанию он имеет значение bash.

    ![Настройка панели инструментов PowerShell](./media/get-sensor-data-from-sensor-partner/power-shell-new-1.png)

5. Перейдите к домашнему каталогу.

    ```azurepowershell-interactive
    cd
    ```

6. Выполните следующую команду. Он подключает учетную запись с проверкой подлинности для использования в запросах Azure AD.

    ```azurepowershell-interactive
    Connect-AzureAD
    ```

7. Выполните следующую команду. Это приведет к скачиванию скрипта в домашний каталог.

    ```azurepowershell-interactive 

    wget –q https://aka.ms/farmbeatspartnerscriptv3 -O ./generatePartnerCredentials.ps1

    ```

8. Выполните следующий сценарий. Сценарий запрашивает идентификатор клиента, который можно получить на   >  странице **обзора** Azure Active Directory.

    ```azurepowershell-interactive

    ./generatePartnerCredentials.ps1

    ```

9. Следуйте инструкциям на экране, чтобы записать значения для **конечных точек API**, **идентификаторов** **клиента, идентификаторов** клиентов, **секретов клиента** и **строки подключения EventHub**.


## <a name="create-device-or-sensor-metadata"></a>Создание метаданных устройства или датчика

 Теперь, когда у вас есть необходимые учетные данные, можно определить устройство и датчики. Для этого создайте метаданные, вызвав интерфейсы API Фармбеатс. Не забудьте вызвать API в качестве клиентского приложения, созданного в предыдущем разделе.

 Фармбеатс Датахуб содержит следующие API, позволяющие создавать метаданные устройства или датчика и управлять ими.

 > [!NOTE]
 > Как партнер, у которого есть доступ только для чтения, создания и обновления метаданных; **параметр удаления ограничен только партнером.**

- /**DeviceModel**: DeviceModel соответствует метаданным устройства, например изготовителю, и типу устройства, которое является либо шлюзом, либо узлом.
- /**Device**. Device соответствует физическому устройству, находящемуся в ферме.
- /**Сенсормодел**: сенсормодел соответствует метаданным датчика, например изготовителю, типу датчика, который является либо аналоговым, либо цифровым, и измерением датчика, таким как температура окружающей среды и давление.
- /**Sensor**. Sensor соответствует физическому датчику, записывающему значения. Датчик обычно подключается к устройству по идентификатору устройства.

| DeviceModel | Предложения |
|--|--|
| Type (node, gateway) | Тип устройства-узла или шлюза |
| Производитель | Имя производителя |
| ProductCode | Код продукта устройства или название или номер модели. Например, EnviroMonitor#6800. |
| порты; | Имя порта и тип (цифровой или аналоговый). |
| Имя | Имя для идентификации ресурса. Например, название модели или продукта. |
| Описание | Информативное описание модели. |
| Свойства | Дополнительные свойства от производителя. |
| **Устройство** |  |
| DeviceModelId | Идентификатор связанной модели устройства. |
| HardwareId | Уникальный идентификатор устройства, например MAC-адрес. |
| ReportingInterval | Отчетный интервал (в секундах). |
| Расположение | Широта устройства (от –90 до + 90), долгота (от –180 до 180) и высота над уровнем моря (в метрах). |
| ParentDeviceId | Идентификатор родительского устройства, к которому подключено это устройство. Например, узел, подключенный к шлюзу. Узел имеет Парентдевицеид в качестве шлюза. |
| Имя | Имя, идентифицирующее ресурс. Партнеры устройств должны отправить имя, которое соответствует имени устройства на стороне партнера. Если имя устройства партнера определено пользователем, то такое же определяемое пользователем имя должно распространяться на Фармбеатс. |
| Описание | Информативное описание. |
| Свойства | Дополнительные свойства от производителя. |
| **SensorModel** |  |
| Type (analog, digital) | Тип датчика, будь то аналоговый или цифровой. |
| Изготовитель | Производитель датчика. |
| ProductCode | Код продукта или название или номер модели. Например, RS-CO2-N01. |
| SensorMeasures > Name | Имя измерения датчика. Поддерживаются только строчные буквы. Для измерений с различных глубин указывается значение глубины. Например, soil_moisture_15cm. Это имя должно соответствовать данным телеметрии. |
| SensorMeasures > DataType | Тип данных телеметрии. Сейчас поддерживается тип double. |
| SensorMeasures > Type | Тип измерения данных телеметрии датчика. Определяемые системой типы: Амбиенттемпературе, ВЫБРОСы, глубина, Електрикалкондуктивити, Леафветнесс, длина, Ликуидлевел, нитрате, O2, PH, фосфате, PointInTime, potassium, давление, RainGauge, RelativeHumidity, Salinity, SoilMoisture, SoilTemperature, SolarRadiation, State, TimeDuration, UVRadiation, UVIndex, том, WindDirection, WindRun, WindSpeed, Evapotranspiration, номинал. Чтобы добавить дополнительные типы, используйте API /ExtendedType. |
| SensorMeasures > Unit | Единица данных телеметрии датчика. Определяемые системой единицы — это "ед., Цельсия, Фаренгейта, Кельвина, Ранкине, Pascal, Меркурий, PSI, миллиметр, сантиметры, метры, дюймы, футы, км, километр, Милесперхаур, Милесперсеконд, Кмперхаур, Кмперсеконд, Метерсперхаур, Метерсперсеконд, градусы, WattsPerSquareMeter, KiloWattsPerSquareMeter, MilliWattsPerSquareCentiMeter, MilliJoulesPerSquareCentiMeter, VolumetricWaterContent, процент, PartsPerMillion, MicroMol, MicroMolesPerLiter, SiemensPerSquareMeterPerMole, MilliSiemensPerCentiMeter, Centibar, DeciSiemensPerMeter, KiloPascal, VolumetricIonContent, Liter, MilliLiter. |
| SensorMeasures > AggregationType | Значения могут быть "None", "Average", "Maximum", "минимум" или "стандартное отклонение". |
| Имя | Имя, идентифицирующее ресурс. Например, название модели или продукта. |
| Описание | Информативное описание модели. |
| Свойства | Дополнительные свойства от производителя. |
| **Sensor** |  |
| HardwareId | Уникальный идентификатор датчика, заданный производителем. |
| SensorModelId | Идентификатор связанной модели датчика. |
| Расположение | Широта (от –90 до + 90), долгота (от –180 до 180) и высота датчика над уровнем моря (в метрах). |
| Port > Name | Имя и тип порта, к которому подключен датчик на устройстве. Это имя должно совпадать с именем, определенным в модели устройства. |
| DeviceID | Идентификатор устройства, к которому подключен датчик. |
| Имя | Имя для идентификации ресурса. Например, имя датчика или название продукта, номер модели или код продукта. |
| Описание | Информативное описание. |
| Свойства | Дополнительные свойства от производителя. |

Дополнительные сведения об объектах см. в разделе [Swagger](https://aka.ms/FarmBeatsDatahubSwagger).

### <a name="api-request-to-create-metadata"></a>Запрос API для создания метаданных

Чтобы выполнить запрос API, необходимо объединить метод HTTP (POST), URL-адрес службы API и URI для запроса, отправки данных в запрос, создания или удаления запроса. Затем добавьте один или несколько заголовков HTTP-запроса. URL-адрес службы API — это конечная точка API, то есть URL-адрес Датахуб (https:// \<yourdatahub> . azurewebsites.NET).

### <a name="authentication"></a>Аутентификация

Фармбеатс Датахуб использует проверку подлинности носителя, для которой требуются следующие учетные данные, созданные в предыдущем разделе.

- Идентификатор клиента
- Секрет клиента
- Tenant ID

Используя эти учетные данные, вызывающий объект может запросить маркер доступа. Маркер должен быть отправлен в последующих запросах API в разделе заголовка следующим образом:

```
headers = *{"Authorization": "Bearer " + access_token, …}*
```

В следующем примере кода Python предоставляется маркер доступа, который можно использовать для последующих вызовов API к Фармбеатс: 

```python
import requests
import json
import msal

# Your service principal App ID
CLIENT_ID = "<CLIENT_ID>"
# Your service principal password
CLIENT_SECRET = "<CLIENT_SECRET>"
# Tenant ID for your Azure subscription
TENANT_ID = "<TENANT_ID>"

AUTHORITY_HOST = 'https://login.microsoftonline.com'
AUTHORITY = AUTHORITY_HOST + '/' + TENANT_ID

ENDPOINT = "https://<yourfarmbeatswebsitename-api>.azurewebsites.net"
SCOPE = ENDPOINT + "/.default"

context = msal.ConfidentialClientApplication(CLIENT_ID, authority=AUTHORITY, client_credential=CLIENT_SECRET)
token_response = context.acquire_token_for_client(SCOPE)
# We should get an access token here
access_token = token_response.get('access_token')
```

**Заголовки запросов HTTP**

Ниже приведены наиболее распространенные заголовки запросов, которые необходимо указать при выполнении вызова API в Фармбеатс Датахуб:

- **Тип содержимого**: Application/JSON
- **Авторизация**: <носителя Access-Token>
- **Принять**: приложение/JSON

### <a name="input-payload-to-create-metadata"></a>Входные полезные данные для создания метаданных

DeviceModel


```json
{
  "type": "Node",
  "manufacturer": "string",
  "productCode": "string",
  "ports": [
    {
      "name": "string",
      "type": "Analog"
    }
  ],
  "name": "string",
  "description": "string",
  "properties": {
    "additionalProp1": {},
    "additionalProp2": {},
    "additionalProp3": {}
  }
}
```

Устройство

```json
{
  "deviceModelId": "string",
  "hardwareId": "string",
  "farmId": "string",
  "reportingInterval": 0,
  "location": {
    "latitude": 0,
    "longitude": 0,
    "elevation": 0
  },
  "parentDeviceId": "string",
  "name": "string",
  "description": "string",
  "properties": {
    "additionalProp1": {},
    "additionalProp2": {},
    "additionalProp3": {}
  }
}
```

SensorModel

```json
{
  "type": "Analog",
  "manufacturer": "string",
  "productCode": "string",
  "sensorMeasures": [
    {
      "name": "string",
      "dataType": "Double",
      "type": "string",
      "unit": "string",
      "aggregationType": "None",
      "depth": 0,
      "description": "string"
    }
  ],
  "name": "string",
  "description": "string",
  "properties": {
    "additionalProp1": {},
    "additionalProp2": {},
    "additionalProp3": {}
  }
}

```
Sensor

```json
{
  "hardwareId": "string",
  "sensorModelId": "string",
  "location": {
    "latitude": 0,
    "longitude": 0,
    "elevation": 0
  },
  "depth": 0,
  "port": {
    "name": "string",
    "type": "Analog"
  },
  "deviceId": "string",
  "name": "string",
  "description": "string",
  "properties": {
    "additionalProp1": {},
    "additionalProp2": {},
    "additionalProp3": {}
  }
}
```

Следующий пример запроса создает устройство. Этот запрос содержит входные данные JSON в качестве полезных данных с текстом запроса.

```bash
curl -X POST "https://<datahub>.azurewebsites.net/Device" -H
"accept: application/json" -H  "Content-Type: application/json" -H
"Authorization: Bearer <Access-Token>" -d "{  \"deviceModelId\": \"ID123\",  \"hardwareId\": \"MHDN123\",
\"reportingInterval\": 900,  \"name\": \"Device123\",
\"description\": \"Test Device 123\"}" *
```

Ниже приведен пример кода на языке Python. Маркер доступа, используемый в этом образце, совпадает с полученным во время проверки подлинности.

```python
import requests
import json

# Got access token - Calling the Device Model API

headers = {
    "Authorization": "Bearer " + access_token,
    "Content-Type" : "application/json"
    }
payload = '{"type" : "Node", "productCode" : "TestCode", "ports": [{"name": "port1","type": "Analog"}], "name" : "DummyDevice"}'
response = requests.post(ENDPOINT + "/DeviceModel", data=payload, headers=headers)
```

> [!NOTE]
> API-интерфейсы возвращают уникальные идентификаторы для каждого созданного экземпляра. Необходимо хранить идентификаторы для отправки соответствующих сообщений телеметрии.

### <a name="send-telemetry"></a>Отправка данных телеметрии

Теперь, когда вы создали устройства и датчики в Фармбеатс, вы можете отправить связанные сообщения телеметрии.

### <a name="create-a-telemetry-client"></a>Создание клиента телеметрии

Необходимо отправить данные телеметрии в концентраторы событий Azure для обработки. Центры событий Azure — это служба, которая обеспечивает прием данных (телеметрии), поступающих от подключенных устройств и приложений. Чтобы отправить данные телеметрии в FarmBeats, создайте клиент, отправляющий сообщения в концентратор событий в FarmBeats. Дополнительные сведения об отправке телеметрии см. в статье [концентраторы событий Azure](../../event-hubs/event-hubs-dotnet-standard-getstarted-send.md).

### <a name="send-a-telemetry-message-as-the-client"></a>Отправка сообщения телеметрии в качестве клиента

После подключения, установленного в качестве клиента концентраторов событий, можно отправить сообщения в концентратор событий в формате JSON.

Ниже приведен пример кода Python, который отправляет данные телеметрии в качестве клиента в указанный концентратор событий:

```python
import azure
from azure.eventhub import EventHubClient, Sender, EventData, Receiver, Offset
EVENTHUBCONNECTIONSTRING = "<EventHub Connection String provided by customer>"
EVENTHUBNAME = "<EventHub Name provided by customer>"

write_client = EventHubClient.from_connection_string(EVENTHUBCONNECTIONSTRING, eventhub=EVENTHUBNAME, debug=False)
sender = write_client.add_sender(partition="0")
write_client.run()
for i in range(5):
    telemetry = "<Canonical Telemetry message>"
    print("Sending telemetry: " + telemetry)
    sender.send(EventData(telemetry))
write_client.stop()

```

Преобразуйте формат исторических данных датчика в канонический формат, распознаваемый Azure Фармбеатс. Каноническое сообщение имеет следующий формат:

```json
{
"deviceid": "<id of the Device created>",
"timestamp": "<timestamp in ISO 8601 format>",
"version" : "1",
"sensors": [
    {
      "id": "<id of the sensor created>",
      "sensordata": [
        {
          "timestamp": "< timestamp in ISO 8601 format >",
          "<sensor measure name (as defined in the Sensor Model)>": <value>
        },
        {
          "timestamp": "<timestamp in ISO 8601 format>",
          "<sensor measure name (as defined in the Sensor Model)>": <value>
        }
      ]
    }
 ]
}
```

После добавления соответствующих устройств и датчиков получите идентификатор устройства и идентификатор датчика в сообщении телеметрии, как описано в предыдущем разделе.

Ниже приведен пример сообщения телеметрии.


 ```json
{
  "deviceid": "7f9b4b92-ba45-4a1d-a6ae-c6eda3a5bd12",
  "timestamp": "2019-06-22T06:55:02.7279559Z",
  "version": "1",
  "sensors": [
    {
      "id": "d8e7beb4-72de-4eed-9e00-45e09043a0b3",
      "sensordata": [
        {
          "timestamp": "2019-06-22T06:55:02.7279559Z",
          "hum_max": 15
        },
        {
          "timestamp": "2019-06-22T06:55:02.7279559Z",
          "hum_min": 42
        }
      ]
    },
    {
      "id": "d8e7beb4-72de-4eed-9e00-45e09043a0b3",
      "sensordata": [
        {
          "timestamp": "2019-06-22T06:55:02.7279559Z",
          "hum_max": 20
        },
        {
          "timestamp": "2019-06-22T06:55:02.7279559Z",
          "hum_min": 89
        }
      ]
    }
  ]
}
```

## <a name="troubleshooting"></a>Устранение неполадок

### <a name="cant-view-telemetry-data-after-ingesting-historicalstreaming-data-from-your-sensors"></a>Не удается просмотреть данные телеметрии после приема исторических и потоковых данных от датчиков

**Признак**. Устройства или датчики развернуты, вы создали устройства/датчики FarmBeats и приняли данные телеметрии в концентраторе событий, однако вам не удается получить или просмотреть данные телеметрии FarmBeats.

**Корректирующее действие**.

1. Убедитесь, что вы выполнили соответствующую регистрацию партнера. Вы можете проверить это, перейдя к датахуб Swagger, перейдите к/партнер API, выполните Get и проверьте, зарегистрирован ли партнер. В противном случае выполните [действия, описанные здесь](get-sensor-data-from-sensor-partner.md#enable-device-integration-with-farmbeats) , чтобы добавить партнера.

2. Убедитесь, что вы создали метаданные (DeviceModel, устройство, Сенсормодел, датчик), используя учетные данные клиента партнера.

3. Убедитесь, что вы использовали правильный формат сообщений телеметрии (как указано ниже):

```json
{
"deviceid": "<id of the Device created>",
"timestamp": "<timestamp in ISO 8601 format>",
"version" : "1",
"sensors": [
    {
      "id": "<id of the sensor created>",
      "sensordata": [
        {
          "timestamp": "< timestamp in ISO 8601 format >",
          "<sensor measure name (as defined in the Sensor Model)>": <value>
        },
        {
          "timestamp": "<timestamp in ISO 8601 format>",
          "<sensor measure name (as defined in the Sensor Model)>": <value>
        }
      ]
    }
 ]
}
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об интеграции на основе REST API см. в разделе [REST API](rest-api-in-azure-farmbeats.md).