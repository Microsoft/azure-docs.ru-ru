---
title: Масштабирование конечных точек потоковой передачи с помощью портала Azure | Документация Майкрософт
description: В этом учебнике пошагово описано, как масштабировать конечные точки потоковой передачи с помощью портала Azure.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: 1008b3a3-2fa1-4146-85bd-2cf43cd1e00e
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.openlocfilehash: 4db53d64131e6fb43ce425cf579d79e62775754a
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103009763"
---
# <a name="scale-streaming-endpoints-with-the-azure-portal"></a>Масштабирование конечных точек потоковой передачи с помощью портала Azure

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

## <a name="overview"></a>Обзор

> [!NOTE]
> Для работы с этим учебником требуется учетная запись Azure. Дополнительные сведения см. в разделе [Бесплатная пробная версия Azure](https://azure.microsoft.com/pricing/free-trial/). 
> 
> 

Конечные точки уровня **Премиум** подходят для выполнения более сложных задач благодаря выделенной пропускной способности и возможности масштабирования. Клиенты, имеющие конечную точку потоковой передачи уровня **Премиум**, по умолчанию получают одну единицу потоковой передачи. Конечную точку потоковой передачи можно масштабировать, добавляя единицы потоковой передачи. Каждая единица потоковой передачи предоставляет дополнительную пропускную способность для приложения. Дополнительные сведения о типах конечных точек потоковой передачи и конфигурации сети CDN см. в статье [Управление конечными точками потоковой передачи с помощью портала Azure](media-services-streaming-endpoints-overview.md).
 
В этом разделе показано, как масштабировать конечную точку потоковой передачи.

Дополнительные сведения о ценах на службы мультимедиа см. [здесь](https://azure.microsoft.com/pricing/details/media-services/).

## <a name="scale-streaming-endpoints"></a>Масштабирование конечных точек потоковой передачи

Чтобы изменить число единиц потоковой передачи, сделайте следующее.

1. Выберите учетную запись служб мультимедиа Azure на [портале Azure](https://portal.azure.com/).
2. В окне **Параметры** щелкните **Конечные точки потоковой передачи**.
3. Щелкните конечную точку потоковой передачи, которую требуется масштабировать. 

    > [!NOTE] 
    > Масштабировать можно только конечные точки потоковой передачи уровня **Премиум**.

4. Переместите ползунок, чтобы указать нужное число единиц потоковой передачи.

    ![конечная точка потоковой передачи](./media/media-services-portal-manage-streaming-endpoints/media-services-manage-streaming-endpoints3.png)

## <a name="next-steps"></a>Дальнейшие действия
Просмотрите схемы обучения работе со службами мультимедиа.

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

