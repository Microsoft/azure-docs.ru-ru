---
title: Подключение данных брандмауэра Sophos XG к Azure Sentinel | Документация Майкрософт
description: Узнайте, как подключить данные брандмауэра Sophos XG к Azure Sentinel.
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
ms.openlocfilehash: 0b8b6247a735bceaf98029740bf9d4f7e233069d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100097574"
---
# <a name="connect-your-sophos-xg-firewall-to-azure-sentinel"></a>Подключение брандмауэра Sophos XG к Azure Sentinel

> [!IMPORTANT]
> Соединитель данных брандмауэра Sophos XG в Azure Sentinel в настоящее время находится в общедоступной предварительной версии.
> Эта функция предоставляется без соглашения об уровне обслуживания и не рекомендуется для рабочих нагрузок. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

В этой статье объясняется, как подключить устройство [брандмауэра SOPHOS XG](https://www.sophos.com/products/next-gen-firewall.aspx) к Azure Sentinel. Соединитель данных брандмауэра Sophos XG позволяет легко подключать журналы брандмауэра Sophos XG к Azure Sentinel, просматривать панели мониторинга, создавать пользовательские оповещения и улучшать расследования. При интеграции между брандмауэром Sophos XG и Sentinel в Azure используется системный журнал.

> [!NOTE]
> Данные будут храниться в географическом расположении рабочей области, на которой вы используете метку Azure.

## <a name="forward-sophos-xg-firewall-logs-to-the-syslog-agent"></a>Пересылка журналов брандмауэра Sophos XG в агент системного журнала  

Настройте брандмауэр Sophos XG для пересылки сообщений Syslog в рабочую область Azure с помощью агента системного журнала.

1. На портале Sentinel Azure щелкните **Data Connectors (соединители данных** ) и выберите **Sophos XG Firewall** Connector (соединитель брандмауэра).

1. Выберите **открыть страницу соединителя**.

1. Следуйте инструкциям на странице **брандмауэра SOPHOS XG** .

## <a name="find-your-data"></a>Поиск данных

После установки успешного подключения данные отображаются в Log Analytics в разделе syslog.

## <a name="validate-connectivity"></a>Проверка подключения

После того, как журналы начнут отображаться в Log Analytics, может пройти до 20 минут.

## <a name="next-steps"></a>Дальнейшие действия

В этом документе вы узнали, как подключить Sophos XG Firewall к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.

- Узнайте, как [отслеживать свои данные и потенциальные угрозы](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.