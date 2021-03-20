---
title: Подключение продуктов Forcepoint к Azure Sentinel | Документация Майкрософт
description: Узнайте, как подключить продукты Forcepoint к Azure Sentinel.
services: sentinel
author: yelevin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/20/2020
ms.author: yelevin
ms.openlocfilehash: eb099a786a84f9b7d0a6f0dc6e6df9c3459af295
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "77588235"
---
# <a name="connect-your-forcepoint-products-to-azure-sentinel"></a>Подключение продуктов Forcepoint к Azure Sentinel

> [!IMPORTANT]
> Соединитель данных продуктов Forcepoint в Azure Sentinel в настоящее время находится в общедоступной предварительной версии. Эта функция предоставляется без соглашения об уровне обслуживания и не рекомендуется для рабочих нагрузок. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).


В этой статье объясняется, как подключить продукты Forcepoint к Azure Sentinel. 

Соединители данных Forcepoint позволяют подключить брокер Forcepoint Cloud Access Security и журналы брандмауэра следующего поколения Forcepoint с помощью Sentinel Azure. Таким образом, можно автоматически экспортировать пользовательские журналы в метку Azure в режиме реального времени. Соединитель обеспечивает углубленную видимость действий пользователей, записанных продуктами Forcepoint. Кроме того, она обеспечивает дополнительную корреляцию с данными из рабочих нагрузок Azure и других веб-каналов, а также улучшает возможности мониторинга с помощью книг в Azure Sentinel.

> [!NOTE]
> Данные будут храниться в географическом расположении рабочей области, на которой вы используете метку Azure.



## <a name="forward-forcepoint-product-logs-to-the-syslog-agent"></a>Пересылка журналов продуктов Forcepoint в агент системного журнала 

Настройте продукт Forcepoint для пересылки сообщений Syslog в формате CEF в рабочую область Azure с помощью агента системного журнала.

1. Настройте продукт Forcepoint для интеграции с Azure Sentinel, как описано в следующих руководствах по установке:
 - [Руководством по интеграции Forcepoint КАСБ](https://frcpnt.com/casb-sentinel)
 - [Руководством по интеграции Forcepoint NGFW](https://frcpnt.com/ngfw-sentinel)

2. Выполните поиск CommonSecurityLog, чтобы использовать соответствующую схему в Log Analytics с именем Девицевендор содержит "Forcepoint". 

3. Перейдите к процедуре [Шаг 3. Проверка подключения](connect-cef-verify.md).



## <a name="next-steps"></a>Дальнейшие действия

В этом документе вы узнали, как подключить продукты Forcepoint к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.

- Узнайте, как [отслеживать свои данные и потенциальные угрозы](quickstart-get-visibility.md).

- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).

- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.