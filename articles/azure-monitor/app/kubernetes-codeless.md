---
title: Мониторинг приложений, выполняемых в службе Azure Kubernetes (AKS), с помощью Application Insights (Azure Monitor) | Документация Майкрософт
description: Azure Monitor без проблем интегрируется с приложением, запущенным в Kubernetes, и позволяет почти мгновенно выявлять проблемы с приложениями.
ms.topic: conceptual
author: MS-jgol
ms.author: jgol
ms.date: 05/13/2020
ms.openlocfilehash: 52bd6d2a98e5126ff2463de1ef99da03ca555567
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "87075302"
---
# <a name="zero-instrumentation-application-monitoring-for-kubernetes---azure-monitor-application-insights"></a>Мониторинг приложений, размещенных в Kubernetes, без инструментирования — Application Insights в Azure Monitor

> [!IMPORTANT]
>  Сейчас вы можете включить мониторинг своих приложений Java, работающих в Kubernetes, без инструментирования кода — используйте [автономный агент Java](./java-in-process-agent.md). Хотя решение, позволяющее легко включить мониторинг приложений, работает и на других языках, используйте пакеты SDK для мониторинга приложений, выполняемых в AKS: [ASP.NET Core](./asp-net-core.md), [ASP.NET](./asp-net.md), [Node.js](./nodejs.md), [JavaScript](./javascript.md) и [Python](./opencensus-python.md).

## <a name="application-monitoring-without-instrumenting-the-code"></a>Мониторинг приложений без инструментирования кода
В настоящий момент только Java позволяет использовать мониторинг приложений без инструментирования кода. Для мониторинга приложений на других языках используются пакеты SDK. 

## <a name="java"></a>Java
После включения агент Java автоматически соберет множество запросов, зависимостей, журналов и метрик из наиболее популярных библиотек и платформ.

Следуйте [подробным инструкциям](./java-in-process-agent.md) для мониторинга приложений Java, выполняемых в приложениях Kubernetes, а также в других средах. 

## <a name="other-languages"></a>Другие языки

Для приложений на других языках сейчас рекомендуется использовать пакеты SDK:
* [ASP.NET Core](./asp-net-core.md)
* [ASP.NET](./asp-net.md)
* [Node.js](./nodejs.md) 
* [JavaScript](./javascript.md)
* [Python](./opencensus-python.md)

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения об [Azure Monitor](../overview.md) и [Application Insights](./app-insights-overview.md).
* Ознакомьтесь с обзором [Распределенной трассировки](./distributed-tracing.md) и узнайте, как [Схемы приложений](./app-map.md?tabs=net) могут помочь вашему бизнесу.
