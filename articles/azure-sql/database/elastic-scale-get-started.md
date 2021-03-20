---
title: Начало работы с инструментами эластичных баз данных
description: Объяснение основных принципов инструментов эластичных баз данных SQL Azure, в том числе пример простого для запуска приложения.
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: anumjs
ms.author: anjangsh
ms.reviewer: sstein
ms.date: 01/25/2019
ms.openlocfilehash: 74343b2f05bb4a59e475449c87524ff66cdd605d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98919550"
---
# <a name="get-started-with-elastic-database-tools"></a>Начало работы с инструментами эластичных баз данных
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

В этом документе описано, как разработать [клиентскую библиотеку эластичной базы данных](elastic-database-client-library.md), которая поможет запустить пример приложения. Используя пример приложения, мы создадим простое сегментированное приложение и изучим основные возможности инструментов эластичных баз данных SQL Azure. В этом документе описано, как [управлять сопоставлениями сегментов](elastic-scale-shard-map-management.md), [настроить маршрутизацию, зависящую от данных](elastic-scale-data-dependent-routing.md), и [создавать многосегментные запросы](elastic-scale-multishard-querying.md). Клиентская библиотека доступна для .NET и Java.

## <a name="elastic-database-tools-for-java"></a>Инструменты эластичных баз данных для Java

### <a name="prerequisites"></a>Предварительные условия

* Java Developer Kit (JDK) версии 1.8 или более поздней
* [Maven](https://maven.apache.org/download.cgi)
* База данных SQL или локальный экземпляр SQL Server

### <a name="download-and-run-the-sample-app"></a>Загрузка и запуск примера приложения

Чтобы создать JAR-файлы и начать работу с примером проекта, сделайте следующее:

1. Клонируйте [репозиторий GitHub](https://github.com/Microsoft/elastic-db-tools-for-java), содержащий клиентскую библиотеку и пример приложения.

2. Измените файл _./sample/src/main/resources/resource.properties_, указав следующее:
    * TEST_CONN_USER;
    * TEST_CONN_PASSWORD;
    * TEST_CONN_SERVER_NAME.

3. Чтобы создать пример проекта, в каталоге _./sample_ выполните следующую команду:

    ```
    mvn install
    ```

4. Чтобы запустить пример проекта, в каталоге _./sample_ выполните следующую команду:

    ```
    mvn -q exec:java "-Dexec.mainClass=com.microsoft.azure.elasticdb.samples.elasticscalestarterkit.Program"
    ```

5. Попробуйте устанавливать различные значения параметров, чтобы более подробно исследовать возможности клиентской библиотеки. Вы можете просмотреть код, чтобы узнать, как реализуется пример приложения.

    ![Описание процесса для Java][5]

Поздравляем! Вы успешно создали и запустили свое первое сегментированное приложение с помощью инструментов эластичных баз данных SQL Azure. Используйте Visual Studio или SQL Server Management Studio для подключения к базе данных и кратко Взгляните на сегменты, созданные этим образцом. Таким образом можно увидеть новые сегментированные базы данных и базу данных диспетчера сопоставлений сегментов, созданные демонстрационным приложением.

Чтобы включить клиентскую библиотеку в свой проект Maven, добавьте следующие зависимости в файл POM:

```xml
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>elastic-db-tools</artifactId>
    <version>1.0.0</version>
</dependency>
```

## <a name="elastic-database-tools-for-net"></a>Инструменты эластичных баз данных для .NET

### <a name="prerequisites"></a>Предварительные условия

* Visual Studio 2012 или более поздней версии с C#. Загрузите бесплатную версию на странице [Загрузок Visual Studio](https://www.visualstudio.com/downloads/download-visual-studio-vs.aspx).
* NuGet 2.7 или более поздней версии. Сведения о получении последней версии см. в разделе [Установка NuGet](https://docs.nuget.org/docs/start-here/installing-nuget).

### <a name="download-and-run-the-sample-app"></a>Загрузка и запуск примера приложения

Чтобы установить библиотеку, перейдите по ссылке [Microsoft.Azure.SqlDatabase.ElasticScale.Client](https://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client/). Эта библиотека устанавливается с примером приложения, описанным в следующем разделе.

Чтобы загрузить и запустить демонстрационное приложение, выполните следующие действия.

1. Скачайте [Пример инструментов эластичной базы данных для SQL Azure начало работы](https://github.com/Azure/elastic-db-tools). Распакуйте пример в выбранное расположение.

2. Чтобы создать проект, откройте решение *еластикдатабасетулс. sln* из каталога *эластичная БД-Tools-Master* . 

3. Задайте проект *решение elasticscalestarterkit* в качестве запускаемого проекта.

4. В проекте *решение elasticscalestarterkit* откройте файл *App.config* . Затем следуйте инструкциям в файле, чтобы добавить имя сервера и сведения о входе (имя пользователя и пароль).

5. Выполните сборку и запуск приложения. После соответствующего запроса разрешите Visual Studio восстановить пакеты NuGet решения. В результате из NuGet скачивается последняя версия клиентской библиотеки эластичной базы данных.

6. Попробуйте устанавливать различные значения параметров, чтобы более подробно исследовать возможности клиентской библиотеки. Обратите внимание на то, какие действия выполняет приложение, отслеживая выводимые им в консоль сообщения, и ознакомьтесь с отвечающим за эти действия программным кодом.

   ![Ход выполнения][4]

Поздравляем! Вы успешно создали и запустили свое первое сегментированное приложение с помощью инструментов эластичных баз данных SQL. Используйте Visual Studio или SQL Server Management Studio для подключения к базе данных и кратко Взгляните на сегменты, созданные этим образцом. Таким образом можно увидеть новые сегментированные базы данных и базу данных диспетчера сопоставлений сегментов, созданные демонстрационным приложением.

> [!IMPORTANT]
> Чтобы обеспечить синхронизацию с обновлениями Azure и базы данных SQL, рекомендуется всегда использовать последнюю версию Management Studio. [Обновите среду SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms).

## <a name="key-pieces-of-the-code-sample"></a>Ключевые фрагменты программного кода демонстрационного приложения

* **Управление сегментами и сопоставлениями сегментов.** Этот программный код, демонстрирующий, каким образом следует работать с сегментами, диапазонами и сопоставлениями, взят из файла *ShardManagementUtils.cs*. Дополнительные сведения см. в статье [Развертывание баз данных с использованием диспетчера карты сегментов](https://go.microsoft.com/?linkid=9862595).  

* **Маршрутизация, зависящая от данных.** Маршрутизация транзакций к необходимому сегменту демонстрируется в файле *DataDependentRoutingSample.cs*. Дополнительные сведения см. в статье [Маршрутизация, зависящая от данных](https://go.microsoft.com/?linkid=9862596).

* **Формирование запросов по нескольким сегментам.** Формирование запросов по сегментам демонстрируется в файле *MultiShardQuerySample.cs*. Дополнительные сведения см. в статье [Многосегментное формирование запросов](https://go.microsoft.com/?linkid=9862597).

* **Добавление пустых сегментов.** Итеративное добавление новых пустых сегментов выполняется программным кодом, который приведен в файле *CreateShardSample.cs*. Дополнительные сведения см. в статье [Развертывание баз данных с использованием диспетчера карты сегментов](https://go.microsoft.com/?linkid=9862595).

## <a name="other-elastic-scale-operations"></a>Другие операции, относящиеся к эластичному масштабированию

* **Разбиение имеющегося сегмента.** Возможность разбиения сегментов реализована с помощью инструмента разбиения и объединения. Дополнительные сведения см. в статье [Перемещение данных между масштабируемыми облачными базами данных](elastic-scale-overview-split-and-merge.md).

* **Объединение имеющихся сегментов.** Объединение сегментов также выполняется с помощью инструмента разбиения и объединения. Дополнительные сведения см. в статье [Перемещение данных между масштабируемыми облачными базами данных](elastic-scale-overview-split-and-merge.md).

## <a name="cost"></a>Cost

Библиотека инструментов эластичных баз данных предоставляется бесплатно. При использовании инструментов эластичных баз данных не взимаются какие-либо дополнительные платежи, помимо оплаты за работу на платформе Azure.

Например, демонстрационное приложение создает новую базу данных. Плата за эту возможность зависит от выбранной версии базы данных SQL и от использования приложением платформы Azure.

Сведения о ценах см. в разделе [сведения о ценах на базу данных SQL](https://azure.microsoft.com/pricing/details/sql-database/).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об инструментах эластичных баз данных см. в приведенных ниже статьях.

* Примеры кода:
  * Инструменты эластичных баз данных ([.NET](https://github.com/Azure/elastic-db-tools), [Java](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22azure-elasticdb-tools%22))
  * [Elastic DB Tools for Azure SQL - Entity Framework Integration](https://code.msdn.microsoft.com/Elastic-Scale-with-Azure-bae904ba?SRC=VSIDE) (Инструменты эластичных баз данных SQL Azure — интеграция с Entity Framework)
  * [Эластичность сегментов в Центре сценариев](https://gallery.technet.microsoft.com/scriptcenter/Elastic-Scale-Shard-c9530cbe)
* Блог: [объявление, касающееся эластичного масштабирования](https://azure.microsoft.com/blog/20../../introducing-elastic-scale-preview-for-azure-sql-database/)
* Channel 9: [видеообзор технологии эластичного масштабирования](https://channel9.msdn.com/Shows/Data-Exposed/Azure-SQL-Database-Elastic-Scale)
* Форум обсуждений: [Microsoft Q&страница вопросов для базы данных SQL Azure](/answers/topics/azure-sql-database.html)
* Измерение производительности: [Счетчики производительности для диспетчера карты сегментов](elastic-database-client-library.md)

<!--Anchors-->
[The Elastic Scale Sample Application]: #The-Elastic-Scale-Sample-Application
[Download and Run the Sample App]: #Download-and-Run-the-Sample-App
[Cost]: #Cost
[Next steps]: #next-steps

<!--Image references-->
[1]: ./media/elastic-scale-get-started/newProject.png
[2]: ./media/elastic-scale-get-started/click-online.png
[3]: ./media/elastic-scale-get-started/click-CSharp.png
[4]: ./media/elastic-scale-get-started/output2.png
[5]: ./media/elastic-scale-get-started/java-client-library.PNG