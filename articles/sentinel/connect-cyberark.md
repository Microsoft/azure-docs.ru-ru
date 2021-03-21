---
title: Подключение данных Циберарк ЕПВ к Azure Sentinel | Документация Майкрософт
description: Узнайте, как использовать соединитель данных Циберарк Enterprise Password Vault (ЕПВ) для извлечения журналов в Azure Sentinel. Просмотр данных Циберарк ЕПВ в книгах, создание оповещений и улучшение расследования.
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
ms.openlocfilehash: aa57963ce369e4c8f84f4aae5f99fe343181ff6b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100530507"
---
# <a name="connect-cyberark-enterprise-password-vault-epv-to-azure-sentinel"></a>Подключение хранилища паролей Циберарк Enterprise (ЕПВ) к Azure Sentinel

> [!IMPORTANT]
> Соединитель Data Vault (ЕПВ) Циберарк предприятия в Azure Sentinel в настоящее время находится в общедоступной предварительной версии. Эта функция предоставляется без соглашения об уровне обслуживания. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Соединитель syslog Циберарк позволяет легко подключать все журналы решений безопасности Циберарк с помощью Sentinel Azure, просматривать панели мониторинга, создавать пользовательские оповещения и улучшать исследование. Интеграция между Циберарк и Sentinel в Azure позволяет использовать соединитель данных CEF для правильного анализа и вывода сообщений системного журнала Циберарк.

> [!NOTE]
> Данные будут храниться в географическом расположении рабочей области, на которой вы используете метку Azure.

## <a name="configure-and-connect-cyberark-epv"></a>Настройка и подключение Циберарк ЕПВ

Журналы ЕПВ Циберарк отправляются из хранилища на сервер пересылки журналов на основе Linux (под управлением rsyslog или syslog-ng) с установленным агентом Log Analytics, который экспортирует журналы в Azure Sentinel. Если у вас нет такого сервера пересылки журналов, ознакомьтесь с [этими инструкциями](connect-cef-agent.md) , чтобы запустить их.

1. На портале Sentinel Azure щелкните **соединители данных**, выберите **циберарк (Предварительная версия) событий хранилища ПАРОЛЕЙ предприятия (ЕПВ)** , а затем **откройте страницу соединителя**.

1. Следуйте [инструкциям по ЦИБЕРАРК ЕПВ](https://docs.cyberark.com/Product-Doc/OnlineHelp/PAS/Latest/en/Content/PASIMP/DV-Integrating-with-SIEM-Applications.htm) , чтобы настроить отправку данных системного журнала на сервер пересылки журналов.

1. Проверка подключения и проверка приема данных с помощью [этих инструкций](connect-cef-verify.md). После того, как журналы начнут отображаться в Log Analytics, может пройти до 20 минут.

## <a name="find-your-data"></a>Поиск данных

После установки успешного подключения данные отображаются в **журналах** в разделе " **Sentinel** " в таблице *CommonSecurityLog* .

Чтобы запросить журналы Циберарк ЕПВ в Log Analytics, введите `CommonSecurityLog` в верхней части окна запроса.

## <a name="next-steps"></a>Дальнейшие действия

В этом документе вы узнали, как подключить журналы хранилища паролей Циберарк предприятия к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.
- Узнайте, как [отслеживать свои данные и потенциальные угрозы](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.
