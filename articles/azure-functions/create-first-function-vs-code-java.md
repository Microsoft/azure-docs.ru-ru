---
title: Создание функции Java с помощью Visual Studio Code — Функции Azure
description: Узнайте, как создать функцию Java, а затем опубликовать локальный проект в бессерверном размещении в Функциях Azure с помощью расширения Функций Azure в Visual Studio Code.
ms.topic: quickstart
ms.date: 11/03/2020
adobe-target: true
adobe-target-activity: DocsExp–386541–A/B–Enhanced-Readability-Quickstarts–2.19.2021
adobe-target-experience: Experience B
adobe-target-content: ./create-first-function-vs-code-java-uiex
ms.openlocfilehash: 10ac677fb65428e8b8855c43b319bbebe7bbbb71
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101701438"
---
# <a name="quickstart-create-a-java-function-in-azure-using-visual-studio-code"></a>Краткое руководство. Создание функции Java в Azure с помощью Visual Studio Code

> [!div class="op_single_selector" title1="Выберите язык функции: "]
> - [Java](create-first-function-vs-code-java.md)
> - [Python](create-first-function-vs-code-python.md)
> - [C#](create-first-function-vs-code-csharp.md)
> - [JavaScript](create-first-function-vs-code-node.md)
> - [PowerShell](create-first-function-vs-code-powershell.md)
> - [TypeScript](create-first-function-vs-code-typescript.md)
> - [Другое (Go/Rust)](create-first-function-vs-code-other.md)

Из этой статьи вы узнаете, как создать функцию Java, которая отвечает на HTTP-запросы, используя Visual Studio Code. После тестирования кода в локальной среде его необходимо развернуть в бессерверной среде Функций Azure.

Выполнение этого краткого руководства предполагает небольшую дополнительную плату в несколько центов США в учетной записи Azure.

> [!NOTE]
> Если вы не хотите использовать Visual Studio Code в качестве средства разработки, ознакомьтесь с аналогичными руководствами для разработчиков Java по использованию [Maven](create-first-function-cli-java.md), [Gradle](./functions-create-first-java-gradle.md) и [IntelliJ IDEA](/azure/developer/java/toolkit-for-intellij/quickstart-functions).

## <a name="configure-your-environment"></a>Настройка среды

Перед началом работы убедитесь, что выполнены следующие предварительные требования.

+ Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.

+ [Java Developer Kit](/azure/developer/java/fundamentals/java-jdk-long-term-support) версии 8 или 11.

+ [Apache Maven](https://maven.apache.org) 3.0 или более поздней версии.

+ [Visual Studio Code](https://code.visualstudio.com/) на одной из [поддерживаемых платформ](https://code.visualstudio.com/docs/supporting/requirements#_platforms).

+ [Пакет расширения Java](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack)  

+ [Расширение "Функции Azure"](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) для Visual Studio Code. 

## <a name="create-your-local-project"></a><a name="create-an-azure-functions-project"></a>Создание локального проекта

В этом разделе показано, как использовать Visual Studio Code, чтобы создать локальный проект Функций Azure на Java. Далее в этой статье вы опубликуете код функции в Azure. 

1. Щелкните значок Azure на панели действий, а затем в области **Azure: Functions** (Azure: Функции) щелкните значок **Создать проект...** .

    ![Выбор варианта "Создать проект"](./media/functions-create-first-function-vs-code/create-new-project.png)

1. Выберите расположение для рабочей области проекта и нажмите кнопку **Выбрать**.

    > [!NOTE]
    > Рассматриваемые в этой статье шаги выполняются вне рабочей области. В этом случае не нужно указывать папку проекта, которая является частью рабочей области.

1. Введите следующие сведения по соответствующим запросам:

    + **Выберите язык для проекта приложения-функции**: Выберите `Java`.

    + **Выберите версию Java**: Выберите `Java 8` или `Java 11` в зависимости от того, какая из версий Java нужна для запуска функций в Azure. Выберите версию Java, которую вы проверили локально.

    + **Укажите идентификатор группы**: Выберите `com.function`.

    + **Укажите идентификатор артефакта**: Выберите `myFunction`.

    + **Укажите версию**: Выберите `1.0-SNAPSHOT`.

    + **Укажите имя пакета**: Выберите `com.function`.

    + **Укажите имя приложения**: Выберите `myFunction-12345`.

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

С помощью [Visual Studio Code](functions-develop-vs-code.md?tabs=java) вы создали приложение-функцию с простой функцией, активируемой HTTP-запросом. В следующей статье показано, как расширить эту функцию путем подключения к службе хранилища Azure. Дополнительные сведения о подключении к другим службам Azure см. в статье [Подключение функций к службам Azure с помощью привязок](add-bindings-existing-function.md?tabs=java). 

> [!div class="nextstepaction"]
> [Подключение Функций Azure к службе хранилища Azure с помощью средств командной строки](functions-add-output-binding-storage-queue-vs-code.md?pivots=programming-language-java)

[Azure Functions Core Tools]: functions-run-local.md
[Azure Functions extension for Visual Studio Code]: https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions
