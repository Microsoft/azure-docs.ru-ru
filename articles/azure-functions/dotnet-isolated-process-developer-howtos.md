---
title: Разработка и публикация функций .NET 5 с помощью функций Azure
description: Узнайте, как создавать и отлаживать функции C# с помощью .NET 5,0, а затем развертывать локальный проект на бессерверном размещении в функциях Azure.
ms.date: 03/03/2021
ms.topic: how-to
zone_pivot_groups: development-environment-functions
ms.openlocfilehash: 70eacc5ec7f6adb65ba6e01c55acc6c6e3075ca9
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102584136"
---
# <a name="develop-and-publish-net-5-function-using-azure-functions"></a>Разработка и публикация функции .NET 5 с помощью функций Azure 

В этой статье показано, как работать с функциями C# с помощью .NET 5,0, которые выполняются вне процесса в среде выполнения функций Azure. Вы узнаете, как создавать, выполнять отладку локально и публиковать эти функции изолированного процесса .NET в Azure. В Azure эти функции выполняются в изолированном процессе, который поддерживает .NET 5,0. Дополнительные сведения см. в разделе " [инструкции по выполнению функций в .net 5,0" в Azure](dotnet-isolated-process-guide.md).

Если не требуется поддержка .NET 5,0 или запуск функций вне процесса, может потребоваться [создать функцию библиотеки классов C#](functions-create-your-first-function-visual-studio.md).

>[!NOTE]
>Разработка функций изолированного процесса .NET в портал Azure в настоящее время не поддерживается. Для создания приложения-функции в Azure, которое поддерживает запуск приложений .NET 5,0 вне процесса, необходимо использовать публикацию Azure CLI или Visual Studio Code.   

## <a name="prerequisites"></a>Предварительные требования

+ Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.

+ [Пакет SDK для .NET версии 5.0](https://www.microsoft.com/net/download)

+ [Azure functions Core Tools](functions-run-local.md#v2) версии 3.0.3381 или более поздней версии.

+ [Azure CLI](/cli/azure/install-azure-cli) версии 2,20 или более поздней версии.  
::: zone pivot="development-environment-vscode"
+ [Visual Studio Code](https://code.visualstudio.com/) на одной из [поддерживаемых платформ](https://code.visualstudio.com/docs/supporting/requirements#_platforms).  

+ [Расширение C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp) для Visual Studio Code.  

+ [Расширение функций Azure](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) для Visual Studio Code, Version 1.3.0 или более поздней версии.
::: zone-end
::: zone pivot="development-environment-vs"
+ [Visual Studio 2019](https://azure.microsoft.com/downloads/), включая рабочую нагрузку **разработки Azure** .  
Шаблоны проектов изолированных функций .NET и публикация в настоящее время недоступны в Visual Studio.
::: zone-end

## <a name="create-a-local-function-project"></a>Создание локального проекта службы "Функции"

В Функциях Azure проект функций представляет собой контейнер для одной или нескольких отдельных функций, каждая из которых реагирует на конкретный триггер. Все функции в проекте совместно используют те же локальные конфигурации и конфигурации размещения. В этом разделе вы создадите проект функций, содержащий одну функцию.

::: zone pivot="development-environment-vs"

>[!NOTE]  
> В настоящее время отсутствуют шаблоны проектов Visual Studio, которые поддерживают создание проектов изолированных функций .NET. В этой статье показано, как использовать основные средства для создания проекта C#, который затем можно запустить локально и выполнить отладку в Visual Studio.  

::: zone-end

::: zone pivot="development-environment-vscode"  
1. Щелкните значок Azure на панели действий, а затем в области **Azure: Functions** (Azure: Функции) щелкните значок **Создать проект...** .

    ![Выбор варианта "Создать проект"](./media/functions-create-first-function-vs-code/create-new-project.png)

1. Выберите расположение для рабочей области проекта и нажмите кнопку **Выбрать**.

    > [!NOTE]
    > Рассматриваемые в этой статье шаги выполняются вне рабочей области. В этом случае не нужно указывать папку проекта, которая является частью рабочей области.

1. Введите следующие сведения по соответствующим запросам:

    + **Выберите язык для проекта приложения-функции**: Выберите `C#`.

    + **Выберите среду выполнения .NET**: выберите `.NET 5 isolated` .

    + **Выберите шаблон для первой функции вашего проекта**. Выберите `HTTP trigger`.

    + **Укажите имя функции**. Введите `HttpExample`.

    + **Укажите пространство имен**. Введите `My.Functions`.

    + **Уровень авторизации**: выберите `Anonymous`, что позволит любому пользователю вызывать конечную точку функции. Дополнительные сведения об уровне авторизации см. в разделе [Authorization keys](functions-bindings-http-webhook-trigger.md#authorization-keys) (Ключи авторизации).

    + **Выберите, как вы хотели бы открыть свой проект**. Выберите `Add to workspace`.

1. Используя эти сведения, Visual Studio Code создает проект функций Azure с триггером HTTP. Файлы локального проекта можно просмотреть в Explorer. Дополнительные сведения см. в разделе [Generated project files](functions-develop-vs-code.md#generated-project-files) (Созданные файлы проекта).
::: zone-end  
::: zone pivot="development-environment-cli,development-environment-vs"  

1. Выполните `func init` следующую команду, чтобы создать проект функций в папке с именем *локалфунктионпрож*:  

    ```console
    func init LocalFunctionProj --worker-runtime dotnetisolated
    ```
    
    Указание `dotnetisolated` создает проект, работающий на платформе .net 5,0.


1. Перейдите в папку проекта:

    ```console
    cd LocalFunctionProj
    ```

    Эта папка содержит различные файлы проекта, включая [local.settings.jsдля](functions-run-local.md#local-settings-file) и [host.jsв](functions-host-json.md) файлах конфигураций. Файл *local.settings.json* может содержать секреты, скачанные из Azure, поэтому файл по умолчанию исключен из системы управления версиями в *GITIGNORE*-файле.

1. Добавьте функцию в проект с помощью приведенной ниже команды, где аргумент `--name` — уникальное имя функции (HttpExample), а аргумент `--template` позволяет указать триггер функции (HTTP).

    ```console
    func new --name HttpExample --template "HTTP trigger" --authlevel "anonymous"
    ``` 

    `func new` создает файл кода HttpExample.cs.
::: zone-end  

::: zone pivot="development-environment-vscode"  

[!INCLUDE [functions-run-function-test-local-vs-code](../../includes/functions-run-function-test-local-vs-code.md)]

::: zone-end  
::: zone pivot="development-environment-cli" 

[!INCLUDE [functions-run-function-test-local-cli](../../includes/functions-run-function-test-local-cli.md)]

::: zone-end

::: zone pivot="development-environment-vs"

## <a name="run-the-function-locally"></a>Локальное выполнение функции

На этом этапе можно выполнить `func start` команду из корня папки проекта, чтобы скомпилировать и запустить проект изолированных функций C#. В настоящее время, если вы хотите отладить код необработанной функции в Visual Studio, необходимо вручную подключить отладчик к процессу среды выполнения выполняемых функций, выполнив следующие действия.  

1. Откройте файл проекта (. csproj) в Visual Studio. Вы можете просматривать и изменять код проекта, а также задавать любые нужные точки останова в коде. 

1. В корневой папке проекта используйте следующую команду из терминала или командной строки, чтобы запустить узел среды выполнения:

    ```console
    func start --dotnet-isolated-debug
    ```

    `--dotnet-isolated-debug`Параметр указывает процессу ожидать присоединения отладчика перед продолжением. В конце выходных данных вы увидите нечто вроде следующих: 
    
    <pre>
    ...
    
    Functions:

        HttpExample: [GET,POST] http://localhost:7071/api/HttpExample

    For detailed output, run func with --verbose flag.
    [2021-03-09T08:41:41.904Z] Azure Functions .NET Worker (PID: 81720) initialized in debug mode. Waiting for debugger to attach...
    ...
    
    </pre> 

    `PID: XXXXXX`Указывает идентификатор процесса (PID) dotnet.exe процесса, который является узлом выполняемых функций.
 
1. В выходных данных среды выполнения функций Azure запишите идентификатор процесса хост-процесса, к которому будет присоединен отладчик. Также обратите внимание на URL-адрес локальной функции.

1. В меню **Отладка** в Visual Studio выберите **присоединить к процессу...**, укажите dotnet.exe процесс, соответствующий идентификатору процесса, и выберите **присоединить**. 
    
    :::image type="content" source="media/dotnet-isolated-process-developer-howtos/attach-to-process.png" alt-text="Подключение отладчика к ведущему процессу функций":::    

    С помощью присоединенного отладчика можно отлаживать код функции как обычная.

1. В адресной строке браузера введите URL-адрес локальной функции, который выглядит следующим образом, и выполните запрос. 

    <http://localhost:7071/api/HttpExample>

    Вы должны увидеть выходные данные трассировки из запроса, записанного в выполняющийся терминал. Выполнение кода останавливается в любой точке останова, заданной в коде функции.

1. Когда все будет готово, перейдите в окно терминала и нажмите клавиши CTRL + C, чтобы прерывать ведущий процесс.
 
Убедившись, что функция выполняется правильно на локальном компьютере, опубликуйте проект в Azure.

> [!NOTE]  
> Публикация Visual Studio сейчас недоступна для приложений с изолированным процессом .NET. После завершения разработки проекта в Visual Studio необходимо использовать Azure CLI для создания удаленных ресурсов Azure. Затем можно снова использовать Azure Functions Core Tools из командной строки для публикации проекта в Azure.
::: zone-end

::: zone pivot="development-environment-cli,development-environment-vs" 
## <a name="create-supporting-azure-resources-for-your-function"></a>Создание вспомогательных ресурсов Azure для функции

Прежде чем развернуть код функции в Azure, необходимо создать три ресурса:

- [группу ресурсов](../azure-resource-manager/management/overview.md) — логический контейнер связанных ресурсов;
- [учетную запись хранения](../storage/common/storage-account-create.md), которая используется для сохранения состояния и других сведений о функциях;
- приложение-функцию — среду для выполнения кода функции. Оно сопоставляется с локальным проектом функций и позволяет группировать функции в логические единицы, чтобы упростить развертывание, масштабирование и совместное использование ресурсов, а также управление ими.

Чтобы создать эти элементы, выполните следующие команды:

1. Войдите в Azure, если вы еще этого не сделали:

    ```azurecli
    az login
    ```

    Чтобы войти в учетную запись Azure, выполните команду [az login](/cli/azure/reference-index#az_login).

1. Создайте группу ресурсов с именем `AzureFunctionsQuickstart-rg` в регионе `westeurope`:

    ```azurecli
    az group create --name AzureFunctionsQuickstart-rg --location westeurope
    ```
 
    Чтобы создать группу ресурсов, выполните команду [az group create](/cli/azure/group#az_group_create). Группу ресурсов и ресурсы целесообразно создавать в ближайшем к вам регионе. Для этого используйте команду `az account list-locations`.

1. В группе ресурсов и регионе создайте учетную запись хранения общего назначения:

    ```azurecli
    az storage account create --name <STORAGE_NAME> --location westeurope --resource-group AzureFunctionsQuickstart-rg --sku Standard_LRS
    ```

    Создайте учетную запись хранения с помощью команды [az storage account create](/cli/azure/storage/account#az_storage_account_create). 

    В предыдущем примере замените `<STORAGE_NAME>` соответствующим именем, которое является уникальным в службе хранилища Azure. Имена должны содержать от трех до 24 символов и только в нижнем регистре. `Standard_LRS` указывает учетную запись общего назначения, которая [поддерживается Функциями](storage-considerations.md#storage-account-requirements).

1. Создайте приложение-функцию в Azure:
   
    ```azurecli
    az functionapp create --resource-group AzureFunctionsQuickstart-rg --consumption-plan-location westeurope --runtime dotnet-isolated --runtime-version 5.0 --functions-version 3 --name <APP_NAME> --storage-account <STORAGE_NAME>
    ```

    Чтобы создать приложение-функцию в Azure, выполните команду [az functionapp create](/cli/azure/functionapp#az_functionapp_create). 
    
    В предыдущем примере замените `<STORAGE_NAME>` именем учетной записи, использованной на предыдущем шаге, и измените `<APP_NAME>` на глобально уникальное имя, подходящее вам. `<APP_NAME>` также является доменом DNS по умолчанию для приложения-функции. 
    
    Эта команда создает приложение-функцию, работающее под управлением .NET 5,0, в [плане потребления функций Azure](consumption-plan.md). Этот план должен быть бесплатным для использования в этой статье. Команда также подготавливает связанный экземпляр Application Insights Azure в той же группе ресурсов. Используйте этот экземпляр для мониторинга приложения функции и просмотра журналов. Дополнительные сведения см. в разделе [Мониторинг функций Azure](functions-monitoring.md). Этот экземпляр не создает затраты, пока вы не активируете его.

[!INCLUDE [functions-publish-project-cli](../../includes/functions-publish-project-cli.md)]

::: zone-end

::: zone pivot="development-environment-vscode"

[!INCLUDE [functions-sign-in-vs-code](../../includes/functions-sign-in-vs-code.md)]

## <a name="publish-the-project-to-azure"></a>Публикация проекта в Azure

В этом разделе показано создание приложения-функции и сопутствующих ресурсов в подписке Azure и последующее развертывание кода.

> [!IMPORTANT]
> Публикация в существующее приложение-функцию перезаписывает содержимое этого приложения в Azure.


1. Щелкните значок Azure на панели действий, а затем в области **Azure: Функции** выберите кнопку **Deploy to function app...** (Развертывание в приложение-функцию).

    ![Публикация проекта в Azure](../../includes/media/functions-publish-project-vscode/function-app-publish-project.png)

1. Введите следующие сведения по соответствующим запросам:

    - **Выбор папки**. Выберите папку из рабочей области или перейдите к папке, содержащей приложение-функцию. Вы не увидите этот запрос, когда уже открыто допустимое приложение-функция.

    - **Выбрать подписку**. Выберите подписку, которую нужно использовать. Вы не увидите этот запрос, если у вас есть только одна подписка.

    - **Select Function App in Azure** (Выбор приложения-функции в Azure). Выберите `- Create new Function App`. (Не выбирайте параметр `Advanced`, так как он не рассматривается в этой статье.)
      
    - **Enter a globally unique name for the function app** (Ввод глобально уникального имени для приложения-функции). Введите имя, допустимое в пути URL-адреса. Имя, которое вы вводите, проверяется, чтобы убедиться, что оно уникально в функциях Azure.
    
    - **Select a runtime stack** (Выберите стек сред выполнения). Выберите `.NET 5 (non-LTS)`. 
    
    - **Select a location for new resources** (Выбор расположения для новых ресурсов).  Для повышения производительности выберите [регион](https://azure.microsoft.com/regions/) рядом с вами. 
    
    В области уведомлений отображаются сведения о состоянии отдельных ресурсов по мере их создания в Azure.

    :::image type="content" source="../../includes/media/functions-publish-project-vscode/resource-notification.png" alt-text="Уведомление о создании ресурса Azure":::
    
1.  После завершения в вашей подписке создаются следующие ресурсы Azure с именами, производными от имени приложения-функции:
    
    [!INCLUDE [functions-vs-code-created-resources](../../includes/functions-vs-code-created-resources.md)]

    После создания приложения-функции и применения пакета развертывания отобразится уведомление. 

    [!INCLUDE [functions-vs-code-create-tip](../../includes/functions-vs-code-create-tip.md)]

4. Выберите **View Output** (Просмотреть выходные данные) в уведомлении, чтобы просмотреть результаты создания и развертывания ресурсов Azure. Если вы пропустили уведомление, щелкните значок колокольчика в правом нижнем углу, чтобы снова просмотреть его.

    ![Создание уведомления о завершении](../../includes/media/functions-publish-project-vscode/function-create-notifications.png)

::: zone-end

::: zone pivot="development-environment-cli"  
[!INCLUDE [functions-run-remote-azure-cli](../../includes/functions-run-remote-azure-cli.md)]  
::: zone-end  

::: zone pivot="development-environment-vscode"  
[!INCLUDE [functions-vs-code-run-remote](../../includes/functions-vs-code-run-remote.md)]  
::: zone-end  

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы создали ресурсы для выполнения этой статьи. Вам могут быть выставлены счета за эти ресурсы в зависимости от [состояния учетной записи](https://azure.microsoft.com/account/) и [цен на службы](https://azure.microsoft.com/pricing/). 

::: zone pivot="development-environment-cli"  
Используйте следующую команду, чтобы удалить группу ресурсов и все содержащиеся в ней ресурсы, чтобы избежать дополнительных затрат.

```azurecli
az group delete --name AzureFunctionsQuickstart-rg
```
::: zone-end  

::: zone pivot="development-environment-vscode"  
Выполните следующие действия, чтобы удалить приложение функции и связанные с ним ресурсы, чтобы избежать дополнительных расходов.

[!INCLUDE [functions-cleanup-resources-vs-code-inner.md](../../includes/functions-cleanup-resources-vs-code-inner.md)]  
::: zone-end  
::: zone pivot="development-environment-vs"   
Выполните следующие действия, чтобы удалить приложение функции и связанные с ним ресурсы, чтобы избежать дополнительных расходов.

1. В Cloud Explorer разверните свою подписку и выберите пункт **Службы приложений**, щелкните правой кнопкой мыши приложение-функцию и выберите пункт **Открыть на портале**. 

1. На странице приложения-функции выберите вкладку **Обзор**, а затем щелкните ссылку в разделе **Группа ресурсов**.

   :::image type="content" source="media/functions-create-your-first-function-visual-studio/functions-app-delete-resource-group.png" alt-text="Выбор группы ресурсов, которую требуется удалить со страницы приложения-функции":::

2. На странице **Группа ресурсов** просмотрите список включенных ресурсов и убедитесь, что именно их нужно удалить.
 
3. Выберите **Удалить группу ресурсов** и следуйте инструкциям.

   Удаление может занять несколько минут. После этого на несколько секунд появится уведомление. Кроме того, можно выбрать значок колокольчика в верхней части страницы, чтобы просмотреть уведомление.
::: zone-end

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Дополнительные сведения о изолированных функциях .NET](dotnet-isolated-process-guide.md)

