---
title: Действие веб-перехватчика в фабрике данных Azure
description: Действие веб-перехватчика не позволяет продолжить выполнение конвейера до тех пор, пока не будет выполнена проверка присоединенного набора данных с определенными критериями, указанными пользователем.
author: dcstwh
ms.author: weetok
ms.reviewer: jburchel
ms.service: data-factory
ms.topic: conceptual
ms.date: 03/25/2019
ms.openlocfilehash: 4c3ff5d7139f4167769f78aa858c7d7a693539a3
ms.sourcegitcommit: f611b3f57027a21f7b229edf8a5b4f4c75f76331
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104785943"
---
# <a name="webhook-activity-in-azure-data-factory"></a>Действие веб-перехватчика в фабрике данных Azure

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Действие веб-перехватчика может управлять выполнением конвейеров с помощью пользовательского кода. С помощью действия веб-перехватчика код клиента может вызвать конечную точку и передать ему URL-адрес обратного вызова. Выполнение конвейера ожидает вызова обратного вызова перед переходом к следующему действию.

> [!IMPORTANT]
> Действие веб-перехватчика теперь позволяет отображать состояние ошибки и пользовательские сообщения обратно в действие и конвейер. Задайте для _репортстатусонкаллбакк_ значение true и включите _StatusCode_ и _ошибку_ в полезные данные обратного вызова. Дополнительные сведения см. в разделе [Дополнительные примечания](#additional-notes) .

## <a name="syntax"></a>Синтаксис

```json

{
    "name": "MyWebHookActivity",
    "type": "WebHook",
    "typeProperties": {
        "method": "POST",
        "url": "<URLEndpoint>",
        "headers": {
            "Content-Type": "application/json"
        },
        "body": {
            "key": "value"
        },
        "timeout": "00:03:00",
        "reportStatusOnCallBack": false,
        "authentication": {
            "type": "ClientCertificate",
            "pfx": "****",
            "password": "****"
        }
    }
}

```

## <a name="type-properties"></a>Свойства типа

Свойство | Описание | Допустимые значения | Обязательно
-------- | ----------- | -------------- | --------
**name** | Имя действия веб-перехватчика. | Строка | Да |
**type** | Для этого свойства необходимо задать значение "веб-перехватчик". | Строка | Да |
**Method** | Метод REST API для целевой конечной точки. | Строка. Поддерживаемый тип — POST. | Да |
**url** | Целевая конечная точка и путь. | Строка или выражение со значением **resultType** строки. | Да |
**Верхний** | Заголовки, которые отправляются в запрос. Ниже приведен пример, который задает язык и тип запроса: `"headers" : { "Accept-Language": "en-us", "Content-Type": "application/json" }` . | Строка или выражение со значением **resultType** строки. | Да. `Content-Type`Требуется заголовок, например `"headers":{ "Content-Type":"application/json"}` . |
**body** | Представляет полезные данные, отправляемые конечной точке. | Допустимый формат JSON или выражение со значением **resultType** в формате JSON. Схему полезных данных запроса см. в статье [схема полезных данных запроса](./control-flow-web-activity.md#request-payload-schema) . | Да |
**идентификаци** | Метод проверки подлинности, используемый для вызова конечной точки. Поддерживаются следующие типы: "базовый" и "ClientCertificate". Дополнительные сведения см. в разделе [Authenticate to the Speech API](./control-flow-web-activity.md#authentication) (Аутентификация в API речи). Если проверка подлинности не требуется, исключите это свойство. | Строка или выражение со значением **resultType** строки. | Нет |
**timeout** | Время ожидания действием обратного вызова, указанного параметром **каллбаккури** для вызова. Значение по умолчанию — 10 минут ("00:10:00"). Значения *имеют формат TimeSpan*. *чч*:*мм*:*СС*. | Строка | Нет |
**Отчет о состоянии при обратном вызове** | Позволяет пользователю сообщать о неудачном завершении действия веб-перехватчика. | Логическое | Нет |

## <a name="authentication"></a>Аутентификация

Действие веб-перехватчика поддерживает следующие типы проверки подлинности.

### <a name="none"></a>Нет

Если проверка подлинности не требуется, не включайте свойство **authentication** .

### <a name="basic"></a>Basic

Укажите имя пользователя и пароль для использования с обычной проверкой подлинности.

```json
"authentication":{
   "type":"Basic",
   "username":"****",
   "password":"****"
}
```

### <a name="client-certificate"></a>Сертификат клиента

Укажите содержимое PFX-файла и пароль в кодировке Base64.

```json
"authentication":{
   "type":"ClientCertificate",
   "pfx":"****",
   "password":"****"
}
```

### <a name="managed-identity"></a>Управляемое удостоверение

Используйте управляемое удостоверение фабрики данных, чтобы указать URI ресурса, для которого запрашивается маркер доступа. Для вызова API управления ресурсами Azure используйте `https://management.azure.com/`. Дополнительные сведения о работе управляемых удостоверений см. в [обзоре управляемых удостоверений для ресурсов Azure](../active-directory/managed-identities-azure-resources/overview.md).

```json
"authentication": {
    "type": "MSI",
    "resource": "https://management.azure.com/"
}
```

> [!NOTE]
> Если ваша фабрика данных настроена с репозиторием Git, необходимо сохранить учетные данные в Azure Key Vault, чтобы использовать проверку подлинности "базовый" или "клиент-сертификат". В службе "Фабрика данных Azure" не хранятся пароли в Git.

## <a name="additional-notes"></a>Дополнительные замечания

Фабрика данных передает дополнительное свойство **каллбаккури** в тексте, отправляемом конечной точке URL-адреса. Фабрика данных ждет, что этот URI будет вызываться до указанного значения времени ожидания. Если URI не вызывается, действие завершается с состоянием «TimedOut».

Действие веб-перехватчика завершается ошибкой при сбое вызова пользовательской конечной точки. Любое сообщение об ошибке может быть добавлено в текст обратного вызова и использоваться в последующих действиях.

При каждом вызове REST API время ожидания клиента истекает, если конечная точка не отвечает в течение одной минуты. Такое поведение является стандартной рекомендацией по протоколу HTTP. Чтобы устранить эту проблему, Реализуйте шаблон 202. В текущем случае конечная точка возвращает 202 (принято) и Клиент опрашивает.

Время ожидания, равное 1 минуте, для запроса не имеет значения времени ожидания действия. Второй метод используется для ожидания обратного вызова, заданного параметром **каллбаккури**.

Текст, переданный обратно в URI обратного вызова, должен быть допустимым JSON. В качестве заголовка `Content-Type` установите `application/json`.

При использовании свойства " **состояние отчета" для обратного вызова** необходимо добавить следующий код в тело при выполнении обратного вызова:

```json
{
    "Output": {
        // output object is used in activity output
        "testProp": "testPropValue"
    },
    "Error": {
        // Optional, set it when you want to fail the activity
        "ErrorCode": "testErrorCode",
        "Message": "error message to show in activity error"
    },
    "StatusCode": "403" // when status code is >=400, activity is marked as failed
}
```

## <a name="next-steps"></a>Следующие шаги

См. следующие действия потока управления, поддерживаемые фабрикой данных.

- [Действие условия If](control-flow-if-condition-activity.md)
- [Действие выполнения конвейера](control-flow-execute-pipeline-activity.md)
- [Действие For Each](control-flow-for-each-activity.md)
- [Действие получения метаданных](control-flow-get-metadata-activity.md)
- [Действие поиска](control-flow-lookup-activity.md)
- [Веб-действие](control-flow-web-activity.md)
- [Действие Until](control-flow-until-activity.md)