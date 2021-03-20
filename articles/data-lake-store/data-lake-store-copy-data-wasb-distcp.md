---
title: Копирование данных из WASB в Azure Data Lake Storage 1-го поколения с помощью DistCp и обратно
description: Используйте средство DistCp для копирования данных в большие двоичные объекты службы хранилища Azure и обратно в Azure Data Lake Storage 1-го поколения
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 01/03/2020
ms.author: twooley
ms.openlocfilehash: c608f357eb1eff9fd36e583b98d26250a71cb923
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85515676"
---
# <a name="use-distcp-to-copy-data-between-azure-storage-blobs-and-azure-data-lake-storage-gen1"></a>Использование DistCp для копирования данных между большими двоичными объектами службы хранилища Azure и Azure Data Lake Storage 1-го поколения

> [!div class="op_single_selector"]
> * [С помощью DistCp](data-lake-store-copy-data-wasb-distcp.md)
> * [С помощью AdlCopy](data-lake-store-copy-data-azure-storage-blob.md)
>
>

Если у вас есть кластер HDInsight с доступом к Azure Data Lake Storage 1-го поколения, вы можете использовать средства экосистемы Hadoop, такие как DistCp, для копирования данных в хранилище кластера HDInsight (WASB) и обратно в учетную запись Data Lake Storage 1-го поколения. В этой статье показано, как использовать средство DistCp.

## <a name="prerequisites"></a>Предварительные требования

* **Подписка Azure**. См. страницу [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).
* **Учетная запись Azure Data Lake Storage 1-го поколения**. За инструкциями по созданию учетной записи обращайтесь к статье [Начало работы с Azure Data Lake Storage 1-го поколения](data-lake-store-get-started-portal.md).
* **Кластер Azure HDInsight** с доступом к учетной записи Data Lake Storage 1-го поколения. Дополнительные сведения см. в статье [Создание кластеров HDInsight, использующих Data Lake Store, с помощью портала Azure](data-lake-store-hdinsight-hadoop-use-portal.md). Убедитесь, что вы включили удаленный рабочий стол для кластера.

## <a name="use-distcp-from-an-hdinsight-linux-cluster"></a>Использование Distcp из кластера HDInsight на платформе Linux

HDInsight An кластер поставляется с инструментом DistCp, который можно использовать для копирования данных из разных источников в кластер HDInsight. Если вы настроили кластер HDInsight для использования Data Lake Storage 1-го поколения в качестве дополнительного хранилища, можно использовать встроенную DistCp для копирования данных в учетную запись Data Lake Storage 1-го поколения и обратно. В этом разделе мы рассмотрим, как использовать средство DistCp.

1. На своем настольном компьютере используйте SSH, чтобы подключиться к кластеру. См. раздел [Подключение к кластеру HDInsight на основе Linux](../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md). Выполните команды в командной строке SSH.

1. Проверьте, есть ли у вас доступ к BLOB-объектам службы хранилища Azure (WASB). Выполните следующую команду.

   ```
   hdfs dfs –ls wasb://<container_name>@<storage_account_name>.blob.core.windows.net/
   ```

   Выходные данные содержат список содержимого в большом двоичном объекте хранилища.

1. Аналогичным образом проверьте, доступна ли учетная запись хранилища Data Lake Storage 1-го поколения из кластера. Выполните следующую команду.

   ```
   hdfs dfs -ls adl://<data_lake_storage_gen1_account>.azuredatalakestore.net:443/
   ```

    Выходные данные содержат список файлов и папок в учетной записи Data Lake Storage 1-го поколения.

1. Используйте DistCp для копирования данных из WASB в учетную запись Data Lake Storage 1-го поколения.

   ```
   hadoop distcp wasb://<container_name>@<storage_account_name>.blob.core.windows.net/example/data/gutenberg adl://<data_lake_storage_gen1_account>.azuredatalakestore.net:443/myfolder
   ```

    Эта команда копирует содержимое папки **/example/data/gutenberg/** в WASB в папку **/myfolder** в учетной записи Data Lake Storage 1-го поколения.

1. Аналогичным образом используйте DistCp для копирования данных из учетной записи Data Lake Storage 1-го поколения в WASB.

   ```
   hadoop distcp adl://<data_lake_storage_gen1_account>.azuredatalakestore.net:443/myfolder wasb://<container_name>@<storage_account_name>.blob.core.windows.net/example/data/gutenberg
   ```

    Команда копирует содержимое **папку/MyFolder** в папку Data Lake Storage 1-го поколения Account to **/example/Data/Gutenberg/** в WASB.

## <a name="performance-considerations-while-using-distcp"></a>Рекомендации по производительности при использовании DistCp

Так как самый низкий уровень детализации инструмента DistCp — это один файл, установка максимального количества одновременных копий является наиболее важным параметром, чтобы оптимизировать его относительно Data Lake Storage 1-го поколения. Количество одновременных копий можно контролировать, задав параметр число модулей сопоставления (m ') в командной строке. Этот параметр указывает максимальное количество модулей сопоставления, которые используются для копирования данных. Значение по умолчанию — 20.

Пример

```
 hadoop distcp wasb://<container_name>@<storage_account_name>.blob.core.windows.net/example/data/gutenberg adl://<data_lake_storage_gen1_account>.azuredatalakestore.net:443/myfolder -m 100
```

### <a name="how-to-determine-the-number-of-mappers-to-use"></a>Как определить количество используемых модулей сопоставления

Ниже представлены некоторые полезные рекомендации.

* **Шаг 1. Определение общего объема памяти YARN.** Первым шагом является определение объема памяти YARN, доступной для кластера, в котором выполняется задание DistCp. Эта информация доступна на портале Ambari, связанном с кластером. Перейдите по адресу YARN и просмотрите вкладку **configs (конфигурации** ), чтобы увидеть память YARN. Чтобы определить общий объем памяти YARN, умножьте объем памяти YARN для одного узла на количество узлов в кластере.

* **Шаг 2. Расчет количества модулей сопоставления.** Чтобы узнать значение **m**, разделите целую часть общего объема памяти YARN на размер контейнера YARN. Сведения о размере контейнера YARN также доступны на портале Ambari. Перейдите по адресу YARN и просмотрите вкладку **configs (конфигурации** ). В этом окне отображается размер контейнера YARN. Уравнение, которое прибывает в числе модулей сопоставления (**m**),:

   `m = (number of nodes * YARN memory for each node) / YARN container size`

Пример

Предположим, что в кластере имеется четыре узла узла d14v2s, и вы хотите переместить 10 ТБ данных из 10 разных папок. Эти папки содержат различные объемы данных, и размеры файлов в каждой папке отличаются.

* Общий объем памяти YARN. На портале Ambari вы узнали, что объем памяти YARN составляет 96 ГБ для узла D14. Таким образом, общий объем памяти YARN для кластера из четырех узлов равен:

   `YARN memory = 4 * 96GB = 384GB`

* Число модулей сопоставления. На портале Ambari вы узнали, что размер контейнера YARN равен 3072 для узла кластера D14. Таким образом, число модулей сопоставления:

   `m = (4 nodes * 96GB) / 3072MB = 128 mappers`

Если другие приложения используют память, можно выбрать использование только части памяти YARN кластера для DistCp.

### <a name="copying-large-datasets"></a>Копирование больших наборов данных

Если размер перемещаемого набора данных велик (например, > 1 ТБ) или существует много разных папок, рассмотрите возможность использования нескольких заданий DistCp. Скорее всего, производительность не повышается, но она распределяет задания таким образом, чтобы при сбое какого-либо задания необходимо перезапустить это задание вместо всего задания.

### <a name="limitations"></a>Ограничения

* DistCp пытается создать модули сопоставления одинакового размера для оптимизации производительности. Увеличение количества модулей сопоставления не всегда приводит к увеличению производительности.

* DistCp ограничивается только одним модулем сопоставления для каждого файла. Поэтому у вас не должно быть больше модулей сопоставления, чем у вас есть файлы. Так как DistCp может назначить только один модуль сопоставления для файла, это ограничивает объем параллелизма, который может использоваться для копирования больших файлов.

* При наличии небольшого числа больших файлов разделите их на фрагменты файлов размером 256 МБ, чтобы обеспечить более возможный параллелизм.

* Если выполняется копирование из учетной записи хранилища BLOB-объектов Azure, задание копирования может регулироваться на стороне хранилища BLOB-объектов. Это снижает производительность задания копирования. Дополнительные сведения об ограничениях хранилища BLOB-объектов Azure см. в статье ограничения хранилища Azure в [подписке Azure и ограничениях службы](../azure-resource-manager/management/azure-subscription-service-limits.md).

## <a name="see-also"></a>См. также раздел

* [Копирование данных из больших двоичных объектов службы хранилища Azure в Data Lake Storage 1-го поколения](data-lake-store-copy-data-azure-storage-blob.md)
* [Защита данных в Data Lake Storage Gen1](data-lake-store-secure-data.md)
* [Начало работы с Azure Data Lake Analytics с помощью портала Azure](../data-lake-analytics/data-lake-analytics-get-started-portal.md)
* [Создание кластеров HDInsight, использующих Data Lake Store, с помощью портала Azure](data-lake-store-hdinsight-hadoop-use-portal.md)
