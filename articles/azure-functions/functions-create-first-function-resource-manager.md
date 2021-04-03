---
title: Создание первой функции с помощью шаблонов Azure Resource Manager
description: Узнайте, как создать и развернуть в Azure простую бессерверную функцию, активируемую по HTTP, с помощью шаблона Azure Resource Manager (шаблона ARM).
ms.date: 3/5/2020
ms.topic: quickstart
ms.service: azure-functions
ms.custom: subject-armqs
ms.openlocfilehash: 1e623405faa89ff41eccdaa57578bc8ac94cd78c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "93422830"
---
# <a name="quickstart-create-and-deploy-azure-functions-resources-from-an-arm-template"></a>Краткое руководство. Создание и развертывание ресурсов Функций Azure с помощью шаблона ARM

В этой статье вы примените шаблон Azure Resource Manager (шаблон ARM) для создания функции, которая отвечает на HTTP-запросы. 

Выполнение этого краткого руководства предполагает небольшую дополнительную плату в несколько центов США в учетной записи Azure. 

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-function-app-create-dynamic%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

### <a name="azure-account"></a>Учетная запись Azure 

Для начала работы вам потребуется учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/) бесплатно.

### <a name="create-a-local-functions-project"></a>Создание локального проекта службы "Функции"

Для работы с этой статьей нужен локальный проект кода для функций, который будет выполняться в создаваемых ресурсах Azure. Если вы не создадите проект для публикации заранее, вам не удастся выполнить инструкции в разделе о развертывании далее в этой статье. 

Выберите одну из следующих вкладок, перейдите по ссылке и выполните действия в том разделе, чтобы создать приложение-функцию на соответствующем языке:

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Создайте локальный проект функций на выбранном языке в Visual Studio Code:  

+ [C#](create-first-function-vs-code-csharp.md)
+ [Java](create-first-function-vs-code-java.md)
+ [JavaScript](create-first-function-vs-code-node.md)
+ [PowerShell](create-first-function-vs-code-powershell.md)
+ [Python](create-first-function-vs-code-python.md)
+ [TypeScript](create-first-function-vs-code-typescript.md)

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[Создание локального проекта функций в Visual Studio](functions-create-your-first-function-visual-studio.md#create-a-function-app-project)

# <a name="command-line"></a>[Командная строка](#tab/command-line)

Создайте локальный проект функций на выбранном языке из командной строки:

+ [C#](create-first-function-cli-csharp.md)
+ [Java](create-first-function-cli-java.md)
+ [JavaScript](create-first-function-cli-node.md)
+ [PowerShell](create-first-function-cli-powershell.md)
+ [Python](create-first-function-cli-python.md)
+ [TypeScript](create-first-function-cli-typescript.md)

---

Завершив создание проекта в локальной среде, создайте в Azure необходимые ресурсы для выполнения этой функции. 

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-function-app-create-dynamic/).

:::code language="json" source="~/quickstart-templates/101-function-app-create-dynamic/azuredeploy.json":::

Этот шаблон создает следующие четыре ресурса Azure:

+ [**Microsoft.Storage/storageAccounts**](/azure/templates/microsoft.storage/storageaccounts) — учетная запись хранения Azure, которая нужна для работы Функций Azure;
+ [**Microsoft.Web/serverfarms**](/azure/templates/microsoft.web/serverfarms) — план размещения для потребления бессерверных ресурсов, которые использует приложение-функция.
+ [**Microsoft.Web/sites**](/azure/templates/microsoft.web/sites) — собственно приложение-функция.
+ [**microsoft.insights/components**](/azure/templates/microsoft.insights/components) — экземпляр Application Insights для мониторинга.

## <a name="deploy-the-template"></a>Развертывание шаблона

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
```azurecli-interactive
read -p "Enter a resource group name that is used for generating resource names:" resourceGroupName &&
read -p "Enter the location (like 'eastus' or 'northeurope'):" location &&
templateUri="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-function-app-create-dynamic/azuredeploy.json" &&
az group create --name $resourceGroupName --location "$location" &&
az deployment group create --resource-group $resourceGroupName --template-uri  $templateUri &&
echo "Press [ENTER] to continue ..." &&
read
```
# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter a resource group name that is used for generating resource names"
$location = Read-Host -Prompt "Enter the location (like 'eastus' or 'northeurope')"
$templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-function-app-create-dynamic/azuredeploy.json"

New-AzResourceGroup -Name $resourceGroupName -Location "$location"
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri

Read-Host -Prompt "Press [ENTER] to continue ..."
```
---

## <a name="validate-the-deployment"></a>Проверка развертывания

Далее опубликуйте проект в Azure и вызовите конечную точку HTTP функции, чтобы проверить ресурсы, созданные для размещения приложения-функции.

### <a name="publish-the-function-project-to-azure"></a>Публикация проекта функций в Azure

Чтобы опубликовать проект на новых ресурсах Azure, выполните следующие шаги.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE [functions-republish-vscode](../../includes/functions-republish-vscode.md)]

Из выходных данных скопируйте URL-адрес HTTP-триггера. Он потребуется для тестирования функции, выполняемой в Azure. 

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

1. Щелкните правой кнопкой мыши проект в **обозревателе решений** и выберите пункт **Опубликовать**.

1. В разделе **Выберите целевой объект публикации** выберите **План использования Функций Azure** с помощью действия **Выбрать существующий**, а затем щелкните **Создать профиль**.

    :::image type="content" source="media/functions-create-first-function-arm/choose-publish-target-visual-studio.png" alt-text="Выбор существующего целевого объекта публикации":::

1. Выберите **подписку**, разверните группу ресурсов, выберите приложение-функцию и щелкните **ОК**.

1. После завершения публикации скопируйте **URL-адрес сайта**.

    :::image type="content" source="media/functions-create-first-function-arm/publish-summary-site-url.png" alt-text="Копирование URL-адреса сайта из сводки публикации":::

1. Добавьте путь `/api/<FUNCTION_NAME>?name=Functions`, где `<FUNCTION_NAME>` соответствует имени функции. URL-адрес для вызова функции, активируемой HTTP-запросом, указывается в следующем формате:

    `http://<APP_NAME>.azurewebsites.net/api/<FUNCTION_NAME>?name=Functions`

Используйте этот URL-адрес для проверки выполняющейся в Azure функции, активируемой по HTTP.

# <a name="command-line"></a>[Командная строка](#tab/command-line)

Чтобы опубликовать локальный код в виде приложения-функции в Azure, используйте команду `publish`:

```cmd
func azure functionapp publish <FUNCTION_APP_NAME>
```

В нашем примере замените `<FUNCTION_APP_NAME>` именем реального приложения-функции. Возможно, потребуется еще раз выполнить вход с помощью `az login`. 

Из выходных данных скопируйте URL-адрес HTTP-триггера. Он потребуется для тестирования функции, выполняемой в Azure.

---

### <a name="invoke-the-function-on-azure"></a>Вызов функции в Azure

Вставьте скопированный URL-адрес для HTTP-запроса в адресную строку браузера, обязательно добавьте строку запроса `name` в формате `?name=Functions` в конец этого URL-адреса и выполните запрос. 

Вы должны получить примерно следующий ответ:

<pre>Hello Functions!</pre>

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы намерены перейти к следующему шагу и добавить выходную привязку очереди службы хранилища Azure, можете сохранить все ресурсы, которые пригодятся на этом этапе.

В противном случае используйте следующую команду, чтобы удалить группу ресурсов и все содержащиеся в ней ресурсы и избежать дополнительных расходов.

```azurecli
az group delete --name <RESOURCE_GROUP_NAME>
```

Замените `<RESOURCE_GROUP_NAME>` именем своей группы ресурсов.

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы опубликовали первую функцию, дополните ее выходной привязкой.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

> [!div class="nextstepaction"]
> [Подключение Функций Azure к службе хранилища Azure с помощью средств командной строки](functions-add-output-binding-storage-queue-vs-code.md)

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

> [!div class="nextstepaction"]
> [Подключение Функций Azure к службе хранилища Azure с помощью средств командной строки](functions-add-output-binding-storage-queue-vs.md)

# <a name="command-line"></a>[Командная строка](#tab/command-line)

> [!div class="nextstepaction"]
> [Подключение Функций Azure к службе хранилища Azure с помощью средств командной строки](functions-add-output-binding-storage-queue-cli.md)

---
