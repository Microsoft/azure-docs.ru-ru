---
author: baanders
description: включить файл для ресурсов, необходимых для создания конечных точек цифровых двойников Azure
ms.service: digital-twins
ms.topic: include
ms.date: 1/26/2021
ms.author: baanders
ms.openlocfilehash: 58c90bae3dea0f3a47489ea7d8de6a79f823dcab
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99054519"
---
### <a name="prerequisite-create-endpoint-resources"></a>Предварительные требования: создание ресурсов конечной точки

Чтобы связать конечную точку с цифровым двойников Azure, раздел сетки событий, концентратор событий или раздел служебной шины, который вы используете для этой конечной точки, должен уже существовать.

Используйте следующую диаграмму, чтобы узнать, какие ресурсы должны быть настроены перед созданием конечной точки.

| Тип конечной точки | Необходимые ресурсы (связанные с инструкциями по созданию) |
| --- | --- |
| Конечная точка сетки событий | [раздел "Сетка событий"](../articles/event-grid/custom-event-quickstart-portal.md#create-a-custom-topic) |
| Конечная точка концентраторов событий | [&nbsp;Пространство имен концентраторов событий &nbsp;](../articles/event-hubs/event-hubs-create.md)<br/><br/>[концентратор событий](../articles/event-hubs/event-hubs-create.md)<br/><br/>Используемых [правило авторизации](../articles/event-hubs/authorize-access-shared-access-signature.md) для проверки подлинности на основе ключей | 
| Конечная точка Service Bus | [Пространство имен служебной шины](../articles/service-bus-messaging/service-bus-quickstart-topics-subscriptions-portal.md)<br/><br/>[Раздел служебной шины](../articles/service-bus-messaging/service-bus-quickstart-topics-subscriptions-portal.md)<br/><br/> Используемых [правило авторизации](../articles/service-bus-messaging/service-bus-authentication-and-authorization.md#shared-access-signature) для проверки подлинности на основе ключей|
