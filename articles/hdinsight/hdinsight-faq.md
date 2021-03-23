---
title: Часто задаваемые вопросы об Azure HDInsight
description: Часто задаваемые вопросы о HDInsight
keywords: часто задаваемые вопросы
author: Ramakoni1
ms.author: ramakoni
ms.service: hdinsight
ms.custom: hdinsightactive,seoapr2020
ms.topic: conceptual
ms.date: 11/20/2019
ms.openlocfilehash: 47f0ea34fa650371d39252ce6b0ee6a15f3124d6
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104871186"
---
# <a name="azure-hdinsight-frequently-asked-questions"></a>Azure HDInsight. Часто задаваемые вопросы

В этой статье содержатся ответы на некоторые наиболее распространенные вопросы о том, как запустить [Azure HDInsight](https://azure.microsoft.com/services/hdinsight/).

## <a name="creating-or-deleting-hdinsight-clusters"></a>Создание или удаление кластеров HDInsight

### <a name="how-do-i-provision-an-hdinsight-cluster"></a>Разделы справки подготавливать кластер HDInsight?

Чтобы просмотреть типы кластеров HDInsight и методы подготовки, см. раздел [Настройка кластеров в hdinsight с помощью Apache Hadoop, Apache Spark, Apache Kafka и других](./hdinsight-hadoop-provision-linux-clusters.md).

### <a name="how-do-i-delete-an-existing-hdinsight-cluster"></a>Разделы справки удалить существующий кластер HDInsight?

Дополнительные сведения об удалении кластера, когда он больше не используется, см. в разделе [Удаление кластера HDInsight](hdinsight-delete-cluster.md).

Попробуйте оставить не менее 30 – 60 минут между операциями создания и удаления. В противном случае операция может завершиться со следующим сообщением об ошибке:

``Conflict (HTTP Status Code: 409) error when attempting to delete a cluster immediately after creation of a cluster. If you encounter this error, wait until the newly created cluster is in operational state before attempting to delete it.``

### <a name="how-do-i-select-the-correct-number-of-cores-or-nodes-for-my-workload"></a>Разделы справки выбрать правильное количество ядер или узлов для рабочей нагрузки?

Соответствующее количество ядер и другие параметры конфигурации зависят от различных факторов.

Дополнительные сведения см. в статье [планирование ресурсов для кластеров HDInsight](./hdinsight-capacity-planning.md).

### <a name="what-are-the-various-types-of-nodes-in-an-hdinsight-cluster"></a>Каковы различные типы узлов в кластере HDInsight?

См. раздел [типы ресурсов в кластерах Azure HDInsight](hdinsight-virtual-network-architecture.md#resource-types-in-azure-hdinsight-clusters).

### <a name="what-are-the-best-practices-for-creating-large-hdinsight-clusters"></a>Каковы рекомендации по созданию больших кластеров HDInsight?

1. Рекомендуется настроить кластеры HDInsight с [пользовательской базой данных Ambari DB](./hdinsight-custom-ambari-db.md) , чтобы повысить масштабируемость кластера.
2. Используйте [Azure Data Lake Storage 2-го поколения](./hdinsight-hadoop-use-data-lake-storage-gen2.md) для создания кластеров HDInsight, чтобы воспользоваться преимуществами более высокой пропускной способности и другими характеристиками производительности Azure Data Lake Storage 2-го поколения.
3. Головных узлах должно быть достаточно большим, чтобы разместить на этих узлах несколько главных служб.
4. Некоторые определенные рабочие нагрузки, такие как интерактивный запрос, также требуют больших узлов Zookeeper. Рекомендуем использовать не менее 8 основных виртуальных машин.
5. В случае Hive и Spark используйте [внешние хранилище метаданных Hive](./hdinsight-use-external-metadata-stores.md).

## <a name="individual-components"></a>Отдельные компоненты

### <a name="can-i-install-additional-components-on-my-cluster"></a>Можно ли установить дополнительные компоненты в кластер?

Да. Чтобы установить дополнительные компоненты или настроить конфигурацию кластера, используйте:

- Скрипты во время или после создания. Скрипты вызываются с помощью [действия скрипта](./hdinsight-hadoop-customize-cluster-linux.md). Действие скрипта — это параметр конфигурации, который можно использовать из портал Azure, командлетов HDInsight Windows PowerShell или пакета SDK для HDInsight .NET. Этот параметр конфигурации можно использовать из портал Azure, из командлетов HDInsight Windows PowerShell или пакета SDK для HDInsight .NET.

- [Платформа приложений HDInsight](https://azure.microsoft.com/services/hdinsight/partner-ecosystem/) для установки приложений.

Список поддерживаемых компонентов см [. в статье что такое Apache Hadoop компоненты и версии, доступные в HDInsight?](./hdinsight-component-versioning.md)

### <a name="can-i-upgrade-the-individual-components-that-are-pre-installed-on-the-cluster"></a>Можно ли обновить отдельные компоненты, предварительно установленные в кластере?

При обновлении встроенных компонентов или приложений, предварительно установленных в кластере, Результирующая конфигурация не будет поддерживаться корпорацией Майкрософт. Эти конфигурации системы не проверялись корпорацией Майкрософт. Попробуйте использовать другую версию кластера HDInsight, которая уже может иметь обновленную версию компонента, предварительно установленную.

Например, обновление Hive в качестве отдельного компонента не поддерживается. HDInsight — это управляемая служба, и многие службы интегрированы с Ambari Server и тестируются. При обновлении куста в собственном коде индексированные двоичные файлы других компонентов изменяются, и в кластере могут возникнуть проблемы с интеграцией компонентов.

### <a name="can-spark-and-kafka-run-on-the-same-hdinsight-cluster"></a>Можно ли запускать Spark и Kafka в одном кластере HDInsight?

Нет, невозможно запустить Apache Kafka и Apache Spark в том же кластере HDInsight. Создайте отдельные кластеры для Kafka и Spark, чтобы избежать проблем с состязанием на ресурсы.

### <a name="how-do-i-change-timezone-in-ambari"></a>Разделы справки изменить часовой пояс в Ambari?

1. Откройте веб-интерфейс Ambari в `https://CLUSTERNAME.azurehdinsight.net` , где имя_кластера — это имя кластера.
2. В правом верхнем углу выберите Admin | Параметры. 

   :::image type="content" source="media/hdinsight-faq/ambari-settings.png" alt-text="Параметры Ambari":::

3. В окне Параметры пользователя выберите новый часовой пояс в раскрывающемся списке Timezone (часовой пояс) и нажмите кнопку Сохранить.

   :::image type="content" source="media/hdinsight-faq/ambari-user-settings.png" alt-text="Параметры пользователя Ambari":::

## <a name="metastore"></a>Хранилище мета-данных

### <a name="how-can-i-migrate-from-the-existing-metastore-to-azure-sql-database"></a>Как выполнить миграцию из существующей хранилище метаданных в базу данных SQL Azure? 

Сведения о миграции из SQL Server в базу данных SQL Azure см. в разделе [учебник. перенос SQL Server в отдельную базу данных или базу данных в составе пула в базе данных SQL Azure в автономном режиме с помощью DMS](../dms/tutorial-sql-server-to-azure-sql.md).

### <a name="is-the-hive-metastore-deleted-when-the-cluster-is-deleted"></a>Удаляется ли хранилище метаданных Hive при удалении кластера?

Это зависит от типа хранилище метаданных, для которого настроен кластер.

Для хранилище метаданных по умолчанию: хранилище метаданных по умолчанию является частью жизненного цикла кластера. При удалении кластера соответствующее хранилище метаданных и метаданные также удаляются.

Для настраиваемого хранилище метаданных: жизненный цикл хранилище метаданных не привязан к жизненному циклу кластера. Таким образом, можно создавать и удалять кластеры без потери метаданных. Метаданные, такие как схемы Hive, сохраняются даже после удаления и повторного создания кластера HDInsight.

Дополнительную информацию см. в статье [Использование внешних хранилищ метаданных в Azure HDInsight](hdinsight-use-external-metadata-stores.md).

### <a name="does-migrating-a-hive-metastore-also-migrate-the-default-policies-of-the-ranger-database"></a>Миграция хранилище метаданных Hive также приводит к переносу политик по умолчанию базы данных Ranger?

Нет, определение политики находится в базе данных Ranger, поэтому миграция базы данных Ranger перенесет ее политику.

### <a name="can-you-migrate-a-hive-metastore-from-an-enterprise-security-package-esp-cluster-to-a-non-esp-cluster-and-the-other-way-around"></a>Можно ли перенести хранилище метаданных Hive из кластера Корпоративный пакет безопасности (ESP) в кластер, отличный от ESP, и наоборот?

Да, можно перенести хранилище метаданных Hive из ESP в кластер, не использующий ESP.

### <a name="how-can-i-estimate-the-size-of-a-hive-metastore-database"></a>Как оценить размер базы данных хранилище метаданных Hive?

Хранилище метаданных Hive используется для хранения метаданных для источников данных, используемых сервером Hive. Требования к размеру частично зависят от количества и сложности источников данных Hive. Предварительная оценка этих элементов невозможна. Как описано в [хранилище метаданных Hive рекомендациях](hdinsight-use-external-metadata-stores.md#hive-metastore-guidelines), можно начать с уровня S2. Уровень составляет 50 DTU и 250 ГБ хранилища, и если вы видите узкие места, увеличьте масштаб базы данных.

### <a name="do-you-support-any-other-database-other-than-azure-sql-database-as-an-external-metastore"></a>Поддерживается ли любая другая база данных, кроме базы данных SQL Azure, как внешняя хранилище метаданных?

Нет, корпорация Майкрософт поддерживает только базу данных SQL Azure в качестве внешней пользовательской хранилище метаданных.

### <a name="can-i-share-a-metastore-across-multiple-clusters"></a>Можно ли совместно использовать хранилище метаданных в нескольких кластерах?

Да, вы можете совместно использовать пользовательские хранилище метаданных в нескольких кластерах, если они используют одну и ту же версию HDInsight.

## <a name="connectivity-and-virtual-networks"></a>Подключение и виртуальные сети  

### <a name="what-are-the-implications-of-blocking-ports-22-and-23-on-my-network"></a>Каковы последствия блокировки портов 22 и 23 в моей сети?

Если вы блокируете порты 22 и 23, у вас нет доступа по протоколу SSH к кластеру. Эти порты не используются службой HDInsight.

Дополнительные сведения см. в следующих документах:

- [Порты, используемые службами Apache Hadoop в HDInsight](./hdinsight-hadoop-port-settings-for-services.md)

- [Защита входящего трафика в кластерах HDInsight в виртуальной сети с помощью частной конечной точки](https://azure.microsoft.com/blog/secure-incoming-traffic-to-hdinsight-clusters-in-a-vnet-with-private-endpoint/)

- [IP-адреса управления HDInsight](./hdinsight-management-ip-addresses.md)

### <a name="can-i-deploy-an-additional-virtual-machine-within-the-same-subnet-as-an-hdinsight-cluster"></a>Можно ли развернуть дополнительную виртуальную машину в той же подсети, что и кластер HDInsight?

Да, можно развернуть дополнительную виртуальную машину в той же подсети, что и кластер HDInsight. Возможны следующие конфигурации:

- Граничные узлы. Вы можете добавить в кластер другой пограничной узел, как описано в разделе [использование пустых граничных узлов в кластерах Apache Hadoop в HDInsight](hdinsight-apps-use-edge-node.md).

- Автономные узлы. Вы можете добавить автономную виртуальную машину в ту же подсеть и получить доступ к кластеру из этой виртуальной машины с помощью закрытой конечной точки `https://<CLUSTERNAME>-int.azurehdinsight.net` . Дополнительные сведения см. в разделе [Управление сетевым трафиком](./control-network-traffic.md).

### <a name="should-i-store-data-on-the-local-disk-of-an-edge-node"></a>Следует ли хранить данные на локальном диске пограничных узлов?

Нет, хранение данных на локальном диске не является хорошей идеей. В случае сбоя узла все хранящиеся локально данные будут потеряны. Мы рекомендуем хранить данные в Azure Data Lake Storage 2-го поколения или хранилище BLOB-объектов Azure либо путем подключения общей папки службы файлов Azure для хранения данных.


### <a name="can-i-add-an-existing-hdinsight-cluster-to-another-virtual-network"></a>Можно ли добавить существующий кластер HDInsight в другую виртуальную сеть?

Нет, Вы не можете. Виртуальная сеть должна быть указана во время подготовки. Если во время подготовки не указана виртуальная сеть, то при развертывании создается внутренняя сеть, недоступная извне. Дополнительные сведения см. [в статье Добавление HDInsight в существующую виртуальную сеть](hdinsight-plan-virtual-network-deployment.md#existingvnet).

## <a name="security-and-certificates"></a>Безопасность и сертификаты

### <a name="what-are-the-recommendations-for-malware-protection-on-azure-hdinsight-clusters"></a>Каковы рекомендации по защите от вредоносных программ в кластерах Azure HDInsight?

Сведения о защите от вредоносных программ см. в [статье антивредоносное по Майкрософт для облачных служб и виртуальных машин Azure](../security/fundamentals/antimalware.md).

### <a name="how-do-i-create-a-keytab-for-an-hdinsight-esp-cluster"></a>Разделы справки создать keytab для кластера HDInsight ESP?

Создайте keytab Kerberos для имени пользователя домена. Позже это keytab можно использовать для проверки подлинности в присоединенных к домену кластерах без ввода пароля. Доменное имя имеет прописные буквы:

```shell
ktutil
ktutil: addent -password -p <username>@<DOMAIN.COM> -k 1 -e RC4-HMAC
Password for <username>@<DOMAIN.COM>: <password>
ktutil: wkt <username>.keytab
ktutil: q
```

### <a name="can-i-use-an-existing-azure-active-directory-tenant-to-create-an-hdinsight-cluster-that-has-the-esp"></a>Можно ли использовать существующий клиент Azure Active Directory для создания кластера HDInsight с ESP?

Перед созданием кластера HDInsight с помощью ESP включите Azure Active Directory доменные службы (Azure AD DS). В случае с открытым исходным кодом Hadoop использует Kerberos для проверки подлинности (в отличие от OAuth).

Чтобы присоединить виртуальные машины к домену, необходимо иметь контроллер домена. Azure AD DS является управляемым контроллером домена и считается расширением Azure Active Directory. Azure AD DS предоставляет все требования Kerberos для создания защищенного кластера Hadoop управляемым способом. HDInsight в качестве управляемой службы интегрируется с Azure AD DS для обеспечения безопасности.

### <a name="can-i-use-a-self-signed-certificate-in-an-aad-ds-secure-ldap-setup-and-provision-an-esp-cluster"></a>Можно ли использовать самозаверяющий сертификат в настройке защищенного протокола LDAP в AAD-DS и подготавливать кластер ESP?

Рекомендуется использовать сертификат, выданный центром сертификации. Но использование самозаверяющего сертификата также поддерживается в ESP. Дополнительные сведения см. в разделе:

- [Включение доменных служб Azure Active Directory](domain-joined/apache-domain-joined-configure-using-azure-adds.md#enable-azure-ad-ds)

- [Руководство по настройке защищенного протокола LDAP для управляемого домена доменных служб Azure AD](../active-directory-domain-services/tutorial-configure-ldaps.md)

### <a name="how-can-i-pull-login-activity-shown-in-ranger"></a>Как можно извлечь действие входа в систему, показанное в Ranger?

Для требований аудита Корпорация Майкрософт рекомендует включить журналы Azure Monitor, как описано в разделе [Использование журналов Azure Monitor для мониторинга кластеров HDInsight](./hdinsight-hadoop-oms-log-analytics-tutorial.md).

### <a name="can-i-disable-clamscan-on-my-cluster"></a>Можно ли отключить `Clamscan` в кластере?

`Clamscan` — это антивирусное программное обеспечение, которое работает в кластере HDInsight и используется системой безопасности Azure (азсекд) для защиты кластеров от атак с вирусами. Корпорация Майкрософт настоятельно рекомендует пользователям не изменять конфигурацию по умолчанию `Clamscan` .

Этот процесс не мешает или не принимает никаких циклов от других процессов. Он всегда будет давать другой процесс. Пики загрузки ЦП `Clamscan` должны отображаться только в том случае, если система бездействует.  

В сценариях, в которых необходимо управлять расписанием, можно выполнить следующие действия.

1. Отключите автоматическое выполнение с помощью следующей команды:
   
  `usr/local/bin/azsecd config -s clamav -d Disabled`азсекд перезагрузка службы sudo sudo 
   
1. Добавьте задание cron, которое выполняет следующую команду в качестве привилегированной:
   
   `/usr/local/bin/azsecd manual -s clamav`

Дополнительные сведения о настройке и выполнении задания cron см. в разделе [разделы справки Настройка задания cron](https://askubuntu.com/questions/2368/how-do-i-set-up-a-cron-job).

### <a name="why-is-llap-available-on-spark-esp-clusters"></a>Почему LLAP доступен в кластерах Spark ESP?
LLAP включен по соображениям безопасности (Apache Ranger), а не к производительности. Используйте виртуальные машины большего размера, чтобы обеспечить использование ресурсов LLAP (например, минимум D13V2). 

### <a name="how-can-i-add-additional-aad-groups-after-creating-an-esp-cluster"></a>Как добавить дополнительные группы AAD после создания кластера ESP?
Эта цель достигается двумя способами: 1 — можно повторно создать кластер и добавить дополнительную группу во время создания кластера. Если вы используете синхронизацию с заданной областью в AAD-DS, убедитесь, что Группа B включена в синхронизацию с заданной областью.
2. Добавьте группу в качестве вложенной подгруппы предыдущей группы, которая использовалась для создания кластера ESP. Например, если вы создали кластер ESP с группой `A` , позднее можно добавить группу в `B` качестве вложенной подгруппы, `A` а через примерно один час она будет синхронизирована и доступна в кластере автоматически. 

## <a name="storage"></a>Память

### <a name="can-i-add-an-azure-data-lake-storage-gen2-to-an-existing-hdinsight-cluster-as-an-additional-storage-account"></a>Можно ли добавить Azure Data Lake Storage 2-го поколения в существующий кластер HDInsight в качестве дополнительной учетной записи хранения?

Нет, в настоящее время невозможно добавить учетную запись хранения Azure Data Lake Storage 2-го поколения в кластер, имеющий хранилище BLOB-объектов в качестве основного хранилища. Дополнительные сведения см. в разделе [Сравнение параметров хранения](hdinsight-hadoop-compare-storage-options.md).

### <a name="how-can-i-find-the-currently-linked-service-principal-for-a-data-lake-storage-account"></a>Как найти связанный субъект-службу для Data Lake учетной записи хранения?

Параметры можно найти в **Data Lake Storage 1-го поколения доступе** в свойствах кластера в портал Azure. Дополнительные сведения см. в разделе [Проверка настройки кластера](../data-lake-store/data-lake-store-hdinsight-hadoop-use-portal.md#verify-cluster-set-up).
 
### <a name="how-can-i-calculate-the-usage-of-storage-accounts-and-blob-containers-for-my-hdinsight-clusters"></a>Как можно рассчитать использование учетных записей хранения и контейнеров больших двоичных объектов для кластеров HDInsight?

Выполните одно из следующих действий.

- [Использование PowerShell](../storage/scripts/storage-blobs-container-calculate-size-powershell.md)

- Найти размер */Усер/Хиве/. Корзина/* папка в кластере HDInsight с помощью следующей командной строки:
  
  `hdfs dfs -du -h /user/hive/.Trash/`

### <a name="how-can-i-set-up-auditing-for-my-blob-storage-account"></a>Как настроить аудит для учетной записи хранения BLOB-объектов?

Для аудита учетных записей хранения BLOB-объектов Настройте мониторинг с помощью процедуры, описанной в разделе [мониторинг учетной записи хранения в портал Azure](../storage/common/manage-storage-analytics-logs.md). Журнал аудита HDFS содержит только данные аудита только для локальной файловой системы HDFS (hdfs://mycluster).  Она не включает операции, выполняемые в удаленном хранилище.

### <a name="how-can-i-transfer-files-between-a-blob-container-and-an-hdinsight-head-node"></a>Как передавать файлы между контейнером больших двоичных объектов и головным узлом HDInsight?

Выполните сценарий, аналогичный следующему сценарию оболочки на головном узле:

```shell
for i in cat filenames.txt
do
   hadoop fs -get $i <local destination>
done
```
 
> [!NOTE]
> Файл *filenames.txt* будет иметь абсолютный путь к файлам в контейнерах больших двоичных объектов.
 
### <a name="are-there-any-ranger-plugins-for-storage"></a>Существуют ли подключаемые модули Ranger для хранения?

В настоящее время для хранилища BLOB-объектов и Azure Data Lake Storage 1-го поколения или Gen2 не существует подключаемого модуля Ranger. Для кластеров ESP следует использовать Azure Data Lake Storage. Можно по крайней мере задать разрешения точной детализации на уровне файловой системы вручную с помощью средств HDFS. Кроме того, при использовании Azure Data Lake Storage кластеры ESP будут выполнять некоторые действия по управлению доступом к файловой системе с помощью Azure Active Directory на уровне кластера. 

Вы можете назначать политики доступа к данным для групп безопасности пользователей с помощью Обозреватель службы хранилища Azure. Дополнительные сведения см. в разделе:

- [Разделы справки задать разрешения для пользователей Azure AD на запрос данных в Data Lake Storage 2-го поколения с помощью Hive или других служб?](hdinsight-hadoop-use-data-lake-storage-gen2.md#how-do-i-set-permissions-for-azure-ad-users-to-query-data-in-data-lake-storage-gen2-by-using-hive-or-other-services)
- [Установка разрешений на уровне файлов и каталогов в Azure Data Lake Storage 2-го поколения с помощью Обозревателя службы хранилища Azure](../storage/blobs/data-lake-storage-explorer.md)

### <a name="can-i-increase-hdfs-storage-on-a-cluster-without-increasing-the-disk-size-of-worker-nodes"></a>Можно ли увеличить хранилище HDFS в кластере, не увеличивая размер диска рабочих узлов?

Нет. Нельзя увеличить размер диска для рабочего узла. Поэтому единственным способом увеличения размера диска является удаление кластера и его повторное создание с помощью крупных рабочих виртуальных машин. Не используйте HDFS для хранения данных HDInsight, так как данные удаляются при удалении кластера. Вместо этого храните данные в Azure. Увеличение масштаба кластера может также увеличить емкость кластера HDInsight.

## <a name="edge-nodes"></a>Граничные узлы

### <a name="can-i-add-an-edge-node-after-the-cluster-has-been-created"></a>Можно ли добавить граничные узлы после создания кластера?

См. статью [использование пустых граничных узлов в кластерах Apache Hadoop в HDInsight](hdinsight-apps-use-edge-node.md).

### <a name="how-can-i-connect-to-an-edge-node"></a>Как подключиться к пограничному узлу?

После создания пограничного узла можно подключиться к нему по протоколу SSH через порт 22. Имя пограничного узла можно найти на портале кластера. Имена обычно оканчиваются на *-ED*.

### <a name="why-are-persisted-scripts-not-running-automatically-on-newly-created-edge-nodes"></a>Почему сохраняемые скрипты не запускаются автоматически на вновь созданных граничных узлах?

Сохраненные сценарии используются для настройки новых рабочих узлов, добавляемых в кластер с помощью операций масштабирования. Сохраненные сценарии не применяются к граничным узлам.

## <a name="rest-api"></a>REST API

### <a name="what-are-the-rest-api-calls-to-pull-a-tez-query-view-from-the-cluster"></a>Какие REST API вызовы, чтобы извлечь представление запроса Tez из кластера?

Для извлечения необходимой информации в формате JSON можно использовать следующие конечные точки RESTFUL. Для выполнения запросов используйте заголовки обычной проверки подлинности.

- `Tez Query View`: *https: \/ / \<cluster name> . azurehdinsight.NET/WS/v1/Timeline/HIVE_QUERY_ID/*
- `Tez Dag View`: *https: \/ / \<cluster name> . azurehdinsight.NET/WS/v1/Timeline/TEZ_DAG_ID/*

### <a name="how-do-i-retrieve-the-configuration-details-from-hdi-cluster-by-using-an-azure-active-directory-user"></a>Разделы справки получить сведения о конфигурации из кластера HDI с помощью Azure Active Directory пользователя?

Чтобы согласовать маркеры проверки подлинности с пользователем AAD, пройдите через шлюз, используя следующий формат:

* https:// `<cluster dnsname>` . azurehdinsight.NET/API/V1/Clusters/testclusterdem/stack_versions/1/repository_versions/1 

### <a name="how-do-i-use-ambari-restful-api-to-monitor-yarn-performance"></a>Разделы справки использовать API RESTful Ambari для мониторинга производительности YARN?

При вызове команды «изогнутая» в той же виртуальной сети или в одноранговой виртуальной сети команда:

```curl
curl -u <cluster login username> -sS -G
http://<headnodehost>:8080/api/v1/clusters/<ClusterName>/services/YARN/components/NODEMANAGER?fields=metrics/cpu
```
 
При вызове команды из-за пределов виртуальной сети или непирингной виртуальной сети формат команды будет следующим:

- Для кластера, не являющегося кластером ESP:
  
  ```curl
  curl -u <cluster login username> -sS -G 
  https://<ClusterName>.azurehdinsight.net/api/v1/clusters/<ClusterName>/services/YARN/components/NODEMANAGER?fields=metrics/cpu
  ```

- Для кластера ESP:
  
  ```curl
  curl -u <cluster login username>-sS -G 
  https://<ClusterName>.azurehdinsight.net/api/v1/clusters/<ClusterName>/services/YARN/components/NODEMANAGER?fields=metrics/cpu
  ```

> [!NOTE]
> В фигурной скобке будет предложено ввести пароль. Необходимо ввести допустимый пароль для имени пользователя для входа в кластер.

## <a name="billing"></a>Выставление счетов

### <a name="how-much-does-it-cost-to-deploy-an-hdinsight-cluster"></a>Какова стоимость развертывания кластера HDInsight?

Дополнительные сведения о ценах и часто задаваемых вопросах, связанных с выставлением счетов, см. на странице [цен на Azure HDInsight](https://azure.microsoft.com/pricing/details/hdinsight/) .

### <a name="when-does-hdinsight-billing-start--stop"></a>Когда начинается оплата с учета HDInsight & останавливаться?

Начисление оплаты начинается после создания кластера HDInsight и прекращается только после его удаления. Стоимость выставления счетов составляет Pro в минуту.

### <a name="how-do-i-cancel-my-subscription"></a>Разделы справки отменить подписку?

Сведения о том, как отменить подписку, см. в разделе [Отмена подписки Azure](../cost-management-billing/manage/cancel-azure-subscription.md).

### <a name="for-pay-as-you-go-subscriptions-what-happens-after-i-cancel-my-subscription"></a>Что происходит после отмены подписки с оплатой по мере использования?

Сведения о подписке после ее отмены см. в разделе [что происходит после отмены подписки?](../cost-management-billing/manage/cancel-azure-subscription.md)

## <a name="hive"></a>Hive

### <a name="why-does-the-hive-version-appear-as-121000-instead-of-21-in-the-ambari-ui-even-though-im-running-an-hdinsight-36-cluster"></a>Почему версия Hive отображается как 1.2.1000 вместо 2,1 в пользовательском интерфейсе Ambari, даже если выполняется кластер HDInsight 3,6?

Хотя в пользовательском интерфейсе Ambari отображается только 1,2, HDInsight 3,6 содержит как Hive 1,2, так и Hive 2,1.

## <a name="other-faq"></a>Другие вопросы и ответы

### <a name="what-does-hdinsight-offer-for-real-time-stream-processing-capabilities"></a>Что предлагает HDInsight для возможностей обработки потоков в режиме реального времени?

Сведения о возможностях интеграции потоковой обработки см. [в статье Выбор технологии потоковой обработки в Azure](/azure/architecture/data-guide/technology-choices/stream-processing).

### <a name="is-there-a-way-to-dynamically-kill-the-head-node-of-the-cluster-when-the-cluster-is-idle-for-a-specific-period"></a>Существует ли способ динамического уничтожения головного узла кластера, когда кластер бездействует в течение определенного периода?

Это действие нельзя выполнить с кластерами HDInsight. В этих сценариях можно использовать фабрику данных Azure.

### <a name="what-compliance-offerings-does-hdinsight-offer"></a>Какие предложения соответствия предлагает HDInsight?

Сведения о соответствии требованиям см. в [центре управления безопасностью Майкрософт](https://www.microsoft.com/trust-center) и в [обзоре соответствия Microsoft Azure](https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942).