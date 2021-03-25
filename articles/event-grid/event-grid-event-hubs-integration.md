---
title: Руководство по Отправка данных Центров событий Azure в хранилище данных в службе "Сетка событий"
description: В этой статье описывается, как хранить данные Центров событий в Azure Synapse Analytics с помощью триггеров служб "Функции Azure" и "Сетка событий".
ms.topic: tutorial
ms.date: 12/07/2020
ms.custom: devx-track-csharp, devx-track-azurecli
ms.openlocfilehash: 7b0e471e32650490e1896bb6ea171c8223b21378
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96854722"
---
# <a name="tutorial-stream-big-data-into-a-data-warehouse"></a>Руководство по Потоковая передача больших данных в хранилище данных
[Сетка событий](overview.md) Azure — это интеллектуальная служба маршрутизации событий, позволяющая реагировать на уведомления или события приложений и служб. Например, она может активировать функцию Azure для обработки данных Центров событий, сохраненных в Хранилище BLOB-объектов или Data Lake Storage. В этом [примере](https://github.com/Azure/azure-event-hubs/tree/master/samples/e2e/EventHubsCaptureEventGridDemo) демонстрируется, как использовать Сетку событий и Функции Azure для переноса сохраненных данных Центров событий из Хранилища BLOB-объектов в Azure Synapse Analytics, а именно в выделенный пул SQL.

[!INCLUDE [event-grid-event-hubs-functions-synapse-analytics.md](../../includes/event-grid-event-hubs-functions-synapse-analytics.md)]

## <a name="next-steps"></a>Дальнейшие действия

* Сведения о различиях между службами обмена сообщениями в Azure см. в статье [Выбор между службами обмена сообщениями Azure](compare-messaging-services.md).
* Общие сведения о службе "Сетка событий" см. в разделе [Общие сведения о службе "Сетка событий Azure"](overview.md).
* Общие сведения о функции "Сбор" в Центрах событий см. в статье [Включение функции "Сбор" в Центрах событий с помощью портала Azure](../event-hubs/event-hubs-capture-enable-through-portal.md).
* Дополнительные сведения о настройке и запуске примера см. в [примере использования функции "Сбор" в Центрах событий и службы "Сетка событий"](https://github.com/Azure/azure-event-hubs/tree/master/samples/e2e/EventHubsCaptureEventGridDemo).
