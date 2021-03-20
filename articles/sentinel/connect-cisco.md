---
title: Подключение данных Cisco к Azure Sentinel | Документация Майкрософт
description: Узнайте, как подключить устройство Cisco ASA к Sentinel Azure для просмотра панелей мониторинга, создания пользовательских оповещений и улучшения расследования.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.assetid: 62029b5c-29d3-4336-8a22-a9db8214eb7e
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/30/2019
ms.author: yelevin
ms.openlocfilehash: e8a64dd3e47384ba2bf7579f8052177252634622
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85566033"
---
# <a name="connect-cisco-asa-to-azure-sentinel"></a>Подключение Cisco ASA к Azure Sentinel



В этой статье объясняется, как подключить устройство Cisco ASA к Azure Sentinel. Соединитель данных Cisco ASA позволяет легко подключать журналы Cisco ASA с помощью Azure Sentinel, просматривать панели мониторинга, создавать пользовательские оповещения и улучшать исследование. С помощью Cisco ASA в Azure Sentinel вы получите более подробные сведения об использовании Интернета в вашей организации и улучшите возможности его работы по обеспечению безопасности. 



## <a name="forward-cisco-asa-logs-to-the-syslog-agent"></a>Пересылка журналов Cisco ASA в агент системного журнала

Cisco ASA не поддерживает CEF, поэтому журналы отправляются в виде системного журнала, и агент Sentinel Azure знает, как их анализировать, как если бы они были журналами CEF. Настройте Cisco ASA для пересылки сообщений Syslog в рабочую область Azure с помощью агента системного журнала:

1. Перейдите к разделу [Отправка сообщений системного журнала на внешний сервер syslog](https://aka.ms/asi-syslog-cisco-forwarding)и следуйте инструкциям по настройке подключения. При появлении запроса используйте следующие параметры:
    - Задайте для параметра **порт** значение 514 или порт, заданный в агенте.
    - Задайте **syslog_ip** IP-адресу агента.

1. Чтобы использовать соответствующую схему в Log Analytics для событий Cisco, выполните поиск по запросу `CommonSecurityLog` .

1. Перейдите к процедуре [Шаг 3. Проверка подключения](connect-cef-verify.md).




## <a name="next-steps"></a>Дальнейшие действия
В этом документе вы узнали, как подключить устройства Cisco ASA к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.
- Узнайте, как [отслеживать свои данные и потенциальные угрозы](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.


