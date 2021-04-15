---
title: Краткое руководство. Первый запрос на Java
description: В этом кратком руководстве показано, как включить пакеты Maven Resource Graph для Java и выполнить первый запрос.
ms.date: 03/30/2021
ms.topic: quickstart
ms.custom: devx-track-java
ms.openlocfilehash: 97c04cb8b8180034bdc5109446c79deb56e457c9
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/02/2021
ms.locfileid: "106223861"
---
# <a name="quickstart-run-your-first-resource-graph-query-using-java"></a>Краткое руководство. Первый запрос Resource Graph с помощью Java

При использовании Azure Resource Graph в первую очередь необходимо проверить, установлены ли необходимые пакеты Maven для Java. В этом кратком руководстве показано, как добавить пакеты Maven в установку Java.

В результате вы добавите пакеты Maven в установку Java и выполните запрос Resource Graph.

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure. Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

- Убедитесь, что установлена последняя версия Azure CLI (не ниже **2.21.0**). Если Azure CLI еще не установлен, см. [эту статью](/cli/azure/install-azure-cli).

  > [!NOTE]
  > Azure CLI требуется для того, чтобы с помощью пакета SDK Azure для Java можно было использовать **проверки подлинности на основе CLI** в следующих примерах. Дополнительные способы аутентификации см. в описании [клиентской библиотеки Azure Identity для Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/identity/azure-identity).

- [Java Developer Kit](/azure/developer/java/fundamentals/java-jdk-long-term-support) версии
  8.

- [Apache Maven](https://maven.apache.org/) версии 3.6 или более поздней.

## <a name="create-the-resource-graph-project"></a>Создание проекта Resource Graph

Чтобы с помощью Java можно было отправлять запросы к Azure Resource Graph, создайте и настройте новое приложение с помощью Maven и установите необходимые пакеты Maven.

1. Инициализируйте новое приложение Java с именем "argQuery" с помощью [архетипа Maven](https://maven.apache.org/guides/introduction/introduction-to-archetypes.html):

   ```cmd
   mvn -B archetype:generate -DarchetypeGroupId="org.apache.maven.archetypes" -DgroupId="com.Fabrikam" -DartifactId="argQuery"
   ```

1. Перейдите в новую папку проекта `argQuery` и откройте `pom.xml` в редакторе. Добавьте следующие узлы `<dependency>` в существующий узел `<dependencies>`:

   ```xml
    <dependency>
        <groupId>com.azure</groupId>
        <artifactId>azure-identity</artifactId>
        <version>1.2.4</version>
    </dependency>
    <dependency>
        <groupId>com.azure.resourcemanager</groupId>
        <artifactId>azure-resourcemanager-resourcegraph</artifactId>
        <version>1.0.0-beta.1</version>
    </dependency>
   ```

1. В файле `pom.xml` добавьте следующий узел `<properties>` в базовый узел `<project>`, чтобы обновить исходную и целевую версии:

   ```xml
   <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
   </properties>
   ```

1. В файле `pom.xml` добавьте следующий узел `<build>` в базовый узел `<project>`, чтобы настроить цель и основной класс для запуска проекта.

   ```xml
   <build>
       <plugins>
           <plugin>
               <groupId>org.codehaus.mojo</groupId>
               <artifactId>exec-maven-plugin</artifactId>
               <version>1.2.1</version>
               <executions>
                   <execution>
                       <goals>
                           <goal>java</goal>
                       </goals>
                   </execution>
               </executions>
               <configuration>
                   <mainClass>com.Fabrikam.App</mainClass>
               </configuration>
           </plugin>
       </plugins>
   </build>
   ```

1. Замените файл `App.java` в `\argQuery\src\main\java\com\Fabrikam` следующим кодом и сохраните обновленный файл:

   ```java
   package com.Fabrikam;
   
   import java.util.Arrays;
   import java.util.List;
   import com.azure.core.management.AzureEnvironment;
   import com.azure.core.management.profile.AzureProfile;
   import com.azure.identity.DefaultAzureCredentialBuilder;
   import com.azure.resourcemanager.resourcegraph.ResourceGraphManager;
   import com.azure.resourcemanager.resourcegraph.models.QueryRequest;
   import com.azure.resourcemanager.resourcegraph.models.QueryRequestOptions;
   import com.azure.resourcemanager.resourcegraph.models.QueryResponse;
   import com.azure.resourcemanager.resourcegraph.models.ResultFormat;
   
   public class App
   {
       public static void main( String[] args )
       {
           List<String> listSubscriptionIds = Arrays.asList(args[0]);
           String strQuery = args[1];
   
           ResourceGraphManager manager = ResourceGraphManager.authenticate(new DefaultAzureCredentialBuilder().build(), new AzureProfile(AzureEnvironment.AZURE));
   
           QueryRequest queryRequest = new QueryRequest()
               .withSubscriptions(listSubscriptionIds)
               .withQuery(strQuery);
           
           QueryResponse response = manager.resourceProviders().resources(queryRequest);
   
           System.out.println("Records: " + response.totalRecords());
           System.out.println("Data:\n" + response.data());
       }
   }
   ```

1. Постройте консольное приложение `argQuery`:

   ```bash
   mvn package
   ```

## <a name="run-your-first-resource-graph-query"></a>Выполните первый запрос график ресурсов

Теперь, когда консольное приложение Java создано и опубликовано, попробуем выполнить простой запрос к Resource Graph. Запрос возвращает первые пять ресурсов Azure с указанными значениями **Имя** и **Тип ресурса** для каждого ресурса.

В каждом вызове к `argQuery` есть переменные, которые необходимо заменить собственными значениями:

- `{subscriptionId}` — замените это значение идентификатором своей подписки.
- `{query}` — замените запросом Azure Resource Graph.

1. Используйте Azure CLI для проверки подлинности с помощью `az login`.

1. Перейдите в папку проекта `argQuery`, которую вы создали с помощью команды `mvn -B archetype:generate`.

1. Запустите первый запрос Azure Resource Graph с помощью Maven, чтобы скомпилировать консольное приложение и передать аргументы. Свойство `exec.args` определяет аргументы по пробелам. Чтобы найти запрос в виде одного аргумента, мы заключим его в одинарные кавычки (`'`).

   ```bash
   mvn compile exec:java -Dexec.args "{subscriptionId} 'Resources | project name, type | limit 5'"
   ```

   > [!NOTE]
   > Так как этот пример запроса не содержит модификатор сортировки, такой как `order by`, повторное выполнение запроса может каждый раз возвращать разные наборы ресурсов.

1. Обновите запрос к `argQuery.exe` и измените запрос на `order by` для свойства **Name**:

   ```bash
   mvn compile exec:java -Dexec.args "{subscriptionId} 'Resources | project name, type | limit 5 | order by name asc'"
   ```

   > [!NOTE]
   > Как и с первым запросом, выполнение этого запроса несколько раз может получить различные наборы ресурсов для каждого запроса. Важен порядок команд запроса. В этом примере `order by` следует после `limit`. Эта последовательность команд сначала ограничивает результаты запроса, а затем упорядочивает их.

1. Измените значение последнего параметра на `argQuery.exe`, чтобы сначала выполнить сортировку (`order by`) по свойству **Name**, а затем ограничить вывод (`limit`) первыми пятью результатами:

   ```bash
   mvn compile exec:java -Dexec.args "{subscriptionId} 'Resources | project name, type | order by name asc | limit 5'"
   ```

Повторное выполнение последней версии запроса, если ничего не изменяется в самой среде, возвращает стабильные результаты, отсортированные по свойству **Name** и ограниченные пятью первыми результатами.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите удалить консольное приложение Java и установленные пакеты, удалите папку проекта `argQuery`.

## <a name="next-steps"></a>Дальнейшие действия

С помощью инструкций из этого краткого руководства вы создали консольное приложение Java с необходимыми пакетами Resource Graph и выполнили свой первый запрос. Чтобы узнать больше о языке Resource Graph, перейдите на страницу сведений о языке запросов.

> [!div class="nextstepaction"]
> [Получите более подробную информацию о языке запросов](./concepts/query-language.md).