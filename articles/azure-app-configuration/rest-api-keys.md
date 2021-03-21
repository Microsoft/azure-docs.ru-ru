---
title: REST API конфигурации приложений Azure — ключи
description: Справочные страницы по работе с ключами с помощью конфигурации приложений Azure REST API
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.topic: reference
ms.date: 08/17/2020
ms.openlocfilehash: 65ca190d7fbd6d8d4df473fbe2112eafbd031fde
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96932546"
---
# <a name="keys"></a>Ключи

версия API: 1,0

Следующий синтаксис представляет ключевой ресурс:

```http
{
    "name": [string]             // Name of the key
}
```

## <a name="operations"></a>Операции

Основные ресурсы поддерживают следующую операцию:

- Список

Для всех операций `name` — необязательный параметр фильтра. Если этот параметр опущен, он подразумевает *любой* ключ.

## <a name="prerequisites"></a>Предварительные условия

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-rest-api-prereqs.md)]

## <a name="list-keys"></a>Отображает ключи.

```http
GET /keys?api-version={api-version} HTTP/1.1
```

**Правляют**

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.microsoft.appconfig.keyset+json; charset=utf-8"
```

```json
{
    "items": [
        {
          "name": "{key-name}"
        },
        ...
    ],
    "@nextLink": "{relative uri}"
}
```

## <a name="pagination"></a>Разбиение на страницы

Если число возвращаемых элементов превышает предельное значение, то разбивка на страницы отводится. Следуйте дополнительным `Link` заголовкам ответа и используйте `rel="next"` для навигации. Кроме того, содержимое предоставляет следующую ссылку в виде `@nextLink` Свойства. Следующая ссылка содержит `api-version` параметр.

```http
GET /keys?api-version={api-version} HTTP/1.1
```

**Ответ**

```http
HTTP/1.1 OK
Content-Type: application/vnd.microsoft.appconfig.keyset+json; charset=utf-8
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

Поддерживается фильтрация по ```name``` .

```http
GET /keys?name={key-name}&api-version={api-version}
```

Поддерживаются следующие фильтры:

|Фильтр ключей|Действие|
|--|--|
|`name` не указан или имеет значение `name=*`|Соответствует **любому** ключу|
|`name=abc`|Соответствует ключу с именем  **ABC**|
|`name=abc*`|Совпадает с ключом, имя которого начинается с **abc**.|
|`name=abc,xyz`|Соответствует именам ключей **ABC** или **XYZ** (не более 5 CSV)|

Зарезервированы следующие символы: `*` , `\` , `,`

Если зарезервированный символ является частью значения, он должен быть экранирован с помощью `\{Reserved Character}` . Незарезервированные символы также могут быть экранированы.

## <a name="filter-validation"></a>Проверка фильтра

В случае ошибки проверки фильтра ответом является HTTP `400` со сведениями об ошибке:

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

## <a name="examples"></a>Примеры

- Все

    ```http
    GET /keys?api-version={api-version}
    ```

- Имя ключа начинается с **ABC**

    ```http
    GET  /keys?name=abc*&api-version={api-version}
    ```

- Имя ключа: **ABC** или **XYZ**

    ```http
    GET /keys?name=abc,xyz&api-version={api-version}
    ```

## <a name="request-specific-fields"></a>Запрашивать определенные поля

Используйте необязательный `$select` параметр строки запроса и укажите список запрошенных полей, разделенных запятыми. Если `$select` параметр пропущен, ответ содержит набор по умолчанию.

```http
GET /keys?$select=name&api-version={api-version} HTTP/1.1
```

## <a name="time-based-access"></a>Доступ к Time-Based

Получить представление результата, которое было в прошлый раз. См. [раздел об](https://tools.ietf.org/html/rfc7089#section-2.1)

```http
GET /keys&api-version={api-version} HTTP/1.1
Accept-Datetime: Sat, 12 May 2018 02:10:00 GMT
```

**Ответ**

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.microsoft.appconfig.keyset+json"
Memento-Datetime: Sat, 12 May 2018 02:10:00 GMT
Link: <relative uri>; rel="original"
```

```json
{
    "items": [
        ....
    ]
}
```
