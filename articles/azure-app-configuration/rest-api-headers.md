---
title: REST API конфигурации приложений Azure — заголовки
description: Справочные страницы для заголовков, используемых в конфигурации приложений Azure REST API
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.topic: reference
ms.date: 08/17/2020
ms.openlocfilehash: 80b20b0b55219766872166685c0b1257f3c39c55
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96932580"
---
# <a name="headers"></a>Заголовки

В этой статье приведены ссылки на страницы справочника по заголовкам, используемым в REST API конфигурации приложений Azure.

## <a name="request-headers"></a>Заголовки запросов

В следующей таблице описаны общие заголовки запросов, используемые в конфигурации приложений Azure.

| Header | Описание | Пример |
|--|--|--|
| **Авторизация** | Используется для [проверки подлинности](./rest-api-authentication-index.md) запроса к службе. См. [раздел 14,8](https://tools.ietf.org/html/rfc2616#section-14.8) | `Authorization: HMAC-SHA256 Credential=<Credential>&SignedHeaders=Host;x-ms-date;x-ms-content-sha256&Signature=<Signature>` |
| **Принятие** | Информирует сервер о том, какой тип носителя будет принимать клиент в ответе HTTP. См. [раздел 14,1](https://tools.ietf.org/html/rfc2616#section-14.1) | `Accept: application/vnd.microsoft.appconfig.kv+json;` |
| **Accept-DateTime** | Запрашивает сервер вернуть его содержимое в виде представления его прежнего состояния. Значение этого заголовка является запрошенным значением DateTime этого состояния. См. [RFC 7089](https://tools.ietf.org/html/rfc7089#section-2.1.1) | `Accept-Datetime: Sat, 12 May 2018 02:10:00 GMT` |
| **Content-Type** | Содержит тип носителя содержимого в тексте HTTP-запроса. См. [раздел 14,17](https://tools.ietf.org/html/rfc2616#section-14.17) | `Content-Type: application/vnd.microsoft.appconfig.kv+json; charset=utf-8;` |
| **Дата** | Дата и время выдачи HTTP-запроса. Этот заголовок используется в [проверке подлинности HMAC](./rest-api-authentication-hmac.md). См. [раздел 14,18](https://tools.ietf.org/html/rfc2616#section-14.18) | `Date: Fri, 11 May 2018 18:48:36 GMT` |
| **Узел** | Указывает клиент, для которого был выдан запрос. Этот заголовок используется в [проверке подлинности HMAC](./rest-api-authentication-hmac.md). См. [раздел 14,23](https://tools.ietf.org/html/rfc2616#section-14.23) | `Host: contoso.azconfig.io` |
| **If-Match** | Используется для выполнения условного запроса HTTP. Этот запрос должен быть выполнен, только если ETag целевого ресурса совпадает со значением этого заголовка. Значение "*" соответствует любому ETag. См. [раздел 14,24](https://tools.ietf.org/html/rfc2616#section-14.24) | `If-Match: "4f6dd610dd5e4deebc7fbaef685fb903"` |
| **If-None-Match** | Используется для выполнения условного запроса HTTP. Этот запрос должен быть выполнен, только если ETag целевого ресурса не соответствует значению этого заголовка. Значение "*" соответствует любому ETag. См. [раздел 14,26](https://tools.ietf.org/html/rfc2616#section-14.26) | `If-None-Match: "4f6dd610dd5e4deebc7fbaef685fb903"` |
| **Sync — Token** | Используется для обеспечения согласованности в режиме реального времени во время последовательности запросов. | `Sync-Token: jtqGc1I4=MDoyOA==;sn=28` |
| **x-ms-client-request-id** | Уникальный идентификатор, предоставленный клиентом для наблюдения за циклическим обменом запроса. | `x-ms-client-request-id: 00000000-0000-0000-0000-000000000000` |
| **x-ms-content-sha256** | Дайджест SHA256 текста HTTP-запроса. Этот заголовок используется в [проверке подлинности HMAC](./rest-api-authentication-hmac.md). | `x-ms-content-sha256: 47DEQpj8HBSa+/TImW+5JCeuQeRkm5NMpJWZG3hSuFU=` |
| **x-ms-date** | Этот заголовок может быть установлен и использован вместо `Date` заголовка, если не удается получить доступ к заголовку даты. Этот заголовок используется в [проверке подлинности HMAC](./rest-api-authentication-hmac.md). | `x-ms-date: Fri, 11 May 2018 18:48:36 GMT` |
| **x-ms-return-client-request-id** | Используется в сочетании с `x-ms-client-request-id` заголовком. Если значение этого заголовка равно true, серверу будет предложено вернуть значение `x-ms-client-request-id` заголовка запроса. | `x-ms-return-client-request-id: true` |

## <a name="response-headers"></a>Заголовки откликов

Сервер может включать в ответы следующие заголовки HTTP.

| Header | Описание | Пример |
|--|--|--|
| **Content-Type** | Содержит тип носителя содержимого в теле HTTP-ответа. См. [раздел 14,17](https://tools.ietf.org/html/rfc2616#section-14.17) | `Content-Type: application/vnd.microsoft.appconfig.kv+json; charset=utf-8;` |
| **Тегом** | Непрозрачный токен, представляющий состояние данного ресурса. Может использоваться в условных операциях. См. [раздел 14,19](https://tools.ietf.org/html/rfc2616#section-14.19) | `ETag: "4f6dd610dd5e4deebc7fbaef685fb903"` |
| **Последний измененный** | Описывает, когда запрошенный ресурс был изменен последним. В формате [http-Date](https://tools.ietf.org/html/rfc2616#section-3.3.1). См. [раздел 14,29](https://tools.ietf.org/html/rfc2616#section-14.29) | `Last-Modified: Tue, 05 Dec 2017 02:41:26 GMT` |
| **Ссылка** | Содержит ссылки на ресурсы, связанные с ответом. Этот заголовок используется для разбиения на страницы с помощью _следующей_ ссылки. См. [RFC 5988](https://tools.ietf.org/html/rfc5988) | `Link: </kv?after={token}>; rel="next"` |
| **Напоминание-DateTime** | Указывает, что содержимое, содержащееся в ответе, представляет прежнее состояние. Значением этого заголовка является дата и время этого состояния. См. [RFC 7089](https://tools.ietf.org/html/rfc7089#section-2.1.1) | `Memento-Datetime: Sat, 12 May 2018 02:10:00 GMT` |
| **Retry-After-MS** | Предоставляет предполагаемый период (в миллисекундах) ожидания клиентом перед повторным выполнением невыполненного запроса. | `retry-after-ms: 10` |
| **x-ms-request-id** | Уникальный идентификатор, созданный сервером, который используется для отслеживания запроса в службе. | `x-ms-request-id: 00000000-0000-0000-0000-000000000000` |
| **WWW-Authenticate** | Используется для запроса клиентов на проверку подлинности и предоставления причины сбоя попытки проверки подлинности. См. [раздел 14,47](https://tools.ietf.org/html/rfc2616#section-14.47) | `WWW-Authenticate: HMAC-SHA256 error="invalid_token" error_description="Invalid Signature"` |
