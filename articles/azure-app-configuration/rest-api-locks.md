---
title: REST API конфигурации приложений Azure — блокировки
description: Справочные страницы по работе с блокировками "ключ-значение" с помощью конфигурации приложения Azure REST API
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.topic: reference
ms.date: 08/17/2020
ms.openlocfilehash: e99ce5595bae8ed64285317d9249da60e0fc1b83
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96932529"
---
# <a name="locks"></a>Блокировки

Этот API (версия 1,0) предоставляет семантику блокировки и разблокировки для ресурса «ключ-значение». Она поддерживает следующие операции:

- Блокировка на месте
- Снять блокировку

Если параметр указан, `label` должен быть явным значением метки (не является подстановочным). Для всех операций это необязательный параметр. Если этот параметр опущен, метка не подразумевается.

## <a name="prerequisites"></a>Предварительные условия

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-rest-api-prereqs.md)]

## <a name="lock-key-value"></a>Заблокировать ключ — значение

- Обязательный: ``{key}`` , ``{api-version}``  
- Необязательно. ``label``

```http
PUT /locks/{key}?label={label}&api-version={api-version} HTTP/1.1
```

**Правляют**

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.microsoft.appconfig.kv+json; charset=utf-8"
```

```json
{
  "etag": "4f6dd610dd5e4deebc7fbaef685fb903",
  "key": "{key}",
  "label": "{label}",
  "content_type": null,
  "value": "example value",
  "created": "2017-12-05T02:41:26.4874615+00:00",
  "locked": true,
  "tags": []
}
```

Если ключ-значение не существует, возвращается следующий ответ:

```http
HTTP/1.1 404 Not Found
```

## <a name="unlock-key-value"></a>Разблокировать ключ — значение

- Обязательный: ``{key}`` , ``{api-version}``  
- Необязательно. ``label``

```http
DELETE /locks/{key}?label={label}?api-version={api-version} HTTP/1.1
```

**Правляют**

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.microsoft.appconfig.kv+json; charset=utf-8"
```

```json
{
  "etag": "4f6dd610dd5e4deebc7fbaef685fb903",
  "key": "{key}",
  "label": "{label}",
  "content_type": null,
  "value": "example value",
  "created": "2017-12-05T02:41:26.4874615+00:00",
  "locked": true,
  "tags": []
}
```

Если ключ-значение не существует, возвращается следующий ответ:

```http
HTTP/1.1 404 Not Found
```

## <a name="conditional-lock-and-unlock"></a>Условная блокировка и разблокировка

Чтобы избежать конкуренции, используйте `If-Match` или `If-None-Match` заголовков запроса. `etag`Аргумент является частью представления ключа. Если `If-Match` `If-None-Match` аргумент или опущен, операция является безусловной.

Следующий запрос применяет операцию только в том случае, если текущее представление "ключ-значение" соответствует указанному `etag` :

```http
PUT|DELETE /locks/{key}?label={label}&api-version={api-version} HTTP/1.1
If-Match: "4f6dd610dd5e4deebc7fbaef685fb903"
```

Следующий запрос применяет операцию только в том случае, если текущее представление "ключ-значение" существует, но не соответствует указанному `etag` :

```http
PUT|DELETE /kv/{key}?label={label}&api-version={api-version} HTTP/1.1
If-None-Match: "4f6dd610dd5e4deebc7fbaef685fb903"
```
