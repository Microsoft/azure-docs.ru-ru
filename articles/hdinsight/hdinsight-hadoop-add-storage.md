---
title: Добавление дополнительных учетных записей хранения Azure в HDInsight
description: Узнайте, как добавить дополнительные учетные записи хранения Azure в существующий кластер HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.custom: seoapr2020
ms.date: 04/27/2020
ms.openlocfilehash: 32757fef131c5e443350f032c0ac987d7f491396
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104864318"
---
# <a name="add-additional-storage-accounts-to-hdinsight"></a>Добавление дополнительных учетных записей хранения в HDInsight

Узнайте, как использовать действия сценария для добавления дополнительных *учетных записей* хранения Azure в HDInsight. Действия, описанные в этом документе, позволяют добавить *учетную запись* хранения в существующий кластер HDInsight. Эта статья относится к *учетным записям* хранения (а не к учетной записи хранения кластера по умолчанию), а не к дополнительным хранилищам, таким как [`Azure Data Lake Storage Gen1`](hdinsight-hadoop-use-data-lake-storage-gen1.md) и [`Azure Data Lake Storage Gen2`](hdinsight-hadoop-use-data-lake-storage-gen2.md) .

> [!IMPORTANT]  
> Сведения в этом документе посвящены добавлению дополнительных учетных записей хранения в кластер после его создания. Сведения о добавлении учетных записей хранения во время создания кластера см. в статье о [настройке кластеров HDInsight с использованием Apache Hadoop, Apache Spark, Apache Kafka и других платформ](hdinsight-hadoop-provision-linux-clusters.md).

## <a name="prerequisites"></a>Предварительные требования

* Кластер Hadoop в HDInsight. Ознакомьтесь со статьей [Краткое руководство. Использование Apache Hadoop и Apache Hive в Azure HDInsight с шаблоном Resource Manager](./hadoop/apache-hadoop-linux-tutorial-get-started.md).
* Имя и ключ учетной записи хранения. См. [раздел Управление ключами доступа учетной записи хранения](../storage/common/storage-account-keys-manage.md).
* При использовании PowerShell вам потребуется модуль AZ.  См. [обзор Azure PowerShell](/powershell/azure/).

## <a name="how-it-works"></a>Принцип работы

Во время обработки скрипт выполняет следующие действия:

* Если учетная запись хранения уже существует в конфигурации core-site.xml кластера, сценарий завершает работу и дальнейшие действия не выполняются.

* Проверяет существование учетной записи хранения и ее доступность с помощью ключа.

* Шифрует ключ с использованием учетных данных кластера.

* Добавляет учетную запись хранилища в файл core-site.xml.

* Останавливает и перезапускает службы Apache Oozie, Apache Hadoop YARN, Apache Hadoop MapReduce2 и Apache Hadoop HDFS. Остановка и запуск этих служб позволяет им использовать новую учетную запись хранения.

> [!WARNING]  
> Использование учетной записи хранения, расположение которой отличается от расположения кластера HDInsight, не поддерживается.

## <a name="add-storage-account"></a>Добавление учетной записи хранения

Используйте [действие скрипта](hdinsight-hadoop-customize-cluster-linux.md#script-action-to-a-running-cluster) , чтобы применить изменения со следующими соображениями.

|Свойство | Значение |
|---|---|
|URI bash-скрипта|`https://hdiconfigactions.blob.core.windows.net/linuxaddstorageaccountv01/add-storage-account-v01.sh`|
|Типы узлов|Head|
|Параметры|`ACCOUNTNAME``ACCOUNTKEY` `-p` (необязательно)|

* `ACCOUNTNAME` имя учетной записи хранения, добавляемой в кластер HDInsight.
* `ACCOUNTKEY` — Это ключ доступа для `ACCOUNTNAME` .
* Аргумент `-p` является необязательным. Если этот параметр указан, ключ не шифруется и хранится в файле core-site.xml в виде обычного текста.

## <a name="verification"></a>Проверка

При просмотре кластера HDInsight в портал Azure при выборе элемента " __учетные записи хранения__ " в разделе " __свойства__ " не отображаются учетные записи хранения, добавленные с помощью этого действия скрипта. Azure PowerShell и Azure CLI не отображают дополнительную учетную запись хранения. Сведения о хранилище не отображаются, так как сценарий изменяет только `core-site.xml` конфигурацию кластера. Эта информация не используется при получении сведений о кластере с помощью API-интерфейсов управления Azure.

Чтобы проверить дополнительное хранилище, используйте один из методов, показанных ниже.

### <a name="powershell"></a>PowerShell

Скрипт вернет имена учетных записей хранения, связанные с данным кластером. Замените `CLUSTERNAME` фактическим именем кластера, а затем запустите скрипт.

```powershell
# Update values
$clusterName = "CLUSTERNAME"

$creds = Get-Credential -UserName "admin" -Message "Enter the cluster login credentials"

$clusterName = $clusterName.ToLower();

# getting service_config_version
$resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName`?fields=Clusters/desired_service_config_versions/HDFS" `
    -Credential $creds -UseBasicParsing
$respObj = ConvertFrom-Json $resp.Content

$configVersion=$respObj.Clusters.desired_service_config_versions.HDFS.service_config_version

$resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/configurations/service_config_versions?service_name=HDFS&service_config_version=$configVersion" `
    -Credential $creds
$respObj = ConvertFrom-Json $resp.Content

# extract account names
$value = ($respObj.items.configurations | Where type -EQ "core-site").properties | Get-Member -membertype properties | Where Name -Like "fs.azure.account.key.*"
foreach ($name in $value ) { $name.Name.Split(".")[4]}
```

### <a name="apache-ambari"></a>Apache Ambari

1. В веб-браузере перейдите на страницу `https://CLUSTERNAME.azurehdinsight.net`, где `CLUSTERNAME` — это имя вашего кластера.

1. Перейдите в раздел  >  **конфигурации** HDFS  >  **Расширенные**  >  **пользовательские основные компоненты — сайт**.

1. Обратите внимание на ключи, которые начинаются с `fs.azure.account.key` . Имя учетной записи будет частью ключа, как показано в этом образце образа:

   :::image type="content" source="./media/hdinsight-hadoop-add-storage/apache-ambari-verification.png" alt-text="Проверка с помощью Apache Ambari":::

## <a name="remove-storage-account"></a>Удаление учетной записи хранения

1. В веб-браузере перейдите на страницу `https://CLUSTERNAME.azurehdinsight.net`, где `CLUSTERNAME` — это имя вашего кластера.

1. Перейдите в раздел  >  **конфигурации** HDFS  >  **Расширенные**  >  **пользовательские основные компоненты — сайт**.

1. Удалите следующие разделы:
    * `fs.azure.account.key.<STORAGE_ACCOUNT_NAME>.blob.core.windows.net`
    * `fs.azure.account.keyprovider.<STORAGE_ACCOUNT_NAME>.blob.core.windows.net`

После удаления этих разделов и сохранения конфигурации необходимо перезапустить Oozie, Yarn, MapReduce2, HDFS и Hive по одному.

## <a name="known-issues"></a>Известные проблемы

### <a name="storage-firewall"></a>Брандмауэр хранилища

Если вы решили защитить учетную запись хранения с ограничениями **брандмауэра и виртуальной сети** в **выбранных сетях**, обязательно включите исключение **Разрешить доверенные службы Майкрософт...** , чтобы HDInsight могла получить доступ к вашей учетной записи хранения.`.`

### <a name="unable-to-access-storage-after-changing-key"></a>Не удается получить доступ к хранилищу после изменения ключа

Если вы измените ключ для учетной записи хранения, HDInsight больше не сможет обращаться к этой учетной записи. HDInsight использует кэшированную копию ключа в core-site.xml для кластера. Эту копию нужно обновить в соответствии с новым ключом.

При повторном выполнении действия скрипта ключ **не** обновляется, так как сценарий проверяет, существует ли уже запись для учетной записи хранения. Если запись уже существует, изменения не вносятся.

Чтобы обойти эту проблему:  
1. Удалите учетную запись хранения.
1. Добавьте учетную запись хранения.

> [!IMPORTANT]  
> Смена ключа хранилища для основной учетной записи хранения, подключенной к кластеру, не поддерживается.

## <a name="next-steps"></a>Дальнейшие действия

Вы узнали, как добавлять дополнительные учетные записи хранения в существующий кластер HDInsight. Дополнительные сведения о действиях скриптов см. в статье [Настройка кластеров HDInsight под управлением Linux с помощью действия сценария](hdinsight-hadoop-customize-cluster-linux.md).