---
title: Подключение данных Proofpoint On Demand с целью защиты от несанкционированного подключения (TAP) к Azure Sentinel | Документация Майкрософт
description: Узнайте, как подключить данные защиты от несанкционированного Proofpoint On Demand (TAP) к Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/17/2020
ms.author: yelevin
ms.openlocfilehash: 86018bafaa42eac01e5dccf8da1d290b64e2475c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100092984"
---
# <a name="connect-your-proofpoint-tap-to-azure-sentinel-with-azure-function"></a>Подключение Proofpoint On Demand TAP к Azure Sentinel с помощью функции Azure

> [!IMPORTANT]
> Соединитель данных TAP для Proofpoint On Demand в Azure Sentinel в настоящее время находится в общедоступной предварительной версии.
> Эта функция предоставляется без соглашения об уровне обслуживания и не рекомендуется для рабочих нагрузок. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Соединитель защиты от назначенных атак Proofpoint On Demand (TAP) позволяет легко подключать все журналы решений безопасности [PROOFPOINT On Demand TAP](https://www.proofpoint.com/us/products/advanced-threat-protection/targeted-attack-protection) с помощью Azure Sentinel, просматривать панели мониторинга, создавать пользовательские оповещения и улучшать исследование. Интеграция между Proofpoint On Demand TAP и Azure Sentinel позволяет использовать функции Azure для извлечения данных журнала с помощью REST API.

> [!NOTE]
> Данные будут храниться в географическом расположении рабочей области, на которой вы используете метку Azure.

## <a name="configure-and-connect-proofpoint-tap"></a>Настройка и подключение Proofpoint On Demand TAP

Функции Azure могут интегрировать и запрашивать события и журналы непосредственно из Proofpoint On Demand TAP и пересылать их в Azure Sentinel.

1. На портале Sentinel Azure щелкните **соединители данных** и выберите соединитель **proofpoint On Demand TAP** .

1. Выберите **открыть страницу соединителя**.

1. Следуйте инструкциям на странице **PROOFPOINT On Demand TAP** .

## <a name="find-your-data"></a>Поиск данных

После установки успешного подключения данные появляются в Log Analytics в таблицах **ProofpointTAPMessagesBlocked_CL**, **ProofpointTAPMessagesDelivered_CL**, **ProofpointTAPClicksPermitted_CL** и **ProofpointTAPClicksBlocked_CL** .

## <a name="validate-connectivity"></a>Проверка подключения

После того, как журналы начнут отображаться в Log Analytics, может пройти до 20 минут.

## <a name="next-steps"></a>Дальнейшие действия

В этом документе вы узнали, как подключить Proofpoint On Demand TAP к Azure Sentinel с помощью приложений функций Azure. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.

- Узнайте, как [отслеживать свои данные и потенциальные угрозы](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.
