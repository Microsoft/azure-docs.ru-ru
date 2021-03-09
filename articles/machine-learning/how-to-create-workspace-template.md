---
title: Создание рабочей области с помощью шаблона Azure Resource Manager
titleSuffix: Azure Machine Learning
description: Сведения о том, как применить шаблон Azure Resource Manager для создания рабочей области Машинного обучения Azure.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to, devx-track-azurecli, devx-track-azurepowershell
ms.author: larryfr
author: Blackmist
ms.date: 09/30/2020
ms.openlocfilehash: d64dfdad0c51950228288b704b3fdb7579496abb
ms.sourcegitcommit: 8d1b97c3777684bd98f2cfbc9d440b1299a02e8f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102488075"
---
# <a name="use-an-azure-resource-manager-template-to-create-a-workspace-for-azure-machine-learning"></a>Создание рабочей области для Машинного обучения Azure с помощью шаблона Azure Resource Manager


<br>

В этой статье мы рассмотрим несколько способов создания рабочей области Машинного обучения Azure с помощью шаблонов Azure Resource Manager. Шаблон Resource Manager позволяет легко создать все ресурсы в отдельной скоординированной операции. Шаблон представляет собой документ JSON, в котором определены необходимые для развертывания ресурсы. Также здесь можно указать параметры развертывания. Эти параметры позволят предоставить входные значения при использовании шаблона.

Узнайте подробнее [о развертывании приложения с помощью шаблона диспетчера ресурсов Azure](../azure-resource-manager/templates/deploy-powershell.md).

## <a name="prerequisites"></a>Предварительные требования

* **Подписка Azure**. Если у вас ее нет, используйте [бесплатную или платную версию Машинного обучения Azure](https://aka.ms/AMLFree).

* Чтобы использовать шаблон из командной строки, вам потребуется [Azure PowerShell](/powershell/azure/?view=azps-1.2.0) или [Azure CLI](/cli/azure/install-azure-cli).

* Для некоторых сценариев требуется открыть запрос в службу поддержки. Они приведены ниже.

    * __Рабочая область с поддержкой Приватного канала с ключом, управляемым клиентом__
    * __Реестр контейнеров Azure для рабочей области, находящейся за виртуальной сетью__

    Дополнительные сведения см. в статье [Управление квотами и их увеличение](how-to-manage-quotas.md#private-endpoint-and-private-dns-quota-increases).

## <a name="limitations"></a>Ограничения

[!INCLUDE [register-namespace](../../includes/machine-learning-register-namespace.md)]

## <a name="workspace-resource-manager-template"></a>Шаблон диспетчер ресурсов рабочей области

Шаблон Azure Resource Manager, используемый в этом документе, можно найти в каталоге [201-Machine-Learning-Advanced](https://github.com/Azure/azure-quickstart-templates/blob/master/201-machine-learning-advanced/azuredeploy.json) статьи шаблоны быстрого запуска Azure в репозитории GitHub.

Этот шаблон создает следующие ресурсы Azure:

* Учетная запись хранения Azure
* Azure Key Vault
* Azure Application Insights
* Реестр контейнеров Azure
* Рабочая область службы "Машинное обучение Azure"

Группа ресурсов выполняет функции контейнера, который содержит службы. Для рабочего пространства Службы машинного обучения Azure нужно несколько служб.

Пример шаблона имеет два **обязательных** параметра:

* **Расположение** , в котором будут создаваться ресурсы.

    Этот шаблон будет использовать выбранное расположение для большинства ресурсов. Исключением является служба Application Insights, которая доступна не во всех тех расположениях, в которых доступны другие службы. Если вы выберете расположение, где эта служба не доступна, ее экземпляр будет создан в регионе "центрально-южная часть США".

* **WorkspaceName**— понятное имя рабочей области машинное обучение Azure.

    > [!NOTE]
    > В имени рабочей области не учитывается регистр.

    Имена для всех остальных служб создаются случайным образом.

> [!TIP]
> Хотя шаблон, связанный с этим документом, создает новый Реестр контейнеров Azure, вы можете создать рабочую область и без реестра контейнеров. Он будет создан автоматически при выполнении любой операции, для которой реестр контейнеров обязателен. Например, при обучении или развертывании модели.
>
> Вы также можете указать в шаблоне Azure Resource Manager существующий реестр контейнеров или учетную запись хранения, а не создавать новые. При этом необходимо либо [использовать управляемое удостоверение](how-to-use-managed-identities.md) (Предварительная версия), либо [включить учетную запись администратора](../container-registry/container-registry-authentication.md#admin-account) для реестра контейнеров.

[!INCLUDE [machine-learning-delete-acr](../../includes/machine-learning-delete-acr.md)]

Дополнительные сведения о шаблонах вы найдете в следующих статьях.

* [Шаблоны диспетчера ресурсов Azure](../azure-resource-manager/templates/template-syntax.md)
* [Развертывание ресурсов с использованием шаблонов Resource Manager и Azure CLI](../azure-resource-manager/templates/deploy-powershell.md)
* [Типы ресурсов Microsoft.MachineLearningServices](/azure/templates/microsoft.machinelearningservices/allversions)

## <a name="deploy-template"></a>Развертывание шаблона

Чтобы развернуть шаблон, необходимо создать группу ресурсов.

Если вы предпочитаете использовать графический пользовательский интерфейс, ознакомьтесь с разделом [портал Azure](#use-the-azure-portal) .

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

```azurecli
az group create --name "examplegroup" --location "eastus"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

```azurepowershell
New-AzResourceGroup -Name "examplegroup" -Location "eastus"
```

---

После успешного создания группы ресурсов разверните шаблон с помощью следующей команды:

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

```azurecli
az deployment group create \
    --name "exampledeployment" \
    --resource-group "examplegroup" \
    --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" \
    --parameters workspaceName="exampleworkspace" location="eastus"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name "exampledeployment" `
  -ResourceGroupName "examplegroup" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" `
  -workspaceName "exampleworkspace" `
  -location "eastus"
```

---

По умолчанию все ресурсы, созданные как часть шаблона, являются новыми. Однако у вас также есть возможность использовать существующие ресурсы. Предоставляя дополнительные параметры для шаблона, можно использовать существующие ресурсы. Например, если вы хотите использовать существующую учетную запись хранения, задайте для параметра **сторажеаккаунтоптион** значение **existing** и укажите имя учетной записи хранения в параметре **storageAccountName** .

> [!IMPORTANT]
> Если вы хотите использовать существующую учетную запись хранения Azure, она не может быть учетной записью Premium (Premium_LRS и Premium_GRS). Он также не может иметь иерархическое пространство имен (используется с Azure Data Lake Storage 2-го поколения). Ни хранилище класса Premium, ни иерархическое пространство имен не поддерживаются в учетной записи хранения по умолчанию рабочей области. Ни хранилище класса Premium, ни иерархическое пространство имен не поддерживаются в учетной записи хранения _по умолчанию_ рабочей области. Вы можете использовать хранилище класса Premium или иерархическое пространство имен с учетными записями хранения, _отличными от стандартных_ .

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

```azurecli
az deployment group create \
    --name "exampledeployment" \
    --resource-group "examplegroup" \
    --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" \
    --parameters workspaceName="exampleworkspace" \
      location="eastus" \
      storageAccountOption="existing" \
      storageAccountName="existingstorageaccountname"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name "exampledeployment" `
  -ResourceGroupName "examplegroup" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" `
  -workspaceName "exampleworkspace" `
  -location "eastus" `
  -storageAccountOption "existing" `
  -storageAccountName "existingstorageaccountname"
```

---

## <a name="deploy-an-encrypted-workspace"></a>Развертывание зашифрованной рабочей области

Создание рабочей области с тремя параметрами показано в следующем примере шаблона:

* Включите параметры высокой конфиденциальности для рабочей области. При этом создается новый экземпляр Cosmos DB.
* Включите шифрование для рабочей области.
* Использует существующий Azure Key Vault для получения ключей, управляемых клиентом. Ключи, управляемые клиентом, используются для создания нового экземпляра Cosmos DB для рабочей области.

    [!INCLUDE [machine-learning-customer-managed-keys.md](../../includes/machine-learning-customer-managed-keys.md)]

> [!IMPORTANT]
> После создания рабочей области вы не сможете изменить параметры для конфиденциальных данных, шифрования, идентификатора хранилища ключей или идентификаторов ключей. Чтобы изменить эти значения, придется создать новую рабочую область с новыми значениями.

Дополнительные сведения см. в статье [Шифрование данных при хранении](concept-data-encryption.md#encryption-at-rest).

> [!IMPORTANT]
> Чтобы использовать этот шаблон, ваша подписка должна соответствовать некоторым дополнительным требованиям.
> * Должен существовать хранилище Azure Key Vault с ключом шифрования.
> * Хранилище Azure Key Vault должно размещаться в том же регионе, где вы намерены создать рабочую область Машинного обучения Azure.
> * Необходимо указать идентификатор Azure Key Vault и универсальный код ресурса (URI) ключа шифрования.

__Чтобы получить значения__ для параметров `cmk_keyvault` (идентификатор Key Vault) и `resource_cmk_uri` (URI ключа), которые нужны в этом шаблоне, выполните следующие действия.    

1. Чтобы получить идентификатор Key Vault, используйте следующую команду:  

    # <a name="azure-cli"></a>[Azure CLI](#tab/azcli)   

    ```azurecli 
    az keyvault show --name <keyvault-name> --query 'id' --output tsv   
    ``` 

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell) 

    ```azurepowershell  
    Get-AzureRMKeyVault -VaultName '<keyvault-name>'    
    ``` 
    --- 

    Эта команда возвращает значение следующего вида: `/subscriptions/{subscription-guid}/resourceGroups/<resource-group-name>/providers/Microsoft.KeyVault/vaults/<keyvault-name>`.  

1. Чтобы получить значение URI для управляемого клиентом ключа, используйте следующую команду:    

    # <a name="azure-cli"></a>[Azure CLI](#tab/azcli)   

    ```azurecli 
    az keyvault key show --vault-name <keyvault-name> --name <key-name> --query 'key.kid' --output tsv  
    ``` 

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell) 

    ```azurepowershell  
    Get-AzureKeyVaultKey -VaultName '<keyvault-name>' -KeyName '<key-name>' 
    ``` 
    --- 

    Эта команда возвращает значение следующего вида: `https://mykeyvault.vault.azure.net/keys/mykey/{guid}`. 

> [!IMPORTANT]  
> После создания рабочей области вы не сможете изменить параметры для конфиденциальных данных, шифрования, идентификатора хранилища ключей или идентификаторов ключей. Чтобы изменить эти значения, придется создать новую рабочую область с новыми значениями.

Чтобы включить использование управляемых клиентом ключей, задайте следующие параметры при развертывании шаблона.

* **Encryption_status** **включить**.
* **cmk_keyvault** значение, `cmk_keyvault` полученное на предыдущих шагах.
* **resource_cmk_uri** значение, `resource_cmk_uri` полученное на предыдущих шагах.

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

```azurecli
az deployment group create \
    --name "exampledeployment" \
    --resource-group "examplegroup" \
    --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" \
    --parameters workspaceName="exampleworkspace" \
      location="eastus" \
      encryption_status="Enabled" \
      cmk_keyvault="/subscriptions/{subscription-guid}/resourceGroups/<resource-group-name>/providers/Microsoft.KeyVault/vaults/<keyvault-name>" \
      resource_cmk_uri="https://mykeyvault.vault.azure.net/keys/mykey/{guid}" \
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name "exampledeployment" `
  -ResourceGroupName "examplegroup" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" `
  -workspaceName "exampleworkspace" `
  -location "eastus" `
  -encryption_status "Enabled" `
  -cmk_keyvault "/subscriptions/{subscription-guid}/resourceGroups/<resource-group-name>/providers/Microsoft.KeyVault/vaults/<keyvault-name>" `
  -resource_cmk_uri "https://mykeyvault.vault.azure.net/keys/mykey/{guid}"
```
---

При использовании ключа, управляемого клиентом, Машинное обучение Azure создает вторичную группу ресурсов, которая содержит экземпляр Cosmos DB. Дополнительные сведения см. [в разделе Шифрование неактивных данных Cosmos DB](concept-data-encryption.md#encryption-at-rest).

Дополнительной конфигурацией, которую можно предоставить для данных, является установка параметра **confidential_data** в **значение true**. Это делает следующее:

* Запускает шифрование локального временного диска для Машинное обучение Azureных кластерных ресурсов, предоставляя не созданные ранее кластеры в подписке. Если вы ранее создали кластер в подписке, отправьте запрос в службу поддержки, чтобы обеспечить шифрование временного диска для ваших кластеров.
* Очищает локальный рабочий диск между запусками.
* Безопасно передает учетные данные для учетной записи хранения, реестра контейнеров и учетной записи SSH из уровня выполнения в вычислительные кластеры с помощью хранилища ключей.
* Включает фильтрацию IP-адресов для того, чтобы базовые пулы пакетов не можно было вызывать из внешних служб, отличных от Азуремачинелеарнингсервице.

    > [!IMPORTANT]
    > После создания рабочей области вы не сможете изменить параметры для конфиденциальных данных, шифрования, идентификатора хранилища ключей или идентификаторов ключей. Чтобы изменить эти значения, придется создать новую рабочую область с новыми значениями.

  Дополнительные сведения см. [в разделе Шифрование неактивных](concept-data-encryption.md#encryption-at-rest)данных.

## <a name="deploy-workspace-behind-a-virtual-network"></a>Развертывание рабочей области за виртуальной сетью

`vnetOption`Если задать для параметра значение `new` или `existing` , вы сможете создать ресурсы, используемые рабочей областью, которая находится за виртуальной сетью.

> [!IMPORTANT]
> Для реестра контейнеров поддерживается только SKU "Премиум".

> [!IMPORTANT]
> Application Insights не поддерживает развертывание за пределами виртуальной сети.

### <a name="only-deploy-workspace-behind-private-endpoint"></a>Развертывать только рабочую область за частной конечной точкой

Если связанные ресурсы не находятся за виртуальной сетью, можно задать для параметра **приватиндпоинттипе** значение `AutoAproval` или, `ManualApproval` чтобы развернуть рабочую область за частной конечной точкой. Это можно сделать как для новых, так и для существующих рабочих областей. При обновлении существующей рабочей области заполните параметры шаблона данными из существующей рабочей области.

> [!IMPORTANT]
> Использование рабочей области Машинное обучение Azure с частной ссылкой недоступна в регионах Azure для государственных организаций.

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

```azurecli
az deployment group create \
    --name "exampledeployment" \
    --resource-group "examplegroup" \
    --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" \
    --parameters workspaceName="exampleworkspace" \
      location="eastus" \
      privateEndpointType="AutoApproval"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name "exampledeployment" `
  -ResourceGroupName "examplegroup" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" `
  -workspaceName "exampleworkspace" `
  -location "eastus" `
  -privateEndpointType "AutoApproval"
```

---

### <a name="use-a-new-virtual-network"></a>Использовать новую виртуальную сеть

Чтобы развернуть ресурс за новой виртуальной сетью, задайте для параметра **внетоптион** значение **New** вместе с параметрами виртуальной сети для соответствующего ресурса. В развертывании ниже показано, как развернуть рабочую область с ресурсом учетной записи хранения за новой виртуальной сетью.

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

```azurecli
az deployment group create \
    --name "exampledeployment" \
    --resource-group "examplegroup" \
    --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" \
    --parameters workspaceName="exampleworkspace" \
      location="eastus" \
      vnetOption="new" \
      vnetName="examplevnet" \
      storageAccountBehindVNet="true"
      privateEndpointType="AutoApproval"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name "exampledeployment" `
  -ResourceGroupName "examplegroup" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" `
  -workspaceName "exampleworkspace" `
  -location "eastus" `
  -vnetOption "new" `
  -vnetName "examplevnet" `
  -storageAccountBehindVNet "true"
  -privateEndpointType "AutoApproval"
```

---

Кроме того, можно развернуть несколько или все зависимые ресурсы за пределами виртуальной сети.

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

```azurecli
az deployment group create \
    --name "exampledeployment" \
    --resource-group "examplegroup" \
    --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" \
    --parameters workspaceName="exampleworkspace" \
      location="eastus" \
      vnetOption="new" \
      vnetName="examplevnet" \
      storageAccountBehindVNet="true" \
      keyVaultBehindVNet="true" \
      containerRegistryBehindVNet="true" \
      containerRegistryOption="new" \
      containerRegistrySku="Premium"
      privateEndpointType="AutoApproval"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name "exampledeployment" `
  -ResourceGroupName "examplegroup" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" `
  -workspaceName "exampleworkspace" `
  -location "eastus" `
  -vnetOption "new" `
  -vnetName "examplevnet" `
  -storageAccountBehindVNet "true"
  -keyVaultBehindVNet "true" `
  -containerRegistryBehindVNet "true" `
  -containerRegistryOption "new" `
  -containerRegistrySku "Premium"
  -privateEndpointType "AutoApproval"
```

---

<!-- Workspaces need a private endpoint when associated resources are behind a virtual network to work properly. To set up a private endpoint for the workspace with a new virtual network:

> [!IMPORTANT]
> The deployment is only valid in regions which support private endpoints.

# [Azure CLI](#tab/azcli)

```azurecli
az deployment group create \
    --name "exampledeployment" \
    --resource-group "examplegroup" \
    --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" \
    --parameters workspaceName="exampleworkspace" \
      location="eastus" \
      vnetOption="new" \
      vnetName="examplevnet" \
      privateEndpointType="AutoApproval"
```

# [Azure PowerShell](#tab/azpowershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name "exampledeployment" `
  -ResourceGroupName "examplegroup" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" `
  -workspaceName "exampleworkspace" `
  -location "eastus" `
  -vnetOption "new" `
  -vnetName "examplevnet" `
  -privateEndpointType "AutoApproval"
```

--- -->

### <a name="use-an-existing-virtual-network--resources"></a>Использование существующих ресурсов виртуальной сети &

Чтобы развернуть рабочую область с существующими связанными ресурсами, необходимо задать для параметра **внетоптион** значение **existing** вместе с параметрами подсети. Однако **перед** развертыванием необходимо создать конечные точки службы в виртуальной сети для каждого из ресурсов. Как и в случае с новыми развертываниями виртуальной сети, у вас может быть один или несколько ресурсов, расположенных за виртуальной сетью.

> [!IMPORTANT]
> Подсеть должна иметь `Microsoft.Storage` конечную точку службы

> [!IMPORTANT]
> Подсети не допускают создание частных конечных точек. Отключите закрытую конечную точку, чтобы включить подсеть.

1. Включите конечные точки службы для ресурсов.

    # <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

    ```azurecli
    az network vnet subnet update --resource-group "examplegroup" --vnet-name "examplevnet" --name "examplesubnet" --service-endpoints "Microsoft.Storage"
    az network vnet subnet update --resource-group "examplegroup" --vnet-name "examplevnet" --name "examplesubnet" --service-endpoints "Microsoft.KeyVault"
    az network vnet subnet update --resource-group "examplegroup" --vnet-name "examplevnet" --name "examplesubnet" --service-endpoints "Microsoft.ContainerRegistry"
    ```

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

    ```azurepowershell
    Get-AzVirtualNetwork -ResourceGroupName "examplegroup" -Name "examplevnet" | Set-AzVirtualNetworkSubnetConfig -Name "examplesubnet" -AddressPrefix "<subnet prefix>" -ServiceEndpoint "Microsoft.Storage" | Set-AzVirtualNetwork
    Get-AzVirtualNetwork -ResourceGroupName "examplegroup" -Name "examplevnet" | Set-AzVirtualNetworkSubnetConfig -Name "examplesubnet" -AddressPrefix "<subnet prefix>" -ServiceEndpoint "Microsoft.KeyVault" | Set-AzVirtualNetwork
    Get-AzVirtualNetwork -ResourceGroupName "examplegroup" -Name "examplevnet" | Set-AzVirtualNetworkSubnetConfig -Name "examplesubnet" -AddressPrefix "<subnet prefix>" -ServiceEndpoint "Microsoft.ContainerRegistry" | Set-AzVirtualNetwork
    ```

    ---

1. Развертывание рабочей области

    # <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

    ```azurecli
    az deployment group create \
    --name "exampledeployment" \
    --resource-group "examplegroup" \
    --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" \
    --parameters workspaceName="exampleworkspace" \
      location="eastus" \
      vnetOption="existing" \
      vnetName="examplevnet" \
      vnetResourceGroupName="examplegroup" \
      storageAccountBehindVNet="true" \
      keyVaultBehindVNet="true" \
      containerRegistryBehindVNet="true" \
      containerRegistryOption="new" \
      containerRegistrySku="Premium" \
      subnetName="examplesubnet" \
      subnetOption="existing"
      privateEndpointType="AutoApproval"
    ```

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)
    ```azurepowershell
    New-AzResourceGroupDeployment `
      -Name "exampledeployment" `
      -ResourceGroupName "examplegroup" `
      -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" `
      -workspaceName "exampleworkspace" `
      -location "eastus" `
      -vnetOption "existing" `
      -vnetName "examplevnet" `
      -vnetResourceGroupName "examplegroup" `
      -storageAccountBehindVNet "true"
      -keyVaultBehindVNet "true" `
      -containerRegistryBehindVNet "true" `
      -containerRegistryOption "new" `
      -containerRegistrySku "Premium" `
      -subnetName "examplesubnet" `
      -subnetOption "existing"
      -privateEndpointType "AutoApproval"
    ```
    ---

<!-- Workspaces need a private endpoint when associated resources are behind a virtual network to work properly. To set up a private endpoint for the workspace with an existing virtual network:

> [!IMPORTANT]
> The deployment is only valid in regions which support private endpoints.

# [Azure CLI](#tab/azcli)

```azurecli
az deployment group create \
    --name "exampledeployment" \
    --resource-group "examplegroup" \
    --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" \
    --parameters workspaceName="exampleworkspace" \
      location="eastus" \
      vnetOption="existing" \
      vnetName="examplevnet" \
      vnetResourceGroupName="rg" \
      privateEndpointType="AutoApproval" \
      subnetName="subnet" \
      subnetOption="existing"
```

# [Azure PowerShell](#tab/azpowershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name "exampledeployment" `
  -ResourceGroupName "examplegroup" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" `
  -workspaceName "exampleworkspace" `
  -location "eastus" `
  -vnetOption "existing" `
  -vnetName "examplevnet" `
  -vnetResourceGroupName "rg"
  -privateEndpointType "AutoApproval"
  -subnetName "subnet"
  -subnetOption "existing"
```

--- -->

## <a name="use-the-azure-portal"></a>Использование портала Azure

1. Выполните инструкции из раздела [Развертывание ресурсов с помощью настраиваемого шаблона](../azure-resource-manager/templates/deploy-portal.md#deploy-resources-from-custom-template). Когда вы поступите на экран __Выбор шаблона__ , выберите шаблон **201-Machine-Learning-Advanced** из раскрывающегося списка.
1. Щелкните __выбрать шаблон__ , чтобы использовать шаблон. Укажите следующие необходимые сведения и другие параметры в зависимости от сценария развертывания.

   * Подписка: Выберите подписку Azure, которую нужно использовать для этих ресурсов.
   * Группа ресурсов: Выберите или создайте группу ресурсов, которая будет содержать службы.
   * Регион: Выберите регион Azure, в котором будут создаваться ресурсы.
   * Имя рабочей области: Это имя, которое будет присвоено созданному рабочему пространству Службы машинного обучения Azure. Для имени рабочего пространства допускается длина от 3 до 33 символов. Имя может содержать только буквы, цифры и дефис ("-").
   * Расположение. Выберите расположение, в котором будут созданы ресурсы.
1. Выберите __Review + create__ (Просмотреть и создать).
1. На экране " __Проверка и создание__ " примите указанные выше условия и нажмите кнопку __создать__.

Дополнительные сведения см. в разделе [Развертывание ресурсов с помощью настраиваемого шаблона](../azure-resource-manager/templates/deploy-portal.md#deploy-resources-from-custom-template).

## <a name="troubleshooting"></a>Устранение неполадок

### <a name="resource-provider-errors"></a>Ошибки поставщика ресурсов

[!INCLUDE [machine-learning-resource-provider](../../includes/machine-learning-resource-provider.md)]

### <a name="azure-key-vault-access-policy-and-azure-resource-manager-templates"></a>Политики доступа Azure Key Vault и шаблоны Azure Resource Manager

Шаблон Azure Resource Manager можно многократно применять для создания рабочей области и связанных с ней ресурсов (в том числе Azure Key Vault). Например, шаблон можно многократно применять с одинаковыми параметрами в составе конвейера непрерывной интеграции и развертывания.

Большинство операций создания ресурсов с использованием шаблонов являются идемпотентными, но Key Vault удаляет политики доступа каждый раз при использовании шаблона. Очистка политик доступа нарушает доступ к Key Vault из любой существующей рабочей области, которая его использует. Например, могут не выполняться функции остановки и создания для виртуальных машин Записных книжек Azure.  

Чтобы избежать этой проблемы, примените один из следующих подходов.

* Не развертывайте шаблон более одного раза с одинаковыми параметрами. Удаляйте существующие ресурсы перед тем, как создавать их повторно с помощью шаблона.

* Изучите политики доступа к Key Vault, а затем примените их для настройки свойства `accessPolicies` в шаблоне. Для просмотра политик доступа можно выполнить следующую команду Azure CLI.

    ```azurecli
    az keyvault show --name mykeyvault --resource-group myresourcegroup --query properties.accessPolicies
    ```

    Дополнительные сведения об использовании раздела `accessPolicies` шаблона см. в разделе [Объект AccessPolicyEntry](/azure/templates/Microsoft.KeyVault/2018-02-14/vaults#AccessPolicyEntry) справочника.

* Проверьте, существует ли ресурс Key Vault. Если существует, его не следует создавать повторно с помощью шаблона. Например, внесите следующие изменения в шаблон, чтобы использовать существующий ресурс Key Vault вместо создания нового:

    * **Добавьте** параметр, который принимает значение идентификатора существующего ресурса Key Vault:

        ```json
        "keyVaultId":{
          "type": "string",
          "metadata": {
            "description": "Specify the existing Key Vault ID."
          }
        }
      ```

    * **Удалите** раздел, который создает ресурс Key Vault:

        ```json
        {
          "type": "Microsoft.KeyVault/vaults",
          "apiVersion": "2018-02-14",
          "name": "[variables('keyVaultName')]",
          "location": "[parameters('location')]",
          "properties": {
            "tenantId": "[variables('tenantId')]",
            "sku": {
              "name": "standard",
              "family": "A"
            },
            "accessPolicies": [
            ]
          }
        },
        ```

    * **Удалите** строку `"[resourceId('Microsoft.KeyVault/vaults', variables('keyVaultName'))]",` из раздела `dependsOn` для рабочей области. Кроме того, **измените** запись `keyVault` в разделе `properties` для рабочей области, включив в нее ссылку на параметр `keyVaultId`:

        ```json
        {
          "type": "Microsoft.MachineLearningServices/workspaces",
          "apiVersion": "2019-11-01",
          "name": "[parameters('workspaceName')]",
          "location": "[parameters('location')]",
          "dependsOn": [
            "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]",
            "[resourceId('Microsoft.Insights/components', variables('applicationInsightsName'))]"
          ],
          "identity": {
            "type": "systemAssigned"
          },
          "sku": {
            "tier": "[parameters('sku')]",
            "name": "[parameters('sku')]"
          },
          "properties": {
            "friendlyName": "[parameters('workspaceName')]",
            "keyVault": "[parameters('keyVaultId')]",
            "applicationInsights": "[resourceId('Microsoft.Insights/components',variables('applicationInsightsName'))]",
            "storageAccount": "[resourceId('Microsoft.Storage/storageAccounts/',variables('storageAccountName'))]"
          }
        }
        ```

    После этих изменений вы сможете указать идентификатор существующего ресурса Key Vault при запуске шаблона. Тогда этот шаблон применит указанное хранилище ключей, присвоив его идентификатор свойству `keyVault` рабочей области.

    Чтобы получить идентификатор Key Vault, можно указать ссылку на выходные данные от выполнения исходного шаблона или применить Azure CLI. Следующая команда является примером использования Azure CLI для получения идентификатора ресурса Key Vault.

    ```azurecli
    az keyvault show --name mykeyvault --resource-group myresourcegroup --query id
    ```

    Эта команда возвращает значение следующего вида:

    ```text
    /subscriptions/{subscription-guid}/resourceGroups/myresourcegroup/providers/Microsoft.KeyVault/vaults/mykeyvault
    ```

### <a name="virtual-network-not-linked-to-private-dns-zone"></a>Виртуальная сеть не связана с частной зоной DNS

При создании рабочей области с закрытой конечной точкой шаблон создает Частная зона DNS зону с именем __privatelink.API.azureml.MS__. В эту частную зону DNS автоматически добавляется __ссылка на виртуальную сеть__ . Эта ссылка добавляется только для первой рабочей области и частной конечной точки, созданной в группе ресурсов. Если вы создаете другую виртуальную сеть и рабочую область с частной конечной точкой в той же группе ресурсов, вторая виртуальная сеть может не добавляться в частную зону DNS.

Чтобы просмотреть связи виртуальной сети, которые уже существуют для частной зоны DNS, используйте следующую команду Azure CLI.

```azurecli
az network private-dns link vnet list --zone-name privatelink.api.azureml.ms --resource-group myresourcegroup
```

Чтобы добавить виртуальную сеть, содержащую другую рабочую область и закрытую конечную точку, выполните следующие действия.

1. Чтобы найти идентификатор виртуальной сети для добавляемой сети, используйте следующую команду:

    ```azurecli
    az network vnet show --name myvnet --resource-group myresourcegroup --query id
    ```
    
    Эта команда возвращает значение, похожее на ""/Субскриптионс/гуид/ресаурцеграупс/миресаурцеграуп/провидерс/Микрософт.Нетворк/виртуалнетворкс/мивнет "". Сохраните это значение и используйте его на следующем шаге.

2. Чтобы добавить ссылку на виртуальную сеть в зону Частная зона DNS privatelink.api.azureml.ms, используйте следующую команду. Для `--virtual-network` параметра используйте выходные данные предыдущей команды:

    ```azurecli
    az network private-dns link vnet create --name mylinkname --registration-enabled true --resource-group myresourcegroup --virtual-network myvirtualnetworkid --zone-name privatelink.api.azureml.ms
    ```

## <a name="next-steps"></a>Дальнейшие действия

* [Развертывание ресурсов с использованием шаблонов и REST API Resource Manager](../azure-resource-manager/templates/deploy-rest.md).
* [Создание и развертывание групп ресурсов Azure с помощью Visual Studio](../azure-resource-manager/templates/create-visual-studio-deployment-project.md).
* [Другие шаблоны, связанные с Машинное обучение Azure, см. в репозитории шаблонов быстрого запуска Azure.](https://github.com/Azure/azure-quickstart-templates)