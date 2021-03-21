---
title: Копирование данных из больших двоичных объектов службы хранилища Azure в Data Lake Storage 1-го поколения
description: Использование средства AdlCopy для копирования данных из больших двоичных объектов хранилища Azure в Azure Data Lake Storage 1-го поколения
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.openlocfilehash: 84ee65b05af4393f49696875bda41df39e283d5d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85980095"
---
# <a name="copy-data-from-azure-storage-blobs-to-azure-data-lake-storage-gen1"></a>Копирование данных из больших двоичных объектов службы хранилища Azure в Azure Data Lake Storage 1-го поколения

> [!div class="op_single_selector"]
> * [С помощью DistCp](data-lake-store-copy-data-wasb-distcp.md)
> * [С помощью AdlCopy](data-lake-store-copy-data-azure-storage-blob.md)
>
>

Data Lake Storage 1-го поколения предоставляет программу командной строки [AdlCopy](https://www.microsoft.com/download/details.aspx?id=50358)для копирования данных из следующих источников:

* Из больших двоичных объектов службы хранилища Azure в Data Lake Storage 1-го поколения. AdlCopy нельзя использовать для копирования данных из Data Lake Storage 1-го поколения в большие двоичные объекты службы хранилища Azure.
* Между двумя учетными записями Data Lake Storage 1-го поколения.

Также можно использовать инструмент AdlCopy в двух различных режимах:

* **Автономный**, при котором для выполнения задачи используются ресурсы Data Lake Storage 1-го поколения.
* **С помощью учетной записи аналитики озера данных**, при котором для выполнения операции копирования используются единицы, назначенные учетной записи аналитики озера данных. Этот вариант можно использовать, если вы хотите, чтобы задачи копирования выполнялись предсказуемым образом.

## <a name="prerequisites"></a>Предварительные требования

Перед началом работы с этой статьей необходимо иметь следующее:

* **Подписка Azure**. См. страницу [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).
* Контейнер **больших двоичных объектов службы хранилища Azure** с некоторыми данными.
* **Учетная запись Data Lake Storage 1-го поколения**. Инструкции по ее созданию см. в статье [Приступая к работе с Azure Data Lake Storage 1-го поколения](data-lake-store-get-started-portal.md)
* **Учетная запись Data Lake Analytics (необязательно)** . инструкции по созданию учетной записи Data Lake Analytics см. в статье [Приступая к работе с Azure Data Lake Analytics](../data-lake-analytics/data-lake-analytics-get-started-portal.md) .
* **Средство AdlCopy**. Установите [средство AdlCopy](https://www.microsoft.com/download/details.aspx?id=50358).

## <a name="syntax-of-the-adlcopy-tool"></a>Синтаксис средства AdlCopy

Используйте следующий синтаксис для работы со средством AdlCopy

```console
AdlCopy /Source <Blob or Data Lake Storage Gen1 source> /Dest <Data Lake Storage Gen1 destination> /SourceKey <Key for Blob account> /Account <Data Lake Analytics account> /Units <Number of Analytics units> /Pattern
```

Параметры синтаксиса описаны ниже:

| Параметр | Описание |
| --- | --- |
| Источник |Указывает расположение исходных данных в хранилище больших двоичных объектов Azure. Источником может быть контейнер больших двоичных объектов, сам большой двоичный объект или другая учетная запись Data Lake Storage 1-го поколения. |
| Dest |Указывает целевое назначение Data Lake Storage 1-го поколения для копирования. |
| SourceKey |Указывает ключ доступа к хранилищу для источника — хранилища больших двоичных объектов Azure. Это необходимо только в том случае, если источником является контейнер больших двоичных объектов или сам большой двоичный объект. |
| Учетная запись |**Необязательно**. Используйте этот вариант, если хотите, чтобы для запуска задания копирования использовалась учетная запись аналитики озера данных Azure. Если при запуске средства указан параметр /Account, но не указана учетная запись аналитики озера данных Azure, то для запуска задания AdlCopy использует учетную запись по умолчанию. Кроме того, при использовании этого параметра необходимо добавить источник (хранилище больших двоичных объектов Azure) и место назначения (Azure Data Lake Storage 1-го поколения) в качестве источников данных для вашей учетной записи Data Lake Analytics. |
| единиц(ы) |Указывает количество единиц аналитики озера данных, которые будут использоваться для задания копирования. Этот параметр является обязательным, если вы указываете учетную запись аналитики озера данных с помощью параметра **/Account** . |
| Шаблон |Определяет шаблон регулярного выражения, который указывает, какие именно большие двоичные объекты и файлы нужно копировать. В AdlCopy используется сопоставление с учетом регистра. Если шаблон не указан, то для копирования всех элементов используется шаблон по умолчанию. Задание нескольких шаблонов не поддерживается. |

## <a name="use-adlcopy-as-standalone-to-copy-data-from-an-azure-storage-blob"></a>Использование AdlCopy (автономно) для копирования данных из большого двоичного объекта службы хранилища Azure

1. Откройте командную строку и перейдите в каталог, в который установлено средство AdlCopy, обычно `%HOMEPATH%\Documents\adlcopy`.
1. Выполните следующую команду, чтобы скопировать заданный большой двоичный объект из контейнера-источника в папку Data Lake Storage 1-го поколения:

    ```console
    AdlCopy /source https://<source_account>.blob.core.windows.net/<source_container>/<blob name> /dest swebhdfs://<dest_adlsg1_account>.azuredatalakestore.net/<dest_folder>/ /sourcekey <storage_account_key_for_storage_container>
    ```

    Пример:

    ```console
    AdlCopy /source https://mystorage.blob.core.windows.net/mycluster/HdiSamples/HdiSamples/WebsiteLogSampleData/SampleLog/909f2b.log /dest swebhdfs://mydatalakestorage.azuredatalakestore.net/mynewfolder/ /sourcekey uJUfvD6cEvhfLoBae2yyQf8t9/BpbWZ4XoYj4kAS5Jf40pZaMNf0q6a8yqTxktwVgRED4vPHeh/50iS9atS5LQ==
    ```

    >[!NOTE]
    >В приведенном выше синтаксисе указывается файл, который будет скопирован в папку в учетной записи Data Lake Storage 1-го поколения. Инструмент AdlCopy создает папку, если указанного имени папки не существует.

    Вам будет предложено ввести учетные данные для подписки Azure, в которой расположена учетная запись Data Lake Storage 1-го поколения. Вы увидите результат, аналогичный приведенному ниже:

    ```output
    Initializing Copy.
    Copy Started.
    100% data copied.
    Finishing Copy.
    Copy Completed. 1 file copied.
    ```

1. Кроме того, можно скопировать все большие двоичные объекты из одного контейнера в учетную запись Data Lake Storage 1-го поколения, используя следующую команду:

    ```console
    AdlCopy /source https://<source_account>.blob.core.windows.net/<source_container>/ /dest swebhdfs://<dest_adlsg1_account>.azuredatalakestore.net/<dest_folder>/ /sourcekey <storage_account_key_for_storage_container>  
    ```      

    Пример:

    ```console
    AdlCopy /Source https://mystorage.blob.core.windows.net/mycluster/example/data/gutenberg/ /dest adl://mydatalakestorage.azuredatalakestore.net/mynewfolder/ /sourcekey uJUfvD6cEvhfLoBae2yyQf8t9/BpbWZ4XoYj4kAS5Jf40pZaMNf0q6a8yqTxktwVgRED4vPHeh/50iS9atS5LQ==
    ```

### <a name="performance-considerations"></a>Вопросы производительности

При копировании из учетной записи хранения BLOB-объектов Azure может выполняться регулирование во время копирования на стороне хранилища BLOB-объектов. Это может снизить производительность задания копирования. Дополнительные сведения об ограничениях хранилища BLOB-объектов Azure см. в статье [Подписка Azure, границы, квоты и ограничения службы](../azure-resource-manager/management/azure-subscription-service-limits.md).

## <a name="use-adlcopy-as-standalone-to-copy-data-from-another-data-lake-storage-gen1-account"></a>Использование AdlCopy (автономно) для копирования данных из другой учетной записи Data Lake Storage 1-го поколения

Кроме того, можно использовать AdlCopy для копирования данных между двумя учетными записями Data Lake Storage 1-го поколения.

1. Откройте командную строку и перейдите в каталог, в который установлено средство AdlCopy, обычно `%HOMEPATH%\Documents\adlcopy`.
1. Чтобы скопировать определенный файл из одной учетной записи Data Lake Storage 1-го поколения в другую, выполните следующую команду.

    ```console
    AdlCopy /Source adl://<source_adlsg1_account>.azuredatalakestore.net/<path_to_file> /dest adl://<dest_adlsg1_account>.azuredatalakestore.net/<path>/
    ```

    Пример:

    ```console
    AdlCopy /Source adl://mydatastorage.azuredatalakestore.net/mynewfolder/909f2b.log /dest adl://mynewdatalakestorage.azuredatalakestore.net/mynewfolder/
    ```

   > [!NOTE]
   > Приведенный выше синтаксис указывает файл, который будет скопирован в папку в целевой учетной записи Data Lake Storage 1-го поколения. Инструмент AdlCopy создает папку, если указанного имени папки не существует.
   >
   >

    Вам будет предложено ввести учетные данные для подписки Azure, в которой расположена учетная запись Data Lake Storage 1-го поколения. Вы увидите результат, аналогичный приведенному ниже:

    ```output
    Initializing Copy.
    Copy Started.|
    100% data copied.
    Finishing Copy.
    Copy Completed. 1 file copied.
    ```
1. Следующая команда копирует все файлы из указанной папки в исходной учетной записи Data Lake Storage 1-го поколения в папку в целевой учетной записи Data Lake Storage 1-го поколения.

    ```console
    AdlCopy /Source adl://mydatastorage.azuredatalakestore.net/mynewfolder/ /dest adl://mynewdatalakestorage.azuredatalakestore.net/mynewfolder/
    ```

### <a name="performance-considerations"></a>Вопросы производительности

При использовании AdlCopy в качестве автономного средства копия выполняется на общих ресурсах, управляемых Azure. Производительность в этой среде зависит от системной нагрузки и доступных ресурсов. Этот режим лучше всего применять для небольших передач, выполняемых при необходимости. При использовании AdlCopy в качестве автономного инструмента параметры не требуют настройки.

## <a name="use-adlcopy-with-data-lake-analytics-account-to-copy-data"></a>Использование AdlCopy (с учетной записью Data Lake Analytics) для копирования данных

Вы также можете воспользоваться своей учетной записью Data Lake Analytics для запуска задания AdlCopy для копирования данных из больших двоичных объектов службы хранилища Azure в Data Lake Storage 1-го поколения. Этот вариант обычно используется, если объем перемещаемых данных измеряется гигабайтами или терабайтами и вам необходима лучшая и прогнозируемая производительность.

Чтобы использовать учетную запись Data Lake Analytics с AdlCopy для копирования из большого двоичного объекта службы хранилища Azure, необходимо добавить источник (большой двоичный объект службы хранилища Azure) в качестве источника данных для вашей учетной записи Data Lake Analytics. Инструкции по добавлению дополнительных источников данных в учетную записи Data Lake Analytics см. в разделе [Управление источниками данных учетной записи](../data-lake-analytics/data-lake-analytics-manage-use-portal.md#manage-data-sources).

> [!NOTE]
> Если копирование выполняется с использованием учетной записи Azure Data Lake Storage 1-го поколения, а источником является учетная запись Data Lake Analytics, то учетную запись Data Lake Storage 1-го поколения не требуется связывать с учетной записью Data Lake Analytics. Связь исходного хранилища с учетной записью Data Lake Analytics требуется только в том случае, если источником является учетная запись хранения Azure.
>
>

Выполните следующую команду, чтобы скопировать данные из большого двоичного объекта службы хранилища Azure в учетную запись Data Lake Storage 1-го поколения, используя учетную запись Data Lake Analytics:

```console
AdlCopy /source https://<source_account>.blob.core.windows.net/<source_container>/<blob name> /dest swebhdfs://<dest_adlsg1_account>.azuredatalakestore.net/<dest_folder>/ /sourcekey <storage_account_key_for_storage_container> /Account <data_lake_analytics_account> /Units <number_of_data_lake_analytics_units_to_be_used>
```

Пример:

```console
AdlCopy /Source https://mystorage.blob.core.windows.net/mycluster/example/data/gutenberg/ /dest swebhdfs://mydatalakestorage.azuredatalakestore.net/mynewfolder/ /sourcekey uJUfvD6cEvhfLoBae2yyQf8t9/BpbWZ4XoYj4kAS5Jf40pZaMNf0q6a8yqTxktwVgRED4vPHeh/50iS9atS5LQ== /Account mydatalakeanalyticaccount /Units 2
```

Точно также выполните следующую команду, чтобы скопировать все файлы из указанной папки в исходной учетной записи Data Lake Storage 1-го поколения в папку в целевой учетной записи Data Lake Storage 1-го поколения с помощью учетной записи Data Lake Analytics.

```console
AdlCopy /Source adl://mysourcedatalakestorage.azuredatalakestore.net/mynewfolder/ /dest adl://mydestdatastorage.azuredatalakestore.net/mynewfolder/ /Account mydatalakeanalyticaccount /Units 2
```

### <a name="performance-considerations"></a>Вопросы производительности

При копировании данных в диапазоне терабайт применение AdlCopy с учетной записью Azure Data Lake Analytics обеспечивает более высокую и более предсказуемую производительность. Параметр, который необходимо настроить, — это количество единиц Azure Data Lake Analytics для задания копирования. Увеличение числа единиц повысит производительность задания копирования. Каждый копируемый файл может использовать не более одной единицы. Если указать количество единиц, превышающее количество копируемых файлов, то это не приведет к повышению производительности.

## <a name="use-adlcopy-to-copy-data-using-pattern-matching"></a>Использование AdlCopy для копирования данных с помощью сопоставления шаблонов

В этом разделе показано, как использовать AdlCopy для копирования данных из источника (в приведенном ниже примере — из большого двоичного объекта службы хранилища Azure) в целевую учетную запись Data Lake Storage 1-го поколения с помощью сопоставления шаблонов. Например, описанные ниже действия можно использовать для копирования всех файлов с расширением CSV из исходного большого двоичного объекта в целевое хранилище.

1. Откройте командную строку и перейдите в каталог, в который установлено средство AdlCopy, обычно `%HOMEPATH%\Documents\adlcopy`.
1. Выполните следующую команду, чтобы скопировать все файлы с расширением CSV из указанного большого двоичного объекта из исходного контейнера в папку Data Lake Storage 1-го поколения:

    ```console
    AdlCopy /source https://<source_account>.blob.core.windows.net/<source_container>/<blob name> /dest swebhdfs://<dest_adlsg1_account>.azuredatalakestore.net/<dest_folder>/ /sourcekey <storage_account_key_for_storage_container> /Pattern *.csv
    ```

    Пример:

    ```console
    AdlCopy /source https://mystorage.blob.core.windows.net/mycluster/HdiSamples/HdiSamples/FoodInspectionData/ /dest adl://mydatalakestorage.azuredatalakestore.net/mynewfolder/ /sourcekey uJUfvD6cEvhfLoBae2yyQf8t9/BpbWZ4XoYj4kAS5Jf40pZaMNf0q6a8yqTxktwVgRED4vPHeh/50iS9atS5LQ== /Pattern *.csv
    ```

## <a name="billing"></a>Выставление счетов

* При автономном использовании средства AdlCopy исходящий трафик для перемещения данных будет тарифицироваться в том случае, если учетная запись службы хранилища Azure источника и учетная запись Data Lake Storage 1-го поколения находятся в разных регионах.
* При использовании средства AdlCopy с вашей учетной записью аналитики озера будут применены стандартные [тарифные ставки аналитики озера данных](https://azure.microsoft.com/pricing/details/data-lake-analytics/) .

## <a name="considerations-for-using-adlcopy"></a>Рекомендации по использованию AdlCopy

* AdlCopy (для версии 1.0.5) поддерживает копирование данных из источников, которые совокупно содержат тысячи файлов и папок. Однако при возникновении проблем с копированием большого набора данных можно распространить файлы и папки в разные вложенные папки и использовать путь к этим вложенным папкам в качестве источника.

## <a name="performance-considerations-for-using-adlcopy"></a>Рекомендации по производительности при использовании AdlCopy

AdlCopy поддерживает копирование данных, содержащих тысячи файлов и папок. Однако при возникновении проблем при копировании большого набора данных файлы и папки можно распространить на небольшие вложенные папки. Инструмент AdlCopy был создан для выполнения задач копирования при необходимости. Если требуется копировать данные на регулярной основе, то лучше воспользоваться [фабрикой данных Azure](../data-factory/connector-azure-data-lake-store.md), которая предоставляет возможности полного управления операциями копирования.

## <a name="release-notes"></a>Заметки о выпуске

* 1.0.13 — при копировании данных в одну и ту же учетную запись Azure Data Lake Storage 1-го поколения в нескольких командах adlcopy больше не требуется повторно вводить свои учетные данные для каждого запуска. Теперь Adlcopy будет кэшировать эти данные в нескольких запусках.

## <a name="next-steps"></a>Дальнейшие действия

* [Защита данных в Data Lake Storage Gen1](data-lake-store-secure-data.md)
* [Начало работы с Azure Data Lake Analytics с помощью портала Azure](../data-lake-analytics/data-lake-analytics-get-started-portal.md)
* [Создание кластеров HDInsight, использующих Data Lake Store, с помощью портала Azure](data-lake-store-hdinsight-hadoop-use-portal.md)
