---
title: Работа с API таблиц Azure Cosmos DB с использованием пакета SDK для .NET Standard
description: Узнайте, как хранить и запрашивать структурированные данные в учетной записи API таблиц Azure Cosmos DB
author: sakash279
ms.author: akshanka
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 12/03/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: f4817e53cb36bc96add19d855750a73a151b4891
ms.sourcegitcommit: 2c1b93301174fccea00798df08e08872f53f669c
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104772303"
---
# <a name="get-started-with-azure-cosmos-db-table-api-and-azure-table-storage-using-the-net-sdk"></a>Начало работы с API таблиц Azure Cosmos DB и хранилищем таблиц Azure с помощью пакета SDK для .NET
[!INCLUDE[appliesto-table-api](includes/appliesto-table-api.md)]

[!INCLUDE [storage-selector-table-include](../../includes/storage-selector-table-include.md)]

[!INCLUDE [storage-table-applies-to-storagetable-and-cosmos](../../includes/storage-table-applies-to-storagetable-and-cosmos.md)]

API таблиц Azure Cosmos DB или хранилище таблиц Azure можно использовать для хранения структурированных NoSQL-данных в облаке, предоставляя хранилище для ключей и атрибутов с бессхемной структурой. Так как API таблиц Azure Cosmos DB и хранилище таблиц являются бессхемными, это позволяет легко адаптировать данные по мере роста приложения. API таблиц Azure Cosmos DB или хранилище таблиц можно использовать для хранения гибких наборов данных, например пользовательских данных для веб-приложений, адресных книг, сведений об устройстве или метаданных любого другого типа, которые требуются вашей службе. 

В этом учебнике описано, как использовать [библиотеки таблиц Azure Cosmos DB для .NET](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table) в сценариях работы с API таблиц Azure Cosmos DB и Хранилищем таблиц Azure. Необходимо использовать подключение, связанное с определенной службой Azure. Эти сценарии рассматриваются с использованием примеров на C#, которые демонстрируют методы создания таблиц, вставки и обновления данных, запроса данных и удаления таблиц.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим примером требуются следующие компоненты:

* [Microsoft Visual Studio](https://www.visualstudio.com/downloads/)

* [Библиотека таблиц Microsoft Azure CosmosDB для .NET](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table). Эта библиотека сейчас доступна для платформ .NET Standard и .NET. 

* [Учетная запись API таблиц Azure Cosmos DB](create-table-dotnet.md#create-a-database-account).

## <a name="create-an-azure-cosmos-db-table-api-account"></a>Создание учетной записи API таблиц Azure Cosmos DB

[!INCLUDE [cosmos-db-create-dbaccount-table](../../includes/cosmos-db-create-dbaccount-table.md)]

## <a name="create-a-net-console-project"></a>Создайте консольный проект .NET

Создайте консольное приложение .NET в Visual Studio. Ниже показано, как создать консольное приложение в Visual Studio 2019. Вы можете использовать библиотеку таблиц Azure Cosmos DB в любом приложении .NET, в том числе облачной службе Azure, веб-приложении Azure, а также классическом или мобильном приложении. Для упрощения в этом руководстве мы будем использовать консольное приложение.

1. Выберите **File** > **New** > **Project** ( Файл > Создать > Проект).

1. Выберите **Консольное приложение (.NET Core)** , а затем щелкните **Далее**.

1. В поле **Имя проекта** введите имя для приложения, например **CosmosTableSamples**. (При необходимости можно указать другое имя).

1. Нажмите кнопку **создания**.

Все примеры кода из этого примера можно добавить в метод Main() в файле **Program.cs** консольного приложения.

## <a name="install-the-required-nuget-package"></a>Установка необходимых пакетов NuGet

Чтобы получить пакеты NuGet, сделайте следующее.

1. Щелкните правой кнопкой мыши проект в **обозревателе решений** и выберите **Управление пакетами NuGet**.

1. Выполните поиск в Интернете по запросу [`Microsoft.Azure.Cosmos.Table`](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table), [`Microsoft.Extensions.Configuration`](https://www.nuget.org/packages/Microsoft.Extensions.Configuration), [`Microsoft.Extensions.Configuration.Json`](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.Json), [`Microsoft.Extensions.Configuration.Binder`](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.Binder) и выберите **Установить**, чтобы установить библиотеку таблиц Microsoft Azure Cosmos DB.

## <a name="configure-your-storage-connection-string"></a>Настройка строки подключения хранилища

1. На [портале Azure](https://portal.azure.com/) перейдите к учетной записи Azure Cosmos или Хранилища таблиц. 

1. Откройте панель **Строка подключения** или **Ключи доступа**. Используйте кнопку копирования в правой части окна, чтобы скопировать **основную строку подключения**.

   :::image type="content" source="./media/create-table-dotnet/connection-string.png" alt-text="Просмотр и копирование основной строки подключения на панели строки подключения":::
   
1. Чтобы настроить строку подключения, щелкните в Visual Studio правой кнопкой мыши проект **CosmosTableSamples**.

1. Выберите **Добавить**, а затем **Новый элемент**. Создайте файл **Settings.json** с типом файла **Файл конфигурации TypeScript JSON**. 

1. Замените код в файл Settings.json следующим кодом и укажите основную строку подключения.

   ```csharp
   {
   "StorageConnectionString": <Primary connection string of your Azure Cosmos DB account>
   }
   ```

1. Щелкните проект **CosmosTableSamples** правой кнопкой мыши. Выберите **Добавить**, **Новый элемент** и добавьте класс с именем **AppSettings.cs**.

1. Добавьте в файл AppSettings.cs указанный ниже код. Этот файл считывает строку подключения из файла Settings.json и назначает ее значение параметру конфигурации:

  :::code language="csharp" source="~/azure-cosmosdb-dotnet-table/CosmosTableSamples/AppSettings.cs":::

## <a name="parse-and-validate-the-connection-details"></a>Синтаксический анализ и проверка сведений о подключении

1. Щелкните проект **CosmosTableSamples** правой кнопкой мыши. Выберите **Добавить**, **Новый элемент** и добавьте класс с именем **Common.cs**. В этом классе вы создадите код для проверки сведений о подключении и создания таблицы.

1. Определите метод `CreateStorageAccountFromConnectionString`, как показано ниже. Этот метод будет анализировать данные строки подключения и проверять допустимость для имени учетной записи и ключа учетной записи, которые указаны в файле Settings.json.

   :::code language="csharp" source="~/azure-cosmosdb-dotnet-table/CosmosTableSamples/Common.cs" id="createStorageAccount":::

## <a name="create-a-table"></a>Создание таблицы 

Класс [CloudTableClient](/dotnet/api/microsoft.azure.cosmos.table.cloudtableclient) позволяет получать таблицы и сущности, хранящиеся в хранилище таблиц. Так как в учетной записи API таблиц Cosmos DB пока нет ни одной таблицы, давайте добавим метод `CreateTableAsync` для их создания в класс **Common.cs**:

:::code language="csharp" source="~/azure-cosmosdb-dotnet-table/CosmosTableSamples/Common.cs" id="CreateTable":::

Если вы получаете ошибку исключения 503 Service Unavailable (Служба недоступна), то возможно, что требуемые порты для прямого подключения блокируются брандмауэром. Чтобы устранить эту проблему, откройте необходимые порты или используйте подключение в режиме шлюза, как показано в следующем коде:

```csharp
tableClient.TableClientConfiguration.UseRestExecutorForCosmosEndpoint = true;
```

## <a name="define-the-entity"></a>Определение сущности 

Сущности сопоставляются с объектами C# с помощью настраиваемого класса, производного от [TableEntity](/dotnet/api/microsoft.azure.cosmos.table.tableentity). Чтобы добавить сущность в таблицу, создайте класс, который определяет свойства сущности.

Щелкните проект **CosmosTableSamples** правой кнопкой мыши. Выберите **Добавить**, **Новая папка** и присвойте ей имя **Model**. В папку Model добавьте класс с именем **CustomerEntity.cs** и добавьте следующий код.

:::code language="csharp" source="~/azure-cosmosdb-dotnet-table/CosmosTableSamples/Model/CustomerEntity.cs":::

Этот код определяет класс сущностей, который использует в качестве ключа строки имя клиента, а в качестве ключа раздела — фамилию клиента. Вместе ключ раздела и ключ строки сущности уникальным образом идентифицируют сущность в таблице. Сущности с одинаковым ключом раздела возвращаются быстрее, чем с разными ключами раздела, но использование разных ключей разделов повышает масштабируемость параллельных операций. В таблицах можно хранить только сущности поддерживаемых типов, например производных от класса [TableEntity](/dotnet/api/microsoft.azure.cosmos.table.tableentity). Свойства сущности, которые вы хотите хранить в таблице, должны быть открытыми свойствами типа и поддерживать получение и настройку значений. Кроме того, тип сущности должен предоставлять конструктор без параметров.

## <a name="insert-or-merge-an-entity"></a>Вставка или слияние сущностей

Следующий пример кода создает объект сущности и добавляет его в таблицу. Метод InsertOrMerge из класса [TableOperation](/dotnet/api/microsoft.azure.cosmos.table.tableoperation) используется для вставки или слияния сущностей. Для выполнения этой операции вызывается метод [CloudTable.ExecuteAsync](/dotnet/api/microsoft.azure.cosmos.table.cloudtable.executeasync). 

Щелкните проект **CosmosTableSamples** правой кнопкой мыши. Выберите **Добавить**, **Новый элемент** и добавьте класс с именем **SamplesUtils.cs**. Этот класс содержит весь необходимый код для выполнения операций CRUD для сущностей. 

:::code language="csharp" source="~/azure-cosmosdb-dotnet-table/CosmosTableSamples/SamplesUtils.cs" id="InsertItem":::

## <a name="get-an-entity-from-a-partition"></a>Получение сущности из раздела

Чтобы получить сущность из раздела, можно применить метод Retrieve из класса [TableOperation](/dotnet/api/microsoft.azure.cosmos.table.tableoperation). Следующий пример кода получает ключ раздела, ключ строки, адрес электронной почты и номер телефона из сущности customer. Также этот пример возвращает сведения о количестве единиц запросов, затраченных на запрос сущности. Чтобы запросить данные сущности, добавьте следующий код в файл **SamplesUtils.cs**:

:::code language="csharp" source="~/azure-cosmosdb-dotnet-table/CosmosTableSamples/SamplesUtils.cs" id="QueryData":::

## <a name="delete-an-entity"></a>Удаление сущности

Сущность можно легко удалить после ее получения с использованием того же шаблона, что и для обновления сущностей. Следующий код извлекает и удаляет сущность клиента. Чтобы удалить сущность, добавьте следующий код в файл **SamplesUtils.cs**: 

:::code language="csharp" source="~/azure-cosmosdb-dotnet-table/CosmosTableSamples/SamplesUtils.cs" id="DeleteItem":::

## <a name="execute-the-crud-operations-on-sample-data"></a>Выполнение операций CRUD для тестовых данных

Определив методы для создания таблицы, вставки и слияния сущностей, примените эти методы для тестовых данных. Для этого щелкните проект **CosmosTableSamples** правой кнопкой мыши. Выберите **Добавить**, **Новый элемент**, добавьте класс с именем **BasicSamples.cs** и добавьте следующий код. Этот код создает таблицу и добавляет в нее сущности.

Если вы не хотите удалять эту сущность и таблицу после завершения проекта, закомментируйте методы `await table.DeleteIfExistsAsync()` и `SamplesUtils.DeleteEntityAsync(table, customer)` в приведенном ниже коде. Мы рекомендуем закомментировать эти методы и проверить данные перед удалением таблицы.

:::code language="csharp" source="~/azure-cosmosdb-dotnet-table/CosmosTableSamples/BasicSamples.cs":::

Приведенный выше код создает таблиц с именем, состоящим из префикса demo и созданного идентификатора GUID. Затем он добавляет сущность customer, задав для нее имя и фамилию Walter Harp, а затем изменяет номер телефона для этого пользователя. 

В этом руководстве описано, как создать код, который позволяет выполнять основные операции CRUD с данными, хранящимися в учетной записи API таблиц. Вы также можете выполнять дополнительные операции, например пакетную вставку данных, получение всех данных из раздела, запрос диапазона данных в разделе или получение списка таблиц в учетной записи, имена которых начинаются с указанного префикса. Полный пример [azure-cosmos-table-dotnet-core-getting-started](https://github.com/Azure-Samples/azure-cosmos-table-dotnet-core-getting-started) вы можете скачать из репозитория GitHub. Класс [AdvancedSamples.cs](https://github.com/Azure-Samples/azure-cosmos-table-dotnet-core-getting-started/blob/main/CosmosTableSamples/AdvancedSamples.cs) поддерживает ряд дополнительных операций, которые можно выполнять с данными.  

## <a name="run-the-project"></a>Запуск проекта

В проекте **CosmosTableSamples** откройте класс с именем **Program.cs** и добавьте следующий код для вызова BasicSamples при запуске проекта.

:::code language="csharp" source="~/azure-cosmosdb-dotnet-table/CosmosTableSamples/Program.cs":::

Теперь скомпилируйте решение и запустите проект, нажав клавишу F5. При выполнении проекта вы увидите в командной строке следующие выходные данные:

:::image type="content" source="./media/tutorial-develop-table-standard/output-from-sample.png" alt-text="Выходные данные в командной строке":::

Если возникнет ошибка с сообщением о том, что при запуске проекта файл Settings.json не удалось найти, эту ошибку можно устранить, добавив в настройки проекта следующую XML-запись. Щелкните правой кнопкой мыши CosmosTableSamples, выберите "Редактировать CosmosTableSamples.csproj" и добавьте следующий элемент itemGroup. 

```csharp
  <ItemGroup>
    <None Update="Settings.json">
      <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    </None>
  </ItemGroup>
```
Теперь можно войти на портал Azure и проверить, появились ли данные в таблице. 

:::image type="content" source="./media/tutorial-develop-table-standard/results-in-portal.png" alt-text="Результаты на портале":::

## <a name="next-steps"></a>Дальнейшие действия

Теперь вы можете перейти к следующему руководству, из которого вы узнаете, как перенести данные в учетную запись API таблиц Azure Cosmos DB. 

> [!div class="nextstepaction"]
>[Перенос данных в учетную запись API таблиц в Azure Cosmos DB](../cosmos-db/table-import.md)
