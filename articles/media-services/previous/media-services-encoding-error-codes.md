---
title: Коды ошибок кодирования служб мультимедиа Azure | Документация Майкрософт
description: В этом разделе перечислены коды ошибок, которые могут быть возвращены при возникновении ошибок во время выполнения задачи кодирования.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: ce4e939f-5aee-41f9-859d-e4429815e9f2
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.openlocfilehash: 7e6848fb49dd63fa67a639d09754a28dd5953a32
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103013488"
---
# <a name="encoding-error-codes"></a>Коды ошибок кодирования

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

В следующей таблице перечислены коды ошибок, которые могут быть возвращены при возникновении ошибок во время выполнения задачи кодирования.  Чтобы получить сведения об ошибке в коде .NET, используйте класс [ErrorDetails](/previous-versions/azure/jj126075(v=azure.100)) . Чтобы получить сведения об ошибке в коде REST, используйте REST API [ErrorDetail](/rest/api/media/operations/errordetail) .

| ErrorDetail.Code | Возможные причины ошибки |
| --- | --- |
| Неизвестно |Неизвестная ошибка при выполнении задачи |
| ErrorDownloadingInputAssetMalformedContent |Категория ошибок, охватывающая ошибки загрузки входного ресурса, такие как некорректное имя файлов, файлы нулевой длины, неправильное форматирование и т. д. |
| ErrorDownloadingInputAssetServiceFailure |Категория ошибок, охватывающая проблемы на стороне службы, например проблемы с сетью или хранилищем во время загрузки. |
| ErrorParsingConfiguration |Категория ошибок, в которых задача \<see cref="MediaTask.PrivateData"/> (конфигурация) является недопустимой, например: конфигурация не является допустимой конфигурацией системы или содержит недопустимый XML. |
| ErrorExecutingTaskMalformedContent |Категория ошибок, возникающих во время выполнения задачи, которые могут быть вызваны проблемами во входных файлах мультимедиа. |
| ErrorExecutingTaskUnsupportedFormat |Категория ошибок, в которой обработчик мультимедиа не может обработать представленные файлы — формат мультимедиа не поддерживается или не соответствует конфигурации. Например, при попытке получить только звук из ресурса, который содержит видео. |
| ErrorProcessingTask |Категория других ошибок обработчика мультимедиа, возникших во время обработки задачи, которые не связаны с содержимым. |
| ErrorUploadingOutputAsset |Категория ошибок при отправке выходного ресурса |
| ErrorCancelingTask |Категория ошибок, охватывающая ошибки, возникающие при попытке отменить задачу |
| TransientError |Категория ошибок, охватывающая временные проблемы (например, временные сетевые проблемы со службой хранилища Azure) |

Для получения помощи от команды разработчиков для **служб мультимедиа** создайте [запрос в службу поддержки](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

## <a name="media-services-learning-paths"></a>Схемы обучения работе со службами мультимедиа
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="related-articles"></a>Похожие статьи
* [Выполняйте расширенные задачи кодирования путем настройки предустановок стандартного кодировщика служб мультимедиа](media-services-custom-mes-presets-with-dotnet.md)
* [Квоты и ограничения](media-services-quotas-and-limitations.md)

<!--Reference links in article-->
[1]: https://azure.microsoft.com/pricing/details/media-services/
