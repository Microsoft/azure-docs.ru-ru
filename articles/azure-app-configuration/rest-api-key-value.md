---
title: Конфигурация приложения Azure REST API — значение ключа
description: Справочные страницы по работе с ключевыми значениями с помощью конфигурации приложений Azure REST API
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.topic: reference
ms.date: 08/17/2020
ms.openlocfilehash: 95081d6f8c2770d01f7836e08b6851860bf47ba8
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96932563"
---
# <a name="key-values"></a>Значения ключей

Значение ключа — это ресурс, определяемый уникальным сочетанием `key`  +  `label` . Аргумент `label` является необязательным. Чтобы явно ссылаться на ключевое значение без метки, используйте "\ 0" (URL-адрес в кодировке ``%00`` ). См. подробные сведения для каждой операции.

Эта статья относится к API версии 1,0.

## <a name="operations"></a>Операции

- Получить
- Список нескольких
- Присвойте параметру
- Удалить

## <a name="prerequisites"></a>Предварительные условия

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-rest-api-prereqs.md)]

## <a name="syntax"></a>Синтаксис

```json
{
  "etag": [string],
  "key": [string],
  "label": [string, optional],
  "content_type": [string, optional],
  "value": [string],
  "last_modified": [datetime ISO 8601],
  "locked": [boolean],
  "tags": [object with string properties, optional]
}
```

## <a name="get-key-value"></a>Получение значения ключа

Обязательный: ``{key}`` , ``{api-version}``  
Необязательно. ``label`` (если не указано, подразумевается ключевое значение без метки.)

```http
GET /kv/{key}?label={label}&api-version={api-version}
```

**Правляют**

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.microsoft.appconfig.kv+json; charset=utf-8;
Last-Modified: Tue, 05 Dec 2017 02:41:26 GMT
ETag: "4f6dd610dd5e4deebc7fbaef685fb903"
```

```json
{
  "etag": "4f6dd610dd5e4deebc7fbaef685fb903",
  "key": "{key}",
  "label": "{label}",
  "content_type": null,
  "value": "example value",
  "last_modified": "2017-12-05T02:41:26+00:00",
  "locked": "false",
  "tags": {
    "t1": "value1",
    "t2": "value2"
  }
}
```

Если ключ не существует, возвращается следующий ответ:

```http
HTTP/1.1 404 Not Found
```

## <a name="get-conditionally"></a>Get (условно)

Чтобы улучшить кэширование клиента, используйте `If-Match` или `If-None-Match` запросите заголовки. `etag`Аргумент является частью представления ключа. Дополнительные сведения см. в [разделах 14,24 и 14,26](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html).

Следующий запрос получает значение ключа, только если текущее представление не соответствует указанному `etag` :

```http
GET /kv/{key}?api-version={api-version} HTTP/1.1
Accept: application/vnd.microsoft.appconfig.kv+json;
If-None-Match: "{etag}"
```

**Правляют**

```http
HTTP/1.1 304 NotModified
```

или диспетчер конфигурации служб

```http
HTTP/1.1 200 OK
```

## <a name="list-key-values"></a>Список значений ключа

Необязательно: ``key`` (если не указано, подразумевается любой ключ.) Необязательно: ``label`` (если не указано, подразумевается любая метка.)

```http
GET /kv?label=*&api-version={api-version} HTTP/1.1
```

**Ответ**

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.microsoft.appconfig.kvset+json; charset=utf-8
```

Дополнительные параметры см. в подразделе «Фильтрация» далее в этой статье.

## <a name="pagination"></a>Разбиение на страницы

Если число возвращаемых элементов превышает предельное значение, то разбивка на страницы отводится. Следуйте дополнительным `Link` заголовкам ответа и используйте `rel="next"` для навигации.
Кроме того, содержимое предоставляет следующую ссылку в форме `@nextLink` Свойства. Связанный URI включает `api-version` аргумент.

```http
GET /kv?api-version={api-version} HTTP/1.1
```

**Ответ**

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.microsoft.appconfig.kvs+json; charset=utf-8
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

`key`Поддерживается сочетание фильтров и `label` .
Используйте необязательные `key` `label` Параметры и строки запроса.

```http
GET /kv?key={key}&label={label}&api-version={api-version}
```

### <a name="supported-filters"></a>Поддерживаемые фильтры

|Фильтр ключей|Действие|
|--|--|
|`key` не указан или имеет значение `key=*`|Соответствует **любому** ключу|
|`key=abc`|Соответствует ключу с именем **ABC**|
|`key=abc*`|Соответствует именам ключей, начинающимся с **ABC**|
|`key=abc,xyz`|Соответствует именам ключей **ABC** или **XYZ** (не более 5 CSV)|

|Фильтр меток|Действие|
|--|--|
|`label` не указан или имеет значение `label=*`|Соответствует **любой** метке|
|`label=%00`|Соответствует KV без метки|
|`label=prod`|Соответствует метке " **произ** .|
|`label=prod*`|Совпадает с метками, начинающимися с **произв** .|
|`label=prod,test`|Совпадает с **метками** "Рабочая или **Тестовая** " (ограничение в 5 CSV)|

***Зарезервированные символы***

`*`, `\`, `,`

Если зарезервированный символ является частью значения, он должен быть экранирован с помощью `\{Reserved Character}` . Незарезервированные символы также могут быть экранированы.

***Проверка фильтра***

В случае ошибки проверки фильтра ответом является HTTP `400` со сведениями об ошибке:

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

**Примеры**

- Все

    ```http
    GET /kv?api-version={api-version}
    ```

- Имя ключа начинается с **ABC** и включает все метки

    ```http
    GET /kv?key=abc*&label=*&api-version={api-version}
    ```

- Имя ключа начинается с **ABC** , а метка равна **v1** или **v2**

    ```http
    GET /kv?key=abc*&label=v1,v2&api-version={api-version}
    ```

## <a name="request-specific-fields"></a>Запрашивать определенные поля

Используйте необязательный `$select` параметр строки запроса и укажите список запрошенных полей с разделителями-запятыми. Если `$select` параметр пропущен, ответ содержит набор по умолчанию.

```http
GET /kv?$select=key,value&api-version={api-version} HTTP/1.1
```

## <a name="time-based-access"></a>Доступ на основе времени

Получить представление результата, которое было в прошлый раз. Дополнительные сведения [см. в](https://tools.ietf.org/html/rfc7089#section-2.1)разделе г. Разбивка на страницы по-прежнему поддерживается, как определено ранее в этой статье.

```http
GET /kv?api-version={api-version} HTTP/1.1
Accept-Datetime: Sat, 12 May 2018 02:10:00 GMT
```

**Ответ**

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.microsoft.appconfig.kvset+json"
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

## <a name="set-key"></a>Задать ключ

- Обязательно: ``{key}``
- Необязательно: ``label`` (если не указано или Label = %00, это подразумевает ключевое значение без метки.)

```http
PUT /kv/{key}?label={label}&api-version={api-version} HTTP/1.1
Content-Type: application/vnd.microsoft.appconfig.kv+json
```

```json
{
  "value": "example value",         // optional
  "content_type": "user defined",   // optional
  "tags": {                         // optional
    "tag1": "value1",
    "tag2": "value2",
  }
}
```

**Правляют**

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.microsoft.appconfig.kv+json; charset=utf-8
Last-Modified: Tue, 05 Dec 2017 02:41:26 GMT
ETag: "4f6dd610dd5e4deebc7fbaef685fb903"
```

```json
{
  "etag": "4f6dd610dd5e4deebc7fbaef685fb903",
  "key": "{key}",
  "label": "{label}",
  "content_type": "user defined",
  "value": "example value",
  "last_modified": "2017-12-05T02:41:26.4874615+00:00",
  "tags": {
    "tag1": "value1",
    "tag2": "value2",
  }
}
```

Если элемент заблокирован, вы получите следующий ответ:

```http
HTTP/1.1 409 Conflict
Content-Type: application/problem+json; charset="utf-8"
```

```json
{
    "type": "https://azconfig.io/errors/key-locked",
    "title": "Modifing key '{key}' is not allowed",
    "name": "{key}",
    "detail": "The key is read-only. To allow modification unlock it first.",
    "status": "409"
}
```

## <a name="set-key-conditionally"></a>Задать ключ (условно)

Чтобы избежать конкуренции, используйте `If-Match` или `If-None-Match` заголовков запроса. `etag`Аргумент является частью представления ключа.
Если `If-Match` `If-None-Match` аргумент или опущен, операция является безусловной.

Следующий ответ обновляет значение только в том случае, если текущее представление соответствует указанному `etag` :

```http
PUT /kv/{key}?label={label}&api-version={api-version} HTTP/1.1
Content-Type: application/vnd.microsoft.appconfig.kv+json
If-Match: "4f6dd610dd5e4deebc7fbaef685fb903"
```

Следующий ответ обновляет значение только в том случае, если текущее представление не соответствует указанному `etag` :

```http
PUT /kv/{key}?label={label}&api-version={api-version} HTTP/1.1
Content-Type: application/vnd.microsoft.appconfig.kv+json;
If-None-Match: "4f6dd610dd5e4deebc7fbaef685fb903"
```

Следующий запрос добавляет значение только в том случае, если представление уже существует:

```http
PUT /kv/{key}?label={label}&api-version={api-version} HTTP/1.1
Content-Type: application/vnd.microsoft.appconfig.kv+json;
If-Match: "*"
```

Следующий запрос добавляет значение только в том случае, если представление еще не существует:

```http
PUT /kv/{key}?label={label}&api-version={api-version} HTTP/1.1
Content-Type: application/vnd.microsoft.appconfig.kv+json
If-None-Match: "*"
```

**Ответы**

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.microsoft.appconfig.kv+json; charset=utf-8
...
```

или диспетчер конфигурации служб

```http
HTTP/1.1 412 PreconditionFailed
```

## <a name="delete"></a>Удалить

- Обязательный: `{key}` , `{api-version}`
- Необязательно: `{label}` (если не указано или Label = %00, это подразумевает ключевое значение без метки.)

```http
DELETE /kv/{key}?label={label}&api-version={api-version} HTTP/1.1
```

**Ответ:** Возвращает удаленное значение ключа или None, если ключ-значение не существует.

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.microsoft.appconfig.kv+json; charset=utf-8
...
```

или диспетчер конфигурации служб

```http
HTTP/1.1 204 No Content
```

## <a name="delete-key-conditionally"></a>Удалить ключ (условно)

Это похоже на раздел «Set Key (условно)» выше в этой статье.
