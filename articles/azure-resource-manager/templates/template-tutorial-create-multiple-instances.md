---
title: Создание нескольких экземпляров ресурсов
description: Сведения о том, как создать шаблон Resource Manager для создания нескольких экземпляров ресурса Azure.
author: mumian
ms.date: 04/23/2020
ms.topic: tutorial
ms.author: jgao
ms.custom: ''
ms.openlocfilehash: e669e27547633639a88674ffee499fb1d84facdf
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97673959"
---
# <a name="tutorial-create-multiple-resource-instances-with-arm-templates"></a>Руководство по Создание нескольких экземпляров ресурса с помощью шаблонов ARM

Узнайте, как выполнить итерацию в шаблоне Azure Resource Manager для создания нескольких экземпляров ресурса Azure. В этом руководстве описано, как изменить шаблон для создания трех экземпляров учетной записи хранения.

![Создание нескольких экземпляров с помощью Azure Resource Manager](./media/template-tutorial-create-multiple-instances/resource-manager-template-create-multiple-instances-diagram.png)

В рамках этого руководства рассматриваются следующие задачи:

> [!div class="checklist"]
> * Открытие шаблона быстрого запуска
> * Изменение шаблона
> * Развертывание шаблона

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

Сведения о модулях Microsoft Learn с описанием копирования ресурсов см. в статье [Управление сложными облачными развертываниями с помощью расширенных функций шаблонов Resource Manager](/learn/modules/manage-deployments-advanced-arm-template-features/).

## <a name="prerequisites"></a>Предварительные требования

Для работы с этой статьей необходимо иметь следующее.

* Visual Studio Code с расширением средств Resource Manager. См. [Краткое руководство. Создание шаблонов ARM с помощью Visual Studio Code](quickstart-create-templates-use-visual-studio-code.md).

## <a name="open-a-quickstart-template"></a>Открытие шаблона быстрого запуска

[Шаблоны быстрого запуска Azure](https://azure.microsoft.com/resources/templates/) — это репозиторий для шаблонов Resource Manager. Вместо создания шаблона с нуля можно найти пример шаблона и настроить его. Шаблон, используемый в этом кратком руководстве, называется [Create a standard storage account](https://azure.microsoft.com/resources/templates/101-storage-account-create/) (Создание стандартной учетной записи хранения). Шаблон определяет ресурс учетной записи службы хранилища Azure.

1. В Visual Studio Code выберите **Файл** > **Открыть файл**.
1. Скопируйте приведенный ниже URL-адрес и вставьте его в поле **Имя файла**.

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json
    ```

1. Чтобы открыть файл, выберите **Открыть**.
1. В шаблоне определен ресурс `Microsoft.Storage/storageAccounts`. Сравните шаблон с примером в [справочнике по шаблону](/azure/templates/Microsoft.Storage/storageAccounts). Прежде чем настраивать шаблон, рекомендуется получить основные сведения о нем.
1. Выберите **Файл** > **Сохранить как**, чтобы сохранить файл в качестве _azuredeploy.json_ на локальном компьютере.

## <a name="edit-the-template"></a>Изменение шаблона

С помощью существующего шаблона можно создать одну учетную запись хранения. Можно настроить шаблон, чтобы создать три учетные записи хранения.

Внесите следующие четыре изменения в Visual Studio Code.

![Создание нескольких экземпляров с помощью Azure Resource Manager](./media/template-tutorial-create-multiple-instances/resource-manager-template-create-multiple-instances.png)

1. Добавьте элемент `copy` к определению ресурса учетной записи хранения. В элементе `copy` укажите число итераций и переменную для этого цикла. Значение count должно быть положительным целым числом не больше 800.
2. Функция `copyIndex()` возвращает текущую итерацию в цикле. Индекс используется в качестве префикса имени. `copyIndex()` отсчитывается, начиная с нуля. Чтобы сместить значение индекса, можно передать нужное значение в функцию `copyIndex()`. Например, `copyIndex(1)`.
3. Удалите элемент `variables`, так как он больше не используется.
4. Удалите элемент `outputs`. Он больше не требуется.

Завершенный шаблон выглядит следующим образом.

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
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-04-01",
      "name": "[concat(copyIndex(),'storage', uniqueString(resourceGroup().id))]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "[parameters('storageAccountType')]"
      },
      "kind": "StorageV2",
      "copy": {
        "name": "storagecopy",
        "count": 3
      },
      "properties": {}
    }
  ]
}
```

Дополнительные сведения о создании нескольких экземпляров см. в статье [Итерация ресурсов в шаблонах Resource Manager](./copy-resources.md).

## <a name="deploy-the-template"></a>Развертывание шаблона

1. Войдите в [Azure Cloud Shell](https://shell.azure.com).

1. В левом верхнем углу выберите используемую среду — **PowerShell** или **Bash** (для CLI). После переключения желательно перезагрузить оболочку.

    ![Файл отправки Cloud Shell на портале Azure](./media/template-tutorial-use-template-reference/azure-portal-cloud-shell-upload-file.png)

1. Выберите **Отправка и скачивание файлов**, а затем **Отправить**. См. предыдущий снимок экрана. Выберите файл, сохраненный ранее. После отправки вы можете использовать команды `ls` и `cat`, чтобы проверить отправку файла.

1. Выполните следующие команды в Cloud Shell. Выберите вкладку, чтобы отобразить код PowerShell или код CLI.

    # <a name="cli"></a>[CLI](#tab/CLI)

    ```azurecli
    echo "Enter a project name that is used to generate resource group name:" &&
    read projectName &&
    echo "Enter the location (i.e. centralus):" &&
    read location &&
    resourceGroupName="${projectName}rg" &&
    az group create --name $resourceGroupName --location "$location" &&
    az deployment group create --resource-group $resourceGroupName --template-file "$HOME/azuredeploy.json"
    ```

    # <a name="powershell"></a>[PowerShell](#tab/PowerShell)

    ```azurepowershell
    $projectName = Read-Host -Prompt "Enter a project name that is used to generate resource group name"
    $location = Read-Host -Prompt "Enter the location (i.e. centralus)"
    $resourceGroupName = "${projectName}rg"

    New-AzResourceGroup -Name $resourceGroupName -Location "$location"
    New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateFile "$HOME/azuredeploy.json"
    ```

    ---

После успешного развертывания шаблона можно отобразить три учетные записи хранения, которые были созданы в указанной группе ресурсов. Сравните имена учетных записей хранения с определением имени в шаблоне.

# <a name="cli"></a>[CLI](#tab/azure-cli)

```azurecli
echo "Enter a project name that is used to generate resource group name:" &&
read projectName &&
resourceGroupName="${projectName}rg" &&
az storage account list --resource-group $resourceGroupName &&
echo "Press [ENTER] to continue ..."
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$projectName = Read-Host -Prompt "Enter a project name that is used to generate resource group name"
$resourceGroupName = "${projectName}rg"

Get-AzStorageAccount -ResourceGroupName $resourceGroupName
Write-Host "Press [ENTER] to continue ..."
```

---

## <a name="clean-up-resources"></a>Очистка ресурсов

Если ресурсы Azure больше не нужны, их можно удалить. Для этого необходимо удалить группу ресурсов.

1. На портале Azure в меню слева выберите **Группа ресурсов**.
2. В поле **Фильтровать по имени** введите имя группы ресурсов.
3. Выберите имя группы ресурсов.  В группе ресурсов должно появиться три ресурса.
4. В главном меню выберите **Удалить группу ресурсов**.

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как создать несколько экземпляров учетной записи хранения. С помощью следующего руководства вы разработаете шаблон с несколькими ресурсами и несколькими типами ресурсов. Некоторые ресурсы обладают зависимыми ресурсами.

> [!div class="nextstepaction"]
> [Создание шаблонов Azure Resource Manager с зависимыми ресурсами](./template-tutorial-create-templates-with-dependent-resources.md)
