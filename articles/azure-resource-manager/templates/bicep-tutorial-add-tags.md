---
title: Учебник. Добавление тегов к ресурсам в Bicep-файле Azure Resource Manager
description: Сведения о том, как добавить теги к ресурсам в Bicep-файле. Теги позволяют логически упорядочивать ресурсы.
author: mumian
ms.date: 03/10/2021
ms.topic: tutorial
ms.author: jgao
ms.custom: ''
ms.openlocfilehash: ea5e078eb692d002b3f86cd43663dd042d692611
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102632619"
---
# <a name="tutorial-add-tags-in-azure-resource-manager-bicep-files"></a>Учебник. Добавление тегов в Bicep-файлы Azure Resource Manager

В этом учебнике описывается, как добавлять теги к ресурсам в Bicep-файлах. [Теги](../management/tag-resources.md) помогают логически упорядочивать ресурсы. Значения тегов отображаются в отчетах о затратах. Для выполнения инструкций из этого учебника требуется **8 минут**.

[!INCLUDE [Bicep preview](../../../includes/resource-manager-bicep-preview.md)]

## <a name="prerequisites"></a>Предварительные требования

Советуем выполнить инструкции из [учебника по шаблонам быстрого запуска](bicep-tutorial-quickstart-template.md), но это необязательно.

Вам понадобится Visual Studio Code с расширением Bicep и либо Azure PowerShell, либо Azure CLI. Дополнительные сведения см. в разделе о [средствах Bicep](bicep-tutorial-create-first-bicep.md#get-tools).

## <a name="review-bicep-file"></a>Проверка Bicep-файла

Предыдущий Bicep-файл развернул учетную запись хранения, план службы приложений и веб-приложение.

:::code language="bicep" source="~/resourcemanager-templates/get-started-with-templates/quickstart-template/azuredeploy.bicep":::

После развертывания этих ресурсов может потребоваться мониторинг затрат и поиск ресурсов, принадлежащих категории. Для решения этих задач можно добавить теги.

## <a name="add-tags"></a>Добавление тегов

Вы добавляете теги в ресурсы, чтобы использовать значения, помогающие определить их использование. Например, можно добавить теги, в которых перечислены среда и проект. Можно добавить теги, которые определяют центр затрат или группу, которой принадлежит ресурс. Добавьте любые значения, имеющие смысл для вашей организации.

В следующем примере показаны изменения Bicep-файла. Скопируйте весь файл и замените Bicep-файлы на его содержимое.

:::code language="bicep" source="~/resourcemanager-templates/get-started-with-templates/add-tags/azuredeploy.bicep" range="1-81" highlight="27-30,38,51,71":::

## <a name="deploy-bicep-file"></a>Развертывание файла Bicep

Давайте развернем Bicep-файл и просмотрим результаты.

Если вы еще не создали группу ресурсов, см. [этот раздел](bicep-tutorial-create-first-bicep.md#create-resource-group). В этом примере предполагается, что для переменной `bicepFile` указан путь к Bicep-файлу, как показано в [первом учебнике](bicep-tutorial-create-first-bicep.md#deploy-bicep-file).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Чтобы выполнить этот командлет развертывания, необходимо иметь [последнюю версию](/powershell/azure/install-az-ps) Azure PowerShell.

```azurepowershell
New-AzResourceGroupDeployment `
  -Name addtags `
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
  --name addtags \
  --resource-group myResourceGroup \
  --template-file $bicepFile \
  --parameters storagePrefix=store storageSKU=Standard_LRS webAppName=demoapp
```

---

> [!NOTE]
> Если развертывание завершилось сбоем, используйте параметр `verbose`, чтобы получить сведения о создаваемых ресурсах. Используйте параметр `debug`, чтобы получить дополнительные сведения для отладки.

## <a name="verify-deployment"></a>Проверка развертывания

Чтобы проверить развертывание, просмотрите группу ресурсов на портале Azure.

1. Войдите на [портал Azure](https://portal.azure.com).
1. В меню слева выберите **Группы ресурсов**.
1. Выберите группу ресурсов, в которую выполнено развертывание.
1. Выберите один из ресурсов, например ресурс учетной записи хранения. Вы увидите, что теперь он содержит теги.

   ![Отображение тегов](./media/bicep-tutorial-add-tags/show-tags.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы переходите к следующему учебнику, вам не нужно удалять группу ресурсов.

Если вы останавливаете работу сейчас, вам может потребоваться очистить развернутые ресурсы, удалив группу ресурсов.

1. На портале Azure в меню слева выберите **Группа ресурсов**.
2. В поле **Фильтровать по имени** введите имя группы ресурсов.
3. Выберите имя группы ресурсов.
4. В главном меню выберите **Удалить группу ресурсов**.

## <a name="next-steps"></a>Дальнейшие действия

В этом учебнике вы добавили теги в ресурсы. В следующем учебнике описывается, как использовать файлы параметров для упрощения передачи значений в развертывание.

> [!div class="nextstepaction"]
> [Использование файла параметров](bicep-tutorial-use-parameter-file.md)
