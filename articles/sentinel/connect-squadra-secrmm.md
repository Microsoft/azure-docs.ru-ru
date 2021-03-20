---
title: Подключение скуадра Technologies Секрмм Data к Azure Sentinel | Документация Майкрософт
description: Узнайте, как подключить скуадра Technologies Секрмм Data to Azure Sentinel.
services: sentinel
author: yelevin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/20/2021
ms.author: yelevin
ms.openlocfilehash: a00b4b1e81c0d644cf1475aa46dda3848fda1365
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98632907"
---
# <a name="connect-your-squadra-technologies-secrmm-data-to-azure-sentinel"></a>Подключение данных Секрмм скуадра Technologies к Azure Sentinel 

Соединитель скуадра Technologies Секрмм позволяет легко подключать ваши скуадра технологии Секрмм журналов решений безопасности с помощью Azure Sentinel. Она позволяет просматривать панели мониторинга, создавать пользовательские предупреждения и улучшать исследование. Этот соединитель позволяет получить представление о событиях съемных носителей USB. Интеграция между скуадра технологиями Секрмм и Azure Sentinel позволяет использовать REST API.

> [!NOTE]
> Данные будут храниться в географическом расположении рабочей области, на которой вы используете метку Azure.

## <a name="configure-and-connect-squadra-technologies-secrmm"></a>Настройка и подключение технологий скуадра Секрмм 

Технологии скуадра Секрмм могут интегрировать и экспортировать журналы непосредственно в Azure Sentinel.
1. На портале Sentinel Azure щелкните соединители данных, выберите скуадра Technologiess Секрмм, а затем откройте страницу соединителя.

2. Выполните действия, описанные в руководстве по [адаптации технологий скуадра для Azure Sentinel](http://www.squadratechnologies.com/StaticContent/ProductDownload/secRMM/9.9.0.0/secRMMAzureSentinelAdministratorGuide.pdf) , чтобы получить данные скуадра Секрмм в Azure Sentinel.   

## <a name="find-your-data"></a>Поиск данных

После установки успешного подключения данные отображаются в **журналах** в разделе **CustomLogs** в `secRMM_CL` таблице.

Чтобы запросить журналы скуадра Technologies Секрмм, введите имя таблицы в верхней части окна запроса.

## <a name="validate-connectivity"></a>Проверка подключения

После того, как журналы начнут отображаться в Log Analytics, может пройти до 20 минут. 

## <a name="next-steps"></a>Дальнейшие действия

В этом документе вы узнали, как подключить скуадра Technologies Секрмм к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.

- Узнайте, как [отслеживать свои данные и потенциальные угрозы](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.
