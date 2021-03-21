---
title: Анализ вариантов использования CDN Azure
description: В этой статье описываются разные типы аналитических отчетов, доступные для продуктов Azure CDN.
services: cdn
author: asudbring
manager: KumudD
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/30/2020
ms.author: allensu
ms.openlocfilehash: 2c593cda7761ce1defa6bdb31e0dbb528b9e5eca
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96483994"
---
# <a name="analyze-azure-cdn-usage-patterns"></a>Анализ вариантов использования CDN Azure

Когда вы включите CDN для вашего приложения, вам, скорее всего, нужно будет отслеживать использование CDN, проверять работоспособность службы доставки и устранять возможные неполадки. Azure CDN позволяет делать это следующим образом. 

## <a name="raw-logs-for-azure-cdn-from-microsoft"></a>Необработанные журналы для Azure CDN от Майкрософт
Используя стандартный профиль Майкрософт, можно включить необработанные журналы и выбрать потоковую передачу журналов в:

* Хранилище Azure
* Центры событий
* Azure Log Analytics

С помощью Azure Log Analytics можно просматривать метрики мониторинга и настраивать оповещения. 

Дополнительные сведения см. в разделе [Azure CDN необработанных журналов HTTP](monitoring-and-access-log.md).


## <a name="core-analytics-via-azure-diagnostic-logs"></a>Базовая аналитика с использованием журналов диагностики Azure

Эта функция доступна на конечных точках CDN всех ценовых категорий. Журналы диагностики Azure позволяют экспортировать основную аналитику в службу хранилища Azure, концентраторы событий или журналы Azure Monitor. Журналы Azure Monitor предлагают решение с настраиваемыми и настраиваемыми графами. См. дополнительные сведения о [журналах диагностики Azure](cdn-azure-diagnostic-logs.md).

## <a name="verizon-core-reports"></a>С помощью базовых отчетов Verizon

**Azure CDN уровня "Стандартный" из Verizon** или **Azure CDN "Премиум" из профилей Verizon** предоставляют основные отчеты. Основные отчеты можно просмотреть на вспомогательном портале Verizon. Verizon основные отчеты доступны через параметр **управления** из портал Azure и предлагают различные виды диаграмм и представлений. См. дополнительные сведения о [базовых отчетах Verizon](cdn-analyze-usage-patterns.md).

## <a name="verizon-custom-reports"></a>Настраиваемые отчеты Verizon

**Azure CDN Standard от Verizon** или **Azure CDN Premium из профилей Verizon** предоставляют настраиваемые отчеты. Вы можете просматривать пользовательские отчеты на вспомогательном портале Verizon. Пользовательские отчеты Verizon доступны через параметр **управления** из портал Azure. 

В пользовательских отчетах отображается количество попаданий или данных, передаваемых для каждой пограничной записи CNAME. Данные группируются по коду ответа HTTP или состоянию кэша за определенный период времени. См. дополнительные сведения о [настраиваемых отчетах Verizon](cdn-verizon-custom-reports.md).

## <a name="azure-cdn-premium-from-verizon-reports"></a>Отчеты Azure CDN уровня "Премиум" от Verizon

С помощью **Azure CDN от Verizon уровня "Премиум"** можно получить доступ к следующим отчетам:
   * [Расширенные HTTP-отчеты.](cdn-advanced-http-reports.md)
   * [Статистика в режиме реального времени.](cdn-real-time-stats.md)
   * [Производительность Azure CDN граничных узлов](cdn-edge-performance.md)

## <a name="next-steps"></a>Дальнейшие действия
В этой статье вы узнали о различных вариантах аналитических отчетов для Azure CDN.

Дополнительные сведения о Azure CDN и других службах Azure, упомянутых в этой статье, см. в следующих статьях:

* [Что такое Azure CDN?](cdn-overview.md)
* [Необработанные журналы HTTP (Azure CDN)](monitoring-and-access-log.md)
