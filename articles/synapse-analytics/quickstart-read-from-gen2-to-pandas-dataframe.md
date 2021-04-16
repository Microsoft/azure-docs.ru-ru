---
title: Краткое руководство. Чтение данных из ADLS 2-го поколения в кадр данных Pandas
description: Выполняйте чтение данных из учетной записи Azure Data Lake Storage 2-го поколения в кадр данных Pandas с помощью Python в Synapse Studio в Azure Synapse Analytics.
services: synapse-analytics
ms.service: synapse-analytics
ms.subservice: machine-learning
ms.topic: quickstart
ms.reviewer: jrasnick, garye, negust
ms.date: 03/23/2021
author: garyericson
ms.author: garye
ms.openlocfilehash: b7358c522cf12e7856496ad71fda393394e7ceab
ms.sourcegitcommit: f5448fe5b24c67e24aea769e1ab438a465dfe037
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105969580"
---
# <a name="quickstart-read-data-from-adls-gen2-to-pandas-dataframe-in-azure-synapse-analytics"></a>Краткое руководство. Чтение данных из ADLS 2-го поколения в кадр данных в Azure Synapse Analytics

В этом кратком руководстве вы узнаете, как использовать Python для чтения данных из Azure Data Lake Storage 2-го поколения (ADLS) в кадр данных Pandas в Azure Synapse Analytics.

С помощью записной книжки Synapse Studio можно выполнять следующие задачи.

- подключение к контейнеру в Data Lake Storage 2-го поколения, связанном с рабочей областью Azure Synapse Analytics;
- чтение данных из записной книжки PySpark с помощью `spark.read.load`;
- преобразование данных в кадр данных Pandas с помощью `.toPandas()`.

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/).
- Рабочая область Synapse Analytics с Data Lake Storage 2-го поколения, настроенным в качестве хранилища по умолчанию. Необходимо быть **участником для данных BLOB-объектов хранилища** файловой системы Data Lake Storage 2-го поколения, с которой вы работаете. Дополнительные сведения о создании рабочей области см. в разделе [Создание рабочей области Synapse](get-started-create-workspace.md).
- Пул Apache Spark в рабочей области. См. раздел [Создание бессерверного пула Apache Spark](get-started-analyze-spark.md#create-a-serverless-apache-spark-pool).

## <a name="sign-in-to-the-azure-portal"></a>Вход на портал Azure

Войдите на [портал Azure](https://portal.azure.com/).

## <a name="upload-sample-data-to-adls-gen2"></a>Отправка демонстрационных данных в ADLS 2-го поколения

1. На портале Azure создайте контейнер в том же Data Lake Storage 2-го поколения, который используется в Synapse Studio. Этот шаг можно пропустить, если вы хотите использовать связанную учетную запись хранения по умолчанию в рабочей области Azure Synapse Analytics.

1. В Synapse Studio щелкните **Данные**, перейдите на вкладку **Связано** и выберите контейнер в разделе **Azure Data Lake Storage 2-го поколения**.

1. Скачайте пример файла [RetailSales.csv](https://github.com/Azure-Samples/Synapse/blob/main/Notebooks/PySpark/Synapse%20Link%20for%20Cosmos%20DB%20samples/Retail/RetailData/RetailSales.csv) и отправьте его в контейнер.

1. Выберите переданный файл, щелкните **Свойства** и скопируйте значение **Путь ABFSS**.

## <a name="read-data-from-adls-gen2-into-a-pandas-dataframe"></a>Чтение данных из ADLS 2-го поколения в кадр данных Pandas

1. В области слева щелкните **Разработка**.

1. Щелкните **+** и выберите "Записная книжка", чтобы создать новую записную книжку.

1. В окне **Присоединить к** выберите пул Apache Spark. Если у вас его нет, щелкните **Создать пул Apache Spark**.

1. В ячейке кода записной книжки вставьте следующий код Python, добавив скопированный ранее путь ABFSS.

   ```python
   %%pyspark
   data_path = spark.read.load('<ABFSS Path to RetailSales.csv>', format='csv', header=True)
   data_path.show(10)
   
   print('Converting to Pandas.')
   
   pdf = data_path.toPandas()
   print(pdf)
   ```

1. Запустите эту ячейку.

Через несколько минут отображаемый текст должен выглядеть следующим образом.

```text
Command executed in 25s 324ms by gary on 03-23-2021 17:40:23.481 -07:00

Job execution Succeeded Spark 2 executors 8 cores

+-------+-----------+--------+-----------+-----------+-----+------------+--------------------+
|storeId|productCode|quantity|logQuantity|advertising|price|weekStarting|                  id|
+-------+-----------+--------+-----------+-----------+-----+------------+--------------------+
|      2| surface.go|     105|9.264828557|          1|  159|   6/15/2017|d6bd47a7-2ad6-4f0...|
|      2| surface.go|      80|8.987196821|          0|  269|   7/27/2017|64cc74c2-c7da-4e1...|
|      2| surface.go|      68|8.831711918|          1|  209|    8/3/2017|9a2d164b-5e44-44d...|
|      2| surface.go|      28|7.965545573|          0|  209|   8/10/2017|b8cd9987-1d5a-4f4...|
|      2| surface.go|      16|7.377758908|          0|  209|   8/24/2017|ac0ec099-e102-4bf...|
|      2| surface.go|     253| 10.1402973|          1|  189|   8/31/2017|3d22c002-b04c-409...|
|      2| surface.go|     107|9.282847063|          0|  189|    9/7/2017|b6e19699-d684-449...|
|      2| surface.go|      66|8.803273983|          0|  189|   9/14/2017|e89a5838-fb8f-413...|
|      2| surface.go|      65|8.793612072|          0|  179|   9/21/2017|c3278682-16c0-483...|
|      2| surface.go|      17|7.454719949|          0|  269|  10/12/2017|f40190c1-b2ed-46f...|
+-------+-----------+--------+-----------+-----------+-----+------------+--------------------+
only showing top 10 rows

Converting to Pandas.
      storeId  ...                                    id
0           2  ...  d6bd47a7-2ad6-4f0a-b8de-ed1386cae5ea
1           2  ...  64cc74c2-c7da-4e12-af64-c95bdf429934
2           2  ...  9a2d164b-5e44-44d7-9837-cf9ae6566c99
3           2  ...  b8cd9987-1d5a-4f4f-9346-719d73b1f7f0
4           2  ...  ac0ec099-e102-4bfc-9775-983b151dcd03
...       ...  ...                                   ...
28942     137  ...  6af00133-7015-415d-831b-ddf05bb5828c
28943     137  ...  1e0d3a21-ab43-49c4-89e2-49d202821807
28944     137  ...  5cc7e50a-6aa4-419b-a933-905a667aa2df
28945     137  ...  650ca506-7a4f-46f8-b2e1-e52ceffadf16
28946     137  ...  9bb216f6-04ec-4b61-9e68-34772b814c44

[28947 rows x 8 columns]
```

## <a name="next-steps"></a>Дальнейшие действия

- [Что такое Azure Synapse Analytics?](overview-what-is.md)
- [Начало работы с Azure Synapse Analytics](get-started.md)
- [Создание бессерверного пула Apache Spark](get-started-analyze-spark.md#create-a-serverless-apache-spark-pool)
