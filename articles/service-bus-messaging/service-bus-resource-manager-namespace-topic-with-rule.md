---
title: Создание подписки и правила раздела служебной шины с помощью шаблона Azure
description: Создание пространства имен служебной шины с разделом, подпиской и правилом с помощью шаблона Azure Resource Manager.
author: spelluru
ms.topic: article
ms.tgt_pltfrm: dotnet
ms.date: 06/23/2020
ms.author: spelluru
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 3f3287dd67f89f678a9875ddce93e2d0d26b2209
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89077630"
---
# <a name="create-a-service-bus-namespace-with-topic-subscription-and-rule-using-an-azure-resource-manager-template"></a>Создание пространства имен служебной шины с разделом, подпиской и правилом с помощью шаблона Azure Resource Manager

В этой статье показывается, как использовать шаблон Azure Resource Manager, создающий пространство имен служебной шины с разделом, подпиской и правилом (фильтром). Здесь объясняется, как указать развертываемые ресурсы и определить параметры, указываемые при развертывании. Этот шаблон можно использовать для собственных развертываний или изменить его в соответствии с вашими требованиями.

Дополнительные сведения о создании шаблонов см. в статье [Создание шаблонов Azure Resource Manager][Authoring Azure Resource Manager templates].

Дополнительные сведения о практиках и шаблонах соглашений об именовании ресурсов Azure см. в разделе [Recommended naming conventions for Azure resources][Recommended naming conventions for Azure resources] (Рекомендуемые соглашения об именовании ресурсов Azure).

Полный шаблон пространства имен служебной шины с разделом, подпиской и правилом приведен [здесь][Service Bus namespace with topic, subscription, and rule].

> [!NOTE]
> Для скачивания и развертывания можно использовать указанные ниже шаблоны диспетчера ресурсов Azure.
> 
> * [Создание пространства имен служебной шины с очередью и правилом авторизации](service-bus-resource-manager-namespace-auth-rule.md)
> * [Создание пространства имен служебной шины с очередью](service-bus-resource-manager-namespace-queue.md)
> * [Создайте пространство имен служебной шины](service-bus-resource-manager-namespace.md)
> * [Создание пространства имен служебной шины с разделом и подпиской](service-bus-resource-manager-namespace-topic.md)
> 
> Чтобы узнать о новых шаблонах, в коллекции [шаблонов быстрого запуска Azure][Azure Quickstart Templates] выполните поиск "Service Bus".

## <a name="what-do-you-deploy"></a>Какой объект развертывается?

С помощью этого шаблона вы развернете пространство имен служебной шины с разделом, подпиской и правилом (фильтром).

[Разделы и подписки служебной шины](service-bus-queues-topics-subscriptions.md#topics-and-subscriptions) предоставляют разновидность взаимодействия "один ко многим" в рамках схемы *публикации или подписки*. При использовании разделов и подписок компоненты распределенного приложения не взаимодействуют напрямую друг с другом, а обмениваются сообщениями через раздел, который выступает в качестве посредника. Подписка на раздел напоминает виртуальную очередь, которая получает копии сообщений, отправленных в раздел. Фильтр позволяет определить, какие посылаемые в раздел сообщения должны появляться в определенной подписке на этот раздел.

## <a name="what-are-rules-filters"></a>Что такое правила (фильтры)?

Во многих ситуациях сообщения с определенными характеристиками должны обрабатываться разными способами. Чтобы включить пользовательскую обработку, можно настроить подписки, обеспечивающие поиск сообщений с нужными свойствами, после чего можно изменить эти свойства. Подписки служебной шины регистрируют все сообщения, отправленные в раздел, однако в виртуальную очередь подписки можно скопировать только подмножество этих сообщений. Это возможно благодаря использованию фильтров подписок. Чтобы узнать больше о правилах (фильтрах), изучите раздел [Правила и действия](service-bus-queues-topics-subscriptions.md#rules-and-actions).

Чтобы выполнить развертывание автоматически, нажмите следующую кнопку.

[![Развертывание в Azure](./media/service-bus-resource-manager-namespace-topic/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-servicebus-create-topic-subscription-rule%2Fazuredeploy.json)

## <a name="parameters"></a>Параметры

С помощью Azure Resource Manager определите параметры для значений, которые должны указываться на этапе развертывания шаблона. В шаблоне есть раздел `Parameters` , содержащий все значения параметров. Определите параметры для тех значений, которые зависят от развертываемого проекта либо от среды, в которую выполняется развертывание. Не определяйте параметры для значений, которые не меняются. Значение каждого параметра в шаблоне определяет развертываемые ресурсы.

Ниже описаны параметры, которые определяет шаблон.

### <a name="servicebusnamespacename"></a>serviceBusNamespaceName

Имя создаваемого пространства имен служебной шины.

```json
"serviceBusNamespaceName": {
"type": "string"
}
```

### <a name="servicebustopicname"></a>serviceBusTopicName

Имя раздела, создаваемого в пространстве имен служебной шины.

```json
"serviceBusTopicName": {
"type": "string"
}
```

### <a name="servicebussubscriptionname"></a>serviceBusSubscriptionName

Имя подписки, создаваемой в пространстве имен служебной шины.

```json
"serviceBusSubscriptionName": {
"type": "string"
}
```

### <a name="servicebusrulename"></a>serviceBusRuleName

Имя правила (фильтра), создаваемого в пространстве имен служебной шины.

```json
   "serviceBusRuleName": {
   "type": "string",
  }
```

### <a name="servicebusapiversion"></a>serviceBusApiVersion

Версия API служебной шины для шаблона.

```json
"serviceBusApiVersion": { 
       "type": "string", 
       "defaultValue": "2017-04-01", 
       "metadata": { 
           "description": "Service Bus ApiVersion used by the template" 
       }
```

## <a name="resources-to-deploy"></a>Развертываемые ресурсы

Создает стандартное пространство имен служебной шины типа **Messaging** с разделом, подпиской и правилами.

```json
 "resources": [{
        "apiVersion": "[variables('sbVersion')]",
        "name": "[parameters('serviceBusNamespaceName')]",
        "type": "Microsoft.ServiceBus/Namespaces",
        "location": "[variables('location')]",
        "sku": {
            "name": "Standard",
        },
        "resources": [{
            "apiVersion": "[variables('sbVersion')]",
            "name": "[parameters('serviceBusTopicName')]",
            "type": "Topics",
            "dependsOn": [
                "[concat('Microsoft.ServiceBus/namespaces/', parameters('serviceBusNamespaceName'))]"
            ],
            "properties": {
                "path": "[parameters('serviceBusTopicName')]"
            },
            "resources": [{
                "apiVersion": "[variables('sbVersion')]",
                "name": "[parameters('serviceBusSubscriptionName')]",
                "type": "Subscriptions",
                "dependsOn": [
                    "[parameters('serviceBusTopicName')]"
                ],
                "properties": {},
                "resources": [{
                    "apiVersion": "[variables('sbVersion')]",
                    "name": "[parameters('serviceBusRuleName')]",
                    "type": "Rules",
                    "dependsOn": [
                        "[parameters('serviceBusSubscriptionName')]"
                    ],
                    "properties": {
                        "filterType": "SqlFilter",
                        "sqlFilter": {
                            "sqlExpression": "StoreName = 'Store1'",
                            "requiresPreprocessing": "false"
                        },
                        "action": {
                            "sqlExpression": "set FilterTag = 'true'"
                        }
                    }
                }]
            }]
        }]
    }]
```

Сведения о синтаксисе и свойствах JSON см. в статьях о [пространствах имен](/azure/templates/microsoft.servicebus/namespaces), [разделах](/azure/templates/microsoft.servicebus/namespaces/topics), [подписках](/azure/templates/microsoft.servicebus/namespaces/topics/subscriptions) и [правилах](/azure/templates/microsoft.servicebus/namespaces/topics/subscriptions/rules).

## <a name="commands-to-run-deployment"></a>Команды для выполнения развертывания

[!INCLUDE [app-service-deploy-commands](../../includes/app-service-deploy-commands.md)]

## <a name="powershell"></a>PowerShell

```powershell-interactive
New-AzureResourceGroupDeployment -Name \<deployment-name\> -ResourceGroupName \<resource-group-name\> -TemplateUri <https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-servicebus-create-topic-subscription-rule/azuredeploy.json>
```

## <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
azure config mode arm

azure group deployment create \<my-resource-group\> \<my-deployment-name\> --template-uri <https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-servicebus-create-topic-subscription-rule/azuredeploy.json>
```

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как управлять этими ресурсами, просмотрев следующие статьи:

* [Управление служебной шиной Azure](service-bus-management-libraries.md)
* [Управление служебной шиной с помощью PowerShell](service-bus-manage-with-ps.md)
* [Управление ресурсами служебной шины с помощью обозревателя служебной шины](https://github.com/paolosalvatori/ServiceBusExplorer/releases)

[Authoring Azure Resource Manager templates]: ../azure-resource-manager/templates/template-syntax.md
[Azure Quickstart Templates]: https://azure.microsoft.com/documentation/templates/?term=service+bus
[Learn more about Service Bus topics and subscriptions]: service-bus-queues-topics-subscriptions.md
[Using Azure PowerShell with Azure Resource Manager]: ../azure-resource-manager/management/manage-resources-powershell.md
[Using the Azure CLI for Mac, Linux, and Windows with Azure Resource Management]: ../azure-resource-manager/management/manage-resources-cli.md
[Recommended naming conventions for Azure resources]: /azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging
[Service Bus namespace with topic, subscription, and rule]: https://github.com/Azure/azure-quickstart-templates/blob/master/201-servicebus-create-topic-subscription-rule/
[Service Bus queues, topics, and subscriptions]: service-bus-queues-topics-subscriptions.md
