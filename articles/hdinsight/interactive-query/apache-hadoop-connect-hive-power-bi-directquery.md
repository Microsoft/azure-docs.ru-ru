---
title: См. статью интерактивный запрос данных Hive с помощью Power BI в Azure HDInsight.
description: Использование Microsoft Power BI для визуализации данных Interactive Query Hive из Azure HDInsight
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: how-to
ms.date: 06/17/2019
ms.openlocfilehash: 7f249bb0e81bf3a371b8743a304ef49baffaed7a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98941394"
---
# <a name="visualize-interactive-query-apache-hive-data-with-microsoft-power-bi-using-direct-query-in-hdinsight"></a>Визуализация интерактивных запросов Apache Hive данных с помощью Microsoft Power BI с использованием прямого запроса в HDInsight

Эта статья рассказывает о том, как подключить Microsoft Power BI к кластерам Interactive Query в Azure HDInsight и визуализировать данные Apache Hive с использованием прямого запроса. Приведенный пример загружает данные из `hivesampletable` таблицы Hive в Power BI. В `hivesampletable` таблице Hive содержатся некоторые данные об использовании мобильного телефона. Затем вы отобразите эти данные на карте мира:

![Отчет карты HDInsight Power BI](./media/apache-hadoop-connect-hive-power-bi-directquery/hdinsight-power-bi-visualization.png)

Можно использовать [драйвер ODBC Apache Hive](../hadoop/apache-hadoop-connect-hive-power-bi.md) для импорта с помощью универсального соединителя ODBC в Power BI Desktop. Но этот драйвер не рекомендуется для рабочих нагрузок бизнес-аналитик, с учетом того что ядро запросов Hive не является интерактивным. Для лучшей производительности используйте [соединитель интерактивных запросов HDInsight ](./apache-hadoop-connect-hive-power-bi-directquery.md) и [соединитель HDInsight Apache Spark](/power-bi/spark-on-hdinsight-with-direct-connect).

## <a name="prerequisites"></a>Предварительные условия
Чтобы выполнить действия, указанные в этой статье, вам потребуется:

* **Кластер HDInsight**. Это может быть кластер HDInsight с Apache Hive или новый кластер Interactive Query. Сведения о создании кластеров см. в [этом разделе](../hadoop/apache-hadoop-linux-tutorial-get-started.md).
* **[Microsoft Power BI Desktop](https://powerbi.microsoft.com/desktop/)**. Копию этой программы можно скачать в [Центре загрузки Майкрософт](https://www.microsoft.com/download/details.aspx?id=45331).

## <a name="load-data-from-hdinsight"></a>Загрузка данных из HDInsight

`hivesampletable`Таблица Hive поставляется со всеми кластерами HDInsight.

1. Запустите Power BI Desktop.

2. В строке меню выберите **Домашняя страница**  >  **получить**  >  **Дополнительные данные.**...

    ![Power BI получения данных в HDInsight](./media/apache-hadoop-connect-hive-power-bi-directquery/hdinsight-power-bi-open-odbc.png)

3. В окне **Получение данных** введите **hdinsight** в поле поиска.  

4. В результатах поиска выберите **HDInsight Interactive Query** и нажмите кнопку **подключить**.  Если вы не видите **интерактивный запрос HDInsight**, необходимо обновить Power BI Desktop до последней версии.

5. Нажмите кнопку **продолжить** , чтобы закрыть диалоговое окно " **Подключение к службе стороннего производителя** ".

6. В окне " **интерактивный запрос HDInsight** " введите следующие сведения и нажмите кнопку " **ОК**":

    |Свойство. | Значение |
    |---|---|
    |Сервер |Введите имя кластера, например *myiqcluster.azurehdinsight.NET*.|
    |База данных |Введите **значение по умолчанию** для этой статьи.|
    |Режим подключения к данным |Выберите **DirectQuery** для этой статьи.|

    ![Интерактивный запрос HDInsight, подключение DirectQuery в Power BI](./media/apache-hadoop-connect-hive-power-bi-directquery/hdinsight-interactive-query-power-bi-connect.png)

7. Введите учетные данные HTTP и нажмите кнопку **подключить**. Имя пользователя по умолчанию — **admin**.

8. В окне **навигатора** в левой области выберите **хивесамплетале**.

9. В главном окне выберите **Load (загрузить** ).

    ![Интерактивный запрос HDInsight, таблица hivesampletable в Power BI](./media/apache-hadoop-connect-hive-power-bi-directquery/hdinsight-interactive-query-power-bi-hivesampletable.png)

## <a name="visualize-data-on-a-map"></a>Визуализация данных на карте

Продолжите из последней процедуры.

1. В области визуализации выберите **Map**— значок земного шара. В главном окне появится Универсальная схема.

    ![Настройка отчета HDInsight Power BI](./media/apache-hadoop-connect-hive-power-bi-directquery/hdinsight-power-bi-customize.png)

2. В области "Поля" выберите **страна** и **devicemake**. Через несколько секунд в главном окне появится геосхема мира с точками данных.

3. Разверните карту.

## <a name="next-steps"></a>Дальнейшие действия
Из этой статьи вы узнали, как визуализировать данные HDInsight с помощью Microsoft Power BI.  Дополнительные сведения о визуализации данных см. в следующих статьях:

* [Визуализируйте Apache Hive данные с помощью Microsoft Power BI, используя ODBC в Azure HDInsight](../hadoop/apache-hadoop-connect-hive-power-bi.md). 
* [Использование Apache Zeppelin для выполнения запросов Apache Hive в Azure HDInsight](../interactive-query/hdinsight-connect-hive-zeppelin.md).
* [Подключение Excel к Hadoop в Azure HDInsight с помощью Microsoft Hive ODBC Driver](../hadoop/apache-hadoop-connect-excel-hive-odbc-driver.md)
* [Подключите Excel к Apache Hadoop с помощью Power Query](../hadoop/apache-hadoop-connect-excel-power-query.md).
* [Подключение к Azure HDInsight и выполнение запросов Apache Hive с помощью Средств Data Lake для Visual Studio](../hadoop/apache-hadoop-visual-studio-tools-get-started.md)
* [Используйте средство Azure HDInsight для Visual Studio Code](../hdinsight-for-vscode.md).
* [Отправка данных в HDInsight](./../hdinsight-upload-data.md).