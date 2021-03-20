---
title: Подключение службы защиты мобильных устройств от угроз Zimperium к Azure Sentinel | Документация Майкрософт
description: Узнайте, как подключить службу защиты мобильных устройств от угроз Zimperium к Azure Sentinel.
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
ms.openlocfilehash: 7cbf1c52af1d2902ae0726fc0dd98dbf12cecc44
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100097455"
---
# <a name="connect-your-zimperium-mobile-threat-defense-to-azure-sentinel"></a>Подключение средства защиты мобильных устройств от угроз Zimperium к Azure Sentinel


> [!IMPORTANT]
> Соединитель данных защиты мобильных устройств от угроз Zimperium в Azure Sentinel в настоящее время находится в общедоступной предварительной версии.
> Эта функция предоставляется без соглашения об уровне обслуживания и не рекомендуется для рабочих нагрузок. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).



Соединитель защиты мобильных устройств от угроз Zimperium позволяет подключаться к журналу угроз Zimperium с помощью Azure Sentinel для просмотра панелей мониторинга, создания пользовательских оповещений и улучшения расследования. Это позволяет получить более подробные сведения о корпоративной угрозе для мобильных устройств, а также расширяет возможности работы по обеспечению безопасности.

> [!NOTE]
> Данные будут храниться в географическом расположении рабочей области, на которой вы используете метку Azure.

## <a name="configure-and-connect-zimperium-mobile-threat-defense"></a>Настройка и подключение средства защиты мобильных устройств от угроз Zimperium

Служба защиты мобильных устройств от угроз Zimperium может интегрировать и экспортировать журналы непосредственно в **Azure Sentinel**.

1. На портале Sentinel Azure щелкните Data Connectors (соединители данных) и выберите **Zimperium мобильной защиты от угроз**.
2. Выберите **открыть страницу соединителя**.
3. Следуйте инструкциям на странице соединитель **защиты мобильных устройств от угроз Zimperium** , как показано ниже.
 1. В zConsole щелкните **Управление** на панели навигации.
 2. Щелкните вкладку **Integrations** (Интеграции).
 3. Нажмите кнопку **отчеты об угрозах** и нажмите кнопку **Добавить интеграции** .
 4. Создайте интеграцию, выбрав **Microsoft Azure Sentinel** из доступных интеграций и введя идентификатор рабочей области и первичный ключ для подключения к Azure Sentinel.
 5. Также доступна возможность выбора уровня фильтра для данных угроз для отправки в Azure Sentinel. 

4. Дополнительные сведения см. на [портале поддержки клиентов Zimperium](https://support.zimperium.com).


## <a name="find-your-data"></a>Поиск данных

После установки успешного подключения данные отображаются в Log Analytics в разделе CustomLogs ZimperiumThreatLog_CL и ZimperiumMitigationLog_CL.

Чтобы использовать соответствующую схему в Log Analytics для защиты мобильных устройств от угроз Zimperium, выполните поиск ZimperiumThreatLog_CL и ZimperiumMitigationLog_CL.


## <a name="validate-connectivity"></a>Проверка подключения

После того, как журналы начнут отображаться в Log Analytics, может пройти до 20 минут.

## <a name="next-steps"></a>Дальнейшие действия

В этом документе вы узнали, как подключить службу защиты мобильных устройств от угроз Zimperium к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.

- Узнайте, как [отслеживать свои данные и потенциальные угрозы](quickstart-get-visibility.md).

- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).

- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.

Дополнительные сведения о Zimperium см. в следующих статьях:

- [Zimperium](https://zimperium.com)

- [Блог по безопасности Zimperium Mobile](https://blog.zimperium.com)

- [Портал поддержки клиентов Zimperium](https://support.zimperium.com)

