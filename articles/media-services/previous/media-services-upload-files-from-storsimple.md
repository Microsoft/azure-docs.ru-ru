---
title: Отправка файлов в учетную запись служб мультимедиа Azure из Azure StorSimple | Документация Майкрософт
description: В этой статье рассматривается использование диспетчера данных Azure StorSimple. Она также содержит ссылки на руководства, в которых описывается, как извлечь данные из StorSimple и передать их в качестве ресурсов в учетную запись служб мультимедиа Azure.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: 1dd09328-262b-43ef-8099-73241b49a925
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 3/10/2021
ms.author: inhenkel
ms.openlocfilehash: 0521904f0ed46b4c5309e5f9df980b1cd7d7d858
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103009034"
---
# <a name="upload-files-into-an-azure-media-services-account-from-azure-storsimple"></a>Отправка файлов в учетную запись служб мультимедиа Azure из Azure StorSimple 

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)] 

> [!NOTE]
> В Cлужбы мультимедиа версии 2 больше не добавляются новые компоненты или функциональные возможности. <br/>Ознакомьтесь с новейшей версией Служб мультимедиа — [версией 3](../latest/index.yml). Также изучите руководство по [миграции из версии 2 в версию 3](../latest/migrate-v-2-v-3-migration-introduction.md).
>
> 
> В настоящее время диспетчер данных Azure StorSimple доступен в качестве закрытой предварительной версии. 
> 

## <a name="overview"></a>Обзор

В службах мультимедиа цифровые файлы отправляются в актив. Asset может содержать видео, аудио, изображения, коллекции эскизов, текстовые дорожки и файлы скрытых субтитров (а также метаданные этих файлов). После отправки этих файлов содержимое сохраняется в безопасном расположении в облаке для дальнейшей обработки и потоковой передачи.

[Azure StorSimple](../../storsimple/index.yml) использует облачное хранилище в качестве расширения локального решения и автоматически связывает данные локального хранилища с данными облачного хранилища. Перед отправкой данных в облако устройство StorSimple дублирует и сжимает их. Таким образом, в облако можно эффективно отправлять большие файлы. Служба [диспетчера данных StorSimple](../../storsimple/storsimple-data-manager-overview.md) предоставляет интерфейсы API, позволяющие извлекать данные из StorSimple и представлять их в качестве ресурсов AMS.

## <a name="get-started"></a>Начало работы

1. [Создайте учетную запись служб мультимедиа](media-services-portal-create-account.md), в которую вы хотите перенести ресурсы.
2. Зарегистрируйтесь для получения предварительной версии диспетчера данных, как описано в статье [Общие сведения о диспетчере данных StorSimple (закрытая предварительная версия)](../../storsimple/storsimple-data-manager-overview.md).
3. Создайте учетную запись диспетчера данных StorSimple.
4. Создайте задание для преобразования данных, которое извлекает данные из устройства StorSimple, а затем переносит их в учетную запись AMS в качестве ресурсов. 

    При запуске задания создается очередь хранилища. Эта очередь по мере готовности заполняется сообщениями о преобразованных больших двоичных объектах. Имя этой очереди совпадает с именем определения задания. Вы можете использовать эту очередь для определения готовности ресурсов, а затем вызвать необходимую операцию служб мультимедиа для этих ресурсов. Например, вы можете использовать эту очередь, чтобы активировать функции Azure, в которых содержится нужный код служб мультимедиа.

## <a name="see-also"></a>См. также раздел

[Использование пакета SDK для .NET для активации заданий в Диспетчере данных](../../storsimple/storsimple-data-manager-dotnet-jobs.md)

## <a name="media-services-learning-paths"></a>Схемы обучения работе со службами мультимедиа
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-steps"></a>Дальнейшие действия

Теперь можно закодировать отправленные ресурсы. Дополнительную информацию см. в статье, посвященной [кодированию ресурсов](media-services-portal-encode.md).
