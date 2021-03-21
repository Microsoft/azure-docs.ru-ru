---
title: Журналы работоспособности и диагностики серверной части
titleSuffix: Azure Application Gateway
description: Узнайте, как включить журналы доступа и производительности для Шлюза приложений Azure и управлять ими.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: article
ms.date: 11/22/2019
ms.author: victorh
ms.openlocfilehash: e18267d5a810826e41e05eb3dde3cd82a8eb9197
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98065326"
---
# <a name="back-end-health-and-diagnostic-logs-for-application-gateway"></a>Журналы работоспособности и диагностики серверной части для шлюза приложений

Вы можете отслеживать ресурсы шлюза приложений Azure следующими способами.

* [Работоспособность серверной части.](#back-end-health) Шлюз приложений позволяет отслеживать работоспособность серверов во внутренних пулах с помощью портала Azure и PowerShell. Состояние работоспособности внутренних пулов можно также узнать из журналов диагностики производительности.

* [Журналы.](#diagnostic-logging) Можно сохранять или использовать данные производительности, доступа и другие данные, относящиеся к ресурсу, чтобы отслеживать его состояние.

* [Метрики](application-gateway-metrics.md). шлюз приложений имеет несколько метрик, которые помогают проверить, что система работает должным образом.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="back-end-health"></a>Работоспособность серверной части

Шлюз приложений позволяет отслеживать работоспособность отдельных участников внутренних пулов с помощью портала, PowerShell и интерфейса командной строки (CLI). Сводные данные работоспособности внутренних пулов можно также найти в журналах диагностики производительности.

В отчете о работоспособности серверной части представлены результаты пробы работоспособности, выполняемой шлюзом приложений в отношении серверных экземпляров. Если проба прошла успешно и серверная часть может получать трафик, она считается работоспособной. В противном случае серверная часть считается неработоспособной.

> [!IMPORTANT]
> Если в подсети шлюза приложений есть группа безопасности сети (NSG), откройте диапазон портов 65503-65534 для номеров SKU v1 и 65200-65535 для SKU v2 в подсети шлюза приложений для входящего трафика. Такой диапазон портов требуется для обмена данными в рамках инфраструктуры Azure. Они защищены (заблокированы) посредством сертификатов Azure. Без подходящих сертификатов внешние сущности, включая клиентов этих шлюзов, не смогут никоим образом изменить конечные точки.


### <a name="view-back-end-health-through-the-portal"></a>Просмотр данных о работоспособности серверной части на портале

На портале данные о работоспособности серверной части предоставляются автоматически. В существующем шлюзе приложений выберите **мониторинг**  >  **работоспособность серверной части**.

На этой странице отображается каждый участник внутреннего пула (сетевой адаптер, IP-адрес или полное доменное имя). На ней показаны имя внутреннего пула, порт, параметры HTTP внутреннего пула и его состояние работоспособности. Допустимые значения состояния работоспособности: **работоспособный**, **неработоспособный** и **неизвестный**.

> [!NOTE]
> Если отображается состояние работоспособности серверной части **Неизвестно**, убедитесь, что доступ к серверной части не заблокирован правилом NSG, маршрутом, определенным пользователем или пользовательской службой DNS в виртуальной сети.

![Работоспособность серверной части][10]

### <a name="view-back-end-health-through-powershell"></a>Просмотр данных о работоспособности серверной части с помощью PowerShell

В приведенном ниже коде PowerShell показано, как просмотреть данные о работоспособности серверной части с помощью командлета `Get-AzApplicationGatewayBackendHealth`.

```powershell
Get-AzApplicationGatewayBackendHealth -Name ApplicationGateway1 -ResourceGroupName Contoso
```

### <a name="view-back-end-health-through-azure-cli"></a>Просмотр данных о работоспособности серверной части с помощью Azure CLI

```azurecli
az network application-gateway show-backend-health --resource-group AdatumAppGatewayRG --name AdatumAppGateway
```

### <a name="results"></a>Результаты

В следующем фрагменте кода приведен пример отклика:

```json
{
"BackendAddressPool": {
    "Id": "/subscriptions/00000000-0000-0000-000000000000/resourceGroups/ContosoRG/providers/Microsoft.Network/applicationGateways/applicationGateway1/backendAddressPools/appGatewayBackendPool"
},
"BackendHttpSettingsCollection": [
    {
    "BackendHttpSettings": {
        "Id": "/00000000-0000-0000-000000000000/resourceGroups/ContosoRG/providers/Microsoft.Network/applicationGateways/applicationGateway1/backendHttpSettingsCollection/appGatewayBackendHttpSettings"
    },
    "Servers": [
        {
        "Address": "hostname.westus.cloudapp.azure.com",
        "Health": "Healthy"
        },
        {
        "Address": "hostname.westus.cloudapp.azure.com",
        "Health": "Healthy"
        }
    ]
    }
]
}
```

## <a name="diagnostic-logs"></a><a name="diagnostic-logging"></a>Журналы диагностики

В Azure можно использовать различные виды журналов для управления шлюзами приложений и устранения возникающих в них неполадок. Доступ к некоторым из этих журналов можно получить через портал. Также можно извлечь все журналы из хранилища BLOB-объектов Azure и просматривать их с помощью таких средств, как [журналы Azure Monitor](../azure-monitor/insights/azure-networking-analytics.md), Excel и Power BI. В списке ниже приведены дополнительные сведения о разных типах журналов:

* **Журнал действий.** В [журналах действий Azure](../azure-resource-manager/management/view-activity-logs.md) (прежнее название — операционные журналы и журналы аудита) можно просматривать все операции, отправляемые в вашу подписку Azure, и состояние этих операций. Записи этого журнала собираются по умолчанию, и их можно просмотреть на портале Azure.
* **Журнал доступа**. Этот журнал можно использовать для просмотра шаблонов доступа к шлюзу приложений и анализа важной информации. Сюда входит IP-адрес вызывающего объекта, запрошенный URL-адреса, задержка ответа, код возврата и байты. Журнал доступа собирается каждые 60 секунд. Этот журнал содержит одну запись для каждого экземпляра шлюза приложений. Экземпляр Шлюза приложений определяется свойством instanceId.
* **Журнал производительности.** С помощью этого журнала можно просматривать, как работают экземпляры шлюза приложений. В этот журнал записываются сведения о производительности каждого экземпляра, включая общее число обработанных запросов, пропускную способность в байтах, число неудачных запросов, а также число работоспособных и неработоспособных серверных экземпляров. Данные для журнала производительности собираются каждые 60 секунд. Журнал производительности доступен только для номера SKU v1. Для номера SKU версии 2 Используйте [метрики](application-gateway-metrics.md) для данных производительности.
* **Журнал брандмауэра.** Этот журнал можно использовать для просмотра запросов, которые были зарегистрированы в режиме обнаружения или предотвращения в шлюзе приложений, в котором настроен брандмауэр веб-приложения. Журналы брандмауэра собираются каждые 60 секунд. 

> [!NOTE]
> Журналы доступны только для ресурсов, развернутых в модели развертывания с помощью Azure Resource Manager. Журналы нельзя использовать для ресурсов в классической модели развертывания. Чтобы получить более полное представление об этих двух моделях, см. статью [Развертывание с помощью Azure Resource Manager и классическое развертывание: сведения о моделях развертывания и состоянии ресурсов](../azure-resource-manager/management/deployment-models.md).

Существует три способа хранения журналов:

* **Учетная запись хранения** лучше всего подходит для длительного хранения журналов и их просмотра по мере необходимости.
* **Концентраторы событий**. концентраторы событий являются отличным вариантом для интеграции с другими средствами управления сведениями о безопасности и событиями (SIEM) для получения оповещений о ресурсах.
* **Журналы Azure Monitor**. журналы Azure Monitor лучше использовать для общего мониторинга приложения в режиме реального времени или для анализа тенденций.

### <a name="enable-logging-through-powershell"></a>Включение ведения журнала с помощью PowerShell

Ведение журнала действий автоматически включается для каждого ресурса Resource Manager. Нужно включить ведение журналов доступа и производительности, чтобы начать сбор связанных данных. Вот как можно включить ведение журнала:

1. Запишите или запомните ИД ресурса учетной записи хранения, где хранятся данные журнала, Это значение имеет вид:/Subscriptions/ \<subscriptionId\> /ResourceGroups/ \<resource group name\> /провидерс/Микрософт.стораже/сторажеаккаунтс/ \<storage account name\> . Можно использовать любую учетную запись хранения в подписке. Получить эти сведения можно на портале Azure.

    ![ИД ресурса для учетной записи хранения на портале](./media/application-gateway-diagnostics/diagnostics1.png)

2. Запишите или запомните идентификатор ресурса шлюза приложений, для которого нужно включить ведение журнала. Это значение имеет вид:/Subscriptions/ \<subscriptionId\> /ResourceGroups/ \<resource group name\> /провидерс/Микрософт.Нетворк/аппликатионгатевайс/ \<application gateway name\> . Получить эти сведения можно на портале.

    ![ИД ресурса для шлюза приложений на портале](./media/application-gateway-diagnostics/diagnostics2.png)

3. Включите ведение журнала диагностики с помощью следующего командлета PowerShell:

    ```powershell
    Set-AzDiagnosticSetting  -ResourceId /subscriptions/<subscriptionId>/resourceGroups/<resource group name>/providers/Microsoft.Network/applicationGateways/<application gateway name> -StorageAccountId /subscriptions/<subscriptionId>/resourceGroups/<resource group name>/providers/Microsoft.Storage/storageAccounts/<storage account name> -Enabled $true     
    ```

> [!TIP]
>Для журналов действий отдельная учетная запись хранения не требуется. За использование хранилища для журналов доступа и производительности взимается плата.

### <a name="enable-logging-through-the-azure-portal"></a>Включение ведения журнала на портале Azure

1. В портал Azure найдите ресурс и выберите **параметры диагностики**.

   Для шлюза приложений доступны три журнала:

   * журнал доступа;
   * журнал производительности;
   * журнал брандмауэра.

2. Чтобы начать сбор данных, выберите **включить диагностику**.

   ![Включение диагностики][1]

3. На странице **Параметры диагностики** представлены параметры журналов диагностики. В этом примере для хранения журналов используется Log Analytics. хранения журналов диагностики можно также использовать концентраторы событий и учетную запись хранения.

   ![Начало процесса настройки][2]

5. Введите имя для параметров, подтвердите параметры и нажмите кнопку **сохранить**.

### <a name="activity-log"></a>Журнал действий

На портале Azure журнал действий создается по умолчанию. Журналы хранятся в течение 90 дней в хранилище журналов событий Azure. Дополнительные сведения об этих журналах см. в статье [Просмотр журналов действий для аудита действий с ресурсами](../azure-resource-manager/management/view-activity-logs.md).

### <a name="access-log"></a>журнал доступа;

Журнал доступа создается, только если он включен для каждого экземпляра шлюза приложений, как описано выше. Данные хранятся в учетной записи хранения, указанной при включении ведения журнала. Каждый доступ к шлюзу приложений заносится в формат JSON, как показано ниже. 

#### <a name="for-application-gateway-standard-and-waf-sku-v1"></a>Для шлюза приложений Standard и WAF SKU (v1)

|Значение  |Описание  |
|---------|---------|
|instanceId     | Экземпляр шлюза приложений, который обрабатывает запрос.        |
|clientIP     | IP-адрес источника запроса.        |
|clientPort     | Порт источника запроса.       |
|httpMethod     | Метод HTTP, используемый для запроса.       |
|requestUri     | URI полученного запроса.        |
|RequestQuery     | **Server-Routed** — экземпляр внутреннего пула, из которого отправлен запрос.</br>**X-AzureApplicationGateway-LOG-ID** — идентификатор корреляции, используемый для запроса. Он может использоваться для устранения неполадок с трафиком на внутренних серверах. </br>**SERVER-STATUS** — код отклика HTTP, полученный шлюзом приложений из серверной части.       |
|UserAgent     | Агент пользователя из заголовка HTTP-запроса.        |
|httpStatus     | Код состояния HTTP, возвращаемый клиенту из шлюза приложений.       |
|httpVersion     | Версия HTTP для запроса.        |
|receivedBytes     | Размер полученного пакета (в байтах).        |
|sentBytes| Размер отправленного пакета (в байтах).|
|timeTaken| Время (в миллисекундах), необходимое для обработки запроса и отправки ответа. Вычисляется как промежуток времени от момента, когда шлюз приложений получает первый байт HTTP-запроса, до завершения отправки ответа. Важно отметить, что в поле Time-Taken обычно указывается время передачи пакетов запросов и ответов по сети. |
|sslEnabled| Используется ли для подключения к внутренним пулам протокол TLS/SSL. Допустимые значения: on и off.|
|узел| Имя узла, с которого запрос был отправлен на внутренний сервер. При переопределении серверного имени узла это имя будет отражать это.|
|оригиналхост| Имя узла, с которым шлюз приложений получил запрос от клиента.|
```json
{
    "resourceId": "/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/PEERINGTEST/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/{applicationGatewayName}",
    "operationName": "ApplicationGatewayAccess",
    "time": "2017-04-26T19:27:38Z",
    "category": "ApplicationGatewayAccessLog",
    "properties": {
        "instanceId": "ApplicationGatewayRole_IN_0",
        "clientIP": "191.96.249.97",
        "clientPort": 46886,
        "httpMethod": "GET",
        "requestUri": "/phpmyadmin/scripts/setup.php",
        "requestQuery": "X-AzureApplicationGateway-CACHE-HIT=0&SERVER-ROUTED=10.4.0.4&X-AzureApplicationGateway-LOG-ID=874f1f0f-6807-41c9-b7bc-f3cfa74aa0b1&SERVER-STATUS=404",
        "userAgent": "-",
        "httpStatus": 404,
        "httpVersion": "HTTP/1.0",
        "receivedBytes": 65,
        "sentBytes": 553,
        "timeTaken": 205,
        "sslEnabled": "off",
        "host": "www.contoso.com",
        "originalHost": "www.contoso.com"
    }
}
```
#### <a name="for-application-gateway-and-waf-v2-sku"></a>Для шлюза приложений и SKU WAF v2

|Значение  |Описание  |
|---------|---------|
|instanceId     | Экземпляр шлюза приложений, который обрабатывает запрос.        |
|clientIP     | IP-адрес источника запроса.        |
|httpMethod     | Метод HTTP, используемый для запроса.       |
|requestUri     | URI полученного запроса.        |
|UserAgent     | Агент пользователя из заголовка HTTP-запроса.        |
|httpStatus     | Код состояния HTTP, возвращаемый клиенту из шлюза приложений.       |
|httpVersion     | Версия HTTP для запроса.        |
|receivedBytes     | Размер полученного пакета (в байтах).        |
|sentBytes| Размер отправленного пакета (в байтах).|
|timeTaken| Время (в **секундах**), затрачиваемое на обработку запроса и отправку ответа. Вычисляется как промежуток времени от момента, когда шлюз приложений получает первый байт HTTP-запроса, до завершения отправки ответа. Важно отметить, что в поле Time-Taken обычно указывается время передачи пакетов запросов и ответов по сети. |
|sslEnabled| Используется ли для обмена данными между внутренними пулами протокол TLS. Допустимые значения: on и off.|
|сслЦифер| Набор шифров, используемый для обмена данными TLS (если включен протокол TLS).|
|sslProtocol| Используемый протокол SSL/TLS (если TLS включен).|
|серверраутед| Внутренний сервер, на который шлюз приложений направляет запрос.|
|serverStatus| Код состояния HTTP внутреннего сервера.|
|серверреспонселатенци| Задержка ответа от внутреннего сервера.|
|узел| Адрес, указанный в заголовке узла запроса. Если переписать с помощью перезаписи заголовка, это поле содержит обновленное имя узла|
|оригиналрекуестуривисаргс| Это поле содержит исходный URL-адрес запроса |
|requestUri| Это поле содержит URL-адрес после операции перезаписи в шлюзе приложений |
|оригиналхост| Это поле содержит исходное имя узла запроса
```json
{
    "resourceId": "/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/PEERINGTEST/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/{applicationGatewayName}",
    "operationName": "ApplicationGatewayAccess",
    "time": "2017-04-26T19:27:38Z",
    "category": "ApplicationGatewayAccessLog",
    "properties": {
        "instanceId": "appgw_1",
        "clientIP": "191.96.249.97",
        "httpMethod": "GET",
        "requestUri": "/phpmyadmin/scripts/setup.php",
        "userAgent": "-",
        "httpStatus": 404,
        "httpVersion": "HTTP/1.0",
        "receivedBytes": 65,
        "sentBytes": 553,
        "timeTaken": 205,
        "sslEnabled": "off",
        "sslCipher": "",
        "sslProtocol": "",
        "serverRouted": "104.41.114.59:80",
        "serverStatus": "200",
        "serverResponseLatency": "0.023",
        "host": "www.contoso.com",
    }
}
```

### <a name="performance-log"></a>журнал производительности;

Журнал производительности создается, только если он включен для каждого экземпляра шлюза приложений, как описано выше. Данные хранятся в учетной записи хранения, указанной при включении ведения журнала. Данные журнала производительности формируются с интервалом в 1 минуту. Он доступен только для номера SKU версии v1. Для номера SKU версии 2 Используйте [метрики](application-gateway-metrics.md) для данных производительности. В журнал записываются следующие данные:


|Значение  |Описание  |
|---------|---------|
|instanceId     |  Экземпляр шлюза приложений, для которого формируются данные о производительности. Если экземпляров шлюза приложений несколько, каждому экземпляру будет соответствовать определенная строка.        |
|healthyHostCount     | Число работоспособных узлов во внутреннем пуле.        |
|unHealthyHostCount     | Число неработоспособных узлов во внутреннем пуле.        |
|requestCount     | Число обрабатываемых запросов.        |
|Задержка | Средняя задержка (в миллисекундах) запросов от экземпляра к серверной части, обрабатывающей запросы. |
|failedRequestCount| Количество невыполненных запросов.|
|throughput| Средняя пропускная способность (в байтах в секунду) с момента создания последнего журнала.|

```json
{
    "resourceId": "/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/{applicationGatewayName}",
    "operationName": "ApplicationGatewayPerformance",
    "time": "2016-04-09T00:00:00Z",
    "category": "ApplicationGatewayPerformanceLog",
    "properties":
    {
        "instanceId":"ApplicationGatewayRole_IN_1",
        "healthyHostCount":"4",
        "unHealthyHostCount":"0",
        "requestCount":"185",
        "latency":"0",
        "failedRequestCount":"0",
        "throughput":"119427"
    }
}
```

> [!NOTE]
> Задержка вычисляется от момента получения первого байта HTTP-запроса до момента отправки последнего байта HTTP-отклика. Это сумма времени обработки шлюза приложений, времени передачи по сети к серверной части, а также времени, необходимого для обработки запроса в серверной части.

### <a name="firewall-log"></a>журнал брандмауэра.

Этот журнал создается, только если он включен для каждого шлюза приложений, как описано выше. Кроме того, в шлюзе приложений должен быть настроен брандмауэр веб-приложения. Данные хранятся в учетной записи хранения, указанной при включении ведения журнала. В журнал записываются следующие данные:


|Значение  |Описание  |
|---------|---------|
|instanceId     | Экземпляр шлюза приложений, для которого формируются данные о брандмауэре. Если экземпляров шлюза приложений несколько, каждому экземпляру будет соответствовать определенная строка.         |
|clientIp     |   IP-адрес источника запроса.      |
|clientPort     |  Порт источника запроса.       |
|requestUri     | URL-адрес полученного запроса.       |
|ruleSetType     | Тип набора правил. Доступное значение — OWASP.        |
|ruleSetVersion     | Используемая версия набора правил. Возможные значения: 2.2.9 и 3.0.     |
|ruleId     | Идентификатор правила события-триггера.        |
|message     | Понятное сообщение для события-триггера. Дополнительные сведения приведены в разделе details.        |
|action     |  Действие, выполняемое с запросом. Доступные значения сопоставлены и блокируются.      |
|site     | Сайт, для которого создан журнал. В нашем случае возможно только значение Global, так как применяются глобальные правила.|
|подробности     | Сведения о событии-триггере.        |
|details.message     | Описание правила.        |
|details.data     | Определенные данные из запроса, которые соответствуют правилу.         |
|details.file     | Файл конфигурации, содержащий правило.        |
|details.line     | Номер строки в файле конфигурации, активировавшей событие.       |
|hostname   | Имя узла или IP-адрес шлюза приложений.    |
|transactionId  | Уникальный идентификатор для данной транзакции, который помогает сгруппировать несколько нарушений правил, произошедших в одном запросе.   |

```json
{
  "resourceId": "/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/{applicationGatewayName}",
  "operationName": "ApplicationGatewayFirewall",
  "time": "2017-03-20T15:52:09.1494499Z",
  "category": "ApplicationGatewayFirewallLog",
  "properties": {
    "instanceId": "ApplicationGatewayRole_IN_0",
    "clientIp": "104.210.252.3",
    "clientPort": "4835",
    "requestUri": "/?a=%3Cscript%3Ealert(%22Hello%22);%3C/script%3E",
    "ruleSetType": "OWASP",
    "ruleSetVersion": "3.0",
    "ruleId": "941320",
    "message": "Possible XSS Attack Detected - HTML Tag Handler",
    "action": "Blocked",
    "site": "Global",
    "details": {
      "message": "Warning. Pattern match \"<(a|abbr|acronym|address|applet|area|audioscope|b|base|basefront|bdo|bgsound|big|blackface|blink|blockquote|body|bq|br|button|caption|center|cite|code|col|colgroup|comment|dd|del|dfn|dir|div|dl|dt|em|embed|fieldset|fn|font|form|frame|frameset|h1|head|h ...\" at ARGS:a.",
      "data": "Matched Data: <script> found within ARGS:a: <script>alert(\\x22hello\\x22);</script>",
      "file": "rules/REQUEST-941-APPLICATION-ATTACK-XSS.conf",
      "line": "865"
    }
    "hostname": "40.90.218.100",
    "transactionId": "AYAcUqAcAcAcAcAcASAcAcAc"
  }
}

```

### <a name="view-and-analyze-the-activity-log"></a>Просмотр и анализ журнала действий

Данные журнала действий можно просматривать и анализировать с помощью любого из следующих методов:

* **Средства Azure.** Информацию из журналов действий можно получать с помощью Azure PowerShell, Azure CLI, REST API Azure или портала Azure. Пошаговые инструкции для каждого метода подробно описаны в статье [Activity operations with Resource Manager](../azure-resource-manager/management/view-activity-logs.md) (Выполнение операций в журналах действий с помощью Resource Manager).
* **Power BI**. Если у вас еще нет учетной записи [Power BI](https://powerbi.microsoft.com/pricing) , вы можете попробовать ее бесплатно. С помощью [Power BIных приложений шаблонов](/power-bi/service-template-apps-overview)можно анализировать данные.

### <a name="view-and-analyze-the-access-performance-and-firewall-logs"></a>Просмотр и анализ журналов доступа, производительности и брандмауэра

[Журналы Azure Monitor](../azure-monitor/insights/azure-networking-analytics.md) позволяют собирать файлы журналов счетчиков и событий из вашей учетной записи хранилища BLOB-объектов. Эта служба предоставляет средства визуализации и эффективные возможности поиска для анализа журналов.

Вы также можете подключиться к учетной записи хранения и извлечь записи журнала JSON для журналов доступа и производительности. После скачивания JSON-файлов их можно преобразовать в формат CSV и просматривать в Excel, Power BI или другом средстве визуализации данных.

> [!TIP]
> Если вы знакомы с Visual Studio и основными понятиями изменения значений констант и переменных в C#, можно использовать [инструменты преобразования журналов](https://github.com/Azure-Samples/networking-dotnet-log-converter), доступные на сайте GitHub.
>
>

#### <a name="analyzing-access-logs-through-goaccess"></a>Анализ журналов доступа через GoAccess

Мы опубликовали шаблон Resource Manager, который устанавливает и запускает популярный анализатор журналов [GoAccess](https://goaccess.io/) для журналов доступа шлюза приложений. GoAccess предоставляет ценную статистику трафика HTTP, такую как уникальные посетители, запрошенные файлы, узлы, операционные системы, браузеры, коды состояния HTTP и многое другое. Дополнительные сведения см. в [файле сведений в папке шаблона Resource Manager на GitHub](https://aka.ms/appgwgoaccessreadme).

## <a name="next-steps"></a>Дальнейшие действия

* См. сведения о визуализации журналов счетчиков и событий с помощью [журналов Azure Monitor](../azure-monitor/insights/azure-networking-analytics.md).
* [Визуализируйте журнал действий Azure с помощью Power BI](https://powerbi.microsoft.com/blog/monitor-azure-audit-logs-with-power-bi/) записи блога.
* Прочтите запись блога [View and analyze Azure Audit Logs in Power BI and more](https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/) (Просмотр и анализ журналов аудита Azure с помощью Power BI и других средств).

[1]: ./media/application-gateway-diagnostics/figure1.png
[2]: ./media/application-gateway-diagnostics/figure2.png
[3]: ./media/application-gateway-diagnostics/figure3.png
[4]: ./media/application-gateway-diagnostics/figure4.png
[5]: ./media/application-gateway-diagnostics/figure5.png
[6]: ./media/application-gateway-diagnostics/figure6.png
[7]: ./media/application-gateway-diagnostics/figure7.png
[8]: ./media/application-gateway-diagnostics/figure8.png
[9]: ./media/application-gateway-diagnostics/figure9.png
[10]: ./media/application-gateway-diagnostics/figure10.png
