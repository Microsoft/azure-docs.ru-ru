---
title: Apache Hive библиотек во время создания кластера Azure HDInsight
description: Узнайте, как добавить библиотеки Apache Hive (JAR-файлы) в кластер HDInsight во время создания кластера.
ms.service: hdinsight
ms.topic: how-to
ms.custom: H1Hack27Feb2017,hdinsightactive
ms.date: 02/14/2020
ms.openlocfilehash: b6695e5e985a30d6f912095225c4899e1c910e34
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98945949"
---
# <a name="add-custom-apache-hive-libraries-when-creating-your-hdinsight-cluster"></a>Добавление пользовательских библиотек Apache Hive при создании кластера HDInsight

Узнайте, как предварительно загрузить библиотеки [Apache Hive](https://hive.apache.org/) в HDInsight. Этот документ содержит сведения об использовании действия скрипта по предварительной загрузке библиотек во время создания кластера. Библиотеки, добавленные с помощью действий, описанных в этом документе, глобально доступны в Hive — нет необходимости использовать [Добавление JAR](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Cli) -файла для их загрузки.

## <a name="how-it-works"></a>Принцип работы

При создании кластера можно использовать действие скрипта для изменения узлов кластера по мере их создания. Скрипт в этом документе принимает один параметр, определяющий расположение библиотек. Это расположение должно быть в пределах учетной записи хранилища Azure, а библиотеки должны быть в формате JAR.

Во время создания кластера сценарий перечисляет файлы, копирует их в каталог `/usr/lib/customhivelibs/` на головных и рабочих узлах, а затем добавляет их в свойство `hive.aux.jars.path` в файле `core-site.xml`. Для кластеров под управлением Linux он также обновляет файл `hive-env.sh` расположением файлов.

Использование действия сценария в этой статье делает библиотеки доступными при использовании клиента Hive для **WebHCat** и **HiveServer2**.

## <a name="the-script"></a>Сценарий

**Расположение скрипта**

[https://hdiconfigactions.blob.core.windows.net/setupcustomhivelibsv01/setup-customhivelibs-v01.ps1](https://hdiconfigactions.blob.core.windows.net/setupcustomhivelibsv01/setup-customhivelibs-v01.ps1)

### <a name="requirements"></a>Требования

* Скрипты необходимо применять к **головным узлам** и **рабочим узлам**.

* JAR-файлы, которые нужно установить, должны храниться в хранилище BLOB-объектов Azure в **одном контейнере**.

* Учетная запись хранилища, содержащая библиотеку JAR-файлов, **должна** быть связана с кластером HDInsight во время создания. Она должна быть учетной записью хранения по умолчанию или учетной записью, добавленной с помощью __параметров учетной записи хранения__.

* Путь WASB к контейнеру необходимо указать в виде параметра для действия сценария. Например, если JAR-файлы хранятся в контейнере **libs** в учетной записи хранения **mystorage**, то параметром будет `wasbs://libs@mystorage.blob.core.windows.net/`.

  > [!NOTE]  
  > В этом документе предполагается, что вы уже создали учетную запись хранения и контейнер BLOB-объектов, а также отправили в него файлы.
  >
  > Если учетная запись хранения не создана, создайте ее на [портале Azure](https://portal.azure.com). Затем с помощью служебной программы, например [Azure Storage Explorer](https://storageexplorer.com/), в учетной записи можно создать контейнер и передать в него файлы.

## <a name="create-a-cluster-using-the-script"></a>Создание кластера с помощью сценария

1. Начните подготовку кластера, выполнив действия, описанные в разделе [Подготовка кластеров HDInsight в Linux](hdinsight-hadoop-provision-linux-clusters.md), но не завершение подготовки. Для создания кластера с помощью этого сценария также можно использовать Azure PowerShell или пакет SDK HDInsight для .NET. Дополнительные сведения об использовании этих методов см. в статье [Настройка кластеров HDInsight под управлением Linux с помощью действия сценария](hdinsight-hadoop-customize-cluster-linux.md). Для портал Azure на вкладке **Конфигурация + цены** выберите **действие + Добавить скрипт**.

1. Для **хранилища**, если учетная запись хранения, содержащая библиотеку JAR-файлов, будет отличаться от учетной записи, используемой для кластера, завершите **дополнительные учетные записи хранения**.

1. Для **действий сценария** укажите следующие сведения.

    |Свойство. |Значение |
    |---|---|
    |Тип скрипта|- Custom|
    |Имя|Библиотеки |
    |URI bash-скрипта|`https://hdiconfigactions.blob.core.windows.net/linuxsetupcustomhivelibsv01/setup-customhivelibs-v01.sh`|
    |Типы узлов|Головной, Рабочий|
    |Параметры|введите адрес WASB к контейнеру и учетной записи хранения, содержащий JAR-файлы Например, `wasbs://libs@mystorage.blob.core.windows.net/`.|

    > [!NOTE]
    > Для Apache Spark 2,1 используйте следующий URI Bash: `https://hdiconfigactions.blob.core.windows.net/linuxsetupcustomhivelibsv01/setup-customhivelibs-v00.sh` .

1. Продолжите подготовку кластера, как описано в статье [Создание кластеров Hadoop под управлением Linux в HDInsight](hdinsight-hadoop-provision-linux-clusters.md).

После завершения создания кластера можно использовать JAR, добавленные с помощью этого скрипта из Hive без использования `ADD JAR` инструкции.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о работе с Hive см. в статье [Обзор Apache Hive и HiveQL в Azure HDInsight](hadoop/hdinsight-use-hive.md).
