---
title: Настройка учетной записи для автономной потоковой передачи содержимого, защищенного с помощью PlayReady, в Azure
description: В этой статье показано, как настроить учетную запись Служб мультимедиа Azure для автономной потоковой передачи содержимого с технологией PlayReady для Windows 10.
services: media-services
keywords: DASH, DRM, режим автономной работы Widevine, ExoPlayer, Android
documentationcenter: ''
author: IngridAtMicrosoft
manager: steveng
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 3/10/2021
ms.author: willzhan
ms.custom: devx-track-csharp
ms.openlocfilehash: 828895894464eafc53f74e7c8b5798db4db03533
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103016140"
---
# <a name="offline-playready-streaming-for-windows-10"></a>Потоковая передача содержимого с технологией PlayReady для Windows 10 (автономный режим)

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector" title1="Выберите версию служб мультимедиа, которую вы используете:"]
> * [Версия 3](../latest/offline-plaready-streaming-for-windows-10.md)
> * [Версия 2](offline-playready-streaming-windows-10.md)

> [!NOTE]
> В Cлужбы мультимедиа версии 2 больше не добавляются новые компоненты или функциональные возможности. <br/>Ознакомьтесь с новейшей версией Служб мультимедиа — [версией 3](../latest/index.yml). Также изучите руководство по [миграции из версии 2 в версию 3](../latest/migrate-v-2-v-3-migration-introduction.md).

Службы мультимедиа Azure поддерживают автономное скачивание и воспроизведение содержимого с защитой DRM. В этой статье рассматривается автономная поддержка Служб мультимедиа Azure для клиентов Windows 10 с PlayReady. Вы можете прочесть о поддержке автономного режима для устройств iOS с FairPlay и Android с Widevine в следующих статьях:

- [Потоковая передача FairPlay в автономном режиме для iOS](media-services-protect-hls-with-offline-fairplay.md)
- [Автономная потоковая передача Widevine для Android](offline-widevine-for-android.md)

## <a name="overview"></a>Обзор

В этом разделе приведены некоторые сведения о воспроизведении в автономном режиме, а именно почему:

* В некоторых странах и регионах доступ к Интернету и (или) пропускная способность по-прежнему ограничены. Пользователи могут сначала скачать содержимое, чтобы иметь возможность смотреть его в достаточно высоком разрешении для удобного просмотра. В этом случае чаще всего проблема заключается не в доступности сети, а в ее ограниченной пропускной способности. Поставщики OTT/OVP запрашивают поддержку автономного режима.
* На конференции акционеров Netflix в 3-м квартале 2016 года генеральный директор Netflix Рид Хэйстингс (Reed Hastings) сказал, что скачивание содержимого — это "часто запрашиваемая функция" и "мы открыты для нее".
* Некоторые поставщики содержимого могут запретить доставку лицензий DRM за рамку страны или региона. Если пользователь хочет смотреть содержимое во время поездок за границу, необходимо автономное скачивание.
 
Проблема, с которой мы сталкиваемся при реализации автономного режима, заключается в следующем:

* Формат MP4 поддерживается многими проигрывателями и инструментами кодирования, но между контейнером MP4 и DRM нет привязки.
* В долгосрочной перспективе CFF и CENC являются самыми оптимальными вариантами. Тем не менее в настоящее время экосистемы поддержки инструментов и проигрывателей еще не существует. Нам необходимо решение сегодня.
 
Идея состоит в том, что формат файла потоковой передачи Smooth Streaming ([PIFF](/iis/media/smooth-streaming/protected-interoperable-file-format)) с H264/AAC имеет привязку к PlayReady (AES-128 CTR). Отдельный ISMV-файл потоковой передачи Smooth Streaming (при условии, что в видео мультиплексный звук) сам по себе имеет формат fMP4 и может использоваться для воспроизведения. Если содержимое потоковой передачи Smooth Streaming проходит через шифрование PlayReady, каждый ISMV-файл получает фрагментированный MP4-формат, который защищен с помощью PlayReady. Мы можем выбрать ISMV-файл с предпочтительной скоростью и переименовать его в MP4 для скачивания.

Для поэтапной загрузки существует два варианта размещения MP4, защищенного с помощью PlayReady:

* файл MP4 можно поместить в один и тот же ресурс службы контейнера или мультимедиа и использовать конечную точку потоковой передачи Служб мультимедиа Azure для поэтапной загрузки;
* для поэтапной загрузки можно использовать указатель SAS непосредственно из службы хранилища Azure, минуя Службы мультимедиа Azure.
 
Вы можете использовать два типа доставки лицензии PlayReady:

* служба доставки лицензий PlayReady в Службах мультимедиа Azure;
* серверы лицензирования PlayReady, размещенные в любом месте.

Ниже приведены два набора тестовых ресурсов. Первый использует доставку лицензий PlayReady в AMS, а второй — сервер лицензий PlayReady, размещенный на виртуальной машине Azure:

Ресурс № 1:

* URL-адрес последовательного скачивания: [https://willzhanmswest.streaming.mediaservices.windows.net/8d078cf8-d621-406c-84ca-88e6b9454acc/20150807-bridges-2500_H264_1644kbps_AAC_und_ch2_256kbps.mp4](https://willzhanmswest.streaming.mediaservices.windows.net/8d078cf8-d621-406c-84ca-88e6b9454acc/20150807-bridges-2500_H264_1644kbps_AAC_und_ch2_256kbps.mp4)
* URL-адрес для приобретения лицензии PlayReady (AMS): `https://willzhanmswest.keydelivery.mediaservices.windows.net/PlayReady/`

Ресурс № 2:

* URL-адрес последовательного скачивания: [https://willzhanmswest.streaming.mediaservices.windows.net/7c085a59-ae9a-411e-842c-ef10f96c3f89/20150807-bridges-2500_H264_1644kbps_AAC_und_ch2_256kbps.mp4](https://willzhanmswest.streaming.mediaservices.windows.net/7c085a59-ae9a-411e-842c-ef10f96c3f89/20150807-bridges-2500_H264_1644kbps_AAC_und_ch2_256kbps.mp4)
* URL-адрес для приобретения лицензии PlayReady (локально): `https://willzhan12.cloudapp.net/playready/rightsmanager.asmx`

Для тестирования воспроизведения использовалось универсальное Windows-приложение на Windows 10. В [примерах приложений универсальной платформы Windows 10](https://github.com/Microsoft/Windows-universal-samples) есть пример простого проигрывателя, который называется [Adaptive Streaming Sample](https://github.com/Microsoft/Windows-universal-samples/tree/master/Samples/AdaptiveStreaming). Все, что нужно сделать, — это добавить код для выбора скачанного видео и использовать его в качестве источника вместо адаптивного источника потоковой передачи. Изменения в обработчике события нажатия кнопки:

```csharp
private async void LoadUri_Click(object sender, RoutedEventArgs e)
{
    //Uri uri;
    //if (!Uri.TryCreate(UriBox.Text, UriKind.Absolute, out uri))
    //{
    // Log("Malformed Uri in Load text box.");
    // return;
    //}
    //LoadSourceFromUriTask = LoadSourceFromUriAsync(uri);
    //await LoadSourceFromUriTask;

    //willzhan change start
    // Create and open the file picker
    FileOpenPicker openPicker = new FileOpenPicker();
    openPicker.ViewMode = PickerViewMode.Thumbnail;
    openPicker.SuggestedStartLocation = PickerLocationId.ComputerFolder;
    openPicker.FileTypeFilter.Add(".mp4");
    openPicker.FileTypeFilter.Add(".ismv");
    //openPicker.FileTypeFilter.Add(".mkv");
    //openPicker.FileTypeFilter.Add(".avi");

    StorageFile file = await openPicker.PickSingleFileAsync();

    if (file != null)
    {
       //rootPage.NotifyUser("Picked video: " + file.Name, NotifyType.StatusMessage);
       this.mediaPlayerElement.MediaPlayer.Source = MediaSource.CreateFromStorageFile(file);
       this.mediaPlayerElement.MediaPlayer.Play();
       UriBox.Text = file.Path;
    }
    else
    {
       // rootPage.NotifyUser("Operation cancelled.", NotifyType.ErrorMessage);
    }

    // On small screens, hide the description text to make room for the video.
    DescriptionText.Visibility = (ActualHeight < 500) ? Visibility.Collapsed : Visibility.Visible;
}
```

 ![Воспроизведение содержимого fMP4, защищенного с помощью PlayReady, в автономном режиме](./media/offline-playready/offline-playready1.jpg)

Так как видео находится под защитой PlayReady, на снимке экрана не отображается видео.

Таким образом, мы добились автономного режима в Службах мультимедиа Azure:

* Перекодирование содержимого и шифрование PlayReady можно выполнить в Службах мультимедиа Azure или других инструментах.
* Содержимое можно размещать в Службах мультимедиа Azure или в службе хранилища Azure для поэтапной загрузки.
* Доставка лицензии PlayReady может происходить из Служб мультимедиа Azure или из другого места.
* Подготовленное содержимое для потоковой передачи Smooth Streaming все еще можно использовать для потоковой передачи в режиме онлайн через DASH или Smooth с использованием PlayReady в качестве DRM.

## <a name="additional-notes"></a>Дополнительные замечания

* Widevine — это служба, которая предоставляется компанией Google Inc. и подпадает под условия предоставления услуг и политику конфиденциальности Google Inc.

## <a name="next-steps"></a>Дальнейшие действия

[Гибридная структура подсистем DRM](hybrid-design-drm-sybsystem.md)
