---
title: Создание рабочих процессов расширенной кодировки с помощью конструктора рабочих процессов | Документация Майкрософт
description: Сведения о создании рабочих процессов с расширенной кодировкой с помощью конструктора рабочих процессов.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: 004815f2-0761-4706-87a1-675ba36e0322
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 3/10/2021
ms.author: anilmur
ms.reviewer: juliako;johndeu
ms.openlocfilehash: 744d302afd21e6521fb17bf7bef6e68d21fb8372
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105639396"
---
# <a name="create-advanced-encoding-workflows-with-workflow-designer"></a>Создание рабочих процессов расширенного кодирования с помощью конструктора рабочих процессов

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

## <a name="overview"></a>Обзор
**Конструктор рабочих процессов** — это классическое средство Windows, которое используется для разработки и создания настраиваемых рабочих процессов шифрования с использованием **рабочего процесса Media Encoder Premium**.
Возможности конструктора рабочих процессов позволяют проектировать и создавать сложные рабочие процессы, которые будут выполняться в **Media Encoder Premium**.  

Рабочие процессы могут включать пользовательские алгоритмы принятия решений и ветвление на основе свойств файла исходных данных. Создание рабочих процессов с перезаписываемыми свойствами и динамическими значениями позволяет упростить повторное выполнение и настройку в облаке даже для самых сложных задач кодирования.

В число рабочих процессов, которые можно создавать, входят следующие:

* Рабочие процессы на основе решений, проверяющие исходный контент на наличие решения и кодирующие только определенные выходные дорожки.  Это позволяет избежать возникновения лишних дорожек, образуемых при масштабировании без дополнительных настроек.
* Для поддержки подписей, наложений и совмещения контента можно использовать несколько файлов исходных данных. 

Кроме того, это средство позволяет также изменить любой из [опубликованных рабочих процессов](media-services-workflow-designer.md#existing_workflows). 

> [!NOTE]
> Чтобы получить конструктор рабочих процессов, обратитесь по адресу mepd@microsoft.com.

После создания файла рабочего процесса его можно передать как ресурс, а затем использовать для кодирования файлов мультимедиа. Сведения о том, как выполнять кодирование, используя **рабочий процесс Media Encoder Premium** и **.NET** см. в разделе [Дополнительное кодирование с помощью рабочего процесса Premium кодировщика мультимедиа](media-services-encode-with-premium-workflow.md).

## <a name="modify-existing-workflows"></a><a id="existing_workflows"></a>Изменение существующих рабочих процессов
[Опубликованные рабочие процессы](media-services-workflow-designer.md#existing_workflows) по умолчанию можно редактировать с помощью средств конструктора. Файлы используемого по умолчанию рабочего процесса можно получить [здесь](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/media-services/previous/media-services-encode-with-premium-workflow.md). Папка также содержит описание этих файлов.

Указанные ниже видеоматериалы демонстрируют, как использовать конструктор.

### <a name="day-1--getting-started"></a>День 1. Приступая к работе
Темы видеоматериала первого дня

* Общие сведения о конструкторе
* Простые рабочие процессы — "Hello World"
* Создание нескольких выходных MP4-файлов для использования в потоковой передаче мультимедиа Azure

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Azure-Premium-Encoder-Workflow-Designer-Training-Videos-Day-1/player]
> 
> 

### <a name="day-2"></a>День 2
Темы видеоматериала второго дня

* Изменение сценариев исходных файлов — обработка аудио
* Рабочие процессы с расширенной логикой
* Графические схемы этапов

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Azure-Premium-Encoder-Workflow-Designer-Training-Videos-Day-2/player]
> 
> 

### <a name="day-3"></a>День 3
Темы видеоматериала третьего дня

* Составление сценариев внутри рабочих процессов или проектов
* Ограничения текущего кодировщика
* Вопросы и ответы

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Azure-Premium-Encoder-Workflow-Designer-Training-Videos-Day-3/player]
> 
> 

## <a name="need-help"></a>Требуется помощь?

Вы можете открыть запрос в службу поддержки, перейдя к разделу [нового запроса на техническую поддержку](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest)

## <a name="next-step"></a>Следующий шаг
Просмотрите схемы обучения работе со службами мультимедиа.

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="see-also"></a>См. также:
[Учебные видеоматериалы по расширенному рабочему процессу кодировщика служб мультимедиа Azure](http://johndeutscher.com/2015/07/06/azure-premium-encoder-workflow-designer-training-videos/)

