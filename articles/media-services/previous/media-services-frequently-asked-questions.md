---
title: Часто задаваемые вопросы о Службах мультимедиа Azure
description: В этой статье предоставлены ответы на часто задаваемые вопросы о Службах мультимедиа Azure.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.openlocfilehash: 220aae64bd9ec493af8c8ee61901e27027dc9798
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103013386"
---
# <a name="media-services-v2-frequently-asked-questions"></a>Часто задаваемые вопросы о Службах мультимедиа Azure версии 2

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

В этой статье рассматриваются часто задаваемые вопросы сообщества пользователей служб мультимедиа Azure (AMS).

## <a name="general-ams-faqs"></a>Общие часто задаваемые вопросы об AMS и ответы на них

Вопрос. Как осуществляется потоковая передача на устройства Apple iOS?

Ответ. Добавьте путь (format=m3u8-aapl) в часть /Manifest URL-адреса, чтобы сообщить серверу-источнику потоковой передачи о необходимости возврата содержимого HLS для использования на собственных устройствах Apple iOS. (см. дополнительные сведения о [доставке содержимого](media-services-deliver-content-overview.md)).

Вопрос. Как осуществляется масштабирование индексирования?

A. Зарезервированные единицы одинаковы для задач кодирования и индексирования. Следуйте инструкциям в разделе о [масштабировании зарезервированных единиц кодирования](media-services-scale-media-processing-overview.md). **Обратите внимание**, что производительность индексирования не зависит от типа зарезервированных единиц.

Вопрос. Видео отправлено, закодировано и опубликовано. Почему видео не воспроизводится?

A. Чаще всего это вызвано отсутствием конечной точки потоковой передачи, из которой выполняется попытка воспроизведения в состоянии **Выполняется**.  

Вопрос. Можно ли объединять видео в динамическом потоке?

A. Объединение видео в динамических потоках сейчас не поддерживается в Службах мультимедиа Azure, поэтому необходимо объединить видео заранее на компьютере.

Вопрос. Можно ли использовать Azure CDN с потоковой трансляцией?

A. Службы мультимедиа поддерживают интеграцию с Azure CDN (дополнительные сведения см. в статье [Управление конечными точками потоковой передачи с помощью портала Azure](media-services-portal-manage-streaming-endpoints.md)).  Вы можете использовать динамическую потоковую передачу с CDN. Службы мультимедиа Azure предоставляют выходные данные в форматах Smooth Streaming, HLS и MPEG-DASH. Все они используют протокол HTTP для передачи данных, а также кэширование HTTP. При динамической потоковой передаче фактические аудио- или видеоданные делятся на фрагменты, которые кэшируются в сети CDN. Обновлять необходимо только данные манифеста. CDN периодически обновляет данные манифеста.

Вопрос. Поддерживают ли Службы мультимедиа Azure хранение образов?

A. Если вы хотите просто хранить JPEG- или PNG-образы, используйте для этого хранилище BLOB-объектов Azure. Не имеет смысла хранить их в вашей учетной записи служб мультимедиа, если вы не хотите связать их с видео- или аудиоресурсами. Или вам может понадобиться использовать наложение изображений в видеокодировщике. Стандартный кодировщик мультимедиа поддерживает наложение изображений на видео, поэтому JPEG и PNG указаны как поддерживаемые форматы входных данных. Дополнительные сведения см. в статье [Создание наложений](media-services-advanced-encoding-with-mes.md#overlay).

Вопрос. Как можно скопировать активы из одной учетной записи Службы мультимедиа в другую?

A. Для копирования ресурсов из одной учетной записи Службы мультимедиа в другую с помощью .NET используйте метод расширения [IAsset.Copy](https://github.com/Azure/azure-sdk-for-media-services-extensions/blob/dev/MediaServices.Client.Extensions/IAssetExtensions.cs#L354), доступный в репозитории [Windows Azure Media Services .NET SDK Extensions](https://github.com/Azure/azure-sdk-for-media-services-extensions/) (Расширения пакета SDK .NET Службы мультимедиа Azure для Windows). Дополнительные сведения см. в [этой теме форума](https://social.msdn.microsoft.com/Forums/azure/28912d5d-6733-41c1-b27d-5d5dff2695ca/migrate-media-services-across-subscription?forum=MediaServices).

Вопрос. Какие знаки поддерживаются в именах файлов при работе с AMS?

A. Службы мультимедиа используют значение свойства IAssetFile.Name при создании URL-адресов для потоковой передачи содержимого (например, http://{AMSAccount}.origin.mediaservices.windows.net/{GUID}/{IAssetFile.Name}/streamingParameters.) По этой причине кодирование с помощью знака процента не допускается. Значение свойства **Name** не может содержать такие [зарезервированные знаки, используемые для кодировки URL-адресов](https://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters): !*'();:@&=+$,/?%#[]". Кроме того, может использоваться только один знак "." для расширения имени файла.

Вопрос. Как можно подключиться с помощью REST?

A. Сведения о подключении к API AMS см. в разделе [Доступ к API служб мультимедиа Azure с помощью аутентификации Azure AD](media-services-use-aad-auth-to-access-ams-api.md). 

Вопрос. Как повернуть видео в процессе кодирования?

A. [Стандартный кодировщик служб мультимедиа](media-services-dotnet-encode-with-media-encoder-standard.md) поддерживает поворот на 90, 180 и 270 градусов. По умолчанию задается значение Auto, при котором система пытается обнаружить метаданные поворота во входящем файле MP4/MOV и обеспечить соответствующую компенсацию. Включите следующий элемент **Sources** в одну из предустановок JSON, определенных [здесь](media-services-mes-presets-overview.md):

```json
"Version": 1.0,
"Sources": [
{
  "Streams": [],
  "Filters": {
    "Rotation": "90"
  }
}
],
"Codecs": [

...
```


## <a name="media-services-learning-paths"></a>Схемы обучения работе со службами мультимедиа
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
