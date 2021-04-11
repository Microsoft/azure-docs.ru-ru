---
title: 'Основные понятия: сопоставление шаблонов в соединителе Azure IoT для FHIR (Предварительная версия) функции Azure API для FHIR'
description: Узнайте, как создать два типа шаблонов сопоставления в соединителе Azure IoT для FHIR (Предварительная версия). Шаблон сопоставления устройств преобразует данные устройства в нормализованную схему. Шаблон сопоставления FHIR преобразует нормализованное сообщение в ресурс наблюдения FHIR.
services: healthcare-apis
author: ms-puneet-nagpal
ms.service: healthcare-apis
ms.subservice: iomt
ms.topic: conceptual
ms.date: 04/05/2021
ms.author: punagpal
ms.openlocfilehash: e43b5068544927ba4d04c86b16d3a15510c32bed
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2021
ms.locfileid: "106448478"
---
# <a name="azure-iot-connector-for-fhir-preview-mapping-templates"></a>Шаблоны сопоставления для соединителя "Azure IoT для FHIR" (предварительная версия)
В этой статье подробно описано, как настроить соединитель Azure IoT для быстрого обеспечения взаимодействия с сферами здравоохранения (FHIR&#174;) * с помощью шаблонов сопоставления.

Для соединителя Azure IoT для FHIR требуется два типа шаблонов сопоставления на основе JSON. Первый тип **сопоставления устройств** отвечает за сопоставление полезных данных устройства, отправленных в `devicedata` конечную точку концентратора событий Azure. Он извлекает типы, идентификаторы устройств, дату и время измерения, а также значения измерений. Второй тип, **FHIR сопоставление**, управляет сопоставлением для ресурса FHIR. Он позволяет настраивать длину периода наблюдения, тип данных FHIR, используемый для хранения значений, и кодов терминологии. 

Шаблоны сопоставления состоят из документа JSON на основе их типа. Затем эти документы JSON добавляются в соединитель Azure IoT для FHIR через портал Azure. Документ сопоставления устройств добавляется на странице **Настройка сопоставления устройств** и в документе сопоставления FHIR на странице **Настройка сопоставления FHIR** .

> [!NOTE]
> Шаблоны сопоставления хранятся в базовом хранилище BLOB-объектов и загружаются из большого двоичного объекта на выполнение вычислительных ресурсов. После обновления они вступят в силу немедленно. 

## <a name="device-mapping"></a>Сопоставление устройств
Сопоставление устройств предоставляет функции сопоставления для извлечения содержимого устройства в общий формат для дальнейшей оценки. Каждое полученное сообщение оценивается по всем шаблонам. Такой подход позволяет Проецирование одного входящего сообщения на несколько исходящих сообщений, которые позже сопоставляются с различными наблюдениями в FHIR. Результатом является Нормализованный объект данных, представляющий значение или значения, проанализированные шаблонами. Нормализованная модель данных содержит несколько обязательных свойств, которые необходимо найти и извлечь:

| Свойство | Описание |
| - | - |
|**Type**|Имя/тип для классификации измерения. Это значение используется для привязки к требуемому шаблону сопоставления FHIR.  Несколько шаблонов могут выводиться в один и тот же тип, что позволяет сопоставлять различные представления между несколькими устройствами с одним общим выходом.|
|**оккуренцетимеутк**|Время, когда произошло измерение.|
|**deviceId**|Идентификатор устройства. Это значение должно соответствовать идентификатору ресурса устройства, который существует на целевом сервере FHIR.|
 |**Свойства**|Извлеките по крайней мере одно свойство, чтобы значение можно было сохранить в созданном ресурсе наблюдения.  Свойства представляют собой коллекцию пар "ключ-значение", извлеченных во время нормализации.|

Ниже приведен концептуальный пример того, что происходит во время нормализации.

![Пример нормализации](media/concepts-iot-mapping-templates/normalization-example.png)

Полезная нагрузка содержимого — это сообщение концентратора событий Azure, которое состоит из трех частей: Body, Properties и Системпропертиес. `Body`Представляет собой массив байтов, представляющий строку в кодировке UTF-8. Во время оценки шаблона массив байтов автоматически преобразуется в строковое значение. `Properties` — Это коллекция значений ключей для использования создателем сообщений. `SystemProperties` — Это также коллекция значений ключей, зарезервированная платформой концентратора событий Azure, с автоматически заполненными записями.

```json
{
    "Body": {
        "content": "value"
    },
    "Properties": {
        "key1": "value1",
        "key2": "value2"
    },
    "SystemProperties": {
        "x-opt-sequence-number": 1,
        "x-opt-enqueued-time": "2019-02-01T22:46:01.8750000Z",
        "x-opt-offset": 1,
        "x-opt-partition-key": "1"
    }
}
```

### <a name="mapping-with-json-path"></a>Сопоставление с путем JSON
Три типа шаблонов содержимого устройств, которые поддерживаются сегодня, зависят от пути JSON к обоим шаблонам и извлеченным значениям. Дополнительные сведения о пути JSON можно найти [здесь](https://goessner.net/articles/JsonPath/). Все три типа шаблонов используют [реализацию JSON .NET](https://www.newtonsoft.com/json/help/html/QueryJsonSelectTokenJsonPath.htm) для разрешения выражений путей JSON.

#### <a name="jsonpathcontenttemplate"></a>жсонпасконтенттемплате
Жсонпасконтенттемплате позволяет сопоставлять и извлекать значения из сообщения концентратора событий с помощью пути JSON.

| Свойство | Описание |<div style="width:150px">Пример</div>
| --- | --- | --- 
|**Имени**|Тип, связываемый с измерениями, которые соответствуют шаблону.|`heartrate`
|**типематчекспрессион**|Выражение пути JSON, которое вычисляется для полезных данных концентратора событий. Если найден соответствующий JToken, шаблон считается совпадением. Все последующие выражения оцениваются по сравнению с извлеченным JToken, указанным здесь.|`$..[?(@heartRate)]`
|**тиместампекспрессион**|Выражение пути JSON для извлечения значения метки времени для Оккуренцетимеутк измерения.|`$.endDate`
|**девицеидекспрессион**|Выражение пути JSON для извлечения идентификатора устройства.|`$.deviceId`
|**патиентидекспрессион**|*Необязательно*: выражение пути JSON для извлечения идентификатора пациента.|`$.patientId`
|**енкаунтеридекспрессион**|*Необязательно*: выражение пути JSON для извлечения идентификатора обнаружения.|`$.encounterId`
|**Values []. ValueName**|Имя, связываемое со значением, извлеченным последующим выражением. Используется для привязки требуемого значения или компонента в шаблоне сопоставления FHIR. |`hr`
|**Values []. ValueExpression**|Выражение пути JSON для извлечения требуемого значения.|`$.heartRate`
|**Values []. Обязательно**|Необходимо, чтобы значение присутствовало в полезных данных.  Если объект не найден, измерение не будет сформировано и будет создано исключение InvalidOperationException.|`true`

##### <a name="examples"></a>Примеры
---
**Частота сердца**

*Message*
```json
{
    "Body": {
        "heartRate": "78",
        "endDate": "2019-02-01T22:46:01.8750000Z",
        "deviceId": "device123"
    },
    "Properties": {},
    "SystemProperties": {}
}
```
*Шаблон*
```json
{
    "templateType": "JsonPathContent",
    "template": {
        "typeName": "heartrate",
        "typeMatchExpression": "$..[?(@heartRate)]",
        "deviceIdExpression": "$.deviceId",
        "timestampExpression": "$.endDate",
        "values": [
            {
                "required": "true",
                "valueExpression": "$.heartRate",
                "valueName": "hr"
            }
        ]
    }
}
```
---
**Давление крови**

*Message*
```json
{
    "Body": {
        "systolic": "123",
        "diastolic" : "87",
        "endDate": "2019-02-01T22:46:01.8750000Z",
        "deviceId": "device123"
    },
    "Properties": {},
    "SystemProperties": {}
}
```
*Шаблон*
```json
{
    "typeName": "bloodpressure",
    "typeMatchExpression": "$..[?(@systolic && @diastolic)]",
    "deviceIdExpression": "$.deviceId",
    "timestampExpression": "$.endDate",
    "values": [
        {
            "required": "true",
            "valueExpression": "$.systolic",
            "valueName": "systolic"
        },
        {
            "required": "true",
            "valueExpression": "$.diastolic",
            "valueName": "diastolic"
        }
    ]
}
```
---

**Проецирование нескольких измерений из одного сообщения**

*Message*
```json
{
    "Body": {
        "heartRate": "78",
        "steps": "2",
        "endDate": "2019-02-01T22:46:01.8750000Z",
        "deviceId": "device123"
    },
    "Properties": {},
    "SystemProperties": {}
}
```
*Шаблон 1*
```json
{
    "templateType": "JsonPathContent",
    "template": {
        "typeName": "heartrate",
        "typeMatchExpression": "$..[?(@heartRate)]",
        "deviceIdExpression": "$.deviceId",
        "timestampExpression": "$.endDate",
        "values": [
            {
                "required": "true",
                "valueExpression": "$.heartRate",
                "valueName": "hr"
            }
        ]
    }
}
```
*Шаблон 2*
```json
{
    "templateType": "JsonPathContent",
    "template": {
        "typeName": "stepcount",
        "typeMatchExpression": "$..[?(@steps)]",
        "deviceIdExpression": "$.deviceId",
        "timestampExpression": "$.endDate",
        "values": [
            {
                "required": "true",
                "valueExpression": "$.steps",
                "valueName": "steps"
            }
        ]
    }
}
```
---

**Проецирование нескольких измерений из массива в сообщении**

*Message*
```json
{
    "Body": [
        {
            "heartRate": "78",
            "endDate": "2019-02-01T22:46:01.8750000Z",
            "deviceId": "device123"
        },
        {
            "heartRate": "81",
            "endDate": "2019-02-01T23:46:01.8750000Z",
            "deviceId": "device123"
        },
        {
            "heartRate": "72",
            "endDate": "2019-02-01T24:46:01.8750000Z",
            "deviceId": "device123"
        }
    ],
    "Properties": {},
    "SystemProperties": {}
}
```
*Шаблон*
```json
{
    "templateType": "JsonPathContent",
    "template": {
        "typeName": "heartrate",
        "typeMatchExpression": "$..[?(@heartRate)]",
        "deviceIdExpression": "$.deviceId",
        "timestampExpression": "$.endDate",
        "values": [
            {
                "required": "true",
                "valueExpression": "$.heartRate",
                "valueName": "hr"
            }
        ]
    }
}
```

#### <a name="iotjsonpathcontenttemplate"></a>иотжсонпасконтенттемплате

Иотжсонпасконтенттемплате похож на Жсонпасконтенттемплате, за исключением того, что Девицеидекспрессион и Тиместампекспрессион не требуются.

При использовании этого шаблона предполагается, что обрабатываемые сообщения были отправлены с помощью пакета [SDK для устройств центра Интернета вещей Azure](../../iot-hub/iot-hub-devguide-sdks.md#azure-iot-hub-device-sdks) или функции  [экспорта данных (устаревшей версии)](../../iot-central/core/howto-export-data-legacy.md) [Azure IOT Central](../../iot-central/core/overview-iot-central.md). При использовании этих пакетов SDK удостоверение устройства (при условии, что идентификатор устройства из центра Интернета вещей Azure или центрального сервера регистрируется в качестве идентификатора ресурса устройства на целевом сервере FHIR), а метка времени сообщения известна. Если вы используете пакеты SDK для устройств центра Интернета вещей Azure, но используете пользовательские свойства в тексте сообщения для удостоверения устройства или измерения времени, вы по-прежнему можете использовать Жсонпасконтенттемплате.

*Примечание. при использовании Иотжсонпасконтенттемплате Типематчекспрессион должен разрешаться во все сообщение как JToken. См. примеры ниже.* 
##### <a name="examples"></a>Примеры
---
**Частота сердца**

*Message*
```json
{
    "Body": {
        "heartRate": "78"        
    },
    "Properties": {
        "iothub-creation-time-utc" : "2019-02-01T22:46:01.8750000Z"
    },
    "SystemProperties": {
        "iothub-connection-device-id" : "device123"
    }
}
```
*Шаблон*
```json
{
    "templateType": "JsonPathContent",
    "template": {
        "typeName": "heartrate",
        "typeMatchExpression": "$..[?(@Body.heartRate)]",
        "deviceIdExpression": "$.deviceId",
        "timestampExpression": "$.endDate",
        "values": [
            {
                "required": "true",
                "valueExpression": "$.Body.heartRate",
                "valueName": "hr"
            }
        ]
    }
}
```
---
**Давление крови**

*Message*
```json
{
    "Body": {
        "systolic": "123",
        "diastolic" : "87"
    },
    "Properties": {
        "iothub-creation-time-utc" : "2019-02-01T22:46:01.8750000Z"
    },
    "SystemProperties": {
        "iothub-connection-device-id" : "device123"
    }
}
```
*Шаблон*
```json
{
    "typeName": "bloodpressure",
    "typeMatchExpression": "$..[?(@Body.systolic && @Body.diastolic)]",
    "values": [
        {
            "required": "true",
            "valueExpression": "$.Body.systolic",
            "valueName": "systolic"
        },
        {
            "required": "true",
            "valueExpression": "$.Body.diastolic",
            "valueName": "diastolic"
        }
    ]
}
```

#### <a name="iotcentraljsonpathcontenttemplate"></a>иотцентралжсонпасконтенттемплате

Иотцентралжсонпасконтенттемплате также не требует Девицеидекспрессион и Тиместампекспрессион и используется при отправке сообщений с помощью функции [экспорта данных](../../iot-central/core/howto-export-data.md) [IOT Central Azure](../../iot-central/core/overview-iot-central.md). При использовании этой функции удостоверение устройства (при условии, что идентификатор устройства из центра Интернета вещей Azure, регистрируется в качестве идентификатора ресурса устройства на целевом сервере FHIR), а метка времени сообщения известна. Если вы используете функцию экспорта данных IoT Central Azure, но используете пользовательские свойства в тексте сообщения для удостоверения устройства или измерения времени, вы по-прежнему можете использовать Жсонпасконтенттемплате.

*Примечание. при использовании Иотцентралжсонпасконтенттемплате Типематчекспрессион должен разрешаться во все сообщение как JToken. См. примеры ниже.* 
##### <a name="examples"></a>Примеры
---
**Частота сердца**

*Message*
```json
{
    "applicationId": "1dffa667-9bee-4f16-b243-25ad4151475e",
    "messageSource": "telemetry",
    "deviceId": "1vzb5ghlsg1",
    "schema": "default@v1",
    "templateId": "urn:qugj6vbw5:___qbj_27r",
    "enqueuedTime": "2020-08-05T22:26:55.455Z",
    "telemetry": {
        "HeartRate": "88",
    },
    "enrichments": {
      "userSpecifiedKey": "sampleValue"
    },
    "messageProperties": {
      "messageProp": "value"
    }
}
```
*Шаблон*
```json
{
    "templateType": "IotCentralJsonPathContent",
    "template": {
        "typeName": "heartrate",
        "typeMatchExpression": "$..[?(@telemetry.HeartRate)]",
        "values": [
            {
                "required": "true",
                "valueExpression": "$.telemetry.HeartRate",
                "valueName": "hr"
            }
        ]
    }
}
```
---
**Давление крови**

*Message*
```json
{
    "applicationId": "1dffa667-9bee-4f16-b243-25ad4151475e",
    "messageSource": "telemetry",
    "deviceId": "1vzb5ghlsg1",
    "schema": "default@v1",
    "templateId": "urn:qugj6vbw5:___qbj_27r",
    "enqueuedTime": "2020-08-05T22:26:55.455Z",
    "telemetry": {
        "BloodPressure": {
            "Diastolic": "87",
            "Systolic": "123"
        }
    },
    "enrichments": {
      "userSpecifiedKey": "sampleValue"
    },
    "messageProperties": {
      "messageProp": "value"
    }
}
```
*Шаблон*
```json
{
    "templateType": "IotCentralJsonPathContent",
    "template": {
        "typeName": "bloodPressure",
        "typeMatchExpression": "$..[?(@telemetry.BloodPressure.Diastolic && @telemetry.BloodPressure.Systolic)]",
        "values": [
            {
                "required": "true",
                "valueExpression": "$.telemetry.BloodPressure.Diastolic",
                "valueName": "bp_diastolic"
            },
            {
                "required": "true",
                "valueExpression": "$.telemetry.BloodPressure.Systolic",
                "valueName": "bp_systolic"
            }
        ]
    }
}
```

## <a name="fhir-mapping"></a>Сопоставление FHIR
После извлечения содержимого устройства в нормализованную модель данные собираются и группируются в соответствии с идентификатором устройства, типом измерения и периодом времени. Выходные данные этой группировки отправляются для преобразования в ресурс FHIR (сейчас[наблюдение](https://www.hl7.org/fhir/observation.html) ). Здесь шаблон сопоставления FHIR определяет, как данные сопоставляются с FHIRным наблюдением. Должно ли наблюдение создаваться на момент времени или за период в час? Какие коды следует добавить к наблюдению? Должно ли значение быть представлено как [сампледдата](https://www.hl7.org/fhir/datatypes.html#SampledData) или [Quantity](https://www.hl7.org/fhir/datatypes.html#Quantity)? Все эти типы данных являются параметрами элементов управления конфигурацией сопоставления FHIR.

### <a name="codevaluefhirtemplate"></a>кодевалуефхиртемплате
В настоящее время в сопоставлении FHIR поддерживается только шаблон Кодевалуефхиртемплате.  Он позволяет определять коды, действующие период и значение наблюдения. Поддерживаются несколько типов значений: [сампледдата](https://www.hl7.org/fhir/datatypes.html#SampledData), [кодеаблеконцепт](https://www.hl7.org/fhir/datatypes.html#CodeableConcept)и [Quantity](https://www.hl7.org/fhir/datatypes.html#Quantity). Вместе с этими настраиваемыми значениями идентификатор ресурса наблюдения и связь с соответствующим устройством и ресурсами пациента обрабатываются автоматически.

| Свойство | Описание 
| --- | ---
|**Имени**| Тип измерения, к которому должен быть привязан этот шаблон. Должен существовать по крайней мере один шаблон сопоставления устройств, который выводит этот тип.
|**периодинтервал**|Период времени, в течение которого должно быть представлено наблюдение. Поддерживаемые значения: 0 (экземпляр), 60 (час), 1440 (день).
|**Категория**|Любое количество [кодеаблеконцептс](http://hl7.org/fhir/datatypes-definitions.html#codeableconcept) для классификации типа создаваемого наблюдения.
|**Коды**|Одно или несколько [кодирований](http://hl7.org/fhir/datatypes-definitions.html#coding) , применяемых к созданному продолжению наблюдения.
|**Codes []. Приведен**|Код для [кода.](http://hl7.org/fhir/datatypes-definitions.html#coding)
|**Codes []. Системой**|Система для [написания кода](http://hl7.org/fhir/datatypes-definitions.html#coding).
|**Codes []. Монитор**|Отображение [кода](http://hl7.org/fhir/datatypes-definitions.html#coding).
|**Значение**|Значение, которое необходимо извлечь и представить в наблюдении. Дополнительные сведения см. в разделе [шаблоны типов значений](#valuetypes).
|**Components**|*Необязательно:* Один или несколько компонентов, которые необходимо создать при наблюдении.
|**Components []. Кода**|Одно [или несколько кодов,](http://hl7.org/fhir/datatypes-definitions.html#coding) применяемых к компоненту.
|**Components []. Значений**|Значение, которое необходимо извлечь и представить в компоненте. Дополнительные сведения см. в разделе [шаблоны типов значений](#valuetypes).

### <a name="value-type-templates"></a>Шаблоны типов значений <a name="valuetypes"></a>
Ниже приведены поддерживаемые в настоящее время шаблоны типов значений. В будущем можно добавить новые шаблоны.
#### <a name="sampleddata"></a>сампледдата
Представляет тип данных [сампледдата](http://hl7.org/fhir/datatypes.html#SampledData) FHIR. Измерения наблюдения записываются в поток значений начиная с точки во времени и увеличиваются вперед с помощью определенного периода. Если значение не указано, в `E` поток данных будет записано. Если точка состоит в том, что два дополнительных значения занимают одну и ту же точку в потоке данных, используется последнее значение. Та же логика применяется при обновлении наблюдения с использованием Сампледдата.

| Свойство | Описание 
| --- | ---
|**дефаултпериод**|Используемый по умолчанию период в миллисекундах. 
|**Единица измерения**|Единица, которая задается в источнике Сампледдата. 

#### <a name="quantity"></a>Количество
Представляет тип данных [Quantity](http://hl7.org/fhir/datatypes.html#Quantity) FHIR. Если в группировании имеется более одного значения, то используется только первое значение. При появлении нового значения, сопоставленного с тем же наблюдением, будет перезаписано старое значение.

| Свойство | Описание 
| --- | --- 
|**Единица измерения**| Базовое представление.
|**Код**| Закодированная форма единицы измерения.
|**Система**| Система, которая определяет форму кодированной единицы.

### <a name="codeableconcept"></a>кодеаблеконцепт
Представляет тип данных [кодеаблеконцепт](http://hl7.org/fhir/datatypes.html#CodeableConcept) FHIR. Фактическое значение не используется.

| Свойство | Описание 
| --- | --- 
|**Text**|Представление в виде обычного текста. 
|**Коды**|Одно или несколько [кодирований](http://hl7.org/fhir/datatypes-definitions.html#coding) , применяемых к созданному продолжению наблюдения.
|**Codes []. Приведен**|Код для [кода.](http://hl7.org/fhir/datatypes-definitions.html#coding)
|**Codes []. Системой**|Система для [написания кода](http://hl7.org/fhir/datatypes-definitions.html#coding).
|**Codes []. Монитор**|Отображение [кода](http://hl7.org/fhir/datatypes-definitions.html#coding).

### <a name="examples"></a>Примеры
**Частота сердца-Сампледдата**
```json
{
    "templateType": "CodeValueFhir",
    "template": {
        "codes": [
            {
                "code": "8867-4",
                "system": "http://loinc.org",
                "display": "Heart rate"
            }
        ],
        "periodInterval": 60,
        "typeName": "heartrate",
        "value": {
            "defaultPeriod": 5000,
            "unit": "count/min",
            "valueName": "hr",
            "valueType": "SampledData"
        }
    }
}
```
---
**Шаги — Сампледдата**
```json
{
    "templateType": "CodeValueFhir",
    "template": {
        "codes": [
            {
                "code": "55423-8",
                "system": "http://loinc.org",
                "display": "Number of steps"
            }
        ],        
        "periodInterval": 60,
        "typeName": "stepsCount",
        "value": {
            "defaultPeriod": 5000,
            "unit": "",
            "valueName": "steps",
            "valueType": "SampledData"
        }
    }
}
```
---
**Давление кровяного давления — Сампледдата**
```json
{
    "templateType": "CodeValueFhir",
    "template": {
        "codes": [
            {
                "code": "85354-9",
                "display": "Blood pressure panel with all children optional",
                "system": "http://loinc.org"
            }
        ],
        "periodInterval": 60,
        "typeName": "bloodpressure",
        "components": [
            {
                "codes": [
                    {
                        "code": "8867-4",
                        "display": "Diastolic blood pressure",
                        "system": "http://loinc.org"
                    }
                ],
                "value": {
                    "defaultPeriod": 5000,
                    "unit": "mmHg",
                    "valueName": "diastolic",
                    "valueType": "SampledData"
                }
            },
            {
                "codes": [
                    {
                        "code": "8480-6",
                        "display": "Systolic blood pressure",
                        "system": "http://loinc.org"
                    }
                ],
                "value": {
                    "defaultPeriod": 5000,
                    "unit": "mmHg",
                    "valueName": "systolic",
                    "valueType": "SampledData"
                }
            }
        ]
    }
}
```
---
**Давление кровяного давления — количество**
```json
{
    "templateType": "CodeValueFhir",
    "template": {
        "codes": [
            {
                "code": "85354-9",
                "display": "Blood pressure panel with all children optional",
                "system": "http://loinc.org"
            }
        ],
        "periodInterval": 0,
        "typeName": "bloodpressure",
        "components": [
            {
                "codes": [
                    {
                        "code": "8867-4",
                        "display": "Diastolic blood pressure",
                        "system": "http://loinc.org"
                    }
                ],
                "value": {
                    "unit": "mmHg",
                    "valueName": "diastolic",
                    "valueType": "Quantity"
                }
            },
            {
                "codes": [
                    {
                        "code": "8480-6",
                        "display": "Systolic blood pressure",
                        "system": "http://loinc.org"
                    }
                ],
                "value": {
                    "unit": "mmHg",
                    "valueName": "systolic",
                    "valueType": "Quantity"
                }
            }
        ]
    }
}
```
---
**Устройство удалено — Кодеаблеконцепт**
```json
{
    "templateType": "CodeValueFhir",
    "template": {
        "codes": [
            {
                "code": "deviceEvent",
                "system": "https://www.mydevice.com/v1",
                "display": "Device Event"
            }
        ],
        "periodInterval": 0,
        "typeName": "deviceRemoved",
        "value": {
            "text": "Device Removed",
            "codes": [
                {
                    "code": "deviceRemoved",
                    "system": "https://www.mydevice.com/v1",
                    "display": "Device Removed"
                }
            ],
            "valueName": "deviceRemoved",
            "valueType": "CodeableConcept"
        }
    }
}
```
---

## <a name="next-steps"></a>Дальнейшие действия

Ознакомьтесь с часто задаваемыми вопросами о соединителе Azure IoT для FHIR (Предварительная версия).

>[!div class="nextstepaction"]
>[Вопросы и ответы о соединителе Azure IoT для FHIR](fhir-faq.md)

*На портале Azure соединитель Azure IoT для FHIR называется соединителем IoT (предварительная версия). FHIR — это зарегистрированная торговая марка организации HL7, которая используется с разрешения HL7.