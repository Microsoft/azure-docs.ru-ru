---
title: Интеграция Data Lake Storage 1-го поколения c другими службами Azure
description: Узнайте, как интегрировать Azure Data Lake Storage 1-го поколения с другими службами Azure.
author: twooley
ms.service: data-lake-store
ms.topic: conceptual
ms.date: 05/29/2018
ms.author: twooley
ms.openlocfilehash: 929853f4dbedca7034c8e2a51e6231651a2dd08f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96461665"
---
# <a name="integrating-azure-data-lake-storage-gen1-with-other-azure-services"></a>Интеграция Azure Data Lake Storage 1-го поколения с другими службами Azure
Azure Data Lake Storage 1-го поколения можно использовать вместе с другими службами Azure для охвата более широкого диапазона сценариев. В следующей статье перечислены службы, с которыми может интегрироваться Data Lake Storage 1-го поколения.

## <a name="use-data-lake-storage-gen1-with-azure-hdinsight"></a>Использование Data Lake Storage 1-го поколения с Azure HDInsight
Можно подготовить кластер [Azure HDInsight](https://azure.microsoft.com/documentation/learning-paths/hdinsight-self-guided-hadoop-training/), который использует Data Lake Storage 1-го поколения в качестве хранилища, соответствующего требованиям HDFS. В этом выпуске для кластеров Hadoop и Storm в Windows и Linux можно использовать Data Lake Storage 1-го поколения только в качестве дополнительного хранилища. В этих кластерах по-прежнему используется хранилище Azure (WASB) в качестве хранилища по умолчанию. Однако для кластеров HBase в Windows и Linux можно использовать Data Lake Storage 1-го поколения в качестве хранилища по умолчанию либо дополнительного хранилища, либо и того, и другого.

Инструкции по подготовке кластера HDInsight с Data Lake Storage 1-го поколения озера данных см. в следующих статьях.

* [Подготавливается кластер HDInsight с Data Lake Storage 1-го поколения с помощью портал Azure](data-lake-store-hdinsight-hadoop-use-portal.md)
* [Создание кластеров HDInsight, использующих Data Lake Storage 1-го как хранилище по умолчанию, с помощью Azure PowerShell](data-lake-store-hdinsight-hadoop-use-powershell-for-default-storage.md)
* [Создание кластеров HDInsight, использующих с Data Lake Storage 1-го поколения как дополнительное хранилище, с помощью Azure PowerShell](data-lake-store-hdinsight-hadoop-use-powershell.md)

## <a name="use-data-lake-storage-gen1-with-azure-data-lake-analytics"></a>Использование Data Lake Storage 1-го поколения с Azure Data Lake Analytics
[Аналитика озера данных Azure](../data-lake-analytics/data-lake-analytics-overview.md) позволяет работать с большими данными в облачных масштабах. Она динамически подготавливает ресурсы и позволяет выполнять аналитические операции с терабайтами и даже эксабайтами данных, которые могут храниться в разных поддерживаемых источниках данных, одним из которых является Data Lake Storage 1-го поколения. Azure Data Lake Analytics оптимизирована для работы с Data Lake Storage 1-го поколения, благодаря чему обеспечивается наивысший уровень производительности, пропускной способности и параллелизации для рабочих нагрузок больших данных.

Инструкции по использованию Data Lake Analytics с Data Lake Storage 1-го поколения см. в статье [Начало работы с Data Lake Analytics с помощью портала Azure](../data-lake-analytics/data-lake-analytics-get-started-portal.md).

## <a name="use-data-lake-storage-gen1-with-azure-data-factory"></a>Использование Data Lake Storage 1-го поколения с фабрикой данных Azure
Можно использовать [Фабрику данных Azure](https://azure.microsoft.com/services/data-factory/) для приема данных из таблиц Azure, Базы данных SQL Azure, хранилища данных SQL Azure, больших двоичных объектов хранилища Azure и локальных баз данных. Фабрика данных Azure является ведущим компонентом экосистемы Azure, ее можно использовать для координации приема данных из этих источников в Data Lake Storage 1-го поколения.

Инструкции по использованию фабрики данных Azure c Data Lake Storage 1-го поколения см. в разделе [Копирование данных в Data Lake Storage 1-го поколения и из него с помощью фабрики данных Azure](../data-factory/connector-azure-data-lake-store.md).

## <a name="copy-data-from-azure-storage-blobs-into-data-lake-storage-gen1"></a>Копирование данных из больших двоичных объектов хранилища Azure в Data Lake Storage 1-го поколения
Azure Data Lake Storage 1-го поколения предоставляет средство командной строки AdlCopy для копирования данных из хранилища больших двоичных объектов Azure в учетную запись Data Lake Storage 1-го поколения. Дополнительные сведения см. в статье [Копирование данных из больших двоичных объектов хранилища Azure в Data Lake Storage 1-го поколения](data-lake-store-copy-data-azure-storage-blob.md).

## <a name="copy-data-between-azure-sql-database-and-data-lake-storage-gen1"></a>Копирования данных между базой данных SQL Azure и Data Lake Storage 1-го поколения
Можно использовать Apache Sqoop для импорта и экспорта данных между базой данных SQL Azure и Data Lake Storage 1-го поколения. Дополнительные сведения см. в статье [копирование данных между Data Lake Storage 1-го поколения и базой данных SQL Azure с помощью Sqoop](data-lake-store-data-transfer-sql-sqoop.md).

## <a name="use-data-lake-storage-gen1-with-stream-analytics"></a>Использование Data Lake Storage 1-го поколения со Stream Analytics
Data Lake Storage 1-го поколения можно использовать для сохранения потока выходных данных с помощью Azure Stream Analytics. Дополнительные сведения см. в статье [Потоковая передача данных из Azure Storage Blob в Data Lake Storage 1-го поколения с помощью Azure Stream Analytics](data-lake-store-stream-analytics.md).

## <a name="use-data-lake-storage-gen1-with-power-bi"></a>Использование Data Lake Storage 1-го поколения с Power BI
Power BI можно использовать, чтобы импортировать данные из учетной записи Data Lake Storage 1-го поколения для анализа и визуализации. Дополнительные сведения см. в статье [Анализ данных в Data Lake Storage 1-го поколения с помощью Power BI](data-lake-store-power-bi.md).

## <a name="use-data-lake-storage-gen1-with-data-catalog"></a>Использование Data Lake Storage 1-го поколения с каталогом данных
Данные из Data Lake Storage 1-го поколения можно зарегистрировать в каталоге данных Azure, чтобы упростить их обнаружение в организации. Дополнительные сведения см. в статье [Регистрация данных из Data Lake Storage 1-го поколения в каталоге данных Azure](data-lake-store-with-data-catalog.md).

## <a name="use-data-lake-storage-gen1-with-sql-server-integration-services-ssis"></a>Использование Data Lake Storage 1-го поколения со службами SQL Server Integration Services (SSIS)
Вы можете использовать диспетчер подключений Data Lake Storage 1-го поколения в SSIS для подключения пакета служб SSIS к Data Lake Storage 1-го поколения. См. дополнительные сведения об [использовании Data Lake Storage 1-го поколения со службами SSIS](/sql/integration-services/connection-manager/azure-data-lake-store-connection-manager).

## <a name="use-data-lake-storage-gen1-with-azure-synapse-analytics"></a>Использование Data Lake Storage 1-го поколения с Azure синапсе Analytics
Polybase можно использовать для загрузки данных из Data Lake Storage 1-го поколения в Azure синапсе Analytics. Дополнительные сведения см. [в статье использование Data Lake Storage 1-го поколения с Azure синапсе Analytics](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-load-from-azure-data-lake-store.md).

## <a name="use-data-lake-storage-gen1-with-azure-event-hubs"></a>Использование Data Lake Storage 1-го поколения с центрами событий Azure
Azure Data Lake Storage 1-го поколения можно использовать для архивации и сбора данных, полученных центрами событий Azure. Дополнительные сведения см. в статье [Запись данных из центров событий с помощью Data Lake Storage 1-го поколения](data-lake-store-archive-eventhub-capture.md).

## <a name="see-also"></a>См. также раздел
* [Общие сведения об Azure Data Lake Storage Gen1](data-lake-store-overview.md)
* [Начало работы с Data Lake Storage 1-го поколения с помощью портала Azure](data-lake-store-get-started-portal.md)
* [Начало работы с Data Lake Storage 1-го поколения с помощью PowerShell](data-lake-store-get-started-powershell.md)