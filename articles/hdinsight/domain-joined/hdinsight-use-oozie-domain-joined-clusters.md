---
title: Рабочие процессы Apache Oozie и Корпоративный пакет безопасности — Azure HDInsight
description: Защита рабочих процессов Apache Oozie с помощью Корпоративного пакета безопасности для Azure HDInsight Узнайте, как определить рабочий процесс и отправить задание для Oozie.
ms.service: hdinsight
ms.topic: conceptual
ms.custom: seoapr2020
ms.date: 05/14/2020
ms.openlocfilehash: dc20a258028bb76351ae70362234e7c4c8c741a5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101699408"
---
# <a name="run-apache-oozie-in-azure-hdinsight-clusters-with-enterprise-security-package"></a>Выполнение Apache Oozie в кластерах Azure HDInsight с Корпоративным пакетом безопасности

Apache Oozie — это система рабочих процессов и координации, управляющая заданиями Apache Hadoop. Служба Oozie интегрирована со стеком Hadoop и поддерживает следующие задания:

- Apache MapReduce
- Apache Pig
- Apache Hive
- Apache Sqoop

Вы также можете использовать Oozie для планирования системных заданий, например Java-программ и сценариев оболочки.

## <a name="prerequisite"></a>Предварительные требования

Кластер Azure HDInsight Hadoop с Корпоративным пакетом безопасности (ESP). Ознакомьтесь со статьей [Настройка кластера HDInsight с корпоративным пакетом безопасности с помощью доменных служб Azure Active Directory](./apache-domain-joined-configure-using-azure-adds.md).

> [!NOTE]  
> Подробные инструкции по использованию Oozie в кластерах без пакета ESP см. в статье [Использование рабочих процессов Apache Oozie в Azure HDInsight под управлением Linux](../hdinsight-use-oozie-linux-mac.md).

## <a name="connect-to-an-esp-cluster"></a>Подключение к кластеру ESP

Дополнительные сведения о Secure Shell (SSH) см. в статье [Подключение к HDInsight (Hadoop) с помощью SSH](../hdinsight-hadoop-linux-use-ssh-unix.md).

1. Подключитесь к кластеру HDInsight с помощью протокола SSH:

    ```bash
    ssh [DomainUserName]@<clustername>-ssh.azurehdinsight.net
    ```

1. Чтобы проверить успешность аутентификации Kerberos, используйте команду `klist`. Если она не пройдена, запустите проверку подлинности Kerberos с помощью команды `kinit`.

1. Войдите в шлюз HDInsight, чтобы зарегистрировать токен OAuth, необходимый для доступа к Azure Data Lake Store:

    ```bash
    curl -I -u [DomainUserName@Domain.com]:[DomainUserPassword] https://<clustername>.azurehdinsight.net
    ```

    Код отклика состояния **200 ОК** указывает на успешную регистрацию. Проверьте имя пользователя и пароль, если получен ответ о неуспешной авторизации (например, 401).

## <a name="define-the-workflow"></a>Определение рабочего процесса

Определения рабочих процессов Oozie записываются на языке определения процессов Apache Hadoop (hPDL). hPDL — это язык определения процессов XML. Для определения рабочего процесса выполните следующие действия:

1. Настройка рабочей области пользователя домена:

   ```bash
   hdfs dfs -mkdir /user/<DomainUser>
   cd /home/<DomainUserPath>
   cp /usr/hdp/<ClusterVersion>/oozie/doc/oozie-examples.tar.gz .
   tar -xvf oozie-examples.tar.gz
   hdfs dfs -put examples /user/<DomainUser>/
   ```

   Замените `DomainUser` именем пользователя домена.
   Замените `DomainUserPath` путем к домашнему каталогу для пользователя домена.
   Замените `ClusterVersion` версией платформы данных, используемой вашим кластером.

2. Для создания и открытия файла выполните следующий запрос:

   ```bash
   nano workflow.xml
   ```

3. Открыв редактор nano, введите следующий код XML в качестве содержимого файла:

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <workflow-app xmlns="uri:oozie:workflow:0.4" name="map-reduce-wf">
       <credentials>
          <credential name="metastore_token" type="hcat">
             <property>
                <name>hcat.metastore.uri</name>
                <value>thrift://<active-headnode-name>-<clustername>.<Domain>.com:9083</value>
             </property>
             <property>
                <name>hcat.metastore.principal</name>
                <value>hive/_HOST@<Domain>.COM</value>
             </property>
          </credential>
          <credential name="hs2-creds" type="hive2">
             <property>
                <name>hive2.server.principal</name>
                <value>${jdbcPrincipal}</value>
             </property>
             <property>
                <name>hive2.jdbc.url</name>
                <value>${jdbcURL}</value>
             </property>
          </credential>
       </credentials>
       <start to="mr-test" />
       <action name="mr-test">
          <map-reduce>
             <job-tracker>${jobTracker}</job-tracker>
             <name-node>${nameNode}</name-node>
             <prepare>
                <delete path="${nameNode}/user/${wf:user()}/examples/output-data/mrresult" />
             </prepare>
             <configuration>
                <property>
                   <name>mapred.job.queue.name</name>
                   <value>${queueName}</value>
                </property>
                <property>
                   <name>mapred.mapper.class</name>
                   <value>org.apache.oozie.example.SampleMapper</value>
                </property>
                <property>
                   <name>mapred.reducer.class</name>
                   <value>org.apache.oozie.example.SampleReducer</value>
                </property>
                <property>
                   <name>mapred.map.tasks</name>
                   <value>1</value>
                </property>
                <property>
                   <name>mapred.input.dir</name>
                   <value>/user/${wf:user()}/${examplesRoot}/input-data/text</value>
                </property>
                <property>
                   <name>mapred.output.dir</name>
                   <value>/user/${wf:user()}/${examplesRoot}/output-data/mrresult</value>
                </property>
             </configuration>
          </map-reduce>
          <ok to="myHive2" />
          <error to="fail" />
       </action>
       <action name="myHive2" cred="hs2-creds">
          <hive2 xmlns="uri:oozie:hive2-action:0.2">
             <job-tracker>${jobTracker}</job-tracker>
             <name-node>${nameNode}</name-node>
             <jdbc-url>${jdbcURL}</jdbc-url>
             <script>${hiveScript2}</script>
             <param>hiveOutputDirectory2=${hiveOutputDirectory2}</param>
          </hive2>
          <ok to="myHive" />
          <error to="fail" />
       </action>
       <action name="myHive" cred="metastore_token">
          <hive xmlns="uri:oozie:hive-action:0.2">
             <job-tracker>${jobTracker}</job-tracker>
             <name-node>${nameNode}</name-node>
             <configuration>
                <property>
                   <name>mapred.job.queue.name</name>
                   <value>${queueName}</value>
                </property>
             </configuration>
             <script>${hiveScript1}</script>
             <param>hiveOutputDirectory1=${hiveOutputDirectory1}</param>
          </hive>
          <ok to="end" />
          <error to="fail" />
       </action>
       <kill name="fail">
          <message>Oozie job failed, error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
       </kill>
       <end name="end" />
    </workflow-app>
    ```

4. Замените `clustername` именем кластера.

5. Чтобы сохранить файл, нажмите **Ctrl+X**. Нажмите клавишу **Y**. Затем нажмите клавишу **ВВОД**.

    Рабочий процесс состоит из двух частей:

   - **Учетные данные.** Этот раздел принимает учетные данные, которые используются для проверки подлинности действий Oozie:

     В этом примере используется проверка подлинности для действий Hive. Дополнительные сведения см. в статье о [проверке подлинности действий](https://oozie.apache.org/docs/4.2.0/DG_ActionAuthentication.html).

     Служба учетных данных позволяет действиям Oozie олицетворять пользователя для доступа к службам Hadoop.

   - **Действие.** В этом разделе содержатся три действия: map-reduce, действия сервера Hive 2 и сервера Hive 1:

     - Действие map-reduce запускает пример из пакета Oozie для выполнения операции map-reduce, в результате которой вы получаете общее количество слов.

     - Действия сервера Hive 2 и сервера Hive 1 выполняют запрос к примеру таблицы Hive, поставляемой с кластером HDInsight.

     Действия Hive используют учетные данные, определенные в разделе учетных данных, для проверки подлинности с помощью ключевого слова `cred` в элементе действия.

6. Выполните следующую команду, чтобы скопировать файл `workflow.xml` в `/user/<domainuser>/examples/apps/map-reduce/workflow.xml`.

    ```bash
    hdfs dfs -put workflow.xml /user/<domainuser>/examples/apps/map-reduce/workflow.xml
    ```

7. Замените `domainuser` на свое имя пользователя для домена.

## <a name="define-the-properties-file-for-the-oozie-job"></a>Определение файла свойств для задания Oozie

1. Для создания и изменения нового файла свойств задания используйте следующий оператор:

    ```bash
    nano job.properties
    ```

2. Открыв редактор nano, используйте следующий код XML в качестве содержимого файла:

   ```bash
   nameNode=adl://home
   jobTracker=headnodehost:8050
   queueName=default
   examplesRoot=examples
   oozie.wf.application.path=${nameNode}/user/[domainuser]/examples/apps/map-reduce/workflow.xml
   hiveScript1=${nameNode}/user/${user.name}/countrowshive1.hql
   hiveScript2=${nameNode}/user/${user.name}/countrowshive2.hql
   oozie.use.system.libpath=true
   user.name=[domainuser]
   jdbcPrincipal=hive/<active-headnode-name>.<Domain>.com@<Domain>.COM
   jdbcURL=[jdbcurlvalue]
   hiveOutputDirectory1=${nameNode}/user/${user.name}/hiveresult1
   hiveOutputDirectory2=${nameNode}/user/${user.name}/hiveresult2
   ```

   - Используйте URI `adl://home` для свойства `nameNode`, если в качестве основного хранилища кластера используется Azure Data Lake Storage 1-го поколения. Если вы используете хранилище BLOB-объектов, укажите `wasb://home`. Если вы используете Azure Data Lake Storage 2-го поколения, укажите `abfs://home`.
   - Замените `domainuser` на свое имя пользователя для домена.
   - Замените `ClusterShortName` коротким именем кластера. Например, если имя кластера — sechadoopcontoso.azurehdisnight.net (https:// *[пример ссылки]* ), `clustershortname` представляет собой первые шесть букв имени кластера: **sechad**.
   - Замените `jdbcurlvalue` на URL-адрес JDBC из файла конфигурации Hive. Например, jdbc:hive2://headnodehost:10001/;transportMode=http.
   - Чтобы сохранить файл, нажмите Ctrl+X, введите `Y`, а затем нажмите клавишу **ВВОД**.

   Этот файл свойств должен присутствовать локально при выполнении заданий Oozie.

## <a name="create-custom-hive-scripts-for-oozie-jobs"></a>Создание пользовательских скриптов Hive для заданий Oozie

Вы можете создать два скрипта Hive для сервера Hive 1 и Hive 2, как показано в следующих разделах.

### <a name="hive-server-1-file"></a>Файл сервера Hive 1

1. Создайте и измените файл для действий сервера Hive 1:

    ```bash
    nano countrowshive1.hql
    ```

2. Создайте скрипт:

    ```sql
    INSERT OVERWRITE DIRECTORY '${hiveOutputDirectory1}'
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
    select devicemake from hivesampletable limit 2;
    ```

3. Сохраните файл в распределенную файловую систему Apache Hadoop (HDFS):

    ```bash
    hdfs dfs -put countrowshive1.hql countrowshive1.hql
    ```

### <a name="hive-server-2-file"></a>Файл сервера Hive 2

1. Создайте и измените поле для действий сервера Hive 2:

    ```bash
    nano countrowshive2.hql
    ```

2. Создайте скрипт:

    ```sql
    INSERT OVERWRITE DIRECTORY '${hiveOutputDirectory1}' 
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' 
    select devicemodel from hivesampletable limit 2;
    ```

3. Сохраните файл в HDFS:

    ```bash
    hdfs dfs -put countrowshive2.hql countrowshive2.hql
    ```

## <a name="submit-oozie-jobs"></a>Отправка заданий Oozie

Отправка заданий Oozie для кластеров ESP аналогична отправке заданий Oozie в кластерах без пакета ESP.

Дополнительные сведения см. в статье [Использование Oozie с Hadoop для определения и запуска рабочих процессов в Azure HDInsight под управлением Linux](../hdinsight-use-oozie-linux-mac.md).

## <a name="results-from-an-oozie-job-submission"></a>Результаты отправки заданий Oozie

Задания Oozie выполняются от имени пользователя. Поэтому в журналах аудита Apache Hadoop YARN и Apache Ranger задания отображаются как выполненные олицетворенным пользователем. Выходные данные интерфейса командной строки для заданий Oozie выглядят аналогично следующему коду:

```output
Job ID : 0000015-180626011240801-oozie-oozi-W
------------------------------------------------------------------------------------------------
Workflow Name : map-reduce-wf
App Path      : adl://home/user/alicetest/examples/apps/map-reduce/wf.xml
Status        : SUCCEEDED
Run           : 0
User          : alicetest
Group         : -
Created       : 2018-06-26 19:25 GMT
Started       : 2018-06-26 19:25 GMT
Last Modified : 2018-06-26 19:30 GMT
Ended         : 2018-06-26 19:30 GMT
CoordAction ID: -

Actions
------------------------------------------------------------------------------------------------
ID                        Status    Ext ID            ExtStatus                 ErrCode
------------------------------------------------------------------------------------------------
0000015-180626011240801-oozie-oozi-W@:start:    OK    -                         OK             -
------------------------------------------------------------------------------------------------
0000015-180626011240801-oozie-oozi-W@mr-test    OK    job_1529975666160_0051    SUCCEEDED      -
------------------------------------------------------------------------------------------------
0000015-180626011240801-oozie-oozi-W@myHive2    OK    job_1529975666160_0053    SUCCEEDED      -
------------------------------------------------------------------------------------------------
0000015-180626011240801-oozie-oozi-W@myHive    OK     job_1529975666160_0055    SUCCEEDED      -
------------------------------------------------------------------------------------------------
0000015-180626011240801-oozie-oozi-W@end       OK     -                         OK             -
-----------------------------------------------------------------------------------------------
```

Журналы аудита Ranger для действий сервера Hive 2 показывают, что задания Oozie выполняли действие от имени пользователя. Представления Ranger и YARN доступны только для администратора кластера.

## <a name="configure-user-authorization-in-oozie"></a>Настройка авторизации пользователей в Oozie

В Oozie уже есть конфигурация авторизации пользователей, которая может запретить пользователям останавливать или удалять задания другого пользователя. Чтобы включить эту конфигурацию, задайте для параметра `oozie.service.AuthorizationService.security.enabled` значение `true`. 

Дополнительные сведения см. в статье об [установке и настройке Apache Oozie](https://oozie.apache.org/docs/3.2.0-incubating/AG_Install.html).

Для таких компонентов, как сервер Hive 1, где подключаемый модуль Ranger недоступен или не поддерживается, возможна только недетализированная авторизация HDFS. Авторизация с высокой степенью детализации доступна только через подключаемые модули Ranger.

## <a name="get-the-oozie-web-ui"></a>Получение веб-интерфейса Oozie

Веб-интерфейс Oozie позволяет получить информацию о состоянии задания Oozie в кластере. Чтобы получить веб-интерфейс, в кластерах ESP выполните действия ниже:

1. Добавить [граничный узел](../hdinsight-apps-use-edge-node.md) и включить [проверку подлинности SSH Kerberos](../hdinsight-hadoop-linux-use-ssh-unix.md).

2. Следовать инструкциям в [веб-интерфейсе Oozie](../hdinsight-use-oozie-linux-mac.md), чтобы включить туннелирование SSH для граничного узла и получить доступ к пользовательскому веб-интерфейсу.

## <a name="next-steps"></a>Дальнейшие действия

- [Использование Oozie с Hadoop для определения и запуска рабочих процессов в Azure HDInsight под управлением Linux](../hdinsight-use-oozie-linux-mac.md).
- [Проверка подлинности при использовании присоединенного к домену кластера HDInsight](../hdinsight-hadoop-linux-use-ssh-unix.md#authentication-domain-joined-hdinsight).
