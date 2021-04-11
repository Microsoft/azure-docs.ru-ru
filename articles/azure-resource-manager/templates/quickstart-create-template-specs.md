---
title: Создание и развертывание спецификации шаблона
description: Узнайте, как создать спецификацию шаблона на основе шаблона ARM. Затем разверните спецификацию шаблона в группе ресурсов в подписке.
author: tfitzmac
ms.date: 12/14/2020
ms.topic: quickstart
ms.author: tomfitz
ms.openlocfilehash: 28987486726f5a88d20efe9fe8a766e536062c2c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104889966"
---
# <a name="quickstart-create-and-deploy-template-spec-preview"></a>Краткое руководство. Создание и развертывание спецификации шаблона (предварительная версия)

В этом кратком руководстве показано, как упаковать шаблон Azure Resource Manager (шаблон ARM) в [спецификацию шаблона](template-specs.md). После упаковки вы выполните развертывание этой спецификации шаблона. Спецификация шаблона содержит шаблон ARM, который развертывает учетную запись хранения.

## <a name="prerequisites"></a>Предварительные требования

Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.

> [!NOTE]
> Сейчас спецификации шаблонов доступны в предварительной версии. Чтобы использовать их с Azure PowerShell, необходимо установить [версию 5.0.0 или более новую](/powershell/azure/install-az-ps). Чтобы использовать их с Azure CLI, воспользуйтесь [версией 2.14.2 или более новой](/cli/azure/install-azure-cli).

## <a name="create-template"></a>Создание шаблона

Спецификация шаблона создается из локального шаблона. Скопируйте следующий шаблон и сохраните его локально в файл с именем **azuredeploy.json**. В этом кратком руководстве предполагается, что вы сохранили путь **c:\Templates\azuredeploy.json**, однако можете использовать любой другой путь.

:::code language="json" source="~/quickstart-templates/101-storage-account-create/azuredeploy.json":::

## <a name="create-template-spec"></a>Создание спецификации шаблона

Спецификация шаблона — это тип ресурсов с именем `Microsoft.Resources/templateSpecs`. Чтобы создать спецификацию шаблона, используйте PowerShell, Azure CLI или шаблон ARM.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Создайте новую группу ресурсов, которая будет содержать спецификацию шаблона.

    ```azurepowershell
    New-AzResourceGroup `
      -Name templateSpecRG `
      -Location westus2
    ```

1. Создайте спецификацию шаблона в этой группе ресурсов. Задайте для новой спецификации шаблона имя **storageSpec**.

    ```azurepowershell
    New-AzTemplateSpec `
      -Name storageSpec `
      -Version "1.0" `
      -ResourceGroupName templateSpecRG `
      -Location westus2 `
      -TemplateFile "c:\Templates\azuredeploy.json"
    ```

# <a name="cli"></a>[CLI](#tab/azure-cli)

1. Создайте новую группу ресурсов, которая будет содержать спецификацию шаблона.

    ```azurecli
    az group create \
      --name templateSpecRG \
      --location westus2
    ```

1. Создайте спецификацию шаблона в этой группе ресурсов. Задайте для новой спецификации шаблона имя **storageSpec**.

    ```azurecli
    az ts create \
      --name storageSpec \
      --version "1.0" \
      --resource-group templateSpecRG \
      --location "westus2" \
      --template-file "c:\Templates\azuredeploy.json"
    ```

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. Войдите на [портал Azure](https://portal.azure.com).
1. Найдите **спецификацию шаблона**. Из предложенных вариантов выберите **Спецификация шаблона**.

    :::image type="content" source="./media/quickstart-create-template-specs/search-template-spec.png" alt-text="поиск спецификаций шаблона":::

1. Выберите **Импортировать шаблон**.

    :::image type="content" source="./media/quickstart-create-template-specs/import-template.png" alt-text="импорт шаблона":::

1. Выберите значок папки.

    :::image type="content" source="./media/quickstart-create-template-specs/open-folder.png" alt-text="открытие папки":::

1. Перейдите к сохраненному локальному шаблону и выберите его. Выберите **Открыть**.
1. Выберите **Импортировать**.

    :::image type="content" source="./media/quickstart-create-template-specs/select-import.png" alt-text="выбор параметра импорта":::

1. Укажите следующие значения.

    - **Имя**. Введите имя спецификации шаблона.  Например, **storageSpec**.
    - **Подписка**. Выберите подписку Azure, используемую для создания спецификации шаблона.
    - **Группа ресурсов**.Щелкните **Создать** и введите имя новой группы ресурсов.  Например, **templateSpecRG**.
    - **Расположение**. Выберите расположение группы ресурсов. Например, **Западная часть США 2**.
    - **Версия**. Введите версию спецификации шаблона. Используйте версию **1.0**.

1. Выберите **Review + Create** (Просмотреть и создать).
1. Нажмите кнопку **создания**.

# <a name="arm-template"></a>[Шаблон ARM](#tab/azure-resource-manager)

> [!NOTE]
> Для создания спецификации шаблона мы рекомендуем использовать PowerShell или CLI вместо шаблона ARM. Эти инструменты автоматически преобразуют связанные шаблоны в артефакты, подключенные к вашему основному шаблону. При использовании шаблона ARM для создания спецификации шаблона вы должны вручную добавить эти связанные шаблоны в качестве артефактов, что может быть затруднительным.

1. Когда вы используете шаблон ARM для создания спецификации шаблона, шаблон внедряется в определение ресурса. Вам необходимо внести некоторые изменения в свой локальный шаблон. Скопируйте следующий шаблон и сохраните его локально с именем **azuredeploy.json**.

    > [!NOTE]
    > Во внедренном шаблоне все [выражения шаблона](template-expressions.md) необходимо экранировать с помощью второй левой скобки. Используйте `"[[` вместо `"[`. Массивы JSON по-прежнему используют одинарную левую скобку.

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {},
      "functions": [],
      "variables": {},
      "resources": [
        {
          "type": "Microsoft.Resources/templateSpecs",
          "apiVersion": "2019-06-01-preview",
          "name": "storageSpec",
          "location": "westus2",
          "properties": {
            "displayName": "Storage template spec"
          },
          "tags": {},
          "resources": [
            {
              "type": "versions",
              "apiVersion": "2019-06-01-preview",
              "name": "1.0",
              "location": "westus2",
              "dependsOn": [ "storageSpec" ],
              "properties": {
                "template": {
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
                      "defaultValue": "[[resourceGroup().location]",
                      "metadata": {
                        "description": "Location for all resources."
                      }
                    }
                  },
                  "variables": {
                    "storageAccountName": "[[concat('store', uniquestring(resourceGroup().id))]"
                  },
                  "resources": [
                    {
                      "type": "Microsoft.Storage/storageAccounts",
                      "apiVersion": "2019-04-01",
                      "name": "[[variables('storageAccountName')]",
                      "location": "[[parameters('location')]",
                      "sku": {
                        "name": "[[parameters('storageAccountType')]"
                      },
                      "kind": "StorageV2",
                      "properties": {}
                    }
                  ],
                  "outputs": {
                    "storageAccountName": {
                      "type": "string",
                      "value": "[[variables('storageAccountName')]"
                    }
                  }
                }
              },
              "tags": {}
            }
          ]
        }
      ],
      "outputs": {}
    }
    ```

1. Используйте Azure CLI или PowerShell, чтобы создать группу ресурсов.

    ```azurepowershell
    New-AzResourceGroup `
      -Name templateSpecRG `
      -Location westus2
    ```

    ```azurecli
    az group create \
      --name templateSpecRG \
      --location westus2
    ```

1. Разверните шаблон с помощью Azure CLI или PowerShell.

    ```azurepowershell
    New-AzResourceGroupDeployment `
      -ResourceGroupName templateSpecRG `
      -TemplateFile "c:\Templates\azuredeploy.json"
    ```

    ```azurecli
    az deployment group create \
      --name templateSpecRG \
      --template-file "c:\Templates\azuredeploy.json"
    ```

---

## <a name="deploy-template-spec"></a>Развертывание спецификации шаблона

Для развертывания спецификации шаблона используйте те же команды развертывания, что и для развертывания шаблона. Передайте идентификатор ресурса спецификации шаблона для развертывания.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Создайте группу ресурсов, которая будет содержать новую учетную запись хранения.

    ```azurepowershell
    New-AzResourceGroup `
      -Name storageRG `
      -Location westus2
    ```

1. Получите ИД ресурса спецификации шаблона.

    ```azurepowershell
    $id = (Get-AzTemplateSpec -ResourceGroupName templateSpecRG -Name storageSpec -Version "1.0").Versions.Id
    ```

1. Разверните спецификацию шаблона.

    ```azurepowershell
    New-AzResourceGroupDeployment `
      -TemplateSpecId $id `
      -ResourceGroupName storageRG
    ```

1. Укажите параметры таким же образом, как и для шаблона ARM. Повторно разверните спецификацию шаблона с параметром для типа учетной записи хранения.

    ```azurepowershell
    New-AzResourceGroupDeployment `
      -TemplateSpecId $id `
      -ResourceGroupName storageRG `
      -storageAccountType Standard_GRS
    ```

# <a name="cli"></a>[CLI](#tab/azure-cli)

1. Создайте группу ресурсов, которая будет содержать новую учетную запись хранения.

    ```azurecli
    az group create \
      --name storageRG \
      --location westus2
    ```

1. Получите ИД ресурса спецификации шаблона.

    ```azurecli
    id=$(az ts show --name storageSpec --resource-group templateSpecRG --version "1.0" --query "id")
    ```

    > [!NOTE]
    > Существует известная проблема при получении идентификатора спецификации шаблона и ее присвоении переменной в Windows PowerShell.

1. Разверните спецификацию шаблона.

    ```azurecli
    az deployment group create \
      --resource-group storageRG \
      --template-spec $id
    ```

1. Укажите параметры таким же образом, как и для шаблона ARM. Повторно разверните спецификацию шаблона с параметром для типа учетной записи хранения.

    ```azurecli
    az deployment group create \
      --resource-group storageRG \
      --template-spec $id \
      --parameters storageAccountType='Standard_GRS'
    ```

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. Выберите созданную спецификацию шаблона.

    :::image type="content" source="./media/quickstart-create-template-specs/select-template-spec.png" alt-text="выбор имени шаблона":::

1. Выберите **Развернуть**.

    :::image type="content" source="./media/quickstart-create-template-specs/deploy-template-spec.png" alt-text="развертывание спецификации шаблона":::

1. Укажите следующие значения.

    - **Подписка**. Выберите подписку Azure для создания ресурса.
    - **Группа ресурсов**. Выберите **Создать**, а затем введите **storageRG**.
    - **Тип учетной записи хранения**. Выберите **Standard_GRS**.

1. Выберите **Review + create** (Просмотреть и создать).
1. Нажмите кнопку **создания**.

# <a name="arm-template"></a>[Шаблон ARM](#tab/azure-resource-manager)

1. Скопируйте следующий шаблон и сохраните его локально в файл с именем **storage.json.** .

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {},
      "functions": [],
      "variables": {},
      "resources": [
        {
          "type": "Microsoft.Resources/deployments",
          "apiVersion": "2020-10-01",
          "name": "demo",
          "properties": {
            "templateLink": {
              "id": "[resourceId('templateSpecRG', 'Microsoft.Resources/templateSpecs/versions', 'storageSpec', '1.0')]"
            },
            "parameters": {
            },
            "mode": "Incremental"
          }
        }
      ],
      "outputs": {}
    }
    ```

1. Используйте Azure CLI или PowerShell для создания новой группы ресурсов для учетной записи хранения.

    ```azurepowershell
    New-AzResourceGroup `
      -Name storageRG `
      -Location westus2
    ```

    ```azurecli
    az group create \
      --name storageRG \
      --location westus2
    ```

1. Разверните шаблон с помощью Azure CLI или PowerShell.

    ```azurepowershell
    New-AzResourceGroupDeployment `
      -ResourceGroupName storageRG `
      -TemplateFile "c:\Templates\storage.json"
    ```

    ```azurecli
    az deployment group create \
      --name storageRG \
      --template-file "c:\Templates\storage.json"
    ```

---

## <a name="grant-access"></a>Предоставление доступа

Если вы хотите позволить другим пользователям в организации развертывать спецификацию шаблона, необходимо предоставить им доступ на чтение. Роль читателя можно назначить группе Azure AD для группы ресурсов, содержащей спецификации шаблонов, к которым вы хотите предоставить общий доступ. Дополнительные сведения см. в статье [Учебник. Предоставление доступа группам к ресурсам Azure с помощью Azure PowerShell](../../role-based-access-control/tutorial-role-assignments-group-powershell.md).

## <a name="update-template"></a>Обновление шаблона

Предположим, вы определили изменение, которое необходимо внести в шаблон, в спецификации шаблона. Следующий шаблон похож на ваш предыдущий шаблон, за исключением того, что он добавляет префикс для имени учетной записи хранения. Скопируйте следующий шаблон и обновите файл azuredeploy.json.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/templatespecs/azuredeploy2.json":::

## <a name="update-template-spec-version"></a>Обновление версии спецификации шаблона

Вместо создания новой спецификации шаблона для измененного шаблона добавьте новую версию с именем `2.0` в имеющуюся спецификацию шаблона. Пользователи могут выбрать любую версию для развертывания.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Создайте новую версию спецификации шаблона.

   ```powershell
   New-AzTemplateSpec `
     -Name storageSpec `
     -Version "2.0" `
     -ResourceGroupName templateSpecRG `
     -Location westus2 `
     -TemplateFile "c:\Templates\azuredeploy.json"
   ```

1. Чтобы развернуть новую версию, получите идентификатор ресурса для версии `2.0`.

   ```powershell
   $id = (Get-AzTemplateSpec -ResourceGroupName templateSpecRG -Name storageSpec -Version "2.0").Versions.Id
   ```

1. Разверните эту версию. Укажите префикс для имени учетной записи хранения.

   ```powershell
   New-AzResourceGroupDeployment `
     -TemplateSpecId $id `
     -ResourceGroupName storageRG `
     -namePrefix "demoaccount"
   ```

# <a name="cli"></a>[CLI](#tab/azure-cli)

1. Создайте новую версию спецификации шаблона.

   ```azurecli
   az ts create \
     --name storageSpec \
     --version "2.0" \
     --resource-group templateSpecRG \
     --location "westus2" \
     --template-file "c:\Templates\azuredeploy.json"
   ```

1. Чтобы развернуть новую версию, получите идентификатор ресурса для версии `2.0`.

   ```azurecli
   id=$(az ts show --name storageSpec --resource-group templateSpecRG --version "2.0" --query "id")
   ```

1. Разверните эту версию. Укажите префикс для имени учетной записи хранения.

   ```azurecli
    az deployment group create \
      --resource-group storageRG \
      --template-spec $id \
      --parameters namePrefix='demoaccount'
    ```

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. В спецификации шаблона выберите **Создание новой версии**.

   :::image type="content" source="./media/quickstart-create-template-specs/select-versions.png" alt-text="создание новой версии":::

1. Задайте имя для новой версии `2.0` и при необходимости добавьте заметки. Выберите **Изменить шаблон**.

   :::image type="content" source="./media/quickstart-create-template-specs/add-version-name.png" alt-text="задание имени для новой версии":::

1. Замените содержимое шаблона обновленным шаблоном. Выберите **Проверить и сохранить**.
1. Щелкните **Save changes** (Сохранить изменения).

1. Чтобы развернуть новую версию, выберите **Версии**.

   :::image type="content" source="./media/quickstart-create-template-specs/see-versions.png" alt-text="список версий":::

1. Для версии, которую необходимо развернуть, выберите три точки и **Развернуть**.

   :::image type="content" source="./media/quickstart-create-template-specs/deploy-version.png" alt-text="выбор версии для развертывания":::

1. Заполните поля, как и при развертывании предыдущей версии.
1. Выберите **Review + create** (Просмотреть и создать).
1. Нажмите кнопку **создания**.

# <a name="arm-template"></a>[Шаблон ARM](#tab/azure-resource-manager)

1. Опять же, вы должны внести некоторые изменения в свой локальный шаблон, чтобы он взаимодействовал со спецификациями шаблона. Скопируйте следующий шаблон и сохраните его локально с именем azuredeploy.json.

   ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {},
      "functions": [],
      "variables": {},
      "resources": [
        {
          "type": "Microsoft.Resources/templateSpecs",
          "apiVersion": "2019-06-01-preview",
          "name": "storageSpec",
          "location": "westus2",
          "properties": {
            "displayName": "Storage template spec"
          },
          "tags": {},
          "resources": [
            {
              "type": "versions",
              "apiVersion": "2019-06-01-preview",
              "name": "2.0",
              "location": "westus2",
              "dependsOn": [ "storageSpec" ],
              "properties": {
                "template": {
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
                      "defaultValue": "[[resourceGroup().location]",
                      "metadata": {
                        "description": "Location for all resources."
                      }
                    },
                    "namePrefix": {
                      "type": "string",
                      "maxLength": 11,
                      "defaultValue": "store",
                      "metadata": {
                        "description": "Prefix for storage account name"
                      }
                    }
                  },
                  "variables": {
                    "storageAccountName": "[[concat(parameters('namePrefix'), uniquestring(resourceGroup().id))]"
                  },
                  "resources": [
                    {
                      "type": "Microsoft.Storage/storageAccounts",
                      "apiVersion": "2019-04-01",
                      "name": "[[variables('storageAccountName')]",
                      "location": "[[parameters('location')]",
                      "sku": {
                        "name": "[[parameters('storageAccountType')]"
                      },
                      "kind": "StorageV2",
                      "properties": {}
                    }
                  ],
                  "outputs": {
                    "storageAccountName": {
                      "type": "string",
                      "value": "[[variables('storageAccountName')]"
                    }
                  }
                }
              },
              "tags": {}
            }
          ]
        }
      ],
      "outputs": {}
    }
    ```

1. Чтобы добавить новую версию в спецификацию шаблона, разверните шаблон с помощью Azure CLI или PowerShell.

    ```azurepowershell
    New-AzResourceGroupDeployment `
      -ResourceGroupName templateSpecRG `
      -TemplateFile "c:\Templates\azuredeploy.json"
    ```

    ```azurecli
    az deployment group create \
      --name templateSpecRG \
      --template-file "c:\Templates\azuredeploy.json"
    ```

1. Скопируйте следующий шаблон и сохраните его локально в файл с именем **storage.json.** .

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {},
      "functions": [],
      "variables": {},
      "resources": [
        {
          "type": "Microsoft.Resources/deployments",
          "apiVersion": "2020-10-01",
          "name": "demo",
          "properties": {
            "templateLink": {
              "id": "[resourceId('templateSpecRG', 'Microsoft.Resources/templateSpecs/versions', 'storageSpec', '2.0')]"
            },
            "parameters": {
            },
            "mode": "Incremental"
          }
        }
      ],
      "outputs": {}
    }
    ```

1. Разверните шаблон с помощью Azure CLI или PowerShell.

    ```azurepowershell
    New-AzResourceGroupDeployment `
      -ResourceGroupName storageRG `
      -TemplateFile "c:\Templates\storage.json"
    ```

    ```azurecli
    az deployment group create \
      --name storageRG \
      --template-file "c:\Templates\storage.json"
    ```

---

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы очистить ресурс, развернутый в этом кратком руководстве, удалите обе созданные группы ресурсов.

1. На портале Azure в меню слева выберите "Группа ресурсов".

1. В поле "Фильтровать по имени" введите имя группы ресурсов (templateSpecRG и storageRG).

1. Выберите имя группы ресурсов.

1. В меню сверху выберите "Удалить группу ресурсов".

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о создании спецификации шаблона, включающей связанные шаблоны, см. в статье [Создание спецификации для связанного шаблона](template-specs-create-linked.md).
