---
title: 'Учебник: добавление выходных данных в Bicep-файл Azure Resource Manager'
description: Добавление выходных данных в Bicep-файл для упрощения синтаксиса.
author: mumian
ms.date: 03/17/2021
ms.topic: tutorial
ms.author: jgao
ms.custom: ''
ms.openlocfilehash: 4b7d7a1414091c516dba2c474e1681ba357b55a1
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104594314"
---
# <a name="tutorial-add-outputs-to-azure-resource-manager-bicep-file"></a>Учебник: добавление выходных данных в Bicep-файл Azure Resource Manager

Их этого учебника вы узнаете, как вернуть значение развертывания. Если требуется значение из развернутого ресурса, используются выходные данные. Для выполнения инструкций из этого учебника требуется **7 минут**.

[!INCLUDE [Bicep preview](../../../includes/resource-manager-bicep-preview.md)]

## <a name="prerequisites"></a>Предварительные требования

Советуем выполнить инструкции из [учебника по переменным](bicep-tutorial-add-variables.md), но это необязательно.

Вам понадобится Visual Studio Code с расширением Bicep и либо Azure PowerShell, либо Azure CLI. Дополнительные сведения см. в разделе о [средствах Bicep](bicep-tutorial-create-first-bicep.md#get-tools).

## <a name="review-bicep-file"></a>Проверка Bicep-файла

В конце предыдущего учебника Bicep-файл содержал следующее содержимое:

:::code language="bicep" source="~/resourcemanager-templates/get-started-with-templates/add-variable/azuredeploy.bicep":::

Он развертывает учетную запись хранения, но не возвращает никаких сведений о ней. Возможно, потребуется записать свойства из нового ресурса, чтобы они были доступны позже для справки.

## <a name="add-outputs"></a>Добавление выходных данных

Выходные данные можно использовать для возврата значений из развертывания. Например, может быть полезно получить конечные точки для новой учетной записи хранения.

В следующем примере показано, как изменить файл Bicep для добавления выходного значения. Скопируйте весь файл и замените Bicep-файлы на его содержимое.

:::code language="bicep" source="~/resourcemanager-templates/get-started-with-templates/add-outputs/azuredeploy.bicep" range="1-33" highlight="33":::

Обратите внимание на некоторые важные элементы добавленного выходного значения.

В качестве типа возвращаемого значения устанавливается `object`. Это означает, что возвращается объект шаблона.

Чтобы получить свойство `primaryEndpoints` из учетной записи хранения, используйте символьное имя учетной записи хранения. Функция автозаполнения Visual Studio Code предоставляет полный список свойств:

   ![Свойства объекта символьного имени Bicep в Visual Studio Code](./media/bicep-tutorial-add-outputs/visual-studio-code-bicep-output-properties.png)

## <a name="deploy-bicep-file"></a>Развертывание файла Bicep

Вы готовы к развертыванию Bicep-файла и просмотру возвращенного значения.

Если вы еще не создали группу ресурсов, см. [этот раздел](bicep-tutorial-create-first-bicep.md#create-resource-group). В этом примере предполагается, что для переменной `bicepFile` указан путь к Bicep-файлу, как показано в [первом учебнике](bicep-tutorial-create-first-bicep.md#deploy-bicep-file).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name addoutputs `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $bicepFile `
  -storagePrefix "store" `
  -storageSKU Standard_LRS
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы выполнить эту команду развертывания, необходимо иметь [последнюю версию](/cli/azure/install-azure-cli) Azure CLI.

```azurecli
az deployment group create \
  --name addoutputs \
  --resource-group myResourceGroup \
  --template-file $bicepFile \
  --parameters storagePrefix=store storageSKU=Standard_LRS
```

---

В выходных данных команды развертывания вы увидите объект, как в следующем примере, только если выходные данные имеют формат JSON:

```json
{
  "dfs": "https://storeluktbfkpjjrkm.dfs.core.windows.net/",
  "web": "https://storeluktbfkpjjrkm.z19.web.core.windows.net/",
  "blob": "https://storeluktbfkpjjrkm.blob.core.windows.net/",
  "queue": "https://storeluktbfkpjjrkm.queue.core.windows.net/",
  "table": "https://storeluktbfkpjjrkm.table.core.windows.net/",
  "file": "https://storeluktbfkpjjrkm.file.core.windows.net/"
}
```

> [!NOTE]
> Если развертывание завершилось сбоем, используйте параметр `verbose`, чтобы получить сведения о создаваемых ресурсах. Используйте параметр `debug`, чтобы получить дополнительные сведения для отладки.

## <a name="review-your-work"></a>Проверка работы

Работая с последними шестью учебниками, вы много чего сделали. Давайте взглянем на выполненную работу. Вы создали Bicep-файл с параметрами, которые легко предоставить. Bicep-файл можно многократно использовать в разных средах, так как его можно настраивать и он может динамически создавать необходимые значения. Он также возвращает сведения об учетной записи хранения, которую можно использовать в скрипте.

Теперь рассмотрим группу ресурсов и журнал развертывания.

1. Войдите на [портал Azure](https://portal.azure.com).
1. В меню слева выберите **Группы ресурсов**.
1. Выберите группу ресурсов, в которую выполнено развертывание.
1. В зависимости от выполненных действий в группе ресурсов должна быть по крайней мере одна учетная запись хранения (или несколько).
1. Кроме того, в журнале должно быть зафиксировано несколько успешных развертываний. Выберите эту ссылку.

   ![Выбор развертываний](./media/bicep-tutorial-add-outputs/select-deployments.png)

1. Вы увидите все развертывания в журнале. Выберите развертывание с именем **addoutputs**.

   ![Отображение журнала развертывания](./media/bicep-tutorial-add-outputs/show-history.png)

1. Вы можете проверить входные данные.

   ![Отображение входных данных](./media/bicep-tutorial-add-outputs/show-inputs.png)

1. Вы можете проверить выходные данные.

   ![Отображение выходных данных](./media/bicep-tutorial-add-outputs/show-outputs.png)

1. Вы можете изучить шаблон JSON.

   ![Отображение шаблона](./media/bicep-tutorial-add-outputs/show-template.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы переходите к следующему учебнику, вам не нужно удалять группу ресурсов.

Если вы останавливаете работу сейчас, вам может потребоваться очистить развернутые ресурсы, удалив группу ресурсов.

1. На портале Azure в меню слева выберите **Группа ресурсов**.
2. В поле **Фильтровать по имени** введите имя группы ресурсов.
3. Выберите имя группы ресурсов.
4. В главном меню выберите **Удалить группу ресурсов**.

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого учебника вы добавили в Bicep-файл возвращаемое значение. Из следующего учебнике вы узнаете, как экспортировать шаблон JSON и использовать его части в Bicep-файле.

> [!div class="nextstepaction"]
> [Учебник по использованию экспортированного шаблона из портала Azure](bicep-tutorial-export-template.md)
