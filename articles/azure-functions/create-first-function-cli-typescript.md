---
title: Создание функции TypeScript из командной строки (Функции Azure)
description: Узнайте, как создать функцию TypeScript в командной строке, а затем опубликовать локальный проект в бессерверном размещении в Функциях Azure.
ms.date: 11/03/2020
ms.topic: quickstart
ms.custom: devx-track-azurecli
ms.openlocfilehash: 488ef9fa3fd5b6c09ed435483dbf8f6fa3eb5bef
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97937201"
---
# <a name="quickstart-create-a-typescript-function-in-azure-from-the-command-line"></a>Краткое руководство. Создание функции TypeScript в Azure из командной строки

[!INCLUDE [functions-language-selector-quickstart-cli](../../includes/functions-language-selector-quickstart-cli.md)]

В этой статье используются средства командной строки для создания функции TypeScript, которая отвечает на HTTP-запросы. После тестирования кода в локальной среде его необходимо развернуть в бессерверной среде Функций Azure.

Выполнение этого краткого руководства предполагает небольшую дополнительную плату в несколько центов США в учетной записи Azure.

Существует также версия этой статьи для [Visual Studio Code](create-first-function-vs-code-typescript.md).

## <a name="configure-your-local-environment"></a>Настройка локальной среды

Перед началом работы убедитесь, что у вас есть такие компоненты.

+ Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.

+ [Azure Functions Core Tools](functions-run-local.md#v2) версии 3.x.

+ Один из следующих инструментов для создания ресурсов Azure:

    + [Azure CLI](/cli/azure/install-azure-cli) версии 2.4 или более поздней.

    + [Azure PowerShell](/powershell/azure/install-az-ps) версии 5.0 или более поздней.

+ [Node.js](https://nodejs.org/), активная версия LTS и версия Maintenance LTS (рекомендуются версии 8.11.1 и 10.14.1).

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
    func init LocalFunctionProj --typescript
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

#### <a name="indexts"></a>index.ts

Файл *index.ts* экспортирует функцию, которая активируется в соответствии с конфигурацией в *function.json*.

:::code language="typescript" source="~/functions-quickstart-templates/Functions.Templates/Templates/HttpTrigger-TypeScript/index.ts":::

Для триггера HTTP функция получает данные запроса в переменной `req` типа **HttpRequest**, как определено в файле *function.json*. Ответом является объект, определенный как `$return` в файле *function.json*. 

#### <a name="functionjson"></a>function.json

*function.json* — это файл конфигурации, который определяет входные и выходные данные для функции `bindings`, в том числе тип триггера. 

:::code language="json" source="~/functions-quickstart-templates/Functions.Templates/Templates/HttpTrigger-JavaScript/function.json":::

Для каждой привязки требуется направление, тип и уникальное имя. В HTTP-триггере есть входная привязка типа [`httpTrigger`](functions-bindings-http-webhook-trigger.md) и выходная привязка типа [`http`](functions-bindings-http-webhook-output.md).

## <a name="run-the-function-locally"></a>Локальное выполнение функции

1. Выполните функцию, запустив локальное хост-приложение среды выполнения Функций Azure из папки *LocalFunctionProj*:

    ```console
    npm install
    npm start
    ```
    
    Ближе к концу выходных данных появятся следующие строки:
    
    <pre>
    ...
    
    Now listening on: http://0.0.0.0:7071
    Application started. Press Ctrl+C to shut down.
    
    Http Functions:
    
            HttpExample: [GET,POST] http://localhost:7071/api/HttpExample
    ...
    
    </pre>
    
    >[!NOTE]  
    > Если результат HttpExample не похож на пример ниже, скорее всего, вы запустили хост-приложение из папки, отличной от корневой папки проекта. В этом случае остановите хост-приложение клавишами **CTRL**+**C**, перейдите в корневую папку проекта и снова выполните указанную выше команду.

1. Скопируйте URL-адрес функции `HttpExample` из этих выходных данных в браузер и добавьте строку запроса `?name=<your-name>`, сформировав полный URL-адрес, например `http://localhost:7071/api/HttpExample?name=Functions`. В браузере должно отобразиться сообщение, например `Hello Functions`:

    ![Получившаяся функция выполняется локально в браузере](./media/functions-create-first-azure-function-azure-cli/function-test-local-browser.png)
    
    Терминал, в котором вы запустили проект, также выводит данные журнала при выполнении запросов.

1. Когда все будет готово, нажмите клавиши **Ctrl**+**C** и выберите `y`, чтобы отключить хост-приложение функции.

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

## <a name="deploy-the-function-project-to-azure"></a>Развертывание проекта функций в Azure

Прежде чем использовать Core Tools для развертывания проекта в Azure, создайте готовую к работе сборку файлов JavaScript из исходных файлов TypeScript.

1. Следующая команда позволяет подготовить проект TypeScript к развертыванию:

    ```console
    npm run build:production 
    ```

1. Теперь, когда необходимые ресурсы готовы, вы можете развернуть проект локальных функций в приложении-функции в Azure с помощью команды [func azure functionapp publish](functions-run-local.md#project-file-deployment). В следующем примере замените `<APP_NAME>` на имя своего приложения.

    ```console
    func azure functionapp publish <APP_NAME>
    ```
    
    Если отображается сообщение об ошибке Can't find app with name... (Не удалось найти приложение с именем...), подождите несколько секунд и повторите попытку. Возможно, в Azure не полностью инициализировано приложение после выполнения предыдущей команды `az functionapp create`.
    
    Команда publish показывает результаты, аналогичные приведенным ниже (усечены для простоты):
    
    <pre>
    ...
    
    Getting site publishing info...
    Creating archive for current directory...
    Performing remote build for functions project.
    
    ...
    
    Deployment successful.
    Remote build succeeded!
    Syncing triggers...
    Functions in msdocs-azurefunctions-qs:
        HttpExample - [httpTrigger]
            Invoke url: https://msdocs-azurefunctions-qs.azurewebsites.net/api/httpexample?code=KYHrydo4GFe9y0000000qRgRJ8NdLFKpkakGJQfC3izYVidzzDN4gQ==
    </pre>

[!INCLUDE [functions-run-remote-azure-cli](../../includes/functions-run-remote-azure-cli.md)]

[!INCLUDE [functions-streaming-logs-cli-qs](../../includes/functions-streaming-logs-cli-qs.md)]

[!INCLUDE [functions-cleanup-resources-cli](../../includes/functions-cleanup-resources-cli.md)]

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Подключение Функций Azure к службе хранилища Azure с помощью средств командной строки](functions-add-output-binding-storage-queue-cli.md?pivots=programming-language-typescript)
