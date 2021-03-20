---
title: Службы Azure, поддерживающие Azure Data Lake Storage 2-го поколения | Документация Майкрософт
description: Узнайте, какие службы Azure интегрируются с Azure Data Lake Storage 2-го поколения
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: conceptual
ms.date: 02/17/2021
ms.author: normesta
ms.reviewer: stewu
ms.openlocfilehash: 36e1a8a288e1f9b2a8d65ab966b607b594d66f4e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100653607"
---
# <a name="azure-services-that-support-azure-data-lake-storage-gen2"></a>Службы Azure, поддерживающие Azure Data Lake Storage 2-го поколения

Службы Azure можно использовать для приема данных, выполнения анализа и создания визуальных представлений. Эта статья содержит список поддерживаемых служб Azure, раскрывает их уровень поддержки и содержит ссылки на статьи, которые помогут вам использовать эти службы с Azure Data Lake Storage 2-го поколения.

## <a name="supported-azure-services"></a>Поддерживаемые службы Azure

В этой таблице перечислены службы Azure, которые можно использовать с Azure Data Lake Storage 2-го поколения. Элементы в этих таблицах со временем будут меняться, так как поддержка постоянно расширяется.

> [!NOTE]
> Уровень поддержки относится только к поддержке службы с Data Lake Storage Gen 2.

|Служба Azure |Уровень поддержки |Azure AD |Общий ключ| Похожие статьи |
|---------------|-------------------|---|---|---|
|Фабрика данных Azure|Общедоступная версия|Да|Да|[Загрузка данных в Azure Data Lake Storage 2-го поколения с помощью Фабрики данных Azure](../../data-factory/load-azure-data-lake-storage-gen2.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)|
|Azure Databricks|Общедоступная версия|Да|Да|[Использование с Azure Databricks](/azure/databricks/data/data-sources/azure/azure-datalake-gen2) <br> [Руководство. Извлечение, преобразование и загрузка данных с помощью Azure Databricks](/azure/databricks/scenarios/databricks-extract-load-sql-data-warehouse) <br>[Учебник. доступ к данным Data Lake Storage 2-го поколения с помощью Azure Databricks с использованием Spark](data-lake-storage-use-databricks-spark.md)|
|концентратору событий Azure|Общедоступная версия|Нет|Да|[Сбор событий из Центров событий Azure в хранилище BLOB-объектов Azure или Azure Data Lake Storage](../../event-hubs/event-hubs-capture-overview.md)|
|Сетка событий Azure|Общедоступная версия|Да|Да|[Руководство по Реализация шаблона сохранения озера данных для обновления таблицы Databricks Delta](data-lake-storage-events.md)|
|Azure Logic Apps|Общедоступная версия|Нет|Да|[Обзор: Azure Logic Apps](../../logic-apps/logic-apps-overview.md)|
|Машинное обучение Azure|Общедоступная версия|Да|Да|[Доступ к данным в службах хранилища Azure](../../machine-learning/how-to-access-data.md)|
|Azure Stream Analytics|Общедоступная версия|Да|Да|[Краткое руководство. по созданию задания Stream Analytics с помощью портала Azure](../../stream-analytics/stream-analytics-quick-create-portal.md) <br> [Исходящий трафик в Azure Data Lake Gen2](../../stream-analytics/stream-analytics-define-outputs.md)|
|Data Box|Общедоступная версия|Нет|Да|[Использование Azure Data Box для переноса данных из локального хранилища HDFS в службу хранилища Azure](data-lake-storage-migrate-on-premises-hdfs-cluster.md)|
|HDInsight |Общедоступная версия|Да|Да|[Использование Azure Data Lake Storage Gen2 с кластерами Azure HDInsight](../../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2.md)<br>[Использование HDFS CLI в Data Lake Storage Gen2](data-lake-storage-use-hdfs-data-lake-storage.md) <br>[Руководство. Извлечение, преобразование и загрузка данных с помощью Apache Hive в Azure HDInsight](data-lake-storage-tutorial-extract-transform-load-hive.md)|
|Центр Интернета вещей |Общедоступная версия|Да|Да|[Использование маршрутизации сообщений в Центре Интернета вещей для отправки с устройства в облако в разные конечные точки](../../iot-hub/iot-hub-devguide-messages-d2c.md)|
|Power BI|Общедоступная версия|Да|Да|[Анализ данных в Data Lake Storage 2-го поколения с помощью Power BI](/power-query/connectors/datalakestorage)|
|Azure Synapse Analytics (прежнее название — Хранилище данных SQL)|Общедоступная версия|Да|Да|[Анализируйте данные в учетной записи хранения](../../synapse-analytics/get-started-analyze-storage.md)|
|SQL Server Integration Services (SSIS)|Общедоступная версия|Да|Да|[Диспетчер подключений службы хранилища Azure](/sql/integration-services/connection-manager/azure-storage-connection-manager)|
|Обозреватель данных Azure|Общедоступная версия|Да|Да|[Запрос данных в Azure Data Lake с помощью Azure Data Explorer](/azure/data-explorer/data-lake-query-data)|
|Когнитивный поиск Azure|Preview (Предварительный просмотр)|Да|Да|[Индексирование и поиск Azure Data Lake Storage 2-го поколения документов (Предварительная версия)](../../search/search-howto-index-azure-data-lake-storage.md)|
|Сеть доставки содержимого Azure|Еще не поддерживается|Неприменимо|Неприменимо|[Индексирование и поиск Azure Data Lake Storage 2-го поколения документов (Предварительная версия)](../../cdn/cdn-overview.md)|
|База данных SQL Azure|Еще не поддерживается|Неприменимо|Неприменимо|[Что такое База данных SQL Azure](../../azure-sql/database/sql-database-paas-overview.md)|

## <a name="see-also"></a>См. также раздел

- [Известные проблемы с Azure Data Lake Storage 2-го поколения](data-lake-storage-known-issues.md)
- [Функции хранилища BLOB-объектов, доступные в Azure Data Lake Storage 2-го поколения](data-lake-storage-supported-blob-storage-features.md)
- [Платформы с открытым кодом, поддерживающие Azure Data Lake Storage 2-го поколения](data-lake-storage-supported-open-source-platforms.md)
- [Multi-protocol access on Azure Data Lake Storage (preview)](data-lake-storage-multi-protocol-access.md) (Доступ с использованием нескольких протоколов на Azure Data Lake Storage (предварительная версия))