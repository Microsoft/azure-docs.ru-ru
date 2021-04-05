---
title: Создание функции PowerShell с помощью Visual Studio Code (Функции Azure)
description: Сведения о том, как создать функцию PowerShell, а затем опубликовать локальный проект в бессерверном размещении в Функциях Azure с помощью расширения Функций Azure в Visual Studio Code.
ms.topic: quickstart
ms.date: 11/04/2020
ms.openlocfilehash: 9833b154cdba24f8760e7e8d9040bfc72bbcd271
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99493603"
---
# <a name="quickstart-create-a-powershell-function-in-azure-using-visual-studio-code"></a>Краткое руководство. Создание функции PowerShell в Azure с помощью Visual Studio Code

[!INCLUDE [functions-language-selector-quickstart-vs-code](../../includes/functions-language-selector-quickstart-vs-code.md)]

Из этой статьи вы узнаете, как создать функцию PowerShell, которая отвечает на HTTP-запросы, используя Visual Studio Code. После тестирования кода в локальной среде его необходимо развернуть в бессерверной среде Функций Azure.

Выполнение этого краткого руководства предполагает небольшую дополнительную плату в несколько центов США в учетной записи Azure.

Существует также версия этой статьи для [интерфейса командной строки](create-first-function-cli-powershell.md).

## <a name="configure-your-environment"></a>Настройка среды

Перед началом работы убедитесь, что выполнены следующие предварительные требования.

+ Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.

+ [Azure Functions Core Tools](functions-run-local.md#install-the-azure-functions-core-tools) версии 3.x.

+ [PowerShell 7](/powershell/scripting/install/installing-powershell-core-on-windows)

+ Среды выполнения [.NET Core 3.1](https://www.microsoft.com/net/download) и [.NET Core 2.1](https://dotnet.microsoft.com/download/dotnet-core/2.1).  

+ [Visual Studio Code](https://code.visualstudio.com/) на одной из [поддерживаемых платформ](https://code.visualstudio.com/docs/supporting/requirements#_platforms).    

+ [Расширение PowerShell для Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-vscode.PowerShell).  

+ [Расширение "Функции Azure"](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) для Visual Studio Code. 

## <a name="create-your-local-project"></a><a name="create-an-azure-functions-project"></a>Создание локального проекта

В этом разделе вы используете Visual Studio Code, чтобы создать локальный проект Функций Azure на PowerShell. Далее в этой статье вы опубликуете код функции в Azure.

1. Щелкните значок Azure на панели действий, а затем в области **Azure: Functions** (Azure: Функции) щелкните значок **Создать проект...** .

    ![Выбор варианта "Создать проект"](./media/functions-create-first-function-vs-code/create-new-project.png)

1. Выберите расположение для рабочей области проекта и нажмите кнопку **Выбрать**.

    > [!NOTE]
    > Рассматриваемые в этой статье шаги выполняются вне рабочей области. В этом случае не нужно указывать папку проекта, которая является частью рабочей области.

1. Введите следующие сведения по соответствующим запросам:

    + **Выберите язык для проекта приложения-функции**: Выберите `PowerShell`.

    + **Выберите шаблон для первой функции вашего проекта**. Выберите `HTTP trigger`.

    + **Укажите имя функции**. Введите `HttpExample`.

    + **Уровень авторизации**: выберите `Anonymous`, что позволит любому пользователю вызывать конечную точку функции. Дополнительные сведения об уровне авторизации см. в разделе [Authorization keys](functions-bindings-http-webhook-trigger.md#authorization-keys) (Ключи авторизации).

    + **Выберите, как вы хотели бы открыть свой проект**. Выберите `Add to workspace`.

1. Используя эти сведения, Visual Studio Code создает проект функций Azure с триггером HTTP. Файлы локального проекта можно просмотреть в Explorer. Дополнительные сведения см. в разделе [Generated project files](functions-develop-vs-code.md#generated-project-files) (Созданные файлы проекта). 

[!INCLUDE [functions-run-function-test-local-vs-code](../../includes/functions-run-function-test-local-vs-code.md)]

Убедившись, что функция выполняется правильно на локальном компьютере, опубликуйте проект в Azure с помощью Visual Studio Code. 

[!INCLUDE [functions-sign-in-vs-code](../../includes/functions-sign-in-vs-code.md)]

[!INCLUDE [functions-publish-project-vscode](../../includes/functions-publish-project-vscode.md)]

[!INCLUDE [functions-vs-code-run-remote](../../includes/functions-vs-code-run-remote.md)]

[!INCLUDE [functions-cleanup-resources-vs-code.md](../../includes/functions-cleanup-resources-vs-code.md)]

## <a name="next-steps"></a>Дальнейшие действия

С помощью [Visual Studio Code](functions-develop-vs-code.md?tabs=powershell) вы создали приложение-функцию с простой функцией, активируемой HTTP-запросом. В следующей статье показано, как расширить эту функцию путем подключения к службе хранилища Azure. Дополнительные сведения о подключении к другим службам Azure см. в статье [Подключение функций к службам Azure с помощью привязок](add-bindings-existing-function.md?tabs=powershell).  

> [!div class="nextstepaction"]
> [Подключение Функций Azure к службе хранилища Azure с помощью средств командной строки](functions-add-output-binding-storage-queue-vs-code.md?pivots=programming-language-powershell)

[Azure Functions Core Tools]: functions-run-local.md
[Azure Functions extension for Visual Studio Code]: https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions
