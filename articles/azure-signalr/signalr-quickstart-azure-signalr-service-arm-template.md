---
title: Краткое руководство. Создание службы Azure SignalR — шаблон Resource Manager
description: Из этого краткого руководства вы узнаете, как с помощью шаблона Azure Resource Manager создать службу Azure SignalR.
author: sffamily
ms.author: zhshang
ms.date: 10/02/2020
ms.topic: quickstart
ms.service: signalr
ms.custom:
- subject-armqs
- devx-track-azurecli
- mode-arm
ms.openlocfilehash: 075915f2a06e29eb5b80425913c6e7a2c476ff99
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107536517"
---
# <a name="quickstart-use-an-arm-template-to-deploy-azure-signalr-service"></a>Краткое руководство. Использование шаблона Resource Manager для развертывания службы Azure SignalR

В этом кратком руководстве объясняется, как создать службу Azure SignalR с помощью шаблона Azure Resource Manager. Вы можете развернуть службу Azure SignalR с помощью портала Azure, PowerShell или CLI.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. После входа шаблон откроется на портале Azure.

[:::image type="content" source="../media/template-deployments/deploy-to-azure.svg" alt-text="Кнопка для развертывания службы Azure SignalR в Azure с помощью шаблона Resource Manager на портале Azure.":::](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-signalr%2fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

# <a name="portal"></a>[Портал](#tab/azure-portal)

Учетная запись Azure с активной подпиской. [Создайте бесплатно](https://azure.microsoft.com/free/).

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

* Учетная запись Azure с активной подпиской. [Создайте бесплатно](https://azure.microsoft.com/free/).
* Если вы хотите выполнить код локально, используйте [Azure PowerShell](/powershell/azure/install-az-ps).

# <a name="cli"></a>[CLI](#tab/CLI)

* Учетная запись Azure с активной подпиской. [Создайте бесплатно](https://azure.microsoft.com/free/).
* Если вы хотите выполнить код локально, сделайте следующее.
    * Оболочка Bash (например, Git Bash, которая включена в [Git для Windows](https://gitforwindows.org)).
    * [Azure CLI](/cli/azure/install-azure-cli).

---

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-signalr/).

:::code language="json" source="~/quickstart-templates/101-signalr/azuredeploy.json":::

Шаблон определяет один ресурс Azure:

* [**Microsoft.SignalRService/SignalR**](/azure/templates/microsoft.signalrservice/signalr)

## <a name="deploy-the-template"></a>Развертывание шаблона

# <a name="portal"></a>[Портал](#tab/azure-portal)

Щелкните следующую ссылку, чтобы развернуть службу Azure SignalR с помощью шаблона Resource Manager на портале Azure:

[:::image type="content" source="../media/template-deployments/deploy-to-azure.svg" alt-text="Кнопка для развертывания службы Azure SignalR в Azure с помощью шаблона Resource Manager на портале Azure.":::](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-signalr%2fazuredeploy.json)

На странице **Развертывание службы Azure SignalR**:

1. При необходимости замените **подписку** по умолчанию.

2. В разделе **Группа ресурсов** щелкните **Создать новую**, введите имя новой группы ресурсов и нажмите **ОК**.

3. Если вы создаете новую группу ресурсов, выберите для нее **Регион**.

4. Вы также можете ввести новое **имя** и **расположение** (например, **eastus2**) службы Azure SignalR. Если имя не указано, оно создается автоматически. Расположение для службы Azure SignalR может отличаться от региона группы ресурсов или совпадать с ним. Если расположение не указано, для него будет задан тот же регион, что и для группы ресурсов.

5. Выберите **ценовую категорию** (**Free_F1** или **Standard_S1**), укажите **емкость** (число единиц SignalR) и выберите для параметра **Режим службы** значение **По умолчанию** (требуется сервер-концентратор), **Бессерверный** (не разрешающий подключение к серверу) или **Классический** (маршрутизация на сервер-концентратор, только если концентратор имеет подключение к серверу). Затем укажите, следует ли **включить журналы подключения** или **включить журналы обмена сообщениями**.

    > [!NOTE]
    > Для ценовой категории **Free_F1** емкость ограничена 1 единицей.

    :::image type="content" source="./media/signalr-quickstart-azure-signalr-service-arm-template/deploy-azure-signalr-service-arm-template-portal.png" alt-text="Снимок экрана: шаблон Resource Manager для создания службы Azure SignalR на портале Azure.":::

6. Выберите **Review + create** (Просмотреть и создать).

7. Ознакомьтесь с условиями, а затем выберите **Создать**.

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

> [!NOTE]
> Если вы хотите запускать скрипты PowerShell локально, сначала введите `Connect-AzAccount`, чтобы настроить учетные данные Azure.

Используйте следующий код, чтобы развернуть службу Azure SignalR с помощью шаблона Resource Manager. В коде запрашиваются следующие элементы:

* Имя и регион новой службы Azure SignalR.
* Имя и регион новой группы ресурсов.
* Ценовая категория Azure (**Free_F1** или **Standard_S1**).
* Единица измерения емкости SignalR (1, 2, 5, 10, 20, 50 или 100).
  > [!NOTE]
  > Для ценовой категории **Free_F1** емкость ограничена 1 единицей.
* Режим службы. **По умолчанию** — чтобы использовать сервер-концентратор, **Бессерверный** — чтобы запретить любое подключение к серверу, или **Классический** — для маршрутизации на сервер-концентратор, только при наличии подключения концентратора к серверу
* Следует ли включать журналы для подключения или обмена сообщениями (**true** или **false**)

```azurepowershell-interactive
$serviceName = Read-Host -Prompt "Enter a name for the new Azure SignalR Service"
$serviceLocation = Read-Host -Prompt "Enter an Azure region (for example, westus2) for the service"
$resourceGroupName = Read-Host -Prompt "Enter a name for the new resource group to contain the service"
$resourceGroupRegion = Read-Host -Prompt "Enter an Azure region (for example, centralus) for the resource group"

$priceTier = Read-Host -Prompt "Enter the pricing tier (Free_F1 or Standard_S1)"
$unitCapacity = Read-Host -Prompt "Enter the number of SignalR units (1, 2, 5, 10, 20, 50, or 100)"
$servicingMode = Read-Host -Prompt "Enter the service mode (Default, Serverless, or Classic)"
$enableConnectionLogs = Read-Host -Prompt "Specify whether to enable connectivity logs (true or false)"
$enableMessageLogs = Read-Host -Prompt "Specify whether to enable messaging logs (true or false)"

Write-Verbose "New-AzResourceGroup -Name $resourceGroupName -Location $resourceGroupRegion" -Verbose
New-AzResourceGroup -Name $resourceGroupName -Location $resourceGroupRegion

$paramObjHashTable = @{
    name = $serviceName
    location = $serviceLocation
    pricingTier = $priceTier
    capacity = [int]$unitCapacity
    serviceMode = $servicingMode
    enableConnectivityLogs = $enableConnectionLogs
    enableMessagingLogs = $enableMessageLogs
}

Write-Verbose "Run New-AzResourceGroupDeployment to create an Azure SignalR Service using an ARM template" -Verbose
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName `
    -TemplateParameterObject $paramObjHashTable `
    -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-signalr/azuredeploy.json
Read-Host "Press [ENTER] to continue"
```

# <a name="cli"></a>[CLI](#tab/CLI)

Используйте следующий код, чтобы развернуть службу Azure SignalR с помощью шаблона Resource Manager. В коде запрашиваются следующие элементы:

* Имя и регион новой службы Azure SignalR.
* Имя и регион новой группы ресурсов.
* Ценовая категория Azure (**Free_F1** или **Standard_S1**).
* Единица измерения емкости SignalR (1, 2, 5, 10, 20, 50 или 100).
    > [!NOTE]
    > Для ценовой категории **Free_F1** емкость ограничена 1 единицей.
* Режим службы. **По умолчанию** — чтобы использовать сервер-концентратор, **Бессерверный** — чтобы запретить любое подключение к серверу, или **Классический** — для маршрутизации на сервер-концентратор, только при наличии подключения концентратора к серверу
* Следует ли включать журналы для подключения или обмена сообщениями (**true** или **false**)

```azurecli-interactive
read -p "Enter a name for the new Azure SignalR Service: " serviceName &&
read -p "Enter an Azure region (for example, westus2) for the service: " serviceLocation &&
read -p "Enter a name for the new resource group to contain the service: " resourceGroupName &&
read -p "Enter an Azure region (for example, centralus) for the resource group: " resourceGroupRegion &&
read -p "Enter the pricing tier (Free_F1 or Standard_S1): " priceTier &&
read -p "Enter the number of SignalR units (1, 2, 5, 10, 20, 50, or 100): " unitCapacity &&
read -p "Enter the service mode (Default, Serverless, or Classic): " servicingMode &&
read -p "Specify whether to enable connectivity logs (true or false): " enableConnectionLogs &&
read -p "Specify whether to enable messaging logs (true or false): " enableMessageLogs &&
params='name='$serviceName' location='$serviceLocation' pricingTier='$priceTier' capacity='$unitCapacity' serviceMode='$servicingMode' enableConnectivityLogs='$enableConnectionLogs' enableMessagingLogs='$enableMessageLogs &&
echo "CREATE RESOURCE GROUP:  az group create --name $resourceGroupName --location $resourceGroupRegion" &&
az group create --name $resourceGroupName --location $resourceGroupRegion &&
echo "RUN az deployment group create, which creates an Azure SignalR Service using an ARM template" &&
az deployment group create --resource-group $resourceGroupName --parameters $params --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-signalr/azuredeploy.json &&
read -p "Press [ENTER] to continue: "
```

---

> [!NOTE]
> Завершение развертывания может занять несколько минут. Запишите имена службы Azure SignalR и группы ресурсов. Вы будете использовать их для просмотра развернутых ресурсов в дальнейшем.

## <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

# <a name="portal"></a>[Портал](#tab/azure-portal)

Чтобы просмотреть общие сведения о новой службе Azure SignalR, выполните следующие действия:

1. На [портале Azure](https://portal.azure.com) найдите и выберите службу **SignalR**.

2. Выберите свою новую службу в списке SignalR. Откроется страница **Обзор** для новой службы Azure SignalR.

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

Выполните следующий интерактивный код, чтобы просмотреть сведения о службе Azure SignalR. Вам нужно будет ввести имя новой службы и группы ресурсов.

```azurepowershell-interactive
$serviceName = Read-Host -Prompt "Enter the name of your Azure SignalR Service"
$resourceGroupName = Read-Host -Prompt "Enter the resource group name"
Write-Verbose "Get-AzSignalR -ResourceGroupName $resourceGroupName -Name $serviceName" -Verbose
Get-AzSignalR -ResourceGroupName $resourceGroupName -Name $serviceName
Read-Host "Press [ENTER] to continue"
```

# <a name="cli"></a>[CLI](#tab/CLI)

Выполните следующий интерактивный код, чтобы просмотреть сведения о службе Azure SignalR. Вам нужно будет ввести имя новой службы и группы ресурсов.

```azurecli-interactive
read -p "Enter the name of your Azure SignalR Service: " serviceName &&
read -p "Enter the resource group name: " resourceGroupName &&
echo "SHOW SERVICE DETAILS:  az signalr show --resource-group $resourceGroupName --name $serviceName" &&
az signalr show --resource-group $resourceGroupName --name $serviceName &&
read -p "Press [ENTER] to continue: "
```

---

## <a name="clean-up-resources"></a>Очистка ресурсов

Удалите ненужную группу ресурсов, когда надобность в ней отпадет. Ресурсы в ней также будут удалены.

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. На [портале Azure](https://portal.azure.com) найдите и выберите элемент **Группы ресурсов**.

2. В списке групп ресурсов выберите имя нужной группы ресурсов.

3. На странице **Обзор** для этой группы ресурсов выберите **Удалить группу ресурсов**.

4. В диалоговом окне подтверждения введите имя группы ресурсов и щелкните **Удалить**.

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the name of the resource group to delete"
Write-Verbose "Remove-AzResourceGroup -Name $resourceGroupName" -Verbose
Remove-AzResourceGroup -Name $resourceGroupName
Read-Host "Press [ENTER] to continue"
```

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli-interactive
read -p "Enter the name of the resource group to delete: " resourceGroupName &&
echo "DELETE A RESOURCE GROUP (AND ITS RESOURCES):  az group delete --name $resourceGroupName" &&
az group delete --name $resourceGroupName &&
read -p "Press [ENTER] to continue: "
```

---

## <a name="next-steps"></a>Дальнейшие действия

Пошаговые инструкции по созданию шаблона ARM см. в следующей статье:

> [!div class="nextstepaction"]
> [Руководство. Создание и развертывание первого шаблона ARM](../azure-resource-manager/templates/template-tutorial-create-first-template.md)
