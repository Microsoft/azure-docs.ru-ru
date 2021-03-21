---
title: Подключение Juniper Networks SRX Data к Azure Sentinel | Документация Майкрософт
description: Узнайте, как использовать соединитель данных Juniper SRX для извлечения журналов Juniper SRX в Azure Sentinel. Просмотр данных Juniper SRX в книгах, создание оповещений и улучшение расследования.
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
ms.date: 01/17/2021
ms.author: yelevin
ms.openlocfilehash: b10c47a31bf1be10c278d4d9e0dce633bc7bff6c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100530643"
---
# <a name="connect-your-juniper-srx-firewall-to-azure-sentinel"></a>Подключение брандмауэра Juniper SRX к Azure Sentinel

> [!IMPORTANT]
> Соединитель Juniper SRX в настоящее время находится на **этапе предварительной версии**. Ознакомьтесь с дополнительными [условиями использования Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) предварительных версий для дополнительных юридических условий, которые относятся к функциям Azure, которые доступны в бета-версии, предварительном просмотре или еще не выпущены в общедоступную версию.

В этой статье объясняется, как подключить устройство брандмауэра Juniper SRX к Azure Sentinel. Соединитель данных Juniper SRX позволяет легко подключать журналы SRX с помощью Sentinel Azure, чтобы можно было просматривать данные в книгах, использовать их для создания пользовательских оповещений и внедрять их для улучшения расследования. При интеграции между Juniper SRX и Sentinel Azure используется системный журнал.

> [!NOTE]
> Данные будут храниться в географическом расположении рабочей области, на которой вы используете метку Azure.

## <a name="prerequisites"></a>Предварительные условия

- У вас должно быть разрешение на чтение и запись в рабочей области "Sentinel" Azure.

- Решение Juniper SRX должно быть настроено для экспорта журналов через системный журнал.

## <a name="forward-juniper-srx-logs-to-the-syslog-agent"></a>Пересылка журналов Juniper SRX в агент системного журнала  

Настройте Juniper SRX для пересылки сообщений Syslog в рабочую область Sentinel Azure с помощью агента системного журнала.

1. В меню навигации меток Azure выберите **соединители данных**.

1. В коллекции **соединителей данных** выберите соединитель **Juniper SRX (Предварительная версия)** , а затем **откройте страницу соединителя**.

1. Следуйте инструкциям на странице соединителя **JUNIPER SRX** :

    1. Установка и подключение агента для Linux

        - Выберите виртуальную машину Linux в Azure или компьютер, не относящийся к Azure Linux (физический или виртуальный).

    1. Настройка собираемых журналов

        - Выберите средства и серьезность в конфигурации агентов рабочей области.

    1. Настройка и подключение Juniper SRX

        - Выполните эти инструкции, чтобы настроить Juniper SRX для пересылки системного журнала.
            - [Журналы трафика (журналы политики безопасности)](https://kb.juniper.net/InfoCenter/index?page=content&id=KB16509&actp=METADATA)
            - [Системные журналы](https://kb.juniper.net/InfoCenter/index?page=content&id=kb16502)
        - Для удаленного сервера используйте IP-адрес компьютера Linux, на котором установлен агент Linux.

## <a name="find-your-data"></a>Поиск данных

После установки успешного подключения данные отображаются в Log Analytics в разделе syslog.

Полезные примеры запросов см. на вкладке **дальнейшие действия** на странице соединителя.

## <a name="validate-connectivity"></a>Проверка подключения

После того, как журналы начнут отображаться в Log Analytics, может пройти до 20 минут.

## <a name="next-steps"></a>Дальнейшие действия

В этом документе вы узнали, как подключить Juniper SRX к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.

- Узнайте, как [отслеживать свои данные и потенциальные угрозы](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.
