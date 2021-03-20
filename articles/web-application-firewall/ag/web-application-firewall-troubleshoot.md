---
title: Устранение неполадок с брандмауэром веб-приложения Azure
description: В этой статье содержатся сведения об устранении неполадок брандмауэра веб-приложения (WAF) для шлюза приложений Azure.
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.date: 11/14/2019
ms.author: ant
ms.topic: conceptual
ms.openlocfilehash: 483d261a8cc107d01cfb7a405eac43667d7efcc6
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92131842"
---
# <a name="troubleshoot-web-application-firewall-waf-for-azure-application-gateway"></a>Устранение неполадок брандмауэра веб-приложения (WAF) для шлюза приложений Azure

Существует несколько действий, которые можно выполнить, если запросы, которые должны пройти через брандмауэр веб-приложения (WAF), заблокированы.

Сначала убедитесь, что вы прочитали [Общие сведения о WAF](ag-overview.md) и документах [конфигурации WAF](application-gateway-waf-configuration.md) . Кроме того, убедитесь, что вы включили [мониторинг WAF](../../application-gateway/application-gateway-diagnostics.md) . в этих статьях объясняется, как работают функции WAF, принципы работы НАБОРОВ правил WAF и как получить доступ к журналам WAF.

Наборы правил OWASP предназначены для более строгого использования, а также для настройки в соответствии с конкретными потребностями приложения или организации с помощью WAF. Он полностью нормальен и, фактически, ожидается во многих случаях, для создания исключений, настраиваемых правил и даже для отключения правил, которые могут вызывать проблемы или ложные срабатывания. Политики "на сайт" и "на-универсальный код ресурса" позволяют вносить эти изменения только на определенные сайты или URI, поэтому любые изменения не должны повлиять на другие сайты, на которых может не работать одна и та же проблема. 

## <a name="understanding-waf-logs"></a>Основные сведения о журналах WAF

Журналы WAF предназначены для отображения всех запросов, соответствующих или заблокированных WAF. Это Главная книга всех проверенных запросов, которые сопоставлены или заблокированы. Если вы заметили, что WAF блокирует запрос, который не должен (ложный положительный результат), можно выполнить несколько действий. Сначала Ограничьте и найдите конкретный запрос. Просмотрите журналы, чтобы найти конкретный URI, метку времени или идентификатор транзакции запроса. При обнаружении связанных записей журнала можно начать действовать с ложными срабатываниями.

Например, предположим, что имеется допустимый трафик, содержащий строку *1 = 1* , которую вы хотите передать через WAF. При попытке выполнить запрос WAF блокирует трафик, содержащий строку *1 = 1* , в любом параметре или поле. Это строка, часто связанная с атакой путем внедрения кода SQL. Можно просмотреть журналы и просмотреть отметку времени запроса и правила, которые были заблокированы или сопоставлены.

В следующем примере можно увидеть, что четыре правила активируются во время одного запроса (с помощью поля ИД транзакции). Первый сообщает, что он соответствует, так как пользователь использовал числовой или IP-адрес для запроса, что увеличивает показатель аномалии на три, так как это предупреждение. Следующее совпадающее правило — 942130, которое нужно найти. В поле можно увидеть *1 = 1* `details.data` . Это еще больше увеличивает показатель аномалии на три, так как это также предупреждение. Как правило, каждое правило с **соответствующим** действием увеличивает показатель аномалии, и на этом этапе показатель аномалии был бы шестью. Дополнительные сведения см. в разделе [режим оценки аномалий](ag-overview.md#anomaly-scoring-mode).

Последние две записи журнала показывают, что запрос был заблокирован, так как показатель аномалии был достаточно высоким. Эти записи имеют разное действие, отличное от двух других. Они показывают, что они действительно *блокировали* запрос. Эти правила являются обязательными и не могут быть отключены. Они не должны рассматриваться как правила, но больше, чем базовая инфраструктура внутренних WAF.

```json
{ 
    "resourceId": "/SUBSCRIPTIONS/A6F44B25-259E-4AF5-888A-386FED92C11B/RESOURCEGROUPS/DEMOWAF_V2/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/DEMOWAF-V2", 
    "operationName": "ApplicationGatewayFirewall", 
    "category": "ApplicationGatewayFirewallLog", 
    "properties": { 
        "instanceId": "appgw_3", 
        "clientIp": "167.220.2.139", 
        "clientPort": "", 
        "requestUri": "\/", 
        "ruleSetType": "OWASP_CRS", 
        "ruleSetVersion": "3.0.0", 
        "ruleId": "920350", 
        "message": "Host header is a numeric IP address", 
        "action": "Matched", 
        "site": "Global", 
        "details": { 
            "message": "Warning. Pattern match \\\"^[\\\\\\\\d.:]+$\\\" at REQUEST_HEADERS:Host. ", 
            "data": "40.90.218.160", 
            "file": "rules\/REQUEST-920-PROTOCOL-ENFORCEMENT.conf\\\"", 
            "line": "791" 
        }, 
        "hostname": "vm000003", 
        "transactionId": "AcAcAcAcAKH@AcAcAcAcAyAt" 
    } 
} 
{ 
    "resourceId": "/SUBSCRIPTIONS/A6F44B25-259E-4AF5-888A-386FED92C11B/RESOURCEGROUPS/DEMOWAF_V2/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/DEMOWAF-V2", 
    "operationName": "ApplicationGatewayFirewall", 
    "category": "ApplicationGatewayFirewallLog", 
    "properties": { 
        "instanceId": "appgw_3", 
        "clientIp": "167.220.2.139", 
        "clientPort": "", 
        "requestUri": "\/", 
        "ruleSetType": "OWASP_CRS", 
        "ruleSetVersion": "3.0.0", 
        "ruleId": "942130", 
        "message": "SQL Injection Attack: SQL Tautology Detected.", 
        "action": "Matched", 
        "site": "Global", 
        "details": { 
            "message": "Warning. Pattern match \\\"(?i:([\\\\\\\\s'\\\\\\\"`\\\\\\\\(\\\\\\\\)]*?)([\\\\\\\\d\\\\\\\\w]++)([\\\\\\\\s'\\\\\\\"`\\\\\\\\(\\\\\\\\)]*?)(?:(?:=|\\u003c=\\u003e|r?like|sounds\\\\\\\\s+like|regexp)([\\\\\\\\s'\\\\\\\"`\\\\\\\\(\\\\\\\\)]*?)\\\\\\\\2|(?:!=|\\u003c=|\\u003e=|\\u003c\\u003e|\\u003c|\\u003e|\\\\\\\\^|is\\\\\\\\s+not|not\\\\\\\\s+like|not\\\\\\\\s+regexp)([\\\\\\\\s'\\\\\\\"`\\\\\\\\(\\\\\\\\)]*?)(?!\\\\\\\\2)([\\\\\\\\d\\\\\\\\w]+)))\\\" at ARGS:text1. ", 
            "data": "Matched Data: 1=1 found within ARGS:text1: 1=1", 
            "file": "rules\/REQUEST-942-APPLICATION-ATTACK-SQLI.conf\\\"", 
            "line": "554" 
        }, 
        "hostname": "vm000003", 
        "transactionId": "AcAcAcAcAKH@AcAcAcAcAyAt" 
    } 
} 
{ 
    "resourceId": "/SUBSCRIPTIONS/A6F44B25-259E-4AF5-888A-386FED92C11B/RESOURCEGROUPS/DEMOWAF_V2/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/DEMOWAF-V2", 
    "operationName": "ApplicationGatewayFirewall", 
    "category": "ApplicationGatewayFirewallLog", 
    "properties": { 
        "instanceId": "appgw_3", 
        "clientIp": "167.220.2.139", 
        "clientPort": "", 
        "requestUri": "\/", 
        "ruleSetType": "", 
        "ruleSetVersion": "", 
        "ruleId": "0", 
        "message": "Mandatory rule. Cannot be disabled. Inbound Anomaly Score Exceeded (Total Score: 8)", 
        "action": "Blocked", 
        "site": "Global", 
        "details": { 
            "message": "Access denied with code 403 (phase 2). Operator GE matched 5 at TX:anomaly_score. ", 
            "data": "", 
            "file": "rules\/REQUEST-949-BLOCKING-EVALUATION.conf\\\"", 
            "line": "57" 
        }, 
        "hostname": "vm000003", 
        "transactionId": "AcAcAcAcAKH@AcAcAcAcAyAt" 
    } 
} 
{ 
    "resourceId": "/SUBSCRIPTIONS/A6F44B25-259E-4AF5-888A-386FED92C11B/RESOURCEGROUPS/DEMOWAF_V2/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/DEMOWAF-V2", 
    "operationName": "ApplicationGatewayFirewall", 
    "category": "ApplicationGatewayFirewallLog", 
    "properties": { 
        "instanceId": "appgw_3", 
        "clientIp": "167.220.2.139", 
        "clientPort": "", 
        "requestUri": "\/", 
        "ruleSetType": "", 
        "ruleSetVersion": "", 
        "ruleId": "0", 
        "message": "Mandatory rule. Cannot be disabled. Inbound Anomaly Score Exceeded (Total Inbound Score: 8 - SQLI=5,XSS=0,RFI=0,LFI=0,RCE=0,PHPI=0,HTTP=0,SESS=0): SQL Injection Attack: SQL Tautology Detected.", 
        "action": "Blocked", 
        "site": "Global", 
        "details": { 
            "message": "Warning. Operator GE matched 5 at TX:inbound_anomaly_score. ", 
            "data": "", 
            "file": "rules\/RESPONSE-980-CORRELATION.conf\\\"", 
            "line": "73" 
        }, 
        "hostname": "vm000003", 
        "transactionId": "AcAcAcAcAKH@AcAcAcAcAyAt" 
    }
}
```

## <a name="fixing-false-positives"></a>Устранение ложных срабатываний

С этой информацией, и знанием правила 942130, которое соответствует строке *1 = 1* , можно выполнить несколько действий, чтобы предотвратить блокировку трафика.

- Использование списка исключений

   Дополнительные сведения о списках исключений см. в разделе [WAF Configuration](application-gateway-waf-configuration.md#waf-exclusion-lists) .
- Отключите правило.

### <a name="using-an-exclusion-list"></a>Использование списка исключений

Чтобы принять обоснованное решение об обработке ложного срабатывания, важно ознакомиться с технологиями, используемыми приложением. Например, предположим, что в вашем технологическом стеке нет SQL Server, и вы получаете ложные срабатывания, связанные с этими правилами. Отключение этих правил не обязательно приводит к ослаблению безопасности.

Одним из преимуществ использования списка исключений является отключение только определенной части запроса. Однако это означает, что определенное исключение применяется ко всему трафику, проходящему через WAF, так как это глобальный параметр. Например, это может привести к возникновению проблемы, если *1 = 1* является допустимым запросом в тексте для определенного приложения, но не для других. Еще одним преимуществом является возможность выбора между телом, заголовками и файлами cookie, которые будут исключены при выполнении определенного условия, в отличие от исключения всего запроса.

Иногда бывают случаи, когда определенные параметры передаются в WAF способом, который не может быть интуитивно понятным. Например, существует токен, который передается при проверке подлинности с помощью Azure Active Directory. Этот маркер, *__RequestVerificationToken*, обычно передается в качестве файла Cookie запроса. Однако в некоторых случаях, когда файлы cookie отключены, этот маркер также передается как атрибут запроса или "arg". В этом случае необходимо убедиться, что *__RequestVerificationToken* добавляется в список исключений как и **имя атрибута запроса** .

![Исключения](../media/web-application-firewall-troubleshoot/exclusion-list.png)

В этом примере необходимо исключить **имя атрибута запроса** , которое равно *Text1*. Это очевидно, поскольку имя атрибута можно увидеть в журналах брандмауэра: **данные: сопоставленные данные: 1 = 1 найдено в аргументах args: Текст1:1 = 1**. Атрибут — **Текст1**. Кроме того, имя этого атрибута можно найти несколькими другими способами. см. раздел [Поиск имен атрибутов запросов](#finding-request-attribute-names).

![Списки исключений WAF](../media/web-application-firewall-troubleshoot/waf-config.png)

### <a name="disabling-rules"></a>Отключение правил

Еще один способ обойти ложное срабатывание — отключить правило, совпадающее с входными данными, на которые WAF считался злоумышленник. Так как вы проанализировали журналы WAF и изменили правило до 942130, вы можете отключить его в портал Azure. См. раздел [Настройка правил брандмауэра веб-приложения с помощью портал Azure](application-gateway-customize-waf-rules-portal.md).

Одним из преимуществ отключения правила является то, что если известно, что весь трафик, содержащий определенное условие, который обычно блокируется, является допустимым трафиком, можно отключить это правило для всей WAF. Однако если в определенном варианте использования действует только трафик, вы открываете уязвимость, отключая это правило для всего WAF, так как это глобальный параметр.

Если вы хотите использовать Azure PowerShell, см. статью [Настройка правил брандмауэра веб-приложения с помощью PowerShell](application-gateway-customize-waf-rules-powershell.md). Если вы хотите использовать Azure CLI, см. раздел [Настройка правил брандмауэра веб-приложения с помощью Azure CLI](application-gateway-customize-waf-rules-cli.md).

![Правила WAF](../media/web-application-firewall-troubleshoot/waf-rules.png)

## <a name="finding-request-attribute-names"></a>Поиск имен атрибутов запроса

С помощью [Fiddler](https://www.telerik.com/fiddler)вы изучаете отдельные запросы и определяете, какие именно поля веб-страницы вызываются. Это может помочь исключить определенные поля из проверки с помощью списков исключений.

В этом примере видно, что поле, в котором была введена строка *1 = 1* , называется **Text1**.

:::image type="content" source="../media/web-application-firewall-troubleshoot/fiddler-1.png" alt-text="Снимок экрана с веб-отладчиком Progress Telerik Fiddler. На вкладке необработанное значение 1 = 1 отображается после имени Text1." border="false":::

Это поле, которое можно исключить. Дополнительные сведения о списках исключений см. в статье [ограничения размера запроса брандмауэра веб-приложения и списки исключений](application-gateway-waf-configuration.md#waf-exclusion-lists). Вы можете исключить оценку в этом случае, настроив следующие исключения:

![Исключение WAF](../media/web-application-firewall-troubleshoot/waf-exclusion-02.png)

Вы также можете просмотреть журналы брандмауэра, чтобы получить сведения, необходимые для добавления в список исключений. Сведения о включении ведения журнала см. в статье [работоспособность серверной части, журналы ресурсов и метрики для шлюза приложений](../../application-gateway/application-gateway-diagnostics.md).

Просмотрите журнал брандмауэра и просмотрите PT1H.jsв файле в течение часа, в течение которого был получен запрос, который требуется проверить.

В этом примере можно увидеть, что у вас есть четыре правила с одним и тем же ИД транзакции, и все они были выполнены в одно и то же время:

```json
-   {
-       "resourceId": "/SUBSCRIPTIONS/A6F44B25-259E-4AF5-888A-386FED92C11B/RESOURCEGROUPS/DEMOWAF_V2/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/DEMOWAF-V2",
-       "operationName": "ApplicationGatewayFirewall",
-       "category": "ApplicationGatewayFirewallLog",
-       "properties": {
-           "instanceId": "appgw_3",
-           "clientIp": "167.220.2.139",
-           "clientPort": "",
-           "requestUri": "\/",
-           "ruleSetType": "OWASP_CRS",
-           "ruleSetVersion": "3.0.0",
-           "ruleId": "920350",
-           "message": "Host header is a numeric IP address",
-           "action": "Matched",
-           "site": "Global",
-           "details": {
-               "message": "Warning. Pattern match \\\"^[\\\\\\\\d.:]+$\\\" at REQUEST_HEADERS:Host. ",
-               "data": "40.90.218.160",
-               "file": "rules\/REQUEST-920-PROTOCOL-ENFORCEMENT.conf\\\"",
-               "line": "791"
-           },
-           "hostname": "vm000003",
-           "transactionId": "AcAcAcAcAKH@AcAcAcAcAyAt"
-       }
-   }
-   {
-       "resourceId": "/SUBSCRIPTIONS/A6F44B25-259E-4AF5-888A-386FED92C11B/RESOURCEGROUPS/DEMOWAF_V2/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/DEMOWAF-V2",
-       "operationName": "ApplicationGatewayFirewall",
-       "category": "ApplicationGatewayFirewallLog",
-       "properties": {
-           "instanceId": "appgw_3",
-           "clientIp": "167.220.2.139",
-           "clientPort": "",
-           "requestUri": "\/",
-           "ruleSetType": "OWASP_CRS",
-           "ruleSetVersion": "3.0.0",
-           "ruleId": "942130",
-           "message": "SQL Injection Attack: SQL Tautology Detected.",
-           "action": "Matched",
-           "site": "Global",
-           "details": {
-               "message": "Warning. Pattern match \\\"(?i:([\\\\\\\\s'\\\\\\\"`\\\\\\\\(\\\\\\\\)]*?)([\\\\\\\\d\\\\\\\\w]++)([\\\\\\\\s'\\\\\\\"`\\\\\\\\(\\\\\\\\)]*?)(?:(?:=|\\u003c=\\u003e|r?like|sounds\\\\\\\\s+like|regexp)([\\\\\\\\s'\\\\\\\"`\\\\\\\\(\\\\\\\\)]*?)\\\\\\\\2|(?:!=|\\u003c=|\\u003e=|\\u003c\\u003e|\\u003c|\\u003e|\\\\\\\\^|is\\\\\\\\s+not|not\\\\\\\\s+like|not\\\\\\\\s+regexp)([\\\\\\\\s'\\\\\\\"`\\\\\\\\(\\\\\\\\)]*?)(?!\\\\\\\\2)([\\\\\\\\d\\\\\\\\w]+)))\\\" at ARGS:text1. ",
-               "data": "Matched Data: 1=1 found within ARGS:text1: 1=1",
-               "file": "rules\/REQUEST-942-APPLICATION-ATTACK-SQLI.conf\\\"",
-               "line": "554"
-           },
-           "hostname": "vm000003",
-           "transactionId": "AcAcAcAcAKH@AcAcAcAcAyAt"
-       }
-   }
-   {
-       "resourceId": "/SUBSCRIPTIONS/A6F44B25-259E-4AF5-888A-386FED92C11B/RESOURCEGROUPS/DEMOWAF_V2/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/DEMOWAF-V2",
-       "operationName": "ApplicationGatewayFirewall",
-       "category": "ApplicationGatewayFirewallLog",
-       "properties": {
-           "instanceId": "appgw_3",
-           "clientIp": "167.220.2.139",
-           "clientPort": "",
-           "requestUri": "\/",
-           "ruleSetType": "",
-           "ruleSetVersion": "",
-           "ruleId": "0",
-           "message": "Mandatory rule. Cannot be disabled. Inbound Anomaly Score Exceeded (Total Score: 8)",
-           "action": "Blocked",
-           "site": "Global",
-           "details": {
-               "message": "Access denied with code 403 (phase 2). Operator GE matched 5 at TX:anomaly_score. ",
-               "data": "",
-               "file": "rules\/REQUEST-949-BLOCKING-EVALUATION.conf\\\"",
-               "line": "57"
-           },
-           "hostname": "vm000003",
-           "transactionId": "AcAcAcAcAKH@AcAcAcAcAyAt"
-       }
-   }
-   {
-       "resourceId": "/SUBSCRIPTIONS/A6F44B25-259E-4AF5-888A-386FED92C11B/RESOURCEGROUPS/DEMOWAF_V2/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/DEMOWAF-V2",
-       "operationName": "ApplicationGatewayFirewall",
-       "category": "ApplicationGatewayFirewallLog",
-       "properties": {
-           "instanceId": "appgw_3",
-           "clientIp": "167.220.2.139",
-           "clientPort": "",
-           "requestUri": "\/",
-           "ruleSetType": "",
-           "ruleSetVersion": "",
-           "ruleId": "0",
-           "message": "Mandatory rule. Cannot be disabled. Inbound Anomaly Score Exceeded (Total Inbound Score: 8 - SQLI=5,XSS=0,RFI=0,LFI=0,RCE=0,PHPI=0,HTTP=0,SESS=0): SQL Injection Attack: SQL Tautology Detected.",
-           "action": "Blocked",
-           "site": "Global",
-           "details": {
-               "message": "Warning. Operator GE matched 5 at TX:inbound_anomaly_score. ",
-               "data": "",
-               "file": "rules\/RESPONSE-980-CORRELATION.conf\\\"",
-               "line": "73"
-           },
-           "hostname": "vm000003",
-           "transactionId": "AcAcAcAcAKH@AcAcAcAcAyAt"
-       }
-   }
```

Зная, как работают правила CR, и что набор правил CR 3,0 работает с системой оценки аномалий (см. раздел [брандмауэр веб-приложения для шлюза приложений Azure](ag-overview.md)), вы понимаете, что нижние правила с параметром **Action: blocked** блокируются на основе общей оценки аномалий. Правила, на которые следует обратить внимание, являются двумя верхними.

Первая запись заносится в журнал, так как пользователь использовал числовой IP-адрес для перехода к шлюзу приложений, который в данном случае можно игнорировать.

Второй из них (правило 942130). В сведениях, которые соответствуют шаблону (1 = 1), можно увидеть, что поле имеет имя **Text1**. Выполните те же предыдущие действия, чтобы исключить **имя атрибута запроса** , **равное** **1 = 1**.

## <a name="finding-request-header-names"></a>Поиск имен заголовков запросов

Fiddler — это удобное средство для поиска имен заголовков запросов. На следующем снимке экрана показаны заголовки для этого запроса GET, которые включают *тип содержимого*, *Агент пользователя* и т. д.

:::image type="content" source="../media/web-application-firewall-troubleshoot/fiddler-2.png" alt-text="Снимок экрана с веб-отладчиком Progress Telerik Fiddler. На вкладке RAW перечислены сведения заголовка запроса, такие как соединение, тип содержимого и агент пользователя." border="false":::

Другим способом просмотра заголовков запросов и ответов является поиск в средствах разработчика Chrome. Можно нажать клавишу F12 или щелкнуть правой кнопкой мыши > **проверить**  ->  **средства для разработчиков** и выбрать вкладку **сеть** . Загрузите веб-страницу и щелкните запрос, который требуется проверить.

![Chrome F12](../media/web-application-firewall-troubleshoot/chrome-f12.png)

## <a name="finding-request-cookie-names"></a>Поиск имен файлов Cookie запроса

Если запрос содержит файлы cookie, можно выбрать вкладку **"файлы cookie** ", чтобы просмотреть их в Fiddler.

## <a name="restrict-global-parameters-to-eliminate-false-positives"></a>Ограничить глобальные параметры, чтобы исключить ложные срабатывания

- Отключить проверку текста запроса

   Установив для параметра **текст запроса проверки** значение OFF, текст запроса для всего трафика не будет оцениваться WAF. Это может быть полезно, если известно, что текст запроса не является вредоносным для вашего приложения.

   Если отключить этот параметр, проверка только текста запроса не выполняется. Заголовки и файлы cookie остаются проверенными, если только отдельные объекты не исключаются с помощью функции списка исключений.

- Ограничения размера файла

   Ограничив размер файла WAF, вы ограничиваете возможность атаки на веб-серверы. При разрешении загрузки больших файлов риск перегрузки серверной части увеличивается. Ограничение размера файла обычным вариантом использования приложения — еще один способ предотвращения атак.

   > [!NOTE]
   > Если вы уверены, что приложение никогда не будет требовать передачи файлов выше заданного размера, можно ограничить его, установив ограничение.

## <a name="firewall-metrics-waf_v1-only"></a>Метрики брандмауэра (только WAF_v1)

Для брандмауэров веб-приложений версии v1 на портале теперь доступны следующие метрики: 

1. Запрос о блокировке брандмауэра веб-приложения число заблокированных запросов
2. Правило "число заблокированных правил брандмауэра веб-приложения" учитывает все правила, которые были сопоставлены, **и** запрос заблокирован
3. Общее распределение правил брандмауэра веб-приложения все правила, сопоставленные во время оценки
     
Чтобы включить метрики, выберите на портале вкладку **метрики** и выберите одну из трех метрик.

## <a name="next-steps"></a>Дальнейшие действия

См. раздел [Настройка брандмауэра веб-приложения в шлюзе приложений](tutorial-restrict-web-traffic-powershell.md).
