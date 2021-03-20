---
title: Концентраторы событий Azure — клиентские пакеты SDK | Документация Майкрософт
description: В этой статье содержатся сведения о клиентских пакетах SDK для концентраторов событий Azure.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 9f4eec603245f1e4ea6fa4d97b843ca6a770f2ad
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88930882"
---
# <a name="azure-event-hubs---client-sdks"></a>Концентраторы событий Azure — клиентские пакеты SDK
В этой статье содержатся следующие сведения о пакетах SDK, поддерживаемых концентраторами событий Azure: 

- Расположение пакета, который можно использовать в приложениях 
- Расположение GitHub, где можно найти исходный код, примеры, файл сведений, журнал изменений, сообщаемые проблемы, а также вызвать новые проблемы 
- Ссылки на краткие руководства 

## <a name="client-sdks"></a>Клиентские пакеты SDK
В следующей таблице описаны все доступные в настоящее время клиенты среды выполнения концентраторов событий Azure. Хотя некоторые из этих библиотек включают в себя ограниченные функции управления, также существуют специальные библиотеки, предназначенные для операций управления. Основное внимание этих библиотек заключается в том, чтобы **отправлять и получать сообщения** из концентратора событий.

| Язык | Пакет | Справочник | 
| -------- | ------- | --------------- | 
| . NET Standard (**Последняя версия** и поддерживает .NET Core и платформа .NET Framework) | [Azure. Messaging. EventHubs](https://www.nuget.org/packages/Azure.Messaging.EventHubs/) |<ul><li>[Расположение GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Azure.Messaging.EventHubs)</li><li>[Руководство](event-hubs-dotnet-standard-getstarted-send.md)</li></ul> |
|       | [Azure. Messaging. EventHubs. Processor](https://www.nuget.org/packages/Azure.Messaging.EventHubs.Processor/) | <ul><li>[Расположение GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Azure.Messaging.EventHubs.Processor)</li><li>[Руководство](event-hubs-dotnet-standard-getstarted-send.md)</li></ul> |
| . NET Standard (**устаревший** и поддерживает .NET Core и платформа .NET Framework) | [Microsoft.Azure.EventHubs](https://www.nuget.org/packages/Microsoft.Azure.EventHubs/) | <ul><li>[Расположение GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Microsoft.Azure.EventHubs)</li><li>[Руководство](event-hubs-dotnet-standard-getstarted-send.md)</li></ul> | 
|       | [Microsoft.Azure.EventHubs.Processor](https://www.nuget.org/packages/Microsoft.Azure.EventHubs.Processor) | <ul><li>[Расположение GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Microsoft.Azure.EventHubs.Processor)</li><li>[Руководство](event-hubs-dotnet-standard-getstarted-send.md)</li></ul> |
| . NET Framework (**старая** версия) | [WindowsAzure.Messaging](https://www.nuget.org/packages/WindowsAzure.ServiceBus/) |<ul><li>[Руководство](event-hubs-dotnet-framework-getstarted-send.md)</li></ul> |
| Java | [azure-messaging-eventhubs](https://search.maven.org/search?q=a:azure-messaging-eventhubs) | <ul><li>[Расположение GitHub](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/eventhubs/azure-messaging-eventhubs)</li><li>[Руководство](event-hubs-java-get-started-send.md)</li></ul> |
|      | [Azure-eventhubs](https://search.maven.org/search?q=a:azure-eventhubs) **(устаревшая)** | <ul><li>[Расположение GitHub](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/eventhubs/microsoft-azure-eventhubs)</li><li>[Руководство](event-hubs-java-get-started-send.md)</li></ul> |
| Python |  [azure-eventhub](https://pypi.org/project/azure-eventhub/) | <ul><li>[Расположение GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/eventhub/azure-eventhub)</li><li>[Руководство](event-hubs-python-get-started-send.md)</li></ul> |
|        | [azure-eventhub-checkpointstoreblob-aio](https://pypi.org/project/azure-eventhub-checkpointstoreblob-aio/) | <ul><li>[Расположение GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/eventhub/azure-eventhub-checkpointstoreblob-aio)</li><li>[Руководство](event-hubs-python-get-started-send.md)</li></ul> |
| JavaScript | [Azure и концентраторы событий](https://www.npmjs.com/package/@azure/event-hubs) | <ul><li>[Расположение GitHub](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/eventhub/event-hubs)</li><li>[Руководство](event-hubs-node-get-started-send.md)</li></ul> |
|            | [Azure/eventhubs-чеккпоинтсторе-BLOB](https://www.npmjs.com/package/@azure/eventhubs-checkpointstore-blob) | <ul><li>[Расположение GitHub](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/eventhub/eventhubs-checkpointstore-blob)</li><li>[Руководство](event-hubs-node-get-started-send.md)</li></ul> |
| Go | [Azure-Event-Hubs-Go](https://github.com/Azure/azure-event-hubs-go) | <ul><li>[Расположение GitHub](https://github.com/Azure/azure-event-hubs-go)</li><li>[Руководство](event-hubs-go-get-started-send.md)</li></ul> |
| C | [Azure-Event-Hubs-c](https://github.com/Azure/azure-event-hubs-c) | <ul><li>[Расположение GitHub](https://github.com/Azure/azure-event-hubs-c)</li><li>[Руководство](event-hubs-c-getstarted-send.md)</li></ul> |

## <a name="management-sdks"></a>Пакеты SDK для управления
В приведенной ниже таблице перечислены все доступные библиотеки, предназначенные для операций управления. Ни одна из этих библиотек не содержит операций среды выполнения и предназначена исключительно для **управления сущностями концентраторов событий**.

| Язык | Пакет | Справочник | 
| -------- | ------- | --------------- | 
| .NET Standard | [Microsoft.Azure.Management.EventHub](https://www.nuget.org/packages/Microsoft.Azure.Management.EventHub) |<ul><li>[Расположение GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Microsoft.Azure.Management.EventHub)</li><li>[Руководство](event-hubs-dotnet-standard-getstarted-send.md)</li></ul> |


## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Центрах событий см. в следующих источниках:

* [Общие сведения о Центрах событий](./event-hubs-about.md)
* [Создание концентратора событий](event-hubs-create.md)
* [Часто задаваемые вопросы о Центрах событий](event-hubs-faq.md)
