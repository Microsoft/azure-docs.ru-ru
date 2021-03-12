---
title: Пошаговое руководство скрытия лиц с помощью аналитики мультимедиа Azure | Документация Майкрософт
description: В этом разделе приведены пошаговые инструкции по выполнению полного рабочего процесса скрытия с помощью Azure Media Services Explorer (AMSE) и Azure Media Redactor Visualizer (инструмент с открытым кодом).
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: d6fa21b8-d80a-41b7-80c1-ff1761bc68f2
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 03/10/2021
ms.author: ril
ms.reviewer: juliako
ms.openlocfilehash: 1a7e20681dfa7da7ce30f46a7c4b0b6df6f78916
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103009569"
---
# <a name="redact-faces-with-azure-media-analytics-walkthrough"></a>Пошаговое руководство скрытия лиц с помощью аналитики мультимедиа Azure

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

## <a name="overview"></a>Обзор

**Редактор мультимедиа Azure** — это обработчик [медиа-аналитики Azure](./legacy-components.md) с возможностью масштабируемого скрытия лиц в облаке. Функция скрытия лиц позволяет изменять видео, размывая изображения лиц выбранных пользователей. Вы можете использовать функцию скрытия лиц в ситуациях, требующих соблюдения общественной безопасности, а также при работе с новостями. Редактирование короткого материала с несколькими лицами вручную может занять несколько часов, тогда как при использовании функции скрытия лиц достаточно выполнить несколько простых действий. Дополнительные сведения см. в [этом](https://azure.microsoft.com/blog/azure-media-redactor/) блоге.

Подробные сведения о **Azure Media Redactor** см. в разделе [Скрытие лиц с помощью медиа-аналитики Azure](media-services-face-redaction.md).

В этом разделе приведены пошаговые инструкции по выполнению полного рабочего процесса скрытия с помощью Azure Media Services Explorer (AMSE) и Azure Media Redactor Visualizer (инструмент с открытым кодом).

Дополнительные сведения см. в [этом](https://azure.microsoft.com/blog/redaction-preview-available-globally) блоге.

## <a name="azure-media-services-explorer-workflow"></a>Рабочий процесс Azure Media Services Explorer

Проще всего приступить к работе с Redactor можно с помощью инструмента с открытым кодом AMSE на сайте GitHub. Можно выполнить упрощенный рабочий процесс в **комбинированном** режиме, если не требуется доступ к аннотациям в формате JSON или изображениям лиц в формате JPG.

### <a name="download-and-setup"></a>Скачивание и установка

1. Скачайте средство AMSE для AMS v2 [отсюда.](https://aka.ms/amseforv2)
1. Войдите в свою учетную запись служб мультимедиа с помощью ключа службы.

    Чтобы получить имя учетной записи и сведения о ключе, перейдите на [портал Azure](https://portal.azure.com/) и выберите свою учетную запись AMS. Последовательно выберите "Параметры" > "Ключи". В окне "Управление ключами" отображается имя учетной записи, а также первичный и вторичный ключи. Скопируйте значения имени учетной записи и первичного ключа.

![На снимке экрана показаны Cлужбы мультимедиа Microsoft Azure, где можно ввести имя и ключ учетной записи.](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough001.png)

### <a name="first-pass--analyze-mode"></a>Первый цикл — режим анализа

1. Передайте файл мультимедиа, щелкнув "Ресурс-контейнер" > "Отправить" или перетащив его. 
1. Щелкните правой кнопкой мыши файла мультимедиа и обработайте его, выбрав "Media Analytics" (Медиа-аналитика) > "Azure Media Redactor" > "Analyze mode" (Режим анализа). 


![На снимке экрана показано меню с Azure Media Redactorами процессов.](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough002.png)

![На снимке экрана показан Azure Media Redactor с первым проходом: режим анализа выбран.](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough003.png)

Выходные данные будут содержать JSON-файл аннотаций с данными расположения лиц и JPG-файл каждого обнаруженного лица. 

![На снимке экрана показаны выходные данные анализа.](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough004.png)

### <a name="second-pass--redact-mode"></a>Второй цикл — режим скрытия

1. Передайте первоначальный ресурс видео в выходные данные из первого цикла и укажите в качестве основного ресурса. 

    ![На снимке экрана показаны основные кнопки отправки и установки.](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough005.png)

2. (Необязательно.) Передайте файл Dance_idlist.txt, содержащий разделенный символами новой строки список идентификаторов, который нужно скрыть. 

    ![На снимке экрана показан параметр для отправки текстового файла.](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough006.png)

3. (Необязательно.) Внесите необходимые изменения в файл annotations.json, например, увеличьте ограничивающую рамку. 
4. Щелкните правой кнопкой мыши выходной ресурс из первого цикла, выберите "Redactor" и запустите Azure Media Redactor в режиме **Redact** (Скрытие). 

    ![Снимок экрана показывает Azure Media Redactor со вторым проходом: режим исправления выбран.](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough007.png)

5. Скачайте окончательный отредактированный выходной ресурс или поделитесь им. 

    ![На снимке экрана показана кнопка загрузки.](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough008.png)

## <a name="azure-media-redactor-visualizer-open-source-tool"></a>Инструмент с открытым кодом Azure Media Redactor Visualizer

[Инструмент визуализации](https://github.com/Microsoft/azure-media-redactor-visualizer) с открытым кодом предназначен помочь разработчикам, только приступающим к работе с форматом аннотаций, выполнять анализ и использовать выходные данные.

После клонирования репозитория, чтобы запустить проект, необходимо скачать компонент FFMPEG с [официального сайта](https://ffmpeg.org/download.html) его поставщика.

Если вы являетесь разработчиком, пытающимся анализировать данные аннотаций в формате JSON, то вы можете ознакомиться с примерами кода в Models.MetaData.

### <a name="set-up-the-tool"></a>Настройка инструмента

1.  Скачайте полное решение и выполните его сборку. 

    ![На снимке экрана показано решение сборки, выбранное в меню.](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough009.png)

2.  Скачайте компонент FFMPEG [отсюда](https://ffmpeg.org/download.html). Данный проект первоначально был разработан с использованием версии be1d324 (2016-10-04) со статическим связыванием. 
3.  Скопируйте файлы ffmpeg.exe и ffprobe.exe в ту же папку выходных данных, что и AzureMediaRedactor.exe. 

    ![На снимке экрана показано содержимое папки, включая FFmpeg и ффпробе.](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough010.png)

4. Запустите AzureMediaRedactor.exe. 

### <a name="use-the-tool"></a>Работа со средством

1. Обработайте видео в учетной записи служб мультимедиа Azure с помощью Redactor MP в режиме анализа. 
2. Скачайте первоначальный видеофайл и выходные данные задания Redaction - Analyze. 
3. Запустите приложение визуализатора и выберите указанные выше файлы. 

    ![На снимке экрана показано, Azure Media Redactor отправкой файлов.](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough011.png)

4. Воспользуйтесь предварительным просмотром файла. Выберите лица, которые следует размыть, с помощью боковой панели справа. 
    
    ![Снимок экрана показывает Azure Media Redactor, где можно просмотреть и выбрать грани для размытия.](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough012.png)

5.  В текстовом поле внизу появятся идентификаторы лиц. Создайте файл idlist.txt, содержащий эти идентификаторы в виде списка, разделенного символами новой строки. 

    >[!NOTE]
    > Файл idlist.txt необходимо сохранить в формате ANSI. Для сохранения в формате ANSI можно использовать Блокнот.
    
6.  Передайте этот файл в выходной ресурс из шага 1. Передайте первоначальное видео в этот ресурс и укажите его в качестве основного ресурса. 
7.  Выполните задание Redaction для этого ресурса в режиме "Redact" (Скрытие), чтобы получить окончательное отредактированное видео. 

## <a name="next-steps"></a>Дальнейшие действия 

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="related-links"></a>Связанные ссылки
[Общие сведения об аналитике служб мультимедиа Azure](./legacy-components.md)

[Демонстрационные материалы для медиааналитики Azure](http://amslabs.azurewebsites.net/demos/Analytics.html)

[Announcing Face Redaction for Azure Media Analytics](https://azure.microsoft.com/blog/azure-media-redactor/) (Анонс функции скрытия лиц с помощью медиа-аналитики Azure)