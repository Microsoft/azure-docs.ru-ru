---
title: Передача маркеров проверки подлинности в службы мультимедиа v3 | Документация Майкрософт
description: Узнайте, как отправить маркеры проверки подлинности от клиента службе доставки ключей версии 3 служб мультимедиа.
services: media-services
author: IngridAtMicrosoft
manager: femila
ms.service: media-services
ms.topic: how-to
ms.date: 03/10/2021
ms.author: inhenkel
ms.openlocfilehash: 590309ad392876ba3c5cb6d21abe777ab5a93efb
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105572543"
---
# <a name="pass-tokens-to-the-azure-media-services-v3-key-delivery-service"></a>Передача токенов в службу доставки ключей служб мультимедиа Azure версии 3

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Клиенты часто спрашивают, как проигрыватель может передать маркеры аутентификации в службу доставки ключей Служб мультимедиа Azure, чтобы получить ключ. Службы мультимедиа поддерживают форматы простого веб-маркера (SWT) и JSON Web Token (JWT). Аутентификация с использованием маркера применима к ключу любого типа, независимо от того, какое шифрование выполняется в системе: общее или шифрование конвертов AES.

 В зависимости от целевой платформы и проигрывателя вы можете передавать маркер с помощью проигрывателя следующими способами.

## <a name="pass-a-token-through-the-http-authorization-header"></a>Передача маркера через заголовок авторизации HTTP

> [!NOTE]
> Для каждой спецификации OAuth 2.0. ожидается префикс Bearer. Выберите **AES (маркер JWT)** или **AES (маркер SWT)**, чтобы указать источник видео. Маркер передается через заголовок авторизации.

## <a name="pass-a-token-via-the-addition-of-a-url-query-parameter-with-tokentokenvalue"></a>Передайте токен с помощью добавления параметра запроса URL-адреса с помощью "Token = tokenvalue".

> [!NOTE]
> Префикс Bearer не ожидается. Так как маркер отправляется через URL-адрес, необходимо защитить строку маркера. В этом примере кода C# показано, как это сделать:

```csharp
    string armoredAuthToken = System.Web.HttpUtility.UrlEncode(authToken);
    string uriWithTokenParameter = string.Format("{0}&token={1}", keyDeliveryServiceUri.AbsoluteUri, armoredAuthToken);
    Uri keyDeliveryUrlWithTokenParameter = new Uri(uriWithTokenParameter);
```

## <a name="pass-a-token-through-the-customdata-field"></a>Передача токена через поле CustomData

Этот параметр необходим только для приобретения лицензии PlayReady. С использованием поля CustomData в запросе на приобретение лицензии PlayReady. В этом случае маркер должен содержаться в XML-документе, описанном ниже.

```xml
    <?xml version="1.0"?>
    <CustomData xmlns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/PlayReadyCustomData/v1"> 
        <Token></Token> 
    </CustomData>
```

Укажите маркер аутентификации в элементе "Маркер".
