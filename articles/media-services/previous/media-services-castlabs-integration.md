---
title: Использование castLabs для предоставления лицензий Widevine для служб мультимедиа Azure | Документация Майкрософт
description: В этой статье описывается использование служб мультимедиа Azure (AMS) для доставки потока, который зашифрован динамически службой AMS, с помощью лицензий DRM PlayReady и Widevine.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: 2a9a408a-a995-49e1-8d8f-ac5b51e17d40
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.reviewer: willzhan
ms.openlocfilehash: 576ac636f166e2daebbb9919d6666fea913a17be
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103017058"
---
# <a name="using-castlabs-to-deliver-widevine-licenses-to-azure-media-services"></a>Использование castLabs для доставки лицензий Widevine для служб мультимедиа Azure

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]
 
> [!div class="op_single_selector"]
> * [Axinom](media-services-axinom-integration.md)
> * [castLabs](media-services-castlabs-integration.md)
> 
> 

## <a name="overview"></a>Обзор

В этой статье описывается использование служб мультимедиа Azure (AMS) для доставки потока, который зашифрован динамически службой AMS, с помощью лицензий DRM PlayReady и Widevine. Лицензию PlayReady выдает сервер лицензирования служб мультимедиа PlayReady, а лицензию Widevine — сервер лицензирования **castLabs** .

Чтобы воспроизвести потоковое содержимое, защищенное с помощью CENC (PlayReady или Widevine), используйте [Проигрыватель мультимедиа Azure](https://aka.ms/azuremediaplayer). Подробные сведения см. в [документе по AMP](https://amp.azure.net/libs/amp/latest/docs/).

На схеме показана высокоуровневая архитектура служб мультимедиа Azure и интеграция castLabs.

![интеграция](./media/media-services-castlabs-integration/media-services-castlabs-integration.png)

## <a name="typical-system-set-up"></a>Настройка типичной системы

* Мультимедийное содержимое хранится в AMS.
* Идентификаторы ключей содержимого хранятся в castLabs и AMS.
* CastLabs и AMS имеют встроенную проверку подлинности маркеров. В следующих разделах рассматриваются маркеры проверки подлинности. 
* Когда клиент запрашивает потоковую передачу видео, содержимое динамически шифруется с помощью **стандартного шифрования** (CENC) и динамически упаковывается сервером AMS для передачи по протоколах Smooth Streaming и DASH. Для протокола потоковой передачи HLS предлагается также шифрование элементарного потока PlayReady M2TS.
* Лицензию PlayReady выдает сервер лицензирования AMS, а лицензию Widevine — сервер лицензирования castLabs. 
* Проигрыватель мультимедиа автоматически выбирает нужную лицензию в соответствии с возможностями платформы клиента. 

## <a name="authentication-token-generation-for-getting-a-license"></a>Создания маркеров проверки подлинности для получения лицензии

CastLabs и AMS поддерживают формат маркера JWT (веб-маркера JSON), который используется для авторизации лицензии. 

### <a name="jwt-token-in-ams"></a>Маркер JWT в AMS

В приведенной ниже таблице описан маркер JWT в AMS. 

| Издатель | Строка издателя из выбранной службы маркеров безопасности |
| --- | --- |
| Аудитория |Строка аудитории из используемой службы маркеров безопасности |
| Утверждения |Набор утверждений |
| NotBefore |Начало действия маркера |
| Expires |Конец действия маркера |
| SigningCredentials |Общий ключ для сервера лицензирования PlayReady, castLabs и службы маркеров безопасности. Он может быть симметричным или асимметричным. |

### <a name="jwt-token-in-castlabs"></a>Маркер JWT в castLabs

В приведенной ниже таблице описан маркер JWT в castLabs. 

| Имя | Описание |
| --- | --- |
| optData |Строка JSON со сведениями о вас. |
| crt |Строка JSON со сведениями о файле, лицензии на него и правах на его воспроизведение. |
| iat |Текущая дата и время в Unix-времени. |
| jti |Уникальный идентификатор этого маркера (каждый маркер может использоваться в системе castLabs только один раз). |

## <a name="sample-solution-setup"></a>Настройка образца решения

[Образец решения](https://github.com/AzureMediaServicesSamples/CastlabsIntegration) состоит из двух проектов.

* Консольное приложение, которое можно использовать для задания ограничений DRM на уже существующий файл для PlayReady и Widevine.
* Веб-приложение, которое передает маркеры, которые можно рассматривать как ОЧЕНЬ УПРОЩЕННУЮ версию службы маркеров безопасности.

Чтобы использовать консольное приложение, сделайте следующее.

1. Измените файл app.config для настройки учетных данных для серверов AMS, castLabs, настройки службы маркеров безопасности и общего ключа.
2. Отправьте файл в AMS.
3. Получите UUID отправленного файла и измените строку 32 в файле Program.cs:
   
      var objIAsset = _context.Assets.Where(x => x.Id == "nb:cid:UUID:dac53a5d-1500-80bd-b864-f1e4b62594cf").FirstOrDefault();
4. Используйте AssetId для именования файла в системе castLabs (строка 44 в файле Program.cs).
   
   Необходимо задать AssetId в **castLabs**; это должна быть уникальная буквенно-цифровая строка.
5. Запустите программу.

Чтобы использовать веб-приложение (служба маркеров безопасности), сделайте следующее.

1. Измените файл web.config для настройки идентификатора подразделения castlabs, службы маркеров безопасности и общего ключа.
2. Разверните приложение на веб-сайтах Azure.
3. Перейдите на веб-сайт.

## <a name="playing-back-a-video"></a>Воспроизведение видео

Чтобы воспроизвести видео со стандартным шифрованием (PlayReady или Widevine), используйте [Проигрыватель мультимедиа Azure](https://aka.ms/azuremediaplayer). При запуске консольного приложения выводится идентификатор ключа содержимого и URL-адрес манифеста.

1. Откройте новую вкладку и запустите службу маркеров безопасности: http://[имя_службы_маркеров_безопасности].azurewebsites.net/api/token/assetid/[ваш_AssetId_в_CastLabs]/contentkeyid/[идентификатор_ключа_содержимого].
2. Перейдите к [Проигрывателю мультимедиа Azure](https://aka.ms/azuremediaplayer).
3. Вставьте URL-адрес для потоковой передачи.
4. Установите флажок **Дополнительные параметры** .
5. В раскрывающемся списке **Защита** выберите PlayReady и/или Widevine.
6. Вставьте маркер, полученный от службы маркеров безопасности в текстовом поле маркера. 
   
   Сервер лицензирования CastLab не использует префикс "Bearer=" перед маркером. Поэтому удалите его перед отправкой маркера.
7. Обновите проигрыватель.
8. Видео должно воспроизводиться.

## <a name="additional-notes"></a>Дополнительные замечания

* Widevine — это служба, которая предоставляется компанией Google Inc. и подпадает под условия предоставления услуг и политику конфиденциальности Google Inc.

## <a name="media-services-learning-paths"></a>Схемы обучения работе со службами мультимедиа
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

