---
title: Аутентификация
titleSuffix: Azure Cognitive Services
description: 'Существует три способа проверки подлинности запроса к ресурсу Azure Cognitive Services: ключ подписки, маркер носителя и подписка на несколько служб. В этой статье вы получите информацию о всех этих методах и узнаете, как выполнить запрос.'
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 11/22/2019
ms.author: erhopf
ms.openlocfilehash: c7aeb9e9f4de7b4de62f9b5a8da6d997e32a2399
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94363329"
---
# <a name="authenticate-requests-to-azure-cognitive-services"></a>Проверка подлинности запросов к Azure Cognitive Services

Каждый запрос к службе Azure Cognitive должен содержать заголовок проверки подлинности. Этот заголовок передает ключ подписки или маркер доступа, который позволяет проверить права подписки на доступ к службе или группе служб. В этой статье вы получите сведения о трех методах проверки подлинности запроса и требованиях для каждого из них.

* Проверка подлинности с помощью ключа подписки с [одной](#authenticate-with-a-single-service-subscription-key) или [несколькими службами](#authenticate-with-a-multi-service-subscription-key)
* Проверка подлинности с помощью [токена](#authenticate-with-an-authentication-token)
* Проверка подлинности с помощью [Azure Active Directory (AAD)](#authenticate-with-azure-active-directory)

## <a name="prerequisites"></a>Предварительные условия

Чтобы выполнять запрос, вам нужно иметь учетную запись Azure и подписку Azure Cognitive Services. Если у вас есть учетная запись, пропустите этот раздел и перейдите к следующему. Если у вас нет учетной записи, у вас есть руководство по настройке в течение нескольких минут: [Создайте учетную запись Cognitive Services для Azure](cognitive-services-apis-create-account.md).

Вы можете получить ключ подписки из [портал Azure](cognitive-services-apis-create-account.md#get-the-keys-for-your-resource) после [создания учетной записи](https://azure.microsoft.com/free/cognitive-services/).

## <a name="authentication-headers"></a>Заголовки проверки подлинности

Давайте кратко рассмотрим заголовки проверки подлинности, которые доступны для использования с Azure Cognitive Services.

| Header | Описание |
|--------|-------------|
| Ocp-Apim-Subscription-Key | Используйте этот заголовок для проверки подлинности по ключу подписки для конкретной службы или для нескольких служб. |
| Ocp-Apim-Subscription-Region | Этот заголовок необходим только при использовании ключа подписки с несколькими службами в [службе переводчиков](./Translator/reference/v3-0-reference.md). Используйте этот заголовок, чтобы указать регион подписки. |
| Авторизация | Используйте этот заголовок, если вы применяете маркер проверки подлинности. В следующих разделах подробно описан процесс обмена маркерами. Значение предоставляется в следующем формате: `Bearer <TOKEN>`. |

## <a name="authenticate-with-a-single-service-subscription-key"></a>Проверка подлинности по ключу подписки для одной службы

Первый вариант — Проверка подлинности запроса с ключом подписки для определенной службы, например транслятора. Ключи можно получить на портале Azure для каждого созданного ресурса. Чтобы использовать ключ подписки для проверки подлинности запроса, этот ключ следует передать в заголовке `Ocp-Apim-Subscription-Key`.

В этих примерах запросов показано использование заголовка `Ocp-Apim-Subscription-Key`. Учтите, что для работы с этим примером вам потребуется действительный ключ подписки.

Вот пример вызова к API Bing для поиска в Интернете:
```cURL
curl -X GET 'https://api.cognitive.microsoft.com/bing/v7.0/search?q=Welsch%20Pembroke%20Corgis' \
-H 'Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY' | json_pp
```

Это пример вызова службы переводчика:
```cURL
curl -X POST 'https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=de' \
-H 'Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY' \
-H 'Content-Type: application/json' \
--data-raw '[{ "text": "How much for the cup of coffee?" }]' | json_pp
```

В следующем видео показано использование ключа Cognitive Services.

## <a name="authenticate-with-a-multi-service-subscription-key"></a>Проверка подлинности по ключу подписки для нескольких служб

>[!WARNING]
> В настоящее время эти службы **не** поддерживают ключи нескольких служб: QnA Maker, службы речи, пользовательское визуальное распознавание и детектор аномалий.

Этот метод проверки подлинности запросов также использует ключ подписки. Основное различие заключается в том, что этот ключ подписки не привязан к конкретной службе, а применяется для проверки подлинности запросов к нескольким службам Cognitive Services. На странице [цен на Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services/) вы найдете сведения о доступности по регионам, поддерживаемых функциях и стоимости.

Ключ подписки предоставляется в заголовке `Ocp-Apim-Subscription-Key` каждого запроса.

[![Демонстрация ключа подписки с несколькими службами для Cognitive Services](./media/index/single-key-demonstration-video.png)](https://www.youtube.com/watch?v=psHtA1p7Cas&feature=youtu.be)

### <a name="supported-regions"></a>Поддерживаемые регионы

Если вы используете ключ подписки для нескольких служб в запросе к `api.cognitive.microsoft.com`, необходимо указать регион в URL-адресе. Например, `westus.api.cognitive.microsoft.com`.

При использовании ключа подписки с несколькими службами в службе переводчика необходимо указать регион подписки с `Ocp-Apim-Subscription-Region` заголовком.

Проверка подлинности по ключу для нескольких служб поддерживается в следующих регионах:

- `australiaeast`
- `brazilsouth`
- `canadacentral`
- `centralindia`
- `eastasia`
- `eastus`
- `japaneast`
- `northeurope`
- `southcentralus`
- `southeastasia`
- `uksouth`
- `westcentralus`
- `westeurope`
- `westus`
- `westus2`

### <a name="sample-requests"></a>Примеры запросов

Вот пример вызова к API Bing для поиска в Интернете:

```cURL
curl -X GET 'https://YOUR-REGION.api.cognitive.microsoft.com/bing/v7.0/search?q=Welsch%20Pembroke%20Corgis' \
-H 'Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY' | json_pp
```

Это пример вызова службы переводчика:

```cURL
curl -X POST 'https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=de' \
-H 'Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY' \
-H 'Ocp-Apim-Subscription-Region: YOUR_SUBSCRIPTION_REGION' \
-H 'Content-Type: application/json' \
--data-raw '[{ "text": "How much for the cup of coffee?" }]' | json_pp
```

## <a name="authenticate-with-an-authentication-token"></a>Проверка подлинности по маркеру проверки подлинности

Некоторые службы Azure Cognitive Services поддерживают или даже требуют маркер проверки подлинности. Сейчас маркеры проверки подлинности поддерживают следующие службы:

* API перевода текстов;
* Речевые службы: REST API преобразования речи в текст
* Речевые службы: преобразование текста в речь REST API

>[!NOTE]
> QnA Maker также использует заголовок авторизации, но требует указывать ключ конечной точки. Дополнительные сведения см. в разделе [QnA Maker: получить ответ из базы знаний](./qnamaker/quickstarts/get-answer-from-knowledge-base-using-url-tool.md).

>[!WARNING]
> Со временем список служб, которые поддерживают маркеры проверки подлинности, может меняться. Обратитесь к документации по API конкретной службы, прежде чем внедрять этот метод проверки подлинности.

Маркер проверки подлинности можно получить в обмен на любой ключ подписки: для одной или для нескольких службы. Маркер проверки подлинности действует в течение 10 минут.

Маркеры проверки подлинности включаются в запрос в заголовке `Authorization`. К значению маркера следует добавить префикс `Bearer`, например так: `Bearer YOUR_AUTH_TOKEN`.

### <a name="sample-requests"></a>Примеры запросов

Используйте этот URL-адрес для обмена ключа подписки на маркер проверки подлинности: `https://YOUR-REGION.api.cognitive.microsoft.com/sts/v1.0/issueToken`.

```cURL
curl -v -X POST \
"https://YOUR-REGION.api.cognitive.microsoft.com/sts/v1.0/issueToken" \
-H "Content-type: application/x-www-form-urlencoded" \
-H "Content-length: 0" \
-H "Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY"
```

Обмен маркерами поддерживается в следующих регионах с несколькими службами:

- `australiaeast`
- `brazilsouth`
- `canadacentral`
- `centralindia`
- `eastasia`
- `eastus`
- `japaneast`
- `northeurope`
- `southcentralus`
- `southeastasia`
- `uksouth`
- `westcentralus`
- `westeurope`
- `westus`
- `westus2`

Получив маркер проверки подлинности, его следует включать в каждый запрос в заголовке `Authorization`. Это пример вызова службы переводчика:

```cURL
curl -X POST 'https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=de' \
-H 'Authorization: Bearer YOUR_AUTH_TOKEN' \
-H 'Content-Type: application/json' \
--data-raw '[{ "text": "How much for the cup of coffee?" }]' | json_pp
```

[!INCLUDE [](../../includes/cognitive-services-azure-active-directory-authentication.md)]

## <a name="see-also"></a>См. также раздел

* [Что такое Cognitive Services?](./what-are-cognitive-services.md)
* [Цены на Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services/)
* [Пользовательские поддомены](cognitive-services-custom-subdomains.md)