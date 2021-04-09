---
title: Автоматизация Azure Application Insights с помощью PowerShell | Документация Майкрософт
description: Автоматизация создания ресурсов, оповещений и тестов доступности и управления ими с помощью PowerShell и шаблона Azure Resource Manager.
ms.topic: conceptual
ms.date: 05/02/2020
ms.openlocfilehash: c2e3d33be487b6a92cb7038d814e17fcd5a10064
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100589805"
---
#  <a name="manage-application-insights-resources-using-powershell"></a>Управление ресурсами Application Insights с помощью PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

В этой статье показано, как автоматизировать создание и обновление ресурсов [Application Insights](./app-insights-overview.md) с помощью управления ресурсами Azure. Эту функцию можно использовать, например, в процессе сборки. Наряду с базовым ресурсом Application Insights можно создавать [веб-тесты доступности](./monitor-web-app-availability.md) и другие ресурсы Azure, а также настраивать [оповещения](../alerts/alerts-log.md) и [схему цен](pricing.md).

Ключ к созданию этих ресурсов — шаблоны JSON для [диспетчера ресурсов Azure](../../azure-resource-manager/management/manage-resources-powershell.md). Базовая процедура: скачайте определения JSON существующих ресурсов, параметризуйте определенные значения (например, имена), выполните шаблон для создания нового ресурса. Несколько ресурсов можно объединить, чтобы создавать их одновременно, например, объединить монитор приложений с тестами доступности, оповещениями и хранилищем для непрерывного экспорта. С параметризацией некоторых значений связаны определенные тонкости, которые мы рассмотрим позднее.

## <a name="one-time-setup"></a>Однократная настройка
Если вы ранее не использовали PowerShell для подписки Azure:

Установите модуль Azure PowerShell на компьютере, где будут выполняться скрипты.

1. Установите [установщик веб-платформы Майкрософт (версии 5 или более поздней)](https://www.microsoft.com/web/downloads/platform.aspx).
2. Используйте его для установки Microsoft Azure PowerShell.

Кроме шаблонов Resource Manager, вы можете использовать обширный набор [командлетов PowerShell для Application Insights](/powershell/module/az.applicationinsights), которые упрощают настройку ресурсов Application Insights программными средствами. Командлеты предоставляют следующие возможности:

* создание и удаление ресурсов Application Insights;
* получение списков и свойств ресурсов Application Insights;
* создание непрерывного экспорта и управление им;
* создание ключей приложения и управление ими;
* установка ежедневного ограничения;
* выбор тарифного плана.

## <a name="create-application-insights-resources-using-a-powershell-cmdlet"></a>Создание ресурсов Application Insights с помощью командлета PowerShell

Далее показано, как создать новый ресурс Application Insights в центре обработки данных в регионе "Восточная часть США" с помощью командлета [New-AzApplicationInsights](/powershell/module/az.applicationinsights/new-azapplicationinsights).

```PS
New-AzApplicationInsights -ResourceGroupName <resource group> -Name <resource name> -location eastus
```


## <a name="create-application-insights-resources-using-a-resource-manager-template"></a>Создание ресурсов Application Insights с помощью шаблона Resource Manager

Далее описано, как создать ресурс Application Insights с помощью шаблона Resource Manager.

### <a name="create-the-azure-resource-manager-template"></a>Создание шаблона Azure Resource Manager

Создайте новый файл JSON — в данном примере назовем его `template1.json` . Скопируйте в него следующее содержимое:

```JSON
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "appName": {
                "type": "string",
                "metadata": {
                    "description": "Enter the name of your Application Insights resource."
                }
            },
            "appType": {
                "type": "string",
                "defaultValue": "web",
                "allowedValues": [
                    "web",
                    "java",
                    "other"
                ],
                "metadata": {
                    "description": "Enter the type of the monitored application."
                }
            },
            "appLocation": {
                "type": "string",
                "defaultValue": "eastus",
                "metadata": {
                    "description": "Enter the location of your Application Insights resource."
                }
            },
            "retentionInDays": {
                "type": "int",
                "defaultValue": 90,
                "allowedValues": [
                    30,
                    60,
                    90,
                    120,
                    180,
                    270,
                    365,
                    550,
                    730
                ],
                "metadata": {
                    "description": "Data retention in days"
                }
            },
            "ImmediatePurgeDataOn30Days": {
                "type": "bool",
                "defaultValue": false,
                "metadata": {
                    "description": "If set to true when changing retention to 30 days, older data will be immediately deleted. Use this with extreme caution. This only applies when retention is being set to 30 days."
                }
            },
            "priceCode": {
                "type": "int",
                "defaultValue": 1,
                "allowedValues": [
                    1,
                    2
                ],
                "metadata": {
                    "description": "Pricing plan: 1 = Per GB (or legacy Basic plan), 2 = Per Node (legacy Enterprise plan)"
                }
            },
            "dailyQuota": {
                "type": "int",
                "defaultValue": 100,
                "minValue": 1,
                "metadata": {
                    "description": "Enter daily quota in GB."
                }
            },
            "dailyQuotaResetTime": {
                "type": "int",
                "defaultValue": 0,
                "metadata": {
                    "description": "Enter daily quota reset hour in UTC (0 to 23). Values outside the range will get a random reset hour."
                }
            },
            "warningThreshold": {
                "type": "int",
                "defaultValue": 90,
                "minValue": 1,
                "maxValue": 100,
                "metadata": {
                    "description": "Enter the % value of daily quota after which warning mail to be sent. "
                }
            }
        },
        "variables": {
            "priceArray": [
                "Basic",
                "Application Insights Enterprise"
            ],
            "pricePlan": "[take(variables('priceArray'),parameters('priceCode'))]",
            "billingplan": "[concat(parameters('appName'),'/', variables('pricePlan')[0])]"
        },
        "resources": [
            {
                "type": "microsoft.insights/components",
                "kind": "[parameters('appType')]",
                "name": "[parameters('appName')]",
                "apiVersion": "2014-04-01",
                "location": "[parameters('appLocation')]",
                "tags": {},
                "properties": {
                    "ApplicationId": "[parameters('appName')]",
                    "retentionInDays": "[parameters('retentionInDays')]"
                },
                "dependsOn": []
            },
            {
                "name": "[variables('billingplan')]",
                "type": "microsoft.insights/components/CurrentBillingFeatures",
                "location": "[parameters('appLocation')]",
                "apiVersion": "2015-05-01",
                "dependsOn": [
                    "[resourceId('microsoft.insights/components', parameters('appName'))]"
                ],
                "properties": {
                    "CurrentBillingFeatures": "[variables('pricePlan')]",
                    "DataVolumeCap": {
                        "Cap": "[parameters('dailyQuota')]",
                        "WarningThreshold": "[parameters('warningThreshold')]",
                        "ResetTime": "[parameters('dailyQuotaResetTime')]"
                    }
                }
            }
        ]
    }
```

### <a name="use-the-resource-manager-template-to-create-a-new-application-insights-resource"></a>Создание нового ресурса Application Insights с помощью шаблона Resource Manager

1. В PowerShell войдите в Azure с помощью `$Connect-AzAccount`.
2. Задайте в качестве контекста подписку с помощью `Set-AzContext "<subscription ID>"`.
2. Выполните новое развертывание, чтобы создать ресурс Application Insights.
   
    ```PS
        New-AzResourceGroupDeployment -ResourceGroupName Fabrikam `
               -TemplateFile .\template1.json `
               -appName myNewApp

    ``` 
   
   * `-ResourceGroupName` — группа, в которой необходимо создать ресурсы.
   * Параметр `-TemplateFile` должен предшествовать пользовательским параметрам.
   * `-appName` — имя создаваемого ресурса.

Можно добавить другие параметры. Их описание доступно в разделе параметров шаблона.

## <a name="get-the-instrumentation-key"></a>Получение ключа инструментирования

После создания ресурса приложения вам понадобится ключ инструментирования. 

1. `$Connect-AzAccount`
2. `Set-AzContext "<subscription ID>"`
3. `$resource = Get-AzResource -Name "<resource name>" -ResourceType "Microsoft.Insights/components"`
4. `$details = Get-AzResource -ResourceId $resource.ResourceId`
5. `$details.Properties.InstrumentationKey`

Чтобы просмотреть список других свойств ресурса Application Insights, выполните следующий код.

```PS
Get-AzApplicationInsights -ResourceGroupName Fabrikam -Name FabrikamProd | Format-List
```

Дополнительные свойства доступны с использованием этих командлетов:
* `Set-AzApplicationInsightsDailyCap`
* `Set-AzApplicationInsightsPricingPlan`
* `Get-AzApplicationInsightsApiKey`
* `Get-AzApplicationInsightsContinuousExport`

Параметры этих командлетов описаны в [подробной документации](/powershell/module/az.applicationinsights).  

## <a name="set-the-data-retention"></a>Настройка срока хранения данных

Ниже приведены три программных способа настройки срока хранения данных для ресурса Application Insights.

### <a name="setting-data-retention-using-a-powershell-commands"></a>Настройка срока хранения данных с помощью команд PowerShell

Ниже приведен простой набор команд PowerShell для настройки срока хранения данных для ресурса Application Insights.

```PS
$Resource = Get-AzResource -ResourceType Microsoft.Insights/components -ResourceGroupName MyResourceGroupName -ResourceName MyResourceName
$Resource.Properties.RetentionInDays = 365
$Resource | Set-AzResource -Force
```

### <a name="setting-data-retention-using-rest"></a>Настройка срока хранения данных с помощью REST

Чтобы получить текущий срок хранения данных для ресурса Application Insights, можно применить средство OSS [ARMClient](https://github.com/projectkudu/ARMClient).  (Дополнительные сведения о клиенте ARMclient см. в статьях авторов [Дэвид Эббо (David Ebbo)](http://blog.davidebbo.com/2015/01/azure-resource-manager-client.html) и [Дэниел Боубайз (Daniel Bowbyes)](https://blog.bowbyes.co.nz/2016/11/02/using-armclient-to-directly-access-azure-arm-rest-apis-and-list-arm-policy-details/).)  В следующем примере текущий срок хранения можно получить с помощью `ARMClient`.

```PS
armclient GET /subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/MyResourceGroupName/providers/microsoft.insights/components/MyResourceName?api-version=2018-05-01-preview
```

Чтобы задать срок хранения, используйте аналогичную команду PUT.

```PS
armclient PUT /subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/MyResourceGroupName/providers/microsoft.insights/components/MyResourceName?api-version=2018-05-01-preview "{location: 'eastus', properties: {'retentionInDays': 365}}"
```

Чтобы задать срок хранения данных длительностью 365 дней, выполните следующее действие с описанным выше шаблоном.

```PS
New-AzResourceGroupDeployment -ResourceGroupName "<resource group>" `
       -TemplateFile .\template1.json `
       -retentionInDays 365 `
       -appName myApp
```

### <a name="setting-data-retention-using-a-powershell-script"></a>Настройка срока хранения данных с помощью скрипта PowerShell

Следующий скрипт также позволяет изменить срок хранения. Скопируйте этот скрипт и сохраните его как `Set-ApplicationInsightsRetention.ps1`.

```PS
Param(
    [Parameter(Mandatory = $True)]
    [string]$SubscriptionId,

    [Parameter(Mandatory = $True)]
    [string]$ResourceGroupName,

    [Parameter(Mandatory = $True)]
    [string]$Name,

    [Parameter(Mandatory = $True)]
    [string]$RetentionInDays
)
$ErrorActionPreference = 'Stop'
if (-not (Get-Module Az.Accounts)) {
    Import-Module Az.Accounts
}
$azProfile = [Microsoft.Azure.Commands.Common.Authentication.Abstractions.AzureRmProfileProvider]::Instance.Profile
if (-not $azProfile.Accounts.Count) {
    Write-Error "Ensure you have logged in before calling this function."    
}
$currentAzureContext = Get-AzContext
$profileClient = New-Object Microsoft.Azure.Commands.ResourceManager.Common.RMProfileClient($azProfile)
$token = $profileClient.AcquireAccessToken($currentAzureContext.Tenant.TenantId)
$UserToken = $token.AccessToken
$RequestUri = "https://management.azure.com/subscriptions/$($SubscriptionId)/resourceGroups/$($ResourceGroupName)/providers/Microsoft.Insights/components/$($Name)?api-version=2015-05-01"
$Headers = @{
    "Authorization"         = "Bearer $UserToken"
    "x-ms-client-tenant-id" = $currentAzureContext.Tenant.TenantId
}
## Get Component object via ARM
$GetResponse = Invoke-RestMethod -Method "GET" -Uri $RequestUri -Headers $Headers 

## Update RetentionInDays property
if($($GetResponse.properties | Get-Member "RetentionInDays"))
{
    $GetResponse.properties.RetentionInDays = $RetentionInDays
}
else
{
    $GetResponse.properties | Add-Member -Type NoteProperty -Name "RetentionInDays" -Value $RetentionInDays
}
## Upsert Component object via ARM
$PutResponse = Invoke-RestMethod -Method "PUT" -Uri "$($RequestUri)" -Headers $Headers -Body $($GetResponse | ConvertTo-Json) -ContentType "application/json"
$PutResponse
```

Теперь вы можете выполнить его следующим образом.

```PS
Set-ApplicationInsightsRetention `
        [-SubscriptionId] <String> `
        [-ResourceGroupName] <String> `
        [-Name] <String> `
        [-RetentionInDays <Int>]
```

## <a name="set-the-daily-cap"></a>Установка ежедневного ограничения

Чтобы получить свойства ежедневного ограничения, используйте командлет [Set-AzApplicationInsightsPricingPlan](/powershell/module/az.applicationinsights/set-azapplicationinsightspricingplan). 

```PS
Set-AzApplicationInsightsDailyCap -ResourceGroupName <resource group> -Name <resource name> | Format-List
```

Чтобы задать свойства ежедневного ограничения, используйте этот же командлет. Например, так можно настроить ограничение в 300 ГБ/день.

```PS
Set-AzApplicationInsightsDailyCap -ResourceGroupName <resource group> -Name <resource name> -DailyCapGB 300
```

Также для получения и задания параметров ежедневного ограничения можно использовать средство [ARMClient](https://github.com/projectkudu/ARMClient).  Чтобы получить текущие значения, выполните следующий код.

```PS
armclient GET /subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/MyResourceGroupName/providers/microsoft.insights/components/MyResourceName/CurrentBillingFeatures?api-version=2018-05-01-preview
```

## <a name="set-the-daily-cap-reset-time"></a>Установка времени сброса ежедневного ограничения

Чтобы задать время сброса ежедневного ограничения, используйте средство [ARMClient](https://github.com/projectkudu/ARMClient). В следующем примере с помощью `ARMClient` можно задать для времени сброса новое значение (12:00 в формате UTC).

```PS
armclient PUT /subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/MyResourceGroupName/providers/microsoft.insights/components/MyResourceName/CurrentBillingFeatures?api-version=2018-05-01-preview "{'CurrentBillingFeatures':['Basic'],'DataVolumeCap':{'ResetTime':12}}"
```

<a id="price"></a>
## <a name="set-the-pricing-plan"></a>Выбор тарифного плана 

Чтобы получить значение тарифного плана, используйте командлет [Set-AzApplicationInsightsPricingPlan](/powershell/module/az.applicationinsights/set-azapplicationinsightspricingplan).

```PS
Set-AzApplicationInsightsPricingPlan -ResourceGroupName <resource group> -Name <resource name> | Format-List
```

Чтобы задать тарифный план, используйте этот же командлет с параметром `-PricingPlan`.  

```PS
Set-AzApplicationInsightsPricingPlan -ResourceGroupName <resource group> -Name <resource name> -PricingPlan Basic
```

Вы также можете задать тарифный план для существующего ресурса Application Insights, используя приведенный выше шаблон Resource Manager, удалив из ресурса выставления счетов ресурс microsoft.insights/components и узел `dependsOn`. Например, чтобы задать план с оплатой за ГБ (ранее — "Базовый"), выполните следующий код.

```PS
        New-AzResourceGroupDeployment -ResourceGroupName "<resource group>" `
               -TemplateFile .\template1.json `
               -priceCode 1 `
               -appName myApp
```

`priceCode` определяется так:

|priceCode|План|
|---|---|
|1|За гигабайт (ранее — "Базовый")|
|2|За узел (ранее — "Корпоративный")|

Наконец, для получения и определения параметров тарифного плана можно использовать средство [ARMClient](https://github.com/projectkudu/ARMClient).  Чтобы получить текущие значения, выполните следующий код.

```PS
armclient GET /subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/MyResourceGroupName/providers/microsoft.insights/components/MyResourceName/CurrentBillingFeatures?api-version=2018-05-01-preview
```

Чтобы задать все эти параметры, выполните следующий код.

```PS
armclient PUT /subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/MyResourceGroupName/providers/microsoft.insights/components/MyResourceName/CurrentBillingFeatures?api-version=2018-05-01-preview
"{'CurrentBillingFeatures':['Basic'],'DataVolumeCap':{'Cap':200,'ResetTime':12,'StopSendNotificationWhenHitCap':true,'WarningThreshold':90,'StopSendNotificationWhenHitThreshold':true}}"
```

Это действие устанавливает ежедневное ограничение в 200 ГБ/день, задает время сброса ежедневного ограничения в 12:00 в формате UTC, настраивает отправку сообщений по электронной почте при достижении этого ограничения и порога предупреждений, а также определяет порог предупреждений на уровне 90 % от значения ограничения.  

## <a name="add-a-metric-alert"></a>Добавление оповещения метрики

Сведения об автоматизации генерации оповещений по метрикам см. [здесь](../alerts/alerts-metric-create-templates.md#template-for-a-simple-static-threshold-metric-alert).


## <a name="add-an-availability-test"></a>Добавление теста доступности

Сведения об автоматизации тестов доступности см. [здесь](../alerts/alerts-metric-create-templates.md#template-for-an-availability-test-along-with-a-metric-alert).

## <a name="add-more-resources"></a>Добавление дополнительных ресурсов

Чтобы автоматизировать создание любого другого ресурса любого типа, создайте пример вручную, а затем скопируйте его код и выполните его параметризацию из [Azure Resource Manager](https://resources.azure.com/). 

1. Откройте [диспетчер ресурсов Azure](https://resources.azure.com/). Перейдите в `subscriptions/resourceGroups/<your resource group>/providers/Microsoft.Insights/components` к своему ресурсу приложения. 
   
    ![Навигация в обозревателе ресурсов Azure](./media/powershell/01.png)
   
    *Компоненты* — это основные ресурсы Application Insights для отображения приложений. Для соответствующих правил оповещения и веб-тестов доступности имеются отдельные ресурсы.
2. Скопируйте JSON компонента в соответствующее место в файле `template1.json`.
3. Удалите следующие свойства:
   
   * `id`
   * `InstrumentationKey`
   * `CreationDate`
   * `TenantId`
4. Откройте разделы `webtests` и `alertrules` и скопируйте код JSON для отдельных элементов в шаблон. (Не копируйте содержимое узлов `webtests` или `alertrules`, а перейдите в элементы под ними.)
   
    С каждым веб-тестом связано правило оповещения, поэтому скопировать необходимо оба элемента.
   
5. Вставьте в каждый ресурс следующую строку.
   
    `"apiVersion": "2015-05-01",`

### <a name="parameterize-the-template"></a>Параметризация шаблона
Теперь отдельные имена необходимо заменить параметрами. Для [параметризации шаблона](../../azure-resource-manager/templates/template-syntax.md) необходимо записать выражения, используя [набор вспомогательных функций](../../azure-resource-manager/templates/template-functions.md). 

Параметризовать только часть строки нельзя, поэтому для формирования строки используйте `concat()` .

Вот примеры возможных замен: Каждая замена используется несколько раз. В вашем шаблоне могут потребоваться другие замены. В данных примерах используются параметры и переменные, определенные в верхней части шаблона.

| поиск | заменить на |
| --- | --- |
| `"hidden-link:/subscriptions/.../../components/MyAppName"` |`"[concat('hidden-link:',`<br/>`resourceId('microsoft.insights/components',` <br/> `parameters('appName')))]"` |
| `"/subscriptions/.../../alertrules/myAlertName-myAppName-subsId",` |`"[resourceId('Microsoft.Insights/alertrules', variables('alertRuleName'))]",` |
| `"/subscriptions/.../../webtests/myTestName-myAppName",` |`"[resourceId('Microsoft.Insights/webtests', parameters('webTestName'))]",` |
| `"myWebTest-myAppName"` |`"[variables(testName)]"'` |
| `"myTestName-myAppName-subsId"` |`"[variables('alertRuleName')]"` |
| `"myAppName"` |`"[parameters('appName')]"` |
| `"myappname"` (строчная) |`"[toLower(parameters('appName'))]"` |
| `"<WebTest Name=\"myWebTest\" ...`<br/>`Url=\"http://fabrikam.com/home\" ...>"` |`[concat('<WebTest Name=\"',` <br/> `parameters('webTestName'),` <br/> `'\" ... Url=\"', parameters('Url'),` <br/> `'\"...>')]"`|

### <a name="set-dependencies-between-the-resources"></a>Установка зависимостей между ресурсами
Служба Azure должна настраивать ресурсы в строгом порядке. Чтобы одна процедура настройки не начиналась прежде, чем завершится предыдущая, добавьте строки зависимости:

* В ресурсе теста доступности:
  
    `"dependsOn": ["[resourceId('Microsoft.Insights/components', parameters('appName'))]"],`
* В ресурсе оповещения для теста доступности:
  
    `"dependsOn": ["[resourceId('Microsoft.Insights/webtests', variables('testName'))]"],`



## <a name="next-steps"></a>Дальнейшие действия
Другие статьи об автоматизации:

* [Создание ресурса Application Insights](./create-new-resource.md#creating-a-resource-automatically) — быстрый метод без использования шаблона.
* [Creating an Application Insights Web Test and Alert Programmatically](../alerts/resource-manager-alerts-metric.md#availability-test-with-metric-alert)
* [Отправка данных системы диагностики Azure в Application Insights](powershell-azure-diagnostics.md)
* [Создание заметок выпуска](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/API/CreateReleaseAnnotation.ps1)