---
title: Получение маркеров доступа к службе
description: Описание создания маркеров для доступа к интерфейсам API служб ARR.
author: florianborn71
ms.author: flborn
ms.date: 02/11/2020
ms.topic: how-to
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 9721685fc3ccd2c1c80b55e9118d6d347cc97a9c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97830706"
---
# <a name="get-service-access-tokens"></a>Получение маркеров доступа к службе

Доступ к API-интерфейсам служб ARR () предоставляется только для полномочных пользователей. Чтобы доказать свою авторизацию, необходимо отправить *маркер доступа* вместе с запросами на другие запросы. Эти маркеры выдаются *службой маркеров безопасности* (STS) в Exchange для ключа учетной записи. Токены имеют **время существования 24 часов** и поэтому могут выдаваться пользователям без предоставления им полного доступа к службе.

В этой статье описывается, как создать такой маркер доступа.

## <a name="prerequisites"></a>Предварительные условия

[Создайте учетную запись arr](create-an-account.md), если она еще не создана.

## <a name="token-service-rest-api"></a>REST API службы маркеров

Для создания маркеров доступа *Служба маркеров безопасности* предоставляет один REST API. URL-адрес службы STS зависит от домена учетной записи удаленной подготовки к просмотру. Он имеет вид https://sts . [ домен учетной записи], например `https://sts.southcentralus.mixedreality.azure.com`

### <a name="get-token-request"></a>Запрос "получение токена"

| URI | Метод |
|-----------|:-----------|
| /аккаунтс/**accountId**/Token | GET |

| Заголовок | Значение |
|--------|:------|
| Авторизация | "Bearer, **accountId**:**accountKey**" |

Замените *accountId* и *accountKey* соответствующими данными.

### <a name="get-token-response"></a>Ответ "получить токен"

| Код состояния | полезные данные JSON | Комментарии |
|-----------|:-----------|:-----------|
| 200 | AccessToken: строка | Успешно |

| Header | Назначение |
|--------|:------|
| MS-КП | Это значение можно использовать для трассировки вызова в службе |

## <a name="getting-a-token-using-powershell"></a>Получение маркера с помощью PowerShell

В следующем примере кода PowerShell показано, как отправить в STS необходимый запрос на ОСТАВШУЮся работу. Затем он выводит маркер в командную строку PowerShell.

```PowerShell
$accountId = "<account_id_from_portal>"
$accountKey = "<account_key_from_portal>"
$accountDomain = "<account_domain_from_portal>

[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.SecurityProtocolType]::Tls12;
$webResponse = Invoke-WebRequest -Uri "https://sts.$accountDomain/accounts/$accountId/token" -Method Get -Headers @{ Authorization = "Bearer ${accountId}:$accountKey" }
$response = ConvertFrom-Json -InputObject $webResponse.Content

Write-Output "Token: $($response.AccessToken)"
```

Сценарий просто выводит маркер в выход, откуда его можно скопировать & вставить. Для реальных проектов этот процесс следует автоматизировать.

## <a name="next-steps"></a>Дальнейшие действия

* [Примеры скриптов PowerShell](../samples/powershell-example-scripts.md)
* [Интерфейсные API Azure](../how-tos/frontend-apis.md)
* [REST API управления сеансами](../how-tos/session-rest-api.md)
