---
title: Создание ресурсов служебной шины Azure с помощью шаблонов
description: С помощью шаблонов Azure Resource Manager можно автоматизировать создание ресурсов служебной шины.
documentationcenter: .net
author: spelluru
ms.topic: article
ms.tgt_pltfrm: dotnet
ms.date: 06/23/2020
ms.author: spelluru
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: df8a7fde9114f03521f0e57e072f81a867efcf39
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89075262"
---
# <a name="create-service-bus-resources-using-azure-resource-manager-templates"></a>Создание ресурсов служебной шины с использованием шаблонов Azure Resource Manager

В этой статье описывается, как создать и развернуть ресурсы служебной шины с помощью шаблонов Azure Resource Manager, PowerShell и поставщика ресурсов служебной шины.

В шаблонах Azure Resource Manager определяются ресурсы для развертывания решения и указываются параметры и переменные, позволяющие вводить значения для различных сред. Шаблон создается в формате JSON и состоит из выражений, на основе которых можно получить значения для развертывания. Дополнительные сведения о создании шаблонов Azure Resource Manager и описание формата шаблонов см. в статье [Описание структуры и синтаксиса шаблонов Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md).

> [!NOTE]
> В примерах в этой статье показано, как использовать Azure Resource Manager для создания пространства имен и объекта обмена сообщениями (очереди) служебной шины. Чтобы узнать о новых шаблонах, в [коллекции шаблонов быстрого запуска Azure][Azure Quickstart Templates gallery] выполните поиск по запросу **служебная шина**.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="service-bus-resource-manager-templates"></a>Шаблоны Resource Manager для служебной шины

Эти шаблоны Azure Resource Manager для служебной шины доступны для скачивания и развертывания. Чтобы получить подробные сведения о каждом из них со ссылками на шаблоны в GitHub, щелкните приведенные ниже ссылки.

* [Создайте пространство имен служебной шины](service-bus-resource-manager-namespace.md)
* [Создание пространства имен служебной шины с очередью](service-bus-resource-manager-namespace-queue.md)
* [Создание пространства имен служебной шины с разделом и подпиской](service-bus-resource-manager-namespace-topic.md)
* [Создание пространства имен служебной шины с очередью и правилом авторизации](service-bus-resource-manager-namespace-auth-rule.md)
* [Создание пространства имен служебной шины с разделом, подпиской и правилом с помощью шаблона Azure Resource Manager](service-bus-resource-manager-namespace-topic-with-rule.md)

## <a name="deploy-with-powershell"></a>Развертывание с помощью PowerShell

Ниже описывается использование PowerShell для развертывания шаблона Azure Resource Manager, который создает пространство имен служебной шины уровня "Стандартный" и очередь в нем. Этот пример основан на шаблоне [Создание пространства имен служебной шины с очередью](https://github.com/Azure/azure-quickstart-templates/tree/master/201-servicebus-create-queue). Ниже приведена примерная последовательность действий.

1. Установите PowerShell.
2. Создайте шаблон и (необязательно) файл параметров.
3. В PowerShell войдите в свою учетную запись Azure.
4. Создайте группу ресурсов, если ее еще нет.
5. Протестируйте развертывание.
6. Если необходимо, задайте режим развертывания.
7. Разверните шаблон.

Подробные сведения о развертывании шаблонов Azure Resource Manager см. в разделе [Развертывание ресурсов с использованием шаблонов Azure Resource Manager][Deploy resources with Azure Resource Manager templates].

### <a name="install-powershell"></a>Установка PowerShell

Установите Azure PowerShell, следуя указаниям в разделе [Getting started with Azure PowerShell](/powershell/azure/get-started-azureps) (Приступая к работе с Azure PowerShell).

### <a name="create-a-template"></a>Создание шаблона

Клонируйте репозиторий и скопируйте шаблон [201-servicebus-create-queue](https://github.com/Azure/azure-quickstart-templates/blob/master/201-servicebus-create-queue/azuredeploy.json) из GitHub:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "serviceBusNamespaceName": {
      "type": "string",
      "metadata": {
        "description": "Name of the Service Bus namespace"
      }
    },
    "serviceBusQueueName": {
      "type": "string",
      "metadata": {
        "description": "Name of the Queue"
      }
    }
  },
  "variables": {
    "defaultSASKeyName": "RootManageSharedAccessKey",
    "authRuleResourceId": "[resourceId('Microsoft.ServiceBus/namespaces/authorizationRules', parameters('serviceBusNamespaceName'), variables('defaultSASKeyName'))]",
    "sbVersion": "2017-04-01"
  },
  "resources": [
    {
      "apiVersion": "2017-04-01",
      "name": "[parameters('serviceBusNamespaceName')]",
      "type": "Microsoft.ServiceBus/Namespaces",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard"
      },
      "properties": {},
      "resources": [
        {
          "apiVersion": "2017-04-01",
          "name": "[parameters('serviceBusQueueName')]",
          "type": "Queues",
          "dependsOn": [
            "[concat('Microsoft.ServiceBus/namespaces/', parameters('serviceBusNamespaceName'))]"
          ],
          "properties": {
            "lockDuration": "PT5M",
            "maxSizeInMegabytes": "1024",
            "requiresDuplicateDetection": "false",
            "requiresSession": "false",
            "defaultMessageTimeToLive": "P10675199DT2H48M5.4775807S",
            "deadLetteringOnMessageExpiration": "false",
            "duplicateDetectionHistoryTimeWindow": "PT10M",
            "maxDeliveryCount": "10",
            "autoDeleteOnIdle": "P10675199DT2H48M5.4775807S",
            "enablePartitioning": "false",
            "enableExpress": "false"
          }
        }
      ]
    }
  ],
  "outputs": {
    "NamespaceConnectionString": {
      "type": "string",
      "value": "[listkeys(variables('authRuleResourceId'), variables('sbVersion')).primaryConnectionString]"
    },
    "SharedAccessPolicyPrimaryKey": {
      "type": "string",
      "value": "[listkeys(variables('authRuleResourceId'), variables('sbVersion')).primaryKey]"
    }
  }
}
```

### <a name="create-a-parameters-file-optional"></a>Создание файла параметров (необязательно)

Чтобы использовать необязательный файл параметров, скопируйте файл [201-servicebus-create-queue](https://github.com/Azure/azure-quickstart-templates/blob/master/201-servicebus-create-queue/azuredeploy.parameters.json). Замените значение `serviceBusNamespaceName` на название пространства имен служебной шины, которое нужно создать в этом развертывании, а значение `serviceBusQueueName` — на имя очереди, которую нужно создать.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "serviceBusNamespaceName": {
            "value": "<myNamespaceName>"
        },
        "serviceBusQueueName": {
            "value": "<myQueueName>"
        },
        "serviceBusApiVersion": {
            "value": "2017-04-01"
        }
    }
}
```

Дополнительные сведения см. в разделе [Параметры](../azure-resource-manager/templates/parameter-files.md).

### <a name="log-in-to-azure-and-set-the-azure-subscription"></a>Вход в Azure и настройка подписки Azure

В командной строке PowerShell выполните следующую команду:

```powershell
Connect-AzAccount
```

Вам будет предложено войти в учетную запись Azure. Войдя в систему, выполните следующую команду, чтобы просмотреть доступные подписки:

```powershell
Get-AzSubscription
```

Эта команда возвращает список доступных подписок Azure. Выберите подписку для текущего сеанса, выполнив приведенную ниже команду. Замените `<YourSubscriptionId>` идентификатором GUID подписки Azure, которую вы хотите использовать:

```powershell
Set-AzContext -SubscriptionID <YourSubscriptionId>
```

### <a name="set-the-resource-group"></a>Задание группы ресурсов

Если у вас нет группы ресурсов, создайте ее с помощью команды **New-AzResourceGroup**. Введите имя группы ресурсов и нужное расположение. Пример:

```powershell
New-AzResourceGroup -Name MyDemoRG -Location "West US"
```

После успешного выполнения операции появится сводка по новой группе ресурсов.

```powershell
ResourceGroupName : MyDemoRG
Location          : westus
ProvisioningState : Succeeded
Tags              :
ResourceId        : /subscriptions/<GUID>/resourceGroups/MyDemoRG
```

### <a name="test-the-deployment"></a>тестирование развертывания

Проверьте развертывание, выполнив командлет `Test-AzResourceGroupDeployment`. При тестировании развернутой службы укажите точно такие же параметры, как и при ее выполнении.

```powershell
Test-AzResourceGroupDeployment -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json
```

### <a name="create-the-deployment"></a>Создание развертывания

Чтобы создать развертывание, выполните командлет `New-AzResourceGroupDeployment` и укажите необходимые параметры при появлении запроса. Параметры включают в себя имя развертывания, имя группы ресурсов и путь к файлу шаблона или его URL-адрес. Если параметр **Режим** не указан, используется стандартное значение **Добавочный**. Дополнительные сведения см. в статье [Добавочные и полные развертывания](../azure-resource-manager/templates/deployment-modes.md).

Следующая команда запрашивает три параметра в окне PowerShell:

```powershell
New-AzResourceGroupDeployment -Name MyDemoDeployment -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json
```

Чтобы использовать вместо этого файл параметров, выполните приведенную ниже команду:

```powershell
New-AzResourceGroupDeployment -Name MyDemoDeployment -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json -TemplateParameterFile <path to parameters file>\azuredeploy.parameters.json
```

При выполнении командлета развертывания также можно использовать встроенные параметры. Команда выглядит следующим образом:

```powershell
New-AzResourceGroupDeployment -Name MyDemoDeployment -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json -parameterName "parameterValue"
```

Чтобы выполнить [полное](../azure-resource-manager/templates/deployment-modes.md) развертывание, установите для параметра **Режим** значение **Полный**.

```powershell
New-AzResourceGroupDeployment -Name MyDemoDeployment -Mode Complete -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json
```

### <a name="verify-the-deployment"></a>Проверка развертывания
В случае успешного развертывания ресурсов в окне PowerShell будет приведена сводка по развертыванию.

```powershell
DeploymentName    : MyDemoDeployment
ResourceGroupName : MyDemoRG
ProvisioningState : Succeeded
Timestamp         : 4/19/2017 10:38:30 PM
Mode              : Incremental
TemplateLink      :
Parameters        :
                    Name             Type                       Value
                    ===============  =========================  ==========
                    serviceBusNamespaceName  String             <namespaceName>
                    serviceBusQueueName  String                 <queueName>
                    serviceBusApiVersion  String                2017-04-01

```

## <a name="next-steps"></a>Дальнейшие действия
Мы рассмотрели базовую процедуру и команды для развертывания шаблона Azure Resource Manager. Для получения более подробных сведений перейдите по следующим ссылкам:

* [Общие сведения об Azure Resource Manager][Azure Resource Manager overview]
* [Развертывание ресурсов с использованием шаблонов Resource Manager и Azure PowerShell][Deploy resources with Azure Resource Manager templates]
* [Создание шаблонов Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md)
* [Microsoft.ServiceBus versions](/azure/templates/microsoft.servicebus/allversions) (Версии типов ресурсов Microsoft.ServiceBus)

[Azure Resource Manager overview]: ../azure-resource-manager/management/overview.md
[Deploy resources with Azure Resource Manager templates]: ../azure-resource-manager/templates/deploy-powershell.md
[Azure Quickstart Templates gallery]: https://azure.microsoft.com/documentation/templates/?term=service+bus
