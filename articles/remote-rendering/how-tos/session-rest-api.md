---
title: REST API управления сеансами
description: Описывает управление сеансами
author: florianborn71
ms.author: flborn
ms.date: 02/11/2020
ms.topic: article
ms.openlocfilehash: d957c5d6521010c7393e2297be16cd7bef41c35f
ms.sourcegitcommit: a4533b9d3d4cd6bb6faf92dd91c2c3e1f98ab86a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/22/2020
ms.locfileid: "97724074"
---
# <a name="use-the-session-management-rest-api"></a>Использование REST API управления сеансами

Чтобы использовать функции удаленной подготовки к просмотру Azure, необходимо создать *сеанс*. Каждый сеанс соответствует виртуальной машине, выделенной в Azure, и ожидает подключения клиентского устройства. При подключении устройства виртуальная машина отображает запрошенные данные и передает результат в виде видеопотока. Во время создания сеанса выбирается тип сервера, на котором будет выполняться, который определяет цены. Когда сеанс больше не нужен, он должен быть остановлен. Если этот параметр не был остановлен вручную, он будет автоматически закрыт по истечении срока *аренды* сеанса.

Мы предоставляем скрипт PowerShell в [репозитории образцов arr](https://github.com/Azure/azure-remote-rendering) в папке *Scripts* с именем *RenderingSession.ps1*, который демонстрирует использование нашей службы. Скрипт и его конфигурация описаны здесь: [примеры сценариев PowerShell](../samples/powershell-example-scripts.md)

> [!TIP]
> Команды PowerShell, перечисленные на этой странице, предназначены для дополнения друг к другу. Если все скрипты выполняются последовательно в одной командной строке PowerShell, они будут построены поверх других.

## <a name="regions"></a>Регионы

См. [список доступных регионов](../reference/regions.md) для базовых URL-адресов, на которые отправляются запросы.

Для примеров сценариев ниже мы выбрали регион *westus2*.

### <a name="example-script-choose-an-endpoint"></a>Пример скрипта. Выбор конечной точки

```PowerShell
$endPoint = "https://remoterendering.westus2.mixedreality.azure.com"
```

## <a name="accounts"></a>Счета

Если у вас нет учетной записи удаленной подготовки, [создайте ее](create-an-account.md). Каждый ресурс идентифицируется путем *accountId*, который используется во всех интерфейсах API сеанса.

### <a name="example-script-set-accountid-accountkey-and-account-domain"></a>Пример скрипта: Настройка accountId, accountKey и домена учетной записи

Домен учетной записи — это расположение учетной записи удаленной подготовки к просмотру. В этом примере используется расположение учетной записи Region *eastus*.

```PowerShell
$accountId = "********-****-****-****-************"
$accountKey = "*******************************************="
$accountDomain = "eastus.mixedreality.azure.com"
```

## <a name="common-request-headers"></a>Общие заголовки запросов

* Заголовок *авторизации* должен иметь значение " `Bearer TOKEN` ", где " `TOKEN` " — токен проверки подлинности, [возвращаемый службой маркеров безопасности](tokens.md).

### <a name="example-script-request-a-token"></a>Пример скрипта: запрос маркера

```PowerShell
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.SecurityProtocolType]::Tls12
$webResponse = Invoke-WebRequest -Uri "https://sts.$accountDomain/accounts/$accountId/token" -Method Get -ContentType "application/json" -Headers @{ Authorization = "Bearer ${accountId}:$accountKey" }
$response = ConvertFrom-Json -InputObject $webResponse.Content
$token = $response.AccessToken;
```

## <a name="common-response-headers"></a>Общие заголовки ответов

* Команда разработчиков может использовать заголовок *MS-КП* для трассировки вызова в рамках службы.

## <a name="create-a-session"></a>Создание сеанса

Эта команда создает сеанс. Он возвращает идентификатор нового сеанса. Для всех остальных команд требуется идентификатор сеанса.

| URI | Метод |
|-----------|:-----------|
| /v1/Accounts/*accountId*/Сессионс/креате | POST |

**Текст запроса:**

* Макслеасетиме (TimeSpan) — значение времени ожидания, когда сеанс будет списан автоматически
* модели (массив): URL-адреса контейнеров ресурсов для предварительной загрузки
* Размер (строка): Размер сервера для настройки ([**"Стандартный"**](../reference/vm-sizes.md) или [**"Премиум"**](../reference/vm-sizes.md)). См. раздел определенные [ограничения размера](../reference/limits.md#overall-number-of-polygons).

**Правляют**

| Код состояния | полезные данные JSON | Комментарии |
|-----------|:-----------|:-----------|
| 202 | -sessionId: GUID | Success |

### <a name="example-script-create-a-session"></a>Пример сценария. Создание сеанса

```PowerShell
Invoke-WebRequest -Uri "$endPoint/v1/accounts/$accountId/sessions/create" -Method Post -ContentType "application/json" -Body "{ 'maxLeaseTime': '4:0:0', 'models': [], 'size': 'standard' }" -Headers @{ Authorization = "Bearer $token" }
```

Выходные данные примера:

```PowerShell
StatusCode        : 202
StatusDescription : Accepted
Content           : {"sessionId":"d31bddca-dab7-498e-9bc9-7594bc12862f"}
RawContent        : HTTP/1.1 202 Accepted
                    MS-CV: 5EqPJ1VdTUakDJZc6/ZhTg.0
                    Content-Length: 52
                    Content-Type: application/json; charset=utf-8
                    Date: Thu, 09 May 2019 16:17:50 GMT
                    Location: accounts/11111111-1111-1111-11...
Forms             : {}
Headers           : {[MS-CV, 5EqPJ1VdTUakDJZc6/ZhTg.0], [Content-Length, 52], [Content-Type, application/json;
                    charset=utf-8], [Date, Thu, 09 May 2019 16:17:50 GMT]...}
Images            : {}
InputFields       : {}
Links             : {}
ParsedHtml        : mshtml.HTMLDocumentClass
RawContentLength  : 52
```

### <a name="example-script-store-sessionid"></a>Пример скрипта: хранилище sessionId

Ответ из приведенного выше запроса включает **идентификатор сеанса**, который необходим для всех дальнейших запросов.

```PowerShell
$sessionId = "d31bddca-dab7-498e-9bc9-7594bc12862f"
```

## <a name="modify-and-query-session-properties"></a>Изменение и запрос свойств сеанса

Существует несколько команд для запроса или изменения параметров существующих сеансов.

> [!CAUTION]
> Как и для всех вызовов RESTFUL, отправка этих команд слишком часто приведет к тому, что сервер будет регулировать и возвращать ошибку в конечном итоге. Код состояния в данном случае — 429 ("слишком много запросов"). Поэтому очень важно, чтобы между последовательными вызовами была задержка в **5–10 секунд**.

### <a name="update-session-parameters"></a>Обновить параметры сеанса

Эта команда обновляет параметры сеанса. В настоящее время можно продлить время аренды сеанса.

> [!IMPORTANT]
> Время аренды всегда указывается как общее время с момента начала сеанса. Это означает, что если вы создали сеанс со временем аренды, равным одному часу, и хотите продлить время аренды на другой час, необходимо обновить его Макслеасетиме до двух часов.

| URI | Метод |
|-----------|:-----------|
| /v1/Accounts/*accountID*/Сессионс/*SessionID* | PATCH |

**Текст запроса:**

* Макслеасетиме (TimeSpan) — значение времени ожидания, когда сеанс будет списан автоматически

**Правляют**

| Код состояния | полезные данные JSON | Комментарии |
|-----------|:-----------|:-----------|
| 200 | | Успешно |

#### <a name="example-script-update-a-session"></a>Пример скрипта. обновление сеанса

```PowerShell
Invoke-WebRequest -Uri "$endPoint/v1/accounts/$accountId/sessions/$sessionId" -Method Patch -ContentType "application/json" -Body "{ 'maxLeaseTime': '5:0:0' }" -Headers @{ Authorization = "Bearer $token" }
```

Выходные данные примера:

```PowerShell
StatusCode        : 200
StatusDescription : OK
Content           : {}
RawContent        : HTTP/1.1 200 OK
                    MS-CV: Fe+yXCJumky82wuoedzDTA.0
                    Content-Length: 0
                    Date: Thu, 09 May 2019 16:27:31 GMT


Headers           : {[MS-CV, Fe+yXCJumky82wuoedzDTA.0], [Content-Length, 0], [Date, Thu, 09 May 2019 16:27:31 GMT]}
RawContentLength  : 0
```

### <a name="get-active-sessions"></a>Получение активных сеансов

Эта команда возвращает список активных сеансов.

| URI | Метод |
|-----------|:-----------|
| /v1/Accounts/*accountId*/Сессионс | GET |

**Правляют**

| Код состояния | полезные данные JSON | Комментарии |
|-----------|:-----------|:-----------|
| 200 | -Sessions: массив свойств сеанса | Описание свойств сеанса см. в разделе "получение свойств сеанса". |

#### <a name="example-script-query-active-sessions"></a>Пример скрипта: запрос активных сеансов

```PowerShell
Invoke-WebRequest -Uri "$endPoint/v1/accounts/$accountId/sessions" -Method Get -Headers @{ Authorization = "Bearer $token" }
```

Выходные данные примера:

```PowerShell
StatusCode        : 200
StatusDescription : OK
Content           : []
RawContent        : HTTP/1.1 200 OK
                    MS-CV: WfB9Cs5YeE6S28qYkp7Bhw.1
                    Content-Length: 15
                    Content-Type: application/json; charset=utf-8
                    Date: Thu, 25 Jul 2019 16:23:50 GMT

                    {"sessions":[]}
Forms             : {}
Headers           : {[MS-CV, WfB9Cs5YeE6S28qYkp7Bhw.1], [Content-Length, 2], [Content-Type, application/json;
                    charset=utf-8], [Date, Thu, 25 Jul 2019 16:23:50 GMT]}
Images            : {}
InputFields       : {}
Links             : {}
ParsedHtml        : mshtml.HTMLDocumentClass
RawContentLength  : 2
```

### <a name="get-sessions-properties"></a>Получение свойств сеансов

Эта команда возвращает сведения о сеансе, например имя узла виртуальной машины.

| URI | Метод |
|-----------|:-----------|
| /v1/Accounts/*accountId*/Сессионс/*SessionID*/пропертиес | GET |

**Правляют**

| Код состояния | полезные данные JSON | Комментарии |
|-----------|:-----------|:-----------|
| 200 | -Message: строка<br/>-Сессионелапседтиме: TimeSpan<br/>-Сессионхостнаме: строка<br/>-sessionId: строка<br/>-Сессионмакслеасетиме: TimeSpan<br/>-Сессионсизе: Enum<br/>-Сессионстатус: Enum | Enum Сессионстатус {Start, Ready, остановка, остановлена, срок действия истек, ошибка}<br/>Если состояние — "ошибка" или "срок действия истек", сообщение будет содержать дополнительные сведения |

#### <a name="example-script-get-session-properties"></a>Пример скрипта. получение свойств сеанса

```PowerShell
Invoke-WebRequest -Uri "$endPoint/v1/accounts/$accountId/sessions/$sessionId/properties" -Method Get -Headers @{ Authorization = "Bearer $token" }
```

Выходные данные примера:

```PowerShell
StatusCode        : 200
StatusDescription : OK
Content           : {"message":null,"sessionElapsedTime":"00:00:01","sessionHostname":"5018fee8-817e-4366-9179-556af79a4240.remoterenderingvm.westeurope.mixedreality.azure.com","sessionId":"e13d2c44-63e0-4591-991e-f9e05e599a93","sessionMaxLeaseTime":"04:00:00","sessionStatus":"Ready"}
RawContent        : HTTP/1.1 200 OK
                    MS-CV: CMXegpZRMECH4pbOW2j5GA.0
                    Content-Length: 60
                    Content-Type: application/json; charset=utf-8
                    Date: Thu, 09 May 2019 16:30:38 GMT

                    {"message":null,...
Forms             : {}
Headers           : {[MS-CV, CMXegpZRMECH4pbOW2j5GA.0], [Content-Length, 60], [Content-Type, application/json;
                    charset=utf-8], [Date, Thu, 09 May 2019 16:30:38 GMT]}
Images            : {}
InputFields       : {}
Links             : {}
ParsedHtml        : mshtml.HTMLDocumentClass
RawContentLength  : 60
```

## <a name="stop-a-session"></a>Остановка сеанса

Эта команда останавливает сеанс. Выделенная виртуальная машина будет освобождена вскоре после.

| URI | Метод |
|-----------|:-----------|
| /v1/Accounts/*accountId*/Сессионс/*SessionID* | DELETE |

**Правляют**

| Код состояния | полезные данные JSON | Комментарии |
|-----------|:-----------|:-----------|
| 204 | | Success |

### <a name="example-script-stop-a-session"></a>Пример скрипта. Завершение сеанса

```PowerShell
Invoke-WebRequest -Uri "$endPoint/v1/accounts/$accountId/sessions/$sessionId" -Method Delete -Headers @{ Authorization = "Bearer $token" }
```

Выходные данные примера:

```PowerShell
StatusCode        : 204
StatusDescription : No Content
Content           : {}
RawContent        : HTTP/1.1 204 No Content
                    MS-CV: YDxR5/7+K0KstH54WG443w.0
                    Date: Thu, 09 May 2019 16:45:41 GMT


Headers           : {[MS-CV, YDxR5/7+K0KstH54WG443w.0], [Date, Thu, 09 May 2019 16:45:41 GMT]}
RawContentLength  : 0
```

## <a name="next-steps"></a>Дальнейшие действия

* [Примеры скриптов PowerShell](../samples/powershell-example-scripts.md)
