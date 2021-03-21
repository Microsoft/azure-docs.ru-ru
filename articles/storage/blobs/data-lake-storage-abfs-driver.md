---
title: Драйвер файловой системы больших двоичных объектов Azure для хранилища Azure Data Lake Storage 2-го поколения
description: Узнайте о драйвере файловой системы BLOB-объектов Azure (АБФС), выделенном драйвере службы хранилища Azure для Hadoop. Доступ к данным в Azure Data Lake Storage 2-го поколения с помощью этого драйвера.
author: normesta
ms.topic: conceptual
ms.author: normesta
ms.reviewer: jamesbak
ms.date: 12/06/2018
ms.service: storage
ms.subservice: data-lake-storage-gen2
ms.openlocfilehash: 54c6245ea5290e2cab0efcd52f2208ba874c0848
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101720449"
---
# <a name="the-azure-blob-filesystem-driver-abfs-a-dedicated-azure-storage-driver-for-hadoop"></a>Драйвер файловой системы больших двоичных объектов Azure (ABFS): выделенный драйвер хранилища Azure для Hadoop

Один из основных способов доступа к данным в хранилище Azure Data Lake Storage 2-го поколения связан с использованием [файловой системы Hadoop](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/filesystem/index.html). В Data Lake Storage 2-го поколения пользователи хранилища BLOB-объектов Azure могут получить доступ к новому драйверу, драйверу файловой системы BLOB-объектов Azure или `ABFS`. ABFS является частью Apache Hadoop и входит во многие коммерческие дистрибутивы Hadoop. При использовании этого драйвера многие приложения и платформы могут получить доступ к данным в хранилище BLOB-объектов Azure2 без какого-либо кода, явно ссылающегося на Data Lake Storage 2-го поколения.

## <a name="prior-capability-the-windows-azure-storage-blob-driver"></a>Предыдущая возможность: драйвер Windows Azure Storage Blob

Драйвер Windows Azure Storage Blob или [WASB](https://hadoop.apache.org/docs/current/hadoop-azure/index.html) предоставлял оригинальную поддержку хранилища BLOB-объектов. Этот драйвер выполнял сложную задачу сопоставления семантики файловой системы (как это требуется интерфейсом файловой системы Hadoop) с семантикой интерфейса стиля хранилища объектов, предоставляемого хранилищем BLOB-объектов Azure. Этот драйвер поддерживает эту модель, обеспечивая высокую производительность доступа к данным, хранящимся в больших двоичных объектах, но содержит значительный объем кода, выполняющего это сопоставление, что усложняет его обслуживание. Кроме того, при некоторых применяемых к каталогам операциях, таких как [FileSystem.rename()](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/filesystem/filesystem.html#boolean_renamePath_src_Path_d) и [FileSystem.delete()](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/filesystem/filesystem.html#boolean_deletePath_p_boolean_recursive), требуется, чтобы драйвер выполнил большое количество операций (из-за того, что объекты хранилища не поддерживают каталоги), что часто приводит к снижению производительности. Драйвер ABFS разработан для преодоления присущих WASB недостатков.

## <a name="the-azure-blob-file-system-driver"></a>Драйвер файловой системы больших двоичных объектов Azure

[Интерфейс REST хранилища Azure Data Lake Storage](/rest/api/storageservices/data-lake-storage-gen2) поддерживает семантику файловой системы с помощью хранилища BLOB-объектов Azure. Учитывая, что файловая система Hadoop предназначена для поддержки той же семантики, сложное сопоставление в драйвере не требуется. Таким образом, драйвер файловой системы больших двоичных объектов Azure (или ABFS) является простой оболочкой клиента для интерфейса REST API.

Однако существуют некоторые функции, которые драйвер все же должен выполнять:

### <a name="uri-scheme-to-reference-data"></a>Схема URI для ссылки на данные

Как и другие реализации файловой системы в Hadoop, драйвер ABFS определяет свою собственную схему URI, чтобы к ресурсам (каталоги и файлы) можно было прямо обращаться. Схему URI см. в статье [Использование URI в хранилище Azure Data Lake Storage Gen2](./data-lake-storage-introduction-abfs-uri.md). URI имеет такую структуру: `abfs[s]://file_system@account_name.dfs.core.windows.net/<path>/<path>/<file_name>`

Применяя вышеуказанный формат URI, стандартные инструменты и платформы Hadoop могут использоваться для ссылки на эти ресурсы:

```bash
hdfs dfs -mkdir -p abfs://fileanalysis@myanalytics.dfs.core.windows.net/tutorials/flightdelays/data
hdfs dfs -put flight_delays.csv abfs://fileanalysis@myanalytics.dfs.core.windows.net/tutorials/flightdelays/data/
```

На внутреннем уровне драйвер ABFS преобразует ресурсы, указанные в URI, в файлы и каталоги и вызывает REST API хранилища Azure Data Lake Storage с использованием этих ссылок.

### <a name="authentication"></a>Аутентификация

Драйвер ABFS поддерживает две формы проверки подлинности, поэтому приложение Hadoop может безопасно обращаться к ресурсам, содержащимся в учетной записи, совместимой с Data Lake Storage 2-го поколения. Дополнительные сведения о схемах проверки подлинности приведены в [Руководстве по безопасности службы хранилища Azure](security-recommendations.md). Их можно сформулировать следующим образом.

- **Общий ключ** позволяет пользователям получать доступ ко всем ресурсам в учетной записи. Ключ шифруется и сохраняется в конфигурации Hadoop.

- **Токен носителя OAuth Azure Active Directory**. Токен носителя Azure AD приобретается и обновляется с помощью драйвера, используя либо идентификатор пользователя, либо настроенный субъект-службу. С помощью этой модели проверки подлинности доступ разрешен для каждого вызова, использующего идентификатор, который связан с предоставленным токеном и оценивается с помощью назначенного списка управления доступом POSIX (ACL).

   > [!NOTE]
   > Azure Data Lake Storage 2-го поколения поддерживает только конечные точки Azure AD версии 1.0.

### <a name="configuration"></a>Конфигурация

Все настройки драйвера ABFS хранятся в файле конфигурации <code>core-site.xml</code>. В дистрибутивах Hadoop с [Ambarі](https://ambari.apache.org/) конфигурацией также можно управлять с помощью веб-портала или REST API Ambari.

Подробная информация обо всех поддерживаемых элементах конфигурации указана в [официальной документации по Hadoop](https://hadoop.apache.org/docs/stable/hadoop-azure/abfs.html).

### <a name="hadoop-documentation"></a>Документация Hadoop

Все сведения о драйвере ABFS см. в [официальной документации по Hadoop](https://hadoop.apache.org/docs/stable/hadoop-azure/abfs.html).

## <a name="next-steps"></a>Дальнейшие действия

- [Создание кластера Azure Databricks](./data-lake-storage-use-databricks-spark.md)
- [Использование URI в Azure Data Lake Storage Gen2](./data-lake-storage-introduction-abfs-uri.md)