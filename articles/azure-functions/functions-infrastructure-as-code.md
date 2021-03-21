---
title: Автоматизация развертывания ресурсов приложения функции в Azure
description: Узнайте, как создать шаблон Azure Resource Manager, позволяющий развертывать приложения-функции.
ms.assetid: d20743e3-aab6-442c-a836-9bcea09bfd32
ms.topic: conceptual
ms.date: 04/03/2019
ms.custom: fasttrack-edit
ms.openlocfilehash: 9df4c62a65fd133c6ea8dc84e33d7c7b02d94cbf
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99494045"
---
# <a name="automate-resource-deployment-for-your-function-app-in-azure-functions"></a>Автоматизация развертывания ресурсов приложения-функции для службы "Функции Azure"

Для развертывания приложения-функции можно использовать шаблон Azure Resource Manager. В этой статье рассматриваются необходимые для этого ресурсы и параметры. В зависимости от [триггеров и привязок](functions-triggers-bindings.md) в приложении-функции может потребоваться развернуть дополнительные ресурсы.

Дополнительные сведения о создании шаблонов см. в статье [Создание шаблонов Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md).

Примеры шаблонов см. в следующих статьях:
- [Function app on Consumption plan] (Приложение-функция в плане потребления)
- [Function app on Azure App Service plan] (Приложение-функция в плане службы приложений Azure).

## <a name="required-resources"></a>Необходимые ресурсы

Развертывание функций Azure обычно состоит из следующих ресурсов:

| Ресурс                                                                           | Требование | Справочник по синтаксису и свойствам                                                         |
|------------------------------------------------------------------------------------|-------------|-----------------------------------------------------------------------------------------|
| Приложение-функция.                                                                     | Обязательно    | [Microsoft. Web/Sites](/azure/templates/microsoft.web/sites)                             |
| Учетная запись [хранения Azure](../storage/index.yml) ;                                   | Обязательно    | [Microsoft.Storage/storageAccounts](/azure/templates/microsoft.storage/storageaccounts) |
| Компонент [Application Insights](../azure-monitor/app/app-insights-overview.md) | Необязательно    | [Microsoft. Insights/компоненты](/azure/templates/microsoft.insights/components)         |
| [План размещения](./functions-scale.md)                                             | Необязательно<sup>1</sup>    | [Microsoft. Web/serverfarms](/azure/templates/microsoft.web/serverfarms)                 |

<sup>1</sup> План размещения требуется только в том случае, если вы решили запустить приложение-функцию в [плане Premium](./functions-premium-plan.md) или в [плане службы приложений](../app-service/overview-hosting-plans.md).

> [!TIP]
> Хотя это и не является обязательным, настоятельно рекомендуется настроить Application Insights для приложения.

<a name="storage"></a>
### <a name="storage-account"></a>Учетная запись хранения

Учетная запись хранения Azure — обязательный ресурс для приложения-функции. Необходима учетная запись общего назначения, поддерживающая большие двоичные объекты, таблицы, очереди и файлы. Дополнительные сведения см. в разделе [Требования к учетной записи хранения](storage-considerations.md#storage-account-requirements).

```json
{
    "type": "Microsoft.Storage/storageAccounts",
    "name": "[variables('storageAccountName')]",
    "apiVersion": "2019-06-01",
    "location": "[resourceGroup().location]",
    "kind": "StorageV2",
    "sku": {
        "name": "[parameters('storageAccountType')]"
    }
}
```

Кроме того, необходимо указать свойство `AzureWebJobsStorage` как параметр приложения в конфигурации сайта. Если приложение-функция не использует Application Insights для мониторинга, в нем также следует также указать `AzureWebJobsDashboard` как параметр приложения.

Чтобы создать внутренние очереди, среда выполнения службы "Функции Azure" использует строку подключения `AzureWebJobsStorage`.  Если служба Application Insights не включена, среда выполнения использует строку подключения `AzureWebJobsDashboard` для входа в хранилище таблиц Azure и отображения данных на вкладке **мониторинга** на портале.

Эти свойства задаются в коллекции `appSettings` в объекте `siteConfig`:

```json
"appSettings": [
    {
        "name": "AzureWebJobsStorage",
        "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]"
    },
    {
        "name": "AzureWebJobsDashboard",
        "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]"
    }
]
```

### <a name="application-insights"></a>Application Insights

Для мониторинга приложений-функций рекомендуется использовать Application Insights. Ресурс Application Insights определяется типом **Microsoft. Insights/Components** и типом **Web**:

```json
        {
            "apiVersion": "2015-05-01",
            "name": "[variables('appInsightsName')]",
            "type": "Microsoft.Insights/components",
            "kind": "web",
            "location": "[resourceGroup().location]",
            "tags": {
                "[concat('hidden-link:', resourceGroup().id, '/providers/Microsoft.Web/sites/', variables('functionAppName'))]": "Resource"
            },
            "properties": {
                "Application_Type": "web",
                "ApplicationId": "[variables('appInsightsName')]"
            }
        },
```

Кроме того, ключ инструментирования необходимо предоставить приложению функции, используя `APPINSIGHTS_INSTRUMENTATIONKEY` параметр приложения. Это свойство указывается в `appSettings` коллекции `siteConfig` объекта:

```json
"appSettings": [
    {
        "name": "APPINSIGHTS_INSTRUMENTATIONKEY",
        "value": "[reference(resourceId('microsoft.insights/components/', variables('appInsightsName')), '2015-05-01').InstrumentationKey]"
    }
]
```

### <a name="hosting-plan"></a>План размещения

Определение плана размещения изменяется и может быть одним из следующих:
* [План потребления](#consumption) (по умолчанию)
* [План категории "Премиум"](#premium)
* [План обслуживания приложения](#app-service-plan)

### <a name="function-app"></a>Приложение-функция

Ресурс приложения-функции определяется с помощью ресурса типа **Microsoft. Web/Sites** и типа **functionapp**:

```json
{
    "apiVersion": "2015-08-01",
    "type": "Microsoft.Web/sites",
    "name": "[variables('functionAppName')]",
    "location": "[resourceGroup().location]",
    "kind": "functionapp",
    "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]",
        "[resourceId('Microsoft.Insights/components', variables('appInsightsName'))]"
    ]
}
```

> [!IMPORTANT]
> Если вы явно определяете план размещения, в массиве dependsOn потребуется дополнительный элемент: `"[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]"`

Приложение-функция должно включать следующие параметры приложения:

| Имя параметра                 | Описание                                                                               | Примеры значений                        |
|------------------------------|-------------------------------------------------------------------------------------------|---------------------------------------|
| AzureWebJobsStorage          | Строка подключения к учетной записи хранения, которую среда выполнения функций использует для внутренней очереди | См. [учетную запись хранения](#storage)       |
| FUNCTIONS_EXTENSION_VERSION  | Версия среды выполнения функций Azure                                                | `~3`                                  |
| FUNCTIONS_WORKER_RUNTIME     | Стек языка, используемый для функций в этом приложении                                   | `dotnet`, `node`, `java`, `python` или `powershell` |
| WEBSITE_NODE_DEFAULT_VERSION | Указывает используемую версию, только если используется `node` языковой стек.              | `10.14.1`                             |

Эти свойства указываются в `appSettings` коллекции в `siteConfig` свойстве:

```json
"properties": {
    "siteConfig": {
        "appSettings": [
            {
                "name": "AzureWebJobsStorage",
                "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]"
            },
            {
                "name": "FUNCTIONS_WORKER_RUNTIME",
                "value": "node"
            },
            {
                "name": "WEBSITE_NODE_DEFAULT_VERSION",
                "value": "10.14.1"
            },
            {
                "name": "FUNCTIONS_EXTENSION_VERSION",
                "value": "~3"
            }
        ]
    }
}
```

<a name="consumption"></a>

## <a name="deploy-on-consumption-plan"></a>Развертывание в плане потребления

План потребления автоматически выделяет мощность вычислений при выполнении кода, масштабируется по мере необходимости для обработки нагрузки, а затем масштабируется, когда код не выполняется. Вам не нужно платить за бездействующие виртуальные машины, и вам не нужно резервировать емкость заранее. Дополнительные сведения см. в статье [Масштабирование и размещение Функций Azure](consumption-plan.md).

Образец шаблона диспетчера ресурсов Azure см. на странице [Function app on Consumption plan] (План потребления приложения-функции)

### <a name="create-a-consumption-plan"></a>Создание плана потребления

План потребления не требуется определять. Он будет автоматически создан или выбран для каждого региона при создании самого ресурса приложения-функции.

План потребления — это особый тип ресурса "ферма серверов". Для Windows это можно указать с помощью `Dynamic` значения `computeMode` `sku` свойств и.

```json
{  
   "type":"Microsoft.Web/serverfarms",
   "apiVersion":"2016-09-01",
   "name":"[variables('hostingPlanName')]",
   "location":"[resourceGroup().location]",
   "properties":{  
      "name":"[variables('hostingPlanName')]",
      "computeMode":"Dynamic"
   },
   "sku":{  
      "name":"Y1",
      "tier":"Dynamic",
      "size":"Y1",
      "family":"Y",
      "capacity":0
   }
}
```

> [!NOTE]
> План потребления нельзя явно определить для Linux. Он будет создан автоматически.

Если вы явно определили план потребления, необходимо задать `serverFarmId` свойство в приложении, чтобы оно указывало на идентификатор ресурса плана. Необходимо также убедиться, что у приложения-функции есть `dependsOn` параметр для плана.

### <a name="create-a-function-app"></a>Создание приложения-функции

Параметры, необходимые для приложения-функции, выполняемого в плане потребления, откладываются между Windows и Linux. 

#### <a name="windows"></a>Windows

В Windows для плана потребления требуется дополнительный параметр в конфигурации сайта: [`WEBSITE_CONTENTAZUREFILECONNECTIONSTRING`](functions-app-settings.md#website_contentazurefileconnectionstring) . Это свойство настраивает учетную запись хранения, в которой хранятся код и конфигурация приложения функции.

```json
{
    "apiVersion": "2016-03-01",
    "type": "Microsoft.Web/sites",
    "name": "[variables('functionAppName')]",
    "location": "[resourceGroup().location]",
    "kind": "functionapp",
    "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
    ],
    "properties": {
        "siteConfig": {
            "appSettings": [
                {
                    "name": "AzureWebJobsStorage",
                    "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]"
                },
                {
                    "name": "WEBSITE_CONTENTAZUREFILECONNECTIONSTRING",
                    "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]"
                },
                {
                    "name": "FUNCTIONS_WORKER_RUNTIME",
                    "value": "node"
                },
                {
                    "name": "WEBSITE_NODE_DEFAULT_VERSION",
                    "value": "10.14.1"
                },
                {
                    "name": "FUNCTIONS_EXTENSION_VERSION",
                    "value": "~3"
                }
            ]
        }
    }
}
```

> [!IMPORTANT]
> Не устанавливайте этот [`WEBSITE_CONTENTSHARE`](functions-app-settings.md#website_contentshare) параметр так, как он создается при создании сайта.  

#### <a name="linux"></a>Linux

В Linux приложение функции должно иметь `kind` значение `functionapp,linux` , а свойство должно иметь `reserved` значение `true` . 

```json
{
    "apiVersion": "2016-03-01",
    "type": "Microsoft.Web/sites",
    "name": "[variables('functionAppName')]",
    "location": "[resourceGroup().location]",
    "kind": "functionapp,linux",
    "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
    ],
    "properties": {
        "siteConfig": {
            "appSettings": [
                {
                    "name": "AzureWebJobsStorage",
                    "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountName'),'2019-06-01').keys[0].value)]"
                },
                {
                    "name": "FUNCTIONS_WORKER_RUNTIME",
                    "value": "node"
                },
                {
                    "name": "WEBSITE_NODE_DEFAULT_VERSION",
                    "value": "10.14.1"
                },
                {
                    "name": "FUNCTIONS_EXTENSION_VERSION",
                    "value": "~3"
                }
            ]
        },
        "reserved": true
    }
}
```

[`WEBSITE_CONTENTAZUREFILECONNECTIONSTRING`](functions-app-settings.md#website_contentazurefileconnectionstring)Параметры и [`WEBSITE_CONTENTSHARE`](functions-app-settings.md#website_contentshare) не поддерживаются в Linux.

<a name="premium"></a>
## <a name="deploy-on-premium-plan"></a>Развертывание в плане Premium

План Premium обеспечивает то же масштабирование, что и план потребления, но включает выделенные ресурсы и дополнительные возможности. Дополнительные сведения см. в статье [план функций Azure Premium](./functions-premium-plan.md).

### <a name="create-a-premium-plan"></a>Создание плана "Премиум"

План "Премиум" — это особый тип ресурса "ферма серверов". Его можно указать с помощью `EP1` , `EP2` или `EP3` для `Name` значения свойства в `sku` [объекте Description](/azure/templates/microsoft.web/2018-02-01/serverfarms#skudescription-object).

```json
{
    "type": "Microsoft.Web/serverfarms",
    "apiVersion": "2018-02-01",
    "name": "[parameters('hostingPlanName')]",
    "location": "[resourceGroup().location]",
    "properties": {
        "name": "[parameters('hostingPlanName')]",
        "workerSize": "[parameters('workerSize')]",
        "workerSizeId": "[parameters('workerSizeId')]",
        "numberOfWorkers": "[parameters('numberOfWorkers')]",
        "hostingEnvironment": "[parameters('hostingEnvironment')]",
        "maximumElasticWorkerCount": "20"
    },
    "sku": {
        "Tier": "ElasticPremium",
        "Name": "EP1"
    }
}
```

### <a name="create-a-function-app"></a>Создание приложения-функции

Для приложения-функции в плане Premium должно быть `serverFarmId` задано значение идентификатора ресурса созданного ранее плана. Кроме того, для плана Premium требуется дополнительный параметр в конфигурации сайта: [`WEBSITE_CONTENTAZUREFILECONNECTIONSTRING`](functions-app-settings.md#website_contentazurefileconnectionstring) . Это свойство настраивает учетную запись хранения, в которой хранятся код и конфигурация приложения функции.

```json
{
    "apiVersion": "2016-03-01",
    "type": "Microsoft.Web/sites",
    "name": "[variables('functionAppName')]",
    "location": "[resourceGroup().location]",
    "kind": "functionapp",            
    "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
    ],
    "properties": {
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "siteConfig": {
            "appSettings": [
                {
                    "name": "AzureWebJobsStorage",
                    "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]"
                },
                {
                    "name": "WEBSITE_CONTENTAZUREFILECONNECTIONSTRING",
                    "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]"
                },
                {
                    "name": "FUNCTIONS_WORKER_RUNTIME",
                    "value": "node"
                },
                {
                    "name": "WEBSITE_NODE_DEFAULT_VERSION",
                    "value": "10.14.1"
                },
                {
                    "name": "FUNCTIONS_EXTENSION_VERSION",
                    "value": "~3"
                }
            ]
        }
    }
}
```
> [!IMPORTANT]
> Не устанавливайте этот [`WEBSITE_CONTENTSHARE`](functions-app-settings.md#website_contentshare) параметр так, как он создается при создании сайта.  

<a name="app-service-plan"></a>

## <a name="deploy-on-app-service-plan"></a>Развертывание в плане службы приложений

В плане службы приложений ваши приложения-функции запускаются на выделенных виртуальных машинах на Basic, Standard и Premium SKU аналогично веб-приложениям. Дополнительную информацию о том, как действует план службы приложений, см. в статье [Подробный обзор планов службы приложений Azure](../app-service/overview-hosting-plans.md).

Образец шаблона Azure Resource Manager см. на странице [Function app on Azure App Service plan] (Приложение-функция в плане службы приложений Azure).

### <a name="create-an-app-service-plan"></a>Создание плана службы приложений

План службы приложений определяется ресурсом "ферма серверов".

```json
{
    "type": "Microsoft.Web/serverfarms",
    "apiVersion": "2018-02-01",
    "name": "[variables('hostingPlanName')]",
    "location": "[resourceGroup().location]",
    "sku": {
        "name": "S1",
        "tier": "Standard",
        "size": "S1",
        "family": "S",
        "capacity": 1
    }
}
```

Чтобы запустить приложение в Linux, необходимо также задать для параметра значение `kind` `Linux` :

```json
{
    "type": "Microsoft.Web/serverfarms",
    "apiVersion": "2018-02-01",
    "name": "[variables('hostingPlanName')]",
    "location": "[resourceGroup().location]",
    "kind": "Linux",
    "sku": {
        "name": "S1",
        "tier": "Standard",
        "size": "S1",
        "family": "S",
        "capacity": 1
    }
}
```

### <a name="create-a-function-app"></a>Создание приложения-функции

Для приложения-функции в плане службы приложений свойство должно иметь `serverFarmId` значение, равное идентификатору ресурса созданного ранее плана.

```json
{
    "apiVersion": "2016-03-01",
    "type": "Microsoft.Web/sites",
    "name": "[variables('functionAppName')]",
    "location": "[resourceGroup().location]",
    "kind": "functionapp",
    "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
    ],
    "properties": {
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "siteConfig": {
            "appSettings": [
                {
                    "name": "AzureWebJobsStorage",
                    "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]"
                },
                {
                    "name": "FUNCTIONS_WORKER_RUNTIME",
                    "value": "node"
                },
                {
                    "name": "WEBSITE_NODE_DEFAULT_VERSION",
                    "value": "10.14.1"
                },
                {
                    "name": "FUNCTIONS_EXTENSION_VERSION",
                    "value": "~3"
                }
            ]
        }
    }
}
```

Приложения Linux также должны содержать `linuxFxVersion` свойство в разделе `siteConfig` . Если вы только разворачиваете код, значение для этого определяется требуемым стеком среды выполнения в ```runtime|runtimeVersion``` следующем формате:

| Стек            | Пример значения                                         |
|------------------|-------------------------------------------------------|
| Python           | `python|3.7`      |
| JavaScript       | `node|12`          |
| .NET             | `dotnet|3.1` |

```json
{
    "apiVersion": "2016-03-01",
    "type": "Microsoft.Web/sites",
    "name": "[variables('functionAppName')]",
    "location": "[resourceGroup().location]",
    "kind": "functionapp",
    "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
    ],
    "properties": {
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "siteConfig": {
            "appSettings": [
                {
                    "name": "AzureWebJobsStorage",
                    "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]"
                },
                {
                    "name": "FUNCTIONS_WORKER_RUNTIME",
                    "value": "node"
                },
                {
                    "name": "WEBSITE_NODE_DEFAULT_VERSION",
                    "value": "10.14.1"
                },
                {
                    "name": "FUNCTIONS_EXTENSION_VERSION",
                    "value": "~3"
                }
            ],
            "linuxFxVersion": "node|12"
        }
    }
}
```

Если вы [развертываете пользовательский образ контейнера](./functions-create-function-linux-custom-image.md), необходимо указать его с помощью `linuxFxVersion` и включить конфигурацию, позволяющую получать образ, как в [веб-приложение для контейнеров](../app-service/index.yml). Кроме того, задайте `WEBSITES_ENABLE_APP_SERVICE_STORAGE` для значение `false` , так как содержимое приложения предоставляется в контейнере:

```json
{
    "apiVersion": "2016-03-01",
    "type": "Microsoft.Web/sites",
    "name": "[variables('functionAppName')]",
    "location": "[resourceGroup().location]",
    "kind": "functionapp",
    "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
    ],
    "properties": {
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "siteConfig": {
            "appSettings": [
                {
                    "name": "AzureWebJobsStorage",
                    "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]"
                },
                {
                    "name": "FUNCTIONS_WORKER_RUNTIME",
                    "value": "node"
                },
                {
                    "name": "WEBSITE_NODE_DEFAULT_VERSION",
                    "value": "10.14.1"
                },
                {
                    "name": "FUNCTIONS_EXTENSION_VERSION",
                    "value": "~3"
                },
                {
                    "name": "DOCKER_REGISTRY_SERVER_URL",
                    "value": "[parameters('dockerRegistryUrl')]"
                },
                {
                    "name": "DOCKER_REGISTRY_SERVER_USERNAME",
                    "value": "[parameters('dockerRegistryUsername')]"
                },
                {
                    "name": "DOCKER_REGISTRY_SERVER_PASSWORD",
                    "value": "[parameters('dockerRegistryPassword')]"
                },
                {
                    "name": "WEBSITES_ENABLE_APP_SERVICE_STORAGE",
                    "value": "false"
                }
            ],
            "linuxFxVersion": "DOCKER|myacr.azurecr.io/myimage:mytag"
        }
    }
}
```

## <a name="customizing-a-deployment"></a>Настройка развертывания

Приложение-функция содержит много дочерних ресурсов, которые можно использовать при развертывании, в том числе параметры приложения и параметры системы управления версиями. Вы также можете удалить дочерний ресурс **sourcecontrols** и использовать вместо него другой [вариант развертывания](functions-continuous-deployment.md) .

> [!IMPORTANT]
> Чтобы с помощью Azure Resource Manager успешно развернуть приложение, важно понимать, каким образом ресурсы развертываются в Azure. В следующем примере конфигурации верхнего уровня применяются с помощью **siteConfig**. Их важно задать на верхнем уровне, так как эти конфигурации передают сведения в среду выполнения функций и механизм развертывания. Сведения верхнего уровня требуются перед применением дочернего ресурса **sourcecontrols/web**. Хотя эти параметры можно настроить в дочернем ресурсе **config/appSettings**, в некоторых сценариях приложение-функцию требуется развернуть *до применения* **config/appSettings**. В таких случаях, например в [Logic Apps](../logic-apps/index.yml), функции зависят от другого ресурса.

```json
{
  "apiVersion": "2015-08-01",
  "name": "[parameters('appName')]",
  "type": "Microsoft.Web/sites",
  "kind": "functionapp",
  "location": "[parameters('location')]",
  "dependsOn": [
    "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]",
    "[resourceId('Microsoft.Web/serverfarms', parameters('appName'))]"
  ],
  "properties": {
     "serverFarmId": "[variables('appServicePlanName')]",
     "siteConfig": {
        "alwaysOn": true,
        "appSettings": [
            {
                "name": "FUNCTIONS_EXTENSION_VERSION",
                "value": "~3"
            },
            {
                "name": "Project",
                "value": "src"
            }
        ]
     }
  },
  "resources": [
     {
        "apiVersion": "2015-08-01",
        "name": "appsettings",
        "type": "config",
        "dependsOn": [
          "[resourceId('Microsoft.Web/Sites', parameters('appName'))]",
          "[resourceId('Microsoft.Web/Sites/sourcecontrols', parameters('appName'), 'web')]",
          "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
        ],
        "properties": {
          "AzureWebJobsStorage": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]",
          "AzureWebJobsDashboard": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]",
          "FUNCTIONS_EXTENSION_VERSION": "~3",
          "FUNCTIONS_WORKER_RUNTIME": "dotnet",
          "Project": "src"
        }
     },
     {
          "apiVersion": "2015-08-01",
          "name": "web",
          "type": "sourcecontrols",
          "dependsOn": [
            "[resourceId('Microsoft.Web/sites/', parameters('appName'))]"
          ],
          "properties": {
            "RepoUrl": "[parameters('sourceCodeRepositoryURL')]",
            "branch": "[parameters('sourceCodeBranch')]",
            "IsManualIntegration": "[parameters('sourceCodeManualIntegration')]"
          }
     }
  ]
}
```
> [!TIP]
> Этот шаблон использует значение параметров приложения [проекта](https://github.com/projectkudu/kudu/wiki/Customizing-deployments#using-app-settings-instead-of-a-deployment-file) , которое задает базовый каталог, в котором подсистема развертывания функций (KUDU) ищет развертываемый код. В нашем репозитории функции находятся во вложенной папке папки **src**. Таким образом, в предыдущем примере мы задаем для параметров приложения значение `src`. Если ваши функции находятся в корневой папке репозитория, или если развертывание выполняется не из системы управления версиями, то это значение параметров приложения можно удалить.

## <a name="deploy-your-template"></a>Развертывание шаблона

Для развертывания шаблона можно использовать любой из следующих способов:

* [PowerShell](../azure-resource-manager/templates/deploy-powershell.md)
* [Azure CLI](../azure-resource-manager/templates/deploy-cli.md)
* [Портал Azure](../azure-resource-manager/templates/deploy-portal.md)
* [REST API](../azure-resource-manager/templates/deploy-rest.md)

### <a name="deploy-to-azure-button"></a>Кнопка "Развертывание в Azure"

Замените ```<url-encoded-path-to-azuredeploy-json>``` версией необработанного пути к файлу `azuredeploy.json` на сайте GitHub, указав его в формате [URL-адреса](https://www.bing.com/search?q=url+encode).

Ниже приведен пример использования разметки:

```markdown
[![Deploy to Azure](https://azuredeploy.net/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/<url-encoded-path-to-azuredeploy-json>)
```

Ниже приведен пример использования HTML:

```html
<a href="https://portal.azure.com/#create/Microsoft.Template/uri/<url-encoded-path-to-azuredeploy-json>" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"></a>
```

### <a name="deploy-using-powershell"></a>Развертывание с помощью PowerShell

Следующие команды PowerShell создают группу ресурсов и развертывают шаблон, который создает приложение-функцию с необходимыми ресурсами. Для локального запуска необходимо установить [Azure PowerShell](/powershell/azure/install-az-ps) . Выполните команду [`Connect-AzAccount`](/powershell/module/az.accounts/connect-azaccount) , чтобы войти.

```powershell
# Register Resource Providers if they're not already registered
Register-AzResourceProvider -ProviderNamespace "microsoft.web"
Register-AzResourceProvider -ProviderNamespace "microsoft.storage"

# Create a resource group for the function app
New-AzResourceGroup -Name "MyResourceGroup" -Location 'West Europe'

# Create the parameters for the file, which for this template is the function app name.
$TemplateParams = @{"appName" = "<function-app-name>"}

# Deploy the template
New-AzResourceGroupDeployment -ResourceGroupName "MyResourceGroup" -TemplateFile template.json -TemplateParameterObject $TemplateParams -Verbose
```

Чтобы протестировать это развертывание, можно использовать [шаблон, аналогичный этому](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-function-app-create-dynamic/azuredeploy.json) , который создает приложение-функцию в Windows в плане потребления. Замените `<function-app-name>` уникальным именем для приложения-функции.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о разработке и настройке Функций Azure:

* [Справочник разработчика по функциям Azure](functions-reference.md)
* [Настройка параметров приложения функции Azure](functions-how-to-use-azure-function-app-settings.md)
* [Создание первой функции Azure](./functions-get-started.md)

<!-- LINKS -->

[Function app on Consumption plan]: https://github.com/Azure/azure-quickstart-templates/blob/master/101-function-app-create-dynamic/azuredeploy.json (Приложение-функция в плане потребления)
[Function app on Azure App Service plan]: https://github.com/Azure/azure-quickstart-templates/blob/master/101-function-app-create-dedicated/azuredeploy.json (Приложение-функция в плане службы приложений Azure).
