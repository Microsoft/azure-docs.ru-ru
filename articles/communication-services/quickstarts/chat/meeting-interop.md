---
title: Начало работы со Службами коммуникации Azure с использованием взаимодействия с Teams
titleSuffix: An Azure Communication Services quickstart
description: Из этого краткого руководства вы узнаете, как присоединиться к собранию в Teams с помощью клиентской библиотеки чата Служб коммуникации Azure.
author: askaur
ms.author: askaur
ms.date: 03/10/2021
ms.topic: quickstart
ms.service: azure-communication-services
ms.openlocfilehash: 75580931b2df33b02b9759b6faaa5bec548daaae
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103496186"
---
# <a name="quickstart-join-your-chat-app-to-a-teams-meeting"></a>Краткое руководство. Подключение приложения чата к собранию в Teams

> [!IMPORTANT]
> Чтобы включить или отключить [взаимодействие арендаторов в Teams](../../concepts/teams-interop.md), заполните [эту форму](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR21ouQM6BHtHiripswZoZsdURDQ5SUNQTElKR0VZU0VUU1hMOTBBMVhESS4u).

Приступите к работе со Службами коммуникации Azure, подключив решение чата к Microsoft Teams с помощью клиентской библиотеки JavaScript. 

## <a name="prerequisites"></a>Предварительные требования 

1.  [Развертывание Teams.](/deployoffice/teams-install) 
2. Рабочее [приложение чата](./get-started.md). 

## <a name="enable-teams-interoperability"></a>Обеспечение взаимодействия с Teams 

Пользователь Служб коммуникации, который присоединяется к собранию в Teams в качестве гостевого пользователя, может получить доступ к чату собрания, только когда он присоединится к вызову собрания в Teams. Дополнительные сведения о добавлении пользователя Служб коммуникации в вызов собрания в Teams см. в документации по [взаимодействию с Teams](../voice-video-calling/get-started-teams-interop.md).

Для использования этой функции пользователь должен быть членом организации-владельца обеих сущностей.

[!INCLUDE [Join Teams meetings](./includes/meeting-interop-javascript.md)]

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите очистить и удалить подписку на Службы коммуникации, вы можете удалить ресурс или группу ресурсов. При этом удаляются все ресурсы, связанные с ней. См. сведения об [очистке ресурсов](../create-communication-resource.md#clean-up-resources).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в следующих статьях:

- Ознакомьтесь с нашим [главным примером функции чата](../../samples/chat-hero-sample.md).
- См. дополнительные сведения о [принципах работы чата](../../concepts/chat/concepts.md).