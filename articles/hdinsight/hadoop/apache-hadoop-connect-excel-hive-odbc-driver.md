---
title: Excel & Apache Hadoop с драйвером ODBC — Azure HDInsight
description: Узнайте, как установить и использовать драйвер Microsoft Hive ODBC для Excel, чтобы запрашивать данные в кластерах HDInsight из Microsoft Excel.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,hdiseo17may2017,seoapr2020
ms.date: 04/22/2020
ms.openlocfilehash: 2c528859ea5abc6267c10a2ede9c2ca99f84e22f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98946807"
---
# <a name="connect-excel-to-apache-hadoop-in-azure-hdinsight-with-the-microsoft-hive-odbc-driver"></a>Подключение Excel к Apache Hadoop с помощью драйвера Microsoft Hive ODBC в Azure HDInsight

[!INCLUDE [ODBC-JDBC-selector](../../../includes/hdinsight-selector-odbc-jdbc.md)]

Решение Microsoft для работы с большими данными интегрирует компоненты бизнес-аналитики Майкрософт с Apache Hadoop кластерами, развернутыми в HDInsight. Примером может быть возможность подключения Excel к хранилищу данных Hive кластера Hadoop. Подключитесь с помощью драйвера Microsoft Hive Open Database Connectivity (ODBC).

Вы можете подключить данные, связанные с кластером HDInsight, из Excel с помощью надстройки Microsoft Power Query для Excel. Дополнительные сведения см. в статье [Подключение Excel к HDInsight с помощью Power Query](./apache-hadoop-connect-excel-power-query.md).

## <a name="prerequisites"></a>Предварительные требования

Перед началом работы с этой статьей необходимо иметь следующее:

* Кластер HDInsight Hadoop. Дополнительные сведения о создании кластера см. в статье [Приступая к работе с Hadoop в HDInsight](apache-hadoop-linux-tutorial-get-started.md).
* Рабочая станция с Office 2010 Professional Plus или более поздней версии или Excel 2010 или более поздней версии.

## <a name="install-microsoft-hive-odbc-driver"></a>Установка драйвера Microsoft Hive ODBC

Скачайте и установите [Microsoft Hive ODBC Driver](https://www.microsoft.com/download/details.aspx?id=40886). Выберите версию, соответствующую версии приложения, в которой вы будете использовать драйвер ODBC.  Для этой статьи используется драйвер для Office Excel.

## <a name="create-apache-hive-odbc-data-source"></a>Создание источника данных Apache Hive ODBC

Ниже показано, как создать источник данных Hive ODBC.

1. В Windows перейдите в меню **пуск > средства администрирования windows > источники данных ODBC (32-разрядная версия)/(64-разрядная версия)**.  Это действие открывает окно **Администратор источников данных ODBC** .

    ![Администратор источника данных ODBC](./media/apache-hadoop-connect-excel-hive-odbc-driver/simbahiveodbc-datasourceadmin1.png "Настройка DSN с помощью администратора источников данных ODBC")

1. На вкладке **DSN пользователя** выберите **Добавить**, чтобы открыть окно **Создание нового источника данных**.

1. Выберите **Microsoft Hive ODBC Driver**, а затем — **Готово**, чтобы открыть окно **Microsoft Hive ODBC Driver DSN Setup** (Настройка DSN Microsoft Hive ODBC Driver).

1. Введите или выберите следующие значения:

   | Свойство. | Описание |
   | --- | --- |
   |  Имя базы данных-источника |Присвойте имя источнику данных |
   |  Узлы |Введите `HDInsightClusterName.azurehdinsight.net`. Например, `myHDICluster.azurehdinsight.net`. Примечание. `HDInsightClusterName-int.azurehdinsight.net` поддерживается до тех пор, пока клиентская виртуальная машина будет соединена с той же виртуальной сетью. |
   |  Порт |Используйте **443**. (Этот порт был изменен с 563 на 443.) |
   |  База данных |Использовать **значение по умолчанию**. |
   |  Механизм |Выберите **Windows Azure HDInsight Service**. |
   |  Имя пользователя |Введите имя пользователя HTTP кластера HDInsight. Имя пользователя по умолчанию — **admin**. |
   |  Пароль |Введите пароль пользователя кластера HDInsight. Установите флажок **Save Password (Encrypted)** (Сохранить пароль (зашифрованный)).|

1. Необязательно: выберите **Дополнительные параметры...**  

   | Параметр | Описание |
   | --- | --- |
   |  Использовать исходный запрос |При выборе этого параметра драйвер ODBC НЕ пытается преобразовать TSQL в HiveQL. Его следует использовать только в том случае, если у вас 100%, что вы отправляете чистые инструкции HiveQL. При подключении к серверу SQL Server или базе данных Azure SQL необходимо снять этот флажок. |
   |  Строки, загружаемые для каждого блока |При получении большого объема записей включение этого параметра может обеспечить оптимальную производительность. |
   |  Длина столбца строки по умолчанию, длина столбца двоичного кода, масштаб столбца десятичных значений |Длина и точность типа данных может повлиять на способ выведения данных. Они приводят к возврату неверных данных из-за потери точности и или усечения. |

    ![Дополнительные параметры конфигурации DSN](./media/apache-hadoop-connect-excel-hive-odbc-driver/hiveodbc-datasource-advancedoptions1.png "Дополнительные параметры конфигурации DSN")

1. Щелкните **Тест** для проверки источника данных. Если источник данных настроен правильно, результат теста будет отображаться **успешно.**

1. Нажмите кнопку **ОК**, чтобы закрыть окно тестов.  

1. Нажмите кнопку **ОК**, чтобы закрыть окно **Microsoft Hive ODBC Driver DSN Setup** (Настройка DSN Microsoft Hive ODBC Driver).  

1. Нажмите кнопку **ОК**, чтобы закрыть окно **Администратор источников данных ODBC**.  

## <a name="import-data-into-excel-from-hdinsight"></a>Импорт данных в Excel из службы HDInsight

Ниже описан способ импорта данных из таблицы Hive в рабочую книгу Excel с помощью источника данных ODBC, созданного в предыдущем разделе.

1. Откройте новую или существующую рабочую книгу в Excel.

2. На вкладке **Данные** перейдите к разделу **Получить данные** > **Из других источников** > **Из ODBC**, чтобы открыть окно **Из ODBC**.

    ![Открытие мастера подключения к данным Excel](./media/apache-hadoop-connect-excel-hive-odbc-driver/simbahiveodbc-excel-dataconnection1.png "Открытие мастера подключения к данным Excel")

3. В раскрывающемся списке выберите имя источника данных, созданное в последнем разделе, и нажмите кнопку **ОК**.

4. При первом использовании откроется диалоговое окно **драйвера ODBC** . В меню слева выберите пункт **Windows** . Затем нажмите кнопку **Подключиться** , чтобы открыть окно **навигатора** .

5. В окне **Навигатор** перейдите к **HIVE** > **по умолчанию** > **hivesampletable**, а затем нажмите кнопку **Загрузить**. Для импорта данных в Excel потребуется несколько секунд.

    ![Навигатор по ODBC для Hive в HDInsight Excel](./media/apache-hadoop-connect-excel-hive-odbc-driver/hdinsight-hive-odbc-navigator.png "Навигатор по ODBC для Hive в HDInsight Excel")

## <a name="next-steps"></a>Дальнейшие действия

В рамках этой статьи вы узнали, как получить данные из службы HDInsight в Excel с помощью драйвера Microsoft Hive ODBC. Аналогичным образом можно получать данные из службы HDInsight в базу данных SQL. Также можно передать данные в службу HDInsight. Дополнительные сведения см. на следующих ресурсах:

* [Визуализируйте Apache Hive данные с помощью Microsoft Power BI в Azure HDInsight](apache-hadoop-connect-hive-power-bi.md).
* [Визуализация данных Hive в интерактивном запросе с помощью Power BI в Azure HDInsight](../interactive-query/apache-hadoop-connect-hive-power-bi-directquery.md).
* [Подключите Excel к Apache Hadoop с помощью Power Query](apache-hadoop-connect-excel-power-query.md).
* [Подключение к Azure HDInsight и выполнение запросов Apache Hive с помощью Средств Data Lake для Visual Studio](apache-hadoop-visual-studio-tools-get-started.md)