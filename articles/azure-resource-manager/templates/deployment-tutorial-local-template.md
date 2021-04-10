---
title: Учебник. Развертывание шаблона Azure Resource Manager
description: Узнайте, как развернуть шаблон Azure Resource Manager (шаблон ARM) на локальном компьютере.
ms.date: 02/10/2021
ms.topic: tutorial
ms.author: jgao
ms.custom: ''
ms.openlocfilehash: d8d54acfa345994edcc401170e70495b3826bfdf
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100384237"
---
# <a name="tutorial-deploy-a-local-arm-template"></a>Руководство по развертыванию шаблона ARM в локальной среде

Узнайте, как развернуть шаблон Azure Resource Manager (шаблон ARM) на локальном компьютере. Выполнение занимает около **8 минут**.

Этот учебник первый в серии. В процессе работы над серией вы модулируете шаблон, создавая связанный шаблон, сохраняете связанный шаблон в учетной записи хранения и защищаете связанный шаблон с помощью маркера SAS, кроме того вы узнаете, как создать конвейер DevOps для развертывания шаблонов. Эта серия посвящена развертыванию шаблона. Если вы хотите изучить разработку шаблонов, ознакомьтесь с [Руководством для начинающих](./template-tutorial-create-first-template.md).

## <a name="get-tools"></a>Получение средств

Убедитесь, что у вас есть средства, необходимые для развертывания шаблонов.

### <a name="command-line-deployment"></a>Развертывание из командной строки

Для развертывания шаблона используйте либо Azure CLI, либо Azure PowerShell. Ознакомьтесь с инструкциями по установке:

- [Установка Azure PowerShell](/powershell/azure/install-az-ps)
- [Установка Azure CLI в Windows](/cli/azure/install-azure-cli-windows)
- [Установка Azure CLI в Linux](/cli/azure/install-azure-cli-linux)
- [Установка Azure CLI в macOS](/cli/azure/install-azure-cli-macos)

После установки Azure PowerShell или Azure CLI убедитесь, что вы вошли в систему в первый раз. Дополнительные сведения см. в разделах для [PowerShell](/powershell/azure/install-az-ps#sign-in) или [Azure CLI](/cli/azure/get-started-with-azure-cli#sign-in).

### <a name="editor-optional"></a>Редактор (необязательно)

Шаблоны — это файлы JSON. Для просмотра и изменения шаблонов нужен редактор JSON. Советуем использовать Visual Studio Code с расширением средств Resource Manager. Если необходимо установить эти средства, см. статью [Краткое руководство. Создание шаблонов ARM с помощью Visual Studio Code](quickstart-create-templates-use-visual-studio-code.md).

## <a name="review-template"></a>Проверка шаблона

Шаблон развертывает учетную запись хранения, план службы приложений и веб-приложение. Если вы заинтересованы в создании шаблона быстрого запуска, изучите [этот учебник](template-tutorial-quickstart-template.md). Однако это не является обязательным для работы с данным руководством.

:::code language="json" source="~/resourcemanager-templates/get-started-deployment/local-template/azuredeploy.json":::

> [!IMPORTANT]
> Имя учетной записи хранения должно содержать от 3 до 24 символов и состоять только из цифр и букв нижнего регистра. Имя должно быть уникальным. В шаблоне имя учетной записи хранения — это имя проекта с добавлением **store**, которое должно содержать от 3 до 11 символов. Поэтому имя проекта должно соответствовать требованиям к имени учетной записи хранения и быть длиной менее 11 символов.

Сохраните копию шаблона на локальном компьютере с расширением _JSON_, например _azuredeploy.json_. Этот шаблон развертывается далее в этом руководстве.

## <a name="sign-in-to-azure"></a>Вход в Azure

Чтобы начать работу с Azure PowerShell или Azure CLI для развертывания шаблона, выполните вход, используя данные своей учетной записи в Azure.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
Connect-AzAccount
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az login
```

---

Если у вас несколько подписок Azure, выберите ту, которую хотите использовать. Замените `[SubscriptionID/SubscriptionName]` и квадратные скобки `[]` сведениями о подписке:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
Set-AzContext [SubscriptionID/SubscriptionName]
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az account set --subscription [SubscriptionID/SubscriptionName]
```

---

## <a name="create-resource-group"></a>Создать группу ресурсов

При развертывании шаблона необходимо указать группу ресурсов, которая будет содержать развернутые ресурсы. Перед выполнением команды развертывания создайте группу ресурсов с помощью Azure CLI или Azure PowerShell. Выберите вкладки в следующем разделе кода, чтобы выбрать между Azure PowerShell и Azure CLI. Примеры интерфейса командной строки в этой статье написаны для оболочки bash.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$projectName = Read-Host -Prompt "Enter a project name that is used to generate resource and resource group names"
$resourceGroupName = "${projectName}rg"

New-AzResourceGroup `
  -Name $resourceGroupName `
  -Location "Central US"
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
echo "Enter a project name that is used to generate resource and resource group names:"
read projectName
resourceGroupName="${projectName}rg"

az group create \
  --name $resourceGroupName \
  --location "Central US"
```

---

## <a name="deploy-template"></a>Развертывание шаблона

Используйте один или оба варианта развертывания для развертывания шаблона.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$projectName = Read-Host -Prompt "Enter the same project name"
$templateFile = Read-Host -Prompt "Enter the template file path and file name"
$resourceGroupName = "${projectName}rg"

New-AzResourceGroupDeployment `
  -Name DeployLocalTemplate `
  -ResourceGroupName $resourceGroupName `
  -TemplateFile $templateFile `
  -projectName $projectName `
  -verbose
```

Дополнительные сведения о развертывании шаблона с помощью Azure PowerShell см. в статье [Развертывание ресурсов с использованием шаблонов ARM и Azure PowerShell](./deploy-powershell.md).

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
echo "Enter the same project name:"
read projectName
echo "Enter the template file path and file name:"
read templateFile
resourceGroupName="${projectName}rg"

az deployment group create \
  --name DeployLocalTemplate \
  --resource-group $resourceGroupName \
  --template-file $templateFile \
  --parameters projectName=$projectName \
  --verbose
```

Дополнительные сведения о развертывании шаблона с помощью Azure CLI см. в статье [Развертывание ресурсов с использованием шаблонов ARM и Azure CLI](./deploy-cli.md).

---

## <a name="clean-up-resources"></a>Очистка ресурсов

Очистите развернутые ресурсы, удалив группу ресурсов.

1. На портале Azure в меню слева выберите **Группа ресурсов**.
2. В поле **Фильтровать по имени** введите имя группы ресурсов.
3. Выберите имя группы ресурсов.
4. В главном меню выберите **Удалить группу ресурсов**.

## <a name="next-steps"></a>Дальнейшие действия

Вы узнали как выполнять развертывание локального шаблона. В следующем руководстве вы разделите шаблон на основной шаблон и связанный, а также узнаете, как сохранить и защитить связанный шаблон.

> [!div class="nextstepaction"]
> [Развертывание связанного шаблона](./deployment-tutorial-linked-template.md)
