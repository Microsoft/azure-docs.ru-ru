---
title: Перемещение данных в хранилище больших двоичных объектов с помощью соединителей SSIS — командный процесс обработки и анализа данных
description: Узнайте, как перемещать данные в хранилище BLOB-объектов Azure или из него с помощью пакета дополнительных компонентов SQL Server Integration Services для Azure.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 9572d612e7ee8e2fd72850ba14447e8449f0f371
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93322002"
---
# <a name="move-data-to-or-from-azure-blob-storage-using-ssis-connectors"></a>Перемещение данных в хранилище BLOB-объектов Azure и из него с помощью соединителей SSIS
[Пакет дополнительных компонентов SQL Server Integration Services для Azure](/sql/integration-services/azure-feature-pack-for-integration-services-ssis) содержит компоненты для подключения к Azure, передачи данных между Azure и локальными источниками данных, а также для обработки данных, хранящихся в Azure.

[!INCLUDE [blob-storage-tool-selector](../../../includes/machine-learning-blob-storage-tool-selector.md)]

После переноса локальных данных в облако они смогут получить доступ к своим данным из любой службы Azure, чтобы использовать все возможности набора технологий Azure. Эти данные можно впоследствии использовать, например, в Машинное обучение Azure или в кластере HDInsight.

Примеры использования этих ресурсов Azure приведены в пошаговых руководствах по [SQL](sql-walkthrough.md) и [HDInsight](hive-walkthrough.md) .

Сведения о канонических сценариях, в рамках которых службы SSIS используются для выполнения бизнес-задач, часто встречающихся в сценариях интеграции гибридных данных, см. в записи блога [Doing more with SQL Server Integration Services Feature Pack for Azure](https://techcommunity.microsoft.com/t5/sql-server-integration-services/doing-more-with-sql-server-integration-services-feature-pack-for/ba-p/388238) (Повышенная эффективность при использовании пакета дополнительных компонентов SQL Server Integration Services для Azure).

> [!NOTE]
> Полное описание базовых принципов использования хранилища BLOB-объектов Azure см. в статьях [Приступая к работе с хранилищем BLOB-объектов Azure с помощью .NET](../../storage/blobs/storage-quickstart-blobs-dotnet.md) и [Основные понятия службы BLOB-объектов](/rest/api/storageservices/Blob-Service-Concepts).
> 
> 

## <a name="prerequisites"></a>Предварительные условия
Для выполнения задач, описанных в этой статье, необходимо настроить подписку Azure и учетную запись хранения Azure. Для отправки или скачивания данных необходимо имя учетной записи хранения Azure и ключ учетной записи.

* Сведения о настройке **подписки Azure** см. на странице [Создайте бесплатную учетную запись Azure уже сегодня](https://azure.microsoft.com/pricing/free-trial/).
* Инструкции по созданию **учетной записи хранения** и сведениям об учетных записях и ключах [см. в](../../storage/common/storage-account-create.md)этой статье.

Чтобы использовать **Соединители SSIS**, необходимо загрузить следующие компоненты:

* **SQL Server 2014 или 2016 Standard (или более поздняя версия)**: установка включает в себя SQL Server Integration Services.
* **Microsoft SQL Server 2014 или 2016 Integration Services пакет дополнительных компонентов для Azure**. эти соединители можно скачать соответственно, с [SQL Server 2014 Integration Services](https://www.microsoft.com/download/details.aspx?id=47366) и [SQL Server 2016 Integration Services](https://www.microsoft.com/download/details.aspx?id=49492) страниц.

> [!NOTE]
> Службы SSIS устанавливаются вместе с SQL Server, но в версию Express они не включены. Сведения о приложениях, которые включены в различные выпуски SQL Server, см. в статье [SQL Server 2016 SP1 editions](https://www.microsoft.com/en-us/server-cloud/products/sql-server-editions/) (Выпуски SQL Server 2016 с пакетом обновления 1 (SP1)).
> 
> 

Обучающие материалы о службах SSIS см. в [этой статье](https://www.microsoft.com/sql-server/training-certification).

Дополнительные сведения о том, как с помощью SISS получить рабочую среду для создания простых пакетов извлечения, преобразования и загрузки (ETL), см. в статье [Службы SSIS: создание пакета ETL](/sql/integration-services/ssis-how-to-create-an-etl-package).

## <a name="download-nyc-taxi-dataset"></a>Загрузка набора данных для такси Нью-Йорка
В приведенном здесь примере используется общедоступный набор данных [Поездки в такси по Нью-Йорку](https://www.andresmh.com/nyctaxitrips/) . Набор данных состоит из информации об около 173 миллионах поездок в такси в Нью-Йорке в 2013 году. В нем используется два типа данных: сведения о поездках и тарифах. Так как для каждого месяца существует файл, у нас есть 24 файла, каждый из которых составляет около 2 ГБ без сжатия.

## <a name="upload-data-to-azure-blob-storage"></a>Отправка данных в хранилище больших двоичных объектов Azure
Чтобы переместить данные с помощью пакета дополнительных компонентов SSIS из локальной среды в хранилище BLOB-объектов Azure, мы используем экземпляр [**задачи отправки BLOB-объектов Azure**](/sql/integration-services/control-flow/azure-blob-upload-task), как показано здесь:

![configure-data-science-vm](./media/move-data-to-azure-blob-using-ssis/ssis-azure-blob-upload-task.png)

Ниже описаны параметры, используемые задачей.

| Поле | Описание |
| --- | --- |
| **AzureStorageConnection** |Указывает существующий диспетчер подключений службы хранилища Azure или создает новый, который ссылается на учетную запись хранения Azure, указывающую на место размещения файлов большого двоичного объекта. |
| **BlobContainer** |Указывает имя контейнера больших двоичных объектов, который содержит отправленные файлы в виде больших двоичных объектов. |
| **BlobDirectory** |Указывает каталог больших двоичных объектов, где загруженный файл хранится в виде блочного BLOB-объекта. Каталог больших двоичных объектов — это виртуальная иерархическая структура. Если большой двоичный объект уже существует, он заменяется. |
| **LocalDirectory** |Задает локальный каталог с файлами для отправки. |
| **FileName** |Указывает имя фильтра для выбора файлов с указанным шаблоном имени. Например, MySheet\*.xls\* включает в себя файлы MySheet001.xls и MySheetABC.xlsx. |
| **TimeRangeFrom/TimeRangeTo** |Задает фильтр диапазона времени. Включаются файлы, измененные после *TimeRangeFrom* и до *TimeRangeTo*. |

> [!NOTE]
> Учетные данные **AzureStorageConnection** должны быть правильными. **BlobContainer** должен быть создан до попытки передачи.
> 
> 

## <a name="download-data-from-azure-blob-storage"></a>Загрузка данных из хранилища BLOB-объектов Azure
Чтобы скачать данные из хранилища BLOB-объектов Azure в локальное хранилище с помощью служб Integration Services, используйте экземпляр [задачи скачивания BLOB-объектов Azure](/sql/integration-services/control-flow/azure-blob-download-task).

## <a name="more-advanced-ssis-azure-scenarios"></a>Более сложные сценарии SSIS-Azure
С помощью пакета дополнительных компонентов SSIS можно обрабатывать более сложные потоки, используя упаковку задач. Например, данные BLOB-объекта можно передать непосредственно в кластер HDInsight. После этого полученный результат можно скачать обратно в BLOB-объект, а затем — в локальное хранилище. Службы SSIS могут запускать задания Hive и Pig в кластере HDInsight с помощью дополнительных соединительных служб SSIS.

* Чтобы запустить сценарий Hive в кластере Azure HDInsight с помощью служб SSIS, используйте [задачу Hive Azure HDInsight](/sql/integration-services/control-flow/azure-hdinsight-hive-task).
* Чтобы запустить сценарий Pig в кластере Azure HDInsight с помощью служб SSIS, используйте [задачу Pig Azure HDInsight](/sql/integration-services/control-flow/azure-hdinsight-pig-task).