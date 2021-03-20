---
title: Справочник для разработчика Java по Функциям Azure
description: Информация о разработке функций на языке Java.
ms.topic: conceptual
ms.date: 09/14/2018
ms.custom: devx-track-java, devx-track-azurecli
ms.openlocfilehash: 1ffbd760ae75605d75652b29d379420d6946aa8f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96326460"
---
# <a name="azure-functions-java-developer-guide"></a>Руководство разработчика Java по Функциям Azure

Это краткое описание содержит подробные сведения, помогающие в разработке функций Azure с помощью Java.

Если вы не знакомы с функциями Azure в качестве разработчика Java, сначала ознакомьтесь с одной из следующих статей:

| Начало работы | Основные понятия| 
| -- | -- |  
| <ul><li>[Функция Java с использованием Visual Studio Code](./create-first-function-vs-code-java.md)</li><li>[Функция Java/Maven в терминале / командной строке](./create-first-function-cli-java.md)</li><li>[Функция Java с использованием Gradle](functions-create-first-java-gradle.md)</li><li>[Функция Java с использованием Eclipse](functions-create-maven-eclipse.md)</li><li>[Функция Java, использующая идею IntelliJ](functions-create-maven-intellij.md)</li></ul> | <ul><li>[Руководство для разработчиков](functions-reference.md)</li><li>[Сравнение вариантов размещения](functions-scale.md)</li><li>[&nbsp;Вопросы производительности](functions-best-practices.md)</li></ul> |

## <a name="java-function-basics"></a>Основы функций Java

Функцией в Java называется метод `public`, дополненный аннотацией `@FunctionName`. Такой метод определяет запись для функции Java и должен быть уникальным в пределах пакета. Пакет может иметь несколько классов с несколькими открытыми методами, заметками в `@FunctionName` . Один пакет развертывается в приложении-функции в Azure. При работе в Azure приложение функции предоставляет контекст развертывания, выполнения и управления для отдельных функций Java.

## <a name="programming-model"></a>Модель программирования 

Понятия [триггеров и привязок](functions-triggers-bindings.md) играют решающую роль в Функциях Azure. Триггеры запускают выполнение вашего кода. Привязки предоставляют возможность передавать данные, а также возвращать их из функции без необходимости написания кода доступа к ним.

## <a name="create-java-functions"></a>Создание функций на Java

Чтобы упростить создание функций на Java, созданы специальные средства и архетипы на основе Maven, которые предоставляют готовые шаблоны Java для создания проектов с определенным триггером функции.    

### <a name="maven-based-tooling"></a>Инструментарий на основе Maven

В следующих средах разработки есть инструментарий Функций Azure, который позволяет создавать проекты для функций Java: 

+ [Visual Studio Code](https://code.visualstudio.com/docs/java/java-azurefunctions)
+ [Eclipse](functions-create-maven-eclipse.md)
+ [IntelliJ](functions-create-maven-intellij.md)

В статьях, доступных по предложенным выше ссылкам, демонстрируется создание простых функций в удобной для вас интегрированной среде разработки. 

### <a name="project-scaffolding"></a>Формирование шаблонов для проекта

Если вы предпочитаете выполнять разработку из командной строки терминала, сформировать шаблоны для проектов функций на основе Java вам проще всего будет с использованием архетипов `Apache Maven`. Архетип Maven для Функций Azure на Java опубликован со следующими идентификаторами _groupId_:_artifactId_: [com.microsoft.azure:azure-functions-archetype](https://search.maven.org/artifact/com.microsoft.azure/azure-functions-archetype/). 

Следующая команда создает новый проект функции Java на основе этого архетипа:

# <a name="bash"></a>[Bash](#tab/bash)

```bash
mvn archetype:generate \
    -DarchetypeGroupId=com.microsoft.azure \
    -DarchetypeArtifactId=azure-functions-archetype
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```cmd
mvn archetype:generate ^
    -DarchetypeGroupId=com.microsoft.azure ^
    -DarchetypeArtifactId=azure-functions-archetype
```

---

Чтобы приступить к использованию архетипа, воспользуйтесь [кратким руководством по Java](./create-first-function-cli-java.md).

## <a name="folder-structure"></a>Структура папок

Ниже приведена структура папок в проекте Функций Azure на Java:

```
FunctionsProject
 | - src
 | | - main
 | | | - java
 | | | | - FunctionApp
 | | | | | - MyFirstFunction.java
 | | | | | - MySecondFunction.java
 | - target
 | | - azure-functions
 | | | - FunctionApp
 | | | | - FunctionApp.jar
 | | | | - host.json
 | | | | - MyFirstFunction
 | | | | | - function.json
 | | | | - MySecondFunction
 | | | | | - function.json
 | | | | - bin
 | | | | - lib
 | - pom.xml
```

Вы можете использовать общий файл [host.json](functions-host-json.md) для настройки приложения-функции. У каждой функции есть собственный файл кода (.java) и файл конфигурации привязки (function.json).

Вы можете добавить несколько функций в проект. Не добавляйте функции в отдельные JAR-файлы. Именно `FunctionApp` из целевого каталога будет развернут в виде приложения-функции в Azure.

## <a name="triggers-and-annotations"></a>Триггеры и заметки

 Функции вызываются триггером, таким как HTTP-запрос, таймер или обновление данных. Функция должна обработать такой триггер и связанные с ним входные данные, а также создать на их основе одно или несколько выходных значений.

Используйте заметки Java, включенные в пакет [com.microsoft.azure.functions.annotation.*](/java/api/com.microsoft.azure.functions.annotation), чтобы привязать входные и выходные данные к своим методам. Дополнительные сведения см. в [справочной документации по Java](/java/api/com.microsoft.azure.functions.annotation).

> [!IMPORTANT] 
> Чтобы локально обрабатывать триггеры хранилища BLOB-объектов, очередей или таблиц Azure, необходимо указать учетную запись хранения Azure в файле [local.settings.json](./functions-run-local.md#local-settings-file).

Пример

```java
public class Function {
    public String echo(@HttpTrigger(name = "req", 
      methods = {HttpMethod.POST},  authLevel = AuthorizationLevel.ANONYMOUS) 
        String req, ExecutionContext context) {
        return String.format(req);
    }
}
```

Ниже приведен соответствующий файл `function.json`, созданный с помощью [azure-functions-maven-plugin](https://mvnrepository.com/artifact/com.microsoft.azure/azure-functions-maven-plugin):

```json
{
  "scriptFile": "azure-functions-example.jar",
  "entryPoint": "com.example.Function.echo",
  "bindings": [
    {
      "type": "httpTrigger",
      "name": "req",
      "direction": "in",
      "authLevel": "anonymous",
      "methods": [ "post" ]
    },
    {
      "type": "http",
      "name": "$return",
      "direction": "out"
    }
  ]
}

```

## <a name="java-versions"></a>Версии Java

Версия Java, используемая при создании приложения-функции, в которой выполняются функции в Azure, указывается в файле pom.xml. Maven архетипа в настоящее время создает pom.xml для Java 8, который можно изменить перед публикацией. Версия Java в pom.xml должна соответствовать версии, на которой локально разрабатывается и протестировано приложение. 

### <a name="supported-versions"></a>Поддерживаемые версии

В следующей таблице показаны текущие поддерживаемые версии Java для каждой основной версии среды выполнения функций по операционной системе:

| Версия службы "Функции" | Версии Java (Windows) | Версии Java (Linux) |
| ----- | ----- | --- |
| 3.x | 11 <br/>8 | 11 <br/>8 |
| 2.x | 8 | Н/Д |

Если для развертывания не указана версия Java, Maven архетипа по умолчанию имеет значение Java 8 во время развертывания в Azure.

### <a name="specify-the-deployment-version"></a>Укажите версию развертывания

Вы можете управлять версией Java, целевой для Maven архетипа, с помощью `-DjavaVersion` параметра. Значение этого параметра может быть либо `8` `11` . 

Maven архетипа создает pom.xml, предназначенную для указанной версии Java. Следующие элементы в pom.xml указывают используемую версию Java:

| Элемент |  Значение Java 8 | Значение Java 11 | Описание |
| ---- | ---- | ---- | --- |
| **`Java.version`** | 1.8 | 11 | Версия Java, используемая подключаемым модулем Maven-компилятора. |
| **`JavaVersion`** | 8 | 11 | Версия Java, размещенная в приложении функции в Azure. |

В следующих примерах показаны параметры для Java 8 в соответствующих разделах файла pom.xml:

#### `Java.version`
:::code language="xml" source="~/functions-quickstart-java/functions-add-output-binding-storage-queue/pom.xml" range="12-19" highlight="14":::

#### `JavaVersion`
:::code language="xml" source="~/functions-quickstart-java/functions-add-output-binding-storage-queue/pom.xml" range="77-85" highlight="80":::

> [!IMPORTANT]
> Необходимо правильно установить JAVA_HOME переменную среды в каталог JDK, используемый во время компиляции кода с помощью Maven. Убедитесь, что версия JDK не меньше максимальной, чем значение `Java.version` параметра. 

### <a name="specify-the-deployment-os"></a>Укажите ОС развертывания

Maven также позволяет указать операционную систему, в которой выполняется приложение функции в Azure. Используйте `os` элемент, чтобы выбрать операционную систему. 

| Элемент |  Windows | Linux | Docker |
| ---- | ---- | ---- | --- |
| **`os`** | Windows | Linux | docker |

В следующем примере показан параметр операционной системы в `runtime` разделе файла pom.xml:

:::code language="xml" source="~/functions-quickstart-java/functions-add-output-binding-storage-queue/pom.xml" range="77-85" highlight="79":::
 
## <a name="jdk-runtime-availability-and-support"></a>Обеспечения доступности и предоставления поддержки времени выполнения пакета JDK 

Для локальной разработки приложений функций Java Скачайте и используйте соответствующее [Azul Zulu Enterprise для Azure](https://assets.azul.com/files/Zulu-for-Azure-FAQ.pdf) Java JDK от [Azul Systems](https://www.azul.com/downloads/azure-only/zulu/). В функциях Azure используется Azul Java JDK Runtime при развертывании приложения функции в облаке.

[Поддержка Azure](https://azure.microsoft.com/support/) для устранения проблем с пакетами JDK и приложениями-функциями предоставляется, если у вас есть [соответствующий план поддержки](https://azure.microsoft.com/support/plans/).

## <a name="customize-jvm"></a>Настройка виртуальной машины Java

Функции Azure позволяют настраивать виртуальную машину Java, которая используется для выполнения функций Java. По умолчанию применяются [следующие параметры виртуальной машины Java](https://github.com/Azure/azure-functions-java-worker/blob/master/worker.config.json#L7).

* `-XX:+TieredCompilation`
* `-XX:TieredStopAtLevel=1`
* `-noverify` 
* `-Djava.net.preferIPv4Stack=true`
* `-jar`

Дополнительные аргументы можно указать в параметре приложения `JAVA_OPTS`. Параметры приложения можно добавить в приложение-функцию, развернутое в Azure, с помощью портала Azure или Azure CLI.

> [!IMPORTANT]  
> В план потребления необходимо также добавить параметр WEBSITE_USE_PLACEHOLDER со значением 0, чтобы работала эта настройка. Но этот параметр увеличивает время холодного запуска для функций Java.

### <a name="azure-portal"></a>Портал Azure

На [портале Azure](https://portal.azure.com) перейдите на вкладку [Параметры приложения](functions-how-to-use-azure-function-app-settings.md#settings) чтобы добавить параметр `JAVA_OPTS`.

### <a name="azure-cli"></a>Azure CLI

Используйте команду [az functionapp config appsettings set](/cli/azure/functionapp/config/appsettings), чтобы задать значение `JAVA_OPTS`, как показано далее.

# <a name="consumption-plan"></a>[План потребления](#tab/consumption/bash)

```azurecli-interactive
az functionapp config appsettings set \
    --settings "JAVA_OPTS=-Djava.awt.headless=true" \
    "WEBSITE_USE_PLACEHOLDER=0" \
    --name <APP_NAME> --resource-group <RESOURCE_GROUP>
```

# <a name="consumption-plan"></a>[План потребления](#tab/consumption/cmd)

```azurecli-interactive
az functionapp config appsettings set ^
    --settings "JAVA_OPTS=-Djava.awt.headless=true" ^
    "WEBSITE_USE_PLACEHOLDER=0" ^
    --name <APP_NAME> --resource-group <RESOURCE_GROUP>
```

# <a name="dedicated-plan--premium-plan"></a>[Выделенный план или план "Премиум"](#tab/dedicated+premium/bash)

```azurecli-interactive
az functionapp config appsettings set \
    --settings "JAVA_OPTS=-Djava.awt.headless=true" \
    --name <APP_NAME> --resource-group <RESOURCE_GROUP>
```

# <a name="dedicated-plan--premium-plan"></a>[Выделенный план или план "Премиум"](#tab/dedicated+premium/cmd)

```azurecli-interactive
az functionapp config appsettings set ^
    --settings "JAVA_OPTS=-Djava.awt.headless=true" ^
    --name <APP_NAME> --resource-group <RESOURCE_GROUP>
```

---

В этом примере показано, как включить режим без монитора. Замените в нем заполнитель `<APP_NAME>` именем приложения-функции, а `<RESOURCE_GROUP>` — именем группы ресурсов. 

## <a name="third-party-libraries"></a>Сторонние библиотеки 

Служба "Функции Azure" поддерживает использование сторонних библиотек. По умолчанию все зависимости, указанные в файле `pom.xml` для проекта, автоматически связываются при сборке с целью [`mvn package`](https://github.com/Microsoft/azure-maven-plugins/blob/master/azure-functions-maven-plugin/README.md#azure-functionspackage). Зависимости библиотек, не указанных как зависимости в файле `pom.xml`, поместите в каталог `lib` в корневой папке функции. Зависимости, размещенные в каталоге `lib`, добавляются в загрузчик системного класса во время выполнения.

Зависимость `com.microsoft.azure.functions:azure-functions-java-library` по умолчанию предоставляется в пути классов, и ее не нужно добавлять в каталог `lib`. Кроме того, [azure-functions-java-worker](https://github.com/Azure/azure-functions-java-worker) добавляет в путь классов все перечисленные [здесь](https://github.com/Azure/azure-functions-java-worker/wiki/Azure-Java-Functions-Worker-Dependencies) зависимости.

## <a name="data-type-support"></a>Поддержка типов данных

Для входных и выходных привязок можно использовать объекты POJO, определенные в `azure-functions-java-library` типы, или примитивные типы данных, как например String или Integer.

### <a name="pojos"></a>Объекты POJO

Для преобразования входных данных в формат POJO [azure-functions-java-worker](https://github.com/Azure/azure-functions-java-worker) использует библиотеку [gson](https://github.com/google/gson). Типам POJO, используемые как входные данные для функций, необходим такой же модификатор доступа `public`.

### <a name="binary-data"></a>Двоичные данные

Чтобы создать привязку для ввода и вывода к `byte[]`, в файле function.json задайте для поля `dataType` значение `binary`.

```java
   @FunctionName("BlobTrigger")
    @StorageAccount("AzureWebJobsStorage")
     public void blobTrigger(
        @BlobTrigger(name = "content", path = "myblob/{fileName}", dataType = "binary") byte[] content,
        @BindingName("fileName") String fileName,
        final ExecutionContext context
    ) {
        context.getLogger().info("Java Blob trigger function processed a blob.\n Name: " + fileName + "\n Size: " + content.length + " Bytes");
    }
```

Если допускаются значения NULL, добавьте `Optional<T>`.

## <a name="bindings"></a>Привязки

Входные и выходные привязки реализуют декларативный способ подключения к данным из кода. У функции может быть несколько входных и выходных привязок.

### <a name="input-binding-example"></a>Пример привязки входа

```java
package com.example;

import com.microsoft.azure.functions.annotation.*;

public class Function {
    @FunctionName("echo")
    public static String echo(
        @HttpTrigger(name = "req", methods = { HttpMethod.PUT }, authLevel = AuthorizationLevel.ANONYMOUS, route = "items/{id}") String inputReq,
        @TableInput(name = "item", tableName = "items", partitionKey = "Example", rowKey = "{id}", connection = "AzureWebJobsStorage") TestInputData inputData,
        @TableOutput(name = "myOutputTable", tableName = "Person", connection = "AzureWebJobsStorage") OutputBinding<Person> testOutputData
    ) {
        testOutputData.setValue(new Person(httpbody + "Partition", httpbody + "Row", httpbody + "Name"));
        return "Hello, " + inputReq + " and " + inputData.getKey() + ".";
    }

    public static class TestInputData {
        public String getKey() { return this.RowKey; }
        private String RowKey;
    }
    public static class Person {
        public String PartitionKey;
        public String RowKey;
        public String Name;

        public Person(String p, String r, String n) {
            this.PartitionKey = p;
            this.RowKey = r;
            this.Name = n;
        }
    }
}
```

Эта функция вызывается с помощью HTTP-запроса. 
- Полезные данные HTTP-запроса передаются в формате `String` через аргумент `inputReq`.
- Из Хранилища таблиц извлекается одна запись, которая передается в формате `TestInputData` через аргумент `inputData`.

Чтобы получить пакет входных данных, выполните привязку к `String[]`, `POJO[]`, `List<String>` или `List<POJO>`.

```java
@FunctionName("ProcessIotMessages")
    public void processIotMessages(
        @EventHubTrigger(name = "message", eventHubName = "%AzureWebJobsEventHubPath%", connection = "AzureWebJobsEventHubSender", cardinality = Cardinality.MANY) List<TestEventData> messages,
        final ExecutionContext context)
    {
        context.getLogger().info("Java Event Hub trigger received messages. Batch size: " + messages.size());
    }
    
    public class TestEventData {
    public String id;
}

```

Эта функция запускается всякий раз, когда в настроенном концентраторе событий появляются новые данные. Так как для `cardinality` установлено значение `MANY`, функция получает пакет сообщений от концентратора событий. Полученный от концентратора событий `EventData` преобразуется в `TestEventData` для выполнения функции.

### <a name="output-binding-example"></a>Пример выходной привязки

Вы можете создать привязку для вывода к возвращаемому значению с помощью `$return`. 

```java
package com.example;

import com.microsoft.azure.functions.annotation.*;

public class Function {
    @FunctionName("copy")
    @StorageAccount("AzureWebJobsStorage")
    @BlobOutput(name = "$return", path = "samples-output-java/{name}")
    public static String copy(@BlobTrigger(name = "blob", path = "samples-input-java/{name}") String content) {
        return content;
    }
}
```

При наличии нескольких выходных привязок используйте возвращаемое значение только для одной из них.

Чтобы отправить несколько выходных значений, используйте тип `OutputBinding<T>`, который определен в пакете `azure-functions-java-library`. 

```java
@FunctionName("QueueOutputPOJOList")
    public HttpResponseMessage QueueOutputPOJOList(@HttpTrigger(name = "req", methods = { HttpMethod.GET,
            HttpMethod.POST }, authLevel = AuthorizationLevel.ANONYMOUS) HttpRequestMessage<Optional<String>> request,
            @QueueOutput(name = "itemsOut", queueName = "test-output-java-pojo", connection = "AzureWebJobsStorage") OutputBinding<List<TestData>> itemsOut, 
            final ExecutionContext context) {
        context.getLogger().info("Java HTTP trigger processed a request.");
       
        String query = request.getQueryParameters().get("queueMessageId");
        String queueMessageId = request.getBody().orElse(query);
        itemsOut.setValue(new ArrayList<TestData>());
        if (queueMessageId != null) {
            TestData testData1 = new TestData();
            testData1.id = "msg1"+queueMessageId;
            TestData testData2 = new TestData();
            testData2.id = "msg2"+queueMessageId;

            itemsOut.getValue().add(testData1);
            itemsOut.getValue().add(testData2);

            return request.createResponseBuilder(HttpStatus.OK).body("Hello, " + queueMessageId).build();
        } else {
            return request.createResponseBuilder(HttpStatus.INTERNAL_SERVER_ERROR)
                    .body("Did not find expected items in CosmosDB input list").build();
        }
    }

     public static class TestData {
        public String id;
    }
```

Эта функция вызывается через HttpRequest. Она записывает несколько значений в Хранилище очередей.

## <a name="httprequestmessage-and-httpresponsemessage"></a>HttpRequestMessage и HttpResponseMessage

 Они определяются в `azure-functions-java-library`. Это вспомогательные типы для работы с функциями типа HttpTrigger.

| Специализированные типы      |       Назначение        | Типичное применение                  |
| --------------------- | :-----------------: | ------------------------------ |
| `HttpRequestMessage<T>`  |    Триггер HTTP     | Получение метода, заголовков или запросов |
| `HttpResponseMessage` | Привязка к выходным данным HTTP | Возврат кодов состояния, отличных от 200   |

## <a name="metadata"></a>Метаданные

Несколько триггеров отправляют [метаданные триггеров](./functions-triggers-bindings.md) вместе с входными данными. Вы можете использовать аннотацию `@BindingName` для привязки к метаданным триггера.


```Java
package com.example;

import java.util.Optional;
import com.microsoft.azure.functions.annotation.*;


public class Function {
    @FunctionName("metadata")
    public static String metadata(
        @HttpTrigger(name = "req", methods = { HttpMethod.GET, HttpMethod.POST }, authLevel = AuthorizationLevel.ANONYMOUS) Optional<String> body,
        @BindingName("name") String queryValue
    ) {
        return body.orElse(queryValue);
    }
}
```
В приведенном выше примере значение `queryValue` привязано к параметру строки запроса `name` в URL-адресе HTTP-запроса `http://{example.host}/api/metadata?name=test`. Вот еще один пример привязки к `Id` из метаданных триггера очереди.

```java
 @FunctionName("QueueTriggerMetadata")
    public void QueueTriggerMetadata(
        @QueueTrigger(name = "message", queueName = "test-input-java-metadata", connection = "AzureWebJobsStorage") String message,@BindingName("Id") String metadataId,
        @QueueOutput(name = "output", queueName = "test-output-java-metadata", connection = "AzureWebJobsStorage") OutputBinding<TestData> output,
        final ExecutionContext context
    ) {
        context.getLogger().info("Java Queue trigger function processed a message: " + message + " with metadaId:" + metadataId );
        TestData testData = new TestData();
        testData.id = metadataId;
        output.setValue(testData);
    }
```

> [!NOTE]
> Предоставленное в заметке имя должно соответствовать свойству метаданных.

## <a name="execution-context"></a>Контекст выполнения

Контекст `ExecutionContext`, определенный в `azure-functions-java-library`, содержит вспомогательные методы для взаимодействия со средой выполнения функций. Дополнительные сведения см. в [справочной статье](/java/api/com.microsoft.azure.functions.executioncontext)по параметру контексте.

### <a name="logger"></a>Средство ведения журнала

Используйте средство `getLogger`, определенное в `ExecutionContext`, для записи журналов из кода функции.

Пример

```java

import com.microsoft.azure.functions.*;
import com.microsoft.azure.functions.annotation.*;

public class Function {
    public String echo(@HttpTrigger(name = "req", methods = {HttpMethod.POST}, authLevel = AuthorizationLevel.ANONYMOUS) String req, ExecutionContext context) {
        if (req.isEmpty()) {
            context.getLogger().warning("Empty request body received by function " + context.getFunctionName() + " with invocation " + context.getInvocationId());
        }
        return String.format(req);
    }
}
```

## <a name="view-logs-and-trace"></a>Просмотр журналов и трассировки

Вы можете использовать интерфейс Azure CLI для потоковой передачи журналов со стандартными данными и журналов ошибок в формате Java, а также ведения других журналов приложений. 

Ниже показано, как настроить для приложения-функции запись журнала приложения с использованием Azure CLI.

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli-interactive
az webapp log config --name functionname --resource-group myResourceGroup --application-logging true
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```azurecli-interactive
az webapp log config --name functionname --resource-group myResourceGroup --application-logging true
```

---

Чтобы выполнять потоковую передачу выходных данных журналов для приложения-функции через Azure CLI, откройте новое окно командной строки, сеанс Bash или терминала и введите следующую команду:

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli-interactive
az webapp log tail --name webappname --resource-group myResourceGroup
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```azurecli-interactive
az webapp log tail --name webappname --resource-group myResourceGroup
```

---

Команда [az webapp log tail](/cli/azure/webapp/log) позволяет фильтровать выходные данные с помощью параметра `--provider`. 

Чтобы скачать файлы журналов как единый ZIP-файл с помощью Azure CLI, откройте командную строку, Bash или сеанс терминала и введите следующую команду:

```azurecli-interactive
az webapp log download --resource-group resourcegroupname --name functionappname
```

Перед выполнением этой команды необходимо включить ведение журнала файловой системы с помощью портала Azure или Azure CLI.

## <a name="environment-variables"></a>Переменные среды

В Функциях [параметры приложения](functions-app-settings.md), такие как строки подключения службы, доступны в виде переменных среды во время выполнения. Доступ к этим параметрам можно получить с помощью `System.getenv("AzureWebJobsStorage")`.

В следующем примере, показано как получить [параметр приложения](functions-how-to-use-azure-function-app-settings.md#settings) по ключу с именем `myAppSetting`:

```java

public class Function {
    public String echo(@HttpTrigger(name = "req", methods = {HttpMethod.POST}, authLevel = AuthorizationLevel.ANONYMOUS) String req, ExecutionContext context) {
        context.getLogger().info("My app setting value: "+ System.getenv("myAppSetting"));
        return String.format(req);
    }
}

```

> [!NOTE]
> Значение параметра приложения FUNCTIONS_EXTENSION_VERSION должно быть ~2 или ~3, чтобы оптимизировать процесс холодного запуска.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о разработке в службе "Функции Azure" на Java см. в следующих статьях.

* [Рекомендации по функциям Azure](functions-best-practices.md)
* [Справочник разработчика по функциям Azure](functions-reference.md)
* [Azure Functions triggers and bindings (Триггеры и привязки в Функциях Azure)](functions-triggers-bindings.md)
* Разработка и отладка в локальной среде с помощью [Visual Studio Code](https://code.visualstudio.com/docs/java/java-azurefunctions), [IntelliJ](functions-create-maven-intellij.md) и [Eclipse](functions-create-maven-eclipse.md).
* [Дистанционная отладка функций на Java с использованием Visual Studio Code](https://code.visualstudio.com/docs/java/java-serverless#_remote-debug-functions-running-in-the-cloud)
* [Подключаемый модуль Maven для Функций Azure](https://github.com/Microsoft/azure-maven-plugins/blob/develop/azure-functions-maven-plugin/README.md) 
* Упростите создание функций благодаря использованию цели `azure-functions:add` и подготовьте промежуточный каталог [для развертывания ZIP-файла](deployment-zip-push.md).
