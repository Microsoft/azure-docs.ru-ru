---
title: Устранение ошибок AMQP в служебной шине Azure | Документация Майкрософт
description: Содержит список ошибок AMQP, которые могут возникнуть при использовании служебной шины Azure, и причины этих ошибок.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 88b10940e0b910f50e6ccf7f8c53134fa7f0ba2f
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88064355"
---
# <a name="amqp-errors-in-azure-service-bus"></a>Ошибки AMQP в служебной шине Azure
В этой статье приведены некоторые ошибки, которые вы получаете при использовании AMQP с служебной шиной Azure. Все это стандартное поведение службы. Вы можете избежать их, сделав вызов отправки и получения для соединения или ссылки, который автоматически воссоздает соединение или связь.

## <a name="link-is-closed"></a>Ссылка закрыта 
Если подключение и ссылка AMQP активны, но нет вызовов (например, отправка или получение), то вы увидите следующую ошибку, которая не будет выполнена через ссылку в течение 10 минут. Таким образом, ссылка будет закрыта. Соединение все еще открыто.

```
amqp:link:detach-forced:The link 'G2:7223832:user.tenant0.cud_00000000000-0000-0000-0000-00000000000000' is force detached by the broker due to errors occurred in publisher(link164614). Detach origin: AmqpMessagePublisher.IdleTimerExpired: Idle timeout: 00:10:00. TrackingId:00000000000000000000000000000000000000_G2_B3, SystemTracker:mynamespace:Topic:MyTopic, Timestamp:2/16/2018 11:10:40 PM
```

## <a name="connection-is-closed"></a>Соединение закрыто
При подключении к AMQP отображается следующая ошибка, когда все ссылки в соединении закрыты из-за отсутствия активности (в режиме простоя), а новая ссылка не была создана в течение 5 минут.

```
Error{condition=amqp:connection:forced, description='The connection was inactive for more than the allowed 300000 milliseconds and is closed by container 'LinkTracker'. TrackingId:00000000000000000000000000000000000_G21, SystemTracker:gateway5, Timestamp:2019-03-06T17:32:00', info=null}
```

## <a name="link-is-not-created"></a>Ссылка не создана 
Эта ошибка возникает при создании нового соединения AMQP, но ссылка не создается в течение 1 минуты создания соединения AMQP.

```
Error{condition=amqp:connection:forced, description='The connection was inactive for more than the allowed 60000 milliseconds and is closed by container 'LinkTracker'. TrackingId:0000000000000000000000000000000000000_G21, SystemTracker:gateway5, Timestamp:2019-03-06T18:41:51', info=null}
```

## <a name="next-steps"></a>Дальнейшие действия

Сведения о служебной шине и AMQP см. по следующим ссылкам.

* [Протокол AMQP служебной шины — обзор]
* [Руководство по использованию протокола AMQP 1.0]
* [Протокол AMQP служебной шины для Windows Server]

[Протокол AMQP служебной шины — обзор]: service-bus-amqp-overview.md
[Руководство по использованию протокола AMQP 1.0]: service-bus-amqp-protocol-guide.md
[Протокол AMQP служебной шины для Windows Server]: /previous-versions/service-bus-archive/dn282144(v=azure.100)