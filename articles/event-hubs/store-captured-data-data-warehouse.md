---
title: Руководство по переносу данных о событиях в Azure Synapse Analytics (Центры событий Azure)
description: В этой статье описывается, как использовать Сетку событий и Функции Azure для переноса сохраненных в Центрах событий данных в Azure Synapse Analytics.
services: event-hubs
ms.date: 12/07/2020
ms.topic: tutorial
ms.custom: devx-track-csharp
ms.openlocfilehash: 77a2b35dea57c71e9e6f07056bd106df2ac109b8
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96854238"
---
# <a name="tutorial-migrate-captured-event-hubs-data-to-azure-synapse-analytics-using-event-grid-and-azure-functions"></a>Руководство по переносу собранных данных из Центров событий Azure в Azure Synapse Analytics с помощью служб "Сетка событий" и "Функции Azure"
Функция [Сбор](./event-hubs-capture-overview.md) в Центрах событий позволяет вам автоматически собирать данные потоковой передачи из Центров событий в Хранилище BLOB-объектов Azure или в Azure Data Lake Storage. В этом учебнике показано, как перенести сохраненные в Центрах событий данные из хранилища в Azure Synapse Analytics с помощью функции Azure, активируемой службой [Сетка событий](../event-grid/overview.md).

[!INCLUDE [event-grid-event-hubs-functions-synapse-analytics.md](../../includes/event-grid-event-hubs-functions-synapse-analytics.md)]

## <a name="next-steps"></a>Дальнейшие действия 
Вы можете использовать мощные инструменты визуализации данных в хранилище данных, чтобы получить аналитические сведения.

В этой статье описано, как использовать [Power BI с Azure Synapse Analytics](/power-bi/connect-data/service-azure-sql-data-warehouse-with-direct-connect).