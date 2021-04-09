---
title: Teams Embed. Инфраструктура пользовательского интерфейса
titleSuffix: An Azure Communication Services quickstart
description: Из этого документа вы узнаете, как использовать возможность Teams Embed в инфраструктуре пользовательского интерфейса в Службах коммуникации Azure для создания готовых интерфейсов взаимодействия.
author: tophpalmer
ms.author: chpalm
ms.date: 11/16/2020
ms.topic: quickstart
ms.service: azure-communication-services
ms.openlocfilehash: e55cfb1a4dff7bfda2323e68777d6f50514b1608
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105930240"
---
# <a name="teams-embed"></a>Teams Embed

[!INCLUDE [Public Preview Notice](../../includes/private-preview-include.md)]


Возможность Teams Embed, предоставляемая в Службах коммуникации Azure, позволяет создавать типичные взаимодействия для звонков между клиентами и партнерами. В основном система Teams Embed нацелена на [голосовые звонки и видеовызовы](../voice-video-calling/calling-sdk-features.md), но также она позволяет применить примитивы вызовов Azure для создания полнофункционального взаимодействия с пользователем на основе собраний в Microsoft Teams.

Пакеты SDK Teams Embed используют закрытый исходный код и предоставляют свои возможности в формате готового комплексного решения. Вам нужно лишь поместить Teams Embed на холст приложения, а пакет SDK самостоятельно обеспечит полное взаимодействие с пользователем. Это взаимодействие будет очень похоже на обычные собрания в Microsoft Teams, а значит вы получите следующие возможности:

- Сокращение времени разработки и сложности проектирования.
- Опыт работы пользователей с Teams.
- Повторное применение [содержимого Teams для обучения пользователей](https://support.microsoft.com/office/meetings-in-teams-e0b0ae21-53ee-4462-a50d-ca9b9e217b67).

Teams Embed предоставляет большинство функций, поддерживаемых в собраниях Teams, в том числе следующие:

- Взаимодействие перед собранием, благодаря которому пользователь может настроить аудио- и видеоустройства.
- Взаимодействие во время собрания для настройки аудио- и видеоустройств.
- [Фоны видео](https://support.microsoft.com/office/change-your-background-for-a-teams-meeting-f77a2381-443a-499d-825e-509a140f4780): позволяет участникам размыть фон или заменить произвольным изображением.
- [Несколько режимов работы видеогалереи](https://support.microsoft.com/office/using-video-in-microsoft-teams-3647fc29-7b92-4c26-8c2d-8a596904cdae): крупная галерея, совместный режим, фокусировка, закрепление и выделение.
- [Совместное использование содержимого](https://support.microsoft.comoffice/share-content-in-a-meeting-in-teams-fcc2bf59-aecd-4481-8f99-ce55dd836ce8#ID0EABAAA=Mobile): участники могут показать другим свой экран.

Дополнительные сведения об этом пользовательском интерфейсе и его сравнение с другими пакетами SDK Служб коммуникации Azure см. в статье [о концепции пользовательского интерфейса SDK](ui-sdk-overview.md). 
