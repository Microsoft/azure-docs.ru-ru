---
title: Создание правила авторизации служебной шины с помощью шаблона Azure
description: Создание правила авторизации служебной шины для пространства имен и очереди с помощью шаблона диспетчера ресурсов Azure
author: spelluru
ms.topic: article
ms.tgt_pltfrm: dotnet
ms.date: 06/23/2020
ms.author: spelluru
ms.custom: devx-track-azurecli
ms.openlocfilehash: b0ffe022f6c61cf7d06c510c4f8fa5c764d72b42
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88067194"
---
# <a name="create-a-service-bus-authorization-rule-for-namespace-and-queue-using-an-azure-resource-manager-template"></a>Создание правила авторизации служебной шины для пространства имен и очереди с помощью шаблона диспетчера ресурсов Azure

В этой статье показывается, как использовать шаблон Azure Resource Manager, который создает [правило авторизации](service-bus-authentication-and-authorization.md#shared-access-signature) для пространства имен и очереди служебной шины. Здесь объясняется, как указать развертываемые ресурсы и определить параметры, указываемые при развертывании. Этот шаблон можно использовать для собственных развертываний или настроить его в соответствии с вашими требованиями.

Дополнительные сведения о создании шаблонов см. в статье [Создание шаблонов Azure Resource Manager][Authoring Azure Resource Manager templates].

Полный шаблон приведен в разделе [Service Bus authorization rule template][Service Bus auth rule template] (Шаблон правила аутентификации для служебной шины) на сайте GitHub.

> [!NOTE]
> Для скачивания и развертывания можно использовать указанные ниже шаблоны диспетчера ресурсов Azure.
> 
> * [Создайте пространство имен служебной шины](service-bus-resource-manager-namespace.md)
> * [Создание пространства имен служебной шины с очередью](service-bus-resource-manager-namespace-queue.md)
> * [Создание пространства имен служебной шины с разделом и подпиской](service-bus-resource-manager-namespace-topic.md)
> * [Создание пространства имен служебной шины с разделом, подпиской и правилом с помощью шаблона Azure Resource Manager](service-bus-resource-manager-namespace-topic-with-rule.md)
> 
> Чтобы узнать о последних шаблонах, перейдите в коллекцию шаблонов быстрого запуска [Azure][Azure Quickstart Templates] и выполните поиск по запросу **Service Bus**.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="what-will-you-deploy"></a>Что вы развернете?

С помощью этого шаблона вы развернете правило авторизации служебной шины для пространства имен и сущности обмена сообщениями (в нашем примере — очереди).

Этот шаблон использует [подписанный URL-адрес (SAS)](service-bus-sas.md) для проверки подлинности. Подписанный URL-адрес (SAS) позволяет проверять подлинность приложений в служебной шине с помощью ключа доступа, настроенного в пространстве имен или в сущности обмена сообщениями (очереди или разделе) с определенными правами. Этот ключ можно использовать для создания маркера SAS, который клиенты могут использовать при проверке подлинности в служебной шине.

Чтобы выполнить развертывание автоматически, нажмите следующую кнопку.

[![Развертывание в Azure](./media/service-bus-resource-manager-namespace-auth-rule/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F301-servicebus-create-authrule-namespace-and-queue%2Fazuredeploy.json)

## <a name="parameters"></a>Параметры

С помощью диспетчера ресурсов Azure можно определить параметры значений, которые должны указываться на этапе развертывания шаблона. В шаблоне есть раздел `Parameters` , содержащий все значения параметров. Для этих значений необходимо определить параметры, которые будут зависеть от развертываемого проекта либо от среды, в которой выполняется развертывание. Не задавайте параметры для значений, которые не меняются. Значение каждого параметра в шаблоне определяет развертываемые ресурсы.

Ниже описаны параметры, которые определяет шаблон.

### <a name="servicebusnamespacename"></a>serviceBusNamespaceName

Имя создаваемого пространства имен служебной шины.

```json
"serviceBusNamespaceName": {
"type": "string"
}
```

### <a name="namespaceauthorizationrulename"></a>namespaceAuthorizationRuleName

Имя правила авторизации для пространства имен.

```json
"namespaceAuthorizationRuleName ": {
"type": "string"
}
```

### <a name="servicebusqueuename"></a>serviceBusQueueName

Имя очереди в пространстве имен служебной шины.

```json
"serviceBusQueueName": {
"type": "string"
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

Создает стандартное пространство имен служебной шины типа **Messaging** и правило авторизации служебной шины для пространства имен и сущности.

```json
"resources": [
        {
            "apiVersion": "[variables('sbVersion')]",
            "name": "[parameters('serviceBusNamespaceName')]",
            "type": "Microsoft.ServiceBus/namespaces",
            "location": "[variables('location')]",
            "kind": "Messaging",
            "sku": {
                "name": "Standard",
            },
            "resources": [
                {
                    "apiVersion": "[variables('sbVersion')]",
                    "name": "[parameters('serviceBusQueueName')]",
                    "type": "Queues",
                    "dependsOn": [
                        "[concat('Microsoft.ServiceBus/namespaces/', parameters('serviceBusNamespaceName'))]"
                    ],
                    "properties": {
                        "path": "[parameters('serviceBusQueueName')]"
                    },
                    "resources": [
                        {
                            "apiVersion": "[variables('sbVersion')]",
                            "name": "[parameters('queueAuthorizationRuleName')]",
                            "type": "authorizationRules",
                            "dependsOn": [
                                "[parameters('serviceBusQueueName')]"
                            ],
                            "properties": {
                                "Rights": ["Listen"]
                            }
                        }
                    ]
                }
            ]
        }, {
            "apiVersion": "[variables('sbVersion')]",
            "name": "[variables('namespaceAuthRuleName')]",
            "type": "Microsoft.ServiceBus/namespaces/authorizationRules",
            "dependsOn": ["[concat('Microsoft.ServiceBus/namespaces/', parameters('serviceBusNamespaceName'))]"],
            "location": "[resourceGroup().location]",
            "properties": {
                "Rights": ["Send"]
            }
        }
    ]
```

Сведения о синтаксисе и свойствах JSON см. в разделах о [пространствах имен](/azure/templates/microsoft.servicebus/namespaces) и [очередях](/azure/templates/microsoft.servicebus/namespaces/queues) и [правилах авторизации](/azure/templates/microsoft.servicebus/namespaces/authorizationrules).

## <a name="commands-to-run-deployment"></a>Команды для выполнения развертывания

[!INCLUDE [app-service-deploy-commands](../../includes/app-service-deploy-commands.md)]

### <a name="powershell"></a>PowerShell

```powershell-interactive
New-AzResourceGroupDeployment -ResourceGroupName \<resource-group-name\> -TemplateFile <https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/301-servicebus-create-authrule-namespace-and-queue/azuredeploy.json>
```

## <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
azure config mode arm

azure group deployment create \<my-resource-group\> \<my-deployment-name\> --template-uri <https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/301-servicebus-create-authrule-namespace-and-queue/azuredeploy.json>
```

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы создали и развернули ресурсы с помощью диспетчера ресурсов Azure, узнайте, как управлять этими ресурсами, изучив следующие статьи:

* [Управление служебной шиной с помощью PowerShell](./service-bus-manage-with-ps.md)
* [Управление ресурсами служебной шины с помощью обозревателя служебной шины](https://github.com/paolosalvatori/ServiceBusExplorer/releases)
* [Аутентификация и авторизация в служебной шине](service-bus-authentication-and-authorization.md)

[Authoring Azure Resource Manager templates]: ../azure-resource-manager/templates/template-syntax.md
[Azure Quickstart Templates]: https://azure.microsoft.com/documentation/templates/?term=service+bus
[Using Azure PowerShell with Azure Resource Manager]: ../azure-resource-manager/management/manage-resources-powershell.md
[Using the Azure CLI for Mac, Linux, and Windows with Azure Resource Management]: ../azure-resource-manager/management/manage-resources-cli.md
[Service Bus auth rule template]: https://github.com/Azure/azure-quickstart-templates/blob/master/301-servicebus-create-authrule-namespace-and-queue/