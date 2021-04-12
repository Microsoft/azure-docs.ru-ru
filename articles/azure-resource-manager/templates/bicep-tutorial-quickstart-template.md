---
title: Учебник. Использование шаблонов быстрого запуска для разработки Bicep-файла Azure Resource Manager
description: Узнайте, как использовать шаблоны быстрого запуска Azure для разработки Bicep-файла.
author: mumian
ms.date: 03/10/2021
ms.topic: tutorial
ms.author: jgao
ms.custom: ''
ms.openlocfilehash: cf655885e01fe6bca99a82c82d6bbbc4c1a080b3
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102632449"
---
# <a name="tutorial-use-azure-quickstart-templates-for-azure-resource-manager-bicep-development"></a>Учебник. Использование шаблонов быстрого запуска Azure для разработки Bicep-файла Azure Resource Manager

[Шаблоны быстрого запуска Azure](https://azure.microsoft.com/resources/templates/) — это репозиторий шаблонов JSON, созданных сообществом. Примеры шаблонов можно использовать для разработки Bicep-файла. При работе с этим учебником вы найдете определение ресурса веб-сайта, декомпилируете его в Bicep и добавите в Bicep-файл. Это занимает около **12 минут**.

[!INCLUDE [Bicep preview](../../../includes/resource-manager-bicep-preview.md)]

## <a name="prerequisites"></a>Предварительные требования

Советуем выполнить инструкции из [учебника по экспортированным шаблонам](bicep-tutorial-export-template.md), но это необязательно.

Вам понадобится Visual Studio Code с расширением Bicep и либо Azure PowerShell, либо Azure CLI. Дополнительные сведения см. в разделе о [средствах Bicep](bicep-tutorial-create-first-bicep.md#get-tools).

## <a name="review-bicep-file"></a>Проверка Bicep-файла

В конце предыдущего учебника Bicep-файл содержал следующее содержимое:

:::code language="bicep" source="~/resourcemanager-templates/get-started-with-templates/export-template/azuredeploy.bicep":::

Этот Bicep-файл подходит для развертывания учетных записей хранения и планов службы приложений, но может потребоваться добавить в него веб-сайт. Вы можете использовать готовые шаблоны для быстрого обнаружения JSON, необходимого для развертывания ресурса. Прежде чем получить примеры Bicep с помощью шаблонов быстрого запуска Azure, для преобразования шаблонов JSON в Bicep-файлы можно использовать конвертеры.

## <a name="find-template"></a>Поиск шаблона

В настоящее время с помощью шаблонов быстрого запуска Azure можно получить только шаблоны JSON. Существуют средства, которые можно использовать, чтобы декомпилировать шаблоны JSON в шаблоны Bicep.

1. Откройте [шаблоны быстрого запуска Azure](https://azure.microsoft.com/resources/templates/)
1. В поле **Поиск** введите _deploy linux web app_.
1. Выберите элемент с названием **Deploy a basic Linux web app** (Развертывание простого веб-приложения Linux). Если у вас возникли проблемы с поиском, используйте [прямую ссылку](https://azure.microsoft.com/resources/templates/101-webapp-basic-linux/).
1. Выберите **Browse on GitHub** (Найти на GitHub).
1. Выберите _azuredeploy.json_. Этот шаблон можно использовать.
1. Выберите **Без форматирования**, а затем создайте копию URL-адреса.
1. Перейдите к **https://bicepdemo.z22.web.core.windows.net/** , выберите **Декомпилировать**, а затем укажите URL-адрес необработанного шаблона.
1. Проверьте шаблон Bicep. В частности, найдите ресурс `Microsoft.Web/sites`.

    ![Веб-сайт краткого руководства по шаблону Resource Manager](./media/bicep-tutorial-quickstart-template/resource-manager-template-quickstart-template-web-site.png)

    Если вы работали с шаблонами JSON, в этом новом ресурсе есть несколько важных компонентов Bicep для работы с шаблонами JSON.

    В шаблонах ARM JSON необходимо вручную указать зависимости ресурсов, используя свойство _dependsOn_. Ресурс веб-сайта зависит от ресурса плана службы приложений. Если используется Bicep, то при ссылке на любое свойство ресурса с помощью символического имени автоматически добавляется свойство dependsOn.

    Вы можете легко ссылаться на идентификатор ресурса, используя символическое имя плана службы приложений (appServicePlanName.id), которое будет преобразовано в функцию resourceId(...) в скомпилированном шаблоне JSON.

## <a name="revise-existing-bicep-file"></a>Изменение имеющегося Bicep-файла

Объедините декомпилированные шаблоны быстрого запуска, используя существующий Bicep-файл. Обновите символическое имя ресурса и имя ресурса в соответствии с соглашением об именовании, как вы делали при работе с предыдущим учебником.

:::code language="bicep" source="~/resourcemanager-templates/get-started-with-templates/quickstart-template/azuredeploy.bicep" range="1-73" highlight="20-25,28,61-71":::

Имя веб-приложения должно быть уникальным в Azure. Чтобы предотвратить дублирование имен, переменная `webAppPortalName` была обновлена с `var webAppPortalName_var = '${webAppName}-webapp'` на `var webAppPortalName = '${webAppName}${uniqueString(resourceGroup().id)}'` .

## <a name="deploy-bicep-file"></a>Развертывание файла Bicep

Для развертывания шаблона Bicep используйте либо Azure CLI, либо Azure PowerShell.

Если вы еще не создали группу ресурсов, см. [этот раздел](bicep-tutorial-create-first-bicep.md#create-resource-group). В этом примере предполагается, что для переменной **bicepFile** указан путь к Bicep-файлу, как показано в [первом учебнике](bicep-tutorial-create-first-bicep.md#deploy-bicep-file).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Чтобы выполнить этот командлет развертывания, необходимо иметь [последнюю версию](/powershell/azure/install-az-ps) Azure PowerShell.

```azurepowershell
New-AzResourceGroupDeployment `
  -Name addwebapp `
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
  --name addwebapp \
  --resource-group myResourceGroup \
  --template-file $bicepFile \
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

Из этого учебника вы узнали, как использовать шаблон быстрого запуска для разработки Bicep-файлов. В следующем учебнике вы узнаете, как добавить в ресурсы теги.

> [!div class="nextstepaction"]
> [Добавление тегов](bicep-tutorial-add-tags.md)
