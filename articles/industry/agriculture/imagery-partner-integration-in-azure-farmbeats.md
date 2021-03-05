---
title: Интеграция с партнерскими решениями для получения снимков
description: В этой статье описывается интеграция партнерских партнеров.
author: uhabiba04
ms.topic: article
ms.date: 11/04/2019
ms.author: v-ummehabiba
ms.custom: has-adal-ref
ms.openlocfilehash: 1bc723892e29ccc24a7612cfbe50d4b274cdacdf
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102183800"
---
# <a name="imagery-partner-integration"></a>Интеграция с партнерскими решениями для получения снимков

В этой статье описывается, как использовать компонент Azure Фармбеатс Translator для отправки данных с изображениями в Фармбеатс. Сельскохозяйственное графические данные могут создаваться из различных источников, таких как камеры мултиспектрал, спутники и дроны. Партнеры по образам сельскохозяйственное могут интегрироваться с Фармбеатс, чтобы предоставить клиентам созданные пользователем карты для своих ферм.

Данные, будучи доступными, могут быть отображены с помощью ускорителя Фармбеатс и могут использоваться для создания данных и модели машинного обучения и искусственного интеллекта (ML/AI), создаваемых сельскохозяйственное предприятиями или интеграторами систем клиентов.

Фармбеатс предоставляет следующие возможности:

- Определите пользовательские типы изображений, источник и формат файлов с помощью API-интерфейсов/Екстендедтипе.
- Прием данных с изображениями из различных источников через API-интерфейсы/сцене и/Сценефиле.

Следующие сведения посвящены получению любой формы изображений в систему Фармбеатс.

При выборе раздела **изображения помощью Дронов** открывается всплывающее окно, в котором отображается изображение помощью Дронов орсомосаик с высоким разрешением. Вы можете получить доступ к программному обеспечению Partner, которое помогает планировать помощью дроновные авиарейсы и получать необработанные данные. Вы по-прежнему будете использовать программное обеспечение партнера для планирования пути и сшивания изображений орсомосаик.

Партнеры помощью Дронов должны позволить клиентам связывать свои учетные записи клиентов с их экземпляром Фармбеатс в Azure.

Для связывания Фармбеатс необходимо использовать следующие учетные данные в программном обеспечении партнера помощью Дронов:

- Конечная точка API
- Tenant ID
- Идентификатор клиента
- Секрет клиента

## <a name="api-development"></a>Разработка API

API-интерфейсы содержат техническую документацию по Swagger. Дополнительные сведения об API и соответствующих запросах или ответах см. в разделе [Swagger](https://aka.ms/FarmBeatsDatahubSwagger).

## <a name="authentication"></a>Аутентификация

Фармбеатс использует [Active Directory](../../app-service/overview-authentication-authorization.md) Microsoft Azure (Azure AD).  Служба приложений Azure предоставляет встроенную поддержку проверки подлинности и авторизации в службе приложений. 

Дополнительные сведения об Azure AD см. в [этой статье](../../app-service/overview-authentication-authorization.md).   

FarmBeats Datahub использует проверку подлинности носителя, для которой требуются следующие учетные данные:

- Идентификатор клиента
- Секрет клиента
- Tenant ID

Используя предыдущие учетные данные, вызывающий объект может запросить маркер доступа, который должен быть отправлен в последующих запросах API, в разделе заголовка следующим образом:

```
headers = {"Authorization": "Bearer " + access_token, …} 
```

В следующем примере кода Python извлекается маркер доступа. Затем можно использовать маркер для последующих вызовов API к Фармбеатс.

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

## <a name="http-request-headers"></a>Заголовки запросов HTTP

Ниже приведены наиболее распространенные заголовки запросов, которые необходимо указать при вызове API к FarmBeats Datahub.

**Верхняя часть** | **Описание и пример**
--- | ---
Content-Type  | Формат запроса (Content-Type: application/<format>). Для API-интерфейсов FarmBeats Datahub используется формат JSON. Content-Type: application/json
Авторизация | Указывает маркер доступа, необходимый для выполнения вызова API. Авторизация: Bearer <маркер доступа>
Принять  | Формат ответа. Для API-интерфейсов FarmBeats Datahub используется формат JSON. Accept: application/json


## <a name="api-requests"></a>Запросы API

Чтобы выполнить запрос REST API, объедините следующие действия:

- Метод HTTP (GET, POST и помещения).
- URL-адрес службы API.
- URI ресурса (для запроса, отправки данных, обновления или удаления).
- Один или несколько заголовков HTTP-запроса.

При необходимости в вызовы GET можно включить параметры запроса для фильтрации, ограничения размера и сортировки данных в ответах.

Следующий пример запроса — получение списка устройств:

```bash
curl -X GET "https://microsoft-farmbeats.azurewebsites.net/Device" -H
"Content-Type: application/json" -H
"Authorization: Bearer <Access-Token>”
```

Для большинства вызовов GET, POST и PUT требуется текст запроса JSON.

Следующий пример запроса предназначен для создания устройства. Этот пример содержит входной JSON с текстом запроса.


```bash
curl -X POST "https://microsoft-farmbeats.azurewebsites.net/Device" -H
"accept: application/json" -H
"Content-Type: application/json" -H "Authorization: Bearer <Access-Token>" -d
"{  \"deviceModelId\": \"ID123\",  \"hardwareId\": \"MHDN123\",  \"reportingInterval\": 900,
\"name\": \"Device123\",  \"description\": \"Test Device 123\",}"
```

## <a name="data-format"></a>Формат данных

JSON является стандартным не зависящим от языка форматом данных, предоставляющим простое текстовое представление произвольных структур данных. Дополнительные сведения см. в разделе [JSON org](https://JSON.org).

## <a name="ingest-imagery-into-farmbeats"></a>Прием изображений в Фармбеатс

После того как у партнера есть учетные данные для подключения к Фармбеатс Датахуб, партнер выполняет следующие действия в компоненте Translator.

1.  Создайте новый расширенный тип для следующих полей в соответствии с типом загружаемых изображений:

    - **Источник сцены**: например, drone_partner_name
    - **Тип сцены**: например, помощью Дронов
    - **Тип файла сцены**: например, члорофилл index
    - **Тип содержимого файла сцены**: например, Image/TIFF

2.  Вызовите API/Фармс, чтобы получить список ферм из системы Azure Фармбеатс.
3.  Предоставьте клиенту возможность выбора отдельной фермы из списка ферм.

    Партнерская система должна показывать ферму в рамках программного обеспечения партнера, чтобы выполнить планирование пути и помощью Дронов рейса и коллекцию образов.

4.  Вызовите API/сцене и укажите необходимые сведения для создания новой сцены с уникальным ИДЕНТИФИКАТОРом сцены.
5.  Получите URL-адрес SAS BLOB, чтобы отправить необходимые образы в Фармбеатс Датахуб в контексте выбранной фермы в системе Фармбеатс.

Ниже приведен подробный поток вызовов API.

### <a name="step-1-extendedtype"></a>Шаг 1. ExtendedType

Проверьте API/Екстендедтипе, чтобы узнать, доступны ли тип и источник файла в Фармбеатс. Для этого вызовите GET в API/Екстендедтипе.

Ниже приведены определяемые системой значения.

```json
{
  "items": [
    {
      "id": "ae2aa527-7d27-42e2-8fcf-841937c651d7",
      "createdAt": "2019-06-17T07:32:30.1135716Z",
      "lastModifiedAt": "2019-10-25T09:47:42.5910344Z",
      "key": "SceneFileType",
      "value": [
        "evi",
        "ndvi",
        "ndwi",
        "tci",
        "soil-moisture",
        "sensor-placement",
        "sentinel-b01",
        "sentinel-b02",
        "sentinel-b03",
        "sentinel-b04",
        "sentinel-b05",
        "sentinel-b06",
        "sentinel-b07",
        "sentinel-b08",
        "sentinel-b8a",
        "sentinel-b09",
        "sentinel-b10",
        "sentinel-b11",
        "sentinel-b12",
        "cloud-mask",
        "farm-mask",
        "image/tiff",
        "orthomosaic"
      ],
      "description": "List of scene file types available in system. User can add more values."
    },
    {
      "id": "b7824d2e-3e43-46d3-8fd4-ec7ba8918d84",
      "createdAt": "2019-06-17T07:32:30.1135716Z",
      "lastModifiedAt": "2019-10-25T09:45:44.8914691Z",
      "key": "SceneFileContentType",
      "value": [
        "image/tiff",
        "image/png",
        "image/jpeg",
        "text/csv",
        "text/plain",
        "text/tab-separated-values",
        "application/json",
        "application/octet-stream",
        "orthomosaic"
      ],
      "description": "List of scene file content types available in system. User can add more values."
    },
    {
      "id": "21d2924d-8b3e-4ce4-a7f8-318e85c0f7f2",
      "createdAt": "2019-06-17T07:32:30.1135716Z",
      "lastModifiedAt": "2019-10-25T09:45:37.5096445Z",
      "key": "SceneSource",
      "value": [
        "sentinel-l1c",
        "sentinel-l2a",
        "farmbeats-model",
        "dji",
        "SlantRange-3P",
        "DJI"
      ],
      "description": "List of scene sources available in system. User can add more values."
    },
    {
      "id": "6e09ca69-cacf-4b53-b63a-53c659aae4a4",
      "createdAt": "2019-06-17T07:32:30.1135716Z",
      "lastModifiedAt": "2019-10-25T09:45:34.7483899Z",
      "key": "SceneType",
      "value": [
        "base-bands",
        "sensor-placement",
        "soil-moisture",
        "evi",
        "ndwi",
        "ndvi",
        "drone",
        "orthomosaic-drone"
      ],
      "description": "List of scene types available in system. User can add more values."
    },
    {
      "id": "55d8436b-3a86-4bea-87ab-e1b51226525f",
      "createdAt": "2019-06-17T07:32:30.1135716Z",
      "lastModifiedAt": "2019-06-17T07:32:30.1135716Z",
      "key": "SensorMeasureType",
      "value": [
        "AmbientTemperature",
        "CO2",
        "Depth",
        "ElectricalConductivity",
        "LeafWetness",
        "Length",
        "LiquidLevel",
        "Nitrate",
        "O2",
        "PH",
        "Phosphate",
        "PointInTime",
        "Potassium",
        "Pressure",
        "RainGauge",
        "RelativeHumidity",
        "Salinity",
        "SoilMoisture",
        "SoilTemperature",
        "SolarRadiation",
        "State",
        "TimeDuration",
        "UVRadiation",
        "UVIndex",
        "Volume",
        "WindDirection",
        "WindRun",
        "WindSpeed",
        "Evapotranspiration",
        "PAR"
      ],
      "description": "List of sensor measure types available in system. User can add more values."
    },
    {
      "id": "0dfd6e6b-df58-428f-9ab8-a0674bfdcbe5",
      "createdAt": "2019-06-17T07:32:30.1135716Z",
      "lastModifiedAt": "2019-06-17T07:32:30.1135716Z",
      "key": "SensorMeasureUnit",
      "value": [
        "NoUnit",
        "Celsius",
        "Fahrenheit",
        "Kelvin",
        "Rankine",
        "Pascal",
        "Mercury",
        "PSI",
        "MilliMeter",
        "CentiMeter",
        "Meter",
        "Inch",
        "Feet",
        "Mile",
        "KiloMeter",
        "MilesPerHour",
        "MilesPerSecond",
        "KMPerHour",
        "KMPerSecond",
        "MetersPerHour",
        "MetersPerSecond",
        "Degree",
        "WattsPerSquareMeter",
        "KiloWattsPerSquareMeter",
        "MilliWattsPerSquareCentiMeter",
        "MilliJoulesPerSquareCentiMeter",
        "VolumetricWaterContent",
        "Percentage",
        "PartsPerMillion",
        "MicroMol",
        "MicroMolesPerLiter",
        "SiemensPerSquareMeterPerMole",
        "MilliSiemensPerCentiMeter",
        "Centibar",
        "DeciSiemensPerMeter",
        "KiloPascal",
        "VolumetricIonContent",
        "Liter",
        "MilliLiter",
        "Seconds",
        "UnixTimestamp",
        "MicroMolPerMeterSquaredPerSecond",
        "InchesPerHour"
      ],
      "description": "List of sensor measure units available in system. User can add more values."
    }
  ]
}
```

Этот шаг является одноразовой настройкой. Область этого нового типа сцены ограничена подпиской, в которой установлена служба Azure Фармбеатс.

Например, чтобы добавить Сценесаурце: "Слантранже", вы замещаете идентификатор API/Екстендедтипе с помощью входных полезных данных "Сценесаурце".

```json
{

      "key": "SceneSource",
      "value": [
        "sentinel-l1c",
        "sentinel-l2a",
        "farmbeats-model",
        "dji",
        "SlantRange-3P"
      ],
      "description": "List of scene sources available in system. User can add more values."
}

```

Зеленым полем является новое добавление к системным исходным значениям сцены.

### <a name="step-2-get-farm-details"></a>Шаг 2. Получение сведений о ферме

Сцены (TIFF-или CSV-файлы) находятся в контексте фермы. Необходимо получить сведения о ферме, выполнив команду GET в API/Фарм. API возвращает список ферм, доступных в Фармбеатс. Вы можете выбрать ферму для приема данных.

ПОЛУЧИТЬ ответ/Фарм:

```json
{
  "id": "07f3e09c-89aa-4619-8d50-e57fb083d8f9",
  "createdAt": "2019-11-01T13:55:41.8804663Z",
  "lastModifiedAt": "2019-11-01T13:55:41.8804663Z",
  "geometry": {
    "type": "Polygon",
    "coordinates": [
      [
        [
          78.34252251428694,
          17.402556352862675
        ],
        [
          78.34278000635095,
          17.407920852463022
        ],
        [
          78.34883106989963,
          17.40878079576528
        ],
        [
          78.3498181228054,
          17.403539173730607
        ],
        [
          78.34805859369493,
          17.39977166504127
        ],
        [
          78.34252251428694,
          17.402556352862675
        ]
      ]
    ]
  },
  "name": "Samplefarm",
  "properties": {
    "crops": "Corn",
    "address": "Sample street"
  }
}
 ```

### <a name="step-3-create-a-scene-id-post-call"></a>Шаг 3. Создание идентификатора сцены (вызов POST)

Создайте новую сцену (TIFF-или CSV-файл) с заданной информацией, которая содержит дату, последовательность и идентификатор фермы, с которыми связана сцена. Метаданные, связанные с сценой, могут быть определены в свойствах, которые включают в себя длительность и тип меры.

Создание сцены создает новый идентификатор сцены, связанный с фермой. После создания идентификатора сцены пользователь может использовать то же самое для создания нового файла (. TIFF или. csv) и сохранения содержимого файла.

Пример входных полезных данных для вызова POST в API/сцене:

```json
{
  "sceneId": "a0505928-c480-491b-ba31-d38285a28c1d",
  "type": "newtype",
  "contentType": "image/tiff",
  "name": "test scene file",
  "description": "test scene file description",
  "properties": {
    "additionalProp1": {},
    "additionalProp2": {},
    "additionalProp3": {}
  }
}
```

Ответ API:

```json
{
  "id": "a0505928-c480-491b-ba31-d38285a28c1d",
  "createdAt": "2019-10-04T16:19:12.4838584Z",
  "lastModifiedAt": "2019-10-04T16:19:12.4838584Z",
  "type": "new type",
  "source": "SlantRange-3P",
  "farmId": "d41a33e7-b73e-480e-9279-0fcb3207332b",
  "date": "2019-10-04T16:13:39.064Z",
  "sequence": 5,
  "name": "test scene",
  "description": "test scene description",
  "properties": {}
}

```

**Создание файла сцены**

Идентификатор сцены, возвращенный на шаге 3, является входными данными для файла сцены. Файл сцены возвращает маркер URL-адреса SAS, который действителен в течение 24 часов.

Если пользователю требуется программный способ отправки потока изображений, можно использовать пакет SDK хранилища BLOB-объектов для определения метода с помощью идентификатора файла сцены, расположения и URL-адреса.

Пример входных полезных данных для вызова POST в API/Сценефиле:

```json
{
  "sceneId": "a0505928-c480-491b-ba31-d38285a28c1d",
  "type": "newtype",
  "contentType": "image/tiff",
  "name": "test scene file",
  "description": "test scene file description",
  "properties": {
    "additionalProp1": {},
    "additionalProp2": {},
    "additionalProp3": {}
  }
}
```
Ответ API:

```json
{
  "uploadSASUrl": "https://storagej2lho.blob.core.windows.net/farm-scene/2019/a0505928-c480-491b-ba31-d38285a28c1d/e91139a7-4ebd-4e2f-b17c-c677822dc840?sv=2018-03-28&sr=b&sig=%2F1426JkDcIFE5g3d%2BjOevCVMIn%2FJo9YKwBn3La5zL8Y%3D&se=2019-10-05T16%3A23%3A57Z&sp=w",
  "id": "e91139a7-4ebd-4e2f-b17c-c677822dc840",
  "createdAt": "2019-10-04T16:23:57.1192916Z",
  "lastModifiedAt": "2019-10-04T16:23:57.1192916Z",
  "blobUrl": "https://storagej2lho.blob.core.windows.net/farm-scene/2019/a0505928-c480-491b-ba31-d38285a28c1d/e91139a7-4ebd-4e2f-b17c-c677822dc840",
  "sceneId": "a0505928-c480-491b-ba31-d38285a28c1d",
  "type": "newtype",
  "contentType": "image/tiff",
  "name": "test scene file",
  "description": "test scene file description",
  "properties": {}
}


```

Вызов метода POST к API/Сценефиле возвращает URL-адрес отправки SAS, который можно использовать для передачи файла CSV или TIFF с помощью клиента или библиотеки хранилища BLOB-объектов Azure.


## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об интеграции на основе REST API см. в разделе [REST API](rest-api-in-azure-farmbeats.md).