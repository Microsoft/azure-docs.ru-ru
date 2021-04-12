---
title: Учебник. Добавление выходных данных в шаблон
description: Сведения о том, как добавить выходные данные в шаблон Azure Resource Manager, чтобы упростить синтаксис.
author: mumian
ms.date: 03/27/2020
ms.topic: tutorial
ms.author: jgao
ms.custom: ''
ms.openlocfilehash: 7e022822f1306a5351cef43498f00a0ac0f2ef63
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101707291"
---
# <a name="tutorial-add-outputs-to-your-arm-template"></a>Руководство по добавлению выходных данных в шаблон ARM

Из этого учебника вы узнаете, как возвращать значения из шаблона Azure Resource Manager (ARM). Если требуется значение из развернутого ресурса, используются выходные данные. Для выполнения инструкций из этого учебника требуется **7 минут**.

## <a name="prerequisites"></a>Предварительные требования

Советуем выполнить инструкции из [учебника по переменным](template-tutorial-add-variables.md), но это необязательно.

Вам понадобится Visual Studio Code с расширением инструментов Resource Manager и либо Azure PowerShell, либо Azure CLI. Дополнительные сведения см. в разделе об [инструментах шаблона](template-tutorial-create-first-template.md#get-tools).

## <a name="review-template"></a>Проверка шаблона

В конце предыдущего учебника шаблон содержал следующий код JSON:

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-variable/azuredeploy.json":::

Он развертывает учетную запись хранения, но не возвращает никаких сведений о ней. Возможно, потребуется записать свойства из нового ресурса, чтобы они были доступны позже для справки.

## <a name="add-outputs"></a>Добавление выходных данных

Выходные данные можно использовать для возврата значений из шаблона. Например, может быть полезно получить конечные точки для новой учетной записи хранения.

В следующем примере демонстрируется изменение шаблона для добавления выходного значения. Скопируйте весь файл и замените шаблон на его содержимое.

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-outputs/azuredeploy.json" range="1-53" highlight="47-52":::

Обратите внимание на некоторые важные элементы добавленного выходного значения.

В качестве типа возвращаемого значения устанавливается `object`. Это означает, что возвращается объект JSON.

Для получения состояния среды выполнения учетной записи хранения используется функция [reference](template-functions-resource.md#reference). Чтобы получить состояние среды выполнения ресурса, необходимо передать имя или идентификатор ресурса. В этом случае вы используете ту же переменную, которая использовалась для создания имени учетной записи хранения.

Наконец, значение возвращает свойство `primaryEndpoints` из учетной записи хранения.

## <a name="deploy-template"></a>Развертывание шаблона

Вы готовы к развертыванию шаблона и просмотру возвращенного значения.

Если вы еще не создали группу ресурсов, см. [этот раздел](template-tutorial-create-first-template.md#create-resource-group). В этом примере предполагается, что для переменной `templateFile` указан путь к файлу шаблона, как показано в [первом учебнике](template-tutorial-create-first-template.md#deploy-template).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name addoutputs `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $templateFile `
  -storagePrefix "store" `
  -storageSKU Standard_LRS
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы выполнить эту команду развертывания, необходимо иметь [последнюю версию](/cli/azure/install-azure-cli) Azure CLI.

```azurecli
az deployment group create \
  --name addoutputs \
  --resource-group myResourceGroup \
  --template-file $templateFile \
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

Работая с последними шестью учебниками, вы много чего сделали. Давайте взглянем на выполненную работу. Вы создали шаблон с параметрами, которые легко предоставить. Шаблон можно многократно использовать в разных средах, так как его можно настраивать и он может динамически создавать необходимые значения. Он также возвращает сведения об учетной записи хранения, которую можно использовать в скрипте.

Теперь рассмотрим группу ресурсов и журнал развертывания.

1. Войдите на [портал Azure](https://portal.azure.com).
1. В меню слева выберите **Группы ресурсов**.
1. Выберите группу ресурсов, в которую выполнено развертывание.
1. В зависимости от выполненных действий в группе ресурсов должна быть по крайней мере одна учетная запись хранения (или несколько).
1. Кроме того, в журнале должно быть зафиксировано несколько успешных развертываний. Выберите эту ссылку.

   ![Выбор развертываний](./media/template-tutorial-add-outputs/select-deployments.png)

1. Вы увидите все развертывания в журнале. Выберите развертывание с именем **addoutputs**.

   ![Отображение журнала развертывания](./media/template-tutorial-add-outputs/show-history.png)

1. Вы можете проверить входные данные.

   ![Отображение входных данных](./media/template-tutorial-add-outputs/show-inputs.png)

1. Вы можете проверить выходные данные.

   ![Отображение выходных данных](./media/template-tutorial-add-outputs/show-outputs.png)

1. Вы можете изучить шаблон.

   ![Отображение шаблона](./media/template-tutorial-add-outputs/show-template.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы переходите к следующему учебнику, вам не нужно удалять группу ресурсов.

Если вы останавливаете работу сейчас, вам может потребоваться очистить развернутые ресурсы, удалив группу ресурсов.

1. На портале Azure в меню слева выберите **Группа ресурсов**.
2. В поле **Фильтровать по имени** введите имя группы ресурсов.
3. Выберите имя группы ресурсов.
4. В главном меню выберите **Удалить группу ресурсов**.

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого учебника вы добавили в шаблон возвращаемое значение. В следующем учебнике вы узнаете, как экспортировать шаблон и использовать его части в шаблоне.

> [!div class="nextstepaction"]
> [Учебник по использованию экспортированного шаблона из портала Azure](template-tutorial-export-template.md)
