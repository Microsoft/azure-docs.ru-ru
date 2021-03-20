---
title: Преобразование данных с помощью действия хранимой процедуры
description: В этой статье содержатся сведения о том, как использовать действия хранимой процедуры SQL Server для вызова хранимой процедуры в базе данных SQL Azure или хранилище данных из конвейера фабрики данных.
ms.service: data-factory
ms.topic: conceptual
author: nabhishek
ms.author: abnarain
ms.custom: seo-lt-2019
ms.date: 11/27/2018
ms.openlocfilehash: b9ba2f9de82522d4348fa341ad0b41d43c3eebcc
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100375652"
---
# <a name="transform-data-by-using-the-sql-server-stored-procedure-activity-in-azure-data-factory"></a>Преобразование данных с помощью действия хранимой процедуры SQL Server в фабрике данных Azure
> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](v1/data-factory-stored-proc-activity.md)
> * [Текущая версия](transform-data-using-stored-procedure.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Действия преобразования данных в [конвейере](concepts-pipelines-activities.md) фабрики данных позволяют преобразовать необработанные данные и переработать их в прогнозы и аналитику. Действие хранимой процедуры — это одно из действий преобразования данных, которые поддерживает фабрика данных. Данная статья основана на материалах статьи о [преобразовании данных](transform-data.md), в которой приведен общий обзор преобразования данных и список поддерживаемых действий преобразования в фабрике данных.

> [!NOTE]
> Если вы не знакомы с Фабрикой данных Azure, сначала ознакомьтесь со статьей [Введение в фабрику данных Azure](introduction.md) и руководством [Преобразование данных в облаке с помощью действия Spark в фабрике данных Azure](tutorial-transform-data-spark-powershell.md) перед чтением этой статьи. 

C его помощью можно вызвать хранимую процедуру в одном из следующих хранилищ данных вашего предприятия или на виртуальной машине Azure. 

- База данных SQL Azure
- Azure Synapse Analytics
- База данных SQL Server.  Если вы используете SQL Server, установите локальную среду выполнения интеграции на том же компьютере, на котором размещена база данных, или на отдельном компьютере, имеющем доступ к базе данных. Локальная среда выполнения интеграции — это компонент, который обеспечивает безопасное и управляемое подключение локальных источников данных или данных виртуальной машины Azure к облачным службам. Дополнительные сведения см. в статье локальная [Среда выполнения интеграции](create-self-hosted-integration-runtime.md) .

> [!IMPORTANT]
> При копировании данных в базу данных SQL Azure или SQL Server можно настроить класс **SqlSink** в действии копирования для вызова хранимой процедуры с помощью свойства **sqlWriterStoredProcedureName**. Дополнительные сведения о свойствах см. в следующих статьях о соединителях: [Перемещение данных в базу данных SQL Azure и из нее с помощью фабрики данных Azure](connector-azure-sql-database.md) и [Перемещение данных в базу данных SQL Server и обратно на локальных компьютерах и виртуальных машинах Azure IaaS с помощью фабрики данных Azure](connector-sql-server.md). Вызов хранимой процедуры при копировании данных в Azure синапсе Analytics с помощью действия копирования не поддерживается. Однако действие хранимой процедуры можно использовать для вызова хранимой процедуры в Azure синапсе Analytics. 
>
> При копировании данных из базы данных SQL Azure или SQL Server или Azure синапсе Analytics можно настроить **SqlSource** в действии копирования, чтобы вызвать хранимую процедуру для считывания данных из базы данных-источника с помощью свойства **sqlReaderStoredProcedureName** . Дополнительные сведения см. в следующих статьях о соединителях: [база данных SQL Azure](connector-azure-sql-database.md), [SQL Server](connector-sql-server.md), [Azure синапсе Analytics](connector-azure-sql-data-warehouse.md) .          

 

## <a name="syntax-details"></a>Сведения о синтаксисе
Ниже приведен формат JSON для определения действия хранимой процедуры:

```json
{
    "name": "Stored Procedure Activity",
    "description":"Description",
    "type": "SqlServerStoredProcedure",
    "linkedServiceName": {
        "referenceName": "AzureSqlLinkedService",
        "type": "LinkedServiceReference"
    },
    "typeProperties": {
        "storedProcedureName": "usp_sample",
        "storedProcedureParameters": {
            "identifier": { "value": "1", "type": "Int" },
            "stringData": { "value": "str1" }

        }
    }
}
```

В следующей таблице описаны эти свойства JSON:

| Свойство.                  | Описание                              | Обязательно |
| ------------------------- | ---------------------------------------- | -------- |
| name                      | Имя действия.                     | Да      |
| description               | Текст, описывающий, для чего используется действие | Нет       |
| type                      | Для действия хранимой процедуры тип действия — **SqlServerStoredProcedure** . | Да      |
| linkedServiceName         | Ссылка на **базу данных SQL Azure** или **Azure синапсе Analytics** или **SQL Server** зарегистрированная как связанная служба в фабрике данных. Дополнительные сведения об этой связанной службе см. в статье [Вычислительные среды, поддерживаемые фабрикой данных Azure](compute-linked-services.md). | Да      |
| storedProcedureName       | Укажите имя хранимой процедуры, которую нужно вызвать. | Да      |
| storedProcedureParameters | Укажите значения для параметров хранимой процедуры. Используйте `"param1": { "value": "param1Value","type":"param1Type" }`, чтобы передать значения параметра и их тип, поддерживаемый источником данных. Если для параметра необходимо передать значение null, используйте синтаксис `"param1": { "value": null }` (все знаки в нижнем регистре). | Нет       |

## <a name="parameter-data-type-mapping"></a>Сопоставление типов данных параметров
Тип данных, указанный для параметра, — это тип фабрики данных Azure, который сопоставляется с типом данных в используемом источнике данных. Сопоставления типов данных для источника данных можно найти в области Соединители. Некоторые примеры

| Источник данных          | Сопоставление типов данных |
| ---------------------|-------------------|
| Azure Synapse Analytics | https://docs.microsoft.com/azure/data-factory/connector-azure-sql-data-warehouse#data-type-mapping-for-azure-sql-data-warehouse |
| База данных SQL Azure   | https://docs.microsoft.com/azure/data-factory/connector-azure-sql-database#data-type-mapping-for-azure-sql-database | 
| Oracle;               | https://docs.microsoft.com/azure/data-factory/connector-oracle#data-type-mapping-for-oracle |
| SQL Server           | https://docs.microsoft.com/azure/data-factory/connector-sql-server#data-type-mapping-for-sql-server |




## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь со следующими ссылками, в которых описаны способы преобразования данных другими способами: 

* [Действие U-SQL](transform-data-using-data-lake-analytics.md)
* [Действие Hive](transform-data-using-hadoop-hive.md)
* [Действие Pig](transform-data-using-hadoop-pig.md)
* [Действие MapReduce](transform-data-using-hadoop-map-reduce.md)
* [Действие потоковой передачи Hadoop](transform-data-using-hadoop-streaming.md)
* [Действие Spark](transform-data-using-spark.md)
* [Настраиваемое действие .NET](transform-data-using-dotnet-custom-activity.md)
* [Действие выполнения пакета в Студии машинного обучения Azure (классическая)](transform-data-using-machine-learning.md)
* [Действие хранимой процедуры](transform-data-using-stored-procedure.md)
