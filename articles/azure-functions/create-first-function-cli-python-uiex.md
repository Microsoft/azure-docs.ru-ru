---
title: Создание функции Python из командной строки для Функций Azure
description: Узнайте, как создать функцию Python в командной строке и опубликовать локальный проект в бессерверном размещении в Функциях Azure.
ms.date: 11/03/2020
ms.topic: quickstart
ms.custom:
- devx-track-python
- devx-track-azurecli
ROBOTS: NOINDEX,NOFOLLOW
ms.openlocfilehash: 626cff867a336880689373c289087e2332a816ee
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107787455"
---
# <a name="quickstart-create-a-python-function-in-azure-from-the-command-line"></a>Краткое руководство. Создание функции Python в Azure из командной строки

> [!div class="op_single_selector" title1="Выберите язык функции: "]
> - [Python](create-first-function-cli-python.md)
> - [C#](create-first-function-cli-csharp.md)
> - [Java](create-first-function-cli-java.md)
> - [JavaScript](create-first-function-cli-node.md)
> - [PowerShell](create-first-function-cli-powershell.md)
> - [TypeScript](create-first-function-cli-typescript.md)

В этой статье используются средства командной строки для создания функции Python, которая отвечает на HTTP-запросы. После локального тестирования кода его необходимо развернуть в <abbr title="Вычислительное окружение среды выполнения, в котором все сведения о сервере прозрачны для разработчиков приложений, что упрощает процесс развертывания и управления кодом.">независимая от сервера.</abbr> окружении <abbr title="Служба Azure, которая предоставляет недорогое окружение бессерверных вычислений для приложений.">Функции Azure</abbr>.

Выполнение этого краткого руководства предполагает небольшую дополнительную плату в несколько центов США в учетной записи Azure.

Существует также версия этой статьи для [Visual Studio Code](create-first-function-vs-code-python.md).

## <a name="1-configure-your-environment"></a>1. Настройка среды

Перед началом работы убедитесь, что у вас есть такие компоненты.

+ . <abbr title="Профиль, который поддерживает данные для выставления счетов за использование Azure.">account</abbr> Azure с активной <abbr title="Базовая организационная структура, в которой можно управлять ресурсами в Azure, обычно связанными с частным лицом или отделом в пределах организации.">Подписка</abbr>. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.

+ [Azure Functions Core Tools](functions-run-local.md#v2) версии 3.x. 
  
+ Или <abbr title="Набор кроссплатформенных программ командной строки для работы с ресурсами Azure с локального компьютера разработки в качестве альтернативы использованию портала Azure.">Azure CLI</abbr> или диспетчер конфигурации служб <abbr title="Модуль PowerShell, который предоставляет команды для работы с ресурсами Azure с локального компьютера разработки в качестве альтернативы использованию портала Azure.">Azure PowerShell</abbr> для созданий ресурсов Azure:

    + [Azure CLI](/cli/azure/install-azure-cli) версии 2.4 или более поздней.

    + [Azure PowerShell](/powershell/azure/install-az-ps) версии 5.0 или более поздней.

+ [Python 3.8 (64-разрядная версия)](https://www.python.org/downloads/release/python-382/), [Python 3.7 (64-разрядная версия)](https://www.python.org/downloads/release/python-375/), [Python 3.6 (64-разрядная версия)](https://www.python.org/downloads/release/python-368/), которые поддерживаются решением "Функции Azure" версии 3.x.

### <a name="11-prerequisite-check"></a>1.1 Проверка предварительных требований

Убедитесь, что предварительные требования (которые зависят от того, используете вы Azure CLI или Azure PowerShell для создания ресурсов Azure) выполнены:

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

+ В терминале или командном окне выполните `func --version`, чтобы убедиться, что версией <abbr title="Набор программ командной строки для работы с Функциями Azure на локальном компьютере.">Azure Functions Core Tools</abbr> является версия 3.x.

+ Выполните команду `az --version`, чтобы убедиться, что используется версия Azure CLI 2.4 или более поздняя.

+ Выполните команду `az login`, чтобы войти в Azure и проверить активную подписку.

+ Выполните команду `python --version` (в ОС Linux и MacOS) или `py --version` (в ОС Windows), чтобы убедиться, что для Python возвращается версия 3.8.x, 3.7.x или 3.6.x.

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

+ В терминале или командном окне выполните `func --version`, чтобы убедиться, что версией <abbr title="Набор программ командной строки для работы с Функциями Azure на локальном компьютере.">Azure Functions Core Tools</abbr> является версия 3.x.

+ Выполните команду `(Get-Module -ListAvailable Az).Version` и убедитесь, что используется версия 5.0 или более поздняя. 

+ Выполните команду `Connect-AzAccount`, чтобы войти в Azure и проверить активную подписку.

+ Выполните команду `python --version` (в ОС Linux и MacOS) или `py --version` (в ОС Windows), чтобы убедиться, что для Python возвращается версия 3.8.x, 3.7.x или 3.6.x.

---

<br/>

---

## <a name="2-create-and-activate-a-virtual-environment"></a>2. <a name="create-venv"></a>Создание и активация виртуальной среды

В подходящей папке выполните следующие команды, чтобы создать и активировать виртуальную среду с именем `.venv`. Обязательно используйте Python 3.8, 3.7 или 3.6, которые поддерживают Функции Azure.

# <a name="bash"></a>[bash](#tab/bash)

```bash
python -m venv .venv
```

```bash
source .venv/bin/activate
```

Если пакет venv не установлен Python для вашего дистрибутива Linux, выполните следующую команду:

```bash
sudo apt-get install python3-venv
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
py -m venv .venv
```

```powershell
.venv\scripts\activate
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```cmd
py -m venv .venv
```

```cmd
.venv\scripts\activate
```

---

Все последующие команды будут выполняться в этой активированной виртуальной среде. 

<br/>

---

## <a name="3-create-a-local-function-project"></a>3. Создание локального проекта службы "Функции"

С помощью инструкций из этого раздела вы создадите локальный <abbr title="Логический контейнер для одной или нескольких отдельных функций, которые можно развертывать и администрировать вместе.">проект Функций Azure</abbr> на Python. Каждая функция в проекте реагирует на определенный <abbr title="Тип события, вызывающего код функции, например HTTP-запрос, сообщение в очереди или определенное время.">триггер</abbr>.

1. Выполните команду `func init`, чтобы создать проект функций в папке с именем *LocalFunctionProj* с указанной средой выполнения:  

    ```console
    func init LocalFunctionProj --python
    ```

1. Перейдите в папку проекта:

    ```console
    cd LocalFunctionProj
    ```
    
    <br/>
    <details>
    <summary><strong>Что создается в папке LocalFunctionProj?</strong></summary>
    
    Эта папка содержит различные файлы проекта, в том числе файлы конфигурации [local.settings.json](functions-run-local.md#local-settings-file) и [host.json](functions-host-json.md). Файл *local.settings.json* может содержать секреты, скачанные из Azure, поэтому файл по умолчанию исключен из системы управления версиями в *GITIGNORE*-файле.
    </details>

1. Добавьте функцию в проект, выполнив следующую команду:

    ```console
    func new --name HttpExample --template "HTTP trigger" --authlevel "anonymous"
    ```   
    Аргумент `--name` — это уникальное имя функции (HttpExample).

    Аргумент `--template` задает триггер функции (HTTP).
    
    `func new` создает вложенную папку с именем функции, в которой содержится файл *\_\_init\_\_.py* с кодом функции и файлом конфигурации с именем *function.json*.

    <br/>    
    <details>
    <summary><strong>Код для __init__.py</strong></summary>
    
    *\_\_init\_\_.py* содержит функцию Python `main()`, которая активируется в соответствии с конфигурацией в *function.json*.
    
    :::code language="python" source="~/functions-quickstart-templates/Functions.Templates/Templates/HttpTrigger-Python/__init__.py":::
    
    Для триггера HTTP функция получает данные запроса в переменной `req`, как определено в файле *function.json*. `req` — это экземпляр [класса azure.functions.HttpRequest](/python/api/azure-functions/azure.functions.httprequest). Возвращаемый объект, определенный как `$return` в файле *function.json*, — это экземпляр класса [azure.functions.HttpResponse](/python/api/azure-functions/azure.functions.httpresponse). Дополнительные сведения см. в статье [Триггеры и привязки HTTP в службе "Функции Azure"](./functions-bindings-http-webhook.md?tabs=python).
    </details>

    <br/>
    <details>
    <summary><strong>Код для function.json</strong></summary>

    *function.json* — это файл конфигурации, определяющий <abbr title="Декларативные соединения между функцией и другими ресурсами. Входная привязка предоставляет данные для функции, а выходная привязка предоставляет данные из функции для других ресурсов.">входные и выходные привязки</abbr> для функции, включая тип триггера.
    
    При необходимости можно изменить `scriptFile`, чтобы вызывать другой файл Python.
    
    :::code language="json" source="~/functions-quickstart-templates/Functions.Templates/Templates/HttpTrigger-Python/function.json":::
    
    Для каждой привязки требуется направление, тип и уникальное имя. В HTTP-триггере есть входная привязка типа [`httpTrigger`](functions-bindings-http-webhook-trigger.md) и выходная привязка типа [`http`](functions-bindings-http-webhook-output.md).    
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

1. Терминал, в котором вы запустили проект, также выводит данные журнала при выполнении запросов.

1. Когда все будет готово, нажмите клавиши <kbd>Ctrl+C</kbd> и выберите <kbd>y</kbd>, чтобы отключить основное приложение функций.

<br/>

---

## <a name="5-create-supporting-azure-resources-for-your-function"></a>5. Создание вспомогательных ресурсов Azure для функции

Прежде чем развернуть код функции в Azure, необходимо создать <abbr title="Логический контейнер для связанных ресурсов Azure, которыми можно управлять как единым целым.">resource group</abbr>, <abbr title="Учетная запись, которая содержит все объекты данных хранилища Azure. Учетная запись хранения предоставляет пространство имен для данных вашего хранилища.">запись хранения Azure</abbr>и <abbr title="Облачный ресурс, в котором размещаются бессерверные функции Azure и который предоставляет базовое вычислительное окружение, в котором выполняются функции.">приложение-функцию,</abbr> выполнив следующие команды:

1. Войдите в Azure, если вы еще этого не сделали:

    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
    ```azurecli
    az login
    ```

    Чтобы войти в учетную запись Azure, выполните команду [az login](/cli/azure/reference-index#az_login).

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell) 
    ```azurepowershell
    Connect-AzAccount
    ```

    Чтобы войти в учетную запись Azure, выполните командлет [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount).

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

    Команда [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) создает группу ресурсов. Группу ресурсов и ресурсы целесообразно создавать в ближайшем к вам регионе. Для этого используйте командлет [Get-AzLocation](/powershell/module/az.resources/get-azlocation).

    ---

    Вы не можете разместить приложения Windows и Linux в одной группе ресурсов. Если у вас есть группа ресурсов `AzureFunctionsQuickstart-rg` с приложением-функцией Windows или веб-приложением, необходимо использовать другую группу ресурсов.

1. В группе ресурсов и регионе создайте учетную запись хранения Azure общего назначения:

    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

    ```azurecli
    az storage account create --name <STORAGE_NAME> --location westeurope --resource-group AzureFunctionsQuickstart-rg --sku Standard_LRS
    ```

    Создайте учетную запись хранения с помощью команды [az storage account create](/cli/azure/storage/account#az_storage_account_create). 

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

    ```azurepowershell
    New-AzStorageAccount -ResourceGroupName AzureFunctionsQuickstart-rg -Name <STORAGE_NAME> -SkuName Standard_LRS -Location westeurope
    ```

    Чтобы создать учетную запись хранения, выполните командлет [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount).

    ---

    Замените `<STORAGE_NAME>` именем, которое понравится вам, и <abbr title="Имя должно быть уникальным для всех учетных записей хранения, используемых всеми клиентами Azure глобально. Например, можно использовать сочетание собственного имени или названия компании с именем приложения и числовым идентификатором, например contosobizappstorage20.">будет уникальным для службы хранилища Azure</abbr>. Имена должны содержать от трех до 24 символов и только в нижнем регистре. `Standard_LRS` указывает учетную запись общего назначения, которая [поддерживается Функциями](storage-considerations.md#storage-account-requirements).
    
    В этом кратком руководстве с учетной записи хранения будет взиматься плата лишь в несколько центов США.

1. Создайте приложение-функцию в Azure:

    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
        
    ```azurecli
    az functionapp create --resource-group AzureFunctionsQuickstart-rg --consumption-plan-location westeurope --runtime python --runtime-version 3.8 --functions-version 3 --name <APP_NAME> --storage-account <STORAGE_NAME> --os-type linux
    ```
    
    Чтобы создать приложение-функцию в Azure, выполните команду [az functionapp create](/cli/azure/functionapp#az_functionapp_create). Если вы используете Python 3.7 или 3.6, измените `--runtime-version` на `3.7` или `3.6` соответственно.
    
    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)
    
    ```azurepowershell
    New-AzFunctionApp -Name <APP_NAME> -ResourceGroupName AzureFunctionsQuickstart-rg -StorageAccount <STORAGE_NAME> -FunctionsVersion 3 -RuntimeVersion 3.8 -Runtime python -Location 'West Europe'
    ```
    
    Чтобы создать приложение-функцию в Azure, выполните командлет [New-AzFunctionApp](/powershell/module/az.functions/new-azfunctionapp). Если вы используете Python 3.7 или 3.6, измените `-RuntimeVersion` на `3.7` или `3.6` соответственно.

    ---
    
    Замените имя `<STORAGE_NAME>` на имя учетной записи, которое вы использовали на предыдущем шаге.

    Замените имя `<APP_NAME>` на <abbr title="Уникальное имя, которое не должно встречаться ни у одного клиента Azure глобально. Например, можно использовать сочетание собственного имени или названия организации с именем приложения и числовым идентификатором, например contoso-bizapp-func-20.">глобально уникальное имя, которое вам подходит</abbr>. `<APP_NAME>` также является доменом DNS по умолчанию для приложения-функции. 
    
    <br/>
    <details>
    <summary><strong>Сколько стоят ресурсы, подготовленные в Azure?</strong></summary>

    Эта команда создает приложение-функцию, работающее в указанной языковой среде выполнения в рамках [плана использования Функций Azure](functions-scale.md#overview-of-plans), который не предусматривает плату за объем, используемый здесь. Эта команда также подготавливает связанный экземпляр Application Insights Azure в той же группе ресурсов. Его можно использовать для мониторинга приложения-функции и просмотра журналов. Дополнительные сведения см. в разделе [Мониторинг функций Azure](functions-monitoring.md). Этот экземпляр не создает затраты, пока вы не активируете его.
    </details>

<br/>

---

## <a name="6-deploy-the-function-project-to-azure"></a>6. Развертывание проекта функций в Azure

После того как вы успешно создадите приложение-функцию в Azure, вы сможете **развернуть локальный проект функций** с помощью команды [func azure functionapp publish](functions-run-local.md#project-file-deployment).  

В следующем примере замените `<APP_NAME>` на имя своего приложения.

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

Функция использует триггер HTTP, поэтому ее необходимо вызывать через HTTP-запрос к URL-адресу в браузере или с помощью такого средства, как <abbr title="Программа командной строки для создания HTTP-запросов к URL-адресу. См. https://curl.se/">curl</abbr>. 

# <a name="browser"></a>[Браузер](#tab/browser)

Скопируйте полный **URL-адрес вызова**, отображаемый в выходных данных команды `publish`, в адресную строку браузера, добавив параметр запроса **&name=Functions**. В браузере должны отображаться выходные данные, аналогичные данным при локальном запуске функции.

![Выходные данные функции, выполняемой в Azure в браузере](../../includes/media/functions-run-remote-azure-cli/function-test-cloud-browser.png)

# <a name="curl"></a>[curl](#tab/curl)

Запустите [`curl`](https://curl.haxx.se/), используя **URL-адрес вызова** и добавив параметр **&name=Functions**. Результатом выполнения этой команды должен быть текст Hello Functions.

![Выходные данные функции, выполненной в Azure в cURL](../../includes/media/functions-run-remote-azure-cli/function-test-cloud-curl.png)

---

### <a name="71-view-real-time-streaming-logs"></a>7.1 Просмотр журналов потоковой передачи в реальном времени

Выполните следующую команду, чтобы просмотреть [журналы потоковой передачи](functions-run-local.md#enable-streaming-logs) практически в реальном времени в Application Insights на портале Azure:

```console
func azure functionapp logstream <APP_NAME> --browser
```

Замените `<APP_NAME>` на имя приложения-функции.

В отдельном окне терминала или в браузере снова вызовите удаленную функцию. В окне терминала отображается подробный журнал выполнения функции в Azure. 

<br/>

---

## <a name="8-clean-up-resources"></a>8. Очистка ресурсов

Если вы планируете перейти к [следующему шагу](#next-steps) и добавить <abbr title="Средство для связывания функции с очередью хранилища, с помощью которого можно будет создавать сообщения в очереди. "> Выходная привязка очереди службы хранилища Azure</abbr>, сохраните все ресурсы в одном месте, поскольку на их основе будут создаваться другие элементы.

В противном случае используйте следующую команду, чтобы удалить группу ресурсов и все содержащиеся в ней ресурсы и избежать дополнительных расходов.

 # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az group delete --name AzureFunctionsQuickstart-rg
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell
Remove-AzResourceGroup -Name AzureFunctionsQuickstart-rg
```

<br/>

---

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Подключение Функций Azure к службе хранилища Azure с помощью средств командной строки](functions-add-output-binding-storage-queue-cli.md?pivots=programming-language-python)

[Возникли проблемы? Сообщите нам!](https://aka.ms/python-functions-qs-survey)
