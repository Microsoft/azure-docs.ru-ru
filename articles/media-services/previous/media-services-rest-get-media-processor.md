---
title: Как получить экземпляр обработчика мультимедиа с помощью REST | Документация Майкрософт
description: Узнайте, как создать компонент обработчика мультимедиа для кодирования, преобразования формата, шифрования или расшифровки мультимедийного контента служб мультимедиа Azure.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: f9ff1997-0da6-4528-aaed-792837e5be41
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 3/10/2021
ms.author: inhenkel
ms.openlocfilehash: 7eb4647ae5ba40688cbf39cbfe8449f59275e7e6
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103014321"
---
# <a name="how-to-get-a-media-processor-instance"></a>Как получить экземпляр обработчика мультимедиа

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector"]
> * [.NET](media-services-get-media-processor.md)
> * [REST](media-services-rest-get-media-processor.md)


## <a name="overview"></a>Обзор

Обработчик мультимедиа — это компонент, который работает со специфическими задачами обработки видео или аудио, включая кодирование, преобразование формата, шифрование или расшифровку мультимедийного содержимого. Для выполнения всех задач, отправленных в Службы мультимедиа, включая кодирование, шифрование или преобразование содержимого видео и аудио, требуется обработчик мультимедиа.

## <a name="azure-media-processors"></a>Обработчики мультимедиа Azure

В следующем разделе приводятся списки обработчиков мультимедиа.

* [Кодировщики мультимедиа](scenarios-and-availability.md)
* [Обработчики мультимедиа аналитики](scenarios-and-availability.md)

>[!NOTE]
>При доступе к сущностям в службах мультимедиа необходимо задать определенные поля и значения заголовков в HTTP-запросах. Дополнительную информацию см. в статье [Обзор интерфейса REST API служб мультимедиа](media-services-rest-how-to-use.md).

## <a name="connect-to-media-services"></a>Подключение к службам мультимедиа

Сведения о подключении к API AMS см. в разделе [Доступ к API служб мультимедиа Azure с помощью аутентификации Azure AD](media-services-use-aad-auth-to-access-ams-api.md). 


## <a name="get-a-media-processor"></a>Получение обработчика мультимедиа

Следующий вызов REST показывает, как получить экземпляр обработчика мультимедиа по имени (в данном случае это **стандартный кодировщик мультимедиа**). 

Запрос:

```console
GET https://media.windows.net/api/MediaProcessors()?$filter=Name%20eq%20'Media%20Encoder%20Standard' HTTP/1.1
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
User-Agent: Microsoft ADO.NET Data Services
Authorization: Bearer <token>
x-ms-version: 2.19
Host: media.windows.net
```

Ответ:

```console
. . .

{  
   "odata.metadata":"https://media.windows.net/api/$metadata#MediaProcessors",
   "value":[  
      {  
         "Id":"nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",
         "Description":"Media Encoder Standard",
         "Name":"Media Encoder Standard",
         "Sku":"",
         "Vendor":"Microsoft",
         "Version":"1.1"
      }
   ]
}
```


## <a name="media-services-learning-paths"></a>Схемы обучения работе со службами мультимедиа
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-steps"></a>Next Steps
Теперь, когда вы знаете, как получить экземпляр обработчика мультимедиа, перейдите к инструкциям по [кодированию файлов](media-services-rest-get-started.md), в котором показано, как использовать Media Encoder Standard для кодирования файлов.

