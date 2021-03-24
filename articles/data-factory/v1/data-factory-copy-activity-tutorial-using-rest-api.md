---
title: 'Руководство по Создание конвейера Фабрики данных Azure с помощью REST API '
description: В этом руководстве вы будете использовать REST API, чтобы создать конвейер Фабрики данных Azure с действием копирования для копирования данных из Хранилища BLOB-объектов Azure в Базу данных SQL Azure.
author: linda33wj
ms.service: data-factory
ms.topic: tutorial
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 7488834252dcd4e231c2d91a1435838befe7b1d1
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100377029"
---
# <a name="tutorial-use-rest-api-to-create-an-azure-data-factory-pipeline-to-copy-data"></a>Руководство по Создание конвейера Фабрики данных Azure для копирования данных с помощью REST API 
> [!div class="op_single_selector"]
> * [Обзор и предварительные требования](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [Мастер копирования](data-factory-copy-data-wizard-tutorial.md)
> * [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md)
> * [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md)
> * [Шаблон Azure Resource Manager](data-factory-copy-activity-tutorial-using-azure-resource-manager-template.md)
> * [REST API](data-factory-copy-activity-tutorial-using-rest-api.md)
> * [API для .NET](data-factory-copy-activity-tutorial-using-dotnet-api.md)
> 
> 

> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию Фабрики данных, ознакомьтесь с руководством по [применению действия копирования](../quickstart-create-data-factory-rest-api.md). 

В этой статье показано, как создать фабрику данных c конвейером, который копирует данные из Хранилища BLOB-объектов Azure в Базу данных SQL Azure, с помощью REST API. Если вы еще не работали с фабрикой данных Azure, перед выполнением действий, описанных в этом руководстве, ознакомьтесь со статьей [Введение в фабрику данных Azure](data-factory-introduction.md).   

В этом руководстве описано, как создать конвейер с одним действием — действием копирования. Действие копирования копирует данные из поддерживаемого хранилища данных в поддерживаемое хранилище данных-приемник. Список хранилищ данных, которые поддерживаются в качестве источников и приемников, см. в разделе [Поддерживаемые хранилища данных и форматы](data-factory-data-movement-activities.md#supported-data-stores-and-formats). Это действие выполняется с помощью глобально доступной службы, обеспечивающей безопасное, надежное и масштабируемое копирование данных между разными хранилищами. Дополнительные сведения о действии копирования см. в статье [Перемещение данных с помощью действия копирования](data-factory-data-movement-activities.md).

Конвейер может содержать сразу несколько действий. Два действия можно объединить в цепочку (выполнить одно действие вслед за другим), настроив выходной набор данных одного действия как входной набор данных другого действия. Дополнительные сведения см. в разделе [Несколько действий в конвейере](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline).

> [!NOTE]
> В этой статье рассматриваются не все REST API фабрики данных. Полную документацию по командлетам фабрики данных см. в [справочнике по REST API фабрики данных](/rest/api/datafactory/).
>  
> В этом руководстве конвейер данных копирует данные из исходного хранилища данных в целевое. Инструкции по преобразованию данных с помощью Фабрики данных Azure см. в [руководстве по созданию конвейера для преобразования данных с использованием кластера Hadoop](data-factory-build-your-first-pipeline.md).

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

* Прочтите [обзор руководства](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) и выполните **предварительные требования** .
* Установите на компьютер программу [curl](https://curl.haxx.se/dlwiz/) . Она будет использоваться с командами REST для создания фабрики данных. 
* Следуя инструкциям в [этой статье](../../active-directory/develop/howto-create-service-principal-portal.md) , выполните следующее: 
  1. Создайте веб-приложение с именем **ADFCopyTutorialApp** в Azure Active Directory.
  2. Получите **идентификатор клиента** и **секретный ключ**. 
  3. Получите значение для **tenant_id**. 
  4. Назначьте приложение **ADFCopyTutorialApp** роли **участника фабрики данных**.  
* Установите [Azure PowerShell](/powershell/azure/).  
* Откройте **PowerShell** и выполните описанные ниже действия. Не закрывайте Azure PowerShell, пока выполняются описанные в учебнике инструкции. Если закрыть и снова открыть это окно, то придется вновь выполнять эти команды.
  
  1. Выполните следующую команду и введите имя пользователя и пароль, которые используются для входа на портал Azure.
    
     ```PowerShell 
     Connect-AzAccount
     ```   
  2. Чтобы просмотреть все подписки для этой учетной записи, выполните следующую команду:

     ```PowerShell     
     Get-AzSubscription
     ``` 
  3. Выполните следующую команду, чтобы выбрать подписку, с которой вы собираетесь работать. Замените **&lt;NameOfAzureSubscription**&gt; именем своей подписки Azure. 
     
     ```PowerShell
     Get-AzSubscription -SubscriptionName <NameOfAzureSubscription> | Set-AzContext
     ```
  4. Создайте группу ресурсов Azure с именем **ADFTutorialResourceGroup** , выполнив следующую команду в PowerShell:  

     ```PowerShell     
      New-AzResourceGroup -Name ADFTutorialResourceGroup  -Location "West US"
     ```
     
      Если группа ресурсов уже есть, укажите, требуется или не требуется ее обновить (Y или N соответственно). 
     
      Некоторые действия, описанные в этом учебнике, предполагают, что вы используете группу ресурсов с именем ADFTutorialResourceGroup. Если вы используете другую группу ресурсов, укажите ее имя вместо ADFTutorialResourceGroup.

## <a name="create-json-definitions"></a>Создание определений JSON
В папке, где находится файл curl.exe, создайте следующие JSON-файлы. 

### <a name="datafactoryjson"></a>datafactory.json
> [!IMPORTANT]
> Имя должно быть глобально уникальным, поэтому может потребоваться добавить к ADFCopyTutorialDF префикс или суффикс. 
> 
> 

```JSON
{  
    "name": "ADFCopyTutorialDF",  
    "location": "WestUS"
}  
```

### <a name="azurestoragelinkedservicejson"></a>azurestoragelinkedservice.json
> [!IMPORTANT]
> Замените значения **accountname** и **accountkey** на имя вашей учетной записи хранения Azure и ее ключ. Сведения о том, как получить ключ доступа к хранилищу, см. в статье [Управление ключами доступа к хранилищу](../../storage/common/storage-account-keys-manage.md).

```JSON
{
    "name": "AzureStorageLinkedService",
    "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
        }
    }
}
```

Дополнительные сведения о свойствах JSON см. в разделе [Связанная служба хранилища Azure](data-factory-azure-blob-connector.md#azure-storage-linked-service).

### <a name="azuresqllinkedservicejson"></a>azuresqllinkedservice.json
> [!IMPORTANT]
> Замените **servername**, **databasename**, **username** и **password** на имя сервера, имя базы данных SQL, имя учетной записи пользователя и ее пароль.  
> 
>

```JSON
{
    "name": "AzureSqlLinkedService",
    "properties": {
        "type": "AzureSqlDatabase",
        "description": "",
        "typeProperties": {
            "connectionString": "Data Source=tcp:<servername>.database.windows.net,1433;Initial Catalog=<databasename>;User ID=<username>;Password=<password>;Integrated Security=False;Encrypt=True;Connect Timeout=30"
        }
    }
}
```

Дополнительные сведения о свойствах JSON см. в разделе [Свойства связанной службы](data-factory-azure-sql-connector.md#linked-service-properties).

### <a name="inputdatasetjson"></a>inputdataset.json

```JSON
{
  "name": "AzureBlobInput",
  "properties": {
    "structure": [
      {
        "name": "FirstName",
        "type": "String"
      },
      {
        "name": "LastName",
        "type": "String"
      }
    ],
    "type": "AzureBlob",
    "linkedServiceName": "AzureStorageLinkedService",
    "typeProperties": {
      "folderPath": "adftutorial/",
      "fileName": "emp.txt",
      "format": {
        "type": "TextFormat",
        "columnDelimiter": ","
      }
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```

В следующей таблице приведены описания свойств JSON, используемых в этом фрагменте кода.

| Свойство | Описание |
|:--- |:--- |
| type | Для свойства типа задано значение **AzureBlob**, так как данные хранятся в хранилище BLOB-объектов Azure. |
| linkedServiceName | Ссылается на созданную ранее службу **AzureStorageLinkedService**. |
| folderPath | Определяет **контейнер** больших двоичных объектов и **папку**, которая содержит входные большие двоичные объекты. В этом руководстве adftutorial — это контейнер больших двоичных объектов, а созданная папка является корневой. | 
| fileName | Это необязательное свойство. Если это свойство не указано, выбираются все файлы из папки folderPath. В этом руководстве для свойства fileName указывается значение **emp.txt**, чтобы обрабатывался только этот файл. |
| format -> type |Входной файл имеет текстовый формат, поэтому укажите значение **TextFormat**. |
| columnDelimiter | Столбцы во входном файле разделяются **запятыми (`,`)** . |
| frequency/interval | Для свойства frequency задано значение **Hour**, а для свойства interval — значение **1**. Это означает, что срезы входных данных будут создаваться **каждый час**. Иными словами, служба фабрики данных будет искать входные данные в корневой папке указанного контейнера BLOB-объектов (**adftutorial**) каждый час. Поиск данных осуществляется в пределах времени начала и времени окончания для конвейера, но не перед этим периодом или после него.  |
| external | Это свойство имеет значение **true**, если этот конвейер не создает данные. В этом руководстве входные данные находятся в файле emp.txt, который не создается этим конвейером, поэтому мы присвоим этому свойству значение true. |

Дополнительные сведения об этих свойствах JSON см. в [этом разделе](data-factory-azure-blob-connector.md#dataset-properties).

### <a name="outputdatasetjson"></a>outputdataset.json

```JSON
{
  "name": "AzureSqlOutput",
  "properties": {
    "structure": [
      {
        "name": "FirstName",
        "type": "String"
      },
      {
        "name": "LastName",
        "type": "String"
      }
    ],
    "type": "AzureSqlTable",
    "linkedServiceName": "AzureSqlLinkedService",
    "typeProperties": {
      "tableName": "emp"
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```
В следующей таблице приведены описания свойств JSON, используемых в этом фрагменте кода.

| Свойство | Описание |
|:--- |:--- |
| type | Свойство type имеет значение **AzureSqlTable**, так как данные копируются в таблицу в Базе данных SQL Azure. |
| linkedServiceName | Ссылается на созданную ранее службу **AzureSqlLinkedService**. |
| tableName | Указывает **таблицу**, в которую копируются данные. | 
| frequency/interval | Для свойства frequency задано значение **Hour**, а для interval — **1**. Это означает, что срезы выходных данных создаются **каждый час** в пределах времени начала и времени окончания для конвейера, но не перед этим периодом или после него.  |

В таблице emp в базе данных есть три столбца: **ID**, **FirstName** и **LastName**. ID — это столбец для идентификаторов, поэтому здесь вам нужно указать только значения **FirstName** и **LastName**.

Дополнительные сведения об этих свойствах JSON см. в [этом разделе](data-factory-azure-sql-connector.md#dataset-properties).

### <a name="pipelinejson"></a>pipeline.json

```JSON
{
  "name": "ADFTutorialPipeline",
  "properties": {
    "description": "Copy data from a blob to Azure SQL table",
    "activities": [
      {
        "name": "CopyFromBlobToSQL",
        "description": "Push Regional Effectiveness Campaign data to Azure SQL Database",
        "type": "Copy",
        "inputs": [
          {
            "name": "AzureBlobInput"
          }
        ],
        "outputs": [
          {
            "name": "AzureSqlOutput"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "BlobSource"
          },
          "sink": {
            "type": "SqlSink",
            "writeBatchSize": 10000,
            "writeBatchTimeout": "60:00:00"
          }
        },
        "Policy": {
          "concurrency": 1,
          "executionPriorityOrder": "NewestFirst",
          "retry": 0,
          "timeout": "01:00:00"
        }
      }
    ],
    "start": "2017-05-11T00:00:00Z",
    "end": "2017-05-12T00:00:00Z"
  }
}
```

Обратите внимание на следующие моменты.

- В разделе действий доступно только одно действие, параметр **type** которого имеет значение **Copy**. Дополнительные сведения о действии копирования см. в статье [Перемещение данных с помощью действия копирования](data-factory-data-movement-activities.md). В решениях фабрики данных можно также использовать [действия преобразования данных](data-factory-data-transformation-activities.md).
- Для этого действия параметру input присвоено значение **AzureBlobInput**, а параметру output — значение **AzureSqlOutput**. 
- В разделе **typeProperties** в качестве типа источника указано **BlobSource**, а в качестве типа приемника — **SqlSink**. Список хранилищ данных, поддерживаемых действием копирования в качестве источников и приемников, см. в разделе [Поддерживаемые хранилища данных и форматы](data-factory-data-movement-activities.md#supported-data-stores-and-formats). Чтобы узнать, как использовать конкретное хранилище данных в качестве источника или приемника, щелкните ссылку в таблице.  
 
Замените значение свойства **start** текущей датой, а значение свойства **end** — датой следующего дня. Можно указать только часть даты и пропустить временную часть указанной даты и времени. Например, 2017-02-03, что эквивалентно 2017-02-03T00:00:00Z.
 
Даты начала и окончания должны быть в [формате ISO](https://en.wikipedia.org/wiki/ISO_8601). Пример: 2016-10-14T16:32:41Z. Время **окончания** указывать не обязательно, однако в этом примере мы будем его использовать. 
 
Если не указать значение свойства **end**, оно вычисляется по формуле "**время начала + 48 часов**". Чтобы запустить конвейер в течение неопределенного срока, укажите значение **9999-09-09** в качестве значения свойства **end**.
 
В примере выше получено 24 среза данных, так как они создаются каждый час.

Описание свойств JSON в определении конвейера см. в статье [Конвейеры и действия в фабрике данных Azure](data-factory-create-pipelines.md). Описание свойств JSON в определении действия копирования см. в статье [Перемещение данных с помощью действия копирования](data-factory-data-movement-activities.md). Описание свойств JSON, поддерживаемых BlobSource, см. в статье о [соединителе больших двоичных объектов Azure](data-factory-azure-blob-connector.md). Описание свойств JSON, поддерживаемых SqlSink, см. в статье о [соединителе базы данных SQL Azure](data-factory-azure-sql-connector.md).

## <a name="set-global-variables"></a>Настройка глобальных переменных
Выполните следующие команды в Azure PowerShell, подставив собственные значения.

> [!IMPORTANT]
> Инструкции по получению значений для параметров client_id, client_secret, tenant и subscription_id см. в разделе [Предварительные требования](#prerequisites).   
> 
> 

```JSON
$client_id = "<client ID of application in AAD>"
$client_secret = "<client key of application in AAD>"
$tenant = "<Azure tenant ID>";
$subscription_id="<Azure subscription ID>";

$rg = "ADFTutorialResourceGroup"
```

После обновления имени используемой фабрики данных выполните следующую команду: 

```
$adf = "ADFCopyTutorialDF"
```

## <a name="authenticate-with-aad"></a>Проверка подлинности с помощью AAD
Выполните следующую команду для аутентификации с помощью Azure Active Directory (AAD): 

```PowerShell
$cmd = { .\curl.exe -X POST https://login.microsoftonline.com/$tenant/oauth2/token  -F grant_type=client_credentials  -F resource=https://management.core.windows.net/ -F client_id=$client_id -F client_secret=$client_secret };
$responseToken = Invoke-Command -scriptblock $cmd;
$accessToken = (ConvertFrom-Json $responseToken).access_token;

(ConvertFrom-Json $responseToken) 
```

## <a name="create-data-factory"></a>Создание фабрики данных
На этом шаге создается фабрика данных Azure с именем **ADFCopyTutorialDF**. Фабрика данных может иметь один или несколько конвейеров. Конвейер может содержать одно или несколько действий. Например, действие копирования, копирующее данные из исходного хранилища данных в целевое. Действие HDInsight Hive, запускающее скрипт Hive для преобразования входных данных в выходные данные продукта. Чтобы создать фабрику данных, выполните следующие команды: 

1. Назначьте команду переменной с именем **cmd**. 
   
    > [!IMPORTANT]
    > Указываемое здесь имя фабрики данных (ADFCopyTutorialDF) должно соответствовать имени, указанному в **datafactory.json**. 
   
    ```PowerShell
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@datafactory.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/ADFCopyTutorialDF0411?api-version=2015-10-01};
    ```
2. Выполните команду с использованием командлета **Invoke-Command**.
   
    ```PowerShell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Просмотрите результаты. После успешного создания фабрики данных в **результатах** появится JSON-файл. В противном случае отобразится сообщение об ошибке.  
   
    ```
    Write-Host $results
    ```

Обратите внимание на следующие моменты.

* Имя фабрики данных Azure должно быть глобально уникальным. Если в результатах отобразится сообщение об ошибке **Имя ADFCopyTutorialDF фабрики данных недоступно**, сделайте следующее:  
  
  1. Измените имя (например, на ваше_имя_ADFCopyTutorialDF) в файле **datafactory.json** .
  2. В первой команде, где переменной **$cmd** присваивается значение, замените ADFCopyTutorialDF на новое имя и выполните команду. 
  3. Выполните следующие две команды, чтобы вызвать REST API для создания фабрики данных и вывода результатов операции. 
     
     Ознакомьтесь с разделом [Фабрика данных — правила именования](data-factory-naming-rules.md) , чтобы узнать о правилах именования артефактов фабрики данных.
* Чтобы создать экземпляры фабрики данных, вы должны быть администратором или участником подписки Azure.
* В будущем имя фабрики данных может быть зарегистрировано в качестве DNS-имени и, следовательно, стать отображаемым.
* Если возникнет ошибка, сделайте следующее. **This subscription is not registered to use namespace Microsoft.DataFactory** (Подписка не зарегистрирована для использования пространства имен Microsoft.DataFactory), выполните одно из следующих действий и повторите попытку публикации: 
  
  * Чтобы зарегистрировать поставщик фабрики данных Azure, выполните следующую команду в Azure PowerShell: 

    ```PowerShell    
    Register-AzResourceProvider -ProviderNamespace Microsoft.DataFactory
    ```
    Чтобы убедиться, что поставщик фабрики данных зарегистрирован, выполните следующую команду: 
    
    ```PowerShell
    Get-AzResourceProvider
    ```
  * Войдите на [портал Azure](https://portal.azure.com) с использованием подписки Azure и откройте колонку фабрики данных или создайте на портале фабрику данных. Поставщик будет зарегистрирован автоматически.

Прежде чем создавать конвейер, необходимо создать несколько сущностей фабрики данных. Сначала вам нужно создать связанные службы, чтобы подключить исходное и целевое хранилища данных к своему хранилищу данных. Затем следует определить входные и выходные наборы данных для представления данных в связанных хранилищах. После этого создается конвейер с действием, которое использует эти наборы данных.

## <a name="create-linked-services"></a>Создание связанных служб
Связанная служба в фабрике данных связывает хранилища данных и службы вычислений с фабрикой данных. В этом руководстве не используются службы вычислений, например Azure HDInsight или Azure Data Lake Analytics. Вы используете два хранилища данных — служба хранилища Azure (источник) и база данных SQL Azure (конечное хранилище). Поэтому нужно создать две связанные службы с именами AzureStorageLinkedService и AzureSqlLinkedService и такими типами: AzureStorage и AzureSqlDatabase.  

Связанная служба хранилища Azure связывает учетную запись хранения Azure с фабрикой данных. В этой учетной записи хранения вы создали контейнер и отправили в нее данные в ходе выполнения предварительных [требований](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).   

Связанная служба SQL Azure (AzureSqlLinkedService) связывает Базу данных SQL Azure с фабрикой данных. В этой базе данных хранятся данные, скопированные из хранилища BLOB-объектов. Вы создали пустую таблицу в этой базе данных в ходе выполнения [предварительных требований](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).  

### <a name="create-azure-storage-linked-service"></a>Создание связанной службы хранения Azure
На этом шаге вы свяжете учетную запись хранения Azure с фабрикой данных. В этом разделе вы укажете имя и ключ вашей учетной записи хранения Azure. Дополнительные сведения о свойствах JSON для определения связанной службы хранилища Azure см. в разделе [Связанная служба хранилища Azure](data-factory-azure-blob-connector.md#azure-storage-linked-service).  

1. Назначьте команду переменной с именем **cmd**. 

    ```PowerShell   
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@azurestoragelinkedservice.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/linkedservices/AzureStorageLinkedService?api-version=2015-10-01};
    ```
2. Выполните команду с использованием командлета **Invoke-Command**.

    ```PowerShell   
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Просмотрите результаты. После успешного создания связанной службы в **результатах** появится JSON-файл. В противном случае отобразится сообщение об ошибке.

    ```PowerShell   
    Write-Host $results
    ```

### <a name="create-azure-sql-linked-service"></a>Создание связанной службы SQL Azure
На этом шаге вы свяжете Базу данных SQL Azure с фабрикой данных. В этом разделе вы укажете имя логического сервера SQL Server, имя базы данных, имя пользователя и пароль. Дополнительные сведения о свойствах JSON для определения связанной службы SQL Azure см. в разделе [Связанная служба SQL Azure](data-factory-azure-sql-connector.md#linked-service-properties).

1. Назначьте команду переменной с именем **cmd**. 
   
    ```PowerShell
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@azuresqllinkedservice.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/linkedservices/AzureSqlLinkedService?api-version=2015-10-01};
    ```
2. Выполните команду с использованием командлета **Invoke-Command**.
   
    ```PowerShell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Просмотрите результаты. После успешного создания связанной службы в **результатах** появится JSON-файл. В противном случае отобразится сообщение об ошибке.
   
    ```PowerShell
    Write-Host $results
    ```

## <a name="create-datasets"></a>Создание наборов данных
На предыдущем шаге вы создали связанные службы, связывающие учетную запись хранения Azure и Базу данных SQL Azure с фабрикой данных. На этом этапе вы определите два набора данных с именами AzureBlobInput и AzureSqlOutput, представляющие входные и выходные данные в хранилищах данных, на которые ссылаются службы AzureStorageLinkedService и AzureSqlLinkedService соответственно.

Связанная служба хранилища Azure указывает строку подключения, которую фабрика данных использует во время выполнения, чтобы подключиться к учетной записи хранения Azure. А входной набор данных больших двоичных объектов (AzureBlobInput) определяет контейнер и папку с входными данными.  

Аналогичным образом связанная служба "База данных SQL Azure" указывает строку подключения, которую служба "Фабрика данных" использует во время выполнения, чтобы подключиться к Базе данных SQL Azure. А выходной набор данных таблицы SQL (OututDataset) определяет таблицу в базе данных, в которую копируются данные из хранилища BLOB-объектов. 

### <a name="create-input-dataset"></a>Создание входного набора данных
На этом этапе вы создадите набор данных с именем AzureBlobInput. Он указывает на файл большого двоичного объекта (emp.txt) в корневой папке контейнера больших двоичных объектов в службе хранилища Azure, которая представлена связанной службой AzureStorageLinkedService. Если не указать значение fileName (или пропустить его), данные из всех больших двоичных объектов в папке входных данных копируются в целевое расположение. В этом руководстве вы укажете значение параметра fileName. 

1. Назначьте команду переменной с именем **cmd**. 

    ```PowerSHell   
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@inputdataset.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/datasets/AzureBlobInput?api-version=2015-10-01};
    ```
2. Выполните команду с использованием командлета **Invoke-Command**.
   
    ```PowerShell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Просмотрите результаты. После успешного создания набора данных в **результатах** появится JSON-файл. В противном случае отобразится сообщение об ошибке.
   
    ```PowerShell
    Write-Host $results
    ```

### <a name="create-output-dataset"></a>Создание выходного набора данных
Связанная служба Базы данных SQL Azure указывает строку подключения, которую служба "Фабрика данных" использует во время выполнения для подключения к Базе данных SQL Azure. Выходной набор данных таблицы SQL (OututDataset), который вы создаете на этом шаге, определяет таблицу в базе данных, в которую копируются данные из хранилища BLOB-объектов.

1. Назначьте команду переменной с именем **cmd**.

    ```PowerShell   
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@outputdataset.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/datasets/AzureSqlOutput?api-version=2015-10-01};
    ```
2. Выполните команду с использованием командлета **Invoke-Command**.
    
    ```PowerShell   
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Просмотрите результаты. После успешного создания набора данных в **результатах** появится JSON-файл. В противном случае отобразится сообщение об ошибке.
   
    ```PowerShell
    Write-Host $results
    ``` 

## <a name="create-pipeline"></a>Создание конвейера
На этом шаге создается конвейер с **действием копирования**, которое использует **AzureBlobInput** в качестве входных данных и **AzureSqlOutput** в качестве выходных.

Сейчас на основе этого набора настраивается расписание. В этом руководстве выходной набор данных создает срез раз в час. Для конвейера настроено время начала и время окончания с разницей в сутки. Таким образом, конвейер создает 24 среза для выходного набора данных. 

1. Назначьте команду переменной с именем **cmd**.

    ```PowerShell   
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@pipeline.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/datapipelines/MyFirstPipeline?api-version=2015-10-01};
    ```
2. Выполните команду с использованием командлета **Invoke-Command**.

    ```PowerShell   
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Просмотрите результаты. После успешного создания набора данных в **результатах** появится JSON-файл. В противном случае отобразится сообщение об ошибке.  

    ```PowerShell   
    Write-Host $results
    ```

**Поздравляем!** Фабрика данных Azure с конвейером, который копирует данные из Хранилища BLOB-объектов Azure в Базу данных SQL Azure, успешно создана.

## <a name="monitor-pipeline"></a>Отслеживание конвейера
На этом шаге используется REST API фабрики данных, чтобы отслеживать срезы, созданные конвейером.

```PowerShell
$ds ="AzureSqlOutput"
```

> [!IMPORTANT] 
> Убедитесь, что время начала и окончания, указанные в следующей команде, соответствуют времени начала и окончания конвейера. 

```PowerShell
$cmd = {.\curl.exe -X GET -H "Authorization: Bearer $accessToken" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/datasets/$ds/slices?start=2017-05-11T00%3a00%3a00.0000000Z"&"end=2017-05-12T00%3a00%3a00.0000000Z"&"api-version=2015-10-01};
```

```PowerShell
$results2 = Invoke-Command -scriptblock $cmd;
```

```PowerShell
IF ((ConvertFrom-Json $results2).value -ne $NULL) {
    ConvertFrom-Json $results2 | Select-Object -Expand value | Format-Table
} else {
        (convertFrom-Json $results2).RemoteException
}
```

Запускайте командлет Invoke-Command, пока не появится срез в состоянии **Готово** или **Сбой**. Когда срез будет в состоянии "Готово", проверьте выходные данные в таблице **emp** в Базе данных SQL Azure. 

Для каждого среза в таблицу emp в Базе данных SQL Azure копируются две строки данных из исходного файла. Таким образом, если все срезы успешно обработаны (в состоянии "Готово"), в таблице emp отобразится 24 записи. 

## <a name="summary"></a>Сводка
В этом руководстве вы с помощью REST API создали фабрику данных Azure для копирования данных из большого двоичного объекта Azure в Базу данных SQL Azure. Вот обобщенные действия, которые вы выполнили в этом руководстве:  

1. создание **фабрики данных Azure**;
2. Создание **связанных служб**.
   1. Служба хранилища Azure — связанная служба для связи с учетной записью хранения Azure, которая содержит входные данные.     
   2. Связанная служба SQL Azure для связи с базой данных, которая содержит выходные данные. 
3. Создание **наборов данных**, описывающих входные и выходные данные для конвейеров.
4. Создание **конвейера** с BlobSource в качестве источника и SqlSink в качестве приемника с помощью действия копирования. 

## <a name="next-steps"></a>Дальнейшие действия
В этом руководстве показано, как использовать Хранилище BLOB-объектов Azure и Базу данных SQL Azure как исходное и целевое хранилища данных в операции копирования. В следующей таблице приведен список хранилищ данных, которые поддерживаются в качестве источников и целевых расположений для действия копирования. 

[!INCLUDE [data-factory-supported-data-stores](../../../includes/data-factory-supported-data-stores.md)]

Чтобы получить дополнительные сведения о том, как скопировать данные в хранилище данных или из него, щелкните ссылку для хранилища данных в таблице.
