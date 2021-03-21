---
title: Создание наложения с помощью Media Encoder Standard
description: Узнайте, как создать наложение с Media Encoder Standard.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: how-to
ms.date: 08/31/2020
ms.openlocfilehash: 69930e06e2ce7f2679feec74ca8ccbc93bdb8d30
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101721129"
---
# <a name="how-to-create-an-overlay-with-media-encoder-standard"></a>Создание наложения с помощью Media Encoder Standard

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Media Encoder Standard позволяет наложить изображение, звуковой файл или другое видео на другое видео. Входные данные должны указывать ровно один файл. Можно указать файл изображения в формате JPG, PNG, GIF или BMP или звуковой файл (например, файл WAV, MP3, WMA или M4A) или видеофайл.


## <a name="prerequisites"></a>Предварительные условия

* Собирайте сведения об учетной записи, необходимые для настройки *appsettings.jsдля* файла в примере. Если вы не знаете, как это сделать, см. раздел [Краткое руководство по регистрации приложения на платформе Microsoft Identity](../../active-directory/develop/quickstart-register-app.md). В *appsettings.jsдля* файла ожидались следующие значения.

    ```json
    {
    "AadClientId": "",
    "AadEndpoint": "https://login.microsoftonline.com",
    "AadSecret": "",
    "AadTenantId": "",
    "AccountName": "",
    "ArmAadAudience": "https://management.core.windows.net/",
    "ArmEndpoint": "https://management.azure.com/",
    "Location": "",
    "ResourceGroup": "",
    "SubscriptionId": ""
    }
    ```

Если вы еще не знакомы с преобразованиями, рекомендуется выполнить следующие действия.

* Чтение [кодирования видео и аудио с помощью служб мультимедиа](encoding-concept.md)
* Узнайте, [как кодировать с помощью пользовательского преобразования — .NET](customize-encoder-presets-how-to.md). Выполните действия, описанные в этой статье, чтобы настроить .NET для работы с преобразованиями, а затем вернитесь сюда, чтобы испытать пример предустановки наложения.
* См. [справочный документ преобразования](/rest/api/media/transforms).

Когда вы знакомы с преобразованиями, Скачайте пример наложения.

## <a name="overlays-preset-sample"></a>Образец предустановки наложения

Скачайте [образец мультимедиа-Services-Overlay](https://github.com/Azure-Samples/media-services-overlays) , чтобы начать работу с наложением.

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [transforms next steps](./includes/transforms-next-steps.md)]
