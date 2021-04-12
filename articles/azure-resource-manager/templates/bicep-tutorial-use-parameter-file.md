---
title: Учебник по использованию файла параметров для развертывания Bicep-файла Azure Resource Manager
description: Использование файлов параметров, которые содержат значения, используемые для развертывания Bicep-файла.
author: mumian
ms.date: 03/10/2021
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: ca3a73cde9549bfcdfd47bc4f1955904fac69d1c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102632362"
---
# <a name="tutorial-use-parameter-files-to-deploy-azure-resource-manager-bicep-file"></a>Учебник по использованию файлов параметров для развертывания Bicep-файла Azure Resource Manager

В этом учебнике вы узнаете, как использовать [файлы параметров](parameter-files.md) для хранения значений, передаваемых во время развертывания. В предыдущих учебниках вы использовали встроенные параметры с помощью команды развертывания. Этот подход сработал для тестирования Bicep-файла, но при автоматизации развертываний может быть проще передать набор значений для вашей среды. Файлы параметров упрощают упаковку значений параметров для конкретной среды. Вы будете использовать тот же файл параметров JSON, что и для развертывания шаблона JSON. В этом учебнике вы создадите файлы параметров для среды разработки и рабочей среды. Это занимает около **12 минут**.

[!INCLUDE [Bicep preview](../../../includes/resource-manager-bicep-preview.md)]

## <a name="prerequisites"></a>Предварительные требования

Советуем выполнить инструкции из [руководства о тегах](bicep-tutorial-add-tags.md), но это необязательно.

Вам понадобится Visual Studio Code с расширением Bicep и либо Azure PowerShell, либо Azure CLI. Дополнительные сведения см. в разделе о [средствах Bicep](bicep-tutorial-create-first-bicep.md#get-tools).

## <a name="review-bicep-file"></a>Проверка Bicep-файла

Bicep-файл содержит множество параметров, которые можно указать во время развертывания. В конце предыдущего учебника Bicep-файл выглядел следующим образом:

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-tags/azuredeploy.bicep":::

Этот Bicep-файл хорошо работает, но теперь вам нужно упростить управление параметрами, передаваемыми для Bicep-файла.

## <a name="add-parameter-files"></a>Добавление файлов параметров

Файлы параметров — это JSON-файлы со структурой, аналогичной шаблонам JSON. В файле укажите значения параметров, которые необходимо передать во время развертывания.

В файле параметров укажите значения для параметров в Bicep-файле. Имя каждого параметра в файле параметров должно совпадать с именем параметра в Bicep-файле. В имени не учитывается регистр. Однако чтобы быстро просматривать совпадающие значения, рекомендуется использовать Bicep-файл в соответствии с регистром.

Вам не нужно указывать значения для каждого параметра. Если для параметра указано значение по умолчанию, это значение используется во время развертывания. Если для параметра не указано значение по умолчанию и он не указан в файле параметров, вам будет предложено указать значение во время развертывания.

Вы не можете указать имя параметра в файле параметров, которое не соответствует имени параметра в Bicep-файле. Если будут указаны неизвестные параметры, вы получите сообщение об ошибке.

В Visual Studio Code создайте файл со следующим содержимым. Сохраните файл с именем _azuredeploy.parameters.dev.json_.

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-tags/azuredeploy.parameters.dev.json":::

Это файл параметров для среды разработки. Обратите внимание, что он использует **Standard_LRS** в качестве учетной записи хранения, именует ресурсы с использованием префикса **dev** и задает для тега `Environment` значение **Dev**.

Опять же, создайте файл со следующим содержимым. Сохраните файл с именем _azuredeploy.parameters.prod.json_.

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-tags/azuredeploy.parameters.prod.json":::

Это файл параметров для рабочей среды. Обратите внимание, что он использует **Standard_GRS** в качестве учетной записи хранения, именует ресурсы с применением префикса **contoso** и задает для тега `Environment`значение **Production**. В реальной рабочей среде также необходимо использовать службу приложений с номером SKU, отличным от "Бесплатный", но мы продолжим использовать его в рамках этого учебника.

## <a name="deploy-bicep-file"></a>Развертывание файла Bicep

Для развертывания Bicep-файла используйте либо Azure CLI, либо Azure PowerShell.

Используя инструкции из этого учебника, мы создадим две группы ресурсов: для среды разработки и для рабочей среды.

Для переменных шаблона и параметров замените `{path-to-the-bicep-file}`, `{path-to-azuredeploy.parameters.dev.json}`, `{path-to-azuredeploy.parameters.prod.json}` и фигурные скобки `{}` путями к Bicep-файлу и файлам параметров.

Сначала мы выполним развертывание в среде разработки.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Чтобы выполнить этот командлет развертывания, необходимо иметь [последнюю версию](/powershell/azure/install-az-ps) Azure PowerShell.

```azurepowershell
$bicepFile = "{path-to-the-bicep-file}"
$parameterFile="{path-to-azuredeploy.parameters.dev.json}"
New-AzResourceGroup `
  -Name myResourceGroupDev `
  -Location "East US"
New-AzResourceGroupDeployment `
  -Name devenvironment `
  -ResourceGroupName myResourceGroupDev `
  -TemplateFile $bicepFile `
  -TemplateParameterFile $parameterFile
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы выполнить эту команду развертывания, необходимо иметь [последнюю версию](/cli/azure/install-azure-cli) Azure CLI.

```azurecli
bicepFile="{path-to-the-bicep-file}"
devParameterFile="{path-to-azuredeploy.parameters.dev.json}"
az group create \
  --name myResourceGroupDev \
  --location "East US"
az deployment group create \
  --name devenvironment \
  --resource-group myResourceGroupDev \
  --template-file $bicepFile \
  --parameters $devParameterFile
```

---

Теперь мы выполним развертывание в рабочей среде.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$parameterFile="{path-to-azuredeploy.parameters.prod.json}"
New-AzResourceGroup `
  -Name myResourceGroupProd `
  -Location "West US"
New-AzResourceGroupDeployment `
  -Name prodenvironment `
  -ResourceGroupName myResourceGroupProd `
  -TemplateFile $templateFile `
  -TemplateParameterFile $parameterFile
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
prodParameterFile="{path-to-azuredeploy.parameters.prod.json}"
az group create \
  --name myResourceGroupProd \
  --location "West US"
az deployment group create \
  --name prodenvironment \
  --resource-group myResourceGroupProd \
  --template-file $templateFile \
  --parameters $prodParameterFile
```

---

> [!NOTE]
> Если развертывание завершилось сбоем, используйте параметр `verbose`, чтобы получить сведения о создаваемых ресурсах. Используйте параметр `debug`, чтобы получить дополнительные сведения для отладки.

## <a name="verify-deployment"></a>Проверка развертывания

Чтобы проверить развертывание, просмотрите группы ресурсов на портале Azure.

1. Войдите на [портал Azure](https://portal.azure.com).
1. В меню слева выберите **Группы ресурсов**.
1. Вы увидите две новые группы ресурсов, развернутые при работе с этим учебником.
1. Выберите любую группу ресурсов и просмотрите развернутые ресурсы. Обратите внимание, что они соответствуют значениям, указанным в файле параметров для этой среды.

## <a name="clean-up-resources"></a>Очистка ресурсов

1. На портале Azure в меню слева выберите **Группа ресурсов**.
2. В поле **Фильтровать по имени** введите имя группы ресурсов.
3. Выберите имя группы ресурсов.
4. В главном меню выберите **Удалить группу ресурсов**.

## <a name="next-steps"></a>Дальнейшие действия

В рамках работы с этим учебником используется файл параметров для развертывания Bicep-файла. Из следующего учебника вы узнаете, как разделить Bicep-файлы.

> [!div class="nextstepaction"]
> [Добавление модулей](./bicep-tutorial-add-modules.md)
