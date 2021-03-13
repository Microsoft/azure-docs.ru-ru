---
title: SQL Server данных в базу данных SQL с помощью фабрики данных Azure. процесс обработки и анализа данных группы
description: Настройка конвейера ADF, который объединяет два действия миграции данных, которые ежедневно перемещают данные между локальными базами данных и в облаке.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 09/03/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: e477e4bb3b31477f9407e981d4c8da2340411f55
ms.sourcegitcommit: d135e9a267fe26fbb5be98d2b5fd4327d355fe97
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102615743"
---
# <a name="move-data-from-a-sql-server-database-to-sql-database-with-azure-data-factory"></a>Перемещение данных из базы данных SQL Server в базу данных SQL с помощью фабрики данных Azure

В этой статье показано, как переместить данные из базы данных SQL Server в базу данных SQL Azure через хранилище BLOB-объектов Azure с помощью фабрики данных Azure (ADF). Этот метод является поддерживаемым традиционным подходом, имеющим преимущества реплицированной копии, но [мы рекомендуем взглянуть на нашу страницу переноса данных для получения последних вариантов](https://datamigration.microsoft.com/scenario/sql-to-azuresqldb?step=1).

Таблица, обобщающая различные варианты перемещения данных в базу данных SQL Azure, содержится в статье [Перемещение данных в базу данных SQL Azure для машинного обучения Azure](move-sql-azure.md).

## <a name="introduction-what-is-adf-and-when-should-it-be-used-to-migrate-data"></a><a name="intro"></a>Введение. Что такое ADF и когда ее использовать при переносе данных?
Фабрика данных Azure представляет собой полностью управляемую облачную службу интеграции информации, которая организует и автоматизирует перемещение и преобразование данных. Ключевым принципом модели ADF является конвейер. Конвейер — это логическая группа действий, каждое из которых определяет операции, выполняемые с данными, содержащимися в наборах данных. Информация, необходимая фабрике данных для подключения к внешним ресурсам, определяется связанными службами.

С ADF существующие службы обработки данных могут быть включены в конвейеры данных, обладающие высокой доступностью и управляемые в облаке. Для этих конвейеров данных можно запланировать прием, подготовку, преобразование, анализ и публикацию данных, а сложными данными и обработкой зависимостей управляет ADF. Решения можно быстро построить и развернуть в облаке, подключив большое количество локальных и облачных источников данных.

ADF стоит использовать в следующих случаях:

* когда данные нужно постоянно переносить в гибридном сценарии, при котором осуществляется доступ как к локальным, так и к облачным ресурсам;
* При необходимости преобразования или добавления к ним бизнес-логики во время переноса.

ADF позволяет выполнять планирование и отслеживание заданий с помощью простых сценариев JSON, управляющих перемещением данных на периодической основе. ADF также обладает другими возможностями, такими как поддержка сложных операций. Дополнительные сведения об ADF см. в документации по [фабрике данных Azure](https://azure.microsoft.com/services/data-factory/).

## <a name="the-scenario"></a><a name="scenario"></a>Сценарий
Мы настраиваем конвейер ADF, который объединяет два действия переноса данных. Вместе они перемещают данные на ежедневной основе между базой данных SQL Server и базой данных SQL Azure. Эти два действия таковы:

* Копирование данных из базы данных SQL Server в учетную запись хранения BLOB-объектов Azure
* Копирование данных из учетной записи хранилища BLOB-объектов Azure в базу данных SQL Azure.

> [!NOTE]
> Приведенные здесь шаги были адаптированы от более подробного руководства, предоставленного группой ADF: [копирование данных из базы данных SQL Server в хранилище BLOB-объектов Azure](../../data-factory/tutorial-hybrid-copy-portal.md) ссылки на соответствующие разделы этого раздела предоставляются при необходимости.
>
>

## <a name="prerequisites"></a><a name="prereqs"></a>Предварительные требования
Для выполнения действий, описанных в этом учебнике, вам необходимо следующее.

* **Подписка Azure**. Если у вас нет подписки, вы можете зарегистрироваться для получения [бесплатной пробной версии](https://azure.microsoft.com/pricing/free-trial/).
* **Учетная запись хранения Azure**. Учетная запись хранения Azure используется в этом учебнике для хранения данных. Если у вас ее нет, см. раздел [Создание учетной записи хранения](../../storage/common/storage-account-create.md). После создания учетной записи хранения необходимо получить ключ, используемый для доступа к хранилищу. См. [раздел Управление ключами доступа учетной записи хранения](../../storage/common/storage-account-keys-manage.md).
* Доступ к **базе данных SQL Azure**. Если необходимо настроить базу данных SQL Azure, раздел [Начало работы with база данных SQL Microsoft Azure](../../azure-sql/database/single-database-create-quickstart.md) предоставляет сведения о том, как подготавливать новый экземпляр базы данных SQL Azure.
* Установленная и настроенная локальная среда **Azure PowerShell**. Инструкции см. в статье [Приступая к работе с командлетами Azure PowerShell](/powershell/azure/).

> [!NOTE]
> В этой процедуре используется [портал Azure](https://portal.azure.com/).
>
>

## <a name="upload-the-data-to-your-sql-server-instance"></a><a name="upload-data"></a> Отправка данных в экземпляр SQL Server
Для демонстрации процесса переноса данных мы используем [набор данных о такси Нью-Йорка](https://chriswhong.com/open-data/foil_nyc_taxi/) . Набор данных о такси Нью-Йорка доступен, как отмечено в этой статье, в хранилище BLOB-объектов Azure [здесь](https://www.andresmh.com/nyctaxitrips/). Данные содержатся в двух файлах: trip_data.csv, содержащем сведения о поездках, и trip_far.csv, содержащем сведения о тарифах для каждой поездки. Пример и описание этих файлов приведены в [описании набора данных «Поездки такси Нью-Йорка»](sql-walkthrough.md#dataset).

Вы можете либо адаптировать описанные здесь процедуры к собственному набору данных, либо выполнить описанные действия с набором данных о такси Нью-Йорка. Чтобы передать набор данных Нью такси в базу данных SQL Server, выполните процедуру, описанную в разделе [инструкции по импорту данных в базу данных SQL Server](sql-walkthrough.md#dbload).

## <a name="create-an-azure-data-factory"></a><a name="create-adf"></a> Создание фабрики данных Azure
Инструкции по созданию фабрики данных Azure и группы ресурсов на [портале Azure](https://portal.azure.com/) представлены [здесь](../../data-factory/tutorial-hybrid-copy-portal.md#create-a-data-factory). Задайте имя *adfdsp* для нового экземпляра ADF и имя *adfdsprg* для созданной группы ресурсов.

## <a name="install-and-configure-azure-data-factory-integration-runtime"></a>Установка и настройка фабрики данных Azure Integration Runtime
Integration Runtime — это управляемая клиентом инфраструктура интеграции данных, используемая фабрикой данных Azure для обеспечения возможности интеграции данных в различных сетевых средах. Эта среда выполнения раньше называлась "Шлюз управления данными".

Чтобы настроить, [следуйте инструкциям по созданию конвейера](../../data-factory/tutorial-hybrid-copy-portal.md#create-a-pipeline) .

## <a name="create-linked-services-to-connect-to-the-data-resources"></a><a name="adflinkedservices"></a>Создание связанных служб для подключения к ресурсам данных
Информация, необходимая фабрике данных для подключения к внешним ресурсам, определяется связанными службами. В этом сценарии есть три ресурса, для которых требуются связанные службы.

1. Локальный сервер SQL Server
2. хранилище BLOB-объектов Azure
3. База данных SQL Azure

Пошаговая инструкция по созданию связанных служб приведена в [этом разделе](../../data-factory/tutorial-hybrid-copy-portal.md#create-a-pipeline).


## <a name="define-and-create-tables-to-specify-how-to-access-the-datasets"></a><a name="adf-tables"></a>Определение и создание таблиц для определения доступа к наборам данных
Для создания таблиц, которые определяют структуру, расположение и доступность наборов данных, используются следующие процедуры на основе сценариев. Для определения таблиц используются файлы JSON. Дополнительные сведения о структуре этих файлов см. в статье [Наборы данных в фабрике данных Azure](../../data-factory/concepts-datasets-linked-services.md).

> [!NOTE]
> Перед выполнением командлета [New-AzureDataFactoryTable](/previous-versions/azure/dn835096(v=azure.100)) необходимо проверить, правильно ли выбрана подписка для его выполнения, выполнив командлет `Add-AzureAccount`. Документацию по этим командлетам см. в статье [Add-AzureAccount](/powershell/module/servicemanagement/azure.service/add-azureaccount).
>
>

В определениях на основе JSON в таблицах используются следующие имена:

* **имя таблицы** в SQL Server *nyctaxi_data*
* the **имя контейнера** в учетной записи хранилища больших двоичных объектов Azure — *containername*

Для этого конвейера ADF необходимо определить три таблицы:

1. [Локальная таблица SQL](#adf-table-onprem-sql).
2. [Таблица больших двоичных объектов](#adf-table-blob-store)
3. [Таблица SQL Azure](#adf-table-azure-sql)

> [!NOTE]
> В этих процедурах для определения и создания действий ADF используется Azure PowerShell. Однако эти задачи также можно выполнить на портале Azure. Дополнительные сведения см. в разделе [Создание наборов данных](../../data-factory/tutorial-hybrid-copy-portal.md#create-a-pipeline).
>
>

### <a name="sql-on-premises-table"></a><a name="adf-table-onprem-sql"></a>Локальная таблица SQL.
Определение таблицы для SQL Server указывается в следующем JSON-файле:

```json
{
    "name": "OnPremSQLTable",
    "properties":
    {
        "location":
        {
            "type": "OnPremisesSqlServerTableLocation",
            "tableName": "nyctaxi_data",
            "linkedServiceName": "adfonpremsql"
        },
        "availability":
        {
            "frequency": "Day",
            "interval": 1,
            "waitOnExternal":
            {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

Сюда не были включены имена столбцов. Можно подделать подзапросы к именам столбцов, включив их здесь (Дополнительные сведения см. в [документации по ADF-документу](../../data-factory/copy-activity-overview.md) ).

Скопируйте определение таблицы JSON в файл с именем *onpremtabledef.json* и сохраните его в известном месте (например, *C:\temp\onpremtabledef.json*). Создайте таблицу в ADF с помощью следующего командлета Azure PowerShell:

```azurepowershell
New-AzureDataFactoryTable -ResourceGroupName ADFdsprg -DataFactoryName ADFdsp –File C:\temp\onpremtabledef.json
```


### <a name="blob-table"></a><a name="adf-table-blob-store"></a>таблица больших двоичных объектов; 
Определение таблицы выходных больших двоичных объектов задается следующим образом (здесь данные, полученные из локальных расположений, сопоставляются с большим двоичным объектом Azure):

```json
{
    "name": "OutputBlobTable",
    "properties":
    {
        "location":
        {
            "type": "AzureBlobLocation",
            "folderPath": "containername",
            "format":
            {
                "type": "TextFormat",
                "columnDelimiter": "\t"
            },
            "linkedServiceName": "adfds"
        },
        "availability":
        {
            "frequency": "Day",
            "interval": 1
        }
    }
}
```

Скопируйте определение таблицы JSON в файл с именем *bloboutputtabledef.json* и сохраните его в известном месте (например, *C:\temp\bloboutputtabledef.json*). Создайте таблицу в ADF с помощью следующего командлета Azure PowerShell:

```azurepowershell
New-AzureDataFactoryTable -ResourceGroupName adfdsprg -DataFactoryName adfdsp -File C:\temp\bloboutputtabledef.json
```

### <a name="sql-azure-table"></a><a name="adf-table-azure-sql"></a>таблица SQL Azure.
Определение таблицы для выходных данных SQL Azure задается следующим образом (эта схема сопоставляет данные, полученные из большого двоичного объекта).

```json
{
    "name": "OutputSQLAzureTable",
    "properties":
    {
        "structure":
        [
            { "name": "column1", "type": "String"},
            { "name": "column2", "type": "String"}
        ],
        "location":
        {
            "type": "AzureSqlTableLocation",
            "tableName": "your_db_name",
            "linkedServiceName": "adfdssqlazure_linked_servicename"
        },
        "availability":
        {
            "frequency": "Day",
            "interval": 1
        }
    }
}
```

Скопируйте определение таблицы JSON в файл с именем *AzureSqlTable.json* и сохраните его в известном месте (например, *C:\temp\AzureSqlTable.json*). Создайте таблицу в ADF с помощью следующего командлета Azure PowerShell:

```azurepowershell
New-AzureDataFactoryTable -ResourceGroupName adfdsprg -DataFactoryName adfdsp -File C:\temp\AzureSqlTable.json
```


## <a name="define-and-create-the-pipeline"></a><a name="adf-pipeline"></a>Определение и создание конвейера
Укажите действия, которые принадлежат конвейеру, и создайте конвейер с помощью следующих процедур на основе сценариев. Для определения свойств конвейера используется файл JSON.

* Сценарий предполагает, что **имя конвейера** — *AMLDSProcessPipeline*.
* Также обратите внимание, что мы задаем периодичность выполнения конвейера ежедневно и используем время выполнения задания по умолчанию (00:00 по Гринвичу).

> [!NOTE]
> В следующих процедурах для определения и создания конвейера ADF используется Azure PowerShell. Однако эту задачу также можно выполнить на портале Azure. Дополнительные сведения см. в разделе [Создание конвейера](../../data-factory/tutorial-hybrid-copy-portal.md#create-a-pipeline).
>
>

Используя приведенные выше определения таблиц, определение конвейера ADF указывается следующим образом:

```json
{
    "name": "AMLDSProcessPipeline",
    "properties":
    {
        "description" : "This pipeline has two activities: the first one copies data from SQL Server to Azure Blob, and the second one copies from Azure Blob to Azure Database Table",
        "activities":
        [
            {
                "name": "CopyFromSQLtoBlob",
                "description": "Copy data from SQL Server to blob",
                "type": "CopyActivity",
                "inputs": [ {"name": "OnPremSQLTable"} ],
                "outputs": [ {"name": "OutputBlobTable"} ],
                "transformation":
                {
                    "source":
                    {
                        "type": "SqlSource",
                        "sqlReaderQuery": "select * from nyctaxi_data"
                    },
                    "sink":
                    {
                        "type": "BlobSink"
                    }
                },
                "Policy":
                {
                    "concurrency": 3,
                    "executionPriorityOrder": "NewestFirst",
                    "style": "StartOfInterval",
                    "retry": 0,
                    "timeout": "01:00:00"
                }
            },
            {
                "name": "CopyFromBlobtoSQLAzure",
                "description": "Push data to Sql Azure",
                "type": "CopyActivity",
                "inputs": [ {"name": "OutputBlobTable"} ],
                "outputs": [ {"name": "OutputSQLAzureTable"} ],
                "transformation":
                {
                    "source":
                    {
                        "type": "BlobSource"
                    },
                    "sink":
                    {
                        "type": "SqlSink",
                        "WriteBatchTimeout": "00:5:00",
                    }
                },
                "Policy":
                {
                    "concurrency": 3,
                    "executionPriorityOrder": "NewestFirst",
                    "style": "StartOfInterval",
                    "retry": 2,
                    "timeout": "02:00:00"
                }
            }
        ]
    }
}
```

Скопируйте определение JSON конвейера в файл с именем *pipelinedef.json* и сохраните его в известном месте (например, *C:\temp\pipelinedef.json*). Создайте конвейер в ADF с помощью следующего командлета Azure PowerShell:

```azurepowershell
New-AzureDataFactoryPipeline  -ResourceGroupName adfdsprg -DataFactoryName adfdsp -File C:\temp\pipelinedef.json
```


## <a name="start-the-pipeline"></a><a name="adf-pipeline-start"></a>Запуск конвейера
Конвейер можно запустить с помощью следующей команды:

```azurepowershell
Set-AzureDataFactoryPipelineActivePeriod -ResourceGroupName ADFdsprg -DataFactoryName ADFdsp -StartDateTime startdateZ –EndDateTime enddateZ –Name AMLDSProcessPipeline
```

Значения параметров *startdate* и *enddate* необходимо заменить фактическими датами, определяющими временной интервал запуска конвейера.

После запуска конвейера вы сможете увидеть, как в контейнере большого двоичного объекта начнут появляться данные, по одному файлу в день.

Мы не использовали функциональные возможности, предоставляемые модулем ADF для добавочного канала данных. Дополнительные сведения об этой и других возможностях ADF см. в [документации по ADF](https://azure.microsoft.com/services/data-factory/).