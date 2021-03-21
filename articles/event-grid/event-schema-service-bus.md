---
title: Служба "служебная шина Azure" как источник "Сетка событий"
description: Описание свойств для событий служебной шины, используемых со службой "Сетка событий Azure"
ms.topic: conceptual
ms.date: 02/12/2021
ms.openlocfilehash: d3e14788d49697a1f86624bbe8d6d0ec2eb072c8
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100363253"
---
# <a name="azure-service-bus-as-an-event-grid-source"></a>Служебная шина Azure в качестве источника службы "Сетка событий"

В этой статье описаны свойства и схема для событий служебной шины. Общие сведения о схемах событий см. в статье [Схема событий службы "Сетка событий Azure"](event-schema.md).

[!INCLUDE [event-grid-service-bus.md](../../includes/event-grid-service-bus.md)]

## <a name="tutorials-and-how-tos"></a>Руководства и инструкции
|Заголовок  |Описание  |
|---------|---------|
| [Руководство. с примерами интеграции служебной шины Azure со службой "Сетка событий Azure"](../service-bus-messaging/service-bus-to-event-grid-integration-example.md?toc=%2fazure%2fevent-grid%2ftoc.json) | Служба "Сетка событий" отправляет сообщения из раздела служебной шины в приложение-функцию и приложение логики. |
| [Интеграция служебной шины Azure в службу "Сетка событий"](../service-bus-messaging/service-bus-to-event-grid-integration-concept.md) | Общие сведения об интеграции служебной шины со службой "Сетка событий". |

## <a name="next-steps"></a>Дальнейшие действия

* См. общие сведения о [службе "Сетка событий Azure"](overview.md).
* Дополнительные сведения о создании подписки на Сетку событий Azure см. в статье [Схема подписки для службы "Сетка событий"](subscription-creation-schema.md).
* Дополнительные сведения об использовании сетки событий Azure со служебной шиной см. в статье [Общие сведения об интеграции служебной шины Azure со службой "Сетка событий Azure"](../service-bus-messaging/service-bus-to-event-grid-integration-concept.md).
* Попробуйте [получать события служебной шины с помощью функций или приложений Logic Apps](../service-bus-messaging/service-bus-to-event-grid-integration-example.md?toc=%2fazure%2fevent-grid%2ftoc.json).
