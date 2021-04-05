---
title: Учебник. Развертывание шаблона с помощью файла параметров
description: Используйте файлы параметров, которые содержат значения, используемые для развертывания шаблона Azure Resource Manager (шаблона ARM).
author: mumian
ms.date: 09/10/2020
ms.topic: tutorial
ms.author: jgao
ms.custom: devx-track-azurecli
ms.openlocfilehash: b6257161017afc9dab692c43fcc64e5d961a90ba
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97368432"
---
# <a name="tutorial-use-parameter-files-to-deploy-your-arm-template"></a>Руководство по Использование файлов параметров для развертывания шаблона ARM

В этом учебнике вы узнаете, как использовать [файлы параметров](parameter-files.md) для хранения значений, передаваемых во время развертывания. В предыдущих учебниках вы использовали встроенные параметры с помощью команды развертывания. Этот подход использовался для тестирования шаблона Azure Resource Manager (шаблон ARM), но при автоматизации развертываний может быть проще передать набор значений для вашей среды. Файлы параметров упрощают упаковку значений параметров для конкретной среды. В этом учебнике вы создадите файлы параметров для среды разработки и рабочей среды. Это занимает около **12 минут**.

## <a name="prerequisites"></a>Предварительные требования

Советуем выполнить инструкции из [руководства о тегах](template-tutorial-add-tags.md), но это необязательно.

Вам понадобится Visual Studio Code с расширением инструментов Resource Manager и либо Azure PowerShell, либо Azure CLI. Дополнительные сведения см. в разделе об [инструментах шаблона](template-tutorial-create-first-template.md#get-tools).

## <a name="review-template"></a>Проверка шаблона

Шаблон содержит множество параметров, которые можно указать во время развертывания. В конце предыдущего учебника шаблон выглядел следующим образом:

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-tags/azuredeploy.json":::

Этот шаблон хорошо работает, но теперь вам нужно упростить управление параметрами, передаваемыми для шаблона.

## <a name="add-parameter-files"></a>Добавление файлов параметров

Файлы параметров — это JSON-файлы со структурой, аналогичной шаблону. В файле укажите значения параметров, которые необходимо передать во время развертывания.

В файле параметров укажите значения для параметров в шаблоне. Имя каждого параметра в файле параметров должно совпадать с именем параметра в шаблоне. В имени не учитывается регистр. Однако чтобы быстро просматривать совпадающие значения, рекомендуется использовать шаблон в соответствии с регистром.

Вам не нужно указывать значения для каждого параметра. Если для параметра указано значение по умолчанию, это значение используется во время развертывания. Если для параметра не указано значение по умолчанию и он не указан в файле параметров, вам будет предложено указать значение во время развертывания.

Вы не можете указать имя параметра в файле параметров, которое не соответствует имени параметра в шаблоне. Если будут указаны неизвестные параметры, вы получите сообщение об ошибке.

В Visual Studio Code создайте файл со следующим содержимым. Сохраните файл с именем _azuredeploy.parameters.dev.json_.

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-tags/azuredeploy.parameters.dev.json":::

Это файл параметров для среды разработки. Обратите внимание, что он использует **Standard_LRS** в качестве учетной записи хранения, именует ресурсы с использованием префикса **dev** и задает для тега `Environment` значение **Dev**.

Опять же, создайте файл со следующим содержимым. Сохраните файл с именем _azuredeploy.parameters.prod.json_.

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-tags/azuredeploy.parameters.prod.json":::

Это файл параметров для рабочей среды. Обратите внимание, что он использует **Standard_GRS** в качестве учетной записи хранения, именует ресурсы с применением префикса **contoso** и задает для тега `Environment`значение **Production**. В реальной рабочей среде также необходимо использовать службу приложений с номером SKU, отличным от "Бесплатный", но мы продолжим использовать его в рамках этого учебника.

## <a name="deploy-template"></a>Развертывание шаблона

Для развертывания шаблона используйте либо Azure CLI, либо Azure PowerShell.

Для окончательной проверки шаблона создадим две группы ресурсов: для среды разработки и для рабочей среды.

Для переменных шаблона и параметров замените `{path-to-the-template-file}`, `{path-to-azuredeploy.parameters.dev.json}`, `{path-to-azuredeploy.parameters.prod.json}` и фигурные скобки `{}` путями к файлам шаблона и параметров.

Сначала мы выполним развертывание в среде разработки.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$templateFile = "{path-to-the-template-file}"
$parameterFile="{path-to-azuredeploy.parameters.dev.json}"
New-AzResourceGroup `
  -Name myResourceGroupDev `
  -Location "East US"
New-AzResourceGroupDeployment `
  -Name devenvironment `
  -ResourceGroupName myResourceGroupDev `
  -TemplateFile $templateFile `
  -TemplateParameterFile $parameterFile
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы выполнить эту команду развертывания, необходимо иметь [последнюю версию](/cli/azure/install-azure-cli) Azure CLI.

```azurecli
templateFile="{path-to-the-template-file}"
devParameterFile="{path-to-azuredeploy.parameters.dev.json}"
az group create \
  --name myResourceGroupDev \
  --location "East US"
az deployment group create \
  --name devenvironment \
  --resource-group myResourceGroupDev \
  --template-file $templateFile \
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
2. В поле **Фильтровать по имени** введите имя группы ресурсов. Если вы поработали со всеми учебниками в этой серии, у вас есть три группы ресурсов для удаления — **myResourceGroup**, **myResourceGroupDev** и **myResourceGroupProd**.
3. Выберите имя группы ресурсов.
4. В главном меню выберите **Удалить группу ресурсов**.

## <a name="next-steps"></a>Дальнейшие действия

Поздравляем, вы завершили ознакомление с развертыванием шаблонов в Azure. Оставьте комментарии и предложения для нас в разделе отзывов. Спасибо!

В следующей серии руководств подробно рассматривается развертывание шаблонов.

> [!div class="nextstepaction"]
> [Развертывание шаблона в локальной среде](./deployment-tutorial-local-template.md)
