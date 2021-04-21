---
title: Создание функции Java из командной строки (Функции Azure)
description: Узнайте, как создать функцию Java в командной строке, а затем опубликовать локальный проект в бессерверном размещении в Функциях Azure.
ms.date: 11/03/2020
ms.topic: quickstart
ms.custom:
- devx-track-java
- devx-track-azurecli
adobe-target: true
adobe-target-activity: DocsExp–386541–A/B–Enhanced-Readability-Quickstarts–2.19.2021
adobe-target-experience: Experience B
adobe-target-content: ./create-first-function-cli-java-uiex
ms.openlocfilehash: 1a1571e83e33ee0aec40df4cf68b3d01b6ef5f04
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107787509"
---
# <a name="quickstart-create-a-java-function-in-azure-from-the-command-line"></a>Краткое руководство. Создание функции Java в Azure из командной строки

[!INCLUDE [functions-language-selector-quickstart-cli](../../includes/functions-language-selector-quickstart-cli.md)]

В этой статье используются средства командной строки для создания функции Java, которая отвечает на HTTP-запросы. После тестирования кода в локальной среде его необходимо развернуть в бессерверной среде Функций Azure.

Выполнение этого краткого руководства предполагает небольшую дополнительную плату в несколько центов США в учетной записи Azure.

> [!NOTE]
> Если вы не хотите использовать Maven в качестве средства разработки, ознакомьтесь с аналогичными руководствами для разработчиков Java по использованию [Gradle](./functions-create-first-java-gradle.md), [IntelliJ IDEA](/azure/developer/java/toolkit-for-intellij/quickstart-functions) и [Visual Studio Code](create-first-function-vs-code-java.md).

## <a name="configure-your-local-environment"></a>Настройка локальной среды

Перед началом работы убедитесь, что у вас есть такие компоненты.

+ Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.

+ [Azure Functions Core Tools](functions-run-local.md#v2) версии 3.x.

+ [Azure CLI](/cli/azure/install-azure-cli) 2.4 или более поздней версии.

+ [Java Developer Kit](/azure/developer/java/fundamentals/java-jdk-long-term-support) версии 8 или 11. Переменной среды `JAVA_HOME` необходимо присвоить расположение установки правильной версии JDK.     

+ [Apache Maven](https://maven.apache.org) 3.0 или более поздней версии.

### <a name="prerequisite-check"></a>Проверка предварительных условий

+ В окне терминала или командном окне запустите `func --version`, чтобы убедиться, что используется версия Azure Functions Core Tools 3.x.

+ Выполните команду `az --version`, чтобы убедиться, что используется версия Azure CLI 2.4 или более поздняя.

+ Выполните команду `az login`, чтобы войти в Azure и проверить активную подписку.

## <a name="create-a-local-function-project"></a>Создание локального проекта службы "Функции"

В Функциях Azure проект функций представляет собой контейнер для одной или нескольких отдельных функций, каждая из которых реагирует на конкретный триггер. Все функции в проекте совместно используют те же локальные конфигурации и конфигурации размещения. В этом разделе вы создадите проект функций, содержащий одну функцию.

1. В пустой папке выполните следующую команду, чтобы создать проект функций из [архетипа Maven](https://maven.apache.org/guides/introduction/introduction-to-archetypes.html). 

    # <a name="bash"></a>[Bash](#tab/bash)
    
    ```bash
    mvn archetype:generate -DarchetypeGroupId=com.microsoft.azure -DarchetypeArtifactId=azure-functions-archetype -DjavaVersion=8
    ```
    
    # <a name="powershell"></a>[PowerShell](#tab/powershell)
    
    ```powershell
    mvn archetype:generate "-DarchetypeGroupId=com.microsoft.azure" "-DarchetypeArtifactId=azure-functions-archetype" "-DjavaVersion=8" 
    ```
    
    # <a name="cmd"></a>[Cmd](#tab/cmd)
    
    ```cmd
    mvn archetype:generate "-DarchetypeGroupId=com.microsoft.azure" "-DarchetypeArtifactId=azure-functions-archetype" "-DjavaVersion=8"
    ```
    
    ---

    > [!IMPORTANT]
    > + Используйте `-DjavaVersion=11`, чтобы функции выполнялись на Java 11. Дополнительные сведения см. в разделе [Версии Java](functions-reference-java.md#java-versions). 
    > + Чтобы выполнить задания из этой статьи, переменной среды `JAVA_HOME` необходимо присвоить расположение установки правильной версии JDK.

1. Maven запрашивает значения, которые позволят завершить создание проекта развертывания.   
    Предоставьте следующие значения в ответ на соответствующие запросы:

    | prompt | Значение | Описание |
    | ------ | ----- | ----------- |
    | **groupId** | `com.fabrikam` | Это значение уникально идентифицирует проект среди всех остальных. Оно должно соответствовать [правилам именования пакетов](https://docs.oracle.com/javase/specs/jls/se6/html/packages.html#7.7) для Java. |
    | **artifactId** | `fabrikam-functions` | Это значение содержит имя JAR-файла, без номера версии. |
    | **version** | `1.0-SNAPSHOT` | Выберите значение по умолчанию. |
    | **package** | `com.fabrikam` | Это значение определяет пакет Java для создаваемого кода функции. Используйте значение по умолчанию. |

1. Введите `Y` или нажмите клавишу ВВОД для подтверждения.

    Maven создаст файлы проекта в новой папке с именем _artifactId_, то есть `fabrikam-functions` в нашем примере. 

1. Перейдите в папку проекта:

    ```console
    cd fabrikam-functions
    ```

    Эта папка содержит различные файлы проекта, в том числе файлы конфигурации [local.settings.json](functions-run-local.md#local-settings-file) и [host.json](functions-host-json.md). Файл *local.settings.json* может содержать секреты, скачанные из Azure, поэтому файл по умолчанию исключен из системы управления версиями в *GITIGNORE*-файле.

### <a name="optional-examine-the-file-contents"></a>Проверка содержимого файла (дополнительно)

При необходимости можно сразу перейти к [локальному запуску функции](#run-the-function-locally) и просмотреть содержимое файла позже.

#### <a name="functionjava"></a>Function.java
Файл *Function.java* содержит метод `run`, получающий данные запроса в переменной `request`. Это запрос [HttpRequestMessage](/java/api/com.microsoft.azure.functions.httprequestmessage), дополненный заметкой [HttpTrigger](/java/api/com.microsoft.azure.functions.annotation.httptrigger), которая определяет поведение триггера. 

:::code language="java" source="~/azure-functions-samples-java/src/main/java/com/functions/Function.java":::

Сообщение-ответ создается API [HttpResponseMessage.Builder](/java/api/com.microsoft.azure.functions.httpresponsemessage.builder).

#### <a name="pomxml"></a>pom.xml

Параметры ресурсов Azure, созданных для размещения приложения, определяются в элементе **конфигурации** подключаемого модуля с **groupId** `com.microsoft.azure` в созданном файле pom.xml. Например, элемент конфигурации ниже указывает развертыванию на основе Maven, что приложение-функцию необходимо создать в группе ресурсов `java-functions-group` в области `westus`. Само приложение-функция работает в Windows, размещенном в плане `java-functions-app-service-plan`, который по умолчанию является бессерверным планом потребления.

:::code language="java" source="~/azure-functions-samples-java/pom.xml" range="62-107":::

Вы можете изменить эти параметры, чтобы управлять созданием ресурсов в Azure, например, изменив `runtime.os` с `windows` на `linux` перед первоначальным развертыванием. Полный список параметров, поддерживаемых подключаемым модулем Maven, см. в [сведениях о конфигурации](https://github.com/microsoft/azure-maven-plugins/wiki/Azure-Functions:-Configuration-Details).

#### <a name="functiontestjava"></a>FunctionTest.java

Архетип также создает модульный тест для функции. При изменении функции для добавления привязок или добавления новых функций в проект необходимо также изменить тесты в файле *FunctionTest.java*.

## <a name="run-the-function-locally"></a>Локальное выполнение функции

1. Выполните функцию, запустив локальное хост-приложение среды выполнения Функций Azure из папки *LocalFunctionProj*:

    ```console
    mvn clean package
    mvn azure-functions:run
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
    
    > [!NOTE]  
    > Если результат HttpExample не похож на пример выше, скорее всего, вы запустили основное приложение из папки, отличной от корневой папки проекта. В этом случае остановите хост-приложение клавишами **CTRL**+**C**, перейдите в корневую папку проекта и снова выполните указанную выше команду.

1. Скопируйте URL-адрес функции `HttpExample` из этих выходных данных в браузер и добавьте строку запроса `?name=<YOUR_NAME>`, сформировав полный URL-адрес, например `http://localhost:7071/api/HttpExample?name=Functions`. Браузер отобразит сообщение, содержащее значение строки запроса. Терминал, в котором вы запустили проект, также выводит данные журнала при выполнении запросов.

1. Когда все будет готово, нажмите клавиши **Ctrl**+**C** и выберите `y`, чтобы отключить хост-приложение функции.

## <a name="deploy-the-function-project-to-azure"></a>Развертывание проекта функций в Azure

Приложение-функция и связанные ресурсы создаются в Azure при первом развертывании проекта функции. Параметры ресурсов Azure, созданных для размещения приложения, определяются в файле [pom.xml](#pomxml). В этой статье вы примете значения по умолчанию.

> [!TIP]
> Чтобы создать приложение-функцию, работающее в Linux вместо Windows, измените элемент `runtime.os` в файле pom.xml с `windows` на `linux`. Работа Linux в плане потребления поддерживается в [этих регионах](https://github.com/Azure/azure-functions-host/wiki/Linux-Consumption-Regions). Приложения, работающие в Linux, и приложения, работающие под управлением ОС Windows, не могут находиться в одной группе ресурсов.

1. Прежде чем выполнять развертывание, войдите в подписку Azure с помощью Azure CLI или Azure PowerShell. 

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

1. Используйте следующую команду, чтобы развернуть проект в виде нового приложения-функции.

    ```console
    mvn azure-functions:deploy
    ```
    
    Это создает в Azure такие ресурсы:
    
    + группа ресурсов; С именем _java-functions-group_.
    + учетная запись хранения; Требуется для Функций Azure. Это имя создается случайным образом на основе требований к именованию учетных записей хранения.
    + План размещения. Бессерверное размещение для приложения-функции в регионе, который указан в параметре _westus_. Имя _java-functions-app-service-plan_.
    + Приложение-функция. Приложение-функция представляет собой минимальную единицу развертывания и выполнения для ваших функций. Имя создается случайным образом на основе _artifactId_, к которому добавляется случайное число.
    
    Развертывание упаковывает файлы проекта и развертывает их в новом приложении-функции [из ZIP-файла](functions-deployment-technologies.md#zip-deploy) Код выполняется из пакета развертывания в Azure.

[!INCLUDE [functions-run-remote-azure-cli](../../includes/functions-run-remote-azure-cli.md)]

[!INCLUDE [functions-streaming-logs-cli-qs](../../includes/functions-streaming-logs-cli-qs.md)]

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы намерены перейти к [следующему шагу](#next-steps) и добавить выходную привязку очереди службы хранилища Azure, можете сохранить все ресурсы, которые пригодятся на этом этапе.

В противном случае используйте следующую команду, чтобы удалить группу ресурсов и все содержащиеся в ней ресурсы и избежать дополнительных расходов.

 # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az group delete --name java-functions-group
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell
Remove-AzResourceGroup -Name java-functions-group
```

---

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Подключение Функций Azure к службе хранилища Azure с помощью средств командной строки](functions-add-output-binding-storage-queue-cli.md?pivots=programming-language-java)
