---
title: Azure Active Directory REST API-Test с помощью инструкции POST
description: Используйте POST для тестирования конфигурации приложения Azure REST API
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.topic: reference
ms.date: 08/17/2020
ms.openlocfilehash: 0b4444b049d181c2f66f8b02a5202dd17a3f4b6b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96932495"
---
# <a name="test-the-azure-app-configuration-rest-api-using-postman"></a>Тестирование REST API конфигурации приложений Azure с помощью POST

Чтобы протестировать REST API с помощью [процедуры POST](https://www.getpostman.com/), необходимо включить в запросы заголовки HTTP, необходимые для [проверки подлинности](./rest-api-authentication-hmac.md) . Вот как можно настроить POST для тестирования REST API, автоматически создавая заголовки проверки подлинности:

1. Создать новый [запрос](https://learning.getpostman.com/docs/postman/sending_api_requests/requests/)
1. Добавьте `signRequest` функцию из [примера проверки подлинности JavaScript](./rest-api-authentication-hmac.md#javascript) в [Скрипт предварительного запроса](https://learning.getpostman.com/docs/postman/scripts/pre_request_scripts/) для запроса.
1. Добавьте следующий код в конец скрипта, выполняемого перед запросом. Обновите ключ доступа, как указано в комментарии TODO.

    ```js
    // TODO: Replace the following placeholders with your access key
    var credential = "<Credential>"; // Id
    var secret = "<Secret>"; // Value

    var isBodyEmpty = pm.request.body === null || pm.request.body === undefined || pm.request.body.isEmpty();

    var headers = signRequest(
        pm.request.url.getHost(),
        pm.request.method,
        pm.request.url.getPathWithQuery(),
        isBodyEmpty ? undefined : pm.request.body.toString(),
        credential,
        secret);

    // Add headers to the request
    headers.forEach(header => {
        pm.request.headers.upsert({key: header.name, value: header.value});
    })
    ```

1. Отправка запроса
