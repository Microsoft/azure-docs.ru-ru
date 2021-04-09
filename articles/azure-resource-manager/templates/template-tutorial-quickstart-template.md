---
title: Учебник. Использование шаблонов быстрого запуска
description: Узнайте, как использовать шаблоны быстрого запуска Azure для разработки шаблонов.
author: mumian
ms.date: 03/27/2020
ms.topic: tutorial
ms.author: jgao
ms.custom: ''
ms.openlocfilehash: 46b32ae7aeb971c9391a69e3ca3d01f669774248
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97106909"
---
# <a name="tutorial-use-azure-quickstart-templates"></a>Руководство по Tutorial: Use Azure Quickstart templates (Учебник. Использование шаблонов быстрого запуска)

[Шаблоны быстрого запуска Azure](https://azure.microsoft.com/resources/templates/) — это репозиторий шаблонов, наполняемый силами сообщества. Примеры шаблонов можно использовать при разработке шаблонов. В этом руководстве вы найдете определение ресурса веб-сайта и добавите его в собственный шаблон. Это занимает около **12 минут**.

## <a name="prerequisites"></a>Предварительные требования

Советуем выполнить инструкции из [учебника по экспортированным шаблонам](template-tutorial-export-template.md), но это необязательно.

Вам понадобится Visual Studio Code с расширением инструментов Resource Manager и либо Azure PowerShell, либо Azure CLI. Дополнительные сведения см. в разделе об [инструментах шаблона](template-tutorial-create-first-template.md#get-tools).

## <a name="review-template"></a>Проверка шаблона

В конце предыдущего учебника шаблон содержал следующий код JSON:

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/export-template/azuredeploy.json":::

Этот шаблон подходит для развертывания учетных записей хранения и планов службы приложений, но может потребоваться добавить в него веб-сайт. Вы можете использовать готовые шаблоны для быстрого обнаружения JSON, необходимого для развертывания ресурса.

## <a name="find-template"></a>Поиск шаблона

1. Откройте [шаблоны быстрого запуска Azure](https://azure.microsoft.com/resources/templates/)
1. В поле **Поиск** введите _deploy linux web app_.
1. Выберите элемент с названием **Deploy a basic Linux web app** (Развертывание простого веб-приложения Linux). Если у вас возникли проблемы с поиском, используйте [прямую ссылку](https://azure.microsoft.com/resources/templates/101-webapp-basic-linux/).
1. Выберите **Browse on GitHub** (Найти на GitHub).
1. Выберите _azuredeploy.json_.
1. Изучите шаблон. В частности, найдите ресурс `Microsoft.Web/sites`.

    ![Веб-сайт краткого руководства по шаблону Resource Manager](./media/template-tutorial-quickstart-template/resource-manager-template-quickstart-template-web-site.png)

## <a name="revise-existing-template"></a>Изменение имеющегося шаблона

Объедините шаблон быстрого запуска с существующим шаблоном:

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/quickstart-template/azuredeploy.json" range="1-108" highlight="32-45,49,85-100":::

Имя веб-приложения должно быть уникальным в Azure. Чтобы предотвратить дублирование имен, переменная `webAppPortalName` была обновлена с `"webAppPortalName": "[concat(parameters('webAppName'), '-webapp')]"` на `"webAppPortalName": "[concat(parameters('webAppName'), uniqueString(resourceGroup().id))]"` .

Добавьте запятую в конце определения `Microsoft.Web/serverfarms`, чтобы отделить определение ресурса от определения `Microsoft.Web/sites`.

В этом новом ресурсе есть несколько важных компонентов.

Вы увидите, что он содержит элемент с именем `dependsOn`, для которого установлен план службы приложений. Этот параметр является обязательным, так как план службы приложений должен существовать до создания веб-приложения. Элемент `dependsOn` указывает Resource Manager, как упорядочивать ресурсы для развертывания.

Свойство `serverFarmId` использует функцию [resourceId](template-functions-resource.md#resourceid). Эта функция возвращает уникальный идентификатор ресурса. Здесь она получает уникальный идентификатор для плана службы приложений. Веб-приложение связано с одним определенным планом службы приложений.

## <a name="deploy-template"></a>Развертывание шаблона

Для развертывания шаблона используйте Azure CLI или Azure PowerShell.

Если вы еще не создали группу ресурсов, см. [этот раздел](template-tutorial-create-first-template.md#create-resource-group). В этом примере предполагается, что для переменной **templateFile** указан путь к файлу шаблона, как показано в [первом учебнике](template-tutorial-create-first-template.md#deploy-template).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name addwebapp `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $templateFile `
  -storagePrefix "store" `
  -storageSKU Standard_LRS `
  -webAppName demoapp
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы выполнить эту команду развертывания, необходимо иметь [последнюю версию](/cli/azure/install-azure-cli) Azure CLI.

```azurecli
az deployment group create \
  --name addwebapp \
  --resource-group myResourceGroup \
  --template-file $templateFile \
  --parameters storagePrefix=store storageSKU=Standard_LRS webAppName=demoapp
```

---

> [!NOTE]
> Если развертывание завершилось сбоем, используйте параметр `verbose`, чтобы получить сведения о создаваемых ресурсах. Используйте параметр `debug`, чтобы получить дополнительные сведения для отладки.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы переходите к следующему учебнику, вам не нужно удалять группу ресурсов.

Если вы останавливаете работу сейчас, вам может потребоваться очистить развернутые ресурсы, удалив группу ресурсов.

1. На портале Azure в меню слева выберите **Группа ресурсов**.
2. В поле **Фильтровать по имени** введите имя группы ресурсов.
3. Выберите имя группы ресурсов.
4. В главном меню выберите **Удалить группу ресурсов**.

## <a name="next-steps"></a>Дальнейшие действия

В этом учебнике вы узнали, как использовать шаблон быстрого запуска при разработке шаблонов. В следующем учебнике вы узнаете, как добавить в ресурсы теги.

> [!div class="nextstepaction"]
> [Добавление тегов](template-tutorial-add-tags.md)
