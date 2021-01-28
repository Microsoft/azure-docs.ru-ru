---
title: Политики Apache Hive в Apache Ranger — Azure HDInsight
description: Сведения о настройке политик Apache Ranger для Hive в службе Azure HDInsight с Корпоративным пакетом безопасности.
author: omidm1
ms.author: omidm
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive
ms.date: 11/27/2019
ms.openlocfilehash: 8ebc03d0847414730c51b899be4cf6586d064696
ms.sourcegitcommit: 2f9f306fa5224595fa5f8ec6af498a0df4de08a8
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/28/2021
ms.locfileid: "98932235"
---
# <a name="configure-apache-hive-policies-in-hdinsight-with-enterprise-security-package"></a>Настройка политик Apache Hive в HDInsight с Корпоративным пакетом безопасности

Узнайте, как настроить политики Apache Ranger для Apache Hive. В этой статье вы создадите две политики Ranger, чтобы ограничить доступ к таблице hivesampletable. Таблица hivesampletable поставляется с кластерами HDInsight. После настройки политик вы используете Excel и драйвер ODBC для подключения к таблицам Hive в HDInsight.

## <a name="prerequisites"></a>Предварительные требования

* Кластер HDInsight с Корпоративным пакетом безопасности. Ознакомьтесь со статьей [Настройка кластера HDInsight с корпоративным пакетом безопасности с помощью доменных служб Azure Active Directory](./apache-domain-joined-configure-using-azure-adds.md).
* Рабочая станция с Microsoft 365ными приложениями для предприятий, Office 2016, Office 2013 профессиональный плюс, Excel 2013 standalone или Office 2010 профессиональный плюс.

## <a name="connect-to-apache-ranger-admin-ui"></a>Подключение к пользовательскому интерфейсу администратора Apache Ranger
**Подключение к пользовательскому интерфейсу администратора Ranger**

1. В браузере перейдите к пользовательскому интерфейсу администратора Ranger, `https://CLUSTERNAME.azurehdinsight.net/Ranger/` где имя_кластера — имя кластера.

   > [!NOTE]  
   > Ranger использует учетные данные, отличающиеся от учетных данных кластера Apache Hadoop. Чтобы браузеры не использовали кэшированные учетные данные Hadoop, подключитесь к пользовательскому интерфейсу администратора Ranger в новом окне браузера в режиме InPrivate.

2. Войдите, используя имя пользователя и пароль домена администратора кластера.

    ![Домашняя страница HDInsight ESP Ranger](./media/apache-domain-joined-run-hive/hdinsight-domain-joined-ranger-home-page.png)

    В настоящее время Ranger работает только с Yarn и Hive.

## <a name="create-domain-users"></a>Создание пользователей домена

Сведения о создании учетных записей hiveruser1 и hiveuser2 см. в разделе [Создание кластера HDInsight с Корпоративным пакетом безопасности](apache-domain-joined-configure-using-azure-adds.md#create-an-hdinsight-cluster-with-esp). В этой статье используются две учетные записи пользователей.

## <a name="create-ranger-policies"></a>Создание политик Ranger

В этом разделе вы создадите две политики Ranger для доступа к таблице hivesampletable. Вам нужно будет предоставить разрешение select для разных наборов столбцов. Оба пользователя созданы при работе с разделом [Создание кластера HDInsight с Корпоративным пакетом безопасности](apache-domain-joined-configure-using-azure-adds.md#create-an-hdinsight-cluster-with-esp). В следующем разделе вы проверите две политики в Excel.

**Создание политик Ranger**

1. Откройте пользовательский интерфейс администратора Ranger. Ознакомьтесь с разделом "Подключение к пользовательскому интерфейсу администратора Apache Ranger".
2. Выберите **CLUSTERNAME_Hive** в разделе **Hive**. Отобразятся две предварительно настроенные политики.
3. Выберите **Добавить новую политику**, а затем введите следующие значения:

    |Свойство |Значение |
    |---|---|
    |Имя политики|Read-hivesampletable — все|
    |База данных Hive|значение по умолчанию|
    |таблица|hivesampletable|
    |Столбец Hive|*|
    |Выберите пользователя|hiveuser1|
    |Разрешения|select|

    ![Настройка политик Hive для Ranger в HDInsight ESP](./media/apache-domain-joined-run-hive/hdinsight-domain-joined-configure-ranger-policy.png).

    > [!NOTE]  
    > Если пользователь домена не указан в поле "Выберите пользователя", подождите несколько минут для синхронизации Ranger с AAD.

4. Щелкните **Добавить**, чтобы сохранить политику.

5. Повторите последние два шага, чтобы создать еще одну политику со следующими свойствами.

    |Свойство |Значение |
    |---|---|
    |Имя политики|Read-hivesampletable-devicemake|
    |База данных Hive|значение по умолчанию|
    |таблица|hivesampletable|
    |Столбец Hive|ClientID, devicemake|
    |Выберите пользователя|hiveuser2|
    |Разрешения|select|

## <a name="create-hive-odbc-data-source"></a>Создание источника данных Hive ODBC

Инструкции см. в разделе [Создание источника данных Hive ODBC](../hadoop/apache-hadoop-connect-excel-hive-odbc-driver.md).  

 | Свойство  |Описание |
 | --- | --- |
 | Имя базы данных-источника | Присвойте имя источнику данных |
 | Узел | Введите CLUSTERNAME.azurehdinsight.net. Например, myHDICluster.azurehdinsight.net |
 | Порт | Используйте **443**. (Этот порт был изменен с 563 на 443.) |
 | База данных | Использовать **значение по умолчанию**. |
 | Тип сервера Hive | Выберите **Hive Server 2**. |
 | Механизм | Выберите **Служба Azure HDInsight**. |
 | Путь HTTP | Оставьте пустым. |
 | Имя пользователя | Введите hiveuser1@contoso158.onmicrosoft.com. Обновите доменное имя, если оно отличается. |
 | Пароль | Введите пароль для hiveuser1. |

Щелкните **Проверка** перед сохранением источника данных.

## <a name="import-data-into-excel-from-hdinsight"></a>Импорт данных в Excel из службы HDInsight

В последнем разделе вы настроили две политики.  У hiveuser1 есть разрешение select на все столбцы, а у hiveuser2 — на два столбца. В этом разделе вы выполните олицетворение двух пользователей для импорта данных в Excel.

1. Откройте новую или существующую рабочую книгу в Excel.

1. На вкладке **Данные** перейдите к разделу **Получить данные** > **Из других источников** > **Из ODBC**, чтобы открыть окно **Из ODBC**.

    ![Мастер подключения к данным](./media/apache-domain-joined-run-hive/simbahiveodbc-excel-dataconnection1.png)

1. В раскрывающемся списке выберите имя источника данных, созданное в последнем разделе, и нажмите кнопку **ОК**.

1. При первом использовании откроется диалоговое окно **драйвера ODBC** . В меню слева выберите пункт **Windows** . Затем нажмите кнопку **Подключиться** , чтобы открыть окно **навигатора** .

1. Подождите открытие диалогового окна **Выбор базы данных и таблицы** . Это может занять несколько секунд.

1. Выберите **hivesampletable** и нажмите кнопку **Далее**.

1. Нажмите кнопку **Готово**.

1. В диалоговом окне **Импорт данных** можно изменить или указать запрос. Для этого выберите **Свойства**. Это может занять несколько секунд.

1. Перейдите на вкладку **Определение** . Текст команды:

    ```sql
    SELECT * FROM "HIVE"."default"."hivesampletable"`
    ```

   В соответствии с заданными политиками Ranger у hiveuser1 есть разрешение select на все столбцы.  Поэтому этот запрос работает с учетными данными учетными hiveuser1, но этот запрос не работает с учетными данными hiveuser2.

1. Нажмите кнопку **ОК** , чтобы закрыть диалоговое окно Свойства соединения.

1. Нажмите кнопку **ОК** , чтобы закрыть диалоговое окно **Импорт данных** .  

1. Введите пароль для hiveuser1 еще раз и нажмите кнопку **ОК**. Пройдет несколько секунд, прежде чем данные будут импортированы в Excel. По завершении вы увидите 11 столбцов данных.

Проверка второй политики (read-hivesampletable-devicemake), созданной в предыдущем разделе

1. Добавьте новый лист в Excel.
2. Выполните последнюю процедуру для импорта данных.  Единственное изменение заключается в том, что нужно использовать учетные данные hiveuser2, а не hiveuser1. При этом произойдет сбой, так как у hiveuser2 есть разрешение только на просмотр двух столбцов. Появится следующая ошибка:

    ```output
    [Microsoft][HiveODBC] (35) Error from Hive: error code: '40000' error message: 'Error while compiling statement: FAILED: HiveAccessControlException Permission denied: user [hiveuser2] does not have [SELECT] privilege on [default/hivesampletable/clientid,country ...]'.
    ```

3. Выполните ту же процедуру для импорта данных. На этот раз используйте учетные данные hiveuser2 и измените инструкцию FROM в предложении SELECT:

    ```sql
    SELECT * FROM "HIVE"."default"."hivesampletable"
    ```

    на:

    ```sql
    SELECT clientid, devicemake FROM "HIVE"."default"."hivesampletable"
    ```

    После этого должны отобразиться два столбца импортируемых данных.

## <a name="next-steps"></a>Дальнейшие действия

* Описание настройки кластера HDInsight с Корпоративным пакетом безопасности см. в статье [Настройка кластера HDInsight с Корпоративным пакетом безопасности с помощью доменных служб Azure Active Directory](./apache-domain-joined-configure-using-azure-adds.md).
* Сведения об управлении кластером HDInsight с помощью ESP см. в статье [Управление кластерами HDInsight с помощью Корпоративного пакета безопасности](apache-domain-joined-manage.md).
* Сведения о выполнении запросов Hive с помощью SSH в кластерах HDInsight с ESP см. в разделе [Проверка подлинности при использовании присоединенного к домену кластера HDInsight](../hdinsight-hadoop-linux-use-ssh-unix.md#authentication-domain-joined-hdinsight).
* Сведения о подключении к Hive с помощью Hive JDBC см. в статье [Отправка запросов в Apache Hive с помощью драйвера JDBC в HDInsight](../hadoop/apache-hadoop-connect-hive-jdbc-driver.md).
* Сведения о подключении Excel к Hadoop с помощью Hive ODBC см. в статье [Подключение Excel к Apache Hadoop с помощью драйвера Microsoft Hive ODBC в Azure HDInsight](../hadoop/apache-hadoop-connect-excel-hive-odbc-driver.md).
* Сведения о подключении Excel к Hadoop с помощью Power Query см. в [этой статье](../hadoop/apache-hadoop-connect-excel-power-query.md).