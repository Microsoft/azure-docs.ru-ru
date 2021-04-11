---
title: Передача маркеров аутентификации в Службы мультимедиа Azure | Документация Майкрософт
description: Отправка маркеров аутентификации из клиента в службу доставки ключей Служб мультимедиа Azure
services: media-services
keywords: content protection, DRM, token authentication
author: IngridAtMicrosoft
manager: femila
ms.assetid: 7c3b35d9-1269-4c83-8c91-490ae65b0817
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/22/2021
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: 1fe692e1eb20956f339c9b861f50163cee9c5063
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105564659"
---
# <a name="learn-how-clients-pass-tokens-to-the-azure-media-services-key-delivery-service"></a>Сведения о том, как клиенты передают маркеры в службу доставки ключей Служб мультимедиа Azure

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Клиенты часто спрашивают, как проигрыватель может передать маркеры аутентификации в службу доставки ключей Служб мультимедиа Azure, чтобы получить ключ. Службы мультимедиа поддерживают форматы простого веб-маркера (SWT) и JSON Web Token (JWT). Аутентификация с использованием маркера применима к ключу любого типа, независимо от того, какое шифрование выполняется в системе: общее или шифрование конвертов AES.

 В зависимости от целевой платформы и проигрывателя вы можете передавать маркер с помощью проигрывателя следующими способами.

- С использованием заголовка авторизации HTTP.
    > [!NOTE]
    > Для каждой спецификации OAuth 2.0. ожидается префикс Bearer. Выберите **AES (маркер JWT)** или **AES (маркер SWT)**, чтобы указать источник видео. Маркер передается через заголовок авторизации.

- Путем добавления параметра запроса URL-адреса со строкой token=tokenvalue.  
    > [!NOTE]
    > Префикс Bearer не ожидается. Так как маркер отправляется через URL-адрес, необходимо защитить строку маркера. В этом примере кода C# показано, как это сделать:

    ```csharp
    string armoredAuthToken = System.Web.HttpUtility.UrlEncode(authToken);
    string uriWithTokenParameter = string.Format("{0}&token={1}", keyDeliveryServiceUri.AbsoluteUri, armoredAuthToken);
    Uri keyDeliveryUrlWithTokenParameter = new Uri(uriWithTokenParameter);
    ```

- С использованием поля CustomData.
Этот параметр необходим только для приобретения лицензии PlayReady. С использованием поля CustomData в запросе на приобретение лицензии PlayReady. В этом случае маркер должен содержаться в XML-документе, описанном ниже.

    ```xml
    <?xml version="1.0"?>
    <CustomData xmlns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/PlayReadyCustomData/v1"> 
        <Token></Token> 
    </CustomData>
    ```
    Укажите маркер аутентификации в элементе "Маркер".

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]
