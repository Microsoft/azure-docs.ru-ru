---
title: Создание функции JavaScript из командной строки (Функции Azure)
description: Узнайте, как создать функцию JavaScript в командной строке, а затем опубликовать локальный проект Node.js в бессерверном размещении в Функциях Azure.
ms.date: 11/03/2020
ms.topic: quickstart
ms.custom: devx-track-azurecli
ms.openlocfilehash: b8db78e56087e7cb777d1aa85391d4b6ac2aae27
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107787545"
---
# <a name="quickstart-create-a-javascript-function-in-azure-from-the-command-line"></a>Краткое руководство. Создание функции JavaScript в Azure из командной строки


[!INCLUDE [functions-language-selector-quickstart-cli](../../includes/functions-language-selector-quickstart-cli.md)]

В этой статье используются средства командной строки для создания функции JavaScript, которая отвечает на HTTP-запросы. После тестирования кода в локальной среде его необходимо развернуть в бессерверной среде Функций Azure. 

Выполнение этого краткого руководства предполагает небольшую дополнительную плату в несколько центов США в учетной записи Azure.

Существует также версия этой статьи для [Visual Studio Code](create-first-function-vs-code-node.md).

## <a name="configure-your-local-environment"></a>Настройка локальной среды

Перед началом работы убедитесь, что у вас есть такие компоненты.

+ Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.

+ [Azure Functions Core Tools](./functions-run-local.md#v2) версии 3.x.

+ Один из следующих инструментов для создания ресурсов Azure:

    + [Azure CLI](/cli/azure/install-azure-cli) версии 2.4 или более поздней.

    + [Azure PowerShell](/powershell/azure/install-az-ps) версии 5.0 или более поздней.

+ [Node.js](https://nodejs.org/) версии 12. Кроме того, поддерживается Node.js версии 10.

### <a name="prerequisite-check"></a>Проверка предварительных условий

Убедитесь, что предварительные требования (зависят от того, используете ли вы Azure CLI или Azure PowerShell для создания ресурсов Azure) выполнены:

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

+ В окне терминала или командном окне запустите `func --version`, чтобы убедиться, что используется версия Azure Functions Core Tools 3.x.

+ Выполните команду `az --version`, чтобы убедиться, что используется версия Azure CLI 2.4 или более поздняя.

+ Выполните команду `az login`, чтобы войти в Azure и проверить активную подписку.

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

+ В окне терминала или командном окне запустите `func --version`, чтобы убедиться, что используется версия Azure Functions Core Tools 3.x.

+ Выполните команду `(Get-Module -ListAvailable Az).Version` и убедитесь, что используется версия 5.0 или более поздняя. 

+ Выполните команду `Connect-AzAccount`, чтобы войти в Azure и проверить активную подписку.

---

## <a name="create-a-local-function-project"></a>Создание локального проекта службы "Функции"

В Функциях Azure проект функций представляет собой контейнер для одной или нескольких отдельных функций, каждая из которых реагирует на конкретный триггер. Все функции в проекте совместно используют те же локальные конфигурации и конфигурации размещения. В этом разделе вы создадите проект функций, содержащий одну функцию.

1. Выполните команду `func init`, как показано ниже, чтобы создать проект функций в папке с именем *LocalFunctionProj* с указанной средой выполнения:  

    ```console
    func init LocalFunctionProj --javascript
    ```

1. Перейдите в папку проекта:

    ```console
    cd LocalFunctionProj
    ```

    Эта папка содержит различные файлы проекта, в том числе файлы конфигурации [local.settings.json](functions-run-local.md#local-settings-file) и [host.json](functions-host-json.md). Файл *local.settings.json* может содержать секреты, скачанные из Azure, поэтому файл по умолчанию исключен из системы управления версиями в *GITIGNORE*-файле.

1. Добавьте функцию в проект с помощью приведенной ниже команды, где аргумент `--name` — уникальное имя функции (HttpExample), а аргумент `--template` позволяет указать триггер функции (HTTP).

    ```console
    func new --name HttpExample --template "HTTP trigger" --authlevel "anonymous"
    ```  
    
    Команда `func new` создает вложенную папку с именем функции. Эта папка содержит файл кода на выбранном для проекта языке и файл конфигурации *function.json*.
    
### <a name="optional-examine-the-file-contents"></a>Проверка содержимого файла (дополнительно)

При необходимости можно сразу перейти к [локальному запуску функции](#run-the-function-locally) и просмотреть содержимое файла позже.

#### <a name="indexjs"></a>Файл index.js

Файл *index.js* экспортирует функцию, которая активируется в соответствии с конфигурацией в *function.json*.

:::code language="javascript" source="~/functions-quickstart-templates/Functions.Templates/Templates/HttpTrigger-JavaScript/index.js":::

Для триггера HTTP функция получает данные запроса в переменной `req`, как определено в файле *function.json*. Ответ определяется так же, как `res` в *function.json*. Доступ к нему можно получить с помощью `context.res`. Дополнительные сведения см. в статье [Триггеры и привязки HTTP в службе "Функции Azure"](./functions-bindings-http-webhook.md?tabs=javascript).

#### <a name="functionjson"></a>function.json

*function.json* — это файл конфигурации, который определяет входные и выходные данные для функции `bindings`, в том числе тип триггера. 

:::code language="json" source="~/functions-quickstart-templates/Functions.Templates/Templates/HttpTrigger-JavaScript/function.json":::

Для каждой привязки требуется направление, тип и уникальное имя. В HTTP-триггере есть входная привязка типа [`httpTrigger`](functions-bindings-http-webhook-trigger.md) и выходная привязка типа [`http`](functions-bindings-http-webhook-output.md).

[!INCLUDE [functions-run-function-test-local-cli](../../includes/functions-run-function-test-local-cli.md)]

[!INCLUDE [functions-create-azure-resources-cli](../../includes/functions-create-azure-resources-cli.md)]

4. Создайте приложение-функцию в Azure:

    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
        
    ```azurecli
    az functionapp create --resource-group AzureFunctionsQuickstart-rg --consumption-plan-location westeurope --runtime node --runtime-version 12 --functions-version 3 --name <APP_NAME> --storage-account <STORAGE_NAME>
    ```
    
    Чтобы создать приложение-функцию в Azure, выполните команду [az functionapp create](/cli/azure/functionapp#az_functionapp_create). Если вы используете Node.js 10, также измените `--runtime-version` на `10`.
    
    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)
    
    ```azurepowershell
    New-AzFunctionApp -Name <APP_NAME> -ResourceGroupName AzureFunctionsQuickstart-rg -StorageAccount <STORAGE_NAME> -Runtime node -RuntimeVersion 12 -FunctionsVersion 3 -Location 'West Europe'
    ```
    
    Чтобы создать приложение-функцию в Azure, выполните командлет [New-AzFunctionApp](/powershell/module/az.functions/new-azfunctionapp). Если вы используете Node.js 10, измените `-RuntimeVersion` на `10`.
    
    ---
    
    В предыдущем примере замените `<STORAGE_NAME>` именем учетной записи, использованной на предыдущем шаге, и измените `<APP_NAME>` на глобально уникальное имя, подходящее вам. `<APP_NAME>` также является доменом DNS по умолчанию для приложения-функции. 
    
    Эта команда создает приложение-функцию, работающее в указанной языковой среде выполнения в рамках [плана использования Функций Azure](consumption-plan.md), который не предусматривает плату за объем, используемый здесь. Эта команда также подготавливает связанный экземпляр Application Insights Azure в той же группе ресурсов. Его можно использовать для мониторинга приложения-функции и просмотра журналов. Дополнительные сведения см. в разделе [Мониторинг функций Azure](functions-monitoring.md). Этот экземпляр не создает затраты, пока вы не активируете его.

[!INCLUDE [functions-publish-project-cli](../../includes/functions-publish-project-cli.md)]

[!INCLUDE [functions-run-remote-azure-cli](../../includes/functions-run-remote-azure-cli.md)]

[!INCLUDE [functions-streaming-logs-cli-qs](../../includes/functions-streaming-logs-cli-qs.md)]

[!INCLUDE [functions-cleanup-resources-cli](../../includes/functions-cleanup-resources-cli.md)]

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Подключение Функций Azure к службе хранилища Azure с помощью средств командной строки](functions-add-output-binding-storage-queue-cli.md?pivots=programming-language-javascript)