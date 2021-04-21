---
title: Советы по использованию Hadoop в HDInsight под управлением Linux в Azure
description: Советы по использованию кластеров HDInsight (Hadoop) на базе Linux в привычной среде Linux, выполняемой в облаке Azure.
ms.service: hdinsight
ms.custom: hdinsightactive,seoapr2020
ms.topic: conceptual
ms.date: 04/29/2020
ms.openlocfilehash: 2d2619c7bd7bc09eeab3845599758db7134b4134
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98945656"
---
# <a name="information-about-using-hdinsight-on-linux"></a>Сведения об использовании HDInsight в Linux

Кластеры Azure HDInsight предоставляют Apache Hadoop в привычной среде Linux, выполняемой в облаке Azure. Для большинства задач они должны работать так же, как и любые другие установки Hadoop в Linux. В этом документе рассматриваются определенные отличия, которые при этом следует учитывать.

## <a name="prerequisites"></a>Предварительные требования

При выполнении многих действий, описанных в этом документе, используются следующие служебные программы, которые может потребоваться установить в системе:

* [cURL](https://curl.haxx.se/) — используется для взаимодействия с веб-службами.
* **jq** — обработчик командной строки JSON.  См. раздел [https://stedolan.github.io/jq/](https://stedolan.github.io/jq/).
* [Azure CLI](/cli/azure/install-azure-cli) используется для удаленного управления службами Azure.
* **Клиент SSH**. Дополнительные сведения см. в руководстве по [подключению к HDInsight (Apache Hadoop) с помощью SSH](hdinsight-hadoop-linux-use-ssh-unix.md).

## <a name="users"></a>Пользователи

Если кластер HDInsight не [присоединен к домену](./domain-joined/hdinsight-security-overview.md), его нужно рассматривать как **однопользовательскую** систему. В кластере создается отдельная учетная запись пользователя SSH с разрешениями уровня администратора. Можно создать и дополнительные учетные записи SSH, но им также предоставляются права администратора для доступа к кластеру.

Присоединенный к домену кластер HDInsight поддерживает несколько пользователей, а также более детализированные параметры разрешений и ролей. Дополнительные сведения см. в статье [Manage Domain-joined HDInsight clusters](./domain-joined/apache-domain-joined-manage.md) (Управление присоединенными к домену кластерами HDInsight).

## <a name="domain-names"></a>Имена доменов

При подключении к кластеру из Интернета следует использовать полное доменное имя (FQDN) `CLUSTERNAME.azurehdinsight.net` или `CLUSTERNAME-ssh.azurehdinsight.net` (только для SSH).

На внутреннем уровне каждый узел в кластере имеет имя, назначаемое при конфигурации кластера. Имена кластеров вы найдете на странице **Hosts** (Узлы) в веб-интерфейсе Ambari. Список узлов можно также получить через интерфейс REST API Ambari следующим образом:

```console
curl -u admin -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/hosts" | jq '.items[].Hosts.host_name'
```

Замените `CLUSTERNAME` именем кластера. При появлении запроса введите пароль для учетной записи администратора. Эта команда возвращает документ JSON, который содержит список узлов в кластере. Программа [jq](https://stedolan.github.io/jq/) извлекает значение элемента `host_name` для каждого узла.

Если требуется найти имя узла для конкретной службы, можно запросить Ambari для этого компонента. Например, чтобы найти узлы для узла имен HDFS, выполните следующую команду:

```console
curl -u admin -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services/HDFS/components/NAMENODE" | jq '.host_components[].HostRoles.host_name'
```

Эта команда возвращает документ JSON с описанием службы, а затем программа [jq](https://stedolan.github.io/jq/) извлекает только значение `host_name` для узлов.

## <a name="remote-access-to-services"></a>Удаленный доступ к службам

* **Веб-интерфейс Ambari** - `https://CLUSTERNAME.azurehdinsight.net`

    Выполните аутентификацию, а затем войдите в Ambari.

    При аутентификации используется открытый текст. Всегда используйте протокол HTTPS, чтобы обеспечить безопасное подключение.

    > [!IMPORTANT]  
    > Некоторые веб-интерфейсы, предоставляемые через узлы Ambari, используют внутреннее доменное имя. Внутренние доменные имена не доступны из Интернета. Вы можете получать ошибку "сервер не найден" при обращении к некоторым функциям через Интернет.
    >
    > Чтобы получить доступ ко всем функциям веб-интерфейса Ambari, используйте туннелирование SSH для проксирования веб-трафика на головной узел кластера. Дополнительные сведения см. в статье об [использовании туннелирования SSH для доступа к веб-интерфейсу Apache Ambari, ResourceManager, JobHistory, NameNode, Oozie и другим веб-интерфейсам](hdinsight-linux-ambari-ssh-tunnel.md).

* **Ambari (REST)**  - `https://CLUSTERNAME.azurehdinsight.net/ambari`

    > [!NOTE]  
    > Выполните аутентификацию, используя имя пользователя и пароль администратора кластера.
    >
    > При аутентификации используется открытый текст. Всегда используйте протокол HTTPS, чтобы обеспечить безопасное подключение.

* **WebHCat (Templeton)**  - `https://CLUSTERNAME.azurehdinsight.net/templeton`

    > [!NOTE]  
    > Выполните аутентификацию, используя имя пользователя и пароль администратора кластера.
    >
    > При аутентификации используется открытый текст. Всегда используйте протокол HTTPS, чтобы обеспечить безопасное подключение.

* **SSH** — CLUSTERNAME-ssh.azurehdinsight.net через порт 22 или 23. Для подключения к основному головному узлу используется порт 22, а для подключения к дополнительному — порт 23. Дополнительные сведения о головных узлах см. в статье [Доступность и надежность кластеров Hadoop в HDInsight](./hdinsight-business-continuity.md).

    > [!NOTE]  
    > Доступ к головным узлам кластера можно получить только по протоколу SSH с клиентского компьютера. После подключения с головного узла можно получить доступ к рабочим узлам по протоколу SSH.

Дополнительные сведения см. в документе о [портах, используемых службами Apache Hadoop в HDInsight](hdinsight-hadoop-port-settings-for-services.md).

## <a name="file-locations"></a>Местоположения файлов

Связанные с Hadoop файлы можно найти на узлах кластера в папке `/usr/hdp`. Этот каталог содержит следующие подкаталоги:

* **2.6.5.3009-43**: именем каталога является версия платформы Hadoop, используемая HDInsight. Это число для вашего кластера может отличаться от указанного здесь.
* **current**: этот каталог содержит ссылки на подкаталоги в каталоге **2.6.5.3009-43**. Он позволяет вам не запоминать номер версии.

Примеры данных и JAR-файлы можно найти в распределенной файловой системе Hadoop в папках `/example` и `/HdiSamples`.

## <a name="hdfs-azure-storage-and-data-lake-storage"></a>HDFS, служба хранилища Azure и Data Lake Storage

В большинстве дистрибутивов Hadoop данные хранятся в HDFS. HDFS реализуется на базе локального хранилища на компьютерах в кластере. Использование локального хранилища в случае облачного решения с почасовой или поминутной платой за вычислительные ресурсы может оказаться весьма затратным.

При использовании HDInsight файлы данных хранятся в облаке с применением хранилища BLOB-объектов Azure и при необходимости Azure Data Lake Storage 1-го или 2-го поколения, что позволяет обеспечить адаптацию и устойчивость. Эти службы предоставляют следующие преимущества:

* Недорогое долговременное хранилище;
* доступность из внешних служб, например веб-сайтов, служебных программ для отправки или скачивания файлов, пакетов SDK для различных языков и веб-браузеров;
* Емкость, рассчитанная на файлы большого размера, и большое адаптируемое хранилище.

Дополнительные сведения см. в статьях о [хранилище BLOB-объектов Azure](../storage/common/storage-introduction.md), [Azure Data Lake Storage 1-го поколения](../data-lake-store/data-lake-store-overview.md) и [Azure Data Lake Storage 2-го поколения](../storage/blobs/data-lake-storage-introduction.md).

Если вы используете хранилище BLOB-объектов Azure либо Data Lake Storage 1-го или 2-го поколения, для доступа к данным не требуется никаких специальных действий в HDInsight. Например, следующая команда отображает список файлов в папке `/example/data` независимо от того, хранится ли она в службе хранилища Azure или в Data Lake Storage:

```console
hdfs dfs -ls /example/data
```

В HDInsight ресурсы хранилища данных (хранилище BLOB-объектов Azure и Azure Data Lake Storage) отделены от вычислительных ресурсов. Кластеры HDInsight можно создавать для выполнения необходимых вычислений и удалять их после завершения работы. При этом файлы данных будут храниться в облачном хранилище столько, сколько потребуется.

### <a name="uri-and-scheme"></a><a name="URI-and-scheme"></a>URI и схема

Некоторые команды требуют при доступе к файлу указывать схему в составе URI. Например, схема обязательна для компонента Storm-HDFS. Если вы используете нестандартное хранилище (добавленное в кластер как дополнительное хранилище), необходимо всегда включать схему в состав URI.

При использовании [**службы хранилища Azure**](./hdinsight-hadoop-use-blob-storage.md) выберите одну из следующих схем URI:

* `wasb:///`: Хранилище по умолчанию без шифрования обмена данными.

* `wasbs:///`: Хранилище по умолчанию с шифрованием обмена данными.  Схема wasbs поддерживается только в HDInsight начиная с версии 3.6.

* `wasb://<container-name>@<account-name>.blob.core.windows.net/`: При взаимодействии с учетной записью хранения, кроме используемой по умолчанию. Например, используется для дополнительной учетной записи хранения или при доступе к данным в общедоступной учетной записи хранения.

При использовании [**Azure Data Lake Storage 2-го поколения**](./hdinsight-hadoop-use-data-lake-storage-gen2.md) выберите следующую схему URI:

* `abfs://`: Хранилище по умолчанию с шифрованием обмена данными.

* `abfs://<container-name>@<account-name>.dfs.core.windows.net/`: При взаимодействии с учетной записью хранения, кроме используемой по умолчанию. Например, используется для дополнительной учетной записи хранения или при доступе к данным в общедоступной учетной записи хранения.

При использовании [**Azure Data Lake Storage 1-го поколения**](../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen1.md) выберите одну из следующих схем URI:

* `adl:///`: доступ к хранилищу Data Lake Storage, используемому по умолчанию для кластера.

* `adl://<storage-name>.azuredatalakestore.net/`: используется при взаимодействии с Data Lake Storage, отличным от используемого по умолчанию. Также применяется для доступа к данным вне корневого каталога вашего кластера HDInsight.

> [!IMPORTANT]  
> Если в качестве хранилища по умолчанию для HDInsight используется Data Lake Storage, необходимо указать путь внутри хранилища, который будет корневым каталогом для хранилища HDInsight. По умолчанию используется путь `/clusters/<cluster-name>/`.
>
> Если для доступа к данным используется `/` или `adl:///`, доступны только те данные, которые расположены в корневом каталоге кластера (например, `/clusters/<cluster-name>/`). Чтобы получить доступ ко всему хранилищу, используйте формат `adl://<storage-name>.azuredatalakestore.net/`.

### <a name="what-storage-is-the-cluster-using"></a>Какие хранилища используются в кластере

С помощью Ambari можно узнать конфигурацию хранилища, используемого по умолчанию для кластера. Для получения информации о конфигурации HDFS используйте команду curl и выполните фильтрацию с помощью [jq](https://stedolan.github.io/jq/):

```bash
curl -u admin -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/configurations/service_config_versions?service_name=HDFS&service_config_version=1" | jq '.items[].configurations[].properties["fs.defaultFS"] | select(. != null)'
```

> [!NOTE]  
> Эта команда возвращает первую конфигурацию, применяемую к серверу (`service_config_version=1`), которая содержит эти сведения. Может потребоваться перечислить все версии конфигурации, чтобы найти последнюю.

Эта команда возвращает значение, похожее на следующие URI:

* `wasb://<container-name>@<account-name>.blob.core.windows.net`, если используется учетная запись хранения Azure.

    Имя учетной записи — это имя вашей учетной записи хранения Azure. Имя контейнера — это контейнер больших двоичных объектов, который является корнем системы хранения данных кластера.

* `adl://home` при использовании Azure Data Lake Storage. Чтобы получить имя Data Lake Storage, используйте следующий вызов REST:

     ```bash
    curl -u admin -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/configurations/service_config_versions?service_name=HDFS&service_config_version=1" | jq '.items[].configurations[].properties["dfs.adls.home.hostname"] | select(. != null)'
    ```

    Эта команда возвращает следующее имя узла: `<data-lake-store-account-name>.azuredatalakestore.net`.

    Чтобы получить каталог хранилища, назначенный корневым для HDInsight, используйте следующий вызов REST:

    ```bash
    curl -u admin -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/configurations/service_config_versions?service_name=HDFS&service_config_version=1" | jq '.items[].configurations[].properties["dfs.adls.home.mountpoint"] | select(. != null)'
    ```

    Эта команда возвращает путь в следующем формате: `/clusters/<hdinsight-cluster-name>/`.

Сведения о хранилище также можно найти на портале Azure, сделав следующее:

1. На [портале Azure](https://portal.azure.com/)выберите свой кластер HDInsight.

2. В разделе **Свойства** выберите **Учетные записи хранения**. Отобразится информация о хранилище для кластера.

### <a name="how-do-i-access-files-from-outside-hdinsight"></a>Доступ к файлам за пределами HDInsight

Для доступа к данным из-за пределов кластера HDInsight есть несколько методов. Ниже приводятся ссылки на служебные программы и пакеты SDK, которые можно использовать для работы с данными.

Если вы используете __хранилище BLOB-объектов Azure__, ознакомьтесь с информацией о способах получения данных, доступной по приведенным ниже ссылкам.

* [Azure CLI.](/cli/azure/install-az-cli2) это набор команд интерфейса командной строки для работы с Azure. После установки используйте команду `az storage` для получения информации по использованию хранилища, а команду `az storage blob` — для получения информации о больших двоичных объектах.
* [blobxfer.py](https://github.com/Azure/blobxfer): сценарий Python для работы с большими двоичными объектами в службе хранилища Azure.
* Различные пакеты SDK:

    * [Java](https://github.com/Azure/azure-sdk-for-java)
    * [Node.js](https://github.com/Azure/azure-sdk-for-node)
    * [PHP](https://github.com/Azure/azure-sdk-for-php)
    * [Python](https://github.com/Azure/azure-sdk-for-python)
    * [Ruby](https://github.com/Azure/azure-sdk-for-ruby)
    * [.NET](https://github.com/Azure/azure-sdk-for-net)
    * [REST API службы хранилища](/rest/api/storageservices/Blob-Service-REST-API)

Если вы используете __Azure Data Lake Storage 1-го поколения__, ознакомьтесь с информацией о способах получения данных, доступной по приведенным ниже ссылкам.

* [веб-браузер](../data-lake-store/data-lake-store-get-started-portal.md);
* [PowerShell](../data-lake-store/data-lake-store-get-started-powershell.md)
* [Azure CLI](../data-lake-store/data-lake-store-get-started-cli-2.0.md)
* [REST API WebHDFS](../data-lake-store/data-lake-store-get-started-rest-api.md);
* [средства Data Lake для Visual Studio](https://www.microsoft.com/download/details.aspx?id=49504);
* [.NET](../data-lake-store/data-lake-store-get-started-net-sdk.md)
* [Java](../data-lake-store/data-lake-store-get-started-java-sdk.md)
* [Python](../data-lake-store/data-lake-store-get-started-python.md)

## <a name="scaling-your-cluster"></a><a name="scaling"></a>Масштабирование кластера

Масштабирование кластера позволяет динамически изменить число узлов данных в кластере. Операции масштабирования можно выполнять параллельно с другими заданиями и процессами, выполняющимися в кластере.  См. раздел [Масштабирование кластеров](./hdinsight-scaling-best-practices.md).

## <a name="how-do-i-install-hue-or-other-hadoop-component"></a>Как установить Hue (или другой компонент Hadoop)?

HDInsight является управляемой службой. В случае обнаружения проблем с кластером Azure может удалить неисправный узел и создать новый для его замены. Если вы установили компоненты в кластере вручную, при выполнении этой операции они не сохранятся. Вместо этого используйте [действия скрипта HDInsight](hdinsight-hadoop-customize-cluster-linux.md). Действие скрипта может использоваться для внесения следующих изменений:

* Установка и настройка службы или веб-сайта.
* Установка и настройка компонента, для которого необходимо изменить конфигурацию на нескольких узлах кластера.

Действия сценариев — это сценарии Bash. Они выполняются при создании кластера, и их можно использовать для установки и настройки дополнительных компонентов. Сведения о разработке собственных действий скриптов см. в статье [Разработка действий сценариев с помощью HDInsight](hdinsight-hadoop-script-actions-linux.md).

### <a name="jar-files"></a>JAR-файлы

Некоторые технологии Hadoop предоставляют автономные JAR-файлы. Эти файлы содержат функции, которые используются в задании MapReduce либо в Pig или Hive. Они часто не требуют какой-либо настройки и могут передаваться в кластер после его создания для непосредственного использования. Если вы хотите, чтобы компонент сохранился при повторном создании образа кластера, сохраните JAR-файл в хранилище по умолчанию для кластера.

Например, если необходимо использовать последнюю версию [Apache DataFu](https://datafu.incubator.apache.org/), можно скачать JAR-файл, содержащий проект, и отправить его в кластер HDInsight. Далее следуйте инструкциям в документации для DataFu по использованию его в Pig или Hive.

> [!IMPORTANT]  
> Некоторые компоненты, представляющие собой автономные JAR-файлы, предоставляются вместе с HDInsight, но не в пути. Если вам необходим определенный компонент, можно выполнить его поиск в кластере.
>
> ```find / -name *componentname*.jar 2>/dev/null```
>
> Эта команда возвращает путь к соответствующим JAR-файлам.

Чтобы использовать другую версию компонента, отправьте нужную версию и используйте ее в заданиях.

> [!IMPORTANT]
> Компоненты, предоставляемые вместе с кластером HDInsight, поддерживаются в полном объеме. Служба технической поддержки Майкрософт помогает выявить и решить проблемы, связанные с этими компонентами.
>
> Настраиваемые компоненты получают ограниченную коммерчески оправданную поддержку, способствующую дальнейшей диагностике проблемы. В результате проблема может быть устранена, либо вас могут попросить воспользоваться доступными каналами по технологиям с открытым исходным кодом, чтобы связаться с экспертами в данной области. Вы можете использовать сайты сообществ, например: [Страница вопросов и ответов Майкрософт для HDInsight](/answers/topics/azure-hdinsight.html), [https://stackoverflow.com](https://stackoverflow.com). Кроме того, для проектов Apache есть соответствующие сайты ([https://apache.org](https://apache.org)), например [Hadoop](https://hadoop.apache.org/), [Spark](https://spark.apache.org/).

## <a name="next-steps"></a>Дальнейшие действия

* [Управление кластерами HDInsight с помощью Apache Ambari REST API](./hdinsight-hadoop-manage-ambari-rest-api.md)
* [Использование Hive и HiveQL с Hadoop в HDInsight для анализа примера файла Apache log4j](hadoop/hdinsight-use-hive.md)
* [Использование заданий MapReduce с HDInsight](hadoop/hdinsight-use-mapreduce.md)