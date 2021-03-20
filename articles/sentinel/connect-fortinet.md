---
title: Подключение данных Fortinet к Azure Sentinel | Документация Майкрософт
description: Подключите устройство Fortinet к Sentinel Azure для просмотра панелей мониторинга, создания пользовательских оповещений и улучшения расследования.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.assetid: add92907-0d7c-42b8-a773-f570f2d705ff
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/30/2019
ms.author: yelevin
ms.openlocfilehash: 60be22f439547d006f54e489833b63171e617e3e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92913999"
---
# <a name="connect-fortinet-to-azure-sentinel"></a>Подключение Fortinet к Azure Sentinel



В этой статье объясняется, как подключить устройство Fortinet к Azure Sentinel. Соединитель данных Fortinet позволяет легко подключать журналы Fortinet с помощью Azure Sentinel, просматривать панели мониторинга, создавать пользовательские оповещения и улучшать исследование. Использование Fortinet в Azure Sentinel предоставит вам более подробные сведения об использовании Интернета в вашей организации и улучшит возможности ее работы. 


 
## <a name="forward-fortinet-logs-to-the-syslog-agent"></a>Пересылка журналов Fortinet в агент системного журнала

Настройте Fortinet для пересылки сообщений системного журнала в формате CEF в рабочую область Azure с помощью агента системного журнала.

1. Откройте интерфейс командной строки на устройстве Fortinet и выполните следующие команды:

    ```console
    config log syslogd setting
    set status enable
    set format cef
    set port 514
    set server <ip_address_of_Receiver>
    end
    ```

    - Замените **IP-адрес** сервера IP-адресом агента.
    - Задайте для **порта системного журнала** значение **514** или порт, установленный на агенте.
    - Чтобы включить формат CEF в ранних версиях Фортиос, может потребоваться выполнить команду Set **CSV Disabled**.
 
   > [!NOTE] 
   > Для получения дополнительных сведений перейдите в [библиотеку документов Fortinet](https://aka.ms/asi-syslog-fortinet-fortinetdocumentlibrary). Выберите свою **версию и воспользуйтесь** **справочной ссылкой и сообщением журнала**.

1. Чтобы использовать соответствующую схему в Azure Monitor Log Analytics для событий Fortinet, выполните поиск по запросу `CommonSecurityLog` .

1. Перейдите к процедуре [Шаг 3. Проверка подключения](connect-cef-verify.md).


## <a name="next-steps"></a>Дальнейшие действия
Из этой статьи вы узнали, как подключить устройства Fortinet к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.
- Узнайте, как [отслеживать свои данные и потенциальные угрозы](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.


