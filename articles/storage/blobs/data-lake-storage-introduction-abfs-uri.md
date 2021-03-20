---
title: Использование URI в Azure Data Lake Storage Gen2
description: Сведения о синтаксисе URI для идентификатора схемы абфс, который представляет драйвер файловой системы больших двоичных объектов Azure (драйвер FileSystem для Azure Data Lake Storage 2-го поколения).
author: normesta
ms.topic: conceptual
ms.author: normesta
ms.date: 12/06/2018
ms.service: storage
ms.subservice: data-lake-storage-gen2
ms.reviewer: jamesbak
ms.openlocfilehash: 948b5aa0ad015f9f3c693e13219ec034724687c0
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95913170"
---
# <a name="use-the-azure-data-lake-storage-gen2-uri"></a>Использование URI в Azure Data Lake Storage Gen2

Драйвер [файловой системы Hadoop](https://www.aosabook.org/en/hdfs.html), совместимый с Azure Data Lake Storage 2-го поколения, обозначается идентификатором схемы `abfs` (файловая система BLOB-объектов Azure). Драйвер ABFS совместим с другими драйверами файловой системы Hadoop и использует формат URI для обращения к файлами и каталогам в учетных записях, поддерживающих Data Lake Storage Gen2.

## <a name="uri-syntax"></a>Синтаксис URI

Синтаксис URI для Data Lake Storage Gen2 зависит от того, настроена ли служба Data Lake Storage Gen2 в качестве файловой системы по умолчанию в вашей учетной записи хранения.

Если вы хотите обратиться к учетной записи, которая поддерживает Data Lake Storage 2-го поколения и при создании которой эта файловая система **не** была настроена как файловая система по умолчанию, используется сокращенный URI со следующим синтаксисом:

<pre>abfs[s]<sup>1</sup>://&lt;file_system&gt;<sup>2</sup>@&lt;account_name&gt;<sup>3</sup>.dfs.core.windows.net/&lt;path&gt;<sup>4</sup>/&lt;file_name&gt;<sup>5</sup></pre>

1. **Scheme identifier** (Идентификатор схемы) — указывает протокол `abfs`. Можно установить соединение с протоколом TLS или без него, ранее известное как SSL (SSL), Connection. Используйте `abfss` для подключения с помощью TLS-подключения.

2. **File system** (Файловая система) — указывает на родительское расположение, в котором хранятся все файлы и папки. Этот параметр аналогичен контейнерам в службе Azure Storage Blob.

3. **Account name** (Имя учетной записи) — содержит имя, которое вы присвоили учетной записи хранения при ее создании.

4. **Paths** (Пути) — содержит представление структуры каталогов, в котором сегменты разделяются косыми чертами (`/`).

5. **Имя файла**: имя отдельного файла. Этот параметр можно не указывать, если вы обращаетесь к каталогу.

Если при создании учетной записи, к которой нужно обратиться, эта файловая система была настроена как файловая система по умолчанию, используется сокращенный URI со следующим синтаксисом:

<pre>/&lt;path&gt;<sup>1</sup>/&lt;file_name&gt;<sup>2</sup></pre>

1. **Path** (Путь) — содержит представление структуры каталогов, в котором сегменты разделяются косыми чертами (`/`).

2. **File Name** (Имя файла) — содержит имя конкретного файла.


## <a name="next-steps"></a>Дальнейшие действия

- [Использование Azure Data Lake Storage Gen2 с кластерами Azure HDInsight](../../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)