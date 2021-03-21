---
title: Управление средой Gen 2 с помощью временных рядов Azure | Документация Майкрософт
description: Узнайте, как управлять средой "аналитика временных рядов Azure" для поколения 2.
author: shipra1mishra
ms.author: shmishr
manager: diviso
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 03/15/2020
ms.custom: seodec18
ms.openlocfilehash: cbd28bdf5318bdaf932447f5d1f936d2c614a7f3
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103461901"
---
# <a name="manage-azure-time-series-insights-gen2"></a>Управление службой "аналитика временных рядов Azure" Gen2

После создания среды Gen2 "аналитика временных рядов Azure" с помощью [Azure CLI](./how-to-create-environment-using-cli.md) или [портал Azure](./how-to-create-environment-using-portal.md)можно изменить политики доступа и другие атрибуты среды в соответствии с потребностями бизнеса.

## <a name="manage-the-environment"></a>Управление средой

Вы можете управлять средой Gen2 службы "аналитика временных рядов Azure" с помощью [портал Azure](https://portal.azure.com/). Существует несколько ключевых различий между средой Gen2 и средами Gen1 S1 или Gen1 S2 при управлении средой с помощью портал Azure:

* Панель портал Azure Gen2 **Overview (обзор** ) содержит следующие изменения.

  * Емкость удалена, так как она не применяется к средам Gen2.
  * Добавляется свойство **идентификатора временных рядов** . Оно определяет способ секционирования данных.
  * Удалены наборы эталонных данных.
  * Отображаемый URL-адрес направляет вас в [Обозреватель службы "аналитика временных рядов Azure](./concepts-ux-panels.md)".
  * Указано имя вашей учетной записи хранения Azure.

* Панель **настройки** портал Azure удаляется, так как единицы масштабирования не применяются к средам Gen2 временных рядов Azure. Однако можно использовать **конфигурацию хранилища** для настройки вновь появившегося горячего хранилища.

* Панель **ссылочных данных** портал Azure удаляется в службе "аналитика временных рядов Azure" в Gen2, так как концепция ссылочных данных заменена [моделью временных рядов (ТСМ)](./concepts-model-overview.md).

:::image type="content" source="media/v2-update-manage/create-and-manage-overview-confirm.png" alt-text="Среда Gen2 &quot;аналитика временных рядов Azure&quot; в портал Azure":::

## <a name="next-steps"></a>Следующие шаги

* Ознакомьтесь со списком рекомендаций по [приему потоковой передачи](./concepts-streaming-ingestion-event-sources.md#streaming-ingestion-best-practices)
* Узнайте, как [диагностировать и устранять неполадки](./how-to-diagnose-troubleshoot.md)
