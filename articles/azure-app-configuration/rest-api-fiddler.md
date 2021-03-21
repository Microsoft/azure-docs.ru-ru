---
title: Azure Active Directory REST API-Test с помощью Fiddler
description: Использование Fiddler для тестирования конфигурации приложения Azure REST API
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.topic: reference
ms.date: 08/17/2020
ms.openlocfilehash: 1142aa25212d87c5484963cda4e172df3d1fbafc
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96932614"
---
# <a name="test-the-azure-app-configuration-rest-api-using-fiddler"></a>Тестирование REST API конфигурации приложений Azure с помощью Fiddler

Чтобы протестировать REST API с помощью [Fiddler](https://www.telerik.com/fiddler), необходимо включить заголовки HTTP, необходимые для [проверки подлинности](./rest-api-authentication-hmac.md) в запросах. Вот как можно настроить Fiddler для тестирования REST API, автоматически создавая заголовки проверки подлинности:

1. Убедитесь, что TLS 1,2 является разрешенным протоколом:
    1. Последовательно выберите **Сервис**  >  **Параметры**  >  **HTTPS**.
    1. Убедитесь, что установлен флажок **расшифровка HTTPS-трафика** .
    1. В списке протоколов добавьте **TLS 1.2** , если он отсутствует.
1. Откройте **Редактор скриптов Fiddler** или нажмите клавиши **CTRL + R** в Fiddler
1. Добавьте следующий код в `Handlers` класс перед `OnBeforeRequest` функцией

    ```js
    static function SignRequest(oSession: Session, credential: String, secret: String) {
        var utcNow = DateTimeOffset.UtcNow.ToString("r", System.Globalization.DateTimeFormatInfo.InvariantInfo);
        var contentHash = ComputeSHA256Hash(oSession.RequestBody);
        var stringToSign = oSession.RequestMethod.ToUpperInvariant() + "\n" + oSession.PathAndQuery + "\n" + utcNow +";" + oSession.hostname + ";" + contentHash;
        var signature = ComputeHMACHash(secret, stringToSign);

        oSession.oRequest.headers["x-ms-date"] = utcNow;
        oSession.oRequest.headers["x-ms-content-sha256"] = contentHash;
        oSession.oRequest.headers["Authorization"] = "HMAC-SHA256 Credential=" + credential + "&SignedHeaders=x-ms-date;host;x-ms-content-sha256&Signature=" + signature;
    }

    static function ComputeSHA256Hash(content: Byte[]) {
        var sha256 = System.Security.Cryptography.SHA256.Create();
        try {
            return Convert.ToBase64String(sha256.ComputeHash(content));
        }
        finally {
            sha256.Dispose();
        }
    }

    static function ComputeHMACHash(secret: String, content: String) {
        var hmac = new System.Security.Cryptography.HMACSHA256(Convert.FromBase64String(secret));
        try {
            return Convert.ToBase64String(hmac.ComputeHash(System.Text.Encoding.ASCII.GetBytes(content)));
        }
        finally {
            hmac.Dispose();
        }
    }
    ```

1. Добавьте следующий код в конец `OnBeforeRequest` функции. Обновите ключ доступа, как указано в комментарии TODO.

    ```js
    if (oSession.isFlagSet(SessionFlags.RequestGeneratedByFiddler) &&
        oSession.hostname.EndsWith(".azconfig.io", StringComparison.OrdinalIgnoreCase)) {

        // TODO: Replace the following placeholders with your access key
        var credential = "<Credential>"; // Id
        var secret = "<Secret>"; // Value

        SignRequest(oSession, credential, secret);
    }
    ```
1. Использование [композитора Fiddler](https://docs.telerik.com/fiddler/Generate-Traffic/Tasks/CreateNewRequest) для создания и отправки запроса
