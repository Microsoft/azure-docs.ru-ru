---
title: Краткое руководство. Развертывание Azure API для FHIR c помощью шаблона ARM
description: Из этого краткого руководства вы узнаете, как развернуть API Azure для стандарта FHIR® (Ресурсы быстрого взаимодействия в сфере здравоохранения) с помощью шаблона Azure Resource Manager.
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: quickstart
ms.custom: subject-armqs, devx-track-azurecli
ms.author: cavoeg
ms.date: 10/06/2020
ms.openlocfilehash: 8d7f71477e50e4771bbc7670312943eadf1a8657
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103019707"
---
# <a name="quickstart-use-an-arm-template-to-deploy-azure-api-for-fhir"></a>Краткое руководство. Использование шаблона Resource Manager для развертывания Azure API для FHIR

Из этого краткого руководства вы узнаете, как воспользоваться шаблоном Azure Resource Manager (шаблоном Resource Manager), чтобы развернуть API Azure для стандарта FHIR® (Ресурсы быстрого взаимодействия в сфере здравоохранения). Вы можете развернуть Azure API для FHIR с помощью портала Azure, PowerShell или CLI.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. После входа шаблон откроется на портале Azure.

[:::image type="content" source="../../media/template-deployments/deploy-to-azure.svg" alt-text="Развертывание службы Azure API для FHIR в Azure с помощью шаблона Resource Manager на портале Azure.":::](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-azure-api-for-fhir%2fazuredeploy.json)

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

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-azure-api-for-fhir/).

:::code language="json" source="~/quickstart-templates/101-azure-api-for-fhir/azuredeploy.json":::

Шаблон определяет один ресурс Azure:

* **Microsoft.HealthcareApis/services**

<!--

Replace the line above with the following line once https://docs.microsoft.com/azure/templates/microsoft.healthcareapis/services goes live:

* [**Microsoft.HealthcareApis/services**](/azure/templates/microsoft.healthcareapis/services)

-->

## <a name="deploy-the-template"></a>Развертывание шаблона

# <a name="portal"></a>[Портал](#tab/azure-portal)

Щелкните следующую ссылку, чтобы развернуть Azure API для FHIR с помощью шаблона Resource Manager на портале Azure:

[:::image type="content" source="../../media/template-deployments/deploy-to-azure.svg" alt-text="Развертывание службы Azure API для FHIR в Azure с помощью шаблона Resource Manager на портале Azure.":::](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-azure-api-for-fhir%2fazuredeploy.json)

На странице **Развертывание Azure API для FHIR** выполните следующие действия:

1. При необходимости замените **подписку** по умолчанию на другую подписку.

2. В разделе **Группа ресурсов** щелкните **Создать новую**, введите имя новой группы ресурсов и нажмите **ОК**.

3. Если вы создаете новую группу ресурсов, выберите для нее **Регион**.

4. Введите новое **имя службы** и выберите **расположение** Azure API для FHIR. Оно может отличаться от региона группы ресурсов или совпадать с ним.

    :::image type="content" source="./media/fhir-paas-arm-template-quickstart/deploy-azure-api-fhir.png" alt-text="Развертывание Azure API для FHIR с помощью шаблона Resource Manager на портале Azure.":::

5. Выберите **Review + create** (Просмотреть и создать).

6. Ознакомьтесь с условиями, а затем выберите **Создать**.

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

> [!NOTE]
> Если вы хотите запускать скрипты PowerShell локально, сначала введите `Connect-AzAccount`, чтобы настроить учетные данные Azure.

Если поставщик ресурсов `Microsoft.HealthcareApis` еще не зарегистрирован для вашей подписки, его можно зарегистрировать с помощью следующего интерактивного кода. Чтобы выполнить этот код в Azure Cloud Shell, щелкните **Попробовать** в правом верхнем углу блока кода.

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.HealthcareApis
```

Используйте следующий код для развертывания службы Azure API для FHIR в Azure с помощью шаблона Resource Manager. Код предложит вам ввести новое имя службы FHIR, имя новой группы ресурсов и их расположение.

```azurepowershell-interactive
$serviceName = Read-Host -Prompt "Enter a name for the new Azure API for FHIR service"
$serviceLocation = Read-Host -Prompt "Enter an Azure region (for example, westus2) for the service"
$resourceGroupName = Read-Host -Prompt "Enter a name for the new resource group to contain the service"
$resourceGroupRegion = Read-Host -Prompt "Enter an Azure region (for example, centralus) for the resource group"

Write-Verbose "New-AzResourceGroup -Name $resourceGroupName -Location $resourceGroupRegion" -Verbose
New-AzResourceGroup -Name $resourceGroupName -Location $resourceGroupRegion
Write-Verbose "Run New-AzResourceGroupDeployment to create an Azure API for FHIR service using an ARM template" -Verbose
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName `
    -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-azure-api-for-fhir/azuredeploy.json `
    -serviceName $serviceName `
    -location $serviceLocation
Read-Host "Press [ENTER] to continue"
```

# <a name="cli"></a>[CLI](#tab/CLI)

Если поставщик ресурсов `Microsoft.HealthcareApis` еще не зарегистрирован для вашей подписки, его можно зарегистрировать с помощью следующего интерактивного кода. Чтобы выполнить этот код в Azure Cloud Shell, щелкните **Попробовать** в правом верхнем углу блока кода.

```azurecli-interactive
az extension add --name healthcareapis
```

Используйте следующий код для развертывания службы Azure API для FHIR в Azure с помощью шаблона Resource Manager. Код предложит вам ввести новое имя службы FHIR, имя новой группы ресурсов и их расположение.

```azurecli-interactive
read -p "Enter a name for the new Azure API for FHIR service: " serviceName &&
read -p "Enter an Azure region (for example, westus2) for the service: " serviceLocation &&
read -p "Enter a name for the new resource group to contain the service: " resourceGroupName &&
read -p "Enter an Azure region (for example, centralus) for the resource group: " resourceGroupRegion &&
params='serviceName='$serviceName' location='$serviceLocation &&
echo "CREATE RESOURCE GROUP:  az group create --name $resourceGroupName --location $resourceGroupRegion" &&
az group create --name $resourceGroupName --location $resourceGroupRegion &&
echo "RUN az deployment group create, which creates an Azure API for FHIR service using an ARM template" &&
az deployment group create --resource-group $resourceGroupName --parameters $params --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-azure-api-for-fhir/azuredeploy.json &&
read -p "Press [ENTER] to continue: "
```

---

> [!NOTE]
> Развертывание занимает несколько минут. Запишите имена службы Azure API для FHIR и группы ресурсов. Они будут использоваться для просмотра развернутых ресурсов в дальнейшем.

## <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

# <a name="portal"></a>[Портал](#tab/azure-portal)

Чтобы просмотреть обзор новой службы Azure API для FHIR, выполните следующие действия.

1. На [портале Azure](https://portal.azure.com) найдите и выберите **Azure API для FHIR**.

2. В списке FHIR выберите свою новую службу. Откроется страница **Обзор** для новой службы Azure API для FHIR.

3. Убедитесь, что новая учетная запись API FHIR подготовлена. Для этого щелкните ссылку рядом с **конечной точкой метаданных FHIR**, чтобы получить отчет о возможностях API FHIR. Ссылка имеет формат `https://<service-name>.azurehealthcareapis.com/metadata`. Если учетная запись подготовлена, отображается большой JSON-файл.

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

Выполните следующий интерактивный код, чтобы просмотреть сведения о службе Azure API для FHIR. Вам нужно будет ввести имя новой службы и группы ресурсов.

```azurepowershell-interactive
$serviceName = Read-Host -Prompt "Enter the name of your Azure API for FHIR service"
$resourceGroupName = Read-Host -Prompt "Enter the resource group name"
Write-Verbose "Get-AzHealthcareApisService -ResourceGroupName $resourceGroupName -Name $serviceName" -Verbose
Get-AzHealthcareApisService -ResourceGroupName $resourceGroupName -Name $serviceName
Read-Host "Press [ENTER] to fetch the FHIR API capability statement, which shows that the new service has been provisioned"

$requestUri="https://" + $serviceName + ".azurehealthcareapis.com/metadata"
$metadata = Invoke-WebRequest -Uri $requestUri
$metadata.RawContent
Read-Host "Press [ENTER] to continue"
```

# <a name="cli"></a>[CLI](#tab/CLI)

Выполните следующий интерактивный код, чтобы просмотреть сведения о службе Azure API для FHIR. Вам нужно будет ввести имя новой службы и группы ресурсов.

```azurecli-interactive
read -p "Enter the name of your Azure API for FHIR service: " serviceName &&
read -p "Enter the resource group name: " resourceGroupName &&
echo "SHOW SERVICE DETAILS:  az healthcareapis service show --resource-group $resourceGroupName --resource-name $serviceName" &&
az healthcareapis service show --resource-group $resourceGroupName --resource-name $serviceName &&
read -p "Press [ENTER] to fetch the FHIR API capability statement, which shows that the new service has been provisioned: " &&
requestUrl='https://'$serviceName'.azurehealthcareapis.com/metadata' &&
curl --url $requestUrl &&
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

Пошаговые инструкции по созданию шаблона Resource Manager см. в [этом учебнике](../../azure-resource-manager/templates/template-tutorial-create-first-template.md).

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве показано, как развернуть Azure API для FHIR в своей подписке. Чтобы задать дополнительные параметры в Azure API для FHIR, см. соответствующее руководство. Если вы уже готовы к использованию API Azure для FHIR, ознакомьтесь с процессом регистрации приложений.

>[!div class="nextstepaction"]
>[Дополнительные параметры в Azure API для FHIR](azure-api-for-fhir-additional-settings.md)

>[!div class="nextstepaction"]
>[Общие сведения о регистрации приложений](fhir-app-registration.md)
