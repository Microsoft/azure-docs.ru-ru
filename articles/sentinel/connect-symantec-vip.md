---
title: Подключение данных Symantec VIP к Azure Sentinel | Документация Майкрософт
description: Узнайте, как подключить данные Symantec VIP к Azure Sentinel.
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
ms.openlocfilehash: c7429108f70d735cb6e314a0d4daa27ba0d31637
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100090417"
---
# <a name="connect-your-symantec-vip-to-azure-sentinel"></a>Подключение Symantec VIP к Azure Sentinel

> [!IMPORTANT]
> Соединитель данных Symantec VIP в Azure Sentinel сейчас находится в общедоступной предварительной версии.
> Эта функция предоставляется без соглашения об уровне обслуживания и не рекомендуется для рабочих нагрузок. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

В этой статье объясняется, как подключить устройство [Symantec VIP](https://vip.symantec.com/) к Azure Sentinel. Соединитель данных Symantec VIP позволяет легко подключать журналы Symantec VIP с помощью Azure Sentinel, просматривать панели мониторинга, создавать пользовательские оповещения и улучшать исследование. Интеграция между Symantec VIP и Sentinel Azure использует системный журнал.

> [!NOTE]
> Данные будут храниться в географическом расположении рабочей области, на которой вы используете метку Azure.

## <a name="forward-symantec-vip-logs-to-the-syslog-agent"></a>Пересылка журналов Symantec VIP в агент системного журнала  

Настройте Symantec VIP для пересылки сообщений Syslog в рабочую область Azure с помощью агента системного журнала.

1. На портале Sentinel Azure щелкните **соединители данных** и выберите соединитель **Symantec VIP** .

1. Выберите **открыть страницу соединителя**.

1. Следуйте инструкциям на странице с **виртуальным IP-адресом Symantec** .

## <a name="find-your-data"></a>Поиск данных

После установки успешного подключения данные отображаются в Log Analytics в разделе syslog.

## <a name="validate-connectivity"></a>Проверка подключения

После того, как журналы начнут отображаться в Log Analytics, может пройти до 20 минут. 

## <a name="next-steps"></a>Дальнейшие действия

В этом документе вы узнали, как подключить журналы Symantec VIP к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.

- Узнайте, как [отслеживать свои данные и потенциальные угрозы](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.