---
title: Подключение данных системы унифицированных вычислений Cisco (UCS) к Azure Sentinel | Документация Майкрософт
description: Узнайте, как использовать соединитель данных Cisco UCS для извлечения журналов Cisco UCS в Azure Sentinel. Просмотр данных Cisco UCS в книгах, создание оповещений и улучшение расследования.
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
ms.openlocfilehash: 15e31b8dc5ac6db5861e3ea0fb5390ddf0a8c945
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100530677"
---
# <a name="connect-your-cisco-unified-computing-system-ucs-to-azure-sentinel"></a>Подключение системы обработки Объединенных вычислений (UCS) Cisco к Azure Sentinel

> [!IMPORTANT]
> Соединитель для Cisco UCS сейчас находится на **этапе предварительной версии**. Ознакомьтесь с дополнительными [условиями использования Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) предварительных версий для дополнительных юридических условий, которые относятся к функциям Azure, которые доступны в бета-версии, предварительном просмотре или еще не выпущены в общедоступную версию.

В этой статье объясняется, как подключить устройство UCS для системы Microsoft Cisco к Sentinel. Соединитель данных Microsoft Cisco UCS позволяет легко подключать свои журналы UCS к Azure Sentinel, чтобы можно было просматривать данные в книгах, использовать их для создания пользовательских оповещений и внедрять их для улучшения расследования. При интеграции между Cisco UCS и Sentinel в Azure используется системный журнал.

> [!NOTE]
> Данные будут храниться в географическом расположении рабочей области, на которой вы используете метку Azure.

## <a name="prerequisites"></a>Предварительные условия

- У вас должно быть разрешение на чтение и запись в рабочей области "Sentinel" Azure.

- Решение Cisco UCS должно быть настроено для экспорта журналов через системный журнал.

## <a name="forward-cisco-ucs-logs-to-the-syslog-agent"></a>Пересылка журналов Cisco UCS в агент системного журнала  

Настройте Cisco UCS для пересылки сообщений Syslog в рабочую область Sentinel Azure с помощью агента системного журнала.

1. В меню навигации меток Azure выберите **соединители данных**.

1. В коллекции **соединителей данных** выберите соединитель **Cisco UCS (Предварительная версия)** , а затем **откройте страницу соединителя**.

1. Следуйте инструкциям на странице соединителя **Cisco UCS** :

    1. Установка и подключение агента для Linux

        - Выберите виртуальную машину Linux в Azure или компьютер, не относящийся к Azure Linux (физический или виртуальный).

    1. Настройка собираемых журналов

        - Выберите средства и серьезность в конфигурации агентов рабочей области.

    1. Настройка и подключение Cisco UCS

        - Выполните [эти инструкции](https://www.cisco.com/c/en/us/support/docs/servers-unified-computing/ucs-manager/110265-setup-syslog-for-ucs.html#configsremotesyslog) , чтобы настроить для сервера Cisco UCS пересылку системного журнала. Для удаленного сервера используйте IP-адрес компьютера Linux, на котором установлен агент Linux.

## <a name="find-your-data"></a>Поиск данных

После установки успешного подключения данные отображаются в Log Analytics в разделе syslog.

Полезные примеры запросов см. на вкладке **дальнейшие действия** на странице соединителя.

## <a name="validate-connectivity"></a>Проверка подключения

После того, как журналы начнут отображаться в Log Analytics, может пройти до 20 минут.

## <a name="next-steps"></a>Дальнейшие действия

В этом документе вы узнали, как подключить Cisco UCS к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.

- Узнайте, как [отслеживать свои данные и потенциальные угрозы](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.
