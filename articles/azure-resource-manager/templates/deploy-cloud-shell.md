---
title: Развертывание шаблонов с помощью Cloud Shell
description: Используйте Azure Resource Manager и Azure Cloud Shell для развертывания ресурсов в Azure. Ресурсы определяются в шаблоне Azure Resource Manager (шаблон ARM).
ms.topic: conceptual
ms.date: 10/22/2020
ms.openlocfilehash: c67251a33b6197603be27086bcc6cd047e0c414b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98028613"
---
# <a name="deploy-arm-templates-from-azure-cloud-shell"></a>Развертывание шаблонов ARM из Azure Cloud Shell

Для развертывания шаблона Azure Resource Manager (шаблон ARM) можно использовать [Azure Cloud Shell](../../cloud-shell/overview.md) . Можно развернуть либо шаблон ARM, который хранится удаленно, либо шаблон ARM, хранящийся в локальной учетной записи хранения для Cloud Shell.

Можно выполнить развертывание в любой области. В этой статье описывается развертывание в группе ресурсов.

## <a name="deploy-remote-template"></a>Развертывание удаленного шаблона

Чтобы развернуть внешний шаблон, укажите URI шаблона точно так же, как и для любого внешнего развертывания. Внешний шаблон может находиться в репозитории GitHub или внешней учетной записи хранения.

1. Откройте запрос Cloud Shell.

   :::image type="content" source="./media/deploy-cloud-shell/open-cloud-shell.png" alt-text="Открыть Cloud Shell":::

1. Чтобы развернуть шаблон, используйте следующие команды:

   # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

   ```azurecli-interactive
   az group create --name ExampleGroup --location "Central US"
   az deployment group create \
     --name ExampleDeployment \
     --resource-group ExampleGroup \
     --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json" \
     --parameters storageAccountType=Standard_GRS
   ```

   # <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

   ```azurepowershell-interactive
   New-AzResourceGroup -Name ExampleGroup -Location "Central US"
   New-AzResourceGroupDeployment `
     -DeploymentName ExampleDeployment `
     -ResourceGroupName ExampleGroup `
     -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json `
     -storageAccountType Standard_GRS
   ```

   ---

## <a name="deploy-local-template"></a>Развертывание локального шаблона

Чтобы развернуть локальный шаблон, сначала необходимо отправить шаблон в учетную запись хранения, подключенную к сеансу Cloud Shell.

1. Войдите на [портал Azure](https://portal.azure.com).

1. Выберите свою группу ресурсов Cloud Shell. Шаблон имени — `cloud-shell-storage-<region>`.

   ![Выбор группы ресурсов](./media/deploy-cloud-shell/select-cloud-shell-resource-group.png)

1. Выберите учетную запись хранения для Cloud Shell.

   :::image type="content" source="./media/deploy-cloud-shell/cloud-shell-storage.png" alt-text="Выбор учетной записи хранения":::

1. Выберите **файловые ресурсы**.

   :::image type="content" source="./media/deploy-cloud-shell/files-shares.png" alt-text="Выбор общих файловых ресурсов":::

1. Выберите общую папку по умолчанию для Cloud Shell. Файловый ресурс имеет формат имени `cs-<user>-<domain>-com-<uniqueGuid>` .

   :::image type="content" source="./media/deploy-cloud-shell/select-file-share.png" alt-text="Общая папка по умолчанию":::

1. Добавьте новый каталог для хранения шаблонов. Выберите этот каталог.

   :::image type="content" source="./media/deploy-cloud-shell/add-directory.png" alt-text="Добавление каталога":::

1. Щелкните **Отправить**.

   :::image type="content" source="./media/deploy-cloud-shell/upload-template.png" alt-text="Отправить шаблон":::

1. Найдите и отправьте свой шаблон.

   :::image type="content" source="./media/deploy-cloud-shell/select-template.png" alt-text="Выбор шаблона":::

1. Откройте запрос Cloud Shell.

   :::image type="content" source="./media/deploy-cloud-shell/open-cloud-shell.png" alt-text="Открыть Cloud Shell":::

1. Перейдите в каталог **clouddrive** . Перейдите к каталогу, который вы добавили для хранения шаблонов.

1. Чтобы развернуть шаблон, используйте следующие команды:

   # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

   ```azurecli-interactive
   az group create --name ExampleGroup --location "South Central US"
   az deployment group create \
     --resource-group ExampleGroup \
     --template-file azuredeploy.json \
     --parameters storageAccountType=Standard_GRS
   ```

   # <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

   ```azurepowershell-interactive
   New-AzResourceGroup -Name ExampleGroup -Location "Central US"
   New-AzResourceGroupDeployment `
     -DeploymentName ExampleDeployment `
     -ResourceGroupName ExampleGroup `
     -TemplateFile azuredeploy.json `
     -storageAccountType Standard_GRS
   ```

   ---

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о командах развертывания см. в статьях [развертывание ресурсов с помощью шаблонов ARM и Azure CLI](deploy-cli.md) и [развертывание ресурсов с помощью шаблонов ARM и Azure PowerShell](deploy-powershell.md).
- Сведения о предварительном просмотре изменений перед развертыванием шаблона см. в разделе [Операция развертывания шаблонов ARM](template-deploy-what-if.md).
