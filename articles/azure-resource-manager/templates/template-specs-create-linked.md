---
title: Создание спецификации шаблона со связанными шаблонами
description: Узнайте, как создать спецификацию шаблона со связанными шаблонами.
ms.topic: conceptual
ms.date: 01/05/2021
ms.openlocfilehash: 31764b9adb1d0fc28a170fa2e6b5073812533083
ms.sourcegitcommit: a67b972d655a5a2d5e909faa2ea0911912f6a828
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104889847"
---
# <a name="tutorial-create-a-template-spec-with-linked-templates-preview"></a>Учебник. Создание спецификации шаблона со связанными шаблонами (Предварительная версия)

Узнайте, как создать [спецификацию шаблона](template-specs.md) с основным шаблоном и [связанным шаблоном](linked-templates.md#linked-template). Спецификации шаблонов можно использовать для совместного использования шаблонов ARM с другими пользователями в Организации. В этой статье показано, как создать спецификацию шаблона для упаковки основного шаблона и связанных с ним шаблонов с помощью `relativePath` свойства [ресурса развертывания](/azure/templates/microsoft.resources/deployments).

## <a name="prerequisites"></a>Предварительные требования

Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.

> [!NOTE]
> Сейчас спецификации шаблонов доступны в предварительной версии. Чтобы использовать их с Azure PowerShell, необходимо установить [версию 5.0.0 или более новую](/powershell/azure/install-az-ps). Чтобы использовать их с Azure CLI, воспользуйтесь [версией 2.14.2 или более новой](/cli/azure/install-azure-cli).

## <a name="create-linked-templates"></a>Создание связанных шаблонов

Создайте основной шаблон и связанный шаблон.

Чтобы связать шаблон, добавьте [ресурс развертывания](/azure/templates/microsoft.resources/deployments) в основной шаблон. В `templateLink` свойстве укажите относительный путь к связанному шаблону в соответствии с путем к родительскому шаблону.

Связанный шаблон называется **linkedTemplate.jsв** и хранится во вложенной папке с именем **артефакты** в пути, где хранится основной шаблон.  Для параметра relativePath можно использовать одно из следующих значений:

- `./artifacts/linkedTemplate.json`
- `/artifacts/linkedTemplate.json`
- `artifacts/linkedTemplate.json`

`relativePath`Свойство всегда имеет относительный путь к файлу шаблона `relativePath` , где объявлено, поэтому если имеется другой linkedTemplate2.js, вызванный из linkedTemplate.jsв и linkedTemplate2.jsв, хранится в той же вложенной папке артефактов, relativePath, указанный в linkedTemplate.js, — это просто `linkedTemplate2.json` .

1. Создайте основной шаблон со следующим JSON. Сохраните основной шаблон как **azuredeploy.jsна** локальном компьютере. В этом учебнике предполагается, что вы сохранили путь в **c:\Templates\linkedTS\azuredeploy.js** , но можете использовать любой путь.

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "location": {
          "type": "string",
          "defaultValue": "westus2",
          "metadata":{
            "description": "Specify the location for the resources."
          }
        },
        "storageAccountType": {
          "type": "string",
          "defaultValue": "Standard_LRS",
          "metadata":{
            "description": "Specify the storage account type."
          }
        }
      },
      "variables": {
        "appServicePlanName": "[concat('plan', uniquestring(resourceGroup().id))]"
      },
      "resources": [
        {
          "type": "Microsoft.Web/serverfarms",
          "apiVersion": "2016-09-01",
          "name": "[variables('appServicePlanName')]",
          "location": "[parameters('location')]",
          "sku": {
            "name": "B1",
            "tier": "Basic",
            "size": "B1",
            "family": "B",
            "capacity": 1
          },
          "kind": "linux",
          "properties": {
            "perSiteScaling": false,
            "reserved": true,
            "targetWorkerCount": 0,
            "targetWorkerSizeId": 0
          }
        },
        {
          "type": "Microsoft.Resources/deployments",
          "apiVersion": "2020-10-01",
          "name": "createStorage",
          "properties": {
            "mode": "Incremental",
            "templateLink": {
              "relativePath": "artifacts/linkedTemplate.json"
            },
            "parameters": {
              "storageAccountType": {
                "value": "[parameters('storageAccountType')]"
              }
            }
          }
        }
      ]
    }
    ```

    > [!NOTE]
    > ApiVersion `Microsoft.Resources/deployments` должен быть 2020-06-01 или более поздней версии.

1. Создайте каталог с именем **артефакты** в папке, в которой сохранен основной шаблон.
1. Создайте связанный шаблон со следующим JSON:

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "storageAccountType": {
          "type": "string",
          "defaultValue": "Standard_LRS",
          "allowedValues": [
            "Standard_LRS",
            "Standard_GRS",
            "Standard_ZRS",
            "Premium_LRS"
          ],
          "metadata": {
            "description": "Storage Account type"
          }
        },
        "location": {
          "type": "string",
          "defaultValue": "[resourceGroup().location]",
          "metadata": {
            "description": "Location for all resources."
          }
        }
      },
      "variables": {
        "storageAccountName": "[concat('store', uniquestring(resourceGroup().id))]"
      },
      "resources": [
        {
          "type": "Microsoft.Storage/storageAccounts",
          "apiVersion": "2019-04-01",
          "name": "[variables('storageAccountName')]",
          "location": "[parameters('location')]",
          "sku": {
            "name": "[parameters('storageAccountType')]"
          },
          "kind": "StorageV2",
          "properties": {}
        }
      ],
      "outputs": {
        "storageAccountName": {
          "type": "string",
          "value": "[variables('storageAccountName')]"
        }
      }
    }
    ```

1. Сохраните шаблон как **linkedTemplate.js** в папке **артефакты** .

## <a name="create-template-spec"></a>Создание спецификации шаблона

Спецификации шаблонов хранятся в группах ресурсов.  Создайте группу ресурсов, а затем создайте шаблон спецификации со следующим скриптом. Название спецификации шаблона — «имя- **Спецификация**».

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroup `
  -Name templateSpecRG `
  -Location westus2

New-AzTemplateSpec `
  -Name webSpec `
  -Version "1.0.0.0" `
  -ResourceGroupName templateSpecRG `
  -Location westus2 `
  -TemplateFile "c:\Templates\linkedTS\azuredeploy.json"
```

# <a name="cli"></a>[CLI](#tab/azure-cli)

```azurecli
az group create \
  --name templateSpecRG \
  --location westus2

az ts create \
  --name webSpec \
  --version "1.0.0.0" \
  --resource-group templateSpecRG \
  --location "westus2" \
  --template-file "<path-to-main-template>"
```

---

По завершении можно просмотреть спецификацию шаблона из портал Azure или с помощью следующего командлета:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Get-AzTemplateSpec -ResourceGroupName templatespecRG -Name webSpec
```

# <a name="cli"></a>[CLI](#tab/azure-cli)

```azurecli
az ts show --name webSpec --resource-group templateSpecRG --version "1.0.0.0"
```

---

## <a name="deploy-template-spec"></a>Развертывание спецификации шаблона

Теперь можно развернуть спецификацию шаблона. Развертывание спецификации шаблона аналогично развертыванию шаблона, содержащегося в ней, за исключением того, что вы передаете ИД ресурса спецификации шаблона. Вы используете те же команды развертывания и при необходимости передаете значения параметров для спецификации шаблона.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroup `
  -Name webRG `
  -Location westus2

$id = (Get-AzTemplateSpec -ResourceGroupName templateSpecRG -Name webSpec -Version "1.0.0.0").Versions.Id

New-AzResourceGroupDeployment `
  -TemplateSpecId $id `
  -ResourceGroupName webRG
```

# <a name="cli"></a>[CLI](#tab/azure-cli)

```azurecli
az group create \
  --name webRG \
  --location westus2

id = $(az ts show --name webSpec --resource-group templateSpecRG --version "1.0.0.0" --query "id")

az deployment group create \
  --resource-group webRG \
  --template-spec $id
```

> [!NOTE]
> Существует известная проблема при получении идентификатора спецификации шаблона и ее присвоении переменной в Windows PowerShell.

---

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о развертывании шаблона спецификации в качестве связанного шаблона см. в разделе [учебник. Развертывание шаблона спецификации в качестве связанного шаблона](template-specs-deploy-linked-template.md).
