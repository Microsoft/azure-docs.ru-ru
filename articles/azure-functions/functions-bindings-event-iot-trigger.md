---
title: Триггер центра Интернета вещей Azure для функций Azure
description: Узнайте, как реагировать на события, отправляемые в поток событий центра Интернета вещей в функциях Azure.
author: craigshoemaker
ms.topic: reference
ms.date: 02/21/2020
ms.author: cshoe
ms.openlocfilehash: 5c9309834b407ee56d29e38afd965ac947fc8a4f
ms.sourcegitcommit: d135e9a267fe26fbb5be98d2b5fd4327d355fe97
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102612292"
---
# <a name="azure-iot-hub-trigger-for-azure-functions"></a>Триггер центра Интернета вещей Azure для функций Azure

В этой статье объясняется, как работать с привязками функций Azure для центра Интернета вещей. Поддержка центра Интернета вещей основана на [привязке концентраторов событий Azure](functions-bindings-event-hubs.md).

Сведения об установке и настройке см. в [этой обзорной статье](functions-bindings-event-iot.md).

> [!IMPORTANT]
> Хотя в следующих примерах кода используется API концентратора событий, заданный синтаксис применим к функциям центра Интернета вещей.

[!INCLUDE [functions-bindings-event-hubs](../../includes/functions-bindings-event-hubs-trigger.md)]

## <a name="hostjson-properties"></a>Свойства host.json

[host.js](functions-host-json.md#eventhub) файла содержит параметры, управляющие поведением триггера концентратора событий. Дополнительные сведения о доступных параметрах см. в разделе [host.jsпо параметрам](functions-bindings-event-iot.md#hostjson-settings) .

## <a name="next-steps"></a>Дальнейшие действия

- [Запись событий в поток событий (Выходная привязка)](./functions-bindings-event-iot-output.md)
