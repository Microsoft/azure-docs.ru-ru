---
title: Подключение к подсети Cisco к Azure Sentinel | Документация Майкрософт
description: Узнайте, как использовать соединитель данных Cisco, чтобы извлечь данные о подразделении в Azure Sentinel. Просмотр данных с символами подсистемы в книгах, создание предупреждений и улучшение расследования.
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
ms.date: 02/03/2021
ms.author: yelevin
ms.openlocfilehash: 22796134dae5c345e3f915e47bc1300affb9f60e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99566902"
---
# <a name="connect-your-cisco-umbrella-to-azure-sentinel"></a>Подключение "бесвирусного" Cisco к Azure Sentinel

> [!IMPORTANT]
> Соединитель беспроводных соединителей Cisco сейчас находится на **этапе предварительной версии**. Ознакомьтесь с дополнительными [условиями использования Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) предварительных версий для дополнительных юридических условий, которые относятся к функциям Azure, которые доступны в бета-версии, предварительном просмотре или еще не выпущены в общедоступную версию.

Соединитель беспроводных контактов Cisco позволяет легко подключать все журналы решений безопасности Cisco с помощью метки Azure, просматривать панели мониторинга, создавать пользовательские оповещения и улучшать исследование. Интеграция между «компанией Cisco» и «Sentinel» Azure использует REST API.

> [!NOTE]
> Данные будут храниться в географическом расположении рабочей области, на которой вы используете метку Azure.

## <a name="configure-and-connect-cisco-umbrella"></a>Настройка и подключение "Cisco"

Тег Cisco может интегрировать и экспортировать журналы непосредственно в Azure Sentinel.

1. В меню навигации меток Azure выберите **соединители данных**.

1. В коллекции **соединителей данных** выберите **Cisco (Предварительная версия)** , а затем **откройте страницу соединителя**.

1. Выполните действия, описанные в разделе **Настройка** на странице соединителя.

## <a name="find-your-data"></a>Поиск данных

После установки успешного подключения данные отображаются в **журналах** в разделе **CustomLogs** в одной или нескольких следующих таблицах:
- `Cisco_Umbrella_dns_CL`
- `Cisco_Umbrella_proxy_CL`
- `Cisco_Umbrella_ip_CL`
- `Cisco_Umbrella_cloudfirewall_CL`

## <a name="validate-connectivity"></a>Проверка подключения

После того, как журналы начнут отображаться в Log Analytics, может пройти до 20 минут.

## <a name="next-steps"></a>Дальнейшие действия

В этом документе вы узнали, как подключить данные "Cisco" к данным в Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.

- Узнайте, как [получить представление о данных и потенциальных угрозах](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.
