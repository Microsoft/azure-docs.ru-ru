---
title: Подключение УЛУЧШЕНной защиты мобильных устройств от угроз (MTD) к Azure Sentinel | Документация Майкрософт
description: Узнайте, как использовать УЛУЧШЕННЫЙ соединитель данных для защиты мобильных устройств от угроз (MTD) для извлечения журналов MTD в Azure Sentinel. Просмотр данных MTD в книгах, создание оповещений и улучшение расследования.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.assetid: 0001cad6-699c-4ca9-b66c-80c194e439a5
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/12/2021
ms.author: yelevin
ms.openlocfilehash: 4828e31b9d15f101740c158ee62c90c95673c9a7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98541564"
---
# <a name="connect-your-better-mobile-threat-defense-mtd-to-azure-sentinel"></a>Подключение УЛУЧШЕНной защиты от угроз для мобильных устройств (MTD) к Azure Sentinel

> [!IMPORTANT]
> УЛУЧШЕННЫЙ соединитель мобильной защиты от угроз (MTD) сейчас находится на **этапе предварительной версии**. Ознакомьтесь с дополнительными [условиями использования Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) предварительных версий для дополнительных юридических условий, которые относятся к функциям Azure, которые доступны в бета-версии, предварительном просмотре или еще не выпущены в общедоступную версию.

УЛУЧШЕННЫЙ соединитель мобильной защиты от угроз (MTD) позволяет легко подключать все более эффективные журналы решений безопасности MTD с помощью Sentinel Azure, просматривать панели мониторинга, создавать пользовательские оповещения и улучшать исследование. Интеграция между УЛУЧШЕНной защитой мобильных угроз и Sentinel в Azure использует REST API.

> [!NOTE]
> Данные будут храниться в географическом расположении рабочей области, на которой вы используете метку Azure.

## <a name="configure-and-connect-better-mobile-threat-defense"></a>Настройка и подключение УЛУЧШЕНной защиты мобильных устройств от угроз

Более эффективные MTD могут интегрировать и экспортировать журналы непосредственно в Azure Sentinel.

1. В меню навигации меток Azure выберите **соединители данных**.

1. В коллекции **соединителей данных** выберите **Улучшенная защита от угроз для мобильных устройств (MTD) (Предварительная версия)** , а затем **откройте страницу соединителя**.

1. Выполните действия на странице соединителя и на [этой странице из лучшего MTD документации](https://mtd-docs.bmobi.net/integrations/azure-sentinel/setup-integration#mtd-integration-configuration) , чтобы завершить ИНТЕГРАЦИЮ с улучшенной консолью MTD.

    По запросу введите **идентификатор рабочей области** и значения **первичного ключа** , скопируйте их со страницы соединителя Azure Sentinel и вставьте их в улучшенную конфигурацию MTD.

    :::image type="content" source="media/connectors/workspace-id-primary-key.png" alt-text="{ИДЕНТИФИКАТОР рабочей области и первичный ключ}":::

## <a name="find-your-data"></a>Поиск данных

После установки успешного подключения данные отображаются в **журналах** в разделе **CustomLogs** в одной или нескольких следующих таблицах:
- `BetterMTDDeviceLog_CL`
- `BetterMTDIncidentLog_CL`
- `BetterMTDAppLog_CL`
- `BetterMTDNetflowLog_CL`

Чтобы запросить более эффективные журналы MTD в правилах аналитики, поиске запросов или в любом месте в Azure Sentinel, введите одно из приведенных выше имен таблиц в верхней части окна запроса.

## <a name="validate-connectivity"></a>Проверка подключения

После того, как журналы начнут отображаться в Log Analytics, может пройти до 20 минут. 

## <a name="next-steps"></a>Дальнейшие действия

В этом документе вы узнали, как подключить УЛУЧШЕНную защиту от угроз для мобильных устройств (MTD) к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.
- Узнайте, как [получить представление о данных и потенциальных угрозах](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.
