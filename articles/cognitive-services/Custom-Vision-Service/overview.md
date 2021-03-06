---
title: Что собой представляет Пользовательское визуальное распознавание
titleSuffix: Azure Cognitive Services
description: Узнайте, как использовать службу "Пользовательское визуальное распознавание Azure" для создания пользовательских ИИ классификаторов изображений и средств обнаружения объектов в облаке Azure.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: overview
ms.date: 12/15/2020
ms.author: pafarley
ms.custom: cog-serv-seo-aug-2020
keywords: распознавание изображений, идентификатор изображения, приложение для распознавания изображений, пользовательское визуальное распознавание
ms.openlocfilehash: 12877f2d43f9b8f864871e5a5ab050aa0eeb61e2
ms.sourcegitcommit: 1140ff2b0424633e6e10797f6654359947038b8d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/30/2020
ms.locfileid: "97814607"
---
# <a name="what-is-custom-vision"></a>Что собой представляет Пользовательское визуальное распознавание

[!INCLUDE [TLS 1.2 enforcement](../../../includes/cognitive-services-tls-announcement.md)]

Пользовательское визуальное распознавание Azure — это когнитивная служба распознавания изображений, которая позволяет создавать, развертывать и улучшать пользовательские идентификаторы изображений. Идентификатор изображений присваивает изображениям метки (соответствующие классам и объектам) по их визуальным характеристикам. В отличие от службы [Компьютерное зрение](../computer-vision/overview.md), служба "Пользовательское визуальное распознавание" позволяет указывать метки и обучать пользовательские модели для их обнаружения.

## <a name="what-it-does"></a>Действие

Служба "Пользовательское визуальное распознавание" использует алгоритм машинного обучения для анализа изображений. Вам (разработчику) необходимо отправить группы изображений, которые содержат и не содержат необходимые характеристики. При этом вы сами присваиваете метки перед отправкой. Затем алгоритм обучается по этим данным и вычисляет собственную точность, проводя тесты на тех же изображениях. После обучения алгоритма вы можете проверить его, повторно обучить и, в конечном итоге, использовать в приложении распознавания изображений для классификации новых изображений. Вы также можете экспортировать саму модель для автономного использования.

### <a name="classification-and-object-detection"></a>Классификация и обнаружение объектов

Функции Пользовательской службы визуального распознавания можно разделить на две части. **Классификация изображений** присваивает изображению одну или несколько меток. **Обнаружение объектов** действует похоже, но также возвращает координаты в изображении, где можно найти присвоенную метку.

### <a name="optimization"></a>Optimization

Служба "Пользовательское визуальное распознавание" оптимизирована для быстрого распознавания основных отличий в изображениях, благодаря чему вы можете создать прототип своей модели на основе небольшого объема данных. Обычно хорошим началом является 50 изображений на метку. Однако это означает, что служба не подходит для обнаружения незначительных отличий в изображениях (например, мелкие трещины или сколы при контроле качества).

Кроме того, вы можете выбрать из различных алгоритмов службы "Пользовательское визуальное распознавание", которые оптимизированы для изображений с определенными объектами, например достопримечательностями или магазинами. Подробные сведения см. в статье [о создании классификатора](getting-started-build-a-classifier.md) или [средства обнаружения объектов](get-started-build-detector.md).

## <a name="what-it-includes"></a>Компоненты

Пользовательская служба визуального распознавания доступна в виде набора собственных пакетов SDK, а также через веб-интерфейс на [ее веб-сайте](https://customvision.ai/). Вы можете создать, тестировать и обучать модель через интерфейс или использовать оба варианта.

![Веб-сайт службы "Пользовательское визуальное распознавание" в окне браузера Chrome](media/browser-home.png)

## <a name="data-privacy-and-security"></a>Конфиденциальность и безопасность данных

Как и в случае со всеми службами Cognitive Services, разработчикам, использующим Пользовательскую службу визуального распознавания, следует учитывать политику корпорации Майкрософт касательно данных клиента. Дополнительные сведения см. на [странице о Cognitive Services](https://www.microsoft.com/trustcenter/cloudservices/cognitiveservices) Центра управления безопасностью Майкрософт.

## <a name="next-steps"></a>Дальнейшие действия

Перейдите к руководству по [созданию классификатора](getting-started-build-a-classifier.md), чтобы начать использовать службу "Пользовательское визуальное распознавание" на веб-портале, или изучите [краткое руководство](quickstarts/image-classification.md), чтобы реализовать основной сценарий в коде.