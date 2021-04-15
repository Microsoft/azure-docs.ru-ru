---
title: Создание функции Java из командной строки (Функции Azure)
description: Узнайте, как создать функцию Java в командной строке, а затем опубликовать локальный проект в бессерверном размещении в Функциях Azure.
ms.date: 11/03/2020
ms.topic: quickstart
ms.custom:
- devx-track-java
ROBOTS: NOINDEX,NOFOLLOW
ms.openlocfilehash: b5bc453e2e0371ee0412824f01d99863b12d91e2
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/13/2021
ms.locfileid: "107375378"
---
# <a name="quickstart-create-a-java-function-in-azure-from-the-command-line"></a>Краткое руководство. Создание функции Java в Azure из командной строки

> [!div class="op_single_selector" title1="Выберите язык функции: "]
> - [Java](create-first-function-cli-java.md)
> - [Python](create-first-function-cli-python.md)
> - [C#](create-first-function-cli-csharp.md)
> - [JavaScript](create-first-function-cli-node.md)
> - [PowerShell](create-first-function-cli-powershell.md)
> - [TypeScript](create-first-function-cli-typescript.md)

С помощью средств командной строки создайте функцию Java, которая отвечает на HTTP-запросы. Протестируйте код в локальной среде, а затем разверните его в бессерверном окружении Функций Azure.

Выполнение этого краткого руководства предполагает небольшую дополнительную плату в несколько центов США, которая будет взиматься с вашей <abbr title="Профиль, который поддерживает данные для выставления счетов за использование Azure.">Учетная запись Azure</abbr>.

Если вы не хотите использовать Maven в качестве средства разработки, ознакомьтесь с аналогичными руководствами для разработчиков Java по использованию [Gradle](./functions-create-first-java-gradle.md), [IntelliJ IDEA](/azure/developer/java/toolkit-for-intellij/quickstart-functions) и [Visual Studio Code](create-first-function-vs-code-java.md).

## <a name="1-prepare-your-environment"></a>1. Подготовка среды

Перед началом работы убедитесь, что у вас есть такие компоненты.

+ Учетная запись Azure с активной <abbr title="Базовая организационная структура, в которой можно управлять ресурсами в Azure, обычно связанными с частным лицом или отделом в пределах организации.">Подписка</abbr>. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.

+ [Azure Functions Core Tools](functions-run-local.md#v2) версии 3.x.

+ [Azure CLI](/cli/azure/install-azure-cli) 2.4 или более поздней версии.

+ [Java Developer Kit](/azure/developer/java/fundamentals/java-jdk-long-term-support) версии 8 или 11. Переменной среды `JAVA_HOME` необходимо присвоить расположение установки правильной версии JDK.

+ [Apache Maven](https://maven.apache.org) 3.0 или более поздней версии.

### <a name="prerequisite-check"></a>Проверка предварительных условий

+ В терминале или командном окне выполните `func --version`, чтобы убедиться, что версией <abbr title="Набор программ командной строки для работы с Функциями Azure на локальном компьютере.">Azure Functions Core Tools</abbr> является версия 3.x.

+ Выполните команду `az --version`, чтобы убедиться, что используется версия Azure CLI 2.4 или более поздняя.

+ Выполните команду `az login`, чтобы войти в Azure и проверить активную подписку.

<br>
<hr/>

## <a name="2-create-a-local-function-project"></a>2. Создание локального проекта службы "Функции"

В Функциях Azure проект функций представляет собой контейнер для одной или нескольких отдельных функций, каждая из которых реагирует на конкретный <abbr title="Тип события, вызывающего код функции, например HTTP-запрос, сообщение в очереди или определенное время.">триггер</abbr>. Все функции в проекте совместно используют те же локальные конфигурации и конфигурации размещения. В этом разделе вы создадите проект функций, содержащий одну функцию.

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

    <br/>
    <details>
    <summary><strong>Чтобы запустить функции на Java 11</strong></summary>

    Используйте `-DjavaVersion=11`, чтобы функции выполнялись на Java 11. Дополнительные сведения см. в разделе [Версии Java](functions-reference-java.md#java-versions).
    </details>

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

<br/>
<details>
<summary><strong>Что создается в папке LocalFunctionProj?</strong></summary>

Эта папка содержит различные файлы для проекта, такие как *Function.java*, *FunctionTest.java* и *pom.xml*, а также файлы конфигурации с именами [local.settings.json](functions-run-local.md#local-settings-file) и [host.json](functions-host-json.md). Файл *local.settings.json* может содержать секреты, скачанные из Azure, поэтому файл по умолчанию исключен из системы управления версиями в *GITIGNORE*-файле.
</details>

<br/>
<details>
<summary><strong>Код для Function.java</strong></summary>

Файл *Function.java* содержит метод `run`, получающий данные запроса в переменной `request`. Это запрос [HttpRequestMessage](/java/api/com.microsoft.azure.functions.httprequestmessage), дополненный заметкой [HttpTrigger](/java/api/com.microsoft.azure.functions.annotation.httptrigger), которая определяет поведение триггера. 

:::code language="java" source="~/azure-functions-samples-java/src/main/java/com/functions/Function.java":::

Сообщение-ответ создается API [HttpResponseMessage.Builder](/java/api/com.microsoft.azure.functions.httpresponsemessage.builder).

Архетип также создает модульный тест для функции. При изменении функции для добавления привязок или добавления новых функций в проект необходимо также изменить тесты в файле *FunctionTest.java*.
</details>

<br/>
<details>
<summary><strong>Код для pom.xml</strong></summary>

Параметры ресурсов Azure, созданных для размещения приложения, определяются в элементе **конфигурации** подключаемого модуля с **groupId** `com.microsoft.azure` в созданном файле *pom.xml*. Например, элемент конфигурации ниже указывает развертыванию на основе Maven, что приложение-функцию необходимо создать в группе ресурсов `java-functions-group` в расположении `westus`. <abbr title="Географическая ссылка на конкретный центр обработки данных Azure, в котором распределены ресурсы.">region</abbr>. Само приложение-функция работает в Windows, размещенном в плане `java-functions-app-service-plan`, который по умолчанию является бессерверным планом потребления.

:::code language="java" source="~/azure-functions-samples-java/pom.xml" range="62-107":::

Вы можете изменить эти параметры, чтобы управлять созданием ресурсов в Azure, например, изменив `runtime.os` с `windows` на `linux` перед первоначальным развертыванием. Полный список параметров, поддерживаемых подключаемым модулем Maven, см. в [сведениях о конфигурации](https://github.com/microsoft/azure-maven-plugins/wiki/Azure-Functions:-Configuration-Details).
</details>

<br>
<hr/>

## <a name="3-run-the-function-locally"></a>3. Локальное выполнение функции

1. **Выполните функцию**, запустив локальное хост-приложение среды выполнения Функций Azure из папки *LocalFunctionProj*:

    ```console
    mvn clean package
    mvn azure-functions:run
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

    Если результат HttpExample не похож на пример выше, скорее всего, вы запустили основное приложение из папки, отличной от корневой папки проекта. В этом случае остановите основное приложение комбинацией клавиш <kbd>Ctrl+C</kbd>, перейдите в корневую папку проекта и снова выполните указанную выше команду.

1. **Скопируйте URL-адрес** функции `HttpExample` из этих выходных данных в браузер и добавьте строку запроса `?name=<YOUR_NAME>`, сформировав полный URL-адрес, например `http://localhost:7071/api/HttpExample?name=Functions`. В браузере должно отобразиться сообщение, например `Hello Functions`:

    ![Получившаяся функция выполняется локально в браузере](./media/functions-create-first-azure-function-azure-cli/function-test-local-browser.png)

    Терминал, в котором вы запустили проект, также выводит данные журнала при выполнении запросов.

1. Когда все будет готово, нажмите клавиши <kbd>Ctrl+C</kbd> и выберите <kbd>y</kbd>, чтобы отключить основное приложение функций.

<br>
<hr/>

## <a name="4-deploy-the-function-project-to-azure"></a>4. Развертывание проекта функций в Azure

Приложение-функция и связанные ресурсы создаются в Azure при первом развертывании проекта функции. Параметры ресурсов Azure, созданных для размещения приложения, определяются в файле *pom.xml*. В этой статье вы примете значения по умолчанию.

<br/>
<details>
<summary><strong>Чтобы создать приложение-функцию, работающее на Linux</strong></summary>

Чтобы создать приложение-функцию, работающее на Linux, а не Windows, измените элемент `runtime.os` в файле *pom.xml* с `windows` на `linux`. Работа Linux в плане потребления поддерживается в [этих регионах](https://github.com/Azure/azure-functions-host/wiki/Linux-Consumption-Regions). Приложения, работающие в Linux, и приложения, работающие под управлением ОС Windows, не могут находиться в одной группе ресурсов.
</details>

1. Прежде чем выполнять развертывание, войдите в подписку Azure с помощью Azure CLI или Azure PowerShell. 

    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
    ```azurecli
    az login
    ```

    Чтобы войти в учетную запись Azure, выполните команду [az login](/cli/azure/reference-index#az-login).

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

    Развертывание упаковывает файлы проекта и развертывает их в новом приложении-функции [из ZIP-файла](functions-deployment-technologies.md#zip-deploy) Код выполняется из пакета развертывания в Azure.

<br/>
<details>
<summary><strong>Что создано в Azure?</strong></summary>

+ группа ресурсов; С именем _java-functions-group_.
+ учетная запись хранения; Требуется для Функций Azure. Это имя создается случайным образом на основе требований к именованию учетных записей хранения.
+ План размещения. Бессерверное размещение для приложения-функции в регионе, который указан в параметре _westus_. Имя _java-functions-app-service-plan_.
+ Приложение-функция. Приложение-функция представляет собой минимальную единицу развертывания и выполнения для ваших функций. Имя создается случайным образом на основе _artifactId_, к которому добавляется случайное число.
</details>

<br>
<hr/>

## <a name="5-invoke-the-function-on-azure"></a>5. Вызов функции в Azure

Функция использует триггер HTTP, поэтому ее необходимо **вызывать через HTTP-запрос к URL-адресу** в браузере или с помощью такого средства, как <abbr title="Программа командной строки для создания HTTP-запросов к URL-адресу. См. https://curl.se/">curl</abbr>.

# <a name="browser"></a>[Браузер](#tab/browser)

Скопируйте полный URL-адрес вызова **Invoke URL**, показанный в выходных данных команды `publish`, в адресную строку браузера, добавив параметр запроса `&name=Functions`. В браузере должны отображаться выходные данные, аналогичные данным при локальном запуске функции.

![Выходные данные функции, выполняемой в Azure в браузере](../../includes/media/functions-run-remote-azure-cli/function-test-cloud-browser.png)

# <a name="curl"></a>[curl](#tab/curl)

Запустите [`curl`](https://curl.haxx.se/), используя **Invoke URL** и добавив параметр `&name=Functions`. Результатом выполнения этой команды должен быть текст Hello Functions.

![Выходные данные функции, выполненной в Azure в cURL](../../includes/media/functions-run-remote-azure-cli/function-test-cloud-curl.png)

---

[!INCLUDE [functions-streaming-logs-cli-qs](../../includes/functions-streaming-logs-cli-qs.md)]

<br>
<hr/>

## <a name="6-clean-up-resources"></a>6. Очистка ресурсов

Если вы планируете перейти к [следующему шагу](#next-steps) и добавить выходную привязку очереди <abbr title="(средство в службе хранилища Azure для связывания функции с очередью хранилища, с его помощью можно будет создавать сообщения в очереди)">службы хранилища Azure</abbr>, сохраните все ресурсы в одном месте, поскольку на их основе будут создаваться другие элементы.

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

<br>
<hr/>

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Подключение Функций Azure к службе хранилища Azure с помощью средств командной строки](functions-add-output-binding-storage-queue-cli.md?pivots=programming-language-java)
