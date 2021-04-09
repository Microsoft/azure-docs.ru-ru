---
title: Использование условия в шаблонах
description: Узнайте, как развернуть ресурсы Azure на основе условий. В этой статье также объясняется, как развернуть новый ресурс либо использовать существующий.
author: mumian
ms.date: 04/23/2020
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: 4affc2add2822702c1d5395f81efe01eeedf448b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98696029"
---
# <a name="tutorial-use-condition-in-arm-templates"></a>Руководство по Использование условия в шаблонах ARM

Узнайте о том, как развернуть ресурсы Azure на основе условий в шаблоне Azure Resource Manager.

В руководстве по [настройке порядка развертывания ресурсов](./template-tutorial-create-templates-with-dependent-resources.md) описано, как создать виртуальную машину, виртуальную сеть и некоторые другие зависимые ресурсы, включая учетную запись хранения. Чтобы не создавать учетную запись хранения каждый раз, вы можете позволить пользователям выбрать, создавать ли новую учетную запись хранения или использовать существующую. Для достижения этой цели нужно определить дополнительный параметр. Если значение параметра равно **new**, будет создана новая учетная запись хранения. В противном случае используется учетная запись хранения с указанным именем.

![Схема использования условия в шаблонах Azure Resource Manager](./media/template-tutorial-use-conditions/resource-manager-template-use-condition-diagram.png)

В рамках этого руководства рассматриваются следующие задачи:

> [!div class="checklist"]
> * Открытие шаблона быстрого запуска
> * Изменение шаблона
> * Развертывание шаблона
> * Очистка ресурсов

В этом учебнике рассматривается только основной сценарий с использованием условий. Дополнительные сведения см. в разделе:

* [Структура файла шаблона: условие](conditional-resource-deployment.md).
* [Условное развертывание ресурса в шаблоне ARM](/azure/architecture/guide/azure-resource-manager/advanced-templates/conditional-deploy).
* [Функция шаблона: If](./template-functions-logical.md#if).
* [Функции сравнения для шаблонов ARM](./template-functions-comparison.md)

Сведения о модулях Microsoft Learn с описанием условий см. в статье [Управление сложными облачными развертываниями с помощью расширенных функций шаблонов Resource Manager](/learn/modules/manage-deployments-advanced-arm-template-features/).

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этой статьей необходимо иметь следующее.

* Visual Studio Code с расширением средств Resource Manager. См. [Краткое руководство. Создание шаблонов ARM с помощью Visual Studio Code](quickstart-create-templates-use-visual-studio-code.md).
* Для повышения уровня безопасности используйте пароль, созданный для учетной записи администратора виртуальной машины. Ниже приведен пример создания пароля.

    ```console
    openssl rand -base64 32
    ```

    Для защиты криптографических ключей и других секретов используйте Azure Key Vault. Дополнительные сведения см. в статье [Учебник. Интеграция с Azure Key Vault при развертывании шаблона ARM](./template-tutorial-use-key-vault.md). Мы также рекомендуем обновлять пароль каждые три месяца.

## <a name="open-a-quickstart-template"></a>Открытие шаблона быстрого запуска

Шаблоны быстрого запуска Azure — это репозиторий для шаблонов ARM. Вместо создания шаблона с нуля можно найти пример шаблона и настроить его. Шаблон, используемый в этом руководстве, называется [Deploy a simple Windows VM](https://azure.microsoft.com/resources/templates/101-vm-simple-windows/) (Развертывание простой виртуальной машины Windows).

1. В Visual Studio Code выберите **Файл** > **Открыть файл**.
1. Скопируйте приведенный ниже URL-адрес и вставьте его в поле **Имя файла**.

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-simple-windows/azuredeploy.json
    ```

1. Чтобы открыть файл, выберите **Открыть**.
1. Шаблоном определено шесть ресурсов:

   * [**Microsoft.Storage/storageAccounts**](/azure/templates/Microsoft.Storage/storageAccounts).
   * [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicipaddresses).
   * [**Microsoft.Network/networkSecurityGroups**](/azure/templates/microsoft.network/networksecuritygroups).
   * [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks).
   * [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkinterfaces).
   * [**Microsoft.Compute/virtualMachines**](/azure/templates/microsoft.compute/virtualmachines).

    Перед настройкой шаблона рекомендуется ознакомиться с его справочником.

1. Выберите **Файл** > **Сохранить как**, чтобы сохранить файл на локальный компьютер с именем _azuredeploy.json_.

## <a name="modify-the-template"></a>Изменение шаблона

Внесите два изменения в имеющийся шаблон:

* Добавьте параметр имени для учетной записи хранения. Пользователи могут указать имя новой или существующей учетной записи хранения.
* Добавьте новый параметр с именем `newOrExisting`. Развертывание использует этот параметр для определения того, нужно ли создавать новую учетную запись хранения или использовать имеющуюся.

Вот что нужно сделать, чтобы внести изменения:

1. Откройте файл _azuredeploy.json_ в Visual Studio Code.
1. Замените во всем шаблоне `variables('storageAccountName')` на `parameters('storageAccountName')` .
1. Удалите следующее определение переменной:

    ![Снимок экрана с выделенным определением переменной, которое необходимо удалить.](./media/template-tutorial-use-conditions/resource-manager-tutorial-use-condition-template-remove-storageaccountname.png)

1. Добавьте следующие два параметра в начало раздела параметров:

    ```json
    "storageAccountName": {
      "type": "string"
    },
    "newOrExisting": {
      "type": "string",
      "allowedValues": [
        "new",
        "existing"
      ]
    },
    ```

    Нажмите клавиши ALT+SHIFT+F, чтобы отформатировать шаблон в Visual Studio Code.

    Определение обновленных параметров выглядит так:

    ![Условие использования Resource Manager](./media/template-tutorial-use-conditions/resource-manager-tutorial-use-condition-template-parameters.png)

1. Добавьте следующую строку в начало определения учетной записи хранения.

    ```json
    "condition": "[equals(parameters('newOrExisting'),'new')]",
    ```

    Условие проверяет значение параметра `newOrExisting`. Если значение параметра равно **new**, то в результате развертывания создается учетная запись хранения.

    Обновленное определение учетной записи хранения выглядит так:

    ![Снимок экрана, на котором показано обновленное определение учетной записи хранения.](./media/template-tutorial-use-conditions/resource-manager-tutorial-use-condition-template.png)
1. Обновите свойство `storageUri` определения ресурса виртуальной машины следующим значением:

    ```json
    "storageUri": "[concat('https://', parameters('storageAccountName'), '.blob.core.windows.net')]"
    ```

    Это изменение необходимо при использовании существующей учетной записи хранения с другой группой ресурсов.

1. Сохраните изменения.

## <a name="deploy-the-template"></a>Развертывание шаблона

1. Войдите в [Azure Cloud Shell](https://shell.azure.com).

1. В левом верхнем углу выберите используемую среду — **PowerShell** или **Bash** (для CLI). После переключения желательно перезагрузить оболочку.

    ![Файл отправки Cloud Shell на портале Azure](./media/template-tutorial-use-template-reference/azure-portal-cloud-shell-upload-file.png)

1. Выберите **Отправка и скачивание файлов**, а затем **Отправить**. См. предыдущий снимок экрана. Выберите файл, сохраненный ранее. После отправки вы можете использовать команды `ls` и `cat`, чтобы проверить отправку файла.

1. а затем выполните следующий сценарий PowerShell для его развертывания.

    > [!IMPORTANT]
    > Имя учетной записи хранения должно быть уникальным в среде Azure. Имя должно содержать только строчные буквы или цифры. Оно должно содержать не больше 24 знаков. Имя учетной записи хранения — это имя проекта, к которому добавлено слово **store**. Убедитесь, что имя проекта и созданное имя учетной записи хранения соответствуют требованиям к имени учетной записи хранения.

    ```azurepowershell
    $projectName = Read-Host -Prompt "Enter a project name that is used to generate resource group name and resource names"
    $newOrExisting = Read-Host -Prompt "Create new or use existing (Enter new or existing)"
    $location = Read-Host -Prompt "Enter the Azure location (i.e. centralus)"
    $vmAdmin = Read-Host -Prompt "Enter the admin username"
    $vmPassword = Read-Host -Prompt "Enter the admin password" -AsSecureString
    $dnsLabelPrefix = Read-Host -Prompt "Enter the DNS Label prefix"

    $resourceGroupName = "${projectName}rg"
    $storageAccountName = "${projectName}store"

    New-AzResourceGroup -Name $resourceGroupName -Location $location
    New-AzResourceGroupDeployment `
        -ResourceGroupName $resourceGroupName `
        -adminUsername $vmAdmin `
        -adminPassword $vmPassword `
        -dnsLabelPrefix $dnsLabelPrefix `
        -storageAccountName $storageAccountName `
        -newOrExisting $newOrExisting `
        -TemplateFile "$HOME/azuredeploy.json"

    Write-Host "Press [ENTER] to continue ..."
    ```

    > [!NOTE]
    > Развертывание завершается сбоем, если `newOrExisting` — **new**, а учетная запись хранения с указанным именем уже существует.

Попробуйте выполнить другое развертывания, задав для `newOrExisting` значение **existing** и указав имеющуюся учетную запись хранения. Чтобы создать учетную запись хранения, обратитесь к [этой статье](../../storage/common/storage-account-create.md).

## <a name="clean-up-resources"></a>Очистка ресурсов

Если ресурсы Azure больше не нужны, их можно удалить. Для этого необходимо удалить группу ресурсов. Для удаления группы ресурсов выберите **Попробовать**, чтобы открыть Cloud Shell. Чтобы вставить сценарий PowerShell, щелкните панель оболочки правой кнопкой мыши и выберите **Вставить**.

```azurepowershell-interactive
$projectName = Read-Host -Prompt "Enter the same project name you used in the last procedure"
$resourceGroupName = "${projectName}rg"

Remove-AzResourceGroup -Name $resourceGroupName

Write-Host "Press [ENTER] to continue ..."
```

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве описано, как разработать шаблон, позволяющий пользователю выбирать между созданием учетной записи хранения и использованием существующей. Чтобы узнать, как получать секреты из Azure Key Vault и использовать их в шаблоне развертывания в качестве паролей, обратитесь к статье:

> [!div class="nextstepaction"]
> [Руководство. Интеграция Azure Key Vault в развертывание шаблона Resource Manager](./template-tutorial-use-key-vault.md)
