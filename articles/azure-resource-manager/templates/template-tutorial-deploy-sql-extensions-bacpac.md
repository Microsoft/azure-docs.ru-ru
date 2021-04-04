---
title: Импорт BACPAC-файлов SQL с помощью шаблонов
description: Сведения о том, как использовать расширение Базы данных SQL Azure для импорта BACPAC-файлов SQL с помощью шаблонов Resource Manager.
author: mumian
ms.date: 12/09/2019
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: 1bd9f7408baf40791c31626ea9e87a73c65b999c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97964003"
---
# <a name="tutorial-import-sql-bacpac-files-with-arm-templates"></a>Руководство по Импорт BACPAC-файлов SQL с помощью шаблонов ARM

Узнайте, как использовать расширение Базы данных SQL Azure для импорта [BACPAC](/sql/relational-databases/data-tier-applications/data-tier-applications#bacpac)-файла с помощью шаблонов Azure Resource Manager (шаблонов ARM). Артефакты развертывания — это любые файлы, которые помимо файлов основного шаблона необходимы для выполнения развертывания. BACPAC-файл является артефактом.

В этом учебнике вы создадите шаблон для развертывания [логического сервера SQL Server](../../azure-sql/database/logical-servers.md), отдельной базы данных и импорта BACPAC-файла. См. сведения о развертывании расширений виртуальной машины Azure с помощью шаблонов ARM в руководстве по [ развертыванию расширений виртуальной машины с помощью шаблонов ARM](./template-tutorial-deploy-vm-extensions.md).

В рамках этого руководства рассматриваются следующие задачи:

> [!div class="checklist"]
>
> * подготовка BACPAC-файла;
> * открытие шаблона быстрого запуска;
> * изменение шаблона;
> * Разверните шаблон.
> * проверка развертывания.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этой статьей необходимо иметь следующее.

* Visual Studio Code с расширением средств Resource Manager. См. [Краткое руководство. Создание шаблонов ARM с помощью Visual Studio Code](./quickstart-create-templates-use-visual-studio-code.md).
* Для повышения уровня безопасности используйте пароль, созданный для учетной записи администратора сервера. Ниже приведен пример создания пароля.

    ```console
    openssl rand -base64 32
    ```

    Для защиты криптографических ключей и других секретов используйте Azure Key Vault. Дополнительные сведения см. в статье [Учебник. Интеграция с Azure Key Vault при развертывании шаблона ARM](./template-tutorial-use-key-vault.md). Мы также рекомендуем обновлять пароль каждые три месяца.

## <a name="prepare-a-bacpac-file"></a>Подготовка BACPAC-файла

BACPAC-файл доступен на сайте [GitHub](https://github.com/Azure/azure-docs-json-samples/raw/master/tutorial-sql-extension/SQLDatabaseExtension.bacpac). См. сведения о создании собственного файла в статье [Экспорт в BACPAC-файл — База данных SQL Azure и Управляемый экземпляр SQL Azure](../../azure-sql/database/database-export.md). Если вы решили опубликовать файл в своем расположении, необходимо обновить шаблон позже в этом руководстве.

BACPAC-файл нужно сохранить в учетной записи службы хранилища Azure, прежде чем его можно будет импортировать с помощью шаблона ARM. Следующий скрипт PowerShell подготавливает BACPAC-файл:

* Скачивание BACPAC-файла.
* Создание учетной записи хранения Azure.
* Создание контейнера больших двоичных объектов в учетной записи хранения.
* Передача BACPAC-файла в этот контейнер.
* Отображение ключа учетной записи хранения и URL-адреса большого двоичного объекта.

1. Выберите **Попробовать**, чтобы открыть Azure Cloud Shell. Затем вставьте приведенный ниже скрипт PowerShell в окно оболочки.

    ```azurepowershell-interactive
    $projectName = Read-Host -Prompt "Enter a project name that is used to generate Azure resource names"
    $location = Read-Host -Prompt "Enter the location (i.e. centralus)"

    $resourceGroupName = "${projectName}rg"
    $storageAccountName = "${projectName}store"
    $containerName = "bacpacfiles"
    $bacpacFileName = "SQLDatabaseExtension.bacpac"
    $bacpacUrl = "https://github.com/Azure/azure-docs-json-samples/raw/master/tutorial-sql-extension/SQLDatabaseExtension.bacpac"

    # Download the bacpac file
    Invoke-WebRequest -Uri $bacpacUrl -OutFile "$HOME/$bacpacFileName"

    # Create a resource group
    New-AzResourceGroup -Name $resourceGroupName -Location $location

    # Create a storage account
    $storageAccount = New-AzStorageAccount -ResourceGroupName $resourceGroupName `
                                           -Name $storageAccountName `
                                           -SkuName Standard_LRS `
                                           -Location $location
    $storageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName `
                                                  -Name $storageAccountName).Value[0]

    # Create a container
    New-AzStorageContainer -Name $containerName -Context $storageAccount.Context

    # Upload the BACPAC file to the container
    Set-AzStorageBlobContent -File $HOME/$bacpacFileName `
                             -Container $containerName `
                             -Blob $bacpacFileName `
                             -Context $storageAccount.Context

    Write-Host "The storage account key is $storageAccountKey"
    Write-Host "The BACPAC file URL is https://$storageAccountName.blob.core.windows.net/$containerName/$bacpacFileName"
    Write-Host "Press [ENTER] to continue ..."
    ```

1. Запишите ключ учетной записи хранения и URL-адрес BACPAC-файла. Эти значения понадобятся при развертывании шаблона.

## <a name="open-a-quickstart-template"></a>Открытие шаблона быстрого запуска

Используемый в этом учебнике шаблон доступен на сайте [GitHub](https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/tutorial-sql-extension/azuredeploy.json).

1. В Visual Studio Code выберите **Файл** > **Открыть файл**.
1. Скопируйте приведенный ниже URL-адрес и вставьте его в поле **Имя файла**.

    ```url
    https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/tutorial-sql-extension/azuredeploy.json
    ```

1. Чтобы открыть файл, выберите **Открыть**.

    В шаблоне определено два ресурса:

   * `Microsoft.Sql/servers`. Ознакомьтесь со статьей о [справочнике по шаблонам](/azure/templates/microsoft.sql/servers).
   * `Microsoft.SQL.servers/databases`. Ознакомьтесь со статьей о [справочнике по шаблонам](/azure/templates/microsoft.sql/servers/databases).

        Прежде чем настраивать шаблон, рекомендуется получить основные сведения о нем.
1. Выберите **Файл** > **Сохранить как**, чтобы сохранить файл на локальный компьютер с именем *azuredeploy.json*.

## <a name="edit-the-template"></a>Изменение шаблона

1. Добавьте два дополнительных параметра в конец раздела `parameters`, чтобы задать ключ учетной записи хранения и URL-адрес BACPAC-файла.

    ```json
        "storageAccountKey": {
          "type":"string",
          "metadata":{
            "description": "Specifies the key of the storage account where the BACPAC file is stored."
          }
        },
        "bacpacUrl": {
          "type":"string",
          "metadata":{
            "description": "Specifies the URL of the BACPAC file."
          }
        }
    ```

    Добавьте запятую после закрывающей фигурной скобки (`}`) свойства `adminPassword`. Чтобы отформатировать JSON-файл из Visual Studio Code, нажмите клавиши SHIFT + ALT + F.

    Сведения о получении этих двух значений см. в разделе [Подготовка BACPAC-файла](#prepare-a-bacpac-file).

1. Добавьте в шаблон два дополнительных ресурса.

    * Чтобы разрешить расширению Базы данных SQL импортировать BACPAC-файлы, необходимо разрешить трафик из служб Azure. Добавьте следующее определение правила брандмауэра в определение сервера:

        ```json
        "resources": [
          {
            "type": "firewallrules",
            "apiVersion": "2015-05-01-preview",
            "name": "AllowAllAzureIps",
            "location": "[parameters('location')]",
            "dependsOn": [
              "[parameters('databaseServerName')]"
            ],
            "properties": {
              "startIpAddress": "0.0.0.0",
              "endIpAddress": "0.0.0.0"
            }
          }
        ]
        ```

        Шаблон выглядит следующим образом:

        ![Шаблон с определением правила брандмауэра](./media/template-tutorial-deploy-sql-extensions-bacpac/resource-manager-tutorial-deploy-sql-extensions-bacpac-firewall.png)

    * Добавьте ресурс расширения Базы данных SQL в определение базы данных с помощью следующего кода JSON:

        ```json
        "resources": [
          {
            "type": "extensions",
            "apiVersion": "2014-04-01",
            "name": "Import",
            "dependsOn": [
              "[resourceId('Microsoft.Sql/servers/databases', parameters('databaseServerName'), parameters('databaseName'))]"
            ],
            "properties": {
              "storageKeyType": "StorageAccessKey",
              "storageKey": "[parameters('storageAccountKey')]",
              "storageUri": "[parameters('bacpacUrl')]",
              "administratorLogin": "[parameters('adminUser')]",
              "administratorLoginPassword": "[parameters('adminPassword')]",
              "operationMode": "Import"
            }
          }
        ]
        ```

        Шаблон выглядит следующим образом:

        ![Шаблон с расширением базы данных SQL](./media/template-tutorial-deploy-sql-extensions-bacpac/resource-manager-tutorial-deploy-sql-extensions-bacpac.png)

        Сведения об определении ресурса см. в [справочнике о расширении Базы данных SQL](/azure/templates/microsoft.sql/servers/databases/extensions). Ниже приведены некоторые важные элементы.

        * `dependsOn`. Ресурс расширения должен быть создан после создания базы данных.
        * `storageKeyType`. Укажите тип ключа к хранилищу данных, который необходимо использовать. Возможные значения: `StorageAccessKey` и `SharedAccessKey`. Используйте `StorageAccessKey` в рамках изучения этого учебника.
        * `storageKey`. Укажите ключ для учетной записи хранения, в которой хранится BACPAC-файл. Если тип ключа к хранилищу данных — `SharedAccessKey`, он должен начинаться с "?".
        * `storageUri`. Укажите URL-адрес BACPAC-файла из учетной записи хранения.
        * `administratorLoginPassword`. Пароль администратора SQL. Используйте сгенерированный пароль. См. раздел [Предварительные требования](#prerequisites).

Завершенный шаблон выглядит следующим образом.

[!code-json[](~/resourcemanager-templates/tutorial-sql-extension/azuredeploy2.json?range=1-106&highlight=38-49,62-76,86-103)]

## <a name="deploy-the-template"></a>Развертывание шаблона

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Дополнительные сведения о процедуре развертывания шаблона см. в [этом разделе](./template-tutorial-create-templates-with-dependent-resources.md#deploy-the-template). Вместо этого используйте следующий сценарий развертывания PowerShell:

```azurepowershell-interactive
$projectName = Read-Host -Prompt "Enter the same project name that is used earlier"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$adminUsername = Read-Host -Prompt "Enter the SQL admin username"
$adminPassword = Read-Host -Prompt "Enter the admin password" -AsSecureString
$storageAccountKey = Read-Host -Prompt "Enter the storage account key"
$bacpacUrl = Read-Host -Prompt "Enter the URL of the BACPAC file"
$resourceGroupName = "${projectName}rg"

#New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -adminUser $adminUsername `
    -adminPassword $adminPassword `
    -TemplateFile "$HOME/azuredeploy.json" `
    -storageAccountKey $storageAccountKey `
    -bacpacUrl $bacpacUrl

Write-Host "Press [ENTER] to continue ..."
```

Рекомендуем применить имя проекта, использованное при подготовке BACPAC-файла, чтобы все ресурсы были сохранены в одной группе ресурсов. Таким образом проще управлять задачами ресурсов, такими как очистка ресурсов. При использовании одного имени проекта вы можете либо удалить команду `New-AzResourceGroup` из скрипта, либо выбрать вариант yes (y) или no (n) в качестве ответа на запрос обновления имеющейся группы ресурсов.

Используйте сгенерированный пароль. См. раздел [Предварительные требования](#prerequisites).

## <a name="verify-the-deployment"></a>Проверка развертывания

Чтобы получить доступ к серверу с клиентского компьютера, добавьте дополнительное правило брандмауэра. Дополнительные сведения см. в разделе о [создании правил брандмауэра для IP-адресов и управление ими](../../azure-sql/database/firewall-configure.md#create-and-manage-ip-firewall-rules).

На портале Azure выберите базу данных в только что развернутой группе ресурсов. Выберите **Редактор запросов (предварительная версия)** , а затем введите учетные данные администратора. Должны отобразиться две таблицы, импортированные в базу данных:

![Редактор запросов (предварительная версия)](./media/template-tutorial-deploy-sql-extensions-bacpac/resource-manager-tutorial-deploy-sql-extensions-bacpac-query-editor.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Если ресурсы Azure больше не нужны, их можно удалить. Для этого необходимо удалить группу ресурсов.

1. На портале Azure в меню слева выберите **Группа ресурсов**.
1. В поле **Фильтровать по имени** введите имя группы ресурсов.
1. Выберите имя группы ресурсов. В группе ресурсов должно появиться шесть ресурсов.
1. В главном меню выберите **Удалить группу ресурсов**.

## <a name="next-steps"></a>Дальнейшие действия

В этом учебнике вы развернули сервер, базу данных и импортированный BACPAC-файл. Инструкции по устранению неполадок при развертывании шаблонов см. в руководстве по

> [!div class="nextstepaction"]
> [устранению неполадок при развертывании шаблонов Resource Manager](./template-tutorial-troubleshoot.md).
