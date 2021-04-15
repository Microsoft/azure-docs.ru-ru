---
title: Часто задаваемые вопросы о Службе Azure SignalR
description: Получите ответы на часто задаваемые вопросы о Службе Azure SignalR, включая сведения об устранении неполадок и типичных сценариях использования.
author: sffamily
ms.service: signalr
ms.topic: overview
ms.custom: devx-track-dotnet
ms.date: 11/13/2019
ms.author: zhshang
ms.openlocfilehash: d3b84756f390930be5124c6bda54d07078d29053
ms.sourcegitcommit: d23602c57d797fb89a470288fcf94c63546b1314
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/01/2021
ms.locfileid: "106166927"
---
# <a name="azure-signalr-service-faq"></a>Вопросы и ответы о Службе SignalR Azure

## <a name="is-azure-signalr-service-ready-for-production-use"></a>Готова ли Служба Azure SignalR для использования в рабочей среде?

Да, поддержка SignalR для [ASP.NET Core](https://dotnet.microsoft.com/apps/aspnet/signalr) и [ASP.NET](/aspnet/signalr/overview/getting-started/introduction-to-signalr) является общедоступной.

## <a name="when-there-are-multiple-application-servers-are-client-messages-sent-to-all-servers-or-just-one-of-them"></a>При наличии нескольких серверов приложений клиентские сообщения отправляются на все серверы или только на один из них?

Между клиентом и сервером приложений происходит сопоставление "один к одному". Сообщения от одного клиента всегда отправляются на один и тот же сервер приложений.

Указанное сопоставление поддерживается, пока клиент или сервер не отключатся.

## <a name="if-one-of-my-application-servers-is-down-how-can-i-find-it-and-get-notified"></a>Если один из моих серверов приложений отключается, как узнать об этом и получить уведомление?

Служба Azure SignalR отслеживает пакеты пульса серверов приложений.
Если пакеты пульса не получены в течение заданного времени, сервер приложений считается отключенным. Все подключения клиентов, которые сопоставляются с этим сервером приложений, будут прерваны.

## <a name="why-does-my-custom-iuseridprovider-throw-an-exception-when-im-switching-from-aspnet-core-signalr-sdk-to-azure-signalr-service-sdk"></a>Почему мой пользовательский `IUserIdProvider` создает исключение, если я переключаюсь с пакета SDK ASP.NET Core Службы SignalR на пакет SDK Службы Azure SignalR?

Параметр `HubConnectionContext context` отличается для пакета SDK ASP.NET Core Службы SignalR и пакета SDK Службы Azure SignalR при вызове `IUserIdProvider`.

В ASP.NET Core `HubConnectionContext context` — это контекст физического клиентского подключения с допустимыми значениями для всех свойств.

В пакете SDK Службы Azure SignalR `HubConnectionContext context` — это контекст из логического подключения клиента. Физический клиент подключен к экземпляру Службы Azure SignalR, поэтому предоставляется только ограниченное число свойств.

На данный момент доступны только `HubConnectionContext.GetHttpContext()` и `HubConnectionContext.User`.
Вы можете [проверить исходный код](https://github.com/Azure/azure-signalr/blob/dev/src/Microsoft.Azure.SignalR/HubHost/ServiceHubConnectionContext.cs).

## <a name="can-i-configure-the-transports-available-in-azure-signalr-service-on-the-server-side-with-aspnet-core-signalr-for-example-can-i-disable-websocket-transport"></a>Можно ли настроить транспортировки, доступные в Службе Azure SignalR, на стороне сервера с помощью ASP.NET Core SignalR? Например, можно ли отключить транспортировку WebSocket?

Нет.

Служба Azure SignalR предоставляет все три типа транспортировки, которые поддерживает ASP.NET Core SignalR по умолчанию. Этот параметр не настраивается. Служба Azure SignalR будет обрабатывать подключения и транспортировки для всех клиентских подключений.

Настройка транспортировок на стороне клиента описана в статье [Конфигурация SignalR для ASP.NET Core](/aspnet/core/signalr/configuration#configure-allowed-transports-1).

## <a name="what-is-the-meaning-of-metrics-like-message-count-or-connection-count-shown-in-the-azure-portal-which-kind-of-aggregation-type-should-i-choose"></a>Что означают такие метрики, как количество сообщений или количество подключений, отображаемые на портале Azure? Какой тип агрегирования мне выбрать?

Сведения о вычислениях этих метрик можно найти в статье [Сообщения и подключения в Службе Azure SignalR](signalr-concept-messages-and-connections.md).

В области обзора ресурсов Службы Azure SignalR мы уже выбрали для вас подходящий тип агрегирования. Если вы перейдете к области "Метрики", вы можете взять тип агрегирования из статьи [Сообщения и подключения в Службе Azure SignalR](../azure-monitor/essentials/metrics-supported.md#microsoftsignalrservicesignalr) в качестве ориентира.

## <a name="what-is-the-meaning-of-the-default-serverless-and-classic-service-modes-how-can-i-choose"></a>Что означают режимы службы `Default`, `Serverless` и `Classic`? Как его выбрать?

Для новых приложений следует использовать только режим по умолчанию и бессерверный режим. Основное различие заключается в том, есть ли у вас серверы приложений, которые устанавливают серверные соединения со службой (т. е. для подключения к службе используются `AddAzureSignalR()`). Если да, используйте режим по умолчанию, в противном случае используйте бессерверный режим.

Классический режим разработан для обеспечения обратной совместимости с имеющимися приложениями, поэтому его не следует использовать для новых приложений.

Дополнительные сведения о режиме службы см. в статье [Режим службы в службе Azure SignalR](concept-service-mode.md).

## <a name="can-i-send-message-from-client-in-serverless-mode"></a>Могу ли я отправить сообщение от клиента в бессерверном режиме?

Вы можете отправить сообщение от клиента, если настроите восходящий поток в своем экземпляре SignalR. Восходящий поток — это набор конечных точек, которые могут получать сообщения и события подключения от службы SignalR. Если восходящий поток не настроен, сообщения от клиента будут игнорироваться.

Дополнительные сведения о восходящем потоке см. в документе [Параметры восходящего потока](concept-upstream.md).

Сейчас восходящий поток предоставляется в режиме общедоступной предварительной версии.

## <a name="are-there-any-feature-differences-in-using-azure-signalr-service-with-aspnet-signalr"></a>Различаются ли функции при использовании Службы Azure SignalR с ASP.NET SignalR?

При использовании Службы Azure SignalR некоторые API-интерфейсы и функции ASP.NET SignalR не поддерживаются:
- Возможность передачи произвольного состояния между клиентами и концентратором (часто называемого `HubState`) не поддерживается.
- Класс `PersistentConnection` не поддерживается.
- *Транспортировка Forever Frame* не поддерживается.
- Служба Azure SignalR больше не воспроизводит сообщения, отправленные клиенту, когда клиент находится в автономном режиме.
- Если вы используете Службу Azure SignalR, трафик для одного клиентского подключения всегда маршрутизируется (т. е. *закрепляется*) к одному экземпляру сервера приложений на протяжении всего соединения.

Поддержка ASP.NET SignalR ориентирована на совместимость, поэтому поддерживаются не все новые функции ASP.NET Core SignalR. Например, *MessagePack* и *Streaming* доступны только для приложений ASP.NET Core SignalR.

Вы можете настроить разные режимы Службы Azure SignalR: `Classic`, `Default` и `Serverless`. Режим `Serverless` не поддерживается для ASP.NET. REST API плоскости данных также не поддерживается.

## <a name="where-does-my-data-reside"></a>Где находятся мои данные?

Служба Azure SignalR функционирует как служба обработки данных. В ней не будет храниться какое-либо содержимое клиента, а место расположения данных предусмотрено дизайном. При использовании Службы Azure SignalR вместе с другими службами Azure, например службой хранилища Azure для диагностики, ознакомьтесь с инструкциями в [этом техническом документе](https://azure.microsoft.com/resources/achieving-compliant-data-residency-and-security-with-azure/) о том, как сохранить место расположения данных в регионах Azure.
