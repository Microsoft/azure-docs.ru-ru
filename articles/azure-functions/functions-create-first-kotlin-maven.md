---
title: Создание функции Kotlin в Функциях Azure с помощью Maven
description: В этой статье описывается создание и публикация в Функциях Azure приложения-функции, активируемого через HTTP, с помощью Kotlin и Maven.
author: dglover
ms.service: azure-functions
ms.topic: quickstart
ms.date: 03/25/2020
ms.author: dglover
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 721fbaf1a3b2418677d07e8ac39fa733164c1459
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102179567"
---
# <a name="quickstart-create-your-first-function-with-kotlin-and-maven"></a>Краткое руководство. Создание первой функции с помощью Kotlin и Maven

В этой статье показано, как использовать средство командной строки Maven для создания и публикации проекта функции Kotlin в службе "Функции Azure". После выполнения действий, описанных в этой статье, код функции будет выполняться в Azure в [Плане потребления](consumption-plan.md) и может запускаться с помощью HTTP-запроса.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

Для разработки функций с помощью Kotlin должны быть установлены следующие компоненты:

- [Java Developer Kit (JDK)](/azure/developer/java/fundamentals/java-jdk-long-term-support) версии 8.
- [Apache Maven](https://maven.apache.org) 3.0 или более поздней версии.
- [Azure CLI](/cli/azure)
- [Azure Functions Core Tools](./functions-run-local.md#v2) 2.6.666 или более поздней версии.

> [!IMPORTANT]
> Переменной среде JAVA_HOME необходимо присвоить расположение установки JDK, чтобы завершить выполнение заданий этого краткого руководства.

## <a name="generate-a-new-azure-functions-project"></a>Создание проекта Функций Azure

В пустой папке выполните следующую команду, чтобы создать проект Функций Azure из [архетипа Maven](https://maven.apache.org/guides/introduction/introduction-to-archetypes.html).

# <a name="bash"></a>[bash](#tab/bash)
```bash
mvn archetype:generate \
    -DarchetypeGroupId=com.microsoft.azure \
    -DarchetypeArtifactId=azure-functions-kotlin-archetype
```

> [!NOTE]
> Если не удается запустить команду, проверьте, какая версия `maven-archetype-plugin` используется. Так как команда выполняется в пустом каталоге без файла `.pom`, она может пытаться использовать более старую версию плагина из расположения `~/.m2/repository/org/apache/maven/plugins/maven-archetype-plugin`, если вы обновили Maven из предыдущей версии. В таком случае попробуйте удалить каталог `maven-archetype-plugin` и повторно выполнить команду.

# <a name="powershell"></a>[PowerShell](#tab/powershell)
```powershell
mvn archetype:generate `
    "-DarchetypeGroupId=com.microsoft.azure" `
    "-DarchetypeArtifactId=azure-functions-kotlin-archetype"
```

# <a name="cmd"></a>[Cmd](#tab/cmd)
```cmd
mvn archetype:generate ^
    -DarchetypeGroupId=com.microsoft.azure ^
    -DarchetypeArtifactId=azure-functions-kotlin-archetype
```
---

Maven запрашивает значения, необходимые для завершения создания проекта. Дополнительную информацию о значениях _groupId_, _artifactId_ и о _версии_ см. по ссылке [соглашения об именовании Maven](https://maven.apache.org/guides/mini/guide-naming-conventions.html). Значение _appName_ должно быть уникальным в Azure, поэтому Maven создает имя приложения на основе ранее заданного по умолчанию _artifactId_. Значение _packageName_ определяет пакет Kotlin для создаваемого кода функции.

Идентификаторы `com.fabrikam.functions` и `fabrikam-functions` ниже используются в качестве примера и для понимания дальнейших шагов в этом кратком руководстве. На этом шаге для Maven рекомендуется ввести собственные значения.

<pre>
[INFO] Parameter: groupId, Value: com.fabrikam.function
[INFO] Parameter: artifactId, Value: fabrikam-function
[INFO] Parameter: version, Value: 1.0-SNAPSHOT
[INFO] Parameter: package, Value: com.fabrikam.function
[INFO] Parameter: packageInPathFormat, Value: com/fabrikam/function
[INFO] Parameter: appName, Value: fabrikam-function-20190524171507291
[INFO] Parameter: resourceGroup, Value: java-functions-group
[INFO] Parameter: package, Value: com.fabrikam.function
[INFO] Parameter: version, Value: 1.0-SNAPSHOT
[INFO] Parameter: groupId, Value: com.fabrikam.function
[INFO] Parameter: appRegion, Value: westus
[INFO] Parameter: artifactId, Value: fabrikam-function
</pre>

Maven создает файлы проекта в новой папке с именем _artifactId_, в этом примере — `fabrikam-functions`. Созданный в проекте и готовый для запуска код — это простая функция [активации HTTP](./functions-bindings-http-webhook.md), возвращающая текст запроса.

```kotlin
class Function {

    /**
     * This function listens at endpoint "/api/HttpTrigger-Java". Two ways to invoke it using "curl" command in bash:
     * 1. curl -d "HTTP Body" {your host}/api/HttpTrigger-Java&code={your function key}
     * 2. curl "{your host}/api/HttpTrigger-Java?name=HTTP%20Query&code={your function key}"
     * Function Key is not needed when running locally, it is used to invoke function deployed to Azure.
     * More details: https://aka.ms/functions_authorization_keys
     */
    @FunctionName("HttpTrigger-Java")
    fun run(
            @HttpTrigger(
                    name = "req",
                    methods = [HttpMethod.GET, HttpMethod.POST],
                    authLevel = AuthorizationLevel.FUNCTION) request: HttpRequestMessage<Optional<String>>,
            context: ExecutionContext): HttpResponseMessage {

        context.logger.info("HTTP trigger processed a ${request.httpMethod.name} request.")

        val query = request.queryParameters["name"]
        val name = request.body.orElse(query)

        name?.let {
            return request
                    .createResponseBuilder(HttpStatus.OK)
                    .body("Hello, $name!")
                    .build()
        }

        return request
                .createResponseBuilder(HttpStatus.BAD_REQUEST)
                .body("Please pass a name on the query string or in the request body")
                .build()
    }
}
```

## <a name="run-the-function-locally"></a>Локальное выполнение функции

Измените каталог на только что созданную папку проекта, создайте и запустите функцию с помощью Maven.

```
cd fabrikam-function
mvn clean package
mvn azure-functions:run
```

> [!NOTE]
> При возникновении исключения `javax.xml.bind.JAXBException` с Java 9 инструкции по его устранению см. на [GitHub](https://github.com/jOOQ/jOOQ/issues/6477).

Вы увидите эти выходные данные, если функция выполняется локально на компьютере и готова отвечать на запросы HTTP:

<pre>
Now listening on: http://0.0.0.0:7071
Application started. Press Ctrl+C to shut down.

Http Functions:

        HttpTrigger-Java: [GET,POST] http://localhost:7071/api/HttpTrigger-Java
</pre>

Вызовите функцию из командной строки, используя curl в новом окне терминала:

```
curl -w '\n' -d LocalFunction http://localhost:7071/api/HttpTrigger-Java
```

<pre>
Hello LocalFunction!
</pre>

Используйте `Ctrl-C` в терминале, чтобы остановить код функции.

## <a name="deploy-the-function-to-azure"></a>Развертывание функции для Azure

В процессе развертывания для Функций Azure используются данные учетной записи из Azure CLI. Прежде чем продолжить, [войдите с помощью Azure CLI](/cli/azure/authenticate-azure-cli).

```azurecli
az login
```

Разверните свой код в новом приложении-функции, используя целевой объект Maven `azure-functions:deploy`.

> [!NOTE]
> При использовании Visual Studio Code для развертывания приложения-функции необходимо выбрать платную подписку, иначе появится сообщение об ошибке. Подписка отображается в левой области интегрированной среды разработки.

```
mvn azure-functions:deploy
```

После завершения развертывания появится URL-адрес, который можно использовать для доступа к приложению-функции:

<pre>
[INFO] Successfully deployed Function App with package.
[INFO] Deleting deployment package from Azure Storage...
[INFO] Successfully deleted deployment package fabrikam-function-20170920120101928.20170920143621915.zip
[INFO] Successfully deployed Function App at https://fabrikam-function-20170920120101928.azurewebsites.net
[INFO] ------------------------------------------------------------------------
</pre>

Проверьте приложение-функцию, работающее в Azure, используя `cURL`. Необходимо изменить URL-адрес из примера ниже, чтобы он соответствовал развернутому URL-адресу приложения-функции из предыдущего шага.

> [!NOTE]
> Убедитесь, что для **прав доступа** установлено значение `Anonymous`. Когда вы выбираете уровень по умолчанию `Function`, вы должны указывать [функциональную клавишу](functions-bindings-http-webhook-trigger.md#authorization-keys) в запросах на доступ к конечной точке вашей функции.

```
curl -w '\n' https://fabrikam-function-20170920120101928.azurewebsites.net/api/HttpTrigger-Java -d AzureFunctions
```

```Output
Hello AzureFunctions!
```

## <a name="make-changes-and-redeploy"></a>Внесение изменений и повторное развертывание

Измените исходный файл `src/main.../Function.java` в созданном проекте, чтобы изменить текст, возвращаемый приложением-функцией. Измените эту строку:

```kotlin
return request
        .createResponseBuilder(HttpStatus.OK)
        .body("Hello, $name!")
        .build()
```

В следующий код:

```kotlin
return request
        .createResponseBuilder(HttpStatus.OK)
        .body("Hi, $name!")
        .build()
```

Сохраните изменения и выполните повторное развертывание, запустив `azure-functions:deploy` в окне терминала, как и в прошлый раз. Приложение-функция и этот запрос обновятся:

```
curl -w '\n' -d AzureFunctionsTest https://fabrikam-functions-20170920120101928.azurewebsites.net/api/HttpTrigger-Java
```

Вы увидите обновленные выходные данные:

<pre>
Hi, AzureFunctionsTest
</pre>


## <a name="reference-bindings"></a>Указание ссылки на привязки

Для работы с [триггерами и привязками функций Azure](functions-triggers-bindings.md), отличных от триггера HTTP и триггера таймера, необходимо установить расширения привязки. Хотя это и не требуется в этой статье, необходимо знать, как включить расширения при работе с другими типами привязок.

[!INCLUDE [functions-extension-bundles](../../includes/functions-extension-bundles.md)]

## <a name="next-steps"></a>Дальнейшие действия

Вы создали приложение-функцию Kotlin с помощью простого триггера HTTP и развернули его в Функциях Azure.

- Дополнительные сведения о разработке функций Java и Kotlin см. в статье [Руководство разработчика Java по Функциям Azure](functions-reference-java.md).
- Добавьте в проект дополнительные функции с помощью различных триггеров целевого объекта Maven`azure-functions:add`.
- Создавайте и отлаживайте функции локально с помощью [Visual Studio Code](https://code.visualstudio.com/docs/java/java-azurefunctions), [IntelliJ](functions-create-maven-intellij.md) и [Eclipse](functions-create-maven-eclipse.md). 
- Отлаживайте функции, развернутые в Azure, с помощью Visual Studio Code. Инструкции см. в документации Visual Studio Code по [бессерверным приложениям Java](https://code.visualstudio.com/docs/java/java-serverless#_remote-debug-functions-running-in-the-cloud).
