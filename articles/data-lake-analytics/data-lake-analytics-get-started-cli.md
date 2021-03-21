---
title: Создание & запроса Azure Data Lake Analytics Azure CLI
description: Узнайте, как использовать интерфейс командной строки Azure для создания учетной записи Data Lake Analytics и отправки задания U-SQL.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: conceptual
ms.date: 06/18/2017
ms.custom: devx-track-azurecli
ms.openlocfilehash: 9123ca6a1bfa90737bb1ce83ee365d1ecf514e1f
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92220998"
---
# <a name="get-started-with-azure-data-lake-analytics-using-azure-cli"></a>Начало работы с Azure Data Lake Analytics с помощью интерфейса командной строки Azure

[!INCLUDE [get-started-selector](../../includes/data-lake-analytics-selector-get-started.md)]

В этой статье описано, как использовать Azure CLI для создания учетной записи Data Lake Analytics, а также отправки заданий и каталогов U-SQL. Задание, которое считывает файл с разделителями-табуляциями (TSV) и преобразует его в файл с разделителями-запятыми (CSV).

## <a name="prerequisites"></a>Предварительные условия

Для работы вам понадобится следующее:

* **Подписка Azure**. См. страницу [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).
* Для этой статьи требуется Azure CLI версии 2.0 или более поздней версии. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0]( /cli/azure/install-azure-cli).

## <a name="sign-in-to-azure"></a>Вход в Azure

Чтобы войти в подписку Azure, выполните следующие действия.

```azurecli
az login
```

Вам будет предложено перейти по URL-адресу и ввести код для аутентификации.  Следуйте инструкциям, чтобы ввести учетные данные.

Войдя в Azure, вы увидите список своих подписок.

Чтобы выбрать нужную подписку, выполните следующую команду:

```azurecli
az account set --subscription <subscription id>
```

## <a name="create-data-lake-analytics-account"></a>Создание учетной записи аналитики озера данных

Для выполнения любых заданий требуется учетная запись Data Lake Analytics. Для создания учетной записи Data Lake Analytics необходимо указать следующие данные.

* **Группа ресурсов Azure**. В группе ресурсов Azure необходимо создать учетную запись Data Lake Analytics. [Azure Resource Manager](../azure-resource-manager/management/overview.md) позволяет работать с ресурсами в приложении в виде группы. Вы можете развертывать, обновлять или удалять все ресурсы для приложения в рамках одной скоординированной операции.  

Чтобы отобразить существующие группы ресурсов в своей подписке выполните следующую команду:

```azurecli
az group list
```

Чтобы создать новую группу ресурсов:

```azurecli
az group create --name "<Resource Group Name>" --location "<Azure Location>"
```

* **Имя учетной записи Data Lake Analytics**. Каждой учетной записи Data Lake Analytics присвоено имя.
* **Расположение**. Используйте один из центров обработки данных Azure, который поддерживает Data Lake Analytics.
* **Учетная запись Data Lake Store по умолчанию** — каждая учетная запись Data Lake Analytics содержит учетную запись Data Lake Store по умолчанию.

Чтобы получить список существующих учетных записей Data Lake Store, выполните эту команду:

```azurecli
az dls account list
```

Чтобы создать новую учетную запись Data Lake Store, выполните эту команду:

```azurecli
az dls account create --account "<Data Lake Store Account Name>" --resource-group "<Resource Group Name>"
```

Используйте следующий синтаксис, чтобы создать учетную запись Data Lake Analytics:

```azurecli
az dla account create --account "<Data Lake Analytics Account Name>" --resource-group "<Resource Group Name>" --location "<Azure location>" --default-data-lake-store "<Default Data Lake Store Account Name>"
```

Создав учетную запись, вы можете получить список учетных записей и просмотреть сведения о них. Для этого выполните следующие команды:

```azurecli
az dla account list
az dla account show --account "<Data Lake Analytics Account Name>"
```

## <a name="upload-data-to-data-lake-store"></a>Передача данных в хранилище озера данных

В этом руководстве обрабатываются некоторые журналы поиска.  Журнал поиска может храниться в хранилище озера данных или в хранилище больших двоичных объектов Azure.

На портале Azure реализован пользовательский интерфейс для копирования файлов с образцами данных, включая файл журнала поиска, в учетную запись Data Lake Store по умолчанию. О передаче данных в учетную запись Data Lake Store по умолчанию см. в разделе [Подготовка исходных данных](data-lake-analytics-get-started-portal.md).

Чтобы передать файлы, используя интерфейс командной строки Azure, выполните следующую команду:

```azurecli
az dls fs upload --account "<Data Lake Store Account Name>" --source-path "<Source File Path>" --destination-path "<Destination File Path>"
az dls fs list --account "<Data Lake Store Account Name>" --path "<Path>"
```

Из аналитики озера данных также доступно хранилище больших двоичных объектов Azure.  Чтобы передать данные в хранилище BLOB-объектов Azure, см. статью [Использование интерфейса командной строки (CLI) Azure со службой хранилища Azure](../storage/blobs/storage-quickstart-blobs-cli.md).

## <a name="submit-data-lake-analytics-jobs"></a>Отправка заданий аналитики озера данных

Задания аналитики озера данных пишутся на языке U-SQL. Дополнительные сведения о языке U-SQL см. в статье о [начале работы с языком U-SQL](data-lake-analytics-u-sql-get-started.md) и в [справочнике по языку U-SQL](/u-sql/).

### <a name="to-create-a-data-lake-analytics-job-script"></a>Создание скрипта задания аналитики озера данных

Создайте текстовый файл со следующим скриптом U-SQL, а затем сохраните текстовый файл на своей рабочей станции.

```usql
@a  =
    SELECT * FROM
        (VALUES
            ("Contoso", 1500.0),
            ("Woodgrove", 2700.0)
        ) AS
              D( customer, amount );
OUTPUT @a
    TO "/data.csv"
    USING Outputters.Csv();
```

Этот сценарий U-SQL считывает файл исходных данных с помощью **Extractors.Tsv()**, а затем создает CSV-файл с помощью **Outputters.Csv()**.

Не меняйте эти два пути, если только исходный файл не был скопирован в другое место.  Data Lake Analytics создаст выходную папку, если ее не существует.

Проще использовать относительные пути для файлов, которые хранятся в учетных записях Data Lake Store по умолчанию. Также можно использовать абсолютные пути.  Пример:

```usql
adl://<Data LakeStorageAccountName>.azuredatalakestore.net:443/Samples/Data/SearchLog.tsv
```

Необходимо использовать абсолютные пути для доступа к файлам в связанных учетных записях хранения.  Для файлов, хранящихся в связанной учетной записи хранения Azure, используется следующий синтаксис:

```usql
wasb://<BlobContainerName>@<StorageAccountName>.blob.core.windows.net/Samples/Data/SearchLog.tsv
```

> [!NOTE]
> Контейнер больших двоичных объектов Azure с общедоступными большими двоичными объектами не поддерживается.
> Контейнер больших двоичных объектов Azure с общедоступными контейнерами не поддерживается.
>

### <a name="to-submit-jobs"></a>Отправка заданий

Чтобы отправить задание, используйте этот синтаксис:

```azurecli
az dla job submit --account "<Data Lake Analytics Account Name>" --job-name "<Job Name>" --script "<Script Path and Name>"
```

Пример:

```azurecli
az dla job submit --account "myadlaaccount" --job-name "myadlajob" --script @"C:\DLA\myscript.txt"
```

### <a name="to-list-jobs-and-show-job-details"></a>Отображение списка заданий и сведений о задании

```azurecli
az dla job list --account "<Data Lake Analytics Account Name>"
az dla job show --account "<Data Lake Analytics Account Name>" --job-identity "<Job Id>"
```

### <a name="to-cancel-jobs"></a>Отмена заданий

```azurecli
az dla job cancel --account "<Data Lake Analytics Account Name>" --job-identity "<Job Id>"
```

## <a name="retrieve-job-results"></a>Получение результатов задания

По окончании задания используйте следующие команды, чтобы вывести и скачать выходные файлы:

```azurecli
az dls fs list --account "<Data Lake Store Account Name>" --source-path "/Output" --destination-path "<Destination>"
az dls fs preview --account "<Data Lake Store Account Name>" --path "/Output/SearchLog-from-Data-Lake.csv"
az dls fs preview --account "<Data Lake Store Account Name>" --path "/Output/SearchLog-from-Data-Lake.csv" --length 128 --offset 0
az dls fs download --account "<Data Lake Store Account Name>" --source-path "/Output/SearchLog-from-Data-Lake.csv" --destination-path "<Destination Path and File Name>"
```

Пример:

```azurecli
az dls fs download --account "myadlsaccount" --source-path "/Output/SearchLog-from-Data-Lake.csv" --destination-path "C:\DLA\myfile.csv"
```

## <a name="next-steps"></a>Дальнейшие действия

* См. справочную документацию по интерфейсу командной строки Azure для Data Lake Analytics в разделе [Data Lake Analytics](/cli/azure/dla).
* См. справочную документацию по интерфейсу командной строки Azure для Data Lake Store в разделе [Data Lake Store](/cli/azure/dls).
* Более сложный запрос можно посмотреть в статье [Анализ журналов веб-сайта с помощью аналитики озера данных Azure](data-lake-analytics-analyze-weblogs.md).