---
title: Подключение VMware ESXi данных к Azure Sentinel | Документация Майкрософт
description: Узнайте, как использовать соединитель данных VMware ESXi для извлечения журналов ESXi в Azure Sentinel. Просмотр данных ESXi в книгах, создание оповещений и улучшение расследования.
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
ms.date: 03/01/2021
ms.author: yelevin
ms.openlocfilehash: 3d478a9ac3cf91f3f6815859b8534efff88f07b1
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101700929"
---
# <a name="connect-your-vmware-esxi-to-azure-sentinel"></a>Подключение VMware ESXi к Azure Sentinel

> [!IMPORTANT]
> Соединитель VMware ESXi в настоящее время находится на **этапе предварительной версии**. Ознакомьтесь с дополнительными [условиями использования Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) предварительных версий для дополнительных юридических условий, которые относятся к функциям Azure, которые доступны в бета-версии, предварительном просмотре или еще не выпущены в общедоступную версию.

В этой статье объясняется, как подключить устройство VMware ESXi к Azure Sentinel. Соединитель данных VMware ESXi позволяет легко принимать журналы VMware ESXi в Azure Sentinel, что позволяет получить более подробные сведения о действиях ESXi в Организации и помочь улучшить возможности обеспечения безопасности. Интеграция между VMware ESXi и Azure Sentinel позволяет использовать сервер syslog с установленным агентом Log Analytics. В нем также используется настраиваемое средство синтаксического анализа журналов, основанное на функции Kusto.

> [!NOTE]
> Данные будут храниться в географическом расположении рабочей области, на которой вы используете метку Azure.

## <a name="prerequisites"></a>Предварительные условия

- У вас должно быть разрешение на чтение и запись в рабочей области "Sentinel" Azure.

- Решение VMware ESXi должно быть настроено для экспорта журналов через системный журнал.

## <a name="send-vmware-esxi-logs-to-the-syslog-agent"></a>Отправка журналов VMware ESXi в агент системного журнала  

Настройте VMware ESXi для пересылки сообщений Syslog в рабочую область Sentinel Azure с помощью агента системного журнала.
3. Следуйте инструкциям на странице **VMware ESXi** .


1. В меню навигации меток Azure выберите **соединители данных**.

1. В коллекции **соединителей данных** выберите соединитель **VMware ESXi (Предварительная версия)** , а затем **откройте страницу соединителя**.

1. Следуйте инструкциям на странице соединителя **VMware ESXi** :

    1. Установка и подключение агента для Linux

        - Выберите виртуальную машину Linux в Azure или компьютер, не относящийся к Azure Linux (физический или виртуальный).

    1. Настройка собираемых журналов

        - Выберите средства и серьезность в **конфигурации агентов рабочей области**.

    1. Настройка и подключение VMware ESXi

        - Выполните следующие инструкции, чтобы настроить VMware ESXi для пересылки syslog:
            - [VMware ESXi 3,5 и 4. x](https://kb.vmware.com/s/article/1016621)
            - [VMware ESXi 5,0 и более поздних версий](https://docs.vmware.com/en/VMware-vSphere/5.5/com.vmware.vsphere.monitoring.doc/GUID-9F67DB52-F469-451F-B6C8-DAE8D95976E7.html)

            Для удаленного сервера используйте IP-адрес компьютера Linux, на котором установлен агент Linux.

## <a name="validate-connectivity-and-find-your-data"></a>Проверка подключения и поиск данных

После того, как журналы начнут отображаться в Azure Sentinel, может пройти до 20 минут. 

После установки успешного подключения данные отображаются в **журналах** в разделе " *Управление журналами* " в таблице *syslog* .

Этот соединитель данных зависит от средства синтаксического анализа, основанного на функции Kusto, чтобы работать должным образом. [Выполните следующие действия](https://aka.ms/sentinel-vmwareesxi-parser) , чтобы создать псевдоним функции **вмварискси** Kusto. Затем можно ввести `VMwareESXi` любое окно запроса в Sentinel в Azure, чтобы запросить данные.

Некоторые полезные примеры запросов см. на вкладке **дальнейшие действия** на странице соединителя.

## <a name="next-steps"></a>Дальнейшие действия

В этом документе вы узнали, как подключить VMware ESXi к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.

- Узнайте, как [получить представление о данных и потенциальных угрозах](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.
