---
title: Использование встроенных команд и функций записных книжек в Azure Cosmos DB C# (предварительная версия)
description: Узнайте, как использовать встроенные команды и функции для выполнения стандартных операций с помощью встроенных записных книжек Azure Cosmos DB C#.
author: deborahc
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 05/19/2020
ms.author: dech
ms.custom: devx-track-python, devx-track-csharp
ms.openlocfilehash: 67fe71d8e2d6ab239989cb30e9bf5a1b4d731037
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "93340487"
---
# <a name="use-built-in-notebook-commands-and-features-in-azure-cosmos-db-c-notebooks-preview"></a>Использование встроенных команд и функций записных книжек в Azure Cosmos DB C# (предварительная версия)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Встроенные записные книжки Jupyter в Azure Cosmos DB позволяют анализировать и визуализировать данные с портала Azure. В этой статье описывается, как использовать встроенные команды и функции записной книжки для выполнения стандартных операций в записных книжках C#.

## <a name="install-a-new-nuget-package"></a>Установка нового пакета NuGet
После включения поддержки записных книжек для учетных записей Azure Cosmos можно открыть новую записную книжку и установить пакет.

В новой ячейке кода вставьте и выполните следующий код, заменив ``PackageToBeInstalled`` на нужный пакет NuGet, а ``optionalVersion`` — на определенную версию пакета (при необходимости). 

```csharp
#r "nuget: PackageToBeInstalled, optionalVersion"
```

В одной ячейке можно установить несколько пакетов NuGet. Пакеты будут доступны для использования из любой записной книжки в рабочей области учетной записи Azure Cosmos. 

В настоящее время рабочая область записных книжек C# не поддерживает рекурсивное разрешение пакетов NuGet. Если пакет NuGet зависит от других пакетов NuGet, которые в настоящее время не установлены, необходимо явно ссылаться на них вместе с родительским пакетом.

> [!TIP]
> Если записной книжке требуется пользовательский пакет, мы рекомендуем добавить ячейку в записную книжку, чтобы установить пакет и сделать его первой ячейкой. Это снижает вероятность конфликтов с другими пакетами, которые Azure Cosmos DB загружает по умолчанию. Также можно легко переустановить пакеты, если [сбросить](#reset-notebooks-workspace) рабочую область, в результате чего все пакеты будут удалены. 

## <a name="use-the-built-in-azure-cosmos-db-net-sdk"></a>Использование встроенного пакета SDK .NET для Azure Cosmos DB
Версия 3 [пакета SDK .NET для Azure Cosmos DB для API SQL](https://github.com/Azure/azure-cosmos-dotnet-v3) установлена и включена в среду записных книжек для учетной записи Azure Cosmos.

Создайте экземпляр ``CosmosClient`` для выполнения любой операции SDK. 

Пример:

```csharp
// Include usings
using System;
using Microsoft.Azure.Cosmos; //namespace for Azure Cosmos DB .NET V3 SDK
using System.Collections;

// Initialize a new instance of CosmosClient using the built-in account endpoint and key parameters
CosmosClient cosmosClient = new CosmosClient(Cosmos.Endpoint, Cosmos.Key);

// Create a new database and container with 400 RU/s
Database database = await cosmosClient.CreateDatabaseIfNotExistsAsync("DatabaseName");
Container container = await database.CreateContainerIfNotExistsAsync("ContainerName", "/partitionKey", 400);
```
Дополнительные сведения см. в разделе [примеров пакета SDK .NET версии 3](https://github.com/Azure/azure-cosmos-dotnet-v3/tree/master/Microsoft.Azure.Cosmos.Samples/Usage). 

> [!IMPORTANT]
> Встроенный пакет SDK .NET для Azure Cosmos DB поддерживается только для учетных записей API SQL (Core). Для других API необходимо [установить драйвер .NET](#install-a-new-nuget-package), соответствующий API. 

## <a name="set-custom-options-using-cosmosclientoptions"></a>Задание настраиваемых параметров с помощью ``CosmosClientOptions``
Для большей гибкости можно задать пользовательское свойство ``CosmosClientOptions`` и передать его в экземпляр ``CosmosClient``. С помощью этого свойства можно выполнять следующие операции:

- задавать имя приложения в суффиксе агента-пользователя, чтобы включать его в каждый запрос;
- задавать предпочтительный регион для использования при выполнении операций со службой;
- задавать пользовательскую политику повтора для обработки запросов с ограничением скорости.

Все поддерживаемые параметры перечислены в справочной статье [API CosmosClientOptions](/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions). Ниже приведен пример, в котором показано, как задать свойство `cosmosClientOptions`:

```csharp
using Microsoft.Azure.Cosmos;

// Configure CosmosClientOptions
var cosmosClientOptions = new CosmosClientOptions
{
    ApplicationName = "ContosoNotebookAppName",
    ApplicationRegion = Regions.RegionName, // By default, this is the region you first created your account in
    MaxRetryAttemptsOnRateLimitedRequests = 9, // By default, this value is 9
};

var client = new CosmosClient(Cosmos.Endpoint, Cosmos.Key, cosmosClientOptions);
```

## <a name="access-the-account-endpoint-and-primary-key-variables"></a>Доступ к переменным конечной точки учетной записи и первичного ключа
Доступ к встроенной конечной точке и ключу учетной записи Azure Cosmos возможен там, где существует записная книжка.

```csharp
var key = Cosmos.Key;
var endpoint = Cosmos.Endpoint;
```

> [!IMPORTANT]
> Переменные ``Cosmos.Key`` и ``Cosmos.Endpoint`` применимы только для API SQL. Для других API-интерфейсов найдите конечную точку и ключ в колонке **Строки подключения** или **Ключи** учетной записи Azure Cosmos.  

## <a name="print-console-output-in-c-code"></a>Вывод выходных данных консоли в коде C#
В коде C# можно использовать синтаксис Display.AsMarkdown() с [интерполяцией строк](/dotnet/csharp/language-reference/tokens/interpolated), чтобы вывести выходные данные консоли, которые будут отображаться при запуске ячейки. 

Пример: 

```csharp
// Print text in the output
Display.AsMarkdown($"Hello world!");

// Print a variable in the output
for (int i = 0; i < 5; i++) {
    Display.AsMarkdown($"Printing out variable: {i}");
}
```
> [!IMPORTANT]
> Синтаксис Console.WriteLine() в настоящее время не поддерживается в записных книжках C#. Используйте Display.AsMarkdown для вывода выходных данных консоли в своей программе. 

## <a name="use-built-in-nteract-data-explorer"></a>Использование встроенного обозревателя данных nteract
Для фильтрации и визуализации коллекции элементов можно использовать встроенный [обозреватель данных nteract](https://blog.nteract.io/designing-the-nteract-data-explorer-f4476d53f897). В ячейке вставьте переменную, которую необходимо визуализировать, в последней строке, которая автоматически отображается в nteract при запуске ячейки.

Так, в примере *GetingStarted_Csharp.ipynb* можно вывести переменную с нашим результатом, ``telemetryEvents``. См. весь пример в разделе [GettingStarted_Csharp.ipynb notebook](https://github.com/Azure-Samples/cosmos-notebooks/blob/master/CSharp_quickstarts/GettingStarted_CSharp.ipynb). 

:::image type="content" source="media/use-notebook-features-and-commands/csharp-query-cell.png" alt-text="Ячейка запроса Csharp":::

:::image type="content" source="media/use-notebook-features-and-commands/csharp-nteract-built-in-chart.png" alt-text="Обозреватель данных nteract":::

## <a name="use-built-in-dictionary-viewer"></a>Использование встроенного средства просмотра словарей
Для просмотра переменной можно использовать встроенное средство просмотра словарей. В ячейке вставьте переменную, которую необходимо визуализировать, в последней строке, которая автоматически отображается при запуске ячейки.

:::image type="content" source="media/use-notebook-features-and-commands/csharp-built-in-dictionary-viewer.png" alt-text="Встроенное средство просмотра словарей":::

## <a name="upload-json-items-to-a-container"></a>Отправка элементов JSON в контейнер
Магическую команду ``%%upload`` можно использовать для передачи данных из JSON-файла в указанный контейнер Azure Cosmos. Используйте для отправки элементов следующую команду:

```csharp
%%upload --databaseName {database_id} --containerName {container_id} --url {url_location_of_file}
```

- Замените ``{database_id}`` и ``{container_id}`` именем базы данных и контейнера в своей учетной записи Azure Cosmos. 
- Замените ``{url_location_of_file}`` расположением JSON-файла. Файл должен представлять собой массив действительных объектов JSON и должен быть доступен через общедоступный Интернет.

Пример:

```csharp
%%upload --database databaseName --container containerName --url 
https://contoso.com/path/to/data.json
```
```
Documents successfully uploaded to ContainerName
Total number of documents imported : 2654
Total time taken : 00:00:38.1228087 hours
Total RUs consumed : 25022.58
```

С помощью выходной статистики можно вычислить эффективный показатель (единиц запросов в секунду) для отправки элементов. Например, если 25 000 единиц запросов было использовано за 38 секунд, то эффективный показатель равен 25 000 единиц запросов / 38 секунд = 658 единиц запросов/с.

## <a name="run-another-notebook-in-current-notebook"></a>Запуск другой записной книжки в данной записной книжке 
Можно использовать магическую команду ``%%run`` для запуска из текущей записной книжки другой записной книжки в своем рабочем пространстве. Используйте следующий синтаксис:

```csharp
%%run ./path/to/{notebookName}.ipynb
```
Замените ``{notebookName}`` именем записной книжки, которую нужно запустить. Записная книжка должна находиться в текущей рабочей области "Мои записные книжки". 

## <a name="reset-notebooks-workspace"></a>Сброс рабочей области записных книжек
Чтобы сбросить рабочую область записных книжек до параметров по умолчанию, выберите **Сбросить рабочую область** на панели команд. При этом будут удалены все установленные пользовательские пакеты и перезапущен сервер Jupyter. Ваши записные книжки, файлы и ресурсы Azure Cosmos затронуты не будут.  

:::image type="content" source="media/use-notebook-features-and-commands/reset-workspace.png" alt-text="Сброс рабочей области записных книжек":::

## <a name="next-steps"></a>Дальнейшие действия

- Сведения о преимуществах записных книжек Jupyter в Azure Cosmos DB см. [здесь](cosmosdb-jupyter-notebooks.md).
- [Пакет SDK .NET для Azure Cosmos DB для API SQL](https://github.com/Azure/azure-cosmos-dotnet-v3)
