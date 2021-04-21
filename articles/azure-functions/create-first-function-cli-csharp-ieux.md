---
title: Создание функции C# из командной строки (Функции Azure)
description: Сведения о том, как создать функцию C# из командной строки, а затем опубликовать локальный проект в бессерверном размещении в Функциях Azure.
ms.date: 10/03/2020
ms.topic: quickstart
ms.custom:
- devx-track-csharp
- devx-track-azurecli
ROBOTS: NOINDEX,NOFOLLOW
ms.openlocfilehash: 2d03f8c820e0a8b6a19394649db66f8028b62781
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107768801"
---
# <a name="quickstart-create-a-c-function-in-azure-from-the-command-line"></a>Краткое руководство. Создание функции C# в Azure из командной строки

> [!div class="op_single_selector" title1="Выберите язык функции: "]
> - [C#](create-first-function-cli-csharp-ieux.md)
> - [Java](create-first-function-cli-java.md)
> - [JavaScript](create-first-function-cli-node.md)
> - [PowerShell](create-first-function-cli-powershell.md)
> - [Python](create-first-function-cli-python.md)
> - [TypeScript](create-first-function-cli-typescript.md)

Из этой статьи вы узнаете, как создать функцию класса C# на основе библиотеки, которая отвечает на HTTP-запросы, используя инструменты командной строки. После локального тестирования кода его необходимо развернуть в <abbr title="Вычислительное окружение среды выполнения, в котором все сведения о сервере прозрачны для разработчиков приложений, что упрощает процесс развертывания и управления кодом.">независимая от сервера.</abbr> окружении <abbr title="Служба Azure, которая предоставляет недорогую среду бессерверных вычислений для приложений.">Функции Azure</abbr>.

Выполнение этого краткого руководства предполагает небольшую дополнительную плату в несколько центов США в учетной записи Azure.

Существует также версия этой статьи для [Visual Studio Code](create-first-function-vs-code-csharp.md).

## <a name="1-prepare-your-environment"></a>1. Подготовка среды

+ Получите <abbr title="Профиль, который поддерживает данные для выставления счетов за использование Azure.">account</abbr> Azure с активной <abbr title="Базовая организационная структура, в которой можно управлять ресурсами в Azure, обычно связанными с частным лицом или отделом в пределах организации.">Подписка</abbr>. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.

+ Установите [пакет SDK для .NET Core 3.1](https://www.microsoft.com/net/download).

+ Установите [Azure Functions Core Tools](functions-run-local.md#v2) версии 3.x.

+ Или <abbr title="Набор кроссплатформенных программ командной строки для работы с ресурсами Azure с локального компьютера разработки в качестве альтернативы использованию портала Azure.">Azure CLI</abbr> или диспетчер конфигурации служб <abbr title="Модуль PowerShell, который предоставляет команды для работы с ресурсами Azure с локального компьютера разработки в качестве альтернативы использованию портала Azure.">Azure PowerShell</abbr> для созданий ресурсов Azure:

    + [Azure CLI](/cli/azure/install-azure-cli) версии 2.4 или более поздней.

    + [Azure PowerShell](/powershell/azure/install-az-ps) версии 5.0 или более поздней.

---

### <a name="2-verify-prerequisites"></a>2. проверка предварительных требований;

Убедитесь, что предварительные требования (которые зависят от того, используете вы Azure CLI или Azure PowerShell для создания ресурсов Azure) выполнены:

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

+ В терминале или командном окне выполните `func --version`, чтобы убедиться, что версией <abbr title="Набор программ командной строки для работы с Функциями Azure на локальном компьютере.">Azure Functions Core Tools</abbr> является версия 3.x.

+ **Выполните команду** `az --version`, чтобы убедиться, что используется версия Azure CLI 2.4 или более поздняя.

+ **Выполните команду** `az login`, чтобы войти в Azure и проверить активную подписку.

+ **Выполните команду** `dotnet --list-sdks`, чтобы проверить, установлен ли пакет SDK для .NET Core версии 3.1.x.

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

+**Выполните команду** `func --version`, чтобы убедиться, что используется версия Azure Functions Core Tools 3.x.

+ **Выполните команду** `(Get-Module -ListAvailable Az).Version` и убедитесь, что используется версия 5.0 или более поздняя. 

+ **Выполните команду** `Connect-AzAccount`, чтобы войти в Azure и проверить активную подписку.

+ **Выполните команду** `dotnet --list-sdks`, чтобы проверить, установлен ли пакет SDK для .NET Core версии 3.1.x.

---

## <a name="3-create-a-local-function-project"></a>3. Создание локального проекта службы "Функции"

С помощью инструкций из этого раздела вы создадите локальный <abbr title="Логический контейнер для одной или нескольких отдельных функций, которые можно развертывать и администрировать вместе.">проект Функций Azure</abbr> на C#. Каждая функция в проекте реагирует на определенный <abbr title="Событие, вызывающее код функции, например HTTP-запрос, сообщение в очереди или определенное время.">триггер</abbr>.

1. Выполните команду `func init`, чтобы создать проект функций в папке с именем *LocalFunctionProj* с указанной средой выполнения:  

    ```csharp
    func init LocalFunctionProj --dotnet
    ```

1. **Выполните команду** cd LocalFunctionProj для перехода к <abbr title="Эта папка содержит различные файлы проекта, в том числе файлы конфигурации local.settings.json и host.json. Файл local.settings.json может содержать секреты, скачанные из Azure, поэтому файл по умолчанию исключен из системы управления версиями в GITIGNORE-файле.">Папка проекта</abbr>.

    ```console
    cd LocalFunctionProj
    ```
    <br/>

1. Добавьте функцию в проект, выполнив следующую команду:
    
    ```console
    func new --name HttpExample --template "HTTP trigger" --authlevel "anonymous"
    ``` 
    Аргумент `--name` — это уникальное имя функции (HttpExample).

    Аргумент `--template` задает триггер функции (HTTP).

    
    <br/>   
    <details>  
    <summary><strong>Необязательно. Код для HttpExample.cs</strong></summary>  
    
    Файл *HttpExample.cs* содержит метод `Run`, получающий данные запроса в переменной `req` — это запрос [HttpRequest](/dotnet/api/microsoft.aspnetcore.http.httprequest), дополненный атрибутом **HttpTriggerAttribute**, который определяет поведение триггера.

    :::code language="csharp" source="~/functions-docs-csharp/http-trigger-template/HttpExample.cs":::
        
    Возвращаемый объект — это [ActionResult](/dotnet/api/microsoft.aspnetcore.mvc.actionresult), который возвращает ответное сообщение в виде [OkObjectResult](/dotnet/api/microsoft.aspnetcore.mvc.okobjectresult) (200) или [BadRequestObjectResult](/dotnet/api/microsoft.aspnetcore.mvc.badrequestobjectresult) (400). Дополнительные сведения см. в статье [Триггеры и привязки HTTP в службе "Функции Azure"](./functions-bindings-http-webhook.md?tabs=csharp).  
    </details>

<br/>

---

## <a name="4-run-the-function-locally"></a>4. Локальное выполнение функции

1. Выполните функцию, запустив локальное хост-приложение среды выполнения Функций Azure из папки *LocalFunctionProj*:

    ```
    func start
    ```

    Ближе к концу выходных данных появятся следующие строки: 

    <pre class="is-monospace is-size-small has-padding-medium has-background-tertiary has-text-tertiary-invert">
    ...

    Now listening on: http://0.0.0.0:7071
    Application started. Press Ctrl+C to shut down.

    Http Functions:

            HttpExample: [GET,POST] http://localhost:7071/api/HttpExample
    ...

    </pre>

    <br/>
    <details>
    <summary><strong>Я не вижу HttpExample в выходных данных</strong></summary>

    Если HttpExample не отображается, скорее всего, вы запустили основное приложение из папки, отличной от корневой папки проекта. В этом случае остановите основное приложение комбинацией клавиш <kbd>Ctrl+C</kbd>, перейдите в корневую папку проекта и снова выполните указанную выше команду.
    </details>

1. Скопируйте URL-адрес функции **HttpExample** из этих выходных данных в браузер и добавьте строку запроса **?name=<YOUR_NAME>** , сформировав полный URL-адрес, например **http://localhost:7071/api/HttpExample?name=Functions** . В браузере должно отобразиться сообщение, например **Hello Functions**:

    ![Получившаяся функция выполняется локально в браузере](../../includes/media/functions-run-function-test-local-cli/function-test-local-browser.png)


1. Нажмите <kbd>Ctrl+C</kbd> и <kbd>y</kbd>, чтобы остановить узел функций.

<br/>

---
    
## <a name="5-create-supporting-azure-resources-for-your-function"></a>5. Создание вспомогательных ресурсов Azure для функции

Прежде чем развернуть код функции в Azure, необходимо создать <abbr title="Логический контейнер для связанных ресурсов Azure, которыми можно управлять как единым целым.">resource group</abbr>, <abbr title="Учетная запись, которая содержит все объекты данных хранилища Azure. Учетная запись хранения предоставляет пространство имен для данных вашего хранилища.">запись хранения Azure</abbr>и <abbr title="Облачный ресурс, в котором размещаются бессерверные функции Azure и который предоставляет базовое вычислительное окружение, в котором выполняются функции.">приложение-функцию,</abbr> выполнив следующие команды:

1. Войдите в Azure, если вы еще этого не сделали:

    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
    ```azurecli
    az login
    ```


    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell) 
    ```azurepowershell
    Connect-AzAccount
    ```


    ---

1. Создайте группу ресурсов с именем `AzureFunctionsQuickstart-rg` в регионе `westeurope`. 

    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

    ```azurecli
    az group create --name AzureFunctionsQuickstart-rg --location westeurope
    ```

    Чтобы создать группу ресурсов, выполните команду [az group create](/cli/azure/group#az_group_create). Группу ресурсов и ресурсы целесообразно создавать в <abbr title="Географическая ссылка на конкретный центр обработки данных Azure, в котором распределены ресурсы.">region</abbr> , расположенном неподалеку, с помощью доступного региона, возвращенного из команды `az account list-locations`.

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

    ```azurepowershell
    New-AzResourceGroup -Name AzureFunctionsQuickstart-rg -Location westeurope
    ```


    ---

    Вы не можете разместить приложения Windows и Linux в одной группе ресурсов. Если у вас есть группа ресурсов `AzureFunctionsQuickstart-rg` с приложением-функцией Windows или веб-приложением, необходимо использовать другую группу ресурсов.
    
1. В группе ресурсов и регионе создайте учетную запись хранения Azure общего назначения:

    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

    ```azurecli
    az storage account create --name <STORAGE_NAME> --location westeurope --resource-group AzureFunctionsQuickstart-rg --sku Standard_LRS
    ```


    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

    ```azurepowershell
    New-AzStorageAccount -ResourceGroupName AzureFunctionsQuickstart-rg -Name <STORAGE_NAME> -SkuName Standard_LRS -Location westeurope
    ```


    ---

    Замените `<STORAGE_NAME>` именем, которое понравится вам, и <abbr title="Имя должно быть уникальным для всех учетных записей хранения, используемых всеми клиентами Azure глобально. Например, можно использовать сочетание собственного имени или названия компании с именем приложения и числовым идентификатором, например contosobizappstorage20">будет уникальным для службы хранилища Azure</abbr>. Имена должны содержать от трех до 24 символов и только в нижнем регистре. `Standard_LRS` указывает учетную запись общего назначения, которая [поддерживается Функциями](storage-considerations.md#storage-account-requirements).


1. Создайте приложение-функцию в Azure.
**Замените** <STORAGE_NAME>** именем на предыдущем шаге.
**Замените**<APP_NAME> глобально уникальным именем.

    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
        
    ```azurecli
    az functionapp create --resource-group AzureFunctionsQuickstart-rg --consumption-plan-location westeurope --runtime dotnet --functions-version 3 --name <APP_NAME> --storage-account <STORAGE_NAME>
    ```
    
    
    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)
    
    ```azurepowershell
    New-AzFunctionApp -Name <APP_NAME> -ResourceGroupName AzureFunctionsQuickstart-rg -StorageAccount <STORAGE_NAME> -Runtime dotnet -FunctionsVersion 3 -Location 'West Europe'
    ```
    
    
    ---
    
    Замените имя `<STORAGE_NAME>` на имя учетной записи, которое вы использовали на предыдущем шаге.

    Замените имя `<APP_NAME>` на <abbr title="Уникальное имя, которое не должно встречаться ни у одного клиента Azure глобально. Например, можно использовать сочетание собственного имени или названия организации с именем приложения и числовым идентификатором, например contoso-bizapp-func-20.">уникальное имя</abbr>. `<APP_NAME>` также является доменом DNS по умолчанию для приложения-функции. 

    <br/>
    <details>
    <summary><strong>Сколько стоят ресурсы, подготовленные в Azure?</strong></summary>

    Эта команда создает приложение-функцию, работающее в указанной языковой среде выполнения в рамках [плана использования Функций Azure](consumption-plan.md), который не предусматривает плату за объем, используемый здесь. Эта команда также подготавливает связанный экземпляр Application Insights Azure в той же группе ресурсов. Его можно использовать для мониторинга приложения-функции и просмотра журналов. Дополнительные сведения см. в разделе [Мониторинг функций Azure](functions-monitoring.md). Этот экземпляр не создает затраты, пока вы не активируете его.
    </details>

<br/>

---

## <a name="6-deploy-the-function-project-to-azure"></a>6. Развертывание проекта функций в Azure


**Скопируйте** команду func azure funtionapp publish <APP_NAME> в ваш терминал и **замените** `<APP_NAME>` названием вашего приложения.
**Выполнить**

```console
func azure functionapp publish <APP_NAME>
```

Команда `publish` выведет результаты, аналогичные приведенным ниже выходным данным (усеченным для простоты):

<pre class="is-monospace is-size-small has-padding-medium has-background-tertiary has-text-tertiary-invert">
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
        Invoke url: https://msdocs-azurefunctions-qs.azurewebsites.net/api/httpexample
</pre>

<br/>

---

## <a name="7-invoke-the-function-on-azure"></a>7. Вызов функции в Azure

Скопируйте полный URL-адрес вызова **Invoke URL**, показанный в выходных данных команды `publish`, в адресную строку браузера. **Добавьте** параметр запроса **&name=Functions**. 

![Выходные данные функции, выполняемой в Azure в браузере](../../includes/media/functions-run-remote-azure-cli/function-test-cloud-browser.png)

<br/>

---

## <a name="8-clean-up-resources"></a>8. Очистка ресурсов

Если вы планируете перейти к [следующему шагу](#next-steps) и добавляете выходную привязку очереди хранения службы хранилища Azure, <abbr title="Декларативное соединение между функцией и другими ресурсами. Входная привязка предоставляет данные для функции, а выходная привязка предоставляет данные из функции для других ресурсов.">binding</abbr>, сохраните все ресурсы в одном месте, поскольку на их основе будут создаваться другие элементы.

В противном случае используйте следующую команду, чтобы удалить группу ресурсов и все содержащиеся в ней ресурсы и избежать дополнительных расходов.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az group delete --name AzureFunctionsQuickstart-rg
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell
Remove-AzResourceGroup -Name AzureFunctionsQuickstart-rg
```

---

<br/>

---

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Подключение Функций Azure к службе хранилища Azure с помощью средств командной строки](functions-add-output-binding-storage-queue-cli.md?pivots=programming-language-csharp)
