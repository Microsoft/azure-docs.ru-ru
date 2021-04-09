---
title: Использование Azure Key Vault в шаблонах
description: Сведения о том, как использовать Azure Key Vault для передачи значений безопасного параметра во время развертывания шаблона Resource Manager.
author: mumian
ms.date: 03/01/2021
ms.topic: tutorial
ms.author: jgao
ms.custom: seodec18
ms.openlocfilehash: 388996dc0054192f6d9f3c87e11ca1d15e8a85e1
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101703891"
---
# <a name="tutorial-integrate-azure-key-vault-in-your-arm-template-deployment"></a>Руководство по Интеграция с Azure Key Vault при развертывании шаблона ARM

Узнайте, как получить секреты из Azure Key Vault и передать эти секреты в качестве параметров во время развертывания шаблона Resource Manager. Значение параметра никогда не предоставляется, так как указывается только его идентификатор в хранилище ключей. Вы можете создать ссылку на секрет хранилища ключей, используя статический или динамический идентификатор. В этом руководстве используется статический идентификатор. С помощью статического идентификатора можно создать ссылку на хранилище ключей в файле параметров шаблона, а не файле шаблона. См. сведения об двух этих подходах в руководстве по [использованию Azure Key Vault для передачи защищенного значения параметра во время развертывания](./key-vault-parameter.md).

С помощью инструкций из [учебника по настройке порядка развертывания ресурсов](./template-tutorial-create-templates-with-dependent-resources.md) вы создадите виртуальную машину. Вам нужно предоставить имя и пароль учетной записи администратора виртуальной машины. Чтобы не предоставлять пароль, можно предварительно сохранить его в Azure Key Vault, а затем настроить шаблона для извлечения пароля из хранилища ключей во время развертывания.

![Схема, отображающая процесс интеграции шаблона Resource Manager с хранилищем ключей](./media/template-tutorial-use-key-vault/resource-manager-template-key-vault-diagram.png)

В рамках этого руководства рассматриваются следующие задачи:

> [!div class="checklist"]
> * Подготовка хранилища ключей.
> * Открытие шаблона быстрого запуска
> * Изменение файла параметров
> * Развертывание шаблона
> * Проверка развертывания
> * Очистка ресурсов

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

Сведения о модулях Microsoft Learn, в котором описывается использование безопасного значения из хранилища ключей, см. в статье [Управление сложными облачными развертываниями с помощью расширенных функций шаблонов Resource Manager](/learn/modules/manage-deployments-advanced-arm-template-features/).

## <a name="prerequisites"></a>Предварительные требования

Для работы с этой статьей необходимо иметь следующее.

* Visual Studio Code с расширением средств Resource Manager. См. [Краткое руководство. Создание шаблонов ARM с помощью Visual Studio Code](quickstart-create-templates-use-visual-studio-code.md).
* С целью повышения уровня безопасности используйте пароль, созданный для учетной записи администратора виртуальной машины. Ниже приведен пример создания пароля.

    ```console
    openssl rand -base64 32
    ```

    Убедитесь, что созданный пароль соответствует требованиям к паролю виртуальной машины. У каждой службы Azure есть определенные требования к паролю. Требования к паролю виртуальной машины см. в разделе [Какие требования к паролю при создании виртуальной машины?](../../virtual-machines/windows/faq.md#what-are-the-password-requirements-when-creating-a-vm).

## <a name="prepare-a-key-vault"></a>Подготовка хранилища ключей.

Из этого раздела вы узнаете, как создать хранилище ключей и добавить в него секрет, который будет получен при развертывании шаблона. Существует много способов создания хранилища ключей. В этом руководстве показано, как развернуть [шаблон ARM](https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/tutorials-use-key-vault/CreateKeyVault.json) с помощью Azure PowerShell. Этот шаблон делает две вещи:

* Создает хранилище ключей с включенным свойством `enabledForTemplateDeployment`. Это свойство должно получить значение *true* до того, как процесс развертывания шаблона сможет получить доступ к секретам, определенным в этом хранилище ключей.
* Добавляет секрет в хранилище ключей. Секрет хранит пароль учетной записи администратора виртуальной машины.

> [!NOTE]
> Если вы (в качестве пользователя для развертывания шаблона виртуальной машины) не являетесь владельцем или участником хранилища ключей, владелец или участник хранилища ключей должен предоставить вам доступ к разрешению `Microsoft.KeyVault/vaults/deploy/action` для хранилища ключей. Дополнительные сведения см. в статье [Использование Azure Key Vault для передачи защищенного значения параметра во время развертывания](./key-vault-parameter.md).

Чтобы запустить следующий сценарий Azure PowerShell, выберите вкладку **Попробовать** для открытия Azure Cloud Shell. Чтобы вставить скрипт, щелкните правой кнопкой на панели оболочки и выберите **Вставить**.

```azurepowershell-interactive
$projectName = Read-Host -Prompt "Enter a project name that is used for generating resource names"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$upn = Read-Host -Prompt "Enter your user principal name (email address) used to sign in to Azure"
$secretValue = Read-Host -Prompt "Enter the virtual machine administrator password" -AsSecureString

$resourceGroupName = "${projectName}rg"
$keyVaultName = $projectName
$adUserId = (Get-AzADUser -UserPrincipalName $upn).Id
$templateUri = "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/tutorials-use-key-vault/CreateKeyVault.json"

New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri -keyVaultName $keyVaultName -adUserId $adUserId -secretValue $secretValue

Write-Host "Press [ENTER] to continue ..."
```

> [!IMPORTANT]
> * Обратите внимание, что имя группы ресурсов — это имя проекта с добавлением **rg**. Чтобы было проще [удалить ресурсы, созданные при работе с этим учебником](#clean-up-resources), используйте то же имя проекта и имя группы ресурсов при [развертывании следующего шаблона](#deploy-the-template).
> * Имя секрета по умолчанию — **vmAdminPassword**. Оно встроено в шаблон.
> * Чтобы шаблон мог извлечь секрет, необходимо активировать политику доступа к хранилищу ключей, предоставленную параметром **Включить доступ к Azure Resource Manager для развертывания шаблонов**. В шаблоне эта политика включена. Подробные сведения об этой политике доступа см. в разделе о [развертывании хранилищ ключей и секретов](./key-vault-parameter.md#deploy-key-vaults-and-secrets).

У шаблона есть выходное значение с именем `keyVaultId`. Этот идентификатор будет использоваться вместе с именем секрета для получения значения секрета далее в этом руководстве. Формат идентификатора ресурса выглядит следующим образом:

```json
/subscriptions/<SubscriptionID>/resourceGroups/mykeyvaultdeploymentrg/providers/Microsoft.KeyVault/vaults/<KeyVaultName>
```

При копировании и вставке идентификатора он может быть разбит на несколько строк. Вам необходимо объединить строки и удалить лишние пробелы.

Чтобы проверить развертывание, выполните следующую команду PowerShell в той же панели оболочки, чтобы получить секрет в виде простого текста. Сценарий работает только в том же сеансе оболочки, так как используется переменная `$keyVaultName`, которая определена в приведенном выше сценарии PowerShell.

```azurepowershell
$secret = Get-AzKeyVaultSecret -VaultName $keyVaultName -Name "vmAdminPassword"
$ssPtr = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($secret.SecretValue)
try {
   $secretValueText = [System.Runtime.InteropServices.Marshal]::PtrToStringBSTR($ssPtr)
} finally {
   [System.Runtime.InteropServices.Marshal]::ZeroFreeBSTR($ssPtr)
}
Write-Output $secretValueText
```

Хранилище ключей и секрет подготовлены. В следующих разделах описано, как настроить существующий шаблон, чтобы получить секрет во время развертывания.

## <a name="open-a-quickstart-template"></a>Открытие шаблона быстрого запуска

Шаблоны быстрого запуска Azure — это репозиторий для шаблонов ARM. Вместо создания шаблона с нуля можно найти пример шаблона и настроить его. Шаблон, используемый в этом учебнике, называется [Deploy a simple Windows VM](https://azure.microsoft.com/resources/templates/101-vm-simple-windows/) (Развертывание простой виртуальной машины Windows).

1. В Visual Studio Code выберите **Файл** > **Открыть файл**.

1. В поле **Имя файла** вставьте следующий URL-адрес:

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-simple-windows/azuredeploy.json
    ```

1. Чтобы открыть файл, выберите **Открыть**. Сценарий аналогичен тому, который используется в статье [Руководство. Создание шаблонов Resource Manager с зависимыми ресурсами](./template-tutorial-create-templates-with-dependent-resources.md).
   Шаблон определяет шесть ресурсов:

   * [**Microsoft.Storage/storageAccounts**](/azure/templates/Microsoft.Storage/storageAccounts).
   * [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicipaddresses).
   * [**Microsoft.Network/networkSecurityGroups**](/azure/templates/microsoft.network/networksecuritygroups).
   * [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks).
   * [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkinterfaces).
   * [**Microsoft.Compute/virtualMachines**](/azure/templates/microsoft.compute/virtualmachines).

   Рекомендуется получить основные сведения о шаблоне, прежде чем переходить к его настройке.

1. Выберите **Файл** > **Сохранить как**, а затем сохраните на свой локальный компьютер копию файла под именем *azuredeploy.json*.

1. Повторите шаги 1–3, чтобы открыть следующий URL-адрес, а затем сохраните файл как *azuredeploy.parameters.json*.

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-simple-windows/azuredeploy.parameters.json
    ```

## <a name="edit-the-parameters-file"></a>Изменение файла параметров

При использовании статического идентификатора вам не нужно вносить изменения в файл шаблона. Получение значения секрета выполняется путем настройки файла параметров шаблона.

1. В Visual Studio Code откройте файл *azuredeploy.parameters.json*, если он еще не открыт.
1. Укажите для параметра `adminPassword` следующий код.

    ```json
    "adminPassword": {
        "reference": {
            "keyVault": {
                "id": "/subscriptions/<SubscriptionID>/resourceGroups/mykeyvaultdeploymentrg/providers/Microsoft.KeyVault/vaults/<KeyVaultName>"
            },
            "secretName": "vmAdminPassword"
        }
    },
    ```

    > [!IMPORTANT]
    > Замените значение для `id` на идентификатор ресурса хранилища ключей, созданного во время предыдущей процедуры. Значение `secretName` прописывается в коде в виде **vmAdminPassword**.  См. сведения о [подготовке хранилища ключей](#prepare-a-key-vault).

    ![Интеграция Key Vault и файла параметров развертывания виртуальной машины шаблона Resource Manager](./media/template-tutorial-use-key-vault/resource-manager-tutorial-create-vm-parameters-file.png)

1. Измените следующие значения:

    * `adminUsername`. Имя учетной записи администратора виртуальной машины.
    * `dnsLabelPrefix`. Присвойте имя значению `dnsLabelPrefix`.

    Примеры имен см. на предыдущем рисунке.

1. Сохраните изменения.

## <a name="deploy-the-template"></a>Развертывание шаблона

1. Войдите в [Azure Cloud Shell](https://shell.azure.com).

1. В левом верхнем углу выберите используемую среду — **PowerShell** или **Bash** (для CLI).  После переключения желательно перезагрузить оболочку.

    ![Файл отправки Cloud Shell на портале Azure](./media/template-tutorial-use-template-reference/azure-portal-cloud-shell-upload-file.png)

1. Выберите **Отправка и скачивание файлов**, а затем **Отправить**. Отправьте *azuredeploy.json* и *azuredeploy.parameters.json* в Cloud Shell. После отправки вы можете использовать команды `ls` и `cat`, чтобы проверить отправку файла.

1. а затем выполните следующий сценарий PowerShell для его развертывания.

    ```azurepowershell
    $projectName = Read-Host -Prompt "Enter the same project name that is used for creating the key vault"
    $location = Read-Host -Prompt "Enter the same location that is used for creating the key vault (i.e. centralus)"
    $resourceGroupName = "${projectName}rg"

    New-AzResourceGroupDeployment `
        -ResourceGroupName $resourceGroupName `
        -TemplateFile "$HOME/azuredeploy.json" `
        -TemplateParameterFile "$HOME/azuredeploy.parameters.json"

    Write-Host "Press [ENTER] to continue ..."
    ```

    При развертывании шаблона используйте ту же группу ресурсов, что и для хранилища ключей. Такой подход упрощает процесс чистки ресурсов, так как вам необходимо удалить одну группу вместо двух.

## <a name="validate-the-deployment"></a>Проверка развертывания

После успешного развертывания виртуальной машины проверьте учетные данные входа в систему с помощью пароля, сохраненного в хранилище ключей.

1. Откройте [портал Azure](https://portal.azure.com).

1. Выберите **Группы ресурсов** >  **\<*YourResourceGroupName*>**  > **simpleWinVM**.
1. Выберите **Подключение** в верхней части страницы.
1. Выберите **Скачать RDP-файл** и следуйте инструкциям, чтобы войти в виртуальную машину с помощью пароля, сохраненного в хранилище ключей.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если ресурсы Azure больше не нужны, очистите их, удалив группу ресурсов.

```azurepowershell-interactive
$projectName = Read-Host -Prompt "Enter the same project name that is used for creating the key vault"
$resourceGroupName = "${projectName}rg"

Remove-AzResourceGroup -Name $resourceGroupName

Write-Host "Press [ENTER] to continue ..."
```

## <a name="next-steps"></a>Дальнейшие действия

С помощью этого учебника вы извлекли секрет из хранилища ключей Azure, а затем использовали этот секрет при развертывании шаблона. Чтобы узнать, как использовать расширения виртуальной машины для задач, выполняемых после развертывания, перейдите к следующей статье:

> [!div class="nextstepaction"]
> [Развертывание расширений виртуальной машины](./template-tutorial-deploy-vm-extensions.md)
