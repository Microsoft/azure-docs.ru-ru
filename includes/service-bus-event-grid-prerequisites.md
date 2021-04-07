---
title: включить файл
description: включить файл
services: service-bus-messaging
author: spelluru
ms.service: service-bus-messaging
ms.topic: include
ms.date: 10/15/2020
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: d12df7197945a514ed8d3d0dca77271fb4bd0903
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96509446"
---
## <a name="prerequisites"></a>Предварительные требования
Если у вас еще нет [подписки Azure](../articles/guides/developer/azure-developer-guide.md#understanding-accounts-subscriptions-and-billing), создайте [бесплатную учетную запись Azure](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio), прежде чем начать работу.

## <a name="create-a-service-bus-namespace"></a>Создание пространства имен служебной шины
Выполните инструкции, описанные здесь: [Краткое руководство. Создание раздела служебной шины и подписок на него с помощью портала Azure](../articles/service-bus-messaging/service-bus-quickstart-topics-subscriptions-portal.md). Так вы сделаете следующее:

- Создадите пространство имен служебной шины уровня **Премиум**. 
- Получите строку подключения. 
- Создадите раздел служебной шины.
- Создание подписки на раздел. В этом учебнике вам понадобится только одна подписка, поэтому создавать подписки S2 и S3 не нужно. 

## <a name="send-messages-to-the-service-bus-topic"></a>Отправка сообщений в раздел служебной шины
На этом шаге вы используете пример приложения для отправки сообщений в раздел служебной шины, созданный на предыдущем шаге. 

1. Клонируйте [репозиторий azure-service-bus GitHub](https://github.com/Azure/azure-service-bus/).
2. В Visual Studio перейдите к папке *\samples\DotNet\Azure.Messaging.ServiceBus\ServiceBusEventGridIntegration* и откройте файл *SBEventGridIntegration.sln*.
3. В окне "Обозреватель решений" разверните проект **MessageSender** и выберите **Program.cs**.
4. Замените `<SERVICE BUS NAMESPACE - CONNECTION STRING>` строкой подключения к пространству имен служебной шины и `<TOPIC NAME>` именем раздела. 

    ```csharp
    const string ServiceBusConnectionString = "<SERVICE BUS NAMESPACE - CONNECTION STRING>";
    const string TopicName = "<TOPIC NAME>";
    ```
5. Создайте и запустите программу для отправки 5 тестовых сообщений (`const int numberOfMessages = 5;`) в раздел служебной шины. 

    :::image type="content" source="./media/service-bus-event-grid-prerequisites/console-app-output.png" alt-text="Выходные данные приложения в консоли":::
