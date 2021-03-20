---
title: Подключение данных операционной системы Инфоблокс Network Identity (Ниос) к Azure Sentinel | Документация Майкрософт
description: Узнайте, как подключить данные операционной системы удостоверений Инфоблокс Networking System (Ниос) к Azure Sentinel.
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
ms.openlocfilehash: 28abb9f09e3bca2522b959c6a9b890de5320b17a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98567435"
---
# <a name="connect-your-infoblox-nios-to-azure-sentinel"></a>Подключение Инфоблокс Ниос к Azure Sentinel

> [!IMPORTANT]
> Соединитель данных Инфоблокс Ниос в Azure Sentinel сейчас находится в общедоступной предварительной версии.
> Эта функция предоставляется без соглашения об уровне обслуживания и не рекомендуется для рабочих нагрузок. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

В этой статье описывается подключение [устройства Инфоблокс Networking System (Ниос)](https://www.infoblox.com/glossary/network-identity-operating-system-nios/) к Azure Sentinel. Соединитель данных Инфоблокс Ниос позволяет легко подключать журналы Инфоблокс к Azure Sentinel, просматривать панели мониторинга, создавать пользовательские оповещения и улучшать исследование. При интеграции между Инфоблокс Ниос и Sentinel Azure используется системный журнал.

> [!NOTE]
> Данные будут храниться в географическом расположении рабочей области, на которой вы используете метку Azure.

## <a name="forward-infoblox-logs-to-the-syslog-agent"></a>Пересылка журналов Инфоблокс в агент системного журнала  

Настройте Инфоблокс для пересылки сообщений Syslog в рабочую область Sentinel Azure с помощью агента системного журнала.

1. На портале Sentinel Azure щелкните **соединители данных** и выберите **инфоблокс Ниос** Connector.

1. Выберите **открыть страницу соединителя**.

1. Следуйте инструкциям на странице **ИНФОБЛОКС Ниос** .

## <a name="find-your-data"></a>Поиск данных

После установки успешного подключения данные отображаются в Log Analytics в разделе syslog.

## <a name="validate-connectivity"></a>Проверка подключения

После того, как журналы начнут отображаться в Log Analytics, может пройти до 20 минут. 

## <a name="next-steps"></a>Дальнейшие действия

В этом документе вы узнали, как подключить Инфоблокс Ниос к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.

- Узнайте, как [отслеживать свои данные и потенциальные угрозы](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.