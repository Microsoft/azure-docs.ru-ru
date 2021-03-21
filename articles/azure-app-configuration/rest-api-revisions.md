---
title: REST API конфигурации приложений Azure — "ключ — значение"
description: Справочные страницы по работе с редакциями "ключ — значение" с помощью конфигурации приложения Azure REST API
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.topic: reference
ms.date: 08/17/2020
ms.openlocfilehash: cfa117d1ed017170c279b7c4e0a146ae4edac108
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96932478"
---
# <a name="key-value-revisions"></a>Редакции "ключ — значение"

*Редакция "ключ-значение* " определяет историческое представление ресурса "ключ-значение". Срок действия исправлений истекает через 7 дней для хранилищ бесплатных уровней или 30 дней для хранилищ уровня "Стандартный". Исправления поддерживают `List` операцию.

Для всех операций ``key`` — необязательный параметр. Если этот параметр опущен, он подразумевает любой ключ.

Для всех операций ``label`` — необязательный параметр. Если этот параметр опущен, он подразумевает любую метку.

Эта статья относится к API версии 1,0.

## <a name="prerequisites"></a>Предварительные условия

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-rest-api-prereqs.md)]

## <a name="list-revisions"></a>Вывод списка редакций

```http
GET /revisions?label=*&api-version={api-version} HTTP/1.1
```

**Правляют**

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.microsoft.appconfig.kvset+json; charset=utf-8"
Accept-Ranges: items
```

```json
{
    "items": [
        {
          "etag": "4f6dd610dd5e4deebc7fbaef685fb903",
          "key": "{key}",
          "label": "{label}",
          "content_type": null,
          "value": "example value",
          "last_modified": "2017-12-05T02:41:26.4874615+00:00",
          "tags": []
        },
        ...
    ],
    "@nextLink": "{relative uri}"
}
```

## <a name="pagination"></a>Разбиение на страницы

Если число возвращаемых элементов превышает предельное значение, то разбивка на страницы отводится. Следуйте указаниям в необязательном ``Link`` заголовке ответа и используйте ``rel="next"`` для навигации. Кроме того, содержимое предоставляет следующую ссылку в виде ``@nextLink`` Свойства.

```http
GET /revisions?api-version={api-version} HTTP/1.1
```

**Ответ**

```http
HTTP/1.1 OK
Content-Type: application/vnd.microsoft.appconfig.kvs+json; charset=utf-8
Accept-Ranges: items
Link: <{relative uri}>; rel="next"
```

```json
{
    "items": [
        ...
    ],
    "@nextLink": "{relative uri}"
}
```

## <a name="list-subset-of-revisions"></a>Перечисление подмножества редакций

Используйте `Range` заголовок запроса. Ответ содержит заголовок `Content-Range`. Если сервер не может удовлетворить запрошенный диапазон, он реагирует на HTTP `416` ( `RangeNotSatisfiable` ).

```http
GET /revisions?api-version={api-version} HTTP/1.1
Range: items=0-2
```

**Ответ**

```http
HTTP/1.1 206 Partial Content
Content-Type: application/vnd.microsoft.appconfig.revs+json; charset=utf-8
Content-Range: items 0-2/80
```

## <a name="filtering"></a>Фильтрация

`key`Поддерживается сочетание фильтров и `label` .
Используйте необязательные `key` `label` Параметры и строки запроса.

```http
GET /revisions?key={key}&label={label}&api-version={api-version}
```

### <a name="supported-filters"></a>Поддерживаемые фильтры

|Фильтр ключей|Действие|
|--|--|
|`key` не указан или имеет значение `key=*`|Соответствует **любому** ключу|
|`key=abc`|Соответствует ключу с именем  **ABC**|
|`key=abc*`|Соответствует именам ключей, начинающимся с **ABC**|
|`key=*abc`|Соответствует именам ключей, заканчивающимся на **ABC**|
|`key=*abc*`|Совпадает с именами ключей, содержащими **ABC**|
|`key=abc,xyz`|Соответствует именам ключей **ABC** или **XYZ** (не более 5 CSV)|

|Фильтр меток|Действие|
|--|--|
|`label` не указан или имеет значение `label=`|Совпадает с записью без метки|
|`label=*`|Соответствует **любой** метке|
|`label=prod`|Соответствует метке " **произ** .|
|`label=prod*`|Совпадает с метками, начинающимися с **произв** .|
|`label=*prod`|Соответствует меткам, заканчивающимся на " **произв** ."|
|`label=*prod*`|Соответствует меткам, содержащим **произ** .|
|`label=prod,test`|Совпадает с **метками** "Рабочая или **Тестовая** " (ограничение в 5 CSV)|

### <a name="reserved-characters"></a>Зарезервированные символы

Зарезервированные символы:

`*`, `\`, `,`

Если зарезервированный символ является частью значения, он должен быть экранирован с помощью `\{Reserved Character}` . Незарезервированные символы также могут быть экранированы.

### <a name="filter-validation"></a>Проверка фильтра

Если возникает ошибка проверки фильтра, ответом является HTTP `400` со сведениями об ошибке:

```http
HTTP/1.1 400 Bad Request
Content-Type: application/problem+json; charset=utf-8
```

```json
{
  "type": "https://azconfig.io/errors/invalid-argument",
  "title": "Invalid request parameter '{filter}'",
  "name": "{filter}",
  "detail": "{filter}(2): Invalid character",
  "status": 400
}
```

### <a name="examples"></a>Примеры

- Все.

    ```http
    GET /revisions
    ```

- Элементы, в которых имя ключа начинается с **ABC**:

    ```http
    GET /revisions?key=abc*&api-version={api-version}
    ```

- Элементы, где имя ключа — **ABC** или **XYZ**, а метки содержат « **произ**.».

    ```http
    GET /revisions?key=abc,xyz&label=*prod*&api-version={api-version}
    ```

## <a name="request-specific-fields"></a>Запрашивать определенные поля

Используйте необязательный `$select` параметр строки запроса и укажите список запрошенных полей с разделителями-запятыми. Если `$select` параметр пропущен, ответ содержит набор по умолчанию.

```http
GET /revisions?$select=value,label,last_modified&api-version={api-version} HTTP/1.1
```

## <a name="time-based-access"></a>Доступ на основе времени

Получить представление результата, которое было в прошлый раз. Дополнительные сведения см. в разделе " [платформа HTTP" для Time-Based доступ к состояниям ресурсов — напоминание](https://tools.ietf.org/html/rfc7089#section-2.1), раздел "г.".

```http
GET /revisions?api-version={api-version} HTTP/1.1
Accept-Datetime: Sat, 12 May 2018 02:10:00 GMT
```

**Ответ**

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.microsoft.appconfig.revs+json"
Memento-Datetime: Sat, 12 May 2018 02:10:00 GMT
Link: <{relative uri}>; rel="original"
```

```json
{
    "items": [
        ....
    ]
}
```
