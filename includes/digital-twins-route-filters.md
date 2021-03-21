---
author: baanders
description: включить файл для параметров фильтра маршрутов Azure Digital двойников
ms.service: digital-twins
ms.topic: include
ms.date: 12/04/2020
ms.author: baanders
ms.openlocfilehash: d93f484e318c10489eb1db3e9c65c6e0c7479c90
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98860174"
---
| Имя фильтра | Описание | Фильтрация текста схемы | Поддерживаемые значения | 
| --- | --- | --- | --- |
| True/false | Позволяет создать маршрут без фильтрации или отключить маршрут, чтобы события не отправлялись | `<true/false>` | `true` = маршрут включен без фильтрации <br> `false` = маршрут отключен |
| Тип | [Тип события](../articles/digital-twins/concepts-route-events.md#types-of-event-messages) , переданного через ваш экземпляр Digital двойника | `type = '<eventType>'` | Возможные значения типа события: <br>`Microsoft.DigitalTwins.Twin.Create` <br> `Microsoft.DigitalTwins.Twin.Delete` <br> `Microsoft.DigitalTwins.Twin.Update`<br>`Microsoft.DigitalTwins.Relationship.Create`<br>`Microsoft.DigitalTwins.Relationship.Update`<br> `Microsoft.DigitalTwins.Relationship.Delete` <br> `microsoft.iot.telemetry`  |
| Источник | Имя экземпляра Digital двойников для Azure | `source = '<hostname>'`| Возможные значения имени узла: <br> **Для уведомлений**: `<yourDigitalTwinInstance>.api.<yourRegion>.digitaltwins.azure.net` <br> **Для телеметрии**: `<yourDigitalTwinInstance>.api.<yourRegion>.digitaltwins.azure.net/<twinId>`|
| Субъект | Описание события в контексте источника событий выше | `subject = '<subject>'` | Возможные значения субъекта: <br>**Для уведомлений**: Тема `<twinid>` <br> или формат URI для субъектов, которые уникально идентифицируются несколькими частями или идентификаторами:<br>`<twinid>/relationships/<relationshipid>`<br> **Для телеметрии**: субъект — это путь к компоненту (если данные телеметрии создаются из компонента двойника), например `comp1.comp2` . Если данные телеметрии не выдаются из компонента, то его поле subject будет пустым. |
| Схема данных | Идентификатор модели ДТДЛ | `dataschema = '<model-dtmi-ID>'` | **Для телеметрии**: Схема данных — это идентификатор модели двойника или компонента, который создает данные телеметрии. Например: `dtmi:example:com:floor4;2` <br>**Для уведомлений (создание или удаление)**: доступ к схеме данных можно получить в тексте уведомления по адресу `$body.$metadata.$model` . <br>**Для уведомлений (обновление)**: доступ к схеме данных можно получить в тексте уведомления по адресу `$body.modelId`|
| Тип содержимого | Тип содержимого значения данных | `datacontenttype = '<contentType>'` | Тип содержимого — `application/json` |
| Версия спецификации | Используемая версия схемы событий | `specversion = '<version>'` | Версия должна быть `1.0` . Это означает, что схема Клаудевентс версии 1,0 |
| Текст уведомления | Ссылка на любое свойство в `data` поле уведомления | `$body.<property>` | Примеры уведомлений см. [*в разделе практические*](../articles/digital-twins/how-to-interpret-event-data.md) сведения. На любое свойство в `data` поле можно ссылаться с помощью `$body`

Следующие типы данных поддерживаются в качестве значений, возвращаемых ссылками на приведенные выше данные:

| Тип данных | Пример |
|-|-|-|
|**String**| `STARTS_WITH($body.$metadata.$model, 'dtmi:example:com:floor')` <br> `CONTAINS(subject, '<twinID>')`|
|**Integer**|`$body.errorCode > 200`|
|**Double**|`$body.temperature <= 5.5`|
|**Логическом**|`$body.poweredOn = true`|
|**Null**|`$body.prop != null`|

При определении фильтров маршрутов поддерживаются следующие операторы:

|Семейство|Операторы|Пример|
|-|-|-|
|Логические|AND, OR, ()|`(type != 'microsoft.iot.telemetry' OR datacontenttype = 'application/json') OR (specversion != '1.0')`|
|Сравнение|<, <=, >, >=, =,! =|`$body.temperature <= 5.5`

При определении фильтров маршрутов поддерживаются следующие функции:

|Компонент|Описание|Пример|
|--|--|--|
|STARTS_WITH (x, y)|Возвращает значение true, если значение `x` начинается со строки `y` .|`STARTS_WITH($body.$metadata.$model, 'dtmi:example:com:floor')`|
|ENDS_WITH (x, y) | Возвращает значение true, если значение `x` заканчивается строкой `y` .|`ENDS_WITH($body.$metadata.$model, 'floor;1')`|
|CONTAINS(x,y)| Возвращает значение true, если значение `x` содержит строку `y` .|`CONTAINS(subject, '<twinID>')`|

При реализации или обновлении фильтра может потребоваться несколько минут для отражения в конвейере данных.