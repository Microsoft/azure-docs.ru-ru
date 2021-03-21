---
title: Установка кластеров в HDInsight с использованием Apache Hadoop, Apache Spark, Apache Kafka и других технологий
description: Сведения об установке кластеров Hadoop, Kafka, Spark, HBase, Microsoft R Server или Storm для HDInsight из браузера с помощью классического Azure CLI, Azure PowerShell, REST или пакета SDK.
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive,hdiseo17may2017,seodec18
ms.date: 08/06/2020
ms.openlocfilehash: a04d976ed3d9be253425b4c8a5a5c0effb6f80b8
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101741843"
---
# <a name="set-up-clusters-in-hdinsight-with-apache-hadoop-apache-spark-apache-kafka-and-more"></a>Установка кластеров в HDInsight с использованием Apache Hadoop, Apache Spark, Apache Kafka и других технологий

[!INCLUDE [selector](../../includes/hdinsight-create-linux-cluster-selector.md)]

Узнайте, как настроить и настроить Apache Hadoop, Apache Spark, Apache Kafka, интерактивный запрос, Apache HBase, службы ML или Apache Storm в HDInsight. Кроме того, вы узнаете, как повысить безопасность кластеров путем их присоединения к домену.

Кластер Hadoop включает в себя несколько виртуальных машин (узлов), которые используются для распределенной обработки задач. Azure HDInsight управляет сведениями об установке и настройке отдельных узлов, поэтому вам нужно указать только общие сведения о конфигурации.

> [!IMPORTANT]  
> Начисление оплаты начинается после создания кластера HDInsight и прекращается только после его удаления. Кластеры оплачиваются поминутно, поэтому всегда следует удалять кластер, когда он больше не нужен. Узнайте, как [удалить кластер](hdinsight-delete-cluster.md).

Если вы используете несколько кластеров вместе, вам нужно создать виртуальную сеть. Если же вы используете кластер Spark, вы также можете использовать Hive Warehouse Connector. См. сведения о [планировании виртуальной сети для Azure HDInsight](./hdinsight-plan-virtual-network-deployment.md) и [интеграции Apache Spark и Apache Hive с Hive Warehouse Connector](interactive-query/apache-hive-warehouse-connector.md).

## <a name="cluster-setup-methods"></a>Способы установки кластера

В приведенной ниже таблице представлены различные способы установки кластера HDInsight.

| Метод создания кластеров | браузер | Командная строка | REST API | SDK |
| --- |:---:|:---:|:---:|:---:|
| [Портал Azure](hdinsight-hadoop-create-linux-clusters-portal.md) |✔ |&nbsp; |&nbsp; |&nbsp; |
| [Фабрика данных Azure](hdinsight-hadoop-create-linux-clusters-adf.md). |✔ |✔ |✔ |✔ |
| [Azure CLI](hdinsight-hadoop-create-linux-clusters-azure-cli.md) |&nbsp; |✔ |&nbsp; |&nbsp; |
| [Azure PowerShell](hdinsight-hadoop-create-linux-clusters-azure-powershell.md) |&nbsp; |✔ |&nbsp; |&nbsp; |
| [cURL](hdinsight-hadoop-create-linux-clusters-curl-rest.md) |&nbsp; |✔ |✔ |&nbsp; |
| [Шаблоны диспетчера ресурсов Azure](hdinsight-hadoop-create-linux-clusters-arm-templates.md) |&nbsp; |✔ |&nbsp; |&nbsp; |

В этой статье описывается настройка в [портал Azure](https://portal.azure.com), где можно создать кластер HDInsight.

## <a name="basics"></a>Основы

![быстрое создание параметров hdinsight — настраиваемая Быстрая](./media/hdinsight-hadoop-provision-linux-clusters/azure-portal-cluster-basics-blank-fs.png)

### <a name="project-details"></a>сведения о проекте;

[Azure Resource Manager](../azure-resource-manager/management/overview.md) помогает работать с ресурсами в приложении в виде группы, называемой [группой ресурсов](../azure-resource-manager/management/overview.md#resource-groups)Azure. Вы можете развертывать, обновлять, отслеживать или удалять все ресурсы для приложения в рамках одной скоординированной операции.

### <a name="cluster-details"></a>Сведения о кластере

#### <a name="cluster-name"></a>Имя кластера

Имена кластеров HDInsight имеют следующие ограничения.

* Допустимые символы: a – z, 0-9, A – Z
* Максимальная длина: 59
* Зарезервированные имена: приложения
* Область именования кластера предназначена для всех Azure во всех подписках. Поэтому имя кластера должно быть уникальным по всему миру.
* Первые шесть символов должны быть уникальными в пределах виртуальной сети.

#### <a name="region"></a>Регион

Расположение кластера не требуется указывать явно: кластер находится в том же расположении, что и хранилище по умолчанию. Чтобы получить список поддерживаемых регионов, выберите раскрывающийся список **регион** на странице [цен на HDInsight](https://go.microsoft.com/fwLink/?LinkID=282635&clcid=0x409).

#### <a name="cluster-type"></a>Тип кластера

В настоящее время Azure HDInsight предоставляет указанные ниже типы кластеров, каждый из которых имеет набор компонентов для предоставления определенных функциональных возможностей.

> [!IMPORTANT]  
> Доступны различные типы кластеров HDInsight, каждый из которых предназначен для отдельной рабочей нагрузки или технологии. Создать кластер, в котором бы объединились несколько типов, например Storm и HBase, нельзя. Если для решения нужны технологии, распределенные по нескольким типам кластеров HDInsight, [виртуальная сеть Azure](../virtual-network/index.yml) может объединять необходимые типы кластеров.

| Тип кластера | функциональное назначение; |
| --- | --- |
| [Hadoop](hadoop/apache-hadoop-introduction.md) |Пакетный запрос и анализ хранимых данных |
| [HBase](hbase/apache-hbase-overview.md) |Обработка больших объемов данных NoSQL без схемы |
| [Интерактивный запрос](./interactive-query/apache-interactive-query-get-started.md) |Кэширование в памяти для обеспечения интерактивных и ускоренных запросов Hive |
| [Kafka](kafka/apache-kafka-introduction.md) | Распределенная платформа потоковой передачи с открытым кодом, которую можно использовать для создания конвейеров и приложений потоковой передачи данных в режиме реального времени. |
| [Службы машинного обучения](r-server/r-server-overview.md) |Разнообразная статистика больших данных, прогнозное моделирование и возможности машинного обучения |
| [Spark](spark/apache-spark-overview.md) |Обработка в памяти, интерактивные запросы, обработка потоков микро-пакетов |
| [Буря](storm/apache-storm-overview.md) |Обработка событий в режиме реального времени |

#### <a name="version"></a>Версия

Выберите версию HDInsight для этого кластера. Дополнительные сведения см. в разделе [Поддерживаемые версии HDInsight](hdinsight-component-versioning.md#supported-hdinsight-versions).

### <a name="cluster-credentials"></a>Учетные данные кластера

Во время создания кластера HDInsight можно настроить две учетные записи пользователя.

* Имя пользователя для входа в кластер: имя пользователя по умолчанию — *Admin*. В нем используется базовая конфигурация портал Azure. Иногда он называется "пользователь кластера" или "пользователь HTTP".
* Secure Shell (SSH) имя пользователя: используется для подключения к кластеру по протоколу SSH. Дополнительные сведения см. в статье [Использование SSH с Hadoop на основе Linux в HDInsight из Linux, Unix или OS X](hdinsight-hadoop-linux-use-ssh-unix.md).

Имя пользователя HTTP имеет следующие ограничения.

* Разрешенные специальные символы: `_` и `@`
* Недопустимые символы: #;. "", \/ : "! *? $ () {} [] <>|&--= +% ~ ^ Space
* Максимальная длина: 20

Имя пользователя SSH имеет следующие ограничения.

* Разрешенные специальные символы: `_` и `@`
* Недопустимые символы: #;. "", \/ : "! *? $ () {} [] <>|&--= +% ~ ^ Space
* Максимальная длина: 64
* Зарезервированные имена: Hadoop, Users, Oozie, Hive, mapred, ambari-QA, Zookeeper, TEZ, HDFS, sqoop, Yarn, хкат, AMS, HBase, множество, администратор, администратор, пользователь, Пользователь1, тест, Пользователь2, test1, user3, admin1, 1, 123, a, актусер, ADM, Admin2, ASPNET, резервное копирование, консоль, Дэвид, гость, Джон, владелец, корень, сервер, SQL, поддержка, support_388945a0, sys, test2, test3, user4, User5, Spark

## <a name="storage"></a>Служба хранилища

![Параметры хранилища кластера: конечные точки, совместимые с HDFS](./media/hdinsight-hadoop-provision-linux-clusters/azure-portal-cluster-storage.png)

Хотя при локальной установке кластера Hadoop для хранилища используется распределенная файловая система Hadoop (HDFS), в облаке применяются конечные точки хранилища, подключенные к кластеру. Использование облачного хранилища позволяет безопасно удалять кластеры HDInsight, используемые для вычислений, сохраняя при этом данные.

Кластеры HDInsight могут использовать следующие варианты хранения:

* Azure Data Lake Storage 2-го поколения
* Хранилище Azure Data Lake Storage 1-го поколения
* Служба хранилища Azure общего назначения v2
* Служба хранилища Azure общего назначения v1
* Блочный BLOB-объект службы хранилища Azure (**поддерживается только в качестве дополнительного хранилища**)

Дополнительные сведения о вариантах хранения с помощью HDInsight см. в статье [Сравнение вариантов хранения для использования с кластерами Azure hdinsight](hdinsight-hadoop-compare-storage-options.md).

> [!WARNING]  
> Использование дополнительной учетной записи хранения, местоположение которой отличается от местоположения кластера HDInsight, не поддерживается.

Во время настройки вы указываете контейнер BLOB-объектов в учетной записи хранения Azure или хранилище Data Lake Storage для конечной точки хранилища по умолчанию. Хранилище по умолчанию содержит журналы приложений и системный журнал. При необходимости вы можете указать дополнительные связанные учетные записи хранения Azure и учетные записи Data Lake Storage, к которым кластер может получать доступ. Кластер HDInsight должен находиться в том же расположении Azure, что и зависимые учетные записи хранения.

[!INCLUDE [secure-transfer-enabled-storage-account](../../includes/hdinsight-secure-transfer.md)]

> [!IMPORTANT]
> Включение безопасного обмена хранилищами после создания кластера может привести к ошибкам с помощью учетной записи хранения и не рекомендуется. Лучше создать новый кластер с использованием учетной записи хранения с уже включенной безопасной защитой.

> [!Note]  
> Azure HDInsight не передает и не копирует данные, хранящиеся в хранилище Azure, из одного региона в другой автоматически.

### <a name="metastore-settings"></a>Параметры хранилище метаданных

Вы можете создать дополнительные хранилища метаданных Hive или Apache Oozie. Однако не все типы кластеров поддерживают метахранилища, а Azure синапсе Analytics несовместим с метахранилища.

Дополнительную информацию см. в статье [Использование внешних хранилищ метаданных в Azure HDInsight](./hdinsight-use-external-metadata-stores.md).

> [!IMPORTANT]  
> Создавая пользовательское хранилище метаданных, не используйте в имени базы данных тире, дефисы и пробелы. Это может привести к сбою при создании кластера.

#### <a name="sql-database-for-hive"></a>База данных SQL для Hive

Если вы хотите сохранить таблицы Hive после удаления кластера HDInsight, используйте пользовательское хранилище метаданных. Затем можно будет подключить это хранилище к другому кластеру HDInsight.

HDInsight An хранилище метаданных, созданный для одной версии кластера HDInsight, нельзя совместно использовать в разных версиях кластера HDInsight. Список версий HDInsight см. в разделе [Поддерживаемые версии HDInsight](hdinsight-component-versioning.md#supported-hdinsight-versions).

> [!IMPORTANT]
> По умолчанию хранилище метаданных предоставляет базу данных SQL Azure с **базовым ограничением DTU уровня 5 (не обновляемым)**. Подходит для базовых целей тестирования. Для больших или рабочих нагрузок рекомендуется переходить на внешний хранилище метаданных.

#### <a name="sql-database-for-oozie"></a>База данных SQL для Oozie

Для повышения производительности Oozie используйте пользовательское хранилище метаданных. Хранилище метаданных также позволяет осуществлять доступ к данным задания Oozie после удаления кластера.

#### <a name="sql-database-for-ambari"></a>База данных SQL для Ambari

Ambari используется для мониторинга кластеров HDInsight, внесения изменений в конфигурацию и хранения сведений об управлении кластерами, а также истории заданий. Функция Custom Ambari DB позволяет развернуть новый кластер и настроить Ambari во внешней управляемой базе данных. Дополнительные сведения см. в статье [Custom AMBARI DB](./hdinsight-custom-ambari-db.md).

> [!IMPORTANT]  
> Повторно использовать хранилище метаданных Oozie невозможно. Чтобы использовать пользовательское хранилище метаданных Oozie, при создании кластера HDInsight необходимо предоставить пустую базу данных SQL Azure.

## <a name="security--networking"></a>Безопасность и работа в сети

![hdinsight create options choose enterprise security package](./media/hdinsight-hadoop-provision-linux-clusters/azure-portal-cluster-security-networking.png)

### <a name="enterprise-security-package"></a>Пакет безопасности корпоративного уровня

Для кластеров типа Hadoop, Spark, HBase, Kafka и Interactive Query вы можете включить **Корпоративный пакет безопасности**. Этот пакет содержит параметр для более безопасной настройки кластера с помощью Apache Ranger и интеграции с Azure Active Directory. Дополнительные сведения см. [в статье обзор корпоративной безопасности в Azure HDInsight](./domain-joined/hdinsight-security-overview.md).

Пакет безопасности предприятия позволяет интегрировать HDInsight с Active Directory и Apache Ranger. При помощи пакета безопасности корпоративного уровня можно создать нескольких пользователей.

Дополнительные сведения о создании присоединенного к домену кластера HDInsight см. в статье о [создании присоединенной к домену среды песочницы HDInsight](./domain-joined/apache-domain-joined-configure-using-azure-adds.md).

### <a name="tls"></a>TLS

Дополнительные сведения см. в разделе [безопасность транспортного уровня](./transport-layer-security.md) .

### <a name="virtual-network"></a>Виртуальная сеть

Если для решения нужны технологии, распределенные по нескольким типам кластеров HDInsight, [виртуальная сеть Azure](../virtual-network/index.yml) может объединять необходимые типы кластеров. Благодаря такой конфигурации кластеры и любой развернутый в них код могут взаимодействовать друг с другом напрямую.

Дополнительные сведения об использовании виртуальной сети Azure с HDInsight см. в статье [планирование виртуальной сети для hdinsight](hdinsight-plan-virtual-network-deployment.md).

Пример использования двух типов кластера в виртуальной сети Azure см. в статье об [использовании структурированного потока Apache Spark при помощи Apache Kafka](hdinsight-apache-kafka-spark-structured-streaming.md). Дополнительные сведения об использовании HDInsight с виртуальной сетью, включая конкретные требования к конфигурации для виртуальной сети, см. в статье [планирование виртуальной сети для HDInsight](hdinsight-plan-virtual-network-deployment.md).

### <a name="disk-encryption-setting"></a>Параметр шифрования диска

Подробнее см. статью о [шифровании диска с управляемыми клиентом ключами](./disk-encryption.md).

### <a name="kafka-rest-proxy"></a>Прокси-сервер REST для Kafka

Этот параметр доступен только для типа кластера Kafka. Дополнительные сведения см. в разделе [использование прокси-сервера RESTful](./kafka/rest-proxy.md).

### <a name="identity"></a>Идентификация

Дополнительные сведения см. [в статье управляемые удостоверения в Azure HDInsight](./hdinsight-managed-identities.md).

## <a name="configuration--pricing"></a>Настройка и цены

![HDInsight выберите размер узла](./media/hdinsight-hadoop-provision-linux-clusters/azure-portal-cluster-configuration.png)

За использование узла взимается плата, если кластер существует. Начисление оплаты начинается после создания кластера и прекращается только после его удаления. Кластер не может быть освобожден или помещен в удержание.

### <a name="node-configuration"></a>конфигурация узла;

Для каждого типа кластера используется своя терминология. Кроме того, типы отличаются количеством узлов и стандартными размерами виртуальных машин. В следующей таблице число узлов каждого типа указано в скобках.

| Type | Узлы | Схема |
| --- | --- | --- |
| Hadoop |Головной узел (2), Рабочий узел (1 +) |![Узлы кластера HDInsight Hadoop](./media/hdinsight-hadoop-provision-linux-clusters/hdinsight-hadoop-cluster-type-nodes.png) |
| HBase |Головной сервер (2), региональный сервер (от 1), основной узел или узел Zookeeper (3) |![Установка типа кластера HDInsight HBase](./media/hdinsight-hadoop-provision-linux-clusters/hdinsight-hbase-cluster-type-setup.png) |
| Буря |Узел Nimbus (2), сервер супервизора (от 1), узел Zookeeper (3) |![Установка типа кластера HDInsight](./media/hdinsight-hadoop-provision-linux-clusters/hdinsight-storm-cluster-type-setup.png) |
| Spark |Головной узел (2), Рабочий узел (1 +), узел ZooKeeper (3) (бесплатно для размера виртуальной машины a1 ZooKeeper) |![Установка типа кластера HDInsight Spark](./media/hdinsight-hadoop-provision-linux-clusters/hdinsight-spark-cluster-type-setup.png) |

Дополнительные сведения см. в разделе [Конфигурация узлов и размеры виртуальных машин по умолчанию для кластеров](hdinsight-supported-node-configuration.md) статьи "Что представляют собой компоненты и версии Hadoop, доступные в HDInsight?"

Затраты на кластеры HDInsight определяются числом узлов и размерами виртуальных машин на узлах.

Кластеры разных типов отличаются типами, количеством и размерами узлов.
* Тип кластера Hadoop по умолчанию:
    * два *головных узла*;  
    * Четыре *рабочих узла*
* Тип кластера Storm по умолчанию:
    * два *узла Nimbus*;
    * три *узла ZooKeeper*;
    * четыре *узла супервизора*.

Если вы только используете HDInsight, мы рекомендуем использовать один рабочий узел. Подробные сведения о ценах на HDInsight см. на [этой странице](https://go.microsoft.com/fwLink/?LinkID=282635&clcid=0x409).

> [!NOTE]  
> Максимальный размер кластера зависит от подписки Azure. Чтобы увеличить лимит, обратитесь в [службу поддержки по вопросам выставления счетов Azure](../azure-portal/supportability/how-to-create-azure-support-request.md).

При использовании портал Azure для настройки кластера размер узла можно получить на вкладке **Настройка и цены** . На портале также можно просмотреть затраты, связанные с различными размерами узлов.

### <a name="virtual-machine-sizes"></a>Размер виртуальных машин

При развертывании кластеров выберите вычислительные ресурсы в зависимости от решения, которое планируется развернуть. Для кластеров HDInsight используются следующие виртуальные машины:

* виртуальные машины серий A и D1–4: [Размеры виртуальных машин Linux общего назначения](../virtual-machines/sizes-general.md);
* виртуальные машины серии D11–14: [Размеры виртуальных машин Linux, оптимизированных для памяти](../virtual-machines/sizes-memory.md).

Сведения о том, какие значения следует указывать для размера виртуальной машины при создании кластера с помощью различных пакетов SDK или Azure PowerShell, см. в разделе [Таблицы размеров](../cloud-services/cloud-services-sizes-specs.md#size-tables). Используйте значение из столбца **Размер** таблиц, приведенных в этой статье.

> [!IMPORTANT]  
> Если в кластере требуется более 32 рабочих узлов, необходимо выбрать размер головного узла с по крайней мере 8 ядрами и 14 ГБ ОЗУ.

Дополнительные сведения см. в статье [размеры виртуальных машин](../virtual-machines/sizes.md). Сведения о расценках на разные размеры см. [здесь](https://azure.microsoft.com/pricing/details/hdinsight).

### <a name="add-application"></a>Добавить приложение

Пользователи могут устанавливать приложения HDInsight в кластере HDInsight под управлением Linux. Вы можете использовать приложения, предоставляемые корпорацией Майкрософт, сторонними производителями или разработанные самостоятельно. Дополнительные сведения см. в статье [Установка сторонних приложений Apache Hadoop в Azure HDInsight](hdinsight-apps-install-applications.md).

Большинство приложений HDInsight устанавливаются в пустой граничный узел.  Пустой граничный узел — это виртуальная машина Linux, на которой установлены и настроены те же клиентские инструменты, что и на головном узле. Граничный узел можно использовать для доступа к кластеру, а также тестирования и размещения клиентских приложений. Подробные сведения см. в статье [Использование пустых граничных узлов в HDInsight](hdinsight-apps-use-edge-node.md).

### <a name="script-actions"></a>Действия сценария

Можно установить дополнительные компоненты или настроить конфигурацию кластера с помощью сценариев во время создания. Такие скрипты вызываются с помощью **действия скрипта** — параметра конфигурации, который может использоваться с помощью портала Azure, командлетов HDInsight PowerShell или пакета SDK для HDInsight .NET. Дополнительные сведения см. в статье [Настройка кластеров HDInsight под управлением Linux с помощью действия сценария](hdinsight-hadoop-customize-cluster-linux.md).

Некоторые собственные компоненты Java, такие как Apache Mahout и Cascading, могут выполняться в кластере как архивные файлы (JAR) Java. Эти JAR-файлы можно распространить в службе хранилища Azure и отправить в кластеры HDInsight с помощью механизмов отправки заданий Hadoop. См. дополнительные сведения об [отправке заданий Apache Hadoop программными средствами](hadoop/submit-apache-hadoop-jobs-programmatically.md).

> [!NOTE]  
> Если при развертывании или вызове JAR-файлов в кластерах HDInsight возникают проблемы, обратитесь в [службу поддержки Майкрософт](https://azure.microsoft.com/support/options/).
>
> Компонент Cascading не поддерживается службой HDInsight и на него не распространяется техническая поддержка Майкрософт. Списки поддерживаемых компонентов см. в статье [Что представляют собой различные компоненты Hadoop, доступные в HDInsight?](hdinsight-component-versioning.md)

В некоторых случаях требуется изменить следующие файлы конфигурации в процессе создания:

* clusterIdentity.xml
* core-site.xml
* gateway.xml
* hbase-env.xml
* hbase-site.xml
* hdfs-site.xml
* hive-env.xml
* Hive-site.xml
* mapred-site
* oozie-site.xml
* oozie-env.xml
* storm-site.xml
* tez-site.xml
* webhcat-site.xml
* yarn-site.xml

Подробные сведения см. в статье [Настройка кластеров HDInsight с помощью начальной загрузки](hdinsight-hadoop-customize-cluster-bootstrap.md).

## <a name="next-steps"></a>Дальнейшие действия

* [Устранение сбоев при создании кластера с помощью Azure HDInsight](./hadoop/hdinsight-troubleshoot-cluster-creation-fails.md)
* [Что такое Azure HDInsight и стек технологий Apache Hadoop](hadoop/apache-hadoop-introduction.md)
* [Краткое руководство. Использование Apache Hadoop и Apache Hive в Azure HDInsight с шаблоном Resource Manager](hadoop/apache-hadoop-linux-tutorial-get-started.md)
* [Работа в экосистеме Hadoop в HDInsight на компьютере с Windows](hdinsight-hadoop-windows-tools.md)