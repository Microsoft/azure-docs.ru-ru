---
title: Фильтрация данных с помощью Azure Data Lake Storage ускорение запросов | Документация Майкрософт
description: Используйте ускорение запросов для получения подмножества данных из вашей учетной записи хранения.
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: how-to
ms.date: 01/06/2021
ms.author: normesta
ms.reviewer: jamsbak
ms.custom: devx-track-csharp, devx-track-azurecli
ms.openlocfilehash: a925d3f55395d094c7f19f65de4b72fd20a11a41
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102213680"
---
# <a name="filter-data-by-using-azure-data-lake-storage-query-acceleration"></a>Фильтрация данных с помощью ускорения запросов Azure Data Lake Storage

В этой статье показано, как использовать ускорение запросов для получения подмножества данных из учетной записи хранения. 

Ускорение запросов позволяет приложениям и платформам аналитики значительно оптимизировать обработку данных, получая только те данные, которые необходимы для выполнения определенной операции. Дополнительные сведения см. в разделе [ускорение запросов Azure Data Lake Storage](data-lake-storage-query-acceleration.md).

## <a name="prerequisites"></a>Предварительные требования

- Для доступа к службе хранилища Azure требуется подписка Azure. Если у вас еще нет подписки, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

- Учетная запись хранения **общего назначения версии 2** . см. раздел [Создание учетной записи хранения](../common/storage-account-create.md).

- Выберите вкладку, чтобы просмотреть все предварительные требования, относящиеся к пакету SDK.

  ### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

  Неприменимо

  ### <a name="net"></a>[.NET](#tab/dotnet)

  [Пакет SDK для .NET](https://dotnet.microsoft.com/download) 

  ### <a name="java"></a>[Java](#tab/java)

  - [Пакет разработчиков Java (JDK)](/java/azure/jdk/) версии 8 или более поздней версии.

  - [Apache Maven](https://maven.apache.org/download.cgi) 

    > [!NOTE] 
    > В этой статье предполагается, что вы создали проект Java с помощью Apache Maven. Пример создания проекта с помощью Apache Maven см. в разделе [Настройка](storage-quickstart-blobs-java.md#setting-up).
  
  ### <a name="python"></a>[Python](#tab/python)

  [Python](https://www.python.org/downloads/) 3,8 или более поздней версии.

  ### <a name="nodejs"></a>[Node.js](#tab/nodejs)

  Для использования пакета SDK для Node.js не требуются дополнительные условия.

---

## <a name="enable-query-acceleration"></a>Включить ускорение запросов

Чтобы использовать ускорение запросов, необходимо зарегистрировать функцию ускорения запросов в подписке. Убедившись, что функция зарегистрирована, необходимо зарегистрировать поставщик ресурсов службы хранилища Azure. 

### <a name="step-1-register-the-query-acceleration-feature"></a>Шаг 1. регистрация функции ускорения запросов

Чтобы использовать ускорение запросов, необходимо сначала зарегистрировать функцию ускорения запросов в подписке. 

#### <a name="powershell"></a>[PowerShell](#tab/powershell)

1. Откройте командное окно Windows PowerShell.

1. Войдите в подписку Azure с помощью команды `Connect-AzAccount` и следуйте инструкциям на экране.

   ```powershell
   Connect-AzAccount
   ```

2. Если ваше удостоверение связано с более чем одной подпиской, установите активную подписку.

   ```powershell
   $context = Get-AzSubscription -SubscriptionId <subscription-id>
   Set-AzContext $context
   ```

   Замените значение заполнителя `<subscription-id>` идентификатором своей подписки.

3. Зарегистрируйте функцию ускорения запросов с помощью команды [Register-азпровидерфеатуре](/powershell/module/az.resources/register-azproviderfeature) .

   ```powershell
   Register-AzProviderFeature -ProviderNamespace Microsoft.Storage -FeatureName BlobQuery
   ```

#### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Откройте [Azure Cloud Shell](../../cloud-shell/overview.md) или, если вы [установили](/cli/azure/install-azure-cli) Azure CLI локально, командное консольное приложение (например, Windows PowerShell).

2. Если ваше удостоверение связано с более чем одной подпиской, установите активную подписку на подписку учетной записи хранения.

   ```azurecli-interactive
   az account set --subscription <subscription-id>
   ```

   Замените значение заполнителя `<subscription-id>` идентификатором своей подписки.

3. Зарегистрируйте функцию ускорения запросов с помощью команды [AZ Feature Register](/cli/azure/feature#az-feature-register) .

   ```azurecli
   az feature register --namespace Microsoft.Storage --name BlobQuery
   ```

---

### <a name="step-2-verify-that-the-feature-is-registered"></a>Шаг 2. Проверка регистрации компонента

#### <a name="powershell"></a>[PowerShell](#tab/powershell)

Чтобы убедиться, что регистрация завершена, используйте команду [Get-азпровидерфеатуре](/powershell/module/az.resources/get-azproviderfeature) .

```powershell
Get-AzProviderFeature -ProviderNamespace Microsoft.Storage -FeatureName BlobQuery
```

#### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы убедиться, что регистрация завершена, используйте команду [AZ Feature](/cli/azure/feature#az-feature-show) .

```azurecli
az feature show --namespace Microsoft.Storage --name BlobQuery
```

---

### <a name="step-3-register-the-azure-storage-resource-provider"></a>Шаг 3. Регистрация поставщика ресурсов службы хранилища Azure

После утверждения регистрации необходимо повторно зарегистрировать поставщик ресурсов службы хранилища Azure. 

#### <a name="powershell"></a>[PowerShell](#tab/powershell)

Чтобы зарегистрировать поставщик ресурсов, используйте команду [Register-азресаурцепровидер](/powershell/module/az.resources/register-azresourceprovider) .

```powershell
Register-AzResourceProvider -ProviderNamespace 'Microsoft.Storage'
```

#### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы зарегистрировать поставщик ресурсов, используйте команду [AZ Provider Register](/cli/azure/provider#az-provider-register) .

```azurecli
az provider register --namespace 'Microsoft.Storage'
```

---

## <a name="set-up-your-environment"></a>Настройка среды

### <a name="step-1-install-packages"></a>Шаг 1. Установка пакетов 

#### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Установите модуль AZ версии 4.6.0 или более поздней.

```powershell
Install-Module -Name Az -Repository PSGallery -Force
```

Чтобы обновить старую версию AZ, выполните следующую команду:

```powershell
Update-Module -Name Az
```

#### <a name="net"></a>[.NET](#tab/dotnet)

1. Откройте командную строку и измените каталог ( `cd` ) в папку проекта, например:

   ```console
   cd myProject
   ```

2. Установите `12.5.0-preview.6` версию или более позднюю версию клиентской библиотеки хранилища BLOB-объектов Azure для пакета .NET с помощью `dotnet add package` команды. 

   ```console
   dotnet add package Azure.Storage.Blobs -v 12.8.0
   ```

3. Примеры, представленные в этой статье, анализируют CSV-файл с помощью библиотеки [ксвхелпер](https://www.nuget.org/packages/CsvHelper/) . Чтобы использовать эту библиотеку, используйте следующую команду.

   ```console
   dotnet add package CsvHelper
   ```

#### <a name="java"></a>[Java](#tab/java)

1. Откройте файл *pom.xml* проекта в текстовом редакторе. Добавьте приведенные ниже элементы зависимости в группу зависимостей. 

   ```xml
   <!-- Request static dependencies from Maven -->
   <dependency>
       <groupId>com.azure</groupId>
       <artifactId>azure-core</artifactId>
       <version>1.6.0</version>
   </dependency>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-csv</artifactId>
        <version>1.8</version>
    </dependency>    
    <dependency>
      <groupId>com.azure</groupId>
      <artifactId>azure-storage-blob</artifactId>
      <version>12.8.0-beta.1</version>
    </dependency>
   ```

#### <a name="python"></a>[Python](#tab/python)

Установите клиентскую библиотеку Azure Data Lake Storage для Python с помощью [PIP](https://pypi.org/project/pip/).

```
pip install azure-storage-blob==12.4.0
```

#### <a name="nodejs"></a>[Node.js](#tab/nodejs)

Установите Data Lake клиентскую библиотеку для JavaScript, открыв окно терминала, а затем введя следующую команду.

```javascript
    npm install @azure/storage-blob
    npm install @fast-csv/parse
```

---

### <a name="step-2-add-statements"></a>Шаг 2. Добавление инструкций

#### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Неприменимо

#### <a name="net"></a>[.NET](#tab/dotnet)

Добавьте эти `using` инструкции в начало файла кода.

```csharp
using Azure.Storage.Blobs;
using Azure.Storage.Blobs.Models;
using Azure.Storage.Blobs.Specialized;
```

Ускорение запросов извлекает данные в формате CSV и JSON. Поэтому обязательно добавьте операторы using для любых библиотек синтаксического анализа CSV или JSON, которые вы решили использовать. Примеры, представленные в этой статье, анализируют CSV-файл с помощью библиотеки [ксвхелпер](https://www.nuget.org/packages/CsvHelper/) , доступной в NuGet. Поэтому мы добавили эти `using` инструкции в начало файла кода.

```csharp
using CsvHelper;
using CsvHelper.Configuration;
```

Для компиляции примеров, представленных в этой статье, также необходимо добавить эти `using` инструкции.

```csharp
using System.Threading.Tasks;
using System.IO;
using System.Globalization;
```

#### <a name="java"></a>[Java](#tab/java)

Добавьте эти `import` инструкции в начало файла кода.

```java
import com.azure.storage.blob.*;
import com.azure.storage.blob.options.*;
import com.azure.storage.blob.models.*;
import com.azure.storage.common.*;
import java.io.*;
import java.util.function.Consumer;
import org.apache.commons.csv.*;
```

#### <a name="python"></a>[Python](#tab/python)

Добавьте эти операторы импорта в начало файла кода.

```python
import sys, csv
from azure.storage.blob import BlobServiceClient, ContainerClient, BlobClient, DelimitedTextDialect, BlobQueryError
```

### <a name="nodejs"></a>[Node.js](#tab/nodejs)

Включите `storage-blob` модуль, поместив этот оператор в начало файла кода. 

```javascript
const { BlobServiceClient } = require("@azure/storage-blob");
```

Ускорение запросов извлекает данные в формате CSV и JSON. Поэтому убедитесь, что добавлены инструкции для всех модулей синтаксического анализа CSV или JSON, которые вы решили использовать. В примерах, приведенных в этой статье, выполняется анализ CSV-файла с помощью модуля [fast/CSV](https://www.npmjs.com/package/fast-csv) . Поэтому мы добавим этот оператор в начало файла кода.

```javascript
const csv = require('@fast-csv/parse');
```

---

## <a name="retrieve-data-by-using-a-filter"></a>Получение данных с помощью фильтра

Вы можете использовать SQL для указания предикатов фильтра строк и проекций столбцов в запросе ускорения запроса. Следующий код запрашивает CSV-файл в хранилище и возвращает все строки данных, в которых третий столбец соответствует значению `Hemingway, Ernest` . 

- В SQL-запросе ключевое слово `BlobStorage` используется для обозначения файла, к которому выполняется запрос.

- Ссылки на столбцы указываются в том виде, в `_N` котором находится первый столбец `_1` . Если исходный файл содержит строку заголовка, то можно ссылаться на столбцы по имени, указанному в строке заголовка. 

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell
Function Get-QueryCsv($ctx, $container, $blob, $query, $hasheaders) {
    $tempfile = New-TemporaryFile
    $informat = New-AzStorageBlobQueryConfig -AsCsv -HasHeader:$hasheaders
    Get-AzStorageBlobQueryResult -Context $ctx -Container $container -Blob $blob -InputTextConfiguration $informat -OutputTextConfiguration (New-AzStorageBlobQueryConfig -AsCsv -HasHeader) -ResultFile $tempfile.FullName -QueryString $query -Force
    Get-Content $tempfile.FullName
}

$container = "data"
$blob = "csv/csv-general/seattle-library.csv"
Get-QueryCsv $ctx $container $blob "SELECT * FROM BlobStorage WHERE _3 = 'Hemingway, Ernest, 1899-1961'" $false

```

### <a name="net"></a>[.NET](#tab/dotnet)

Асинхронный метод `BlobQuickQueryClient.QueryAsync` отправляет запрос в API ускорения запросов, а затем отправляет результаты обратно в приложение в виде объекта [потока](/dotnet/api/system.io.stream) .

```cs
static async Task QueryHemingway(BlockBlobClient blob)
{
    string query = @"SELECT * FROM BlobStorage WHERE _3 = 'Hemingway, Ernest, 1899-1961'";
    await DumpQueryCsv(blob, query, false);
}

private static async Task DumpQueryCsv(BlockBlobClient blob, string query, bool headers)
{
    try
    {
        var options = new BlobQueryOptions() {
            InputTextConfiguration = new BlobQueryCsvTextOptions() { HasHeaders = headers },
            OutputTextConfiguration = new BlobQueryCsvTextOptions() { HasHeaders = true },
            ProgressHandler = new Progress<long>((finishedBytes) => Console.Error.WriteLine($"Data read: {finishedBytes}"))
        };
        options.ErrorHandler += (BlobQueryError err) => {
            Console.ForegroundColor = ConsoleColor.Red;
            Console.Error.WriteLine($"Error: {err.Position}:{err.Name}:{err.Description}");
            Console.ResetColor();
        };
        // BlobDownloadInfo exposes a Stream that will make results available when received rather than blocking for the entire response.
        using (var reader = new StreamReader((await blob.QueryAsync(
                query,
                options)).Value.Content))
        {
            using (var parser = new CsvReader(reader, new CsvConfiguration(CultureInfo.CurrentCulture, hasHeaderRecord: true) { HasHeaderRecord = true }))
            {
                while (await parser.ReadAsync())
                {
                    Console.Out.WriteLine(String.Join(" ", parser.Parser.Record));
                }
            }
        }
    }
    catch (Exception ex)
    {
        Console.Error.WriteLine("Exception: " + ex.ToString());
    }
}

```

### <a name="java"></a>[Java](#tab/java)

Метод `BlobQuickQueryClient.openInputStream()` отправляет запрос в API ускорения запросов, а затем передает результаты обратно в приложение как `InputStream` объект, который может быть считан как любой другой объект InputStream.

```java
static void QueryHemingway(BlobClient blobClient) {
    String expression = "SELECT * FROM BlobStorage WHERE _3 = 'Hemingway, Ernest, 1899-1961'";
    DumpQueryCsv(blobClient, expression, true);
}

static void DumpQueryCsv(BlobClient blobClient, String query, Boolean headers) {
    try {
        BlobQuerySerialization input = new BlobQueryDelimitedSerialization()
            .setRecordSeparator('\n')
            .setColumnSeparator(',')
            .setHeadersPresent(headers)
            .setFieldQuote('\0')
            .setEscapeChar('\\');
        BlobQuerySerialization output = new BlobQueryDelimitedSerialization()
            .setRecordSeparator('\n')
            .setColumnSeparator(',')
            .setHeadersPresent(true)
            .setFieldQuote('\0')
            .setEscapeChar('\n');
        Consumer<BlobQueryError> errorConsumer = System.out::println;
        Consumer<BlobQueryProgress> progressConsumer = progress -> System.out.println("total bytes read: " + progress.getBytesScanned());
        BlobQueryOptions queryOptions = new BlobQueryOptions(query)
            .setInputSerialization(input)
            .setOutputSerialization(output)
            .setErrorConsumer(errorConsumer)
            .setProgressConsumer(progressConsumer);            

        /* Open the query input stream. */
        InputStream stream = blobClient.openQueryInputStream(queryOptions).getValue();
        try (BufferedReader reader = new BufferedReader(new InputStreamReader(stream))) {
            /* Read from stream like you normally would. */
            for (CSVRecord record : CSVParser.parse(reader, CSVFormat.EXCEL.withHeader())) {
                System.out.println(record.toString());
            }
        }
    } catch (Exception e) {
        System.err.println("Exception: " + e.toString());
        e.printStackTrace(System.err);
    }
}
```

### <a name="python"></a>[Python](#tab/python)

```python
def query_hemingway(blob: BlobClient):
    query = "SELECT * FROM BlobStorage WHERE _3 = 'Hemingway, Ernest, 1899-1961'"
    dump_query_csv(blob, query, False)

def dump_query_csv(blob: BlobClient, query: str, headers: bool):
    qa_reader = blob.query_blob(query, blob_format=DelimitedTextDialect(has_header=headers), on_error=report_error, encoding='utf-8')
    # records() returns a generator that will stream results as received. It will not block pending all results.
    csv_reader = csv.reader(qa_reader.records())
    for row in csv_reader:
        print("*".join(row))
```

### <a name="nodejs"></a>[Node.js](#tab/nodejs)

В этом примере запрос отправляется в API ускорения запросов, а затем потоки возвращаются обратно. `blob`Объект, переданный в `queryHemingway` вспомогательную функцию, имеет тип [блоккблобклиент](/javascript/api/@azure/storage-blob/blockblobclient). Дополнительные сведения о том, как получить объект [блоккблобклиент](/javascript/api/@azure/storage-blob/blockblobclient) , см. [в разделе Краткое руководство. Управление большими двоичными объектами с помощью пакета SDK для JavaScript версии 12 в Node.js](storage-quickstart-blobs-nodejs.md).

```javascript
async function queryHemingway(blob)
{
    const query = "SELECT * FROM BlobStorage WHERE _3 = 'Hemingway, Ernest, 1899-1961'";
    await dumpQueryCsv(blob, query, false);
}

async function dumpQueryCsv(blob, query, headers)
{
    var response = await blob.query(query, {
        inputTextConfiguration: {
            kind: "csv",
            recordSeparator: '\n',
            hasHeaders: headers
        },
        outputTextConfiguration: {
            kind: "csv",
            recordSeparator: '\n',
            hasHeaders: true
        },
        onProgress: (progress) => console.log(`Data read: ${progress.loadedBytes}`),
        onError: (err) => console.error(`Error: ${err.position}:${err.name}:${err.description}`)});
    return new Promise(
        function (resolve, reject) {
            csv.parseStream(response.readableStreamBody)
                .on('data', row => console.log(row))
                .on('error', error => {
                    console.error(error);
                    reject(error);
                })
                .on('end', rowCount => resolve());
    });
}
```

---

## <a name="retrieve-specific-columns"></a>Получение конкретных столбцов

Результаты можно ограничить подмножеством столбцов. Таким образом вы получаете только столбцы, необходимые для выполнения данного вычисления. Это повышает производительность приложения и сокращает затраты, поскольку по сети передается меньше данных. 

Этот код извлекает только `BibNum` столбец для всех книг в наборе данных. Он также использует сведения из строки заголовка исходного файла для ссылки на столбцы в запросе.

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell
Function Get-QueryCsv($ctx, $container, $blob, $query, $hasheaders) {
    $tempfile = New-TemporaryFile
    $informat = New-AzStorageBlobQueryConfig -AsCsv -HasHeader:$hasheaders
    Get-AzStorageBlobQueryResult -Context $ctx -Container $container -Blob $blob -InputTextConfiguration $informat -OutputTextConfiguration (New-AzStorageBlobQueryConfig -AsCsv -HasHeader) -ResultFile $tempfile.FullName -QueryString $query -Force
    Get-Content $tempfile.FullName
}

$container = "data"
$blob = "csv/csv-general/seattle-library-with-headers.csv"
Get-QueryCsv $ctx $container $blob "SELECT BibNum FROM BlobStorage" $true

```

### <a name="net"></a>[.NET](#tab/dotnet)

```cs
static async Task QueryBibNum(BlockBlobClient blob)
{
    string query = @"SELECT BibNum FROM BlobStorage";
    await DumpQueryCsv(blob, query, true);
}
```

### <a name="java"></a>[Java](#tab/java)

```java
static void QueryBibNum(BlobClient blobClient)
{
    String expression = "SELECT BibNum FROM BlobStorage";
    DumpQueryCsv(blobClient, expression, true);
}
```

### <a name="python"></a>[Python](#tab/python)

```python
def query_bibnum(blob: BlobClient):
    query = "SELECT BibNum FROM BlobStorage"
    dump_query_csv(blob, query, True)
```

### <a name="nodejs"></a>[Node.js](#tab/nodejs)

```javascript
async function queryBibNum(blob)
{
    const query = "SELECT BibNum FROM BlobStorage";
    await dumpQueryCsv(blob, query, true);
}
```

---

Следующий код сочетает фильтрацию строк и проекции столбцов в одном запросе. 

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell
Get-QueryCsv $ctx $container $blob $query $true

Function Get-QueryCsv($ctx, $container, $blob, $query, $hasheaders) {
    $tempfile = New-TemporaryFile
    $informat = New-AzStorageBlobQueryConfig -AsCsv -HasHeader:$hasheaders
    Get-AzStorageBlobQueryResult -Context $ctx -Container $container -Blob $blob -InputTextConfiguration $informat -OutputTextConfiguration (New-AzStorageBlobQueryConfig -AsCsv -HasHeader) -ResultFile $tempfile.FullName -QueryString $query -Force
    Get-Content $tempfile.FullName
}

$container = "data"
$query = "SELECT BibNum, Title, Author, ISBN, Publisher, ItemType 
            FROM BlobStorage 
            WHERE ItemType IN 
                ('acdvd', 'cadvd', 'cadvdnf', 'calndvd', 'ccdvd', 'ccdvdnf', 'jcdvd', 'nadvd', 'nadvdnf', 'nalndvd', 'ncdvd', 'ncdvdnf')"

```

### <a name="net"></a>[.NET](#tab/dotnet)

```cs
static async Task QueryDvds(BlockBlobClient blob)
{
    string query = @"SELECT BibNum, Title, Author, ISBN, Publisher, ItemType 
        FROM BlobStorage 
        WHERE ItemType IN 
            ('acdvd', 'cadvd', 'cadvdnf', 'calndvd', 'ccdvd', 'ccdvdnf', 'jcdvd', 'nadvd', 'nadvdnf', 'nalndvd', 'ncdvd', 'ncdvdnf')";
    await DumpQueryCsv(blob, query, true);
}
```

### <a name="java"></a>[Java](#tab/java)

```java
static void QueryDvds(BlobClient blobClient)
{
    String expression = "SELECT BibNum, Title, Author, ISBN, Publisher, ItemType " +
                        "FROM BlobStorage " +
                        "WHERE ItemType IN " +
                        "   ('acdvd', 'cadvd', 'cadvdnf', 'calndvd', 'ccdvd', 'ccdvdnf', 'jcdvd', 'nadvd', 'nadvdnf', 'nalndvd', 'ncdvd', 'ncdvdnf')";
    DumpQueryCsv(blobClient, expression, true);
}
```

### <a name="python"></a>[Python](#tab/python)

```python
def query_dvds(blob: BlobClient):
    query = "SELECT BibNum, Title, Author, ISBN, Publisher, ItemType "\
        "FROM BlobStorage "\
        "WHERE ItemType IN "\
        "   ('acdvd', 'cadvd', 'cadvdnf', 'calndvd', 'ccdvd', 'ccdvdnf', 'jcdvd', 'nadvd', 'nadvdnf', 'nalndvd', 'ncdvd', 'ncdvdnf')"
    dump_query_csv(blob, query, True)
```

### <a name="nodejs"></a>[Node.js](#tab/nodejs)

```javascript
async function queryDvds(blob)
{
    const query = "SELECT BibNum, Title, Author, ISBN, Publisher, ItemType " +
                  "FROM BlobStorage " +
                  "WHERE ItemType IN " + 
                  " ('acdvd', 'cadvd', 'cadvdnf', 'calndvd', 'ccdvd', 'ccdvdnf', 'jcdvd', 'nadvd', 'nadvdnf', 'nalndvd', 'ncdvd', 'ncdvdnf')";
    await dumpQueryCsv(blob, query, true);
}
```

---

## <a name="next-steps"></a>Дальнейшие действия

- [Ускорение запросов Azure Data Lake Storage](data-lake-storage-query-acceleration.md)
- [Справочник по языку SQL для ускорения запросов](query-acceleration-sql-reference.md)
