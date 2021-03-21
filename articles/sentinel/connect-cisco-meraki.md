---
title: Подключение данных Cisco Meraki к Azure Sentinel | Документация Майкрософт
description: Узнайте, как использовать соединитель Cisco Meraki для извлечения журналов Cisco Meraki в Azure Sentinel. Просмотр данных Cisco Meraki в книгах, создание оповещений и улучшение расследования.
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
ms.date: 02/28/2021
ms.author: yelevin
ms.openlocfilehash: dc2d2a0724f18a02a5b667eb1004963a326ec360
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101747086"
---
# <a name="connect-your-cisco-meraki-to-azure-sentinel"></a>Подключение Meraki Cisco к Azure Sentinel

> [!IMPORTANT]
> Соединитель Cisco Meraki сейчас находится на **этапе предварительной версии**. Ознакомьтесь с дополнительными [условиями использования Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) предварительных версий для дополнительных юридических условий, которые относятся к функциям Azure, которые доступны в бета-версии, предварительном просмотре или еще не выпущены в общедоступную версию.

В этой статье объясняется, как подключить устройства Cisco Meraki (MX/MS/MR) к Azure Sentinel. Соединитель данных Cisco Meraki позволяет легко подключать журналы Cisco Meraki к Azure Sentinel, просматривать панели мониторинга, создавать пользовательские оповещения и улучшать исследование. При интеграции между Cisco Meraki и Azure Sentinel используется сервер syslog с установленным агентом Log Analytics. В нем также используется настраиваемое средство синтаксического анализа журналов, основанное на функции Kusto.

> [!NOTE]
> Данные будут храниться в географическом расположении рабочей области, на которой вы используете метку Azure.

## <a name="prerequisites"></a>Предварительные условия

- У вас должно быть разрешение на чтение и запись в рабочей области "Sentinel" Azure.

- Решение Cisco Meraki должно быть настроено для экспорта журналов через системный журнал.

## <a name="send-cisco-meraki-logs-to-azure-sentinel-via-the-syslog-agent"></a>Отправка журналов Cisco Meraki в Azure Sentinel с помощью агента системного журнала  

Настройте Cisco Meraki для пересылки сообщений Syslog в рабочую область Sentinel Azure с помощью агента системного журнала.

1. В меню навигации меток Azure выберите **соединители данных**.

1. В коллекции **соединителей данных** выберите соединитель **Cisco Meraki (Предварительная версия)** , а затем **откройте страницу соединителя**.

1. Следуйте инструкциям на странице соединителя **Cisco Meraki** :

    1. Установка и подключение агента для Linux

        - Выберите виртуальную машину Linux в Azure или компьютер, не относящийся к Azure Linux (физический или виртуальный).

    1. Настройка собираемых журналов

        - Выберите средства и серьезность в **конфигурации агентов рабочей области**.

    1. Настройка и подключение устройств Cisco Meraki

        - Выполните [эти инструкции](https://documentation.meraki.com/General_Administration/Monitoring_and_Reporting/Meraki_Device_Reporting_-_Syslog%2C_SNMP_and_API) , чтобы настроить устройства Cisco Meraki для пересылки системного журнала. Для удаленного сервера используйте IP-адрес компьютера Linux, на котором установлен агент Linux.

## <a name="validate-connectivity-and-find-your-data"></a>Проверка подключения и поиск данных

После того, как журналы начнут отображаться в Azure Sentinel, может пройти до 20 минут. 

После установки успешного подключения данные отображаются в **журналах** в разделе " *Управление журналами* " в таблице *syslog* .

Этот соединитель данных зависит от средства синтаксического анализа, основанного на функции Kusto, чтобы работать должным образом. [Выполните следующие действия](https://aka.ms/sentinel-ciscomeraki-parser) , чтобы создать псевдоним функции **Цискомераки** Kusto. Затем можно ввести `CiscoMeraki` окно запроса, чтобы запросить данные.

Некоторые полезные примеры запросов см. на вкладке **дальнейшие действия** на странице соединителя.

## <a name="next-steps"></a>Дальнейшие действия

В этом документе вы узнали, как подключить Cisco Meraki к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.

- Узнайте, как [получить представление о данных и потенциальных угрозах](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.
