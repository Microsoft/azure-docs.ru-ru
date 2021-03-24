---
title: Подключение Excel к Apache Hadoop с помощью Power Query — Azure HDInsight
description: Узнайте, как пользоваться преимуществами компонентов бизнес-аналитики и применять Power Query для Excel для получения доступа к данным, хранящимся в Azure HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/17/2019
ms.openlocfilehash: 13862e642c6a91fe6f3c635df2efde91672ecbad
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104866817"
---
# <a name="connect-excel-to-apache-hadoop-by-using-power-query"></a>Подключение Excel к Apache Hadoop с помощью Power Query

Одной из ключевых особенностей решения Майкрософт для работы с большими данными является интеграция компонентов бизнес-аналитики Майкрософт с кластерами Apache Hadoop в службе Azure HDInsight. Важнейшим примером является возможность подключения Excel к учетной записи хранения Azure, в которой хранятся данные, связанные с кластером Hadoop, с помощью надстройки Microsoft Power Query для Excel. В этой статье приводится пошаговое руководство по настройке и использованию Power Query для запроса данных, связанных с кластером Hadoop, который управляется с помощью HDInsight.

## <a name="prerequisites"></a>Предварительные требования

* Кластер Apache Hadoop в HDInsight. Ознакомьтесь со статьей [Краткое руководство. Использование Apache Hadoop и Apache Hive в Azure HDInsight с шаблоном Resource Manager](./apache-hadoop-linux-tutorial-get-started.md).
* Рабочая станция под управлением Windows 10, 7, Windows Server 2008 R2 или более поздней операционной системы.
* Microsoft 365 приложения для Enterprise, Office 2016, Office 2013 профессиональный плюс, Excel 2013 standalone или Office 2010 профессиональный плюс.

## <a name="install-microsoft-power-query"></a>Установка Microsoft Power Query

Power Query может импортировать данные, которые были выведены или созданы заданием Hadoop, выполняющимся в кластере HDInsight.

В Excel 2016 надстройка Power Query находится на вкладке "Данные" ленты в группе "Скачать & преобразовать". В предыдущих версиях Excel необходимо скачать надстройку Microsoft Power Query для Excel из [Центра загрузки Майкрософт](https://go.microsoft.com/fwlink/?LinkID=286689) и установить ее.

## <a name="import-hdinsight-data-into-excel"></a>Импорт данных HDInsight в Excel

Надстройка Power Query для Excel удобна для импорта данных из кластера HDInsight в Excel, где можно использовать средства бизнес-аналитики, такие как PowerPivot и Power Map, для изучения, анализа и представления данных.

1. Запустите Excel.

1. Создайте новую пустую книгу.

1. Выполните указанные ниже действия для вашей версии Excel.

   * Excel 2016

     * Выберите > **данные**  >  **Получение данных**  >  **из Azure**  >  **HDInsight (HDFS)**.

       :::image type="content" source="./media/apache-hadoop-connect-excel-power-query/powerquery-selecthdisource-excel2016.png" alt-text="HDi. PowerQuery. Селексдисаурце. 2016" border="true":::

   * Excel 2013 или 2010

     * Выберите **Power Query**  >  **из Azure**  >  **Microsoft Azure HDInsight**.

       :::image type="content" source="./media/apache-hadoop-connect-excel-power-query/powerquery-selecthdisource.png" alt-text="HDI.PowerQuery.SelectHdiSource" border="true":::

       **Примечание.** Если меню **Power Query** не отображается, последовательно выберите пункты **файл**  >  **Параметры** надстройки  >  и **надстройки COM** в раскрывающемся списке **управления** в нижней части страницы. Нажмите кнопку **Перейти...** и убедитесь, что установлен флажок «Power Query для Excel».

       **Примечание.** Power Query также позволяет импортировать данные из HDFS путем выбора **из других источников**.

1. В диалоговом окне **Azure HDInsight (HDFS)** в текстовом поле **имя учетной записи или URL-адрес** введите имя учетной записи хранилища больших двоичных объектов Azure, связанной с кластером. Нажмите кнопку **ОК**. Это может быть учетная запись хранения по умолчанию или связанная учетная запись хранения.  Формат — `https://StorageAccountName.blob.core.windows.net/`.

1. В поле **ключ учетной записи** введите ключ для учетной записи хранения BLOB-объектов и нажмите кнопку **подключить**. (Вводить данные учетной записи требуется только при первом доступе к этому магазину.)

1. В области **Навигатор** слева от редактора запросов дважды щелкните имя контейнера хранилища BLOB-объектов, связанного с кластером. По умолчанию имя контейнера совпадает с именем кластера.

1. Расположение **HiveSampleData.txt** в столбце **имени** (путь к папке — **.. /Хиве/варехаусе/хивесамплетабле/**), а затем выберите **двоичный файл** слева от HiveSampleData.txt. HiveSampleData.txt поставляется вместе с кластером. При необходимости можно использовать собственный файл.

    :::image type="content" source="./media/apache-hadoop-connect-excel-power-query/powerquery-importdata.png" alt-text="Импорт данных Power Query HDI в Excel" border="true":::

1. Если необходимо, можно переименовать имена столбцов. Когда будете готовы, нажмите кнопку **закрыть & загрузить**.  Данные загружены в книгу.

    :::image type="content" source="./media/apache-hadoop-connect-excel-power-query/powerquery-importedtable.png" alt-text="HDI Excel Power Query Imported Table" border="true":::

## <a name="next-steps"></a>Дальнейшие действия

В этой статье было показано, как использовать Power Query для извлечения данных из HDInsight в Excel. Аналогичным образом можно извлекать данные из HDInsight в базу данных SQL Azure. Также можно передать данные в HDInsight. Дополнительные сведения см. в следующих статьях:

* [Визуализируйте Apache Hive данные с помощью Microsoft Power BI в Azure HDInsight](apache-hadoop-connect-hive-power-bi.md).
* [Визуализация данных Hive в интерактивном запросе с помощью Power BI в Azure HDInsight](../interactive-query/apache-hadoop-connect-hive-power-bi-directquery.md).
* [Использование Apache Zeppelin для выполнения запросов Apache Hive в Azure HDInsight](../interactive-query/hdinsight-connect-hive-zeppelin.md).
* [Подключение Excel к Hadoop в Azure HDInsight с помощью Microsoft Hive ODBC Driver](apache-hadoop-connect-excel-hive-odbc-driver.md)
* [Подключение к Azure HDInsight и выполнение запросов Apache Hive с помощью Средств Data Lake для Visual Studio](apache-hadoop-visual-studio-tools-get-started.md)
* [Используйте средство Azure HDInsight для Visual Studio Code](../hdinsight-for-vscode.md).
* [Отправка данных в HDInsight](./../hdinsight-upload-data.md).
