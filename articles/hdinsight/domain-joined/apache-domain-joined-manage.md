---
title: Управление кластерами Корпоративный пакет безопасности в Azure HDInsight
description: Узнайте, как управлять кластерами Azure HDInsight с помощью Корпоративный пакет безопасности.
ms.service: hdinsight
ms.topic: how-to
ms.date: 12/04/2019
ms.openlocfilehash: bc31c3d71590a6b8c0b324ffcb8c10129a9f8699
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104863247"
---
# <a name="manage-hdinsight-clusters-with-enterprise-security-package"></a>Управление кластерами HDInsight с помощью корпоративного пакета безопасности

Познакомьтесь с пользователями и ролями в корпоративном пакете безопасности (ESP) HDInsight, а также с тем, как управлять кластерами ESP.

## <a name="use-vscode-to-link-to-domain-joined-cluster"></a>Использование Visual Studio Code для связывания присоединенного к домену кластера

Вы можете связать обычный кластер с помощью управляемого имени пользователя Apache Ambari, а кластер безопасности Apache Hadoop — с помощью имени пользователя домена (например `user1@contoso.com`).

1. Откройте [Visual Studio Code](https://code.visualstudio.com/). Убедитесь, что расширение " [инструменты Hive" для Spark &](../hdinsight-for-vscode.md) установлено.

1. Выполните действия из раздела [связывание кластера](../hdinsight-for-vscode.md#link-a-cluster) для Visual Studio Code.

## <a name="use-intellij-to-link-to-domain-joined-cluster"></a>Использование IntelliJ для связывания присоединенного к домену кластера

Вы можете связать обычный кластер с помощью управляемого имени пользователя Ambari, а кластер безопасности — с помощью имени пользователя домена (например, `user1@contoso.com`).

1. Откройте IntelliJ IDEA. Убедитесь, что выполнены все [Предварительные условия](../spark/apache-spark-intellij-tool-plugin.md#prerequisites) .

1. Выполните действия из раздела [связывание кластера](../spark/apache-spark-intellij-tool-plugin.md#link-a-cluster) для IntelliJ.

## <a name="use-eclipse-to-link-to-domain-joined-cluster"></a>Использование Eclipse для связывания присоединенного к домену кластера

Вы можете связать обычный кластер с помощью управляемого имени пользователя Ambari, а кластер безопасности — с помощью имени пользователя домена (например, `user1@contoso.com`).

1. Откройте Eclipse. Убедитесь, что выполнены все [Предварительные условия](../spark/apache-spark-eclipse-tool-plugin.md#prerequisites) .

1. Выполните действия из раздела [связывание кластера](../spark/apache-spark-eclipse-tool-plugin.md#link-a-cluster) для Eclipse.

## <a name="access-the-clusters-with-enterprise-security-package"></a>Доступ к кластерам с помощью Корпоративный пакет безопасности

Пакет безопасности предприятия (прежнее название — HDInsight Premium) обеспечивает многопользовательский доступ к кластеру, где выполняется аутентификация Active Directory, а также авторизация с использованием Apache Ranger и ACL ADLS. Авторизация обеспечивает безопасное разделение пользователей, разрешая только привилегированным лицам доступ к данным на основе политик авторизации.

Обеспечение безопасности и изоляция пользователей важны для кластера HDInsight с пакетом безопасности предприятия. Чтобы удовлетворить эти требования, доступ SSH к кластеру с Корпоративный пакет безопасности поддерживается для локального пользователя, выбранного во время создания кластера, а также для пользователей, доступных в AAD-DS (т. е. Kerberos). В следующей таблице описаны способы доступа, рекомендуемые для каждого типа кластера:

|Рабочая нагрузка|Сценарий|Метод доступа|
|--------|--------|-------------|
|Apache Hadoop|Hive — интерактивные задания и запросы    |<ul><li>[Beeline](#beeline)</li><li>[Представление Hive](../hadoop/apache-hadoop-use-hive-ambari-view.md)</li><li>[ODBC/JDBC – Power BI](../hadoop/apache-hadoop-connect-hive-power-bi.md)</li><li>[Инструменты Visual Studio](../hadoop/apache-hadoop-visual-studio-tools-get-started.md)</li></ul>|
|Apache Spark|Интерактивные задания и запросы, интерактивные задания PySpark|<ul><li>[Beeline](#beeline)</li><li>[Zeppelin с Livy](../spark/apache-spark-zeppelin-notebook.md)</li><li>[Представление Hive](../hadoop/apache-hadoop-use-hive-ambari-view.md)</li><li>[ODBC/JDBC – Power BI](../hadoop/apache-hadoop-connect-hive-power-bi.md)</li><li>[Инструменты Visual Studio](../hadoop/apache-hadoop-visual-studio-tools-get-started.md)</li></ul>|
|Apache Spark|Сценарии пакетной службы — отправка Spark, PySpark|<ul><li>[Livy](../spark/apache-spark-livy-rest-interface.md)</li></ul>|
|Интерактивный запрос (LLAP)|Интерактивно|<ul><li>[Beeline](#beeline)</li><li>[Представление Hive](../hadoop/apache-hadoop-use-hive-ambari-view.md)</li><li>[ODBC/JDBC – Power BI](../hadoop/apache-hadoop-connect-hive-power-bi.md)</li><li>[Инструменты Visual Studio](../hadoop/apache-hadoop-visual-studio-tools-get-started.md)</li></ul>|
|Любой|Установка пользовательского приложения|<ul><li>[Элемент "Действия скрипта"](../hdinsight-hadoop-customize-cluster-linux.md)</li></ul>|

   > [!NOTE]  
   > В Корпоративном пакете безопасности записная книжка Jupyter не установлена или не поддерживается.

Использование стандартных API помогает обеспечить безопасность. Вы также получаете следующие преимущества:

- **Управление** — вы можете управлять кодом и заданиями автоматизации с помощью стандартных API — Livy, HS2 и т. д.
- **Аудит** . при использовании SSH нет способа провести аудит, который пользователи SSH применяют к кластеру. Это не так, когда задания создаются с помощью стандартных конечных точек, так как они будут выполняться в контексте пользователя.

### <a name="use-beeline"></a><a name="beeline"></a>Использование Beeline

Установите Beeline на компьютер подключитесь через общедоступный сегмент Интернета, используя следующие параметры:

```
- Connection string: -u 'jdbc:hive2://<clustername>.azurehdinsight.net:443/;ssl=true;transportMode=http;httpPath=/hive2'
- Cluster login name: -n admin
- Cluster login password -p 'password'
```

Если Beeline установлен локально и подключение устанавливается через виртуальную сеть Azure, используйте следующие параметры:

```
Connection string: -u 'jdbc:hive2://<headnode-FQDN>:10001/;transportMode=http'
```

Чтобы найти полное доменное имя головного узла, используйте сведения из руководства по получению полного доменного имени для узлов кластера.

## <a name="users-of-hdinsight-clusters-with-esp"></a>Пользователи кластеров HDInsight с корпоративным пакетом безопасности

При создании кластера HDInsight без ESP в нем создаются две учетные записи пользователя:

- **Администратор Ambari**: эта учетная запись называется также *Пользователь Hadoop* или *Пользователь HTTP*. Эту учетную запись можно использовать для входа в Ambari по адресу `https://CLUSTERNAME.azurehdinsight.net` . Его также можно использовать для выполнения запросов к представлениям Ambari, выполнения заданий через внешние средства (например, PowerShell, Templeton, Visual Studio) и проверки подлинности с помощью драйвера Hive ODBC и средств бизнес-аналитики (например, Excel, Power BI или Tableau).

В кластере HDInsight с корпоративным пакетом безопасности, кроме администратора Ambari, создаются еще три пользователя.

- **Администратор Ranger**: это локальная учетная запись администратора Apache Ranger. Он не является пользователем домена Active Directory. Эту учетную запись можно использовать для настройки политик, создания других пользователей-администраторов или делегированных администраторов (чтобы они могли управлять политиками). По умолчанию используется имя пользователя *admin* и такой же пароль, как для администратора Ambari. Этот пароль можно изменить на странице настроек в Ranger.

- **Пользователь домена и администратор кластера**: это пользователь домена Active Directory, который является администратором кластера Hadoop, а также служб Ambari и Ranger. Учетные данные этого пользователя необходимо указать во время создания кластера. Этот пользователь имеет следующие права.
    - Присоединение компьютеров к домену и помещение их в определенное подразделение, которое указывается во время создания кластера.
    - Создание субъектов-служб в определенном подразделении, которое указывается во время создания кластера.
    - Создание обратных записей DNS.

    Обратите внимание, что эти права есть и у других пользователей AD.

    Существует несколько конечных точек в кластере (например, Templeton), которые не управляются Ranger и, следовательно, не являются безопасными. Такие конечные точки закрываются для всех пользователей, за исключением пользователя домена и администратора кластера.

- **Обычный**: во время создания кластера можно указать несколько групп Active Directory. Пользователи в эти группах будут синхронизированы с Ranger и Ambari. Эти пользователи являются пользователями домена и будут иметь доступ только к конечным точкам, которые управляются службой Ranger (например, Hiveserver2). Для этих пользователей применяются все политики RBAC и правила аудита.

## <a name="roles-of-hdinsight-clusters-with-esp"></a>Роли кластеров HDInsight с корпоративным пакетом безопасности

Корпоративный пакет безопасности HDInsight предоставляет следующие роли:

- администратор кластера;
- оператор кластера;
- Администратор служб
- оператор службы;
- пользователь кластера.

**Просмотр разрешений для этих ролей**

1. Откройте пользовательский интерфейс управления Ambari.  См. раздел [Вход в пользовательский интерфейс управления Ambari](#open-the-ambari-management-ui).
2. В меню слева выберите **роли**.
3. Выберите синий вопросительный знак, чтобы просмотреть разрешения:

    :::image type="content" source="./media/apache-domain-joined-manage/hdinsight-domain-joined-roles-permissions.png" alt-text="Разрешения роли HDInsight с корпоративным пакетом безопасности" border="true":::

## <a name="open-the-ambari-management-ui"></a>Вход в пользовательский интерфейс управления Ambari

1. Перейдите в `https://CLUSTERNAME.azurehdinsight.net/` папку, где имя_кластера — имя кластера.
1. Войдите в Ambari с помощью имени пользователя домена администратора кластера и пароля.
1. Выберите раскрывающееся меню **администратора** в правом верхнем углу и выберите **Управление Ambari**.

    :::image type="content" source="./media/apache-domain-joined-manage/hdinsight-domain-joined-manage-ambari.png" alt-text="ESP HDInsight управление Ambari Apache" border="true":::

    Пользовательский интерфейс выглядит примерно так:

    :::image type="content" source="./media/apache-domain-joined-manage/hdinsight-domain-joined-ambari-management-ui.png" alt-text="Пользовательский интерфейс управления ESP HDInsight Apache Ambari" border="true":::

## <a name="list-the-domain-users-synchronized-from-your-active-directory"></a>Список пользователей домена, синхронизированных из Active Directory

1. Откройте пользовательский интерфейс управления Ambari.  См. раздел [Вход в пользовательский интерфейс управления Ambari](#open-the-ambari-management-ui).
2. В меню слева выберите **Пользователи**. Вы увидите всех пользователей, синхронизированных в кластер HDInsight из Active Directory.

    :::image type="content" source="./media/apache-domain-joined-manage/hdinsight-domain-joined-ambari-management-ui-users.png" alt-text="Список пользователей интерфейса управления Ambari для HDInsight с корпоративным пакетом безопасности" border="true":::

## <a name="list-the-domain-groups-synchronized-from-your-active-directory"></a>Список групп домена, синхронизированных из Active Directory

1. Откройте пользовательский интерфейс управления Ambari.  См. раздел [Вход в пользовательский интерфейс управления Ambari](#open-the-ambari-management-ui).
2. В меню слева выберите **группы**. Вы увидите все группы, синхронизированные в кластер HDInsight из Active Directory.

    :::image type="content" source="./media/apache-domain-joined-manage/hdinsight-domain-joined-ambari-management-ui-groups.png" alt-text="Список групп интерфейса управления Ambari для HDInsight с корпоративным пакетом безопасности" border="true":::

## <a name="configure-hive-views-permissions"></a>Настройка разрешений для представлений Hive

1. Откройте пользовательский интерфейс управления Ambari.  См. раздел [Вход в пользовательский интерфейс управления Ambari](#open-the-ambari-management-ui).
2. В меню слева выберите **представления**.
3. Выберите **Hive** , чтобы просмотреть сведения.

    :::image type="content" source="./media/apache-domain-joined-manage/hdinsight-domain-joined-ambari-management-ui-hive-views.png" alt-text="Представления Hive в интерфейсе управления Ambari для HDInsight с корпоративным пакетом безопасности" border="true":::

4. Щелкните ссылку **представление Hive** , чтобы настроить представления Hive.
5. Выполните прокрутку вниз до раздела **Разрешения**.

    :::image type="content" source="./media/apache-domain-joined-manage/hdinsight-domain-joined-ambari-management-ui-hive-views-permissions.png" alt-text="Настройка разрешений для представлений Hive в интерфейсе управления Ambari для HDInsight с ESP" border="true":::

6. Выберите **Добавить пользователя** или **Добавить группу**, а затем укажите пользователей или группы, которые могут использовать представления Hive.

## <a name="configure-users-for-the-roles"></a>Настройка ролей для пользователей

 Список ролей и соответствующих разрешений см. в разделе "Роли кластеров HDInsight с корпоративным пакетом безопасности".

1. Откройте пользовательский интерфейс управления Ambari.  См. раздел [Вход в пользовательский интерфейс управления Ambari](#open-the-ambari-management-ui).
2. В меню слева выберите **роли**.
3. Выберите **Добавить пользователя** или **Добавить группу** , чтобы назначить пользователей и группы различным ролям.

## <a name="next-steps"></a>Дальнейшие действия

- Описание настройки кластера HDInsight с Корпоративным пакетом безопасности см. в статье [Настройка кластера HDInsight с Корпоративным пакетом безопасности с помощью доменных служб Azure Active Directory](./apache-domain-joined-configure-using-azure-adds.md).
- Описание настройки политик Hive и выполнения запросов Hive см. в статье [Настройка политик Apache Hive в HDInsight с Корпоративным пакетом безопасности](apache-domain-joined-run-hive.md).