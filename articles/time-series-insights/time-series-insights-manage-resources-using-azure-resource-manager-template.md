---
title: Управление средой с помощью шаблонов Azure Resource Manager — Аналитика временных рядов Azure | Документация Майкрософт
description: Узнайте, как программно управлять средой "Аналитика временных рядов Azure" с помощью Azure Resource Manager.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.devlang: csharp
ms.workload: big-data
ms.topic: conceptual
ms.date: 09/30/2020
ms.custom: seodec18
ms.openlocfilehash: 493750e69b1fdc935b04d6dc705cfd046b6b086e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96011665"
---
# <a name="create-azure-time-series-insights-gen-1-resources-using-azure-resource-manager-templates"></a>Создание ресурсов "аналитика временных рядов Azure" с помощью шаблонов Azure Resource Manager

> [!CAUTION]
> Эта статья посвящена службе "Аналитика временных рядов Azure" 1-го поколения.

В этой статье описывается, как создать и развернуть ресурсы службы "аналитика временных рядов Azure" с помощью [шаблонов Azure Resource Manager](../azure-resource-manager/index.yml), PowerShell и поставщика ресурсов Azure Time Series Insights.

Служба "аналитика временных рядов Azure" поддерживает следующие ресурсы:

   | Ресурс | Описание |
   | --- | --- |
   | Среда | Среда службы "аналитика временных рядов Azure" — это логическая группа событий, которые считываются из брокера событий, хранятся и становятся доступными для запроса. Дополнительные сведения см. в руководстве по [планированию среды "Аналитика временных рядов Azure"](time-series-insights-environment-planning.md). |
   | Источник события | Источник событий — это соединение с брокером событий, из которого Azure Time Series Insights считывает и принимает события в среде. Источники событий, поддерживаемые в настоящее время: Центр Интернета вещей и концентратор событий. |
   | Набор эталонных данных | Наборы эталонных данных предоставляют метаданные о событиях в среде. Метаданные в наборах эталонных данных во время приема будут соединены с событиями. Наборы эталонных данных определяются своими свойствами ключей событий как ресурсы. Фактические метаданные, составляющие набор эталонных данных, отправляются или изменяются через API плоскости данных. |
   | Политика доступа | Политики доступа предоставляют разрешения для отправки запросов данных, обработки ссылочных данных в среде, а также предоставляют другим пользователям среды доступ к сохраненным запросам и перспективам. Дополнительные сведения см. [в статье предоставление доступа к данным в среде службы "аналитика временных рядов Azure" с помощью портал Azure](./concepts-access-policies.md) |

Шаблоны Resource Manager являются файлами в формате JSON, определяющими инфраструктуру и конфигурацию ресурсов в группе ресурсов. В следующих документах файлы шаблонов описываются более подробно:

- [Развертывание шаблона Azure Resource Manager](../azure-resource-manager/templates/overview.md)
- [Развертывание ресурсов с использованием шаблонов Resource Manager и Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md)
- [Microsoft.TimeSeriesInsights resource types](/azure/templates/microsoft.timeseriesinsights/allversions) (Типы ресурсов Microsoft.TimeSeriesInsights)

Шаблон быстрого запуска [201-timeseriesinsights-environment-with-eventhub](https://github.com/Azure/azure-quickstart-templates/tree/master/201-timeseriesinsights-environment-with-eventhub) опубликован на GitHub. Этот шаблон создает среду службы "аналитика временных рядов Azure", дочерний источник событий, настроенный для использования событий из концентратора событий, и политики доступа, предоставляющие доступ к данным среды. Если не указан имеющийся концентратор событий, он будет создан при развертывании.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="specify-deployment-template-and-parameters"></a>Указание шаблона и параметров развертывания

В следующей процедуре описывается, как использовать PowerShell для развертывания шаблона Azure Resource Manager, который создает среду службы "аналитика временных рядов Azure", дочернего источника событий, настроенного для использования событий из концентратора событий, и политик доступа, которые предоставляют доступ к данным среды. Если не указан имеющийся концентратор событий, он будет создан при развертывании.

1. Установите Azure PowerShell, следуя указаниям в разделе [Getting started with Azure PowerShell](/powershell/azure/get-started-azureps) (Приступая к работе с Azure PowerShell).

1. Клонируйте или скопируйте шаблон [201-timeseriesinsights-environment-with-eventhub](https://github.com/Azure/azure-quickstart-templates/blob/master/201-timeseriesinsights-environment-with-eventhub/azuredeploy.json) из GitHub.

   - Создание файла параметров

     Чтобы создать файл параметров, скопируйте файл [201-timeseriesinsights-environment-with-eventhub](https://github.com/Azure/azure-quickstart-templates/blob/master/201-timeseriesinsights-environment-with-eventhub/azuredeploy.parameters.json).

      [!code-json[deployment-parameters](~/quickstart-templates/201-timeseriesinsights-environment-with-eventhub/azuredeploy.parameters.json)]

    <div id="required-parameters"></div>

   - Необходимые параметры

     | Параметр | Описание |
     | --- | --- |
     | eventHubNamespaceName | Пространство имен концентратора событий источника. |
     | eventHubName | Имя концентратора событий источника. |
     | consumerGroupName | Имя группы потребителей, которую служба "аналитика временных рядов Azure" будет использовать для чтения данных из концентратора событий. **Примечание.** Чтобы избежать конфликта ресурсов, эта группа потребителей должна быть выделена службе "аналитика временных рядов Azure" и не была предоставлена другим читателям. |
     | EnvironmentName | Имя среды. Имя не может содержать `<`, `>`, `%`, `&`, `:`, `\\`, `?`, `/` и любые управляющие символы. Все остальные символы разрешены.|
     | eventSourceName | Имя дочернего ресурса источника событий. Имя не может содержать `<`, `>`, `%`, `&`, `:`, `\\`, `?`, `/` и любые управляющие символы. Все остальные символы разрешены. |

    <div id="optional-parameters"></div>

   - Необязательные параметры

     | Параметр | Описание |
     | --- | --- |
     | existingEventHubResourceId | Необязательный идентификатор ресурса существующего концентратора событий, который будет подключен к среде службы "аналитика временных рядов Azure" через источник событий. **ПРИМЕЧАНИЕ.** Пользователь, развертывающий шаблоны, должен иметь привилегии, чтобы выполнять операцию listkeys в концентраторе событий. Если значение не передается, новый концентратор событий будет создан с помощью шаблона. |
     | environmentDisplayName | Дополнительное понятное имя среды, отображаемое в инструменте или пользовательских интерфейсах вместо имени среды. |
     | environmentSkuName | Имя номера SKU. Дополнительные сведения см. на странице цен на службу " [аналитика временных рядов Azure](https://azure.microsoft.com/pricing/details/time-series-insights/)".  |
     | environmentSkuCapacity | Емкость единиц номера SKU. Дополнительные сведения см. на странице цен на службу " [аналитика временных рядов Azure](https://azure.microsoft.com/pricing/details/time-series-insights/)".|
     | environmentDataRetentionTime | Минимальный период, в течение которого события среды будут доступны для запроса. Значение указывается в формате ISO 8601, например `P30D` для политики хранения в течение 30 дней. |
     | eventSourceDisplayName | Дополнительное понятное имя среды, отображаемое в инструменте или пользовательских интерфейсах вместо имени источника события. |
     | eventSourceTimestampPropertyName | Свойство события, которое будет использоваться в качестве метки времени источника события. Если для timestampPropertyName значение не указано, задано значение NULL или указана пустая строка, будет использоваться время создания события. |
     | eventSourceKeyName | Имя общего ключа доступа, который будет использоваться службой "аналитика временных рядов Azure" для подключения к концентратору событий. |
     | accessPolicyReaderObjectIds | Список идентификаторов объектов пользователей или приложений в Azure AD, которые должны иметь доступ на чтение к среде. Идентификатор объекта субъекта-службы можно получить, вызвав командлет **Get-AzADUser** или **Get-AzADServicePrincipal**. Создание политики доступа для группы Azure AD еще не поддерживается. |
     | accessPolicyContributorObjectIds | Список идентификаторов объектов пользователей или приложений в Azure AD, которые должны иметь доступ участника к среде. Идентификатор объекта субъекта-службы можно получить, вызвав командлет **Get-AzADUser** или **Get-AzADServicePrincipal**. Создание политики доступа для группы Azure AD еще не поддерживается. |

   - Например, следующий файл параметров будет использоваться для создания среды и источника событий, который считывает события из имеющегося концентратора событий. Он также создает две политики доступа, которые предоставляют доступ участника к среде.

     ```JSON
     {
         "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
         "contentVersion": "1.0.0.0",
         "parameters": {
             "eventHubNamespaceName": {
                 "value": "tsiTemplateTestNamespace"
             },
             "eventHubName": {
                 "value": "tsiTemplateTestEventHub"
             },
             "consumerGroupName": {
                 "value": "tsiTemplateTestConsumerGroup"
             },
             "environmentName": {
                 "value": "tsiTemplateTestEnvironment"
             },
             "eventSourceName": {
                 "value": "tsiTemplateTestEventSource"
             },
             "existingEventHubResourceId": {
                 "value": "/subscriptions/{yourSubscription}/resourceGroups/MyDemoRG/providers/Microsoft.EventHub/namespaces/tsiTemplateTestNamespace/eventhubs/tsiTemplateTestEventHub"
             },
             "accessPolicyContributorObjectIds": {
                 "value": [
                     "AGUID001-0000-0000-0000-000000000000",
                     "AGUID002-0000-0000-0000-000000000000"
                 ]
             }
         }
     }
     ```

   - Дополнительные сведения см. в статье [Параметры](../azure-resource-manager/templates/parameter-files.md).

## <a name="deploy-the-quickstart-template-locally-using-powershell"></a>Развертывание шаблона быстрого запуска локально с помощью PowerShell

> [!IMPORTANT]
> Операции командной строки, показанные ниже, описывают [модуль PowerShell Az](/powershell/azure/).

1. В PowerShell войдите в свою учетную запись Azure.

    - В командной строке PowerShell выполните следующую команду:

      ```powershell
      Connect-AzAccount
      ```

    - Вам будет предложено войти в учетную запись Azure. Войдя в систему, выполните следующую команду, чтобы просмотреть доступные подписки:

      ```powershell
      Get-AzSubscription
      ```

    - Эта команда возвращает список доступных подписок Azure. Выберите подписку для текущего сеанса, выполнив приведенную ниже команду. Замените `<YourSubscriptionId>` идентификатором GUID подписки Azure, которую вы хотите использовать:

      ```powershell
      Set-AzContext -SubscriptionID <YourSubscriptionId>
      ```

1. Создайте группу ресурсов, если ее еще нет.

   - Если у вас нет группы ресурсов, создайте ее с помощью команды **New-AzResourceGroup**. Введите имя группы ресурсов и нужное расположение. Пример:

     ```powershell
     New-AzResourceGroup -Name MyDemoRG -Location "West US"
     ```

   - После успешного выполнения операции появится сводка по новой группе ресурсов.

     ```powershell
     ResourceGroupName : MyDemoRG
     Location          : westus
     ProvisioningState : Succeeded
     Tags              :
     ResourceId        : /subscriptions/<GUID>/resourceGroups/MyDemoRG
     ```

1. Протестируйте развертывание.

   - Проверьте развертывание, выполнив командлет `Test-AzResourceGroupDeployment`. При тестировании развернутой службы укажите точно такие же параметры, как и при ее выполнении.

     ```powershell
     Test-AzResourceGroupDeployment -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json -TemplateParameterFile <path to parameters file>\azuredeploy.parameters.json
     ```

1. Создание развертывания

    - Чтобы создать развертывание, выполните командлет `New-AzResourceGroupDeployment` и укажите необходимые параметры при появлении запроса. Параметры включают в себя имя развертывания, имя группы ресурсов и путь к файлу шаблона или его URL-адрес. Если параметр **Режим** не указан, используется стандартное значение **Добавочный**. Дополнительные сведения см. в статье [Добавочные и полные развертывания](../azure-resource-manager/templates/deployment-modes.md).

    - Следующая команда запрашивает пять обязательных параметров в окне PowerShell:

      ```powershell
      New-AzResourceGroupDeployment -Name MyDemoDeployment -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json
      ```

    - Чтобы использовать вместо этого файл параметров, выполните приведенную ниже команду:

      ```powershell
      New-AzResourceGroupDeployment -Name MyDemoDeployment -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json -TemplateParameterFile <path to parameters file>\azuredeploy.parameters.json
      ```

    - При выполнении командлета развертывания также можно использовать встроенные параметры. Команда выглядит следующим образом:

      ```powershell
      New-AzResourceGroupDeployment -Name MyDemoDeployment -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json -parameterName "parameterValue"
      ```

    - Чтобы выполнить [полное](../azure-resource-manager/templates/deployment-modes.md) развертывание, установите для параметра **Режим** значение **Полный**.

      ```powershell
      New-AzResourceGroupDeployment -Name MyDemoDeployment -Mode Complete -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json
      ```

1. Проверка развертывания

    - В случае успешного развертывания ресурсов в окне PowerShell будет приведена сводка по развертыванию.

      ```powershell
       DeploymentName          : MyDemoDeployment
       ResourceGroupName       : MyDemoRG
       ProvisioningState       : Succeeded
       Timestamp               : 10/11/2019 3:20:37 AM
       Mode                    : Incremental
       TemplateLink            :
       Parameters              :
                                 Name                                Type                       Value
                                 ==================================  =========================  ==========
                                 eventHubNewOrExisting               String                     new
                                 eventHubResourceGroup               String                     MyDemoRG
                                 eventHubNamespaceName               String                     tsiquickstartns
                                 eventHubName                        String                     tsiquickstarteh
                                 consumerGroupName                   String                     tsiquickstart
                                 environmentName                     String                     tsiquickstart
                                 environmentDisplayName              String                     tsiquickstart
                                 environmentSkuName                  String                     S1
                                 environmentSkuCapacity              Int                        1
                                 environmentDataRetentionTime        String                     P30D
                                 eventSourceName                     String                     tsiquickstart
                                 eventSourceDisplayName              String                     tsiquickstart
                                 eventSourceTimestampPropertyName    String
                                 eventSourceKeyName                  String                     manage
                                 accessPolicyReaderObjectIds         Array                      []
                                 accessPolicyContributorObjectIds    Array                      []
                                 location                            String                     westus

       Outputs                 :
                                  Name              Type                       Value
                                  ================  =========================  ==========
                                  dataAccessFQDN    String
                                  11aa1aa1-a1aa-1a1a-a11a-aa111a111a11.env.timeseries.azure.com

       DeploymentDebugLogLevel :
      ```

1. Развертывание шаблона быстрого запуска через портал Azure

   - Домашняя страница шаблона быстрого запуска на GitHub также содержит кнопку **Развертывание в Azure**. При нажатии на нее открывается страница "Настраиваемое развертывание" на портале Azure. На этой странице можно ввести или выбрать значения для каждого параметра из таблиц [обязательных](#required-parameters) или [необязательных параметров](#optional-parameters). Если нажать кнопку **Приобрести** после заполнения параметров, начнется развертывание шаблона.
    </br>
    </br>
    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-timeseriesinsights-environment-with-eventhub%2Fazuredeploy.json" target="_blank">
       <img src="https://azuredeploy.net/deploybutton.png" alt="The Deploy to Azure button."/>
    </a>

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о программном управлении ресурсами службы "аналитика временных рядов Azure" с помощью API-интерфейсов RESTFUL см. в статье [Управление аналитикой временных рядов Azure](/rest/api/time-series-insights-management/).