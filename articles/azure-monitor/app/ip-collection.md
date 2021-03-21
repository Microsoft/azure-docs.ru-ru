---
title: Коллекция IP-адресов Application Insights Azure | Документация Майкрософт
description: Основные сведения об обработке IP-адресов и географического расположения с помощью Azure Application Insights
ms.topic: conceptual
ms.date: 09/23/2020
ms.custom: devx-track-js
ms.openlocfilehash: 91b3aa07720e39aa8aeeceb9c35e38205e7d7c76
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100584081"
---
# <a name="geolocation-and-ip-address-handling"></a>Обработка геолокации и IP-адресов

В этой статье объясняется, как работает поиск и обработка IP-адресов географического расположения в Application Insights вместе с изменением поведения по умолчанию.

## <a name="default-behavior"></a>Поведение по умолчанию

По умолчанию IP-адреса временно собираются, но не хранятся в Application Insights. Базовый процесс выглядит следующим образом:

Когда данные телеметрии отправляются в Azure, IP-адрес используется для уточняющего запроса географического расположения с помощью [GeoLite2 из максминд](https://dev.maxmind.com/geoip/geoip2/geolite2/). Результаты этого поиска используются для заполнения полей `client_City` , `client_StateOrProvince` и `client_CountryOrRegion` . Затем адрес отбрасывается и `0.0.0.0` записывается в `client_IP` поле.

* Данные телеметрии браузера. мы временно соберем IP-адрес отправителя. IP-адрес вычисляется конечной точкой приема.
* Данные телеметрии сервера. модуль телеметрии Application Insights временно собирает IP-адрес клиента. IP-адрес не собирается локально, если `X-Forwarded-For` задан заголовок.

Это поведение предназначено для предотвращения ненужной коллекции персональных данных. По возможности рекомендуется избегать сбора персональных данных. 

## <a name="overriding-default-behavior"></a>Переопределение поведения по умолчанию

Хотя по умолчанию не собираются IP-адреса. Мы по-прежнему предлагаем гибкость для переопределения этого поведения. Однако рекомендуется проверить, что коллекция не нарушает требования соответствия или локальные правила. 

Дополнительные сведения об обработке персональных данных в Application Insights см. в [руководстве по личным данным](../logs/personal-data-mgmt.md).

## <a name="storing-ip-address-data"></a>Хранение данных IP-адреса

Чтобы включить сбор и хранение IP-адресов, `DisableIpMasking` свойству компонента Application Insights необходимо присвоить значение `true` . Это свойство можно задать с помощью шаблонов Azure Resource Manager или путем вызова REST API. 

### <a name="azure-resource-manager-template"></a>Шаблон Azure Resource Manager

```json
{
       "id": "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/microsoft.insights/components/<resource-name>",
       "name": "<resource-name>",
       "type": "microsoft.insights/components",
       "location": "westcentralus",
       "tags": {
              
       },
       "kind": "web",
       "properties": {
              "Application_Type": "web",
              "Flow_Type": "Redfield",
              "Request_Source": "IbizaAIExtension",
              // ...
              "DisableIpMasking": true
       }
}
```

### <a name="portal"></a>Портал 

Если необходимо изменить поведение только для одного Application Insights ресурса, используйте портал Azure. 

1. Выберите Application Insights ресурс >   >  **шаблон экспорта** службы автоматизации 

2. Выберите **Развернуть**.

    ![Кнопка с словом "развернуть", выделенным красным цветом](media/ip-collection/deploy.png)

3. Выберите **изменить шаблон**.

    ![Кнопка с словом "Изменить", выделенное красным цветом](media/ip-collection/edit-template.png)

4. Внесите следующие изменения в JSON для ресурса, а затем выберите **сохранить**:

    ![Снимок экрана добавляет запятую после "Ибизааиекстенсион" и добавляет новую строку ниже с "Дисаблеипмаскинг": true](media/ip-collection/save.png)

    > [!WARNING]
    > Если возникнет сообщение об ошибке: **_Группа ресурсов находится в расположении, которое не поддерживается одним или несколькими ресурсами в шаблоне. Выберите другую группу ресурсов._** Временно выберите из раскрывающегося списка другую группу ресурсов, а затем повторно выберите исходную группу ресурсов, чтобы устранить эту ошибку.

5. Выберите **я согласен**  >  **приобрести**. 

    ![Флажок с словами «я принимаю указанные выше условия», выделенный красным рядом с кнопкой «покупка», выделенной красным цветом.](media/ip-collection/purchase.png)

    В этом случае на самом деле ничего не приобретается. Выполняется обновление только конфигурации существующего Application Insights ресурса.

6. После завершения развертывания будут записаны новые данные телеметрии.

    Если выбрать и изменить шаблон еще раз, вы увидите только шаблон по умолчанию без вновь добавленного свойства. Если вы не видите данные IP-адреса и хотите подтвердить, что `"DisableIpMasking": true` задано, выполните следующую команду PowerShell: 
    
    ```powershell
    # Replace `Fabrikam-dev` with the appropriate resource and resource group name.
    # If you aren't using the cloud shell you will need to connect to your Azure account
    # Connect-AzAccount 
    $AppInsights = Get-AzResource -Name 'Fabrikam-dev' -ResourceType 'microsoft.insights/components' -ResourceGroupName 'Fabrikam-dev'
    $AppInsights.Properties
    ```
    
    В результате будет возвращен список свойств. Необходимо прочитать одно из свойств `DisableIpMasking: true` . Если запустить PowerShell перед развертыванием нового свойства с Azure Resource Manager, это свойство не будет существовать.

### <a name="rest-api"></a>Rest API

Полезная нагрузка [API-интерфейса](/rest/api/azure/) для внесения одних и тех же изменений выглядит следующим образом:

```
PATCH https://management.azure.com/subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/microsoft.insights/components/<resource-name>?api-version=2018-05-01-preview HTTP/1.1
Host: management.azure.com
Authorization: AUTH_TOKEN
Content-Type: application/json
Content-Length: 54

{
       "location": "<resource location>",
       "kind": "web",
       "properties": {
              "Application_Type": "web",
              "DisableIpMasking": true
       }
}
```

## <a name="telemetry-initializer"></a>Инициализатор телеметрии

Если вам требуется более гибкая альтернатива `DisableIpMasking` , можно использовать [инициализатор телеметрии](./api-filtering-sampling.md#addmodify-properties-itelemetryinitializer) , чтобы скопировать весь IP-адрес или его часть в настраиваемое поле. 

# <a name="net"></a>[.NET](#tab/net)

### <a name="aspnet--aspnet-core"></a>ASP.NET/ASP.NET Core

```csharp
using Microsoft.ApplicationInsights.Channel;
using Microsoft.ApplicationInsights.DataContracts;
using Microsoft.ApplicationInsights.Extensibility;

namespace MyWebApp
{
    public class CloneIPAddress : ITelemetryInitializer
    {
        public void Initialize(ITelemetry telemetry)
        {
            ISupportProperties propTelemetry = telemetry as ISupportProperties;

            if (propTelemetry !=null && !propTelemetry.Properties.ContainsKey("client-ip"))
            {
                string clientIPValue = telemetry.Context.Location.Ip;
                propTelemetry.Properties.Add("client-ip", clientIPValue);
            }
        }
    } 
}
```

> [!NOTE]
> Если вам не удается получить доступ `ISupportProperties` , проверьте и убедитесь, что вы используете последний стабильный выпуск пакета SDK для Application Insights. `ISupportProperties` предназначены для больших значений количества элементов, тогда как `GlobalProperties` более подходящие для значений низкого уровня кратности, таких как имя региона, имя среды и т. д. 

### <a name="enable-telemetry-initializer-for-aspnet"></a>Включение инициализатора телеметрии для ASP.NET

```csharp
using Microsoft.ApplicationInsights.Extensibility;


namespace MyWebApp
{
    public class MvcApplication : System.Web.HttpApplication
    {
        protected void Application_Start()
        {
              //Enable your telemetry initializer:
              TelemetryConfiguration.Active.TelemetryInitializers.Add(new CloneIPAddress());
        }
    }
}

```

### <a name="enable-telemetry-initializer-for-aspnet-core"></a>Включение инициализатора телеметрии для ASP.NET Core

Вы можете создать свой инициализатор телеметрии так же, как для ASP.NET Core как ASP.NET, но чтобы включить инициализатор, используйте следующий пример для справки:

```csharp
 using Microsoft.ApplicationInsights.Extensibility;
 using CustomInitializer.Telemetry;
 public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton<ITelemetryInitializer, CloneIPAddress>();
}
```
# <a name="nodejs"></a>[Node.js](#tab/nodejs)

### <a name="nodejs"></a>Node.js

```javascript
appInsights.defaultClient.addTelemetryProcessor((envelope) => {
    const baseData = envelope.data.baseData;
    if (appInsights.Contracts.domainSupportsProperties(baseData)) {
        const ipAddress = envelope.tags[appInsights.defaultClient.context.keys.locationIp];
        if (ipAddress) {
            baseData.properties["client-ip"] = ipAddress;
        }
    }
});
```
# <a name="client-side-javascript"></a>[Клиентская часть JavaScript](#tab/javascript)

### <a name="client-side-javascript"></a>Клиентская часть JavaScript

В отличие от пакетов SDK на стороне сервера, пакет SDK JavaScript на стороне клиента не вычисляет IP-адрес. По умолчанию вычисление IP-адреса для телеметрии на стороне клиента происходит в конечной точке приема в Azure. 

Если требуется вычислить IP-адрес непосредственно на стороне клиента, необходимо добавить собственную пользовательскую логику и использовать результат для задания `ai.location.ip` тега. Если `ai.location.ip` задано значение, вычисление IP-адресов не выполняется конечной точкой приема, а предоставленный IP-адрес используется для поиска по географическому положению. В этом случае IP-адрес по-прежнему будет обнулен. 

Чтобы обеспечить вычисление всего IP-адреса из пользовательской логики, можно использовать инициализатор телеметрии, который скопирует данные IP-адреса, предоставленные в, в `ai.location.ip` отдельное настраиваемое поле. Но опять же, в отличие от серверных пакетов SDK, не полагаться на сторонние библиотеки или собственную логику коллекции, клиентский пакет SDK не вычислит этот адрес.    


```javascript
appInsights.addTelemetryInitializer((item) => {
    const ipAddress = item.tags && item.tags["ai.location.ip"];
    if (ipAddress) {
        item.baseData.properties = {
            ...item.baseData.properties,
            "client-ip": ipAddress
        };
    }
});

```  

Если клиентские данные проходят через прокси перед пересылкой конечной точке приема, то вычисление IP-адресов может отобразить IP-адрес прокси-сервера, а не клиента. 

---

### <a name="view-the-results-of-your-telemetry-initializer"></a>Просмотр результатов инициализатора телеметрии

При отправке нового трафика на сайт и подождите несколько минут. Затем можно выполнить запрос, чтобы подтвердить работу коллекции:

```kusto
requests
| where timestamp > ago(1h) 
| project appName, operation_Name, url, resultCode, client_IP, customDimensions.["client-ip"]
```

Новые собранные IP-адреса отобразятся в `customDimensions_client-ip` столбце. По умолчанию в `client-ip` столбце по-прежнему будут обнулены все четыре октета. 

Если проверка выполняется с localhost, а значение параметра `customDimensions_client-ip` — `::1` , то это ожидаемое поведение. `::1` представляет петлевой адрес в IPv6. Он эквивалентен `127.0.0.1` в IPv4.

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о [сборе персональных данных](../logs/personal-data-mgmt.md) в Application Insights.

* Узнайте больше о том, как работает [Коллекция IP-адресов](https://apmtips.com/posts/2016-07-05-client-ip-address/) в Application Insights. (Эта статья содержит старую внешнюю запись блога, написанную одним из наших инженеров. Он предварительно выполняет текущее поведение по умолчанию, где IP-адрес записывается как `0.0.0.0` , но он переходит в более глубокое описание механизма встроенной среды `ClientIpHeaderTelemetryInitializer` .)
