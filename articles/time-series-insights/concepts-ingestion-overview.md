---
title: Общие сведения о приеме данных — Аналитика временных рядов Azure 2-го поколения | Документация Майкрософт
description: Узнайте о приеме данных в службе "Аналитика временных рядов Azure" 2-го поколения.
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 12/02/2020
ms.custom: seodec18
ms.openlocfilehash: b688bbf454b3df9f6ae368c66a62657a5522d720
ms.sourcegitcommit: c2a41648315a95aa6340e67e600a52801af69ec7
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2021
ms.locfileid: "106505456"
---
# <a name="azure-time-series-insights-gen2-data-ingestion-overview"></a>Общие сведения о приеме данных в службе "Аналитика временных рядов Azure" 2-го поколения

Среда Аналитики временных рядов Azure 2-го поколения содержит *механизм приема* для сбора, обработки и хранения потоковых данных временных рядов. По мере поступления данных в источники событий служба "Аналитика временных рядов Azure" 2-го поколения будет принимать и хранить данные практически в реальном времени.

[![Общие сведения о приеме данных](media/concepts-ingress-overview/ingress-overview.png)](media/concepts-ingress-overview/ingress-overview.png#lightbox)

## <a name="ingestion-topics"></a>Темы категории "Прием данных"

В следующих статьях подробно рассмотрена обработка данных, включая опыт профессионалов.

* Ознакомьтесь с [источниками событий](./concepts-streaming-ingestion-event-sources.md) и рекомендациями по выбору отметки времени источника события.

* Просмотрите [поддерживаемые типы данных](./concepts-supported-data-types.md)

* Узнайте о том, как подсистема приема будет применять набор [правил](./concepts-json-flattening-escaping-rules.md) к свойствам JSON для создания столбцов учетной записи хранения.

* Проверьте [ограничения пропускной способности](./concepts-streaming-ingress-throughput-limits.md) среды, чтобы спланировать требования к масштабированию.

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительная информация об [источниках событий](./concepts-streaming-ingestion-event-sources.md) для среды "Аналитика временных рядов Azure" 2-го поколения.
