---
title: Использование справочника по шаблонам
description: Сведения о том, как создать шаблон с помощью справочника по шаблонам Azure Resource Manager.
author: mumian
ms.date: 04/23/2020
ms.topic: tutorial
ms.author: jgao
ms.custom: seodec18
ms.openlocfilehash: a44852fb2f491dd949b58217eca3e4f3e392cf17
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97584142"
---
# <a name="tutorial-utilize-the-arm-template-reference"></a>Руководство по использованию справочной документации по шаблонам Resource Manager

Узнайте, как найти сведения о схеме шаблона и использовать их для создания шаблонов Azure Resource Manager.

При работе с этим руководством вы будете использовать базовый шаблон из шаблонов быстрого запуска Azure. Вы настроите шаблон с использованием соответствующей справочной документации.

![Учетная запись хранения справочных материалов для развертывания шаблона Resource Manager](./media/template-tutorial-use-template-reference/resource-manager-template-tutorial-deploy-storage-account.png)

В рамках этого руководства рассматриваются следующие задачи:

> [!div class="checklist"]
> * Открытие шаблона быстрого запуска
> * Общие сведения о шаблоне
> * Поиск ссылки на шаблон
> * Изменение шаблона
> * Развертывание шаблона

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

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
1. Выберите **Файл** > **Сохранить как**, чтобы сохранить файл в качестве _azuredeploy.json_ на локальном компьютере.

## <a name="understand-the-schema"></a>Изучение схемы

1. В VS Code сверните шаблон до корневого уровня. Имеется простейшая структура со следующими элементами.

    ![Простейшая структура шаблона Resource Manager](./media/template-tutorial-use-template-reference/resource-manager-template-simplest-structure.png)

    * Параметр `$schema` указывает расположение файла схемы JSON, который описывает версию языка шаблона.
    * Параметр `contentVersion` задает этому элементу любое значение, чтобы документировать важные изменения в шаблоне.
    * Параметр `parameters` указывает значения, которые предоставляются при выполнении развертывания для настройки развертывания ресурсов.
    * Параметр `variables` указывает значения, используемые в виде фрагментов JSON в шаблоне для упрощения выражений на языке шаблона.
    * Параметр `resources` указывает типы ресурсов, которые развертываются или обновляются в группе ресурсов.
    * Параметр `outputs` указывает значения, возвращаемые после развертывания.

1. Разверните раздел `resources`. Вы увидите определенный ресурс `Microsoft.Storage/storageAccounts`. В качестве имени SKU используется значение параметра. Имя параметра — `storageAccountType`.

    ![Определение учетной записи хранения в шаблоне Resource Manager](./media/template-tutorial-use-template-reference/resource-manager-template-storage-resource.png)

1. Разверните раздел `parameters`, чтобы увидеть, как определен `storageAccountType`. Параметр имеет четыре допустимых значения. Вы увидите остальные допустимые значения, а потом проверите определение параметра.

    ![Номера SKU ресурсов учетной записи хранения в шаблоне Resource Manager](./media/template-tutorial-use-template-reference/resource-manager-template-storage-resources-skus-old.png)

## <a name="find-the-template-reference"></a>Поиск ссылки на шаблон

1. Перейти к [справочнику по шаблонам Azure](/azure/templates/).
1. В поле **Фильтр по названию** введите **учетные записи хранения** и выберите первый пункт **Учетные записи хранения** в разделе **Справочные материалы > Хранилище**.

    ![Учетная запись хранения ссылки на шаблон Resource Manager](./media/template-tutorial-use-template-reference/resource-manager-template-resources-reference-storage-accounts.png)

    У поставщика ресурсов обычно бывает несколько версий API:

    ![Справочные материалы по шаблону Resource Manager: версии учетной записи хранения](./media/template-tutorial-use-template-reference/resource-manager-template-resources-reference-storage-accounts-versions.png)

1. Выберите элемент **Все ресурсы** в разделе **Хранилище** на панели слева. На этой странице перечислены типы ресурсов и версии поставщика ресурсов хранилища. Рекомендуется использовать последние версии API для типов ресурсов, определенных в шаблоне.

    ![Справочные материалы по шаблону Resource Manager: версии типов учетной записи хранения](./media/template-tutorial-use-template-reference/resource-manager-template-resources-reference-storage-accounts-types-versions.png)

1. Выберите последнюю версию типа ресурса `storageAccount`. Номер последней версии на момент написании этой статьи — **2019-06-01**. Убедитесь, что эта версия соответствует версии, используемой для ресурса учетной записи хранения в шаблоне. При обновлении версии API убедитесь, что определение ресурса соответствует, указанному в справочнике по шаблонам.

1. На этой странице приведены сведения о типе ресурса storageAccount. Например, на ней перечислены допустимые значения для объекта **SKU**. Существует больше номеров SKU, чем указано в шаблоне быстрого запуска, который вы открыли ранее. Вы можете настроить шаблон быстрого запуска, включив в него все доступные типы хранилища.

    ![Справочные материалы по шаблону Resource Manager: номера SKU учетной записи хранения](./media/template-tutorial-use-template-reference/resource-manager-template-resources-reference-storage-accounts-skus.png)

## <a name="edit-the-template"></a>Изменение шаблона

В Visual Studio Code добавьте дополнительные типы учетной записи хранения, как показано на следующем снимке экрана:

![Ресурсы учетной записи хранения в шаблоне Resource Manager](./media/template-tutorial-use-template-reference/resource-manager-template-storage-resources-skus.png)

## <a name="deploy-the-template"></a>Развертывание шаблона

1. Войдите в [Azure Cloud Shell](https://shell.azure.com).

1. В левом верхнем углу выберите используемую среду — **PowerShell** или **Bash** (для CLI).  После переключения желательно перезагрузить оболочку.

    ![Файл отправки Cloud Shell на портале Azure](./media/template-tutorial-use-template-reference/azure-portal-cloud-shell-upload-file.png)

1. Выберите **Отправка и скачивание файлов**, а затем **Отправить**. См. предыдущий снимок экрана. Выберите файл, сохраненный ранее. После отправки вы можете использовать команды `ls` и `cat`, чтобы проверить отправку файла.

1. Выполните следующие команды в Cloud Shell. Выберите вкладку, чтобы отобразить код PowerShell или код CLI.

   При развертывании шаблона укажите параметр `storageAccountType` с новым значением, например **Standard_RAGRS**. Если вы использовали исходный шаблон быстрого запуска, при развертывании произойдет сбой, так как **Standard_RAGRS** не является допустимым значением.

    # <a name="cli"></a>[CLI](#tab/CLI)

    ```azurecli
    echo "Enter a project name that is used to generate resource group name:" &&
    read projectName &&
    echo "Enter the location (i.e. centralus):" &&
    read location &&
    resourceGroupName="${projectName}rg" &&
    az group create --name $resourceGroupName --location "$location" &&
    az deployment group create --resource-group $resourceGroupName --template-file "$HOME/azuredeploy.json" --parameters storageAccountType='Standard_RAGRS'
    ```

    # <a name="powershell"></a>[PowerShell](#tab/PowerShell)

    ```azurepowershell
    $projectName = Read-Host -Prompt "Enter a project name that is used to generate resource group name"
    $location = Read-Host -Prompt "Enter the location (i.e. centralus)"
    $resourceGroupName = "${projectName}rg"

    New-AzResourceGroup -Name $resourceGroupName -Location "$location"
    New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateFile "$HOME/azuredeploy.json" -storageAccountType "Standard_RAGRS"
    ```

    ---

## <a name="clean-up-resources"></a>Очистка ресурсов

Если ресурсы Azure больше не нужны, их можно удалить. Для этого необходимо удалить группу ресурсов.

1. На портале Azure в меню слева выберите **Группа ресурсов**.
1. В поле **Фильтровать по имени** введите имя группы ресурсов.
1. Выберите имя группы ресурсов.  В группе ресурсов должно появится шесть ресурсов.
1. В главном меню выберите **Удалить группу ресурсов**.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве описано, как использовать справочник по шаблонам для настройки существующего шаблона. Узнайте, как создать несколько экземпляров учетной записи хранения

> [!div class="nextstepaction"]
> [Развертывание нескольких экземпляров](./template-tutorial-create-multiple-instances.md)
