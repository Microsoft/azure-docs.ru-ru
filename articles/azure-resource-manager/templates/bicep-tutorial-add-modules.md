---
title: Учебник. Добавление модулей в Bicep-файл Azure Resource Manager
description: Использование модулей, чтобы инкапсулировать комплексные сведения об объявлении необработанного ресурса.
author: mumian
ms.date: 03/25/2021
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: 8c7ab1038cbe62d6f15faf56796193df12b38546
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105568772"
---
# <a name="tutorial-add-modules-to-azure-resource-manager-bicep-file"></a>Учебник. Добавление модулей в Bicep-файл Azure Resource Manager

Из[предыдущего учебника](bicep-tutorial-use-parameter-file.md) вы узнали, как использовать файл параметров для развертывания Bicep-файла. В этом учебнике вы узнаете, как использовать модули Bicep, чтобы инкапсулировать комплексные сведения об объявлении необработанного ресурса. Вы можете совместно и повторно использовать эти модули в решении.  Это занимает около **12 минут**.

[!INCLUDE [Bicep preview](../../../includes/resource-manager-bicep-preview.md)]

## <a name="prerequisites"></a>Предварительные требования

Советуем выполнить инструкции из [учебника по файлу параметров](bicep-tutorial-use-parameter-file.md), но это необязательно.

Вам понадобится Visual Studio Code с расширением Bicep и либо Azure PowerShell, либо Azure CLI. Дополнительные сведения см. в разделе о [средствах Bicep](bicep-tutorial-create-first-bicep.md#get-tools).

## <a name="review-bicep-file"></a>Проверка Bicep-файла

В конце предыдущего учебника Bicep-файл содержал следующее содержимое:

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-tags/azuredeploy.bicep":::

Этот Bicep-файл работает хорошо. Но для более крупных решений необходимо разбить Bicep-файл на множество связанных модулей, чтобы вы могли совместно и повторно использовать их. Текущий файл Bicep развертывает учетную запись хранения, план службы приложений и веб-сайт.  Давайте разделим учетную запись хранения на модули.

## <a name="create-bicep-module"></a>Создание модуля Bicep

Каждый Bicep-файл можно использовать как модуль, поэтому для определения модуля не существует специального синтаксиса. Создайте файл _storage.bicep_ со следующим содержимым:

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-module/storage.bicep":::

Этот модуль содержит ресурс учетной записи хранения, а также связанные параметры и переменные. Значения параметра _location_ и параметров _resourceTags_ были удалены. Эти значения будут переданы из основного Bicep-файла.

## <a name="consume-bicep-module"></a>Использование модуля Bicep

Замените определение ресурса учетной записи хранения в существующем файле _azuredeploy.bicep_ на следующее содержимое Bicep:

```bicep
module stg './storage.bicep' = {
  name: 'storageDeploy'
  params: {
    storagePrefix: storagePrefix
    location: location
    resourceTags: resourceTags
  }
}
```

- **module**. Ключевое слово.
- **symbolic name** (stg). Это идентификатор для модуля.
- **module file**. Путь к модулю в этом примере указан с помощью относительного пути (./storage.bicep). Все пути в Bicep должны быть указаны с помощью разделителя каталогов косой черты (/), чтобы обеспечить последовательную компиляцию на разных платформах. Символ обратной косой черты Windows (\)) не поддерживается.

Чтобы получить конечную точку хранилища, замените выходные данные в _azuredeploy.bicep_ на следующее содержимое Bicep:

```bicep
output storageEndpoint object = stg.outputs.storageEndpoint
```

Завершенный файл azuredeploy.bicep имеет следующее содержимое:

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-module/azuredeploy.bicep":::

## <a name="deploy-template"></a>Развертывание шаблона

Для развертывания шаблона используйте либо Azure CLI, либо Azure PowerShell.

Если вы еще не создали группу ресурсов, см. [этот раздел](bicep-tutorial-create-first-bicep.md#create-resource-group). В этом примере предполагается, что для переменной `bicepFile` указан путь к Bicep-файлу, как показано в [первом учебнике](bicep-tutorial-create-first-bicep.md#deploy-bicep-file).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Чтобы выполнить этот командлет развертывания, необходимо иметь [последнюю версию](/powershell/azure/install-az-ps) Azure PowerShell.

```azurepowershell
New-AzResourceGroupDeployment `
  -Name addmodule `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $bicepFile `
  -storagePrefix "store" `
  -storageSKU Standard_LRS `
  -webAppName demoapp
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы выполнить эту команду развертывания, необходимо иметь [последнюю версию](/cli/azure/install-azure-cli) Azure CLI.

```azurecli
az deployment group create \
  --name addmodule \
  --resource-group myResourceGroup \
  --template-file $bicepFile \
  --parameters storagePrefix=store storageSKU=Standard_LRS webAppName=demoapp
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

Поздравляем, вы завершили ознакомление с развертыванием Bicep-файлов в Azure. Оставьте комментарии и предложения для нас в разделе отзывов. Спасибо!

В следующей серии руководств подробно рассматривается развертывание шаблонов.

> [!div class="nextstepaction"]
> [Развертывание шаблона в локальной среде](./deployment-tutorial-local-template.md)
