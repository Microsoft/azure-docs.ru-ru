---
title: Подключение устройства шлюза Imperva WAF к Azure Sentinel | Документация Майкрософт
description: Узнайте, как использовать соединитель Imperva WAF Gateway для извлечения журналов Imperva WAF в Azure Sentinel. Просмотр данных Imperva WAF в книгах, создание оповещений и улучшение расследования.
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
ms.date: 02/03/2021
ms.author: yelevin
ms.openlocfilehash: a37abf369d1f34dc8f4a27802dfad88dab79be44
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101698439"
---
# <a name="connect-your-imperva-waf-gateway-appliance-to-azure-sentinel"></a>Подключение устройства шлюза Imperva WAF к Azure Sentinel

> [!IMPORTANT]
> Соединитель шлюза Imperva WAF в настоящее время находится на **этапе предварительной версии**. Ознакомьтесь с дополнительными [условиями использования Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) предварительных версий для дополнительных юридических условий, которые относятся к функциям Azure, которые доступны в бета-версии, предварительном просмотре или еще не выпущены в общедоступную версию.

В этой статье объясняется, как подключить устройство шлюза Imperva WAF к Azure Sentinel. Соединитель данных Imperva WAF Gateway позволяет легко подключать журналы шлюза Imperva WAF с помощью Sentinel Azure, чтобы можно было просматривать данные в книгах, использовать их для создания пользовательских оповещений и внедрять их для улучшения расследования. При интеграции между шлюзом Imperva WAF и Azure Sentinel используется CEF-отформатированный системный журнал, сервер пересылки журналов на основе Linux и агент Log Analytics.

> [!NOTE]
> Данные будут храниться в географическом расположении рабочей области, на которой вы используете метку Azure.

## <a name="prerequisites"></a>Предварительные условия

- У вас должны быть разрешения на чтение и запись в рабочей области Sentinel Azure.

- Необходимо иметь разрешения на чтение общих ключей для рабочей области. Дополнительные [сведения о ключах рабочих областей](../azure-monitor/agents/log-analytics-agent.md#workspace-id-and-key).

## <a name="send-imperva-waf-gateway-logs-to-azure-sentinel"></a>Отправка журналов шлюза Imperva WAF в Azure Sentinel

Чтобы получить журналы в Azure Sentinel, настройте устройство шлюза Imperva WAF для отправки сообщений Syslog в формате CEF на сервер пересылки журналов на основе Linux (под управлением rsyslog или syslog-ng). На этом сервере будет установлен агент Log Analytics, а агент перенаправит журналы в рабочую область Sentinel Azure.

1. В меню навигации меток Azure выберите **соединители данных**.

1. В коллекции **соединителей данных** выберите **Imperva WAF Gateway (Предварительная версия)**, а затем **откройте страницу соединителя**.

1. Следуйте инструкциям на вкладке **инструкции** в разделе **Конфигурация**.

    1. В **1. Конфигурация агента syslog для Linux** . Выполните этот шаг, если у вас еще нет запущенной службы пересылки журналов или если вам требуется другой сервер. Более подробные инструкции и пояснения см. в разделе [Шаг 1. Развертывание сервера пересылки журналов](connect-cef-agent.md) в документации по Sentinel Azure.

    1. В разделе **2. Пересылка журналов типов распространенных событий (CEF) в агент системного журнала** . для этого соединителя требуется, чтобы **интерфейс действия** и **набор действий** создавались в консоли управления **Imperva секуресфере MX** . Выполните инструкции Imperva, чтобы [включить регистрацию оповещений шлюза IMPERVA WAF в Azure Sentinel](https://community.imperva.com/blogs/craig-burlingame1/2020/11/13/steps-for-enabling-imperva-waf-gateway-alert). Эта конфигурация должна включать следующие элементы:
        - Назначение журнала — имя узла и/или IP-адрес сервера пересылки журналов.
        - Протокол и порт — TCP 514
        - Формат журнала — CEF
        - Типы журналов — все доступные

    1. Ниже **3. Проверка подключения** . Проверка приема данных путем копирования команды на страницу соединителя и запуска ее на сервере пересылки журналов. Более подробные инструкции и пояснения см. в разделе [Шаг 3. Проверка подключения](connect-cef-verify.md) в документации Azure Sentinel.

        После того, как журналы начнут отображаться в Log Analytics, может пройти до 20 минут.

## <a name="find-your-data"></a>Поиск данных

После установки успешного подключения данные отображаются в **журналах** в разделе " **Sentinel** " в таблице *CommonSecurityLog* .

Чтобы запросить данные шлюза Imperva WAF в Log Analytics, скопируйте следующий фрагмент в окно запроса, применяя другие фильтры по своему выбору:

```kusto
CommonSecurityLog 
| where DeviceVendor == "Imperva Inc." 
| where DeviceProduct == "WAF Gateway" 
| where TimeGenerated == ago(5m)
```

Более полезные примеры запросов см. на вкладке **дальнейшие действия** на странице соединителя.

## <a name="next-steps"></a>Дальнейшие действия
В этом документе вы узнали, как подключить шлюз Imperva WAF к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.

- Узнайте, как [получить представление о данных и потенциальных угрозах](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.