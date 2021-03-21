---
title: Подключение данных F5 ASM к Azure Sentinel | Документация Майкрософт
description: 'Узнайте, как использовать соединитель F5 ASM Data Connector для извлечения журналов ASM по клавише F5 в Azure Sentinel. Просмотр данных f # ASM в книгах, создание предупреждений и улучшенное исследование.'
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
ms.date: 05/20/2020
ms.author: yelevin
ms.openlocfilehash: 81a0f0e01827c48518f2eb1fba062e9e6536d9df
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94655703"
---
# <a name="connect-f5-asm-to-azure-sentinel"></a>Подключение F5 ASM к Azure Sentinel

В этой статье объясняется, как с помощью соединителя F5 ASM с легкостью извлечь журналы F5 ASM в Azure Sentinel. Это позволит просматривать данные F5 ASM в книгах, использовать их для создания настраиваемых оповещений и внедрять их для улучшения расследования. Наличие данных F5 ASM в Azure Sentinel позволяет получить более подробные сведения о безопасности веб-приложений вашей организации и улучшит возможности по обеспечению безопасности. 

## <a name="configure-your-f5-asm-to-send-cef-messages"></a>Настройка F5 ASM для отправки сообщений CEF

1. Следуйте инструкциям в разделе [Настройка F5 для ведения журналов событий безопасности приложений](https://techdocs.f5.com/kb/en-us/products/big-ip_asm/manuals/product/asm-implementations-11-5-0/12.html), чтобы настроить удаленное ведение журнала, следуя приведенным ниже инструкциям.
   - Для параметра **Тип удаленного хранилища** выберите значение **CEF**.
   - Для параметра **Протокол** выберите **TCP**.
   - Для параметра **IP-адрес** задайте IP-адрес сервера системного журнала.
   - В качестве **номера порта** задайте **514** либо укажите порт, используемый агентом.
   - Вы можете задать для параметра **Максимальный размер строки запроса** значение, совпадающее с уже настроенным для агента.

1. Чтобы использовать соответствующую схему в Log Analytics для событий CEF, выполните поиск по запросу `CommonSecurityLog`.

1. Перейдите к процедуре [Шаг 3. Проверка подключения](connect-cef-verify.md).


## <a name="next-steps"></a>Дальнейшие действия
В этом документе вы узнали, как подключить F5 ASM к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.
- Узнайте, как [отслеживать свои данные и потенциальные угрозы](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](./tutorial-detect-threats-built-in.md).
- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.