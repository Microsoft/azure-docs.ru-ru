---
title: Руководство по созданию веб-приложения Java с использованием Azure Cosmos DB и API SQL
description: Руководство по В этом руководстве по разработке веб-приложения Java показано, как использовать службу Azure Cosmos DB и API SQL для хранения данных и доступа к ним из приложения Java, размещенного на веб-сайтах Azure.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: tutorial
ms.date: 02/10/2021
ms.author: anfeldma
ms.custom: devx-track-java
ms.openlocfilehash: 489f4aad78437edad2184e773d97d35af24f3249
ms.sourcegitcommit: b8995b7dafe6ee4b8c3c2b0c759b874dff74d96f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/03/2021
ms.locfileid: "106286000"
---
# <a name="tutorial-build-a-java-web-application-using-azure-cosmos-db-and-the-sql-api"></a>Руководство по Создание веб-приложения Java с использованием Azure Cosmos DB и API SQL
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [.NET](sql-api-dotnet-application.md)
> * [Java](sql-api-java-application.md)
> * [Node.js](sql-api-nodejs-application.md)
> * [Python](./create-sql-api-python.md)
> * [Xamarin](mobile-apps-with-xamarin.md)
> 

В этом руководстве по разработке веб-приложения Java показано, как использовать службу [Microsoft Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) для хранения данных и обеспечения доступа к ним из приложения Java, размещенного в Azure при помощи функции "Веб-приложения службы приложений". В этой статье вы узнаете следующее:

* Как создать основное приложение JavaServer Pages (JSP) в Eclipse.
* Как работать со службой Azure Cosmos DB с помощью [пакета SDK для Java для Azure Cosmos DB](https://github.com/Azure/azure-documentdb-java).

В данном руководстве рассказывается о том, как создать веб-приложение для управления задачами, с помощью которого можно создавать и извлекать задачи, а также помечать их как завершенные, как показано на рисунке ниже. Каждая задача из списка ToDo будет храниться в виде документов JSON в службе Azure Cosmos DB.

:::image type="content" source="./media/sql-api-java-application/image1.png" alt-text="Приложение My ToDo List на Java":::

> [!TIP]
> В данном учебнике по разработке приложения предполагается, что у вас имеется некоторый опыт использования Java. Если вы никогда не работали с Java или [необходимыми инструментами](#Prerequisites), мы рекомендуем скачать полный учебный проект [todo](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-todo-app) с портала GitHub и создать его, следуя [инструкциям, приведенным в конце этой статьи](#GetProject). После создания ознакомьтесь со статьей, чтобы разобраться в коде этого проекта.  

## <a name="prerequisites-for-this-java-web-application-tutorial"></a><a id="Prerequisites"></a>Необходимые условия для изучения этого учебника по разработке веб-приложения Java

Для работы с этим учебником необходимы:

* Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу. 

  [!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

* [Комплект разработчика Java (JDK 7 +)](/java/azure/jdk/).
* [Интегрированная среда разработки Eclipse для разработчиков Java EE.](https://www.eclipse.org/downloads/packages/release/luna/sr1/eclipse-ide-java-ee-developers)
* [Открытый веб-сайт Azure со средой выполнения Java (например, Tomcat или Jetty).](../app-service/quickstart-java.md)

При первой установке этих средств воспользуйтесь пошаговыми инструкциями, представленными в разделе "Быстрый запуск" статьи [Tutorial: Installing TomCat7 and Using it with Eclipse](https://www.youtube.com/watch?v=jOdCfW7-ybI&t=2s) (Руководство по установке TomCat7 и его использованию с Eclipse) на сайте coreservlets.com.

## <a name="create-an-azure-cosmos-db-account"></a><a id="CreateDB"></a>Создание учетной записи Azure Cosmos DB

Давайте сначала создадим учетную запись Azure Cosmos DB. Если у вас уже есть учетная запись или вы используете эмулятор Azure Cosmos DB для работы с этим руководством, можно перейти к разделу [Шаг 2. Создание приложения Java JSP](#CreateJSP).

[!INCLUDE [create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

[!INCLUDE [keys](../../includes/cosmos-db-keys.md)]

## <a name="create-the-java-jsp-application"></a><a id="CreateJSP"></a>Создание приложения Java JSP

Для создания приложения JSP:

1. Сначала необходимо создать проект Java. В меню Eclipse выберите **File** (Файл), щелкните **New** (Создать), а затем выберите **Dynamic Web Project** (Динамический веб-проект). Если элемента **Dynamic Web Project** (Динамический веб-проект) нет в списке доступных проектов, откройте меню **File** (Файл), щелкните пункт **New** (Создать), а затем выберите **Project** (Проект), разверните список **Web** (Интернет), выберите **Dynamic Web Project** (Динамический веб-проект) и нажмите кнопку **Next** (Далее).
   
    :::image type="content" source="./media/sql-api-java-application/image10.png" alt-text="Разработка приложений JSP Java":::

1. Введите имя проекта в **соответствующем** поле и в раскрывающемся меню **Target Runtime** (Целевая среда выполнения). Если необходимо, выберите значение (например, Apache Tomcat v7.0), а затем нажмите кнопку **Finish** (Готово). Выбор целевой среды выполнения позволит вам запустить локальный проект через Eclipse.

1. В обозревателе проектов Eclipse разверните проект. Щелкните правой кнопкой мыши **WebContent**, выберите **Создать**, затем щелкните **JSP File** (JSP-файл).

1. В диалоговом окне **New JSP File** (Новый JSP-файл) укажите для файла имя **index.jsp**. Сохраните родительскую папку с именем **WebContent**, как показано на следующей иллюстрации, и нажмите кнопку **Next** (Далее).
   
    :::image type="content" source="./media/sql-api-java-application/image11.png" alt-text="Создание файла JSP — учебник по разработке веб-приложений Java":::

1. В диалоговом окне **Select JSP Template** (Выбор шаблона JSP) в целях обучения выберите **New JSP File (html)** (Новый JSP-файл (html)) и нажмите кнопку **Finish** (Готово).

1. Открыв файл *index.jsp* в Eclipse, добавьте текст для отображения **Hello World!** в существующий элемент `<body>`. Обновленное содержимое элемента `<body>` должно отображаться следующим образом:

   ```html
   <body>
     <% out.println("Hello World!"); %>
   </body>
   ```

1. Сохраните файл *index.jsp*.

1. Если вы задали целевую среду выполнения в рамках шага 2, можете щелкнуть **Project** (Проект), а затем **Run** (Запуск) для локального запуска приложения JSP:

   :::image type="content" source="./media/sql-api-java-application/image12.png" alt-text="Hello World — руководство по разработке приложений Java":::

## <a name="install-the-sql-java-sdk"></a><a id="InstallSDK"></a>Установка пакета Java SDK для SQL

Самый простой способ извлечь данные из пакета Java SDK для SQL и его зависимости — использовать [Apache Maven](https://maven.apache.org/). Для этого необходимо преобразовать проект в проект Maven, выполнив приведенные ниже действия.

1. Щелкните правой кнопкой мыши проект в обозревателе проектов, выберите **Configure** (Настроить) и щелкните **Convert to Maven Project** (Преобразовать в проект Maven).

1. В окне **Create new POM** (Создать новый POM) примите параметры по умолчанию и нажмите кнопку **Finish** (Готово).

1. В **обозревателе проектов** откройте файл pom.xml.

1. На вкладке **Dependencies** (Зависимости) в **соответствующей** колонке щелкните **Add** (Добавить).

1. В окне **Select Dependency** (Выбор зависимости) выполните следующие действия.
   
   * В поле **Group Id** (Идентификатор группы) введите `com.azure`.
   * В поле **Artifact Id** (Идентификатор артефакта) введите `azure-cosmos`.
   * В поле **Version** (Версия) введите `4.11.0`.
  
   Либо можно добавить зависимость XML для идентификаторов группы и артефакта непосредственно в файл *pom.xml*:

   ```xml
   <dependency>
     <groupId>com.azure</groupId>
     <artifactId>azure-cosmos</artifactId>
     <version>4.11.0</version>
   </dependency>
   ```

1. Нажмите кнопку **OK**, и Maven установит пакет Java SDK для SQL или сохранит файл pom.xml.

## <a name="use-the-azure-cosmos-db-service-in-your-java-application"></a><a id="UseService"></a>Использование службы Azure Cosmos DB в приложении Java

Теперь мы добавим в веб-приложение модели, представления и контроллеры.

### <a name="add-a-model"></a>Добавление модели

Сначала определим модель в новом файле *TodoItem.java*. Класс `TodoItem` определяет схему элемента вместе с методами получения и задания:

:::code language="java" source="~/samples-cosmosdb-java-v4-web-app/src/com/microsoft/azure/cosmos/sample/model/TodoItem.java":::

### <a name="add-the-data-access-objectdao-classes"></a>Добавление классов объекта доступа к данным (DAO)

Создайте объект доступа к данным (DAO) для абстрагирования сохранения элементов ToDo в Azure Cosmos DB. Для сохранения элементов ToDo в коллекции клиент должен знать, какие базы данных и коллекции необходимо связать (с помощью самостоятельных ссылок). Как правило, лучше всего кэшировать базы данных и коллекции, если это возможно для того, чтобы избежать дополнительных обращений к базе данных.

1. Чтобы вызвать службу Azure Cosmos DB, необходимо создать объект `cosmosClient`. В общем, рекомендуется повторно использовать объект `cosmosClient` вместо создания клиента для каждого последующего запроса. Вы можете повторно использовать клиент, определив его в классе `cosmosClientFactory`. Обновите значения HOST и MASTER_KEY, сохраненные на [шаге 1](#CreateDB). Замените переменную HOST значением URI, а MASTER_KEY — значением первичного ключа. Используйте следующий код, чтобы создать класс `CosmosClientFactory` в файле *CosmosClientFactory.java*:

   :::code language="java" source="~/samples-cosmosdb-java-v4-web-app/src/com/microsoft/azure/cosmos/sample/dao/CosmosClientFactory.java":::

1. Создайте файл *TodoDao.java* и добавьте класс `TodoDao` для создания, обновления, чтения и удаления элементов ToDo:

   :::code language="java" source="~/samples-cosmosdb-java-v4-web-app/src/com/microsoft/azure/cosmos/sample/dao/TodoDao.java":::

1. Создайте файл *MockDao.java* и добавьте класс `MockDao`. Этот класс реализует класс `TodoDao` для выполнения операций CRUD с элементами:

   :::code language="java" source="~/samples-cosmosdb-java-v4-web-app/src/com/microsoft/azure/cosmos/sample/dao/MockDao.java":::

1. Создайте файл *DocDbDao.java* и добавьте класс `DocDbDao`. Этот класс определяет код для сохранения TodoItems в контейнере, извлекает базу данных и коллекцию, если она существует, или создает новую, если она не существует. В этом примере мы используем [Gson](https://code.google.com/p/google-gson/) для сериализации и десериализации элементов TodoItem POJO в документы JSON. Для сохранения элементов ToDo в коллекции клиент должен знать, какие базы данных и коллекции необходимо связать (с помощью самостоятельных ссылок). Этот класс также определяет вспомогательную функцию для получения документов с определенным атрибутом (например, ID) без использования ссылок на себя. Вспомогательный метод можно использовать для извлечения документа TodoItem JSON по идентификатору и затем десериализовать его в POJO.

   Вы также можете использовать клиентский объект `cosmosClient` для получения коллекции или перечня элементов TodoItems с помощью SQL-запроса. Наконец, определите метод Delete, чтобы удалить TodoItem из списка. В следующем коде представлено содержимое класса `DocDbDao`.

   :::code language="java" source="~/samples-cosmosdb-java-v4-web-app/src/com/microsoft/azure/cosmos/sample/dao/DocDbDao.java":::

1. Затем создайте файл *TodoDaoFactory.java* и добавьте класс `TodoDaoFactory`, который создает объект DocDbDao.

   :::code language="java" source="~/samples-cosmosdb-java-v4-web-app/src/com/microsoft/azure/cosmos/sample/dao/TodoDaoFactory.java":::

### <a name="add-a-controller"></a>Добавление контроллера

Добавьте в приложение контроллер *TodoItemController*. В этом проекте вы используете [проект Lombok](https://projectlombok.org/) для создания конструктора, получателя, задания и построителя. Кроме того, можно написать следующий код вручную либо создать его в IDE.

:::code language="java" source="~/samples-cosmosdb-java-v4-web-app/src/com/microsoft/azure/cosmos/sample/controller/TodoItemController.java":::

### <a name="create-a-servlet"></a>Создание сервлета

Далее создайте сервлет, чтобы перенаправлять запросы HTTP к контроллеру. Создайте файл *ApiServlet.java* и определите в нем приведенный ниже код.

:::code language="java" source="~/samples-cosmosdb-java-v4-web-app/src/com/microsoft/azure/cosmos/sample/ApiServlet.java":::

## <a name="wire-the-rest-of-the-of-java-app-together"></a><a id="Wire"></a>Подключение другой части приложения Java

Теперь нам необходимо создать пользовательский интерфейс и привязать его к нашему объекту DAO.

1. Вам потребуется пользовательский веб-интерфейс, который будет отображаться для пользователя. Повторно напишем файл *index.jsp*, созданный ранее, использовав следующий код:

   :::code language="java" source="~/samples-cosmosdb-java-v4-web-app/WebContent/index.jsp":::

1. Наконец, запишем несколько сценариев Javascript на стороне клиента, чтобы связать пользовательский веб-интерфейс и сервлет.

   :::code language="java" source="~/samples-cosmosdb-java-v4-web-app/WebContent/assets/todo.js":::

1. Теперь осталось протестировать приложение. Запустите приложение локально и добавьте несколько элементов Todo, заполнив имя элемента и категории и нажав кнопку **Add Task**(Добавить задачу). После появления элемента вы можете проверить его завершение, отметив флажком и щелкнув команду **Обновить задачи**.

## <a name="deploy-your-java-application-to-azure-web-sites"></a><a id="Deploy"></a>Развертывание приложения Java на веб-сайтах Azure

Веб-сайты Azure упрощают развертывание приложений Java с помощью простого экспорта приложения в виде WAR-файла либо с помощью загрузки через систему управления версиями (например, GIT) или через клиент FTP.

1. Чтобы экспортировать приложение в виде WAR-файла, щелкните правой кнопкой мыши проект в **обозревателе проектов**, выберите **Export** (Экспорт) и щелкните **WAR File** (WAR-файл).

1. В окне **WAR Export** (Экспорт WAR-файла) выполните следующие действия:
   
   * В поле веб-проекта введите azure-cosmos-java-sample.
   * В поле Destination (Назначение) выберите место назначения для сохранения WAR-файла.
   * Нажмите кнопку **Готово**.

1. Теперь, когда вы создали WAR-файл, можно просто передать его на веб-сайт Azure в каталог **webapps**. Инструкции по передаче файла см. в статье [Добавление приложения Java в веб-приложения службы приложений Azure](../app-service/quickstart-java.md). После загрузки WAR-файла в каталог веб-приложения среда выполнения обнаруживает, что вы добавили его, и автоматически его загрузит.

1. Чтобы оценить готовый продукт, перейдите к `http://YOUR\_SITE\_NAME.azurewebsites.net/azure-cosmos-java-sample/` и приступайте к добавлению задач.

## <a name="get-the-project-from-github"></a><a id="GetProject"></a>Получение проекта из GitHub

Все примеры в этом учебнике включены в проект [todo](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-todo-app) на портале GitHub. Чтобы импортировать проект todo в Eclipse, убедитесь, что у вас есть программное обеспечение и ресурсы, перечисленные в разделе [Необходимые условия](#Prerequisites) , а затем выполните следующие действия.

1. Установите [проект Lombok](https://projectlombok.org/). Lombok используется для формирования конструкторов, получателей и заданий в проекте. Дважды щелкните загруженный файл lombok.jar для установки или установите его из командной строки.

1. Если открыта рабочая область Eclipse, закройте ее и запустите снова для загрузки проекта Lombok.

1. В Eclipse откройте меню **File** (Файл) и щелкните **Import** (Импорт).

1. В окне **Import** (Импорт) щелкните **Git**, а затем выберите **Projects from Git** (Проекты из Git) и нажмите кнопку **Next** (Далее).

1. На экране **Select Repository Source** (Выбрать источник репозитория) щелкните **Clone URI** (Клон URI).

1. На экране **исходного репозитория Git** в поле **URI** введите https://github.com/Azure-Samples/azure-cosmos-java-sql-api-todo-app, а затем нажмите кнопку **Далее**.

1. На экране **Branch Selection** (Выбор ветви) выберите значение **main** и нажмите кнопку **Next** (Далее).

1. На экране **Local Destination** (Локальная папка назначения) нажмите кнопку **Browse** (Обзор), выберите папку, в которую можно копировать репозитории, и нажмите кнопку **Next** (Далее).

1. Проверьте, установлен ли на экране **Select a wizard to use for importing projects** (Выбор мастера для импорта проектов) флажок **Import existing projects** (Импортировать существующие проекты), и нажмите кнопку **Next** (Далее).

1. На экране **Import Projects** (Импорт проектов) снимите флажок возле проекта **DocumentDB** и нажмите кнопку **Finish** (Готово). В проекте DocumentDB содержится пакет Java SDK для Azure Cosmos DB, который будет добавлен в качестве зависимости.

1. В **обозревателе проектов** перейдите в расположение azure-cosmos-java-sample\src\com.microsoft.azure.cosmos.sample.dao\DocumentClientFactory.java и замените значения HOST и MASTER_KEY значениями URI и первичного ключа своей учетной записи Azure Cosmos DB, а затем сохраните файл. Дополнительные сведения см. в инструкции [Шаг 1. Создание учетной записи базы данных Azure Cosmos](#CreateDB).

1. В **обозревателе проектов** щелкните правой кнопкой элемент **azure-cosmos-java-sample**, а затем выберите **Build Path** (Путь сборки) и щелкните **Configure Build Path** (Настроить путь сборки).

1. На экране **Java Build Path** (Путь сборки Java) в правой области откройте вкладку **Libraries** (Библиотеки) и щелкните **Add External JAR** (Добавить внешние JAR-файлы). Перейдите в расположение файла lombok.jar, щелкните **Open** (Открыть), а затем нажмите кнопку **ОК**.

1. Используйте инструкцию из шага 12, то есть снова откройте окно **Properties** (Свойства), а затем в левой области щелкните **Targeted Runtimes** (Целевые среды выполнения).

1. На экране **Targeted Runtimes** (Целевые среды выполнения) щелкните **New** (Создать), выберите **Apache Tomcat v7.0** и нажмите кнопку **ОК**.

1. Используйте инструкцию из шага 12, то есть снова откройте окно **Properties** (Свойства), а затем в левой области щелкните **Project Facets** (Аспекты проекта).

1. На экране **Project Facets** (Аспекты проекта) выберите **Dynamic Web Module** (Динамический веб-модуль) и **Java**, а затем нажмите кнопку **ОК**.

1. На вкладке **Servers** (Серверы) в нижней части экрана щелкните правой кнопкой мыши **Tomcat v7.0 Server at localhost** (Tomcat v7.0 Server на localhost), а затем нажмите кнопку **Add and Remove** (Добавить или удалить).

1. В окне **Add and Remove** (Добавление и удаление) переместите элемент **azure-cosmos-java-sample** в поле **Configured** (Настроено), а затем нажмите кнопку **Finish** (Готово).

1. На вкладке **Server** (Сервер) щелкните правой кнопкой мыши **Tomcat v7.0 Server at localhost** (Сервер Tomcat версии 7.0 на localhost), а затем нажмите кнопку **Restart** (Перезапуск).

1. В браузере перейдите к `http://localhost:8080/azure-cosmos-java-sample/` и приступите к добавлению задач в список. Обратите внимание, что в случае изменения заданных по умолчанию значений портов необходимо изменить значение 8080 выбранным вами значением.

1. Инструкции по развертыванию проекта на веб-сайтах Azure см. в пункте [Шаг 6. Развертывание приложений на веб-сайтах Azure](#Deploy).

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Создание приложения Node.js с помощью Azure Cosmos DB](sql-api-nodejs-application.md)
