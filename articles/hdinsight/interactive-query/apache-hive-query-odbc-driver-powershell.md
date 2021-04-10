---
title: Отправка запросов в Apache Hive с помощью PowerShell и драйвера ODBC — Azure HDInsight
description: Используйте PowerShell и драйвер Microsoft Hive ODBC, чтобы запрашивать кластеры Apache Hive в Azure HDInsight.
keywords: hive,hive odbc,powershell
ms.service: hdinsight
ms.topic: tutorial
ms.date: 06/27/2019
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 34ee7eb25b1d106ea8bb53197d69ca5a9d528773
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104871611"
---
# <a name="tutorial-query-apache-hive-with-odbc-and-powershell"></a>Руководство. Отправка запросов в Apache Hive с помощью PowerShell и драйвера ODBC

Драйверы Microsoft ODBC обеспечивают гибкий способ взаимодействия с разными типами источников данных, включая Apache Hive. Вы можете написать код на языках сценариев, например PowerShell, использующим драйверы ODBC, чтобы открыть подключение к кластеру Hive, передать выбранный запрос и отобразить результаты.

В этом учебнике описано, как выполнять такие задачи:

> [!div class="checklist"]
> * Скачивание и установка драйвера Microsoft Hive ODBC.
> * Создание источника данных Apache Hive ODBC, связанного с кластером.
> * Запрос примера сведений из кластера с помощью PowerShell.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Перед началом работы с этим руководством необходимо иметь следующее:

* Кластер интерактивного запроса в HDInsight. Дополнительные сведения о создании кластера см. в статье [Приступая к работе с Hadoop в HDInsight](../hdinsight-hadoop-provision-linux-clusters.md). Выберите тип кластера **Interactive Query** (Интерактивный запрос).

## <a name="install-microsoft-hive-odbc-driver"></a>Установка драйвера Microsoft Hive ODBC

Скачайте и установите [Microsoft Hive ODBC Driver](https://www.microsoft.com/download/details.aspx?id=40886).

## <a name="create-apache-hive-odbc-data-source"></a>Создание источника данных Apache Hive ODBC

Ниже показано, как создать источник данных Apache Hive ODBC.

1. В Windows откройте **Пуск** > **Средства администрирования Windows** > **ODBC Data Sources (32-bit)/(64-bit)** (Источники данных ODBC (32-разрядная или 64-разрядная версия)).  Откроется окно **Администратор источников данных ODBC**.

    :::image type="content" source="./media/apache-hive-query-odbc-driver-powershell/hive-odbc-driver-dsn-setup.png " alt-text="Администратор источника данных ODBC" border="true":::

1. На вкладке **DSN пользователя** выберите **Добавить**, чтобы открыть окно **Создание нового источника данных**.

1. Выберите **Microsoft Hive ODBC Driver**, а затем — **Готово**, чтобы открыть окно **Microsoft Hive ODBC Driver DSN Setup** (Настройка DSN Microsoft Hive ODBC Driver).

1. Введите или выберите следующие значения:

   | Свойство | Описание |
   | --- | --- |
   |  Имя базы данных-источника |Присвойте имя источнику данных |
   |  Узлы |Введите `CLUSTERNAME.azurehdinsight.net`. Например `myHDICluster.azurehdinsight.net`. |
   |  Порт |Используйте **443**.|
   |  База данных |Используйте **значение по умолчанию**. |
   |  Механизм |Выберите **Windows Azure HDInsight Service**. |
   |  Имя пользователя |Введите имя пользователя HTTP кластера HDInsight. Имя пользователя по умолчанию — **admin**. |
   |  Пароль |Введите пароль пользователя кластера HDInsight. Установите флажок **Save Password (Encrypted)** (Сохранить пароль (зашифрованный)).|

1. (Необязательно) Выберите **Дополнительные параметры**.  

   | Параметр | Описание |
   | --- | --- |
   |  Использовать исходный запрос |При выборе этого параметра драйвер ODBC НЕ пытается преобразовать TSQL в HiveQL. Используйте этот параметр, только если полностью уверены в отправке действительных инструкций HiveQL. При подключении к серверу SQL Server или базе данных Azure SQL необходимо снять этот флажок. |
   |  Строки, загружаемые для каждого блока |При получении большого объема записей включение этого параметра может обеспечить оптимальную производительность. |
   |  Длина столбца строки по умолчанию, длина столбца двоичного кода, масштаб столбца десятичных значений |Длина и точность типа данных может повлиять на способ выведения данных. Это приведет к возврату недопустимой информации из-за потери точности и усечения. |

    :::image type="content" source="./media/apache-hive-query-odbc-driver-powershell/odbc-data-source-advanced-options.png " alt-text="Дополнительные параметры конфигурации DSN" border="true":::

1. Щелкните **Тест** для проверки источника данных. При правильной настройке источника данных в результате теста отображается **Успешно**.  

1. Нажмите кнопку **ОК**, чтобы закрыть окно тестов.  

1. Нажмите кнопку **ОК**, чтобы закрыть окно **Microsoft Hive ODBC Driver DSN Setup** (Настройка DSN Microsoft Hive ODBC Driver).  

1. Нажмите кнопку **ОК**, чтобы закрыть окно **Администратор источников данных ODBC**.  

## <a name="query-data-with-powershell"></a>Запрос данных с помощью PowerShell

Следующий сценарий PowerShell является функцией ODBC для запроса к кластеру Hive.

```powershell
function Get-ODBC-Data {

   param(
   [string]$query=$(throw 'query is required.'),
   [string]$dsn,  
   [PSCredential] $cred = (Get-Credential)  
   )

   $conn = New-Object System.Data.Odbc.OdbcConnection
   $uname = $cred.UserName

   $pswd = (New-Object System.Net.NetworkCredential -ArgumentList "", $cred.Password).Password
   $conn.ConnectionString = "DSN=$dsn;Uid=$uname;Pwd=$pswd;"
   $conn.open()
   $cmd = New-object System.Data.Odbc.OdbcCommand($query,$conn)

   $ds = New-Object system.Data.DataSet

   (New-Object system.Data.odbc.odbcDataAdapter($cmd)).fill($ds) #| out-null
   $conn.close()
   $ds.Tables
}
```

В следующем фрагменте кода используется указанная выше функция для выполнения запроса к кластеру Interactive Query, созданному в начале этого учебника. Замените `DATASOURCENAME`**именем источника данных**, которое вы указали на экране **Microsoft Hive ODBC Driver DSN Setup** (Настройка DSN Microsoft Hive ODBC Driver). При запросе учетных данных введите имя пользователя и пароль, которые вы вводили в разделе **Имя пользователя для входа в кластер** и **Пароль для входа в кластер**, когда создавали кластер.

```powershell

$dsn = "DATASOURCENAME"

$query = "select count(distinct clientid) AS total_clients from hivesampletable"

Get-ODBC-Data -query $query -dsn $dsn
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Ставшие ненужными группу ресурсов, кластер HDInsight и учетную запись хранения можно удалить. Для этого выберите группу ресурсов, в которой был создан кластер, и нажмите кнопку **Удалить**.

## <a name="next-steps"></a>Дальнейшие действия

В этом учебнике вы узнали, как с помощью драйвера Microsoft Hive ODBC и PowerShell получить данные из кластера Interactive Query Azure HDInsight.

> [!div class="nextstepaction"]
> [Connect Excel to Apache Hadoop in Azure HDInsight with the Microsoft Hive ODBC driver](../hadoop/apache-hadoop-connect-excel-hive-odbc-driver.md) (Подключение Excel к Apache Hadoop в Azure HDInsight с помощью драйвера Microsoft Hive ODBC)
