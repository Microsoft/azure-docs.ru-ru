---
title: Подключение данных Palo Alto Networks к Azure Sentinel | Документация Майкрософт
description: Узнайте, как использовать соединитель данных Palo Alto Networks для простого подключения журналов Palo Alto Networks к Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.assetid: a4b21d67-1a72-40ec-bfce-d79a8707b4e1
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/30/2019
ms.author: yelevin
ms.openlocfilehash: 245db436fc3216fe5c8c8f51c50c0ac03190f9eb
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85564555"
---
# <a name="connect-palo-alto-networks-to-azure-sentinel"></a>Подключение Palo Alto Networks к Azure Sentinel



В этой статье объясняется, как подключить устройство Palo Alto Networks к Azure Sentinel. Соединитель данных Palo Alto Networks позволяет легко подключать журналы Palo Alto Networks к Azure Sentinel, просматривать панели мониторинга, создавать пользовательские оповещения и улучшать исследование. Использование Palo Alto Networks в Azure Sentinel предоставит вам более подробные сведения об использовании Интернета в вашей организации и улучшит возможности ее работы. 


## <a name="forward-palo-alto-networks-logs-to-the-syslog-agent"></a>Пересылка журналов Palo Alto Networks в агент системного журнала

Настройте Palo Alto Networks для пересылки сообщений системного журнала в формате CEF в рабочую область Azure с помощью агента syslog:
1.  Перейдите в раздел "Общие сведения о [настройке формата событий (CEF)](https://docs.paloaltonetworks.com/resources/cef) " и скачайте PDF-файл для типа устройства. Следуйте всем инструкциям в этом разделе, чтобы настроить устройство Palo Alto Networks для получения сведений о событиях CEF. 

1.  Перейдите к разделу [Настройка мониторинга системного журнала](https://docs.paloaltonetworks.com/pan-os/8-1/pan-os-admin/monitoring/use-syslog-for-monitoring/configure-syslog-monitoring) и выполните шаги 2 и 3, чтобы настроить пересылку событий CEF с устройства Palo Alto Networks на метку Azure Sentinel.

    1. Убедитесь, что в качестве **формата syslog-сервера** выбрано **BSD**.

       > [!NOTE]
       > Операции копирования и вставки из PDF-файла могут изменить текст и вставить случайные символы. Чтобы избежать этого, скопируйте текст в редактор и удалите все символы, которые могут нарушить формат журнала, прежде чем вставлять его, как видно в этом примере.
 
        ![Проблема копирования текста CEF](./media/connect-cef/paloalto-text-prob1.png)

1. Чтобы использовать соответствующую схему в Log Analytics для событий Palo Alto Networks, выполните поиск по запросу **CommonSecurityLog**.

1. Перейдите к процедуре [Шаг 3. Проверка подключения](connect-cef-verify.md).




## <a name="next-steps"></a>Дальнейшие действия
В этом документе вы узнали, как подключить устройства Palo Alto Networks к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.
- Узнайте, как [отслеживать свои данные и потенциальные угрозы](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.


