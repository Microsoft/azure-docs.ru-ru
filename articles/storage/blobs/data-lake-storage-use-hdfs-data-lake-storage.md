---
title: Использование HDFS CLI в хранилище Azure Data Lake Storage 2-го поколения
description: Используйте интерфейс командной строки Hadoop распределенная файловая система (HDFS) для Azure Data Lake Storage 2-го поколения. Создайте контейнер, получите список файлов или каталогов и многое другое.
services: storage
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 12/06/2018
ms.author: normesta
ms.subservice: data-lake-storage-gen2
ms.reviewer: artek
ms.openlocfilehash: b73e79c2f7bbd37b640d3e83d98be68d85ed3b5e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101735375"
---
# <a name="using-the-hdfs-cli-with-data-lake-storage-gen2"></a>Использование HDFS CLI в Data Lake Storage Gen2

Вы можете получать доступ к данным в учетной записи хранения и управлять ими с помощью интерфейса командной строки точно так же, как с [Распределенная файловая система Hadoop (HDFS)](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html). В этой статье приводятся некоторые примеры, которые помогут приступить к работе.

HDInsight предоставляет доступ к распределенному контейнеру, который локально подключен к этим узлам. Доступ к этому контейнеру можно получить с помощью оболочки, которая напрямую взаимодействует с HDFS и другими файловыми системами, поддерживаемыми Hadoop.

Дополнительные сведения об интерфейсе HDFS CLI см. в [официальной документации](https://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-common/FileSystemShell.html) и [руководстве по разрешениям HDFS](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsPermissionsGuide.html) .

>[!NOTE]
>Если вы используете Azure Databricks вместо HDInsight и хотите взаимодействовать с данными с помощью интерфейса командной строки, можно использовать CLI модуля данных для взаимодействия с файловой системой модуля данных. См. раздел [CLI кирпичей](/azure/databricks/dev-tools/cli/).

## <a name="use-the-hdfs-cli-with-an-hdinsight-hadoop-cluster-on-linux"></a>Использование HDFS CLI с кластером HDInsight Hadoop в Linux

Сначала установите [удаленный доступ к службам](../../hdinsight/hdinsight-hadoop-linux-information.md#remote-access-to-services). Если выбрать параметр [SSH](../../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md), образец кода PowerShell будет выглядеть следующим образом.

```powershell
#Connect to the cluster via SSH.
ssh sshuser@clustername-ssh.azurehdinsight.net
#Execute basic HDFS commands. Display the hierarchy.
hdfs dfs -ls /
#Create a sample directory.
hdfs dfs -mkdir /samplefolder
```
Строка подключения находится на портале Azure в колонке кластера HDInsight в разделе "SSH и вход в кластер". Учетные данные SSH были указаны во время создания кластера.

>[!IMPORTANT]
>Начисление оплаты начинается после создания кластера HDInsight и прекращается только после его удаления. Кластеры оплачиваются поминутно, поэтому всегда следует удалять кластер, когда он больше не нужен. Сведения об удалении кластера см. в [статье на эту тему](../../hdinsight/hdinsight-delete-cluster.md). Данные, размещенные в учетной записи хранения с поддержкой Data Lake Storage 2-го поколения, сохраняются даже после удаления кластера HDInsight.

## <a name="create-a-container"></a>Создание контейнера

`hdfs dfs -D "fs.azure.createRemoteFileSystemDuringInitialization=true" -ls abfs://<container-name>@<storage-account-name>.dfs.core.windows.net/`

* Замените заполнитель `<container-name>` именем, которое хотите присвоить своему контейнеру.

* Замените заполнитель `<storage-account-name>` именем вашей учетной записи хранения.

## <a name="get-a-list-of-files-or-directories"></a>Получение списка файлов или каталогов

`hdfs dfs -ls <path>`

Замените `<path>` заполнитель URI контейнера или папки контейнера.

Пример: `hdfs dfs -ls abfs://my-file-system@mystorageaccount.dfs.core.windows.net/my-directory-name`

## <a name="create-a-directory"></a>Создание каталога

`hdfs dfs -mkdir [-p] <path>`

Замените `<path>` заполнитель именем корневого контейнера или папкой в контейнере.

Пример: `hdfs dfs -mkdir abfs://my-file-system@mystorageaccount.dfs.core.windows.net/`

## <a name="delete-a-file-or-directory"></a>Удаление файла или каталога

`hdfs dfs -rm <path>`

Замените заполнитель `<path>` на URI файла или папки, которые нужно удалить.

Пример: `hdfs dfs -rmdir abfs://my-file-system@mystorageaccount.dfs.core.windows.net/my-directory-name/my-file-name`

## <a name="display-the-access-control-lists-acls-of-files-and-directories"></a>Отображение списков управления доступом (ACL) файлов и каталогов

`hdfs dfs -getfacl [-R] <path>`

Пример

`hdfs dfs -getfacl -R /dir`

См. [getfacl](https://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-common/FileSystemShell.html#getfacl)

## <a name="set-acls-of-files-and-directories"></a>Настройка ACL файлов и каталогов

`hdfs dfs -setfacl [-R] [-b|-k -m|-x <acl_spec> <path>]|[--set <acl_spec> <path>]`

Пример

`hdfs dfs -setfacl -m user:hadoop:rw- /file`

См. [setfacl](https://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-common/FileSystemShell.html#setfacl)

## <a name="change-the-owner-of-files"></a>Изменение владельца файлов

`hdfs dfs -chown [-R] <new_owner>:<users_group> <URI>`

См. [chown](https://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-common/FileSystemShell.html#chown)

## <a name="change-group-association-of-files"></a>Изменение связи группы файлов

`hdfs dfs -chgrp [-R] <group> <URI>`

См. [chgrp](https://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-common/FileSystemShell.html#chgrp)

## <a name="change-the-permissions-of-files"></a>Изменение разрешений файлов

`hdfs dfs -chmod [-R] <mode> <URI>`

См. [chmod](https://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-common/FileSystemShell.html#chmod)

Полный список команд можно просмотреть на веб-сайте [Apache Hadoop 2.4.1 File System Guide](https://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-common/FileSystemShell.html).

## <a name="next-steps"></a>Дальнейшие действия

* [Краткое руководство. Запуск задания Spark в Azure Databricks с помощью портала Azure](./data-lake-storage-use-databricks-spark.md)

* [Сведения о списках управления доступом к файлам и каталогам](./data-lake-storage-access-control.md)