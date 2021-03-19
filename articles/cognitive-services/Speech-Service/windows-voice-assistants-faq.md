---
title: Речевые помощники в Windows — часто задаваемые вопросы
titleSuffix: Azure Cognitive Services
description: Часто встречающиеся вопросы, которые часто возникают при разработке агента Windows Voice.
services: cognitive-services
author: cfogg6
manager: trrwilson
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/15/2020
ms.author: travisw
ms.openlocfilehash: 3cb4dd16592f3af962b32fe3005d2ad4b241d831
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "84457020"
---
# <a name="samples-and-faqs"></a>Примеры и часто задаваемые вопросы

## <a name="the-uwp-voice-assistant-sample"></a>Пример речевого помощника UWP

Образец "помощник по UWP" является голосовым помощником в Windows, который служит для

- Демонстрация использования API-интерфейсов Windows Конверсатионалажент
- Отображение рекомендаций для агента голоса в Windows
- предоставление адаптируемого приложения и многократно используемых компонентов для приложения агента MVP
- Покажите, как с помощью API-интерфейсов Windows Конверсатионалажент можно использовать прямую линейную речь, а также платформу Bot или пользовательские команды для полноценного агента голосовой связи.

Документация, предоставленная в примере приложения, проходит через путь кода для активации голоса и управления агентами, начиная с предварительных требований к запуску с помощью правильной очистки.

> [!div class="nextstepaction"]
> [Посетите репозиторий GitHub для примера UWP](https://aka.ms/MVA/sample)

## <a name="frequently-asked-questions"></a>Часто задаваемые вопросы

### <a name="how-do-i-contact-microsoft-for-resources-like-limited-access-feature-tokens-and-keyword-model-files"></a>Разделы справки обратиться в корпорацию Майкрософт для получения таких ресурсов, как маркеры функций ограниченного доступа и файлы модели ключевых слов?

Обратитесь winvoiceassistants@microsoft.com к, чтобы запросить эти ресурсы.

### <a name="my-app-is-showing-in-a-small-window-when-i-activate-it-by-voice-how-can-i-transition-from-the-compact-view-to-a-full-application-window"></a>Мое приложение отображается в маленьком окне при активации с помощью голоса. Как перейти из компактного представления в полное окно приложения?

Когда приложение сначала активируется голосовым методом, оно запускается в компактном представлении. Ознакомьтесь с [руководством по проектированию для предварительной версии активации голоса](windows-voice-assistants-best-practices.md#design-guidance-for-voice-activation-preview) , чтобы получить рекомендации по различным представлениям и переходам между ними для речевых помощников в Windows.

Чтобы перевести переход с Compact представления на полное представление приложения, используйте API Аппвиев `TryEnterViewModeAsync` :

`var appView = ApplicationView.GetForCurrentView();
 await appView.TryEnterViewModeAsync(ApplicationViewMode.Default);`

### <a name="why-are-voice-assistant-features-on-windows-only-enabled-for-uwp-applications"></a>Почему функции голосового помощника в Windows включены только для приложений UWP?

Учитывая риски конфиденциальности, связанные с такими функциями, как активация голоса, необходимы функции платформы UWP, позволяющие обеспечить достаточную безопасность функций речевого помощника в Windows.

### <a name="the-uwp-voice-assistant-sample-uses-direct-line-speech-do-i-have-to-use-direct-line-speech-for-my-voice-assistant-on-windows"></a>В примере голосового помощника UWP используется прямая речь. Нужно ли использовать прямой голосовый голос для моего помощника по работе с Windows?

Пример приложения UWP был разработан с помощью прямой голосовой речи и пакета SDK речевых служб в качестве демонстрации использования службы диалоговых окон с возможностью агента Windows CONVERSATION. Однако можно использовать любую службу для локальной и облачной проверки ключевых слов, преобразования речи в текст, диалогового окна Bot и преобразования текста в речь.