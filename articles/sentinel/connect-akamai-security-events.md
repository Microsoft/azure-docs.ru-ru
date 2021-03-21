---
title: Подключение сборщика событий безопасности Akamai к Azure Sentinel | Документация Майкрософт
description: Узнайте, как использовать соединитель событий безопасности Akamai для извлечения журналов безопасности решений Akamai в Azure Sentinel. Просмотр данных Akamai в книгах, создание оповещений и улучшение расследования.
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
ms.openlocfilehash: 8aa5a52a06713b4f00b43205a57148049a8ef8da
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101711966"
---
# <a name="connect-your-akamai-security-events-collector-to-azure-sentinel"></a>Подключение сборщика событий безопасности Akamai к Azure Sentinel

> [!IMPORTANT]
> Соединитель событий безопасности Akamai сейчас находится на **этапе предварительной версии**. Ознакомьтесь с дополнительными [условиями использования Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) предварительных версий для дополнительных юридических условий, которые относятся к функциям Azure, которые доступны в бета-версии, предварительном просмотре или еще не выпущены в общедоступную версию.

В этой статье объясняется, как подключить сборщик событий безопасности Akamai к Azure Sentinel. Соединитель данных событий безопасности Akamai позволяет легко подключать журналы Akamai с помощью Sentinel Azure, чтобы можно было просматривать данные в книгах, запрашивать их для создания пользовательских оповещений и внедрять их для улучшения расследования. Интеграция между сборщиком событий безопасности Akamai и Azure Sentinel использует CEF системный журнал, сервер пересылки журналов на основе Linux и агент Log Analytics. В нем также используется настраиваемое средство синтаксического анализа журналов, основанное на функции Kusto.

> [!NOTE]
> Данные будут храниться в географическом расположении рабочей области, на которой вы используете метку Azure.

## <a name="prerequisites"></a>Предварительные условия

- У вас должны быть разрешения на чтение и запись в рабочей области Sentinel Azure.

- Необходимо иметь разрешения на чтение общих ключей для рабочей области. Дополнительные [сведения о ключах рабочих областей](../azure-monitor/agents/log-analytics-agent.md#workspace-id-and-key).

## <a name="send-akamai-security-events-logs-to-azure-sentinel"></a>Отправка журналов событий безопасности Akamai в Azure Sentinel

Чтобы получить журналы в Azure Sentinel, настройте сборщик событий безопасности Akamai для отправки сообщений Syslog в формате CEF на сервер пересылки журналов на основе Linux (под управлением rsyslog или syslog-ng). На этом сервере будет установлен агент Log Analytics, а агент перенаправит журналы в рабочую область Sentinel Azure.

1. В меню навигации меток Azure выберите **соединители данных**.

1. В коллекции **соединителей данных** выберите **события безопасности Akamai (Предварительная версия)**, а затем **откройте страницу соединителя**.

1. Следуйте инструкциям на вкладке **инструкции** в разделе **Конфигурация**.

    1. В **1. Конфигурация агента syslog для Linux** . Выполните этот шаг, если у вас еще нет запущенной службы пересылки журналов или если вам требуется другой сервер. Дополнительные сведения см. в разделе [Шаг 1. Развертывание сервера пересылки журналов](connect-cef-agent.md) в документации по Sentinel Azure для получения сведений о размере, более подробных инструкций и подробное описание.

    1. В разделе **2. Пересылка журналов типов распространенных событий (CEF) в агент системного журнала** . Следуйте инструкциям Akamai, чтобы [настроить интеграцию SIEM](https://developer.akamai.com/tools/integrations/siem) и [настроить соединитель CEF](https://developer.akamai.com/tools/integrations/siem/siem-cef-connector). Этот соединитель получает события безопасности из решений Akamai практически в реальном времени с помощью SIEM OPEN API и преобразует их из JSON в формат CEF.
    
        Эта конфигурация должна включать следующие элементы:
    
        - Назначение журнала — имя узла и/или IP-адрес сервера пересылки журналов.
        - Протокол и порт — TCP 514 (если в противном случае это не рекомендуется, обязательно выполните параллельное изменение в управляющей программе syslog на сервере пересылки журналов).
        - Формат журнала — CEF
        - Типы журналов — все доступные

    1. Ниже **3. Проверка подключения** . Проверка приема данных путем копирования команды на страницу соединителя и запуска ее на сервере пересылки журналов. Более подробные инструкции и пояснения см. в разделе [Шаг 3. Проверка подключения](connect-cef-verify.md) в документации Azure Sentinel.

        После того, как журналы начнут отображаться в Log Analytics, может пройти до 20 минут.

## <a name="find-your-data"></a>Поиск данных

После установки успешного подключения данные отображаются в **журналах** в разделе " **Sentinel** " в таблице *CommonSecurityLog* .

Этот соединитель данных зависит от средства синтаксического анализа, основанного на функции Kusto, чтобы работать должным образом. Выполните следующие действия, чтобы настроить функцию **акамаисиемевент** Kusto для использования в запросах и книгах.

1. В меню навигации меток Azure выберите **журналы**.

1. Скопируйте следующий запрос и вставьте его в окно запроса.
    ```kusto
    CommonSecurityLog 
    | where DeviceVendor == 'Akamai'
    | where DeviceProduct == 'akamai_siem'
    | extend EventVendor = 'Akamai'
    | extend EventProduct = 'akamai_siem'
    | extend EventProductVersion = '1.0'
    | extend EventId = DeviceEventClassID
    | extend EventCategory = Activity
    | extend EventSeverity = LogSeverity
    | extend DvcAction = DeviceAction
    | extend NetworkApplicationProtocol = ApplicationProtocol
    | extend Ipv6Src = DeviceCustomIPv6Address2
    | extend RuleName = DeviceCustomString1
    | extend RuleMessages = DeviceCustomString2
    | extend RuleData = DeviceCustomString3
    | extend RuleSelectors = DeviceCustomString4
    | extend ClientReputation = DeviceCustomString5
    | extend ApiId = DeviceCustomString6
    | extend RequestId = DevicePayloadId
    | extend DstDvcHostname = DestinationHostName
    | extend DstPortNumber = DestinationPort
    | extend ConfigId = FlexString1
    | extend PolicyId = FlexString2
    | extend NetworkBytes = SentBytes
    | extend UrlOriginal = RequestURL
    | extend HttpRequestMethod = RequestMethod
    | extend SrcIpAddr = SourceIP
    | extend EventStartTime = datetime(1970-01-01) + tolong(extract(@'.*start=(.*?);', 1, AdditionalExtensions)) * 1s 
    | extend SlowPostAction = extract(@'.*AkamaiSiemSlowPostAction=(.*?);', 1, AdditionalExtensions)
    | extend SlowPostRate = extract(@'.*AkamaiSiemSlowPostRate=(.*?);', 1, AdditionalExtensions)
    | extend RuleVersions = extract(@'.*AkamaiSiemRuleVersions=,?(.*?);', 1, AdditionalExtensions)
    | extend RuleTags = extract(@'.*AkamaiSiemRuleTags=(.*?);', 1, AdditionalExtensions)
    | extend ApiKey = extract(@'.*AkamaiSiemApiKey=(.*?);', 1, AdditionalExtensions)
    | extend Tls = extract(@'.*AkamaiSiemTLSVersion=(.*?);', 1, AdditionalExtensions)
    | extend RequestHeaders = extract(@'.*AkamaiSiemRequestHeaders=;?(.*?);', 1, AdditionalExtensions)
    | extend ResponseHeaders = extract(@'.*AkamaiSiemResponseHeaders=(.*?);', 1, AdditionalExtensions)
    | extend HttpStatusCode = extract(@'.*AkamaiSiemResponseStatus=(.*?);', 1, AdditionalExtensions)
    | extend GeoContinent = extract(@'.*AkamaiSiemContinent=(.*?);', 1, AdditionalExtensions)
    | extend SrcGeoCountry = extract(@'.*AkamaiSiemCountry=(.*?);', 1, AdditionalExtensions)
    | extend SrcGeoCity = extract(@'.*AkamaiSiemCity=(.*?);', 1, AdditionalExtensions)
    | extend SrcGeoRegion = extract(@'.*AkamaiSiemRegion=(.*?);', 1, AdditionalExtensions)
    | extend GeoAsn = extract(@'.*AkamaiSiemASN=(\d+)', 1, AdditionalExtensions)
    | extend Custom = extract(@'.*AkamaiSiemCusomData=(.*?)', 1, AdditionalExtensions)
    | project TimeGenerated
            , EventVendor
            , EventProduct
            , EventProductVersion
            , EventStartTime
            , EventId
            , EventCategory
            , EventSeverity
            , DvcAction
            , NetworkApplicationProtocol
            , Ipv6Src
            , RuleName
            , RuleMessages
            , RuleData
            , RuleSelectors
            , ClientReputation
            , ApiId
            , RequestId
            , DstDvcHostname
            , DstPortNumber
            , ConfigId
            , PolicyId
            , NetworkBytes
            , UrlOriginal
            , HttpRequestMethod
            , SrcIpAddr
            , SlowPostAction
            , SlowPostRate
            , RuleVersions
            , RuleTags
            , ApiKey
            , Tls
            , RequestHeaders
            , ResponseHeaders
            , HttpStatusCode
            , GeoContinent
            , SrcGeoCountry
            , SrcGeoCity
            , SrcGeoRegion
            , GeoAsn
            , Custom
    ```

1. Щелкните раскрывающийся список **сохранить** и нажмите кнопку **сохранить**. На панели " **сохранить** "

    1. В поле **имя** введите **акамаисиемевент**.

    1. В разделе **Сохранить как** выберите **функция**.

    1. В разделе **псевдоним функции** введите **акамаисиемевент**.

    1. В разделе **Категория** введите **функции**.

    1. Выберите команду **Сохранить**.

    Активация приложений функций обычно занимает от 10 до 15 минут.

Теперь вы можете запрашивать данные Akamai, вводя `AkamaiSIEMEvent` их в верхней строке окна запроса.

Дополнительные примеры запросов см. на вкладке **дальнейшие действия** на странице соединителя.

## <a name="next-steps"></a>Дальнейшие действия

В этом документе вы узнали, как подключить события безопасности Akamai к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.

- Узнайте, как [получить представление о данных и потенциальных угрозах](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.