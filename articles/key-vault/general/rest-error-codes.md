---
title: Коды ошибок REST API — Azure Key Vault
description: Эти коды ошибок могут возвращаться операцией в Azure Key Vault веб-службе.
keywords: ''
services: machine-learning
author: msmbaldwin
ms.custom: seodec18
ms.author: mbaldwin
ms.service: key-vault
ms.subservice: general
ms.topic: reference
ms.date: 12/16/2019
ms.openlocfilehash: 30b7e34f2a791cfd8dec1a6d8e81d706fa07939f
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91631228"
---
# <a name="azure-key-vault-rest-api-error-codes"></a>Коды ошибок Azure Key Vault REST API
 
Следующие коды ошибок могут возвращаться операцией для веб-службы Azure Key Vault.
 
## <a name="http-401-unauthenticated-request"></a>HTTP 401: запрос без проверки подлинности

401 означает, что запрос не прошел проверку подлинности для Key Vault. 

Проверка подлинности запроса выполняется в следующих случаях:

- Хранилище ключей знает идентификатор вызывающего объекта; перетаскивани
- Вызывающая сторона может попытаться получить доступ к ресурсам Key Vault. 

Существует несколько различных причин, по которым запрос может вернуть 401.

### <a name="no-authentication-token-attached-to-the-request"></a>Нет маркера проверки подлинности, присоединенного к запросу. 

Ниже приведен пример запроса на выстановку с указанием значения секрета:

``` 
PUT https://putreqexample.vault.azure.net//secrets/DatabaseRotatingPassword?api-version=7.0 HTTP/1.1
x-ms-client-request-id: 03d275a2-52a4-4bed-82c8-6fe15165affb
accept-language: en-US
Authorization: Bearer     eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Im5iQ3dXMTF3M1hrQi14VWFYd0tSU0xqTUhHUSIsImtpZCI6Im5iQ3dXMTF3M1hrQi14VWFYd0tSU0xqTUhHUSJ9.eyJhdWQiOiJodHRwczovL3ZhdWx0LmF6dXJlLm5ldCIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzcyZjk4OGJmLTg2ZjEtNDFhZi05MWFiLTJkN2NkMDExZGI0Ny8iLCJpYXQiOjE1NDg2OTc1MTMsIm5iZiI6MTU0ODY5NzUxMywiZXhwIjoxNTQ4NzAxNDEzLCJhaW8iOiI0MkpnWUhoODVqaVBnZHF5ZlRGZE5TdHY3bGUvQkFBPSIsImFwcGlkIjoiZmFkN2Q1YjMtNjlkNi00YjQ4LTkyNTktOGQxMjEyNGUxY2YxIiwiYXBwaWRhY3IiOiIxIiwiaWRwIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvNzJmOTg4YmYtODZmMS00MWFmLTkxYWItMmQ3Y2QwMTFkYjQ3LyIsIm9pZCI6IjM5NzVhZWVkLTdkMDgtNDUzYi1iNmY0LTQ0NWYzMjY5ODA5MSIsInN1YiI6IjM5NzVhZWVkLTdkMDgtNDUzYi1iNmY0LTQ0NWYzMjY5ODA5MSIsInRpZCI6IjcyZjk4OGJmLTg2ZjEtNDFhZi05MWFiLTJkN2NkMDExZGI0NyIsInV0aSI6IjItZ3JoUmtlSWs2QmVZLUxuNDJtQUEiLCJ2ZXIiOiIxLjAifQ.fgubiz1MKqTJTXI8dHIV7t9Fle6FdHrkaGYKcBeVRX1WtLVuk1QVxzIFDlZKLXJ7QPNs0KWpeiWQI9IWIRK-8wO38yCqKTfDlfHOiNWGOpkKddlG729KFqakVf2w0GPyGPFCONRDAR5wjQarN9Bt8I8YbHwZQz_M1hztlnv-Lmsk1jBmech9ujD9-lTMBmSfFFbHcqquev119V7sneI-zxBZLf8C0pIDkaXf1t8y6Xr8CUJDMdlWLslCf3pBCNIOy65_TyGvy4Z4AJryTPBarNBPwOkNAtjCfZ4BDc2KqUZM5QN_VK4foP64sVzUL6mSr0Gh7lQJIL5b1qIpJxjxyQ
User-Agent: FxVersion/4.7.3324.0 OSName/Windows OSVersion/6.2.9200.0 Microsoft.Azure.KeyVault.KeyVaultClient/3.0.3.0
Content-Type: application/json; charset=utf-8
Host: putreqexample.vault.azure.net
Content-Length: 31

{
   "value": "m*gBJ7$Zuoz)"
}
```

Заголовок Authorization — это маркер доступа, необходимый для каждого вызова Key Vault для операций на плоскости данных. Если заголовок отсутствует, ответ должен быть 401.

### <a name="the-token-lacks-the-correct-resource-associated-with-it"></a>Маркер не имеет нужного ресурса, связанного с ним. 

При запросе маркера доступа из конечной точки Azure OAUTH параметр с именем Resource является обязательным. Значение важно для поставщика токена, так как он ограничивает область токена для его предполагаемого использования. Ресурс для **всех** токенов для доступа к Key Vault — *https: \/ /Vault.keyvault.NET* (без замыкающей косой черты).

### <a name="the-token-is-expired"></a>Срок действия маркера истек

Токены кодируются в кодировке Base64, и значения могут быть декодированы на веб-сайтах, например [http://jwt.calebb.net](http://jwt.calebb.net) . Ниже приведена декодированная лексема:

```
    {
 typ: "JWT",
 alg: "RS256",
 x5t: "nbCwW11w3XkB-xUaXwKRSLjMHGQ",
 kid: "nbCwW11w3XkB-xUaXwKRSLjMHGQ"
}.
{
 aud: "https://vault.azure.net",
 iss: "https://sts.windows.net/72f988bf-86f1-41af-91ab-2d7cd011db47/",
 iat: 1548697513,
 nbf: 1548697513,
 exp: 1548701413,
 aio: "42JgYHh85jiPgdqyfTFdNStv7le/BAA=",
 appid: "fad7d5b3-69d6-4b48-9259-8d12124e1cf1",
 appidacr: "1",
 idp: "https://sts.windows.net/72f988bf-86f1-41af-91ab-2d7cd011db47/",
 oid: "3975aeed-7d08-453b-b6f4-445f32698091",
 sub: "3975aeed-7d08-453b-b6f4-445f32698091",
 tid: "72f988bf-86f1-41af-91ab-2d7cd011db47",
 uti: "2-grhRkeIk6BeY-Ln42mAA",
 ver: "1.0"
}.
[signature]
```

В этом маркере можно увидеть много важных частей:

- AUD (аудитория) — ресурс токена. Обратите внимание, что это `https://vault.azure.net` . Этот токен не будет работать ни для одного ресурса, который явно не соответствует этому значению, например Graph.
- IAT (выдана в): число тактов с начала эпохи, когда был выдан маркер.
- NBF (не ранее): число тактов с начала эпохи, когда этот маркер станет действительным.
- EXP (срок действия). число тактов с начала эпохи после истечения срока действия маркера.
- AppID (идентификатор приложения): идентификатор GUID для идентификатора приложения, выполняющего этот запрос.
- TID (идентификатор клиента): идентификатор GUID для идентификатора клиента участника, выполняющего этот запрос.

Важно, чтобы все значения правильно определялись в токене, чтобы запрос работал. Если все правильно, запрос не будет приводить к 401.

### <a name="troubleshooting-401"></a>Устранение неполадок 401

перед запросом к хранилищу ключей следует исследовать 401S с момента создания маркера. Обычно код используется для запроса маркера. После получения маркера он передается в запрос Key Vault. Если код выполняется локально, можно использовать Fiddler для записи запроса или ответа в `https://login.microsoftonline.com` . Запрос выглядит следующим образом:

``` 
POST https://login.microsoftonline.com/<key vault tenant ID>/oauth2/token HTTP/1.1
Accept: application/json
Content-Type: application/x-www-form-urlencoded; charset=utf-8
Host: login.microsoftonline.com
Content-Length: 192

resource=https%3A%2F%2Fvault.azure.net&client_id=<registered-app-ID>&client_secret=<registered-app-secret>&client_info=1&grant_type=client_credentials
```

Указанные пользователем сведения должны быть правильными:

- Идентификатор клиента хранилища ключей
- Значение ресурса, установленное в HTTPS %3, %2 F %2 F Vault. Azure. NET (в кодировке URL)
- Идентификатор клиента
- Секрет клиента

Убедитесь, что остальная часть запроса практически идентична.

Если вы можете получить маркер доступа отклика, его можно декодировать (как показано выше), чтобы убедиться, что идентификатор клиента, идентификатор приложения и ресурс.

## <a name="http-403-insufficient-permissions"></a>HTTP 403: недостаточно разрешений

HTTP 403 означает, что запрос прошел проверку подлинности (он знает о запросе удостоверения), но удостоверение не имеет разрешения на доступ к запрошенному ресурсу. Существует две причины.

- Политика доступа для удостоверения отсутствует.
- IP-адрес запрашивающего ресурса не утверждается в параметрах брандмауэра в хранилище ключей.

HTTP 403 часто возникает, когда клиентское приложение не использует идентификатор клиента, который его считает. Это обычно означает, что политики доступа неправильно настроены для фактического вызова удостоверения.

### <a name="troubleshooting-403"></a>Устранение неполадок 403

Сначала включите ведение журнала. Инструкции о том, как это сделать, см. в разделе [ведение журнала Azure Key Vault](logging.md).

После включения ведения журнала можно определить, вызвана ли 403 политиками доступа или брандмауэра.

#### <a name="error-due-to-firewall-policy"></a>Ошибка из-за политики брандмауэра

"Адрес клиента (00.00.00.00) не является полномочным и вызывающий объект не является доверенной службой"

Существует ограниченный список "доверенных служб Azure". Веб-сайты Azure **не** являются доверенными службами Azure. Дополнительные сведения см. в записи блога [Key Vault доступ к брандмауэру с помощью служб приложений Azure](https://azidentity.azurewebsites.net/post/2019/01/03/key-vault-firewall-access-by-azure-app-services).

Чтобы он работал, необходимо добавить в Key Vault IP-адрес сайта Azure.

Если из-за политики доступа: найдите идентификатор объекта для запроса и убедитесь, что идентификатор объекта соответствует объекту, которому пользователь пытается назначить политику доступа. В AAD часто встречается несколько объектов, имеющих одно и то же имя, поэтому очень важно выбрать правильный вариант. После удаления и повторного добавления политики доступа можно увидеть, существует ли несколько объектов с одним и тем же именем.

Кроме того, большинство политик доступа не требует использования "полномочного приложения", как показано на портале. Разрешенное приложение используется для сценариев проверки подлинности "от имени", которые являются редкими. 


## <a name="http-429-too-many-requests"></a>HTTP 429: слишком много запросов

Регулирование происходит, когда количество запросов превышает заданное максимальное значение для интервала времени. Если происходит регулирование, ответ Key Vault будет иметь значение HTTP 429. Для типов запросов указано максимальное значение. Например, создание ключа HSM 2048-bit составляет 5 запросов в течение 10 секунд, но все остальные транзакции HSM 1000 имеют ограничение Request/10 секунд. Поэтому важно понимать, какие типы вызовов выполняются при определении причины регулирования.
В общем случае запросы к Key Vault ограничены 2000 запросами/10 секунд. Исключения являются ключевыми операциями, как описано в разделе [ограничения службы Key Vault](service-limits.md)

### <a name="troubleshooting-429"></a>Устранение неполадок 429
Регулирование обходится с помощью этих методов:

- Сократите количество запросов к Key Vault, определив шаблоны для запрошенного ресурса и попытайтесь кэшировать их в вызывающем приложении. 

- При выполнении регулирования Key Vault адаптировать запрашивающий код для использования экспоненциальной задержки при повторной попытке. Алгоритм описан здесь: [регулирование приложения](overview-throttling.md#how-to-throttle-your-app-in-response-to-service-limits)

- Если количество запросов не может быть сокращено с помощью кэширования, а задержка задержки не работает, рассмотрите возможность разделения ключей на несколько хранилищ ключей. Ограничение службы для одной подписки — 5x ограничения индивидуального Key Vault. При использовании более 5 хранилищ ключей следует учитывать необходимость использования нескольких подписок. 

Подробное руководство, в том числе запрос на увеличение ограничений, можно найти здесь: [Key Vault руководство по регулированию](overview-throttling.md)
