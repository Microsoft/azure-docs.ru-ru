---
title: Подключение данных Citrix WAF к Azure Sentinel | Документация Майкрософт
description: Узнайте, как использовать соединитель данных Citrix WAF для извлечения журналов в Azure Sentinel. Просматривайте данные Citrix WAF в книгах, создавайте оповещения и улучшайте расследование.
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
ms.date: 10/25/2020
ms.author: yelevin
ms.openlocfilehash: 475e04ad3dd8c7cc51d6345b4b51dd68cff5b597
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93103127"
---
# <a name="connect-your-citrix-waf-to-azure-sentinel"></a>Подключение Citrix WAF к Azure Sentinel

> [!IMPORTANT]
> Соединитель данных брандмауэра Citrix Web Application Firewall (WAF) в Azure Sentinel сейчас находится в общедоступной предварительной версии. Эта функция предоставляется без соглашения об уровне обслуживания. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

В этой статье объясняется, как подключить устройство "брандмауэр веб-приложения Citrix" (WAF) к Azure Sentinel. Соединитель данных Citrix WAF позволяет легко подключать журналы Citrix WAF с помощью Azure Sentinel, просматривать панели мониторинга, создавать пользовательские оповещения и улучшать исследование. Подключив журналы Citrix WAF CEF к Sentinel Azure, вы можете воспользоваться преимуществами поиска и корреляции, предупреждений и анализа угроз для каждого журнала.

> [!NOTE]
> Данные будут храниться в географическом расположении рабочей области, на которой вы используете метку Azure.

## <a name="forward-citrix-waf-logs-to-the-syslog-agent"></a>Пересылка журналов Citrix WAF в агент системного журнала  

Citrix WAF отправляет сообщения syslog в формате CEF на сервер пересылки журналов на основе Linux (под управлением rsyslog или syslog-ng) с установленным агентом Log Analytics, который перенаправляет журналы в рабочую область Sentinel Azure.

1. Если у вас нет такого сервера пересылки журналов, ознакомьтесь с [этими инструкциями](connect-cef-agent.md) , чтобы запустить их.

1. Следуйте инструкциям Citrix, чтобы [настроить WAF](https://support.citrix.com/article/CTX234174), [настроить ведение журнала CEF](https://support.citrix.com/article/CTX136146)и [настроить отправку журналов в сервер пересылки журналов](https://docs.citrix.com/en-us/citrix-adc/13/system/audit-logging/configuring-audit-logging.html). Убедитесь, что журналы отправлены на порт TCP 514 на IP-адрес компьютера сервера пересылки журналов.

1. Проверка подключения и проверка приема данных с помощью [этих инструкций](connect-cef-verify.md). После того, как журналы начнут отображаться в Log Analytics, может пройти до 20 минут.

## <a name="find-your-data"></a>Поиск данных

После установки успешного подключения данные отображаются в **журналах** в разделе " **Sentinel** " в таблице *CommonSecurityLog* .

Чтобы запросить журналы Citrix WAF в Log Analytics, введите `CommonSecurityLog` в верхней части окна запроса.

## <a name="next-steps"></a>Дальнейшие действия

В этом документе вы узнали, как подключить Citrix WAF к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.
- Узнайте, как [отслеживать свои данные и потенциальные угрозы](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.