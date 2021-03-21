---
title: Создание входных данных задания из URL-адреса HTTPS
description: В этом разделе показано, как создать входные данные задания служб мультимедиа Azure из URL-адреса HTTPS.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: how-to
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: 1073a13f477894e1b35d732fd1cd6191747a62a2
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98955993"
---
# <a name="create-a-job-input-from-an-https-url"></a>Создание входных данных задания из URL-адреса HTTPS

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

При отправке заданий из Служб мультимедиа версии 3 необходимо указать расположение входного видео. Это можно сделать, указав URL-адрес HTTPS как входные данные задания (как показано в этом примере). Обратите внимание, что в настоящее время AMS версии 3 не поддерживает кодирование блочной передачи по URL-адресам HTTPS. Полный пример см. в этом [репозитории GitHub](https://github.com/Azure-Samples/media-services-v3-dotnet-quickstarts/blob/master/AMSV3Quickstarts/EncodeAndStreamFiles/Program.cs).

> [!TIP]
> Перед началом разработки ознакомьтесь [с разработкой с помощью API-интерфейсов служб мультимедиа v3](media-services-apis-overview.md) (содержит сведения о доступе к API, соглашения об именовании и т. д.).

## <a name="net-sample"></a>Пример кода .NET

В следующем коде показано, как создать задание с помощью входных данных URL-адреса HTTPS.

[!code-csharp[Main](../../../media-services-v3-dotnet-quickstarts/AMSV3Quickstarts/EncodeAndStreamFiles/Program.cs#SubmitJob)]

## <a name="job-error-codes"></a>Коды ошибок задания

См. статью о [кодах ошибок](/rest/api/media/jobs/get#joberrorcode).

## <a name="next-steps"></a>Дальнейшие действия

[Создание входных данных задания из локального файла](job-input-from-local-file-how-to.md).
