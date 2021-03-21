---
title: Триггер концентраторов событий Azure для функций Azure
description: Узнайте, как использовать триггер концентраторов событий Azure в функциях Azure.
author: craigshoemaker
ms.assetid: daf81798-7acc-419a-bc32-b5a41c6db56b
ms.topic: reference
ms.date: 02/21/2020
ms.author: cshoe
ms.openlocfilehash: c2b8302e64f7dcc657fd20ed5d918ed6816d750d
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102608928"
---
# <a name="azure-event-hubs-trigger-for-azure-functions"></a>Триггер концентраторов событий Azure для функций Azure

В этой статье объясняется, как работать с триггером [концентраторов событий Azure](../event-hubs/event-hubs-about.md) для функций Azure. Функции Azure поддерживают привязки триггера и [выходных данных](functions-bindings-event-hubs-output.md) для концентраторов событий.

Сведения об установке и настройке см. в [этой обзорной статье](functions-bindings-event-hubs.md).

[!INCLUDE [functions-bindings-event-hubs-trigger](../../includes/functions-bindings-event-hubs-trigger.md)]

## <a name="hostjson-settings"></a>Параметры файла host.json

[host.js](functions-host-json.md#eventhub) файла содержит параметры, управляющие поведением триггера концентратора событий. Дополнительные сведения о доступных параметрах см. в разделе [host.jsпо параметрам](functions-bindings-event-hubs.md#hostjson-settings) .

## <a name="next-steps"></a>Следующие шаги

- [Запись событий в поток событий (Выходная привязка)](./functions-bindings-event-hubs-output.md)
