---
title: Управление Azure Data Lake Analytics с помощью пакета SDK Azure для .NET
description: В этой статье описывается, как использовать пакет SDK для Azure .NET для написания приложений, управляющих заданиями, источниками данных и пользователями Data Lake Analytics.
ms.reviewer: jasonh
ms.service: data-lake-analytics
ms.topic: how-to
ms.date: 06/18/2017
ms.custom: devx-track-csharp
ms.openlocfilehash: a9ece3fe54ca8596f610439f8a0dd228ae1ef907
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92220913"
---
# <a name="manage-azure-data-lake-analytics-a-net-app"></a>Управление Azure Data Lake Analytics с помощью приложения .NET

[!INCLUDE [manage-selector](../../includes/data-lake-analytics-selector-manage.md)]

Из этой статьи вы узнаете, как управлять учетными записями, источниками данных, пользователями и заданиями Azure Data Lake Analytics, используя приложение, написанное с помощью пакета SDK Azure для .NET. 

## <a name="prerequisites"></a>Предварительные требования

* **Visual Studio 2015, Visual Studio 2013 с обновлением 4 или Visual Studio 2012 с установленным Visual C++**
* **Microsoft Azure SDK для .NET (версии 2.5 или выше)**.  Вы можете установить его с помощью [установщика веб-платформы](https://www.microsoft.com/web/downloads/platform.aspx).
* **Необходимые пакеты Nuget**

### <a name="install-nuget-packages"></a>Установка пакетов Nuget

|Пакет|Версия|
|-------|-------|
|[Microsoft.Rest.ClientRuntime.Azure.Authentication](https://www.nuget.org/packages/Microsoft.Rest.ClientRuntime.Azure.Authentication)| 2.3.1|
|[Microsoft.Azure.Management.DataLake.Analytics](https://www.nuget.org/packages/Microsoft.Azure.Management.DataLake.Analytics)|3.0.0|
|[Microsoft.Azure.Management.DataLake.Store](https://www.nuget.org/packages/Microsoft.Azure.Management.DataLake.Store)|2.2.0|
|[Microsoft.Azure.Management.ResourceManager](https://www.nuget.org/packages/Microsoft.Azure.Management.ResourceManager)|1.6.0-preview|
|[Microsoft.Azure.Graph.RBAC](https://www.nuget.org/packages/Microsoft.Azure.Management.ResourceManager)|3.4.0-preview|

Эти пакеты можно установить в командной строке NuGet с помощью следующих команд:

```powershell
Install-Package -Id Microsoft.Rest.ClientRuntime.Azure.Authentication  -Version 2.3.1
Install-Package -Id Microsoft.Azure.Management.DataLake.Analytics  -Version 3.0.0
Install-Package -Id Microsoft.Azure.Management.DataLake.Store  -Version 2.2.0
Install-Package -Id Microsoft.Azure.Management.ResourceManager  -Version 1.6.0-preview
Install-Package -Id Microsoft.Azure.Graph.RBAC -Version 3.4.0-preview
```

## <a name="common-variables"></a>Общие переменные

```csharp
string subid = "<Subscription ID>"; // Subscription ID (a GUID)
string tenantid = "<Tenant ID>"; // AAD tenant ID or domain. For example, "contoso.onmicrosoft.com"
string rg == "<value>"; // Resource  group name
string clientid = "1950a258-227b-4e31-a9cf-717495945fc2"; // Sample client ID (this will work, but you should pick your own)
```

## <a name="authentication"></a>Аутентификация

Существует несколько способов входа в Azure Data Lake Analytics. В приведенном ниже фрагменте кода показан пример проверки подлинности посредством интерактивной проверки подлинности пользователей с помощью всплывающего окна.

``` csharp
using System;
using System.IO;
using System.Threading;
using System.Security.Cryptography.X509Certificates;

using Microsoft.Rest;
using Microsoft.Rest.Azure.Authentication;
using Microsoft.Azure.Management.DataLake.Analytics;
using Microsoft.Azure.Management.DataLake.Analytics.Models;
using Microsoft.Azure.Management.DataLake.Store;
using Microsoft.Azure.Management.DataLake.Store.Models;
using Microsoft.IdentityModel.Clients.ActiveDirectory;
using Microsoft.Azure.Graph.RBAC;

public static Program
{
   public static string TENANT = "microsoft.onmicrosoft.com";
   public static string CLIENTID = "1950a258-227b-4e31-a9cf-717495945fc2";
   public static System.Uri ARM_TOKEN_AUDIENCE = new System.Uri( @"https://management.core.windows.net/");
   public static System.Uri ADL_TOKEN_AUDIENCE = new System.Uri( @"https://datalake.azure.net/" );
   public static System.Uri GRAPH_TOKEN_AUDIENCE = new System.Uri( @"https://graph.windows.net/" );

   static void Main(string[] args)
   {
      string MY_DOCUMENTS= System.Environment.GetFolderPath( System.Environment.SpecialFolder.MyDocuments);
      string TOKEN_CACHE_PATH = System.IO.Path.Combine(MY_DOCUMENTS, "my.tokencache");

      var tokenCache = GetTokenCache(TOKEN_CACHE_PATH);
      var armCreds = GetCreds_User_Popup(TENANT, ARM_TOKEN_AUDIENCE, CLIENTID, tokenCache);
      var adlCreds = GetCreds_User_Popup(TENANT, ADL_TOKEN_AUDIENCE, CLIENTID, tokenCache);
      var graphCreds = GetCreds_User_Popup(TENANT, GRAPH_TOKEN_AUDIENCE, CLIENTID, tokenCache);
   }
}
```

Исходный код **GetCreds_User_Popup** и код для других вариантов проверки подлинности можно найти на странице [Способы проверки подлинности .NET в Data Lake Analytics](https://github.com/Azure-Samples/data-lake-analytics-dotnet-auth-options).


## <a name="create-the-client-management-objects"></a>Создание объектов управления клиентами

``` csharp
var resourceManagementClient = new ResourceManagementClient(armCreds) { SubscriptionId = subid };

var adlaAccountClient = new DataLakeAnalyticsAccountManagementClient(armCreds);
adlaAccountClient.SubscriptionId = subid;

var adlsAccountClient = new DataLakeStoreAccountManagementClient(armCreds);
adlsAccountClient.SubscriptionId = subid;

var adlaCatalogClient = new DataLakeAnalyticsCatalogManagementClient(adlCreds);
var adlaJobClient = new DataLakeAnalyticsJobManagementClient(adlCreds);

var adlsFileSystemClient = new DataLakeStoreFileSystemManagementClient(adlCreds);

var  graphClient = new GraphRbacManagementClient(graphCreds);
graphClient.TenantID = domain;

```

## <a name="manage-accounts"></a>Управление учетными записями

### <a name="create-an-azure-resource-group"></a>Создание группы ресурсов Azure

Создайте группу ресурсов Azure, если она еще не создана, чтобы создать компоненты Data Lake Analytics. Потребуются учетные данные для проверки подлинности, идентификатор подписки и сведения о расположении. Ниже приведен код, создающий группу ресурсов.

``` csharp
var resourceGroup = new ResourceGroup { Location = location };
resourceManagementClient.ResourceGroups.CreateOrUpdate(groupName, rg);
```

Дополнительные сведения см. в разделе "Группы ресурсов Azure и Data Lake Analytics".

### <a name="create-a-data-lake-store-account"></a>Создание учетной записи хранения озера данных

Для каждой учетной записи ADLA требуется учетная запись ADLS. Если у вас ее еще нет, вы можете создать ее с помощью следующего кода:

``` csharp
var new_adls_params = new DataLakeStoreAccount(location: _location);
adlsAccountClient.Account.Create(rg, adls, new_adls_params);
```

### <a name="create-a-data-lake-analytics-account"></a>Создание учетной записи аналитики озера данных

Приведенный ниже код создает учетную запись ADLS.

``` csharp
var new_adla_params = new DataLakeAnalyticsAccount()
{
   DefaultDataLakeStoreAccount = adls,
   Location = location
};

adlaClient.Account.Create(rg, adla, new_adla_params);
```

### <a name="list-data-lake-store-accounts"></a>Получение списка учетных записей Data Lake Store

``` csharp
var adlsAccounts = adlsAccountClient.Account.List().ToList();
foreach (var adls in adlsAccounts)
{
   Console.WriteLine($"ADLS: {0}", adls.Name);
}
```

### <a name="list-data-lake-analytics-accounts"></a>Список учетных записей аналитики озера данных

``` csharp
var adlaAccounts = adlaClient.Account.List().ToList();

for (var adla in AdlaAccounts)
{
   Console.WriteLine($"ADLA: {0}, adla.Name");
}
```

### <a name="checking-if-an-account-exists"></a>Проверка существования учетной записи

``` csharp
bool exists = adlaClient.Account.Exists(rg, adla));
```

### <a name="get-information-about-an-account"></a>Получение сведений об учетной записи

``` csharp
bool exists = adlaClient.Account.Exists(rg, adla));
if (exists)
{
   var adla_accnt = adlaClient.Account.Get(rg, adla);
}
```

### <a name="delete-an-account"></a>Удаление учетной записи

``` csharp
if (adlaClient.Account.Exists(rg, adla))
{
   adlaClient.Account.Delete(rg, adla);
}
```

### <a name="get-the-default-data-lake-store-account"></a>Получение учетной записи Data Lake Store по умолчанию

Для каждой учетной записи Data Lake Analytics необходима учетная запись Data Lake Store по умолчанию. Этот код позволяет определить учетную запись хранения по умолчанию для учетной записи Analytics.

``` csharp
if (adlaClient.Account.Exists(rg, adla))
{
  var adla_accnt = adlaClient.Account.Get(rg, adla);
  string def_adls_account = adla_accnt.DefaultDataLakeStoreAccount;
}
```

## <a name="manage-data-sources"></a>Управление источниками данных

Аналитика озера данных в настоящее время поддерживает следующие источники данных:

* [Azure Data Lake Storage](../data-lake-store/data-lake-store-overview.md)
* [Учетная запись хранения Azure](../storage/common/storage-introduction.md)

### <a name="link-to-an-azure-storage-account"></a>Связывание с учетной записью хранения Azure

Можно создать ссылки на учетные записи хранения Azure.

``` csharp
string storage_key = "xxxxxxxxxxxxxxxxxxxx";
string storage_account = "mystorageaccount";
var addParams = new AddStorageAccountParameters(storage_key);            
adlaClient.StorageAccounts.Add(rg, adla, storage_account, addParams);
```

### <a name="list-azure-storage-data-sources"></a>Получение списка источников данных службы хранилища Azure

``` csharp
var stg_accounts = adlaAccountClient.StorageAccounts.ListByAccount(rg, adla);

if (stg_accounts != null)
{
  foreach (var stg_account in stg_accounts)
  {
      Console.WriteLine($"Storage account: {0}", stg_account.Name);
  }
}
```

### <a name="list-data-lake-store-data-sources"></a>Получение списка источников данных Data Lake

``` csharp
var adls_accounts = adlsClient.Account.List();

if (adls_accounts != null)
{
  foreach (var adls_accnt in adls_accounts)
  {
      Console.WriteLine($"ADLS account: {0}", adls_accnt.Name);
  }
}
```

### <a name="upload-and-download-folders-and-files"></a>Отправка и загрузка папок и файлов

Объект управления клиентами файловой системы Data Lake Store можно использовать для отправки и загрузки отдельных файлов или папок из Azure на локальный компьютер с помощью следующих методов:

- UploadFolder
- UploadFile
- DownloadFolder
- DownloadFile

Первый параметр для этих методов переставляет собой имя учетной записи Data Lake Store. За ним следуют параметры для исходного пути и целевого пути.

В следующем примере показано, как загрузить папку в Data Lake Store.

``` csharp
adlsFileSystemClient.FileSystem.DownloadFolder(adls, sourcePath, destinationPath);
```

### <a name="create-a-file-in-a-data-lake-store-account"></a>Создание файла в учетной записи Data Lake Store

``` csharp
using (var memstream = new MemoryStream())
{
   using (var sw = new StreamWriter(memstream, UTF8Encoding.UTF8))
   {
      sw.WriteLine("Hello World");
      sw.Flush();
      
      memstream.Position = 0;

      adlsFileSystemClient.FileSystem.Create(adls, "/Samples/Output/randombytes.csv", memstream);
   }
}
```

### <a name="verify-azure-storage-account-paths"></a>Проверка путей к учетной записи хранения Azure

Следующий код проверяет наличие учетной записи хранения Azure (storageAccntName) в учетной записи Data Lake Analytics (analyticsAccountName), а также наличие контейнера (containerName) в учетной записи хранения Azure.

``` csharp
string storage_account = "mystorageaccount";
string storage_container = "mycontainer";
bool accountExists = adlaClient.Account.StorageAccountExists(rg, adla, storage_account));
bool containerExists = adlaClient.Account.StorageContainerExists(rg, adla, storage_account, storage_container));
```

## <a name="manage-catalog-and-jobs"></a>Управление каталогами и заданиями

Объект DataLakeAnalyticsCatalogManagementClient предоставляет методы для управления базой данных SQL, предоставляемой для каждой учетной записи Azure Data Lake Analytics. Объект DataLakeAnalyticsJobManagementClient предоставляет методы для отправки заданий, выполняемых в базе данных с помощью сценариев U-SQL, и управления ими.

### <a name="list-databases-and-schemas"></a>Получение списка баз данных и схем

Самые распространенные компоненты, список которых можно получить — это базы данных и их схемы. Следующий код получает набор баз данных и перечисляет схемы для каждой из них.

``` csharp
var databases = adlaCatalogClient.Catalog.ListDatabases(adla);
foreach (var db in databases)
{
  Console.WriteLine($"Database: {db.Name}");
  Console.WriteLine(" - Schemas:");
  var schemas = adlaCatalogClient.Catalog.ListSchemas(adla, db.Name);
  foreach (var schm in schemas)
  {
      Console.WriteLine($"\t{schm.Name}");
  }
}
```

### <a name="list-table-columns"></a>Получение списка столбцов таблицы

Ниже приведен код, получающий доступ к базе данных с помощью клиента управления каталогами Data Lake Analytics для получения списка столбцов в указанной таблице.

```csharp
var tbl = adlaCatalogClient.Catalog.GetTable(adla, "master", "dbo", "MyTableName");
IEnumerable<USqlTableColumn> columns = tbl.ColumnList;

foreach (USqlTableColumn utc in columns)
{
  Console.WriteLine($"\t{utc.Name}");
}
```

### <a name="submit-a-u-sql-job"></a>Отправка задания U-SQL

В следующем примере кода показано, как отправить задание с помощью клиента управления заданиями Data Lake Analytics.

``` csharp
string scriptPath = "/Samples/Scripts/SearchResults_Wikipedia_Script.txt";
Stream scriptStrm = adlsFileSystemClient.FileSystem.Open(_adlsAccountName, scriptPath);
string scriptTxt = string.Empty;
using (StreamReader sr = new StreamReader(scriptStrm))
{
    scriptTxt = sr.ReadToEnd();
}

var jobName = "SR_Wikipedia";
var jobId = Guid.NewGuid();
var properties = new USqlJobProperties(scriptTxt);
var parameters = new JobInformation(jobName, JobType.USql, properties, priority: 1, degreeOfParallelism: 1, jobId: jobId);
var jobInfo = adlaJobClient.Job.Create(adla, jobId, parameters);
Console.WriteLine($"Job {jobName} submitted.");
```

### <a name="list-failed-jobs"></a>Получение списка невыполненных заданий

Ниже приведен код, получающий список сведений о невыполненных заданиях.

```csharp
var odq = new ODataQuery<JobInformation> { Filter = "result eq 'Failed'" };
var jobs = adlaJobClient.Job.List(adla, odq);
foreach (var j in jobs)
{
   Console.WriteLine($"{j.Name}\t{j.JobId}\t{j.Type}\t{j.StartTime}\t{j.EndTime}");
}
```

### <a name="list-pipelines"></a>Получение списка конвейеров

Приведенный ниже код выводит сведения о каждом конвейере заданий, переданном в учетную запись.

``` csharp
var pipelines = adlaJobClient.Pipeline.List(adla);
foreach (var p in pipelines)
{
   Console.WriteLine($"Pipeline: {p.Name}\t{p.PipelineId}\t{p.LastSubmitTime}");
}
```

### <a name="list-recurrences"></a>Получение списка повторений

Приведенный ниже код выводит сведения о каждом повторении задания, переданном в учетную запись.

``` csharp
var recurrences = adlaJobClient.Recurrence.List(adla);
foreach (var r in recurrences)
{
   Console.WriteLine($"Recurrence: {r.Name}\t{r.RecurrenceId}\t{r.LastSubmitTime}");
}
```

## <a name="common-graph-scenarios"></a>Общие сценарии, связанные с графами

### <a name="look-up-user-in-the-aad-directory"></a>Поиск пользователя в каталоге AAD

``` csharp
var userinfo = graphClient.Users.Get( "bill@contoso.com" );
```

### <a name="get-the-objectid-of-a-user-in-the-aad-directory"></a>Получение идентификатора ObjectId пользователя в каталоге AAD

``` csharp
var userinfo = graphClient.Users.Get( "bill@contoso.com" );
Console.WriteLine( userinfo.ObjectId )
```

## <a name="manage-compute-policies"></a>Управление политиками вычислений

Объект DataLakeAnalyticsAccountManagementClient предоставляет методы для управления политиками вычислений для учетной записи Data Lake Analytics.

### <a name="list-compute-policies"></a>Вывод списка политик вычислений

Следующий код извлекает список политик вычислений для учетной записи Data Lake Analytics.

``` csharp
var policies = adlaAccountClient.ComputePolicies.ListByAccount(rg, adla);
foreach (var p in policies)
{
   Console.WriteLine($"Name: {p.Name}\tType: {p.ObjectType}\tMax AUs / job: {p.MaxDegreeOfParallelismPerJob}\tMin priority / job: {p.MinPriorityPerJob}");
}
```

### <a name="create-a-new-compute-policy"></a>Создание новой политики вычислений

Следующий код создает новую политику вычислений для учетной записи Data Lake Analytics, задавая максимальное количество AU, доступных для указанного пользователя, равным 50 и минимальный приоритет задания равным 250.

``` csharp
var userAadObjectId = "3b097601-4912-4d41-b9d2-78672fc2acde";
var newPolicyParams = new ComputePolicyCreateOrUpdateParameters(userAadObjectId, "User", 50, 250);
adlaAccountClient.ComputePolicies.CreateOrUpdate(rg, adla, "GaryMcDaniel", newPolicyParams);
```

## <a name="next-steps"></a>Дальнейшие действия

* [Обзор аналитики озера данных Microsoft Azure](data-lake-analytics-overview.md)
* [Управление аналитикой озера данных Azure с помощью портала Azure](data-lake-analytics-manage-use-portal.md)
* [Устранение неполадок с заданиями аналитики озера данных Azure с помощью портала Azure](data-lake-analytics-monitor-and-troubleshoot-jobs-tutorial.md)