---
title: Руководство. Начало работы с анализом данных с помощью бессерверного пула SQL
description: В этом руководстве показано, как анализировать данные с помощью бессерверного пула SQL, используя данные, хранимые в базах данных Spark.
services: synapse-analytics
author: saveenr
ms.author: saveenr
manager: julieMSFT
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.subservice: sql
ms.topic: tutorial
ms.date: 12/31/2020
ms.openlocfilehash: 5f0a7477df2e281748c053ea8c7e7d3e79626296
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104588024"
---
# <a name="analyze-data-with-a-serverless-sql-pool"></a>Анализ данных с помощью бессерверного пула SQL

В этом руководстве показано, как анализировать данные с помощью бессерверного пула SQL, используя данные, хранимые в базах данных Spark. 

## <a name="the-built-in-serverless-sql-pool"></a>Бессерверный пул SQL "Встроенный"

Бессерверные пулы SQL позволяют использовать SQL без необходимости резервировать мощность. Выставление счетов за использование бессерверного пула SQL зависит от объема данных, обработанных для выполнения запроса, а не от количества узлов, используемых для выполнения запроса.

Каждая рабочая область поставляется с предварительно настроенным бессерверным пулом SQL, который называется **встроенным**. 

## <a name="analyze-nyc-taxi-data-in-blob-storage-using-serverless-sql-pool"></a>Анализ данных такси Нью-Йорка в Хранилище BLOB-объектов с помощью бессерверного пула SQL

В этом разделе показано, как с помощью серверного пула SQL выполнить анализ данных такси Нью-Йорка в учетной записи Хранилища BLOB-объектов Azure.

1. В Synapse Studio перейдите в центр **Разработка**.
1. Создайте новый скрипт SQL.
1. Вставьте следующий код в сценарий.

    ```
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK     'https://azureopendatastorage.blob.core.windows.net/nyctlc/yellow/puYear=*/puMonth=*/*.parquet',
            FORMAT = 'parquet'
        ) AS [result];
    ```
1. Нажмите кнопку **Выполнить**

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Анализ данных с помощью бессерверного пула Spark](get-started-analyze-spark.md)
