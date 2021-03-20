---
title: Создание пространства имен служебной шины Azure с помощью шаблона
description: Используйте шаблон Azure Resource Manager для создания пространства имен для обмена сообщениями служебной шины
documentationcenter: .net
author: spelluru
ms.topic: article
ms.tgt_pltfrm: dotnet
ms.date: 06/23/2020
ms.author: spelluru
ms.openlocfilehash: 1b7aafca331170100ce99c084a11c96c97df7781
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88067398"
---
# <a name="create-a-service-bus-namespace-by-using-an-azure-resource-manager-template"></a>Создание пространства имен служебной шины с помощью шаблона Azure Resource Manager

Узнайте, как развернуть шаблон Azure Resource Manager для создания пространства имен служебной шины. Этот шаблон можно использовать для собственных развертываний или настроить его в соответствии с вашими требованиями. Дополнительные сведения о создании шаблонов см. в [документации по Azure Resource Manager](../azure-resource-manager/index.yml).

Для создания пространств имен служебной шины также доступны следующие шаблоны:

* [Создание пространства имен служебной шины с очередью](./service-bus-resource-manager-namespace-queue.md)
* [Создание пространства имен служебной шины с разделом и подпиской](./service-bus-resource-manager-namespace-topic.md)
* [Создание пространства имен служебной шины с очередью и правилом авторизации](./service-bus-resource-manager-namespace-auth-rule.md)
* [Создание пространства имен служебной шины с разделом, подпиской и правилом с помощью шаблона Azure Resource Manager](./service-bus-resource-manager-namespace-topic-with-rule.md)

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="create-a-service-bus-namespace"></a>Создание пространства имен служебной шины

В этом кратком руководстве вы используете [существующий шаблон диспетчер ресурсов](https://github.com/Azure/azure-quickstart-templates/blob/master/101-servicebus-create-namespace/azuredeploy.json) из [шаблонов](https://azure.microsoft.com/resources/templates/)быстрого запуска Azure:

[!code-json[create-azure-service-bus-namespace](~/quickstart-templates/101-servicebus-create-namespace/azuredeploy.json)]

См. [примеры шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Servicebus&pageNumber=1&sort=Popular).

Создание пространства имен служебной шины путем развертывания шаблона.

1. Выберите вариант **попробовать** в следующем блоке кода, а затем следуйте инструкциям по входу в Azure Cloud Shell.

    ```azurepowershell-interactive
    $serviceBusNamespaceName = Read-Host -Prompt "Enter a name for the service bus namespace to be created"
    $location = Read-Host -Prompt "Enter the location (i.e. centralus)"
    $resourceGroupName = "${serviceBusNamespaceName}rg"
    $templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-servicebus-create-namespace/azuredeploy.json"

    New-AzResourceGroup -Name $resourceGroupName -Location $location
    New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri -serviceBusNamespaceName $serviceBusNamespaceName

    Write-Host "Press [ENTER] to continue ..."
    ```

    Имя группы ресурсов — это имя пространства имен служебной шины с добавленным **RG** .

2. Нажмите кнопку **Копировать**, чтобы скопировать сценарий PowerShell.
3. Щелкните правой кнопкой в консоли оболочки и выберите **Вставить**.

Создание концентратора событий занимает несколько секунд.

## <a name="verify-the-deployment"></a>Проверка развертывания

Чтобы просмотреть развернутое пространство имен служебной шины, можно либо открыть группу ресурсов из портал Azure, либо использовать следующий Azure PowerShell скрипта. Если Cloud Shell все еще открыт, вам не нужно копировать и запускать первую и вторую строки следующего скрипта.

```azurepowershell-interactive
$serviceBusNamespaceName = Read-Host -Prompt "Enter the same service bus namespace name used earlier"
$resourceGroupName = "${serviceBusNamespaceName}rg"

Get-AzServiceBusNamespace -ResourceGroupName $resourceGroupName -Name $serviceBusNamespaceName

Write-Host "Press [ENTER] to continue ..."
```

Azure PowerShell используется для развертывания шаблона в этом руководстве. Другие методы развертывания шаблонов см. в следующих статьях:

* С [помощью портал Azure](../azure-resource-manager/templates/deploy-portal.md).
* С [помощью Azure CLI](../azure-resource-manager/templates/deploy-cli.md).
* С [помощью REST API](../azure-resource-manager/templates/deploy-rest.md).

## <a name="clean-up-resources"></a>Очистка ресурсов

Если ресурсы Azure больше не нужны, их можно удалить. Для этого необходимо удалить группу ресурсов. Если Cloud Shell все еще открыт, вам не нужно копировать и запускать первую и вторую строки следующего скрипта.

```azurepowershell-interactive
$serviceBusNamespaceName = Read-Host -Prompt "Enter the same service bus namespace name used earlier"
$resourceGroupName = "${serviceBusNamespaceName}rg"

Remove-AzResourceGroup -ResourceGroupName $resourceGroupName

Write-Host "Press [ENTER] to continue ..."
```

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы создали пространство имен служебной шины. Ознакомьтесь с другими краткими руководствами, чтобы узнать, как создавать очереди, разделы и подписки и использовать их.

* [Начало работы с очередями служебной шины](service-bus-dotnet-get-started-with-queues.md)
* [Начало работы с разделами служебной шины](service-bus-dotnet-how-to-use-topics-subscriptions.md)