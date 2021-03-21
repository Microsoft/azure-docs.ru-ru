---
title: REST API конфигурации приложений Azure — метки
description: Справочные страницы по работе с метками с помощью конфигурации приложения Azure REST API
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.topic: reference
ms.date: 08/17/2020
ms.openlocfilehash: 5a59f5910d44f2a2b4cd75e7a1d51c2ed5dd51a4
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96932512"
---
# <a name="labels"></a>Метки

версия API: 1,0

Ресурс **Label** определяется следующим образом:

```json
{
      "name": [string]             // Name of the label
}
```

Поддерживает следующие операции:

- Список

Для всех операций ``name`` — необязательный параметр фильтра. Если он не указан, подразумевается **любая** метка.

## <a name="prerequisites"></a>Предварительные условия

- Все HTTP-запросы должны пройти проверку подлинности. См. раздел [Проверка подлинности](./rest-api-authentication-index.md) .
- Все HTTP-запросы должны предоставлять явный запрос `api-version` . См. раздел [Управление версиями](./rest-api-versioning.md) .

## <a name="list-labels"></a>Список меток

```http
GET /labels?api-version={api-version} HTTP/1.1
```

**Правляют**

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.microsoft.appconfig.labelset+json; charset=utf-8"
```

```json
{
    "items": [
        {
          "name": "{label-name}"
        },
        ...
    ],
    "@nextLink": "{relative uri}"
}
```

## <a name="pagination"></a>Разбиение на страницы

Если число возвращаемых элементов превышает предельное значение, то разбивка на страницы отводится. Следуйте дополнительным `Link` заголовкам ответа и используйте `rel="next"` для навигации. Кроме того, содержимое предоставляет следующую ссылку в форме `@nextLink` Свойства. Следующая ссылка содержит `api-version` параметр.

```http
GET /labels?api-version={api-version} HTTP/1.1
```

**Ответ**

```http
HTTP/1.1 OK
Content-Type: application/vnd.microsoft.appconfig.labelset+json; charset=utf-8
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

## <a name="filtering"></a>Фильтрация

Поддерживается фильтрация по `name` .

```http
GET /labels?name={label-name}&api-version={api-version}
```

### <a name="supported-filters"></a>Поддерживаемые фильтры

|Фильтр ключей|Действие|
|--|--|
|`name` не указан или имеет значение `name=*`|Соответствует **любой** метке|
|`name=abc`|Соответствует метке с именем  **ABC**|
|`name=abc*`|Соответствует именам меток, начинающимся с **ABC**|
|`name=abc,xyz`|Соответствует именам меток **ABC** или **XYZ** (не более 5 CSV)|

### <a name="reserved-characters"></a>Зарезервированные символы

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
  "title": "Invalid request parameter 'name'",
  "name": "name",
  "detail": "name(2): Invalid character",
  "status": 400
}
```

### <a name="examples"></a>Примеры

- Все

    ```http
    GET /labels?api-version={api-version}
    ```

- Имя метки начинается с **ABC**

    ```http
    GET  /labels?name=abc*&api-version={api-version}
    ```

- Имя метки — **ABC** или **XYZ**

    ```http
    GET /labels?name=abc,xyz&api-version={api-version}
    ```

## <a name="request-specific-fields"></a>Запрашивать определенные поля

Используйте необязательный `$select` параметр строки запроса и укажите список запрошенных полей с разделителями-запятыми. Если `$select` параметр пропущен, ответ содержит набор по умолчанию.

```http
GET /labels?$select=name&api-version={api-version} HTTP/1.1
```

## <a name="time-based-access"></a>Доступ к Time-Based

Получить представление результата, которое было в прошлый раз. См. [раздел об](https://tools.ietf.org/html/rfc7089#section-2.1)

```http
GET /labels&api-version={api-version} HTTP/1.1
Accept-Datetime: Sat, 12 May 2018 02:10:00 GMT
```

**Ответ**

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.microsoft.appconfig.labelset+json"
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
