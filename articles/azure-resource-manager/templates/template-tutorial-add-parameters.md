---
title: Учебник. Добавление параметров в шаблон
description: Сведения о том, как добавить параметры в шаблон Resource Manager для его повторного использования.
author: mumian
ms.date: 03/31/2020
ms.topic: tutorial
ms.author: jgao
ms.custom: devx-track-azurecli
ms.openlocfilehash: e983f8499cbeaf400a8da6f48d7f6c8b75c4795a
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97107068"
---
# <a name="tutorial-add-parameters-to-your-arm-template"></a>Руководство по добавлению параметров в шаблон ARM

Из [предыдущего учебника](template-tutorial-add-resource.md) вы узнали, как добавлять учетную запись хранения в шаблон и развертывать ее. Из этого учебника вы узнаете, как улучшить шаблон Azure Resource Manager (ARM), добавив в него параметры. Для работы с этим учебником потребуется около **14 минут**.

## <a name="prerequisites"></a>Предварительные требования

Советуем выполнить инструкции из [учебника по ресурсам](template-tutorial-add-resource.md), но это необязательно.

Вам понадобится Visual Studio Code с расширением инструментов Resource Manager и либо Azure PowerShell, либо Azure CLI. Дополнительные сведения см. в разделе об [инструментах шаблона](template-tutorial-create-first-template.md#get-tools).

## <a name="review-template"></a>Проверка шаблона

В конце предыдущего учебника шаблон содержал следующий код JSON:

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-storage/azuredeploy.json":::

Возможно, вы заметили, что существует проблема с этим шаблоном. Имя учетной записи хранения жестко запрограммировано. Этот шаблон можно использовать только для развертывания одной и той же учетной записи хранения каждый раз. Чтобы развернуть учетную запись хранения с другим именем, вам нужно будет создать шаблон. Это, очевидно, не практичный способ автоматизации развертываний.

## <a name="make-template-reusable"></a>Изменение шаблона для повторного использования

Чтобы сделать шаблон многократно используемым, необходимо добавить параметр, который вы можете применять для передачи имени учетной записи хранения. Выделенный код JSON в следующем примере показывает изменения в шаблоне. Параметр `storageName` определяется как строка. Для максимальной длины установлено значение, равное 24 символам, чтобы не задавать слишком длинные имена.

Скопируйте весь файл и замените шаблон на его содержимое.

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-name/azuredeploy.json" range="1-26" highlight="4-10,15":::

## <a name="deploy-template"></a>Развертывание шаблона

Давайте развернем шаблон. В следующем примере шаблон развертывается с помощью Azure CLI или PowerShell. Обратите внимание: вы указываете имя учетной записи хранения в качестве одного из значений в команде развертывания. Для имени учетной записи хранения укажите то же имя, которое вы использовали в предыдущем учебнике.

Если вы еще не создали группу ресурсов, см. [этот раздел](template-tutorial-create-first-template.md#create-resource-group). В этом примере предполагается, что для переменной `templateFile` указан путь к файлу шаблона, как показано в [первом учебнике](template-tutorial-create-first-template.md#deploy-template).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name addnameparameter `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $templateFile `
  -storageName "{your-unique-name}"
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы выполнить эту команду развертывания, необходимо иметь [последнюю версию](/cli/azure/install-azure-cli) Azure CLI.

```azurecli
az deployment group create \
  --name addnameparameter \
  --resource-group myResourceGroup \
  --template-file $templateFile \
  --parameters storageName={your-unique-name}
```

---

## <a name="understand-resource-updates"></a>Концепция обновления ресурсов

В предыдущем разделе вы развернули учетную запись хранения с созданным ранее именем. Вас может интересовать, как ресурс повлиял на повторное развертывание.

Если ресурс уже существует и в свойствах не обнаружено изменений, никакие действия не предпринимаются. Если ресурс уже существует, а свойство изменилось, ресурс обновляется. Если ресурс не существует, он создается.

Этот способ обработки обновлений означает, что шаблон может включать в себя все ресурсы, необходимые для решения Azure. Вы можете безопасно повторно развернуть шаблон и знать, что ресурсы изменяются или создаются только при необходимости. Например, если вы добавили файлы в свою учетную запись хранения, вы можете повторно развернуть учетную запись хранения без потери этих файлов.

## <a name="customize-by-environment"></a>Настройка с учетом среды

Параметры позволяют настраивать развертывание путем предоставления значений, предназначенных для конкретной среды. Например, вы можете передавать различные значения в зависимости от того, развертываете ли вы среду для разработки, тестирования и производства.

Предыдущий шаблон всегда развертывал учетную запись хранения **Standard_LRS**. Возможно, вам потребуется гибкость для развертывания различных номеров SKU в зависимости от среды. В следующем примере показаны изменения для добавления параметра для номера SKU. Скопируйте весь файл и вставьте в шаблон.

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-sku/azuredeploy.json" range="1-40" highlight="10-23,32":::

Параметр `storageSKU` имеет значение по умолчанию. Оно используется, если во время развертывания не указано значение. В нем также содержится список допустимых значений. Они соответствуют значениям, требуемым для создания учетной записи хранения. Вы не хотите, чтобы пользователи шаблона передавали не рабочие номера SKU.

## <a name="redeploy-template"></a>Повторное развертывание шаблона

Теперь все готово к повторному развертыванию. Так как для SKU по умолчанию установлено значение **Standard_LRS**, вам не нужно указывать значение для этого параметра.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name addskuparameter `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $templateFile `
  -storageName "{your-unique-name}"
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az deployment group create \
  --name addskuparameter \
  --resource-group myResourceGroup \
  --template-file $templateFile \
  --parameters storageName={your-unique-name}
```

---

> [!NOTE]
> Если развертывание завершилось сбоем, используйте параметр `verbose`, чтобы получить сведения о создаваемых ресурсах. Используйте параметр `debug`, чтобы получить дополнительные сведения для отладки.

Чтобы убедиться в гибкости шаблона, давайте снова его развернем. На этот раз установите для параметра SKU значение **Standard_GRS**. Вы можете передать новое имя, чтобы создать другую учетную запись хранения, или использовать то же имя, чтобы обновить имеющуюся учетную запись хранения. Оба варианта работают.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name usenondefaultsku `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $templateFile `
  -storageName "{your-unique-name}" `
  -storageSKU Standard_GRS
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az deployment group create \
  --name usenondefaultsku \
  --resource-group myResourceGroup \
  --template-file $templateFile \
  --parameters storageSKU=Standard_GRS storageName={your-unique-name}
```

---

Наконец, давайте запустим еще один тест и посмотрим, что произойдет, если вы передадите номер SKU, который не входит в список допустимых значений. В этом случае мы проверяем сценарий, в котором пользователь шаблона считает, что **базовый** является одним из номеров SKU.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name testskuparameter `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $templateFile `
  -storageName "{your-unique-name}" `
  -storageSKU basic
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az deployment group create \
  --name testskuparameter \
  --resource-group myResourceGroup \
  --template-file $templateFile \
  --parameters storageSKU=basic storageName={your-unique-name}
```

---

Выполнение команды немедленно завершается сбоем с сообщением об ошибке, в котором указаны допустимые значения. Resource Manager выявляет ошибку до начала развертывания.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы переходите к следующему учебнику, вам не нужно удалять группу ресурсов.

Если вы останавливаете работу сейчас, вам может потребоваться очистить развернутые ресурсы, удалив группу ресурсов.

1. На портале Azure в меню слева выберите **Группа ресурсов**.
2. В поле **Фильтровать по имени** введите имя группы ресурсов.
3. Выберите имя группы ресурсов.
4. В главном меню выберите **Удалить группу ресурсов**.

## <a name="next-steps"></a>Дальнейшие действия

Вы улучшили шаблон, созданный в [первом учебнике](template-tutorial-create-first-template.md), добавив параметры. Из следующего учебника вы узнаете о функциях шаблонов.

> [!div class="nextstepaction"]
> [Учебник по добавлению функций шаблона в шаблон Azure Resource Manager](template-tutorial-add-functions.md)
