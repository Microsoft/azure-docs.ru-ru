---
title: Учебник. Добавление параметров в Bicep-файл Azure Resource Manager
description: Добавление параметров в Bicep-файл для его повторного использования.
author: mumian
ms.date: 03/10/2021
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: 00df2ffc6272011127c5a1eb0c1e302011f8de5f
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102632787"
---
# <a name="tutorial-add-parameters-to-azure-resource-manager-bicep-file"></a>Учебник. Добавление параметров в Bicep-файл Azure Resource Manager

Из [предыдущего учебника](bicep-tutorial-create-first-bicep.md) вы узнали, как развернуть учетную запись хранения. Из этого учебника вы узнаете, как улучшить Bicep-файл, добавив в него параметры. Для работы с этим учебником потребуется около **14 минут**.

[!INCLUDE [Bicep preview](../../../includes/resource-manager-bicep-preview.md)]

## <a name="prerequisites"></a>Предварительные требования

Советуем выполнить инструкции из [учебника по созданию Bicep-файла](bicep-tutorial-create-first-bicep.md), но это необязательно.

Вам понадобится Visual Studio Code с расширением Bicep и либо Azure PowerShell, либо Azure CLI. Дополнительные сведения см. в разделе о [средствах Bicep](bicep-tutorial-create-first-bicep.md#get-tools).

## <a name="review-bicep-file"></a>Проверка Bicep-файла

В конце предыдущего учебника Bicep-файл выглядел следующим образом:

:::code language="bicep" source="~/resourcemanager-templates/get-started-with-templates/add-storage/azuredeploy.bicep":::

Возможно, вы заметили, что существует проблема с этим Bicep-файлом. Имя учетной записи хранения жестко запрограммировано. Этот Bicep-файл можно использовать только для развертывания одной и той же учетной записи хранения каждый раз. Чтобы развернуть учетную запись хранения с другим именем, вам нужно будет создать Bicep-файл. Это, очевидно, не практичный способ автоматизации развертываний.

## <a name="make-bicep-file-reusable"></a>Изменение Bicep-файла для повторного использования

Чтобы сделать Bicep-файл многократно используемым, необходимо добавить параметр, который вы можете применять для передачи имени учетной записи хранения. Следующий Bicep-файл демонстрирует, что изменилось в вашем файле. Параметр `storageName` определяется как строка. Для максимальной длины установлено значение, равное 24 символам, чтобы не задавать слишком длинные имена.

Скопируйте весь файл и замените его содержимое указанными ниже значениями.

:::code language="bicep" source="~/resourcemanager-templates/get-started-with-templates/add-name/azuredeploy.bicep" range="1-15" highlight="1-3,6":::

Обратите внимание на то, что на параметры можно ссылаться напрямую, используя их имена в Bicep. В шаблоне JSON ARM необходимо указывать [parameters('storageName')].

## <a name="deploy-bicep-file"></a>Развертывание файла Bicep

Развернем Bicep-файл. В следующем примере Bicep-файл развертывается с помощью Azure CLI или PowerShell. Обратите внимание: вы указываете имя учетной записи хранения в качестве одного из значений в команде развертывания. Для имени учетной записи хранения укажите то же имя, которое вы использовали в предыдущем учебнике.

Если вы еще не создали группу ресурсов, см. [этот раздел](bicep-tutorial-create-first-bicep.md#create-resource-group). В этом примере предполагается, что для переменной `bicepFile` указан путь к Bicep-файлу, как показано в [первом учебнике](bicep-tutorial-create-first-bicep.md#deploy-bicep-file).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Чтобы выполнить этот командлет развертывания, необходимо иметь [последнюю версию](/powershell/azure/install-az-ps) Azure PowerShell.

```azurepowershell
New-AzResourceGroupDeployment `
  -Name addnameparameter `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $bicepFile `
  -storageName "{your-unique-name}"
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы выполнить эту команду развертывания, необходимо иметь [последнюю версию](/cli/azure/install-azure-cli) Azure CLI.

```azurecli
az deployment group create \
  --name addnameparameter \
  --resource-group myResourceGroup \
  --template-file $bicepFile \
  --parameters storageName={your-unique-name}
```

---

## <a name="understand-resource-updates"></a>Концепция обновления ресурсов

В предыдущем разделе вы развернули учетную запись хранения с созданным ранее именем. Вас может интересовать, как ресурс повлиял на повторное развертывание.

Если ресурс уже существует и в свойствах не обнаружено изменений, никакие действия не предпринимаются. Если ресурс уже существует, а свойство изменилось, ресурс обновляется. Если ресурс не существует, он создается.

Этот способ обработки обновлений означает, что Bicep-файл может включать в себя все ресурсы, необходимые для решения Azure. Вы можете безопасно повторно развернуть Bicep-файл и знать, что ресурсы изменяются или создаются только при необходимости. Например, если вы добавили файлы в свою учетную запись хранения, вы можете повторно развернуть учетную запись хранения без потери этих файлов.

## <a name="customize-by-environment"></a>Настройка с учетом среды

Параметры позволяют настраивать развертывание путем предоставления значений, предназначенных для конкретной среды. Например, вы можете передавать различные значения в зависимости от того, развертываете ли вы среду для разработки, тестирования и производства.

Предыдущий Bicep-файл всегда развертывал учетную запись хранения **Standard_LRS**. Возможно, вам потребуется гибкость для развертывания различных номеров SKU в зависимости от среды. В следующем примере показаны изменения для добавления параметра для номера SKU. Скопируйте весь файл и вставьте в Bicep-файл.

:::code language="bicep" source="~/resourcemanager-templates/get-started-with-templates/add-sku/azuredeploy.bicep" range="1-27" highlight="5-15,21":::

Параметр `storageSKU` имеет значение по умолчанию. Оно используется, если во время развертывания не указано значение. В нем также содержится список допустимых значений. Они соответствуют значениям, требуемым для создания учетной записи хранения. Вы не хотите, чтобы пользователи Bicep-файла передавали нерабочие номера SKU.

## <a name="redeploy-bicep-file"></a>Повторное развертывание файла Bicep

Теперь все готово к повторному развертыванию. Так как для SKU по умолчанию установлено значение **Standard_LRS**, вам не нужно указывать значение для этого параметра.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name addskuparameter `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $bicepFile `
  -storageName "{your-unique-name}"
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az deployment group create \
  --name addskuparameter \
  --resource-group myResourceGroup \
  --template-file $bicepFile \
  --parameters storageName={your-unique-name}
```

---

> [!NOTE]
> Если развертывание завершилось сбоем, используйте параметр `verbose`, чтобы получить сведения о создаваемых ресурсах. Используйте параметр `debug`, чтобы получить дополнительные сведения для отладки.

Чтобы убедиться в гибкости Bicep-файла, давайте снова его развернем. На этот раз установите для параметра SKU значение **Standard_GRS**. Вы можете передать новое имя, чтобы создать другую учетную запись хранения, или использовать то же имя, чтобы обновить имеющуюся учетную запись хранения. Оба варианта работают.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name usenondefaultsku `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $bicepFile `
  -storageName "{your-unique-name}" `
  -storageSKU Standard_GRS
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az deployment group create \
  --name usenondefaultsku \
  --resource-group myResourceGroup \
  --template-file $bicepFile \
  --parameters storageSKU=Standard_GRS storageName={your-unique-name}
```

---

Наконец, давайте запустим еще один тест и посмотрим, что произойдет, если вы передадите номер SKU, который не входит в список допустимых значений. В этом случае мы проверяем сценарий, в котором пользователь Bicep-файла считает, что **базовый** является одним из номеров SKU.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name testskuparameter `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $bicepFile `
  -storageName "{your-unique-name}" `
  -storageSKU basic
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az deployment group create \
  --name testskuparameter \
  --resource-group myResourceGroup \
  --template-file $bicepFile \
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

Вы улучшили Bicep-файл, созданный в [первом учебнике](bicep-tutorial-create-first-bicep.md), добавив параметры. Из следующего учебника вы узнаете о функциях Bicep-файла.

> [!div class="nextstepaction"]
> [Добавление функций](bicep-tutorial-add-functions.md)
