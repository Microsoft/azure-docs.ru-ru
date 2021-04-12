---
title: Создание функции C# из командной строки (Функции Azure)
description: Сведения о том, как создать функцию C# из командной строки, а затем опубликовать локальный проект в бессерверном размещении в Функциях Azure.
ms.date: 10/03/2020
ms.topic: quickstart
ms.custom:
- devx-track-csharp
- devx-track-azurecli
adobe-target: true
adobe-target-activity: DocsExp–386541–A/B–Enhanced-Readability-Quickstarts–2.19.2021
adobe-target-experience: Experience B
adobe-target-content: ./create-first-function-cli-csharp-ieux
ms.openlocfilehash: 08b1f2b112542a4b4772744d122ce1260b0edffd
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101704775"
---
# <a name="quickstart-create-a-c-function-in-azure-from-the-command-line"></a>Краткое руководство. Создание функции C# в Azure из командной строки

[!INCLUDE [functions-language-selector-quickstart-cli](../../includes/functions-language-selector-quickstart-cli.md)]

Из этой статьи вы узнаете, как создать функцию класса C# на основе библиотеки, которая отвечает на HTTP-запросы, используя инструменты командной строки. После тестирования кода в локальной среде его необходимо развернуть в бессерверной среде Функций Azure.

Выполнение этого краткого руководства предполагает небольшую дополнительную плату в несколько центов США в учетной записи Azure.

Существует также версия этой статьи для [Visual Studio Code](create-first-function-vs-code-csharp.md).

## <a name="configure-your-local-environment"></a>Настройка локальной среды

Перед началом работы убедитесь, что у вас есть такие компоненты.

+ Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.

+ Пакет [SDK для .NET Core 3.1](https://www.microsoft.com/net/download).

+ [Azure Functions Core Tools](functions-run-local.md#v2) версии 3.x.

+ Один из следующих инструментов для создания ресурсов Azure:

    + [Azure CLI](/cli/azure/install-azure-cli) версии 2.4 или более поздней.

    + [Azure PowerShell](/powershell/azure/install-az-ps) версии 5.0 или более поздней.

### <a name="prerequisite-check"></a>Проверка предварительных условий

Убедитесь, что предварительные требования (зависят от того, используете ли вы Azure CLI или Azure PowerShell для создания ресурсов Azure) выполнены:

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

+ В окне терминала или командном окне запустите `func --version`, чтобы убедиться, что используется версия Azure Functions Core Tools 3.x.

+ Выполните команду `az --version`, чтобы убедиться, что используется версия Azure CLI 2.4 или более поздняя.

+ Выполните команду `az login`, чтобы войти в Azure и проверить активную подписку.

+ Выполните команду `dotnet --list-sdks`, чтобы проверить, установлен ли пакет SDK для .NET Core версии 3.1.x.

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

+ В окне терминала или командном окне запустите `func --version`, чтобы убедиться, что используется версия Azure Functions Core Tools 3.x.

+ Выполните команду `(Get-Module -ListAvailable Az).Version` и убедитесь, что используется версия 5.0 или более поздняя. 

+ Выполните команду `Connect-AzAccount`, чтобы войти в Azure и проверить активную подписку.

+ Выполните команду `dotnet --list-sdks`, чтобы проверить, установлен ли пакет SDK для .NET Core версии 3.1.x.

---

## <a name="create-a-local-function-project"></a>Создание локального проекта службы "Функции"

В Функциях Azure проект функций представляет собой контейнер для одной или нескольких отдельных функций, каждая из которых реагирует на конкретный триггер. Все функции в проекте совместно используют те же локальные конфигурации и конфигурации размещения. В этом разделе вы создадите проект функций, содержащий одну функцию.

1. Выполните команду `func init`, как показано ниже, чтобы создать проект функций в папке с именем *LocalFunctionProj* с указанной средой выполнения:  

    ```csharp
    func init LocalFunctionProj --dotnet
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

    Команда `func new` создает файл кода HttpExample.cs.

### <a name="optional-examine-the-file-contents"></a>Проверка содержимого файла (дополнительно)

При необходимости можно сразу перейти к [локальному запуску функции](#run-the-function-locally) и просмотреть содержимое файла позже.

#### <a name="httpexamplecs"></a>HttpExample.cs

Файл *HttpExample.cs* содержит метод `Run`, получающий данные запроса в переменной `req` — это запрос [HttpRequest](/dotnet/api/microsoft.aspnetcore.http.httprequest), дополненный атрибутом **HttpTriggerAttribute**, который определяет поведение триггера.

:::code language="csharp" source="~/functions-docs-csharp/http-trigger-template/HttpExample.cs":::

Возвращаемый объект — это [ActionResult](/dotnet/api/microsoft.aspnetcore.mvc.actionresult), который возвращает ответное сообщение в виде [OkObjectResult](/dotnet/api/microsoft.aspnetcore.mvc.okobjectresult) (200) или [BadRequestObjectResult](/dotnet/api/microsoft.aspnetcore.mvc.badrequestobjectresult) (400). Дополнительные сведения см. в статье [Триггеры и привязки HTTP в службе "Функции Azure"](./functions-bindings-http-webhook.md?tabs=csharp).

[!INCLUDE [functions-run-function-test-local-cli](../../includes/functions-run-function-test-local-cli.md)]

[!INCLUDE [functions-create-azure-resources-cli](../../includes/functions-create-azure-resources-cli.md)]

4. Создайте приложение-функцию в Azure:

    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
        
    ```azurecli
    az functionapp create --resource-group AzureFunctionsQuickstart-rg --consumption-plan-location westeurope --runtime dotnet --functions-version 3 --name <APP_NAME> --storage-account <STORAGE_NAME>
    ```
    
    Чтобы создать приложение-функцию в Azure, выполните команду [az functionapp create](/cli/azure/functionapp#az_functionapp_create). 
    
    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)
    
    ```azurepowershell
    New-AzFunctionApp -Name <APP_NAME> -ResourceGroupName AzureFunctionsQuickstart-rg -StorageAccount <STORAGE_NAME> -Runtime dotnet -FunctionsVersion 3 -Location 'West Europe'
    ```
    
    Чтобы создать приложение-функцию в Azure, выполните командлет [New-AzFunctionApp](/powershell/module/az.functions/new-azfunctionapp). 
    
    ---
    
    В предыдущем примере замените `<STORAGE_NAME>` именем учетной записи, использованной на предыдущем шаге, и измените `<APP_NAME>` на глобально уникальное имя, подходящее вам. `<APP_NAME>` также является доменом DNS по умолчанию для приложения-функции. 
    
    Эта команда создает приложение-функцию, работающее в указанной языковой среде выполнения в рамках [плана использования Функций Azure](consumption-plan.md), который не предусматривает плату за объем, используемый здесь. Эта команда также подготавливает связанный экземпляр Application Insights Azure в той же группе ресурсов. Его можно использовать для мониторинга приложения-функции и просмотра журналов. Дополнительные сведения см. в разделе [Мониторинг функций Azure](functions-monitoring.md). Этот экземпляр не создает затраты, пока вы не активируете его.

[!INCLUDE [functions-publish-project-cli](../../includes/functions-publish-project-cli.md)]

[!INCLUDE [functions-run-remote-azure-cli](../../includes/functions-run-remote-azure-cli.md)]

[!INCLUDE [functions-streaming-logs-cli-qs](../../includes/functions-streaming-logs-cli-qs.md)]

[!INCLUDE [functions-cleanup-resources-cli](../../includes/functions-cleanup-resources-cli.md)]

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Подключение Функций Azure к службе хранилища Azure с помощью средств командной строки]

[Подключение Функций Azure к службе хранилища Azure с помощью средств командной строки]: functions-add-output-binding-storage-queue-cli.md?pivots=programming-language-csharp
