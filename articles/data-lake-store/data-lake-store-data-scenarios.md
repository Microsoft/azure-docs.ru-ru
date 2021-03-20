---
title: Сценарии работы с данными с использованием Data Lake Storage Gen1 | Документация Майкрософт
description: Сведения о разных сценариях и средствах для приема, обработки, скачивания и визуализации данных в Data Lake Storage Gen1 (прежнее название — Azure Data Lake Store).
services: data-lake-store
documentationcenter: ''
author: twooley
manager: mtillman
ms.service: data-lake-store
ms.devlang: na
ms.topic: conceptual
ms.date: 06/27/2018
ms.author: twooley
ms.openlocfilehash: 0bd895b0c19293a199b2a9b135915b7dc45b9e5e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98702503"
---
# <a name="using-azure-data-lake-storage-gen1-for-big-data-requirements"></a>Использование Data Lake Storage Gen1 для обеспечения соответствия требованиям больших данных

[!INCLUDE [data-lake-storage-gen1-rename-note.md](../../includes/data-lake-storage-gen1-rename-note.md)]

Процесс обработки больших данных состоит из указанных далее четырех ключевых этапов.

* Прием больших объемов данных в хранилище данных в режиме реального времени или в пакетах
* Обработка данных
* Загрузка данных
* Визуализация данных

В этой статье мы рассмотрим эти этапы применительно к Azure Data Lake Storage 1-го поколения, чтобы понять параметры и средства, доступные для удовлетворения потребностей в обработке больших объемов данных.

## <a name="ingest-data-into-data-lake-storage-gen1"></a>Прием данных в Azure Data Lake Storage 1-го поколения
В этом разделе описываются различные источники данных и способы поступления этих данных в учетную запись Azure Data Lake Storage 1-го поколения.

![Прием данных в Azure Data Lake Storage 1-го поколения](./media/data-lake-store-data-scenarios/ingest-data.png "Прием данных в Azure Data Lake Storage 1-го поколения")

### <a name="ad-hoc-data"></a>Специальные данные
Это небольшие наборы данных, которые используются для создания прототипов приложений для работы с большими данными. В зависимости от источника данных применяются разные способы приема специальных данных.

| Источник данных | Средство для приема |
| --- | --- |
| Локальный компьютер |<ul> <li>[Портал Azure](data-lake-store-get-started-portal.md)</li> <li>[Azure PowerShell](data-lake-store-get-started-powershell.md)</li> <li>[Azure CLI](data-lake-store-get-started-cli-2.0.md)</li> <li>[Data Lake Tools для Visual Studio](../data-lake-analytics/data-lake-analytics-data-lake-tools-get-started.md) </li></ul> |
| Большой двоичный объект хранилища Azure |<ul> <li>[Фабрика данных Azure](../data-factory/connector-azure-data-lake-store.md).</li> <li>[Средство AdlCopy](data-lake-store-copy-data-azure-storage-blob.md)</li><li>[DistCp, запущенный на кластере HDInsight](data-lake-store-copy-data-wasb-distcp.md)</li> </ul> |

### <a name="streamed-data"></a>Потоковые данные
Представляет данные, которые могут быть созданы различными источниками, такими как приложения, устройства, датчики и т. д. Эти данные можно принимать в Data Lake Storage 1-го поколения с помощью различных средств. Как правило, эти средства собирают и обрабатывают данные на основе событий в режиме реального времени, а затем записывают события в пакетном режиме в Data Lake Storage 1-го поколения для последующей обработки.

Ниже перечислены средства, которые можно использовать:

* [Azure Stream Analytics](../stream-analytics/stream-analytics-define-outputs.md). События, принятые в Центры событий, могут записываться в Azure Data Lake Storage 1-го поколения с помощью выходных данных Azure Data Lake Storage 1-го поколения.
* [Azure Storm для HDInsight](../hdinsight/storm/apache-storm-write-data-lake-store.md). Данные из кластера Storm можно напрямую записывать в Data Lake Storage 1-го поколения.
* [EventProcessorHost](../event-hubs/event-hubs-dotnet-standard-getstarted-send.md). Вы можете получать события из Центров событий, а затем записывать их в Data Lake Storage 1-го поколения с помощью [пакета SDK Data Lake Storage 1-го поколения для .NET](data-lake-store-get-started-net-sdk.md).

### <a name="relational-data"></a>Реляционные данные
Можно также извлекать данные из реляционных баз данных. В течение определенного периода времени реляционные базы данных собирают огромные объемы данных, которые после обработки с помощью конвейера больших данных могут предоставлять ценные сведения. Для перемещения таких данных в Data Lake Storage 1-го поколения можно использовать следующие средства.

* [Apache Sqoop](data-lake-store-data-transfer-sql-sqoop.md)
* [Фабрика данных Azure](../data-factory/copy-activity-overview.md).

### <a name="web-server-log-data-upload-using-custom-applications"></a>Данные журналов веб-сервера (отправка с помощью настраиваемых приложений)
Этот тип набора данных вызывается специально, так как анализ данных журналов веб-сервера часто используется в приложениях по работе с большими данными, и для его выполнения требуется отправка больших объемов файлов журналов в Data Lake Storage 1-го поколения. Для отправки таких данных воспользуйтесь следующими средствами или напишите собственные сценарии или приложения.

* [Azure CLI](data-lake-store-get-started-cli-2.0.md)
* [Azure PowerShell](data-lake-store-get-started-powershell.md)
* [Пакет SDK Data Lake Storage 1-го поколения для .NET](data-lake-store-get-started-net-sdk.md)
* [Фабрика данных Azure](../data-factory/copy-activity-overview.md).

Отличным способом отправки данных журналов веб-сервера и других типов данных (например данных общественных мнений) является использование собственных написанных сценариев или приложений, поскольку вы можете включить компонент отправки данных в состав более масштабного приложения по работе с большими объемами данных. В одних случаях этот код может иметь форму сценария или простой программы командной строки. В других случаях код может использоваться для интеграции обработки больших данных в бизнес-приложение или решение.

### <a name="data-associated-with-azure-hdinsight-clusters"></a>Данные, связанные с кластерами HDInsight Azure
Большинство типов кластера HDInsight (Hadoop, HBase, Storm) поддерживают Data Lake Storage 1-го поколения в качестве репозитория хранения данных. Кластеры HDInsight обращаются к данным из BLOB-объектов хранилища Azure (WASB). Для повышения производительности можно скопировать данные из WASB в учетную запись Data Lake Storage 1-го поколения, связанную с кластером. Для копирования данных можно использовать указанные далее средства.

* [Apache DistCp](data-lake-store-copy-data-wasb-distcp.md)
* [Служба AdlCopy](data-lake-store-copy-data-azure-storage-blob.md)
* [Фабрика данных Azure](../data-factory/connector-azure-data-lake-store.md).

### <a name="data-stored-in-on-premises-or-iaas-hadoop-clusters"></a>Данные, хранящиеся в локальных кластерах Hadoop или кластерах Hadoop в IaaS
Большие объемы данных могут храниться в кластерах Hadoop, размещенных локально на компьютерах, использующих HDFS. Кластеры Hadoop могут быть развернуты локально или работать в кластере IaaS в Azure. К копированию таких данных в Azure Data Lake Storage 1-го поколения могут предъявляться требования, в зависимости от того, является ли эта операция одноразовой или повторяющейся. Существуют различные возможности выполнить их. Ниже приведен список альтернативных вариантов и связанные с ними компромиссы.

| Метод | Сведения | Преимущества | Рекомендации |
| --- | --- | --- | --- |
| Использование Фабрики данных Azure (ADF) для копирования данных напрямую из кластеров Hadoop в Azure Data Lake Storage 1-го поколения |[ADF поддерживает HDFS в качестве источника данных.](../data-factory/connector-hdfs.md) |ADF реализована готовая поддержка HDFS, а также первоклассные инструменты комплексного управления и мониторинга. |Требуется развернуть шлюз управления данными в локальном кластере или кластере IaaS. |
| Экспорт данных из Hadoop в виде файлов. Затем — копирование этих файлов в Azure Data Lake Storage 1-го поколения с помощью соответствующего механизма. |Вы можете скопировать файлы в Azure Data Lake Storage 1-го поколения с помощью: <ul><li>[Azure PowerShell только для Windows](data-lake-store-get-started-powershell.md)</li><li>[Azure CLI](data-lake-store-get-started-cli-2.0.md)</li><li>Пользовательское приложение, использующее любой пакет SDK для Azure Data Lake Storage 1-го поколения</li></ul> |Можно быстро приступить к работе. Возможны настраиваемые передачи данных. |Многоэтапный процесс с использованием нескольких технологий. Учитывая настраиваемый характер инструментов, со временем будет все сложнее осуществлять управление и мониторинг. |
| Использование Distcp для копирования данных из Hadoop в службу хранилища Azure. Затем — копирование данных из службы хранилища Azure в Data Lake Storage 1-го поколения с помощью соответствующего механизма. |Скопировать данные из службы хранилища Azure в Data Lake Storage 1-го поколения можно с помощью: <ul><li>[Фабрика данных Azure](../data-factory/copy-activity-overview.md).</li><li>[Средство AdlCopy](data-lake-store-copy-data-azure-storage-blob.md)</li><li>[Apache DistCp, запущенный в кластерах HDInsight](data-lake-store-copy-data-wasb-distcp.md)</li></ul> |Можно использовать инструменты с открытым кодом. |Многоэтапный процесс с использованием нескольких технологий. |

### <a name="really-large-datasets"></a>Очень большие наборы данных
Для отправки наборов данных размером в несколько терабайт использование описанных выше методов иногда может быть медленным и затратным процессом. В таких ситуациях будут уместны следующие варианты.

* **Использование Azure ExpressRoute.** Azure ExpressRoute позволяет создавать закрытые соединения между ЦОД Azure и вашей локальной инфраструктурой. Это надежный вариант для передачи больших объемов данных. Дополнительные сведения см. в [техническом обзоре ExpressRoute](../expressroute/expressroute-introduction.md).
* **Автономная передача данных**. Если по какой-то причине использовать Azure ExpressRoute нельзя, используйте [службу импорта и экспорта Azure](../import-export/storage-import-export-service.md) для доставки жестких дисков с данными в центр обработки данных Azure. Данные сначала будут отправлены в хранилище BLOB-объектов Azure. Затем с помощью [Фабрики данных Azure](../data-factory/connector-azure-data-lake-store.md) или [инструмента AdlCopy](data-lake-store-copy-data-azure-storage-blob.md) можно скопировать данные из больших двоичных объектов службы хранилища Azure в Data Lake Storage 1-го поколения.

  > [!NOTE]
  > При использовании службы импорта и экспорта размеры файлов на дисках, отправляемых в центр обработки данных Azure, не должны превышать 195 ГБ.
  >
  >

## <a name="process-data-stored-in-data-lake-storage-gen1"></a>Обработка данных, хранящихся в Data Lake Storage 1-го поколения
Данные, доступные в Data Lake Storage 1-го поколения, можно проанализировать с помощью поддерживаемых приложений для работы с большими данными. Сейчас для запуска заданий анализа для данных, хранящихся в Data Lake Storage 1-го поколения, можно использовать Azure HDInsight и Azure Data Lake Analytics.

![Анализ данных в Data Lake Storage 1-го поколения](./media/data-lake-store-data-scenarios/analyze-data.png "Анализ данных в Data Lake Storage 1-го поколения")

Рассмотрите следующие примеры.

* [Создание кластера HDInsight с Data Lake Storage 1-го поколения в качестве хранилища](data-lake-store-hdinsight-hadoop-use-portal.md)
* [Начало работы с Azure Data Lake Analytics с помощью портала Azure](../data-lake-analytics/data-lake-analytics-get-started-portal.md)

## <a name="download-data-from-data-lake-storage-gen1"></a>Копирование из Data Lake Storage 1-го поколения
Возможно, вам потребуется загрузить или переместить данные из Azure Data Lake Storage 1-го поколения в случаях, аналогичных указанным далее.

* Перемещение данных в другие репозитории для взаимодействия с существующими конвейерами обработки данных. Например, может потребоваться переместить данные из Data Lake Storage 1-го поколения в базу данных SQL Azure или SQL Server.
* Загрузка данных на локальный компьютер для обработки в средах IDE при создании прототипов приложений.

![Исходящие данные из Data Lake Storage 1-го поколения](./media/data-lake-store-data-scenarios/egress-data.png "Исходящие данные из Data Lake Storage 1-го поколения")

В таких ситуациях можно использовать любой из следующих вариантов:

* [Apache Sqoop](data-lake-store-data-transfer-sql-sqoop.md)
* [Фабрика данных Azure](../data-factory/copy-activity-overview.md).
* [Apache DistCp](data-lake-store-copy-data-wasb-distcp.md)

Для написания сценария или приложения для загрузки данных из Data Lake Storage 1-го поколения можно воспользоваться следующими средствами.

* [Azure CLI](data-lake-store-get-started-cli-2.0.md)
* [Azure PowerShell](data-lake-store-get-started-powershell.md)
* [Пакет SDK Data Lake Storage 1-го поколения для .NET](data-lake-store-get-started-net-sdk.md)

## <a name="visualize-data-in-data-lake-storage-gen1"></a>Визуализация данных в Data Lake Storage 1-го поколения
Для создания визуальных представлений данных, хранящихся в Data Lake Storage 1-го поколения, можно использовать сочетание служб.

![Визуализация данных в Data Lake Storage 1-го поколения](./media/data-lake-store-data-scenarios/visualize-data.png "Визуализация данных в Data Lake Storage 1-го поколения")

* Вы можете начать с использования [фабрики данных Azure для перемещения данных из Data Lake Storage 1-го поколения в Azure синапсе Analytics](../data-factory/copy-activity-overview.md) .
* После этого можно [интегрировать Power BI с Azure синапсе Analytics](/power-bi/connect-data/service-azure-sql-data-warehouse-with-direct-connect) для создания визуального представления данных.