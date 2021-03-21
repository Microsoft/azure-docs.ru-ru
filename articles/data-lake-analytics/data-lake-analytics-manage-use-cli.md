---
title: Управление Azure Data Lake Analytics с помощью Azure CLI
description: В этой статье описано, как с помощью Azure CLI управлять источниками данных, пользователями и заданиями Data Lake Analytics.
ms.service: data-lake-analytics
ms.topic: how-to
ms.date: 01/29/2018
ms.openlocfilehash: 19b471d85a52fe38b72ad55847d022fb56b3c4f0
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92220930"
---
# <a name="manage-azure-data-lake-analytics-using-the-azure-command-line-interface-cli"></a>Управление Azure Data Lake Analytics с помощью интерфейса командной строки (CLI) Azure

[!INCLUDE [manage-selector](../../includes/data-lake-analytics-selector-manage.md)]

Узнайте, как управлять учетными записями, источниками данных, пользователями и заданиями Azure Data Lake Analytics с помощью Azure CLI. Для просмотра статей, посвященных управлению с помощью других инструментов, щелкните селектор вкладок выше.

## <a name="prerequisites"></a>Предварительные условия

Для работы с этим руководством вам потребуются следующие ресурсы:

- Подписка Azure. См. страницу [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).

- Azure CLI. См. статью [Установка и настройка интерфейса командной строки Azure](/cli/azure/install-azure-cli).

  - Для выполнения этой демонстрации загрузите и установите **предварительный выпуск** [программ CLI Azure](https://github.com/MicrosoftBigData/AzureDataLake/releases) .

- Выполните аутентификацию с помощью команды `az login` и выберите подписку, которую хотите использовать. Дополнительные сведения об аутентификации с помощью рабочей или учебной учетной записи см. в статье [Подключение к среде Azure с использованием интерфейса командной строки Azure (Azure CLI)](/cli/azure/authenticate-azure-cli).

   ```azurecli
   az login
   az account set --subscription <subscription id>
   ```

   Теперь вы можете получить доступ к командам Data Lake Analytics и Data Lake Store. Выполните следующую команду, чтобы получить список команд Data Lake Store и Data Lake Analytics:

   ```azurecli
   az dls -h
   az dla -h
   ```

## <a name="manage-accounts"></a>Управление учетными записями

Перед выполнением любого задания аналитики озера данных необходимо иметь учетную запись аналитики озера данных. В отличие от Azure HDInsight учетная запись аналитики не оплачивается, если ни одно задание не выполняется. Вы платите только за время, когда выполняется задание.  Дополнительные сведения см. в разделе [Обзор аналитики озера данных Azure](data-lake-analytics-overview.md).

### <a name="create-accounts"></a>Создание учетных записей

Чтобы создать учетную запись Data Lake, выполните следующую команду.

   ```azurecli
   az dla account create --account "<Data Lake Analytics account name>" --location "<Location Name>" --resource-group "<Resource Group Name>" --default-data-lake-store "<Data Lake Store account name>"
   ```

### <a name="update-accounts"></a>Обновление учетных записей

Следующая команда используется для обновления свойств существующей учетной записи данных озера аналитики.

   ```azurecli
   az dla account update --account "<Data Lake Analytics Account Name>" --firewall-state "Enabled" --query-store-retention 7
   ```

### <a name="list-accounts"></a>Список учетных записей

Список учетных записей аналитики озера данных в конкретной группе ресурсов

   ```azurecli
   az dla account list "<Resource group name>"
   ```

## <a name="get-details-of-an-account"></a>Получение сведений об учетной записи

   ```azurecli
   az dla account show --account "<Data Lake Analytics account name>" --resource-group "<Resource group name>"
   ```

### <a name="delete-an-account"></a>Удаление учетной записи

   ```azurecli
   az dla account delete --account "<Data Lake Analytics account name>" --resource-group "<Resource group name>"
   ```

## <a name="manage-data-sources"></a>Управление источниками данных

Data Lake Analytics сейчас поддерживает два источника данных:

- [Azure Data Lake Storage](../data-lake-store/data-lake-store-overview.md)
- [Хранилище Azure](../storage/common/storage-introduction.md)

При создании учетной записи аналитики необходимо указать учетную запись хранения озера данных Azure в качестве учетной записи хранения по умолчанию. В учетной записи хранения Data Lake, используемой по умолчанию, хранятся метаданные задания и журналы аудита задания. После создания учетной записи аналитики можно добавить дополнительные учетные записи хранения озера данных и учетные записи хранения Azure.

### <a name="find-the-default-data-lake-store-account"></a>Поиск учетной записи хранения озера данных по умолчанию

Чтобы узнать, какая учетная запись Data Lake Store используется по умолчанию, выполните команду `az dla account show`. Имя учетной записи по умолчанию указано под свойством defaultDataLakeStoreAccount.

   ```azurecli
   az dla account show --account "<Data Lake Analytics account name>"
   ```

### <a name="add-additional-blob-storage-accounts"></a>Добавление дополнительных учетных записей хранилища BLOB-объектов

   ```azurecli
   az dla account blob-storage add --access-key "<Azure Storage Account Key>" --account "<Data Lake Analytics account name>" --storage-account-name "<Storage account name>"
   ```

> [!NOTE]
> Поддерживаются только короткие имена хранилищ BLOB-объектов. Не следует использовать полное доменное имя, например "myblob.blob.core.windows.net".
>

### <a name="add-additional-data-lake-store-accounts"></a>Добавление дополнительных учетных записей хранения озера данных

Следующая команда обновляет указанную учетную запись Data Lake Analytics с помощью дополнительной учетной записи Data Lake Store:

   ```azurecli
   az dla account data-lake-store add --account "<Data Lake Analytics account name>" --data-lake-store-account-name "<Data Lake Store account name>"
   ```

### <a name="update-existing-data-source"></a>Обновление существующего источника данных

Обновление существующего ключа учетной записи хранилища BLOB-объектов

   ```azurecli
   az dla account blob-storage update --access-key "<New Blob Storage Account Key>" --account "<Data Lake Analytics account name>" --storage-account-name "<Data Lake Store account name>"
   ```

### <a name="list-data-sources"></a>Получение списка источников данных

Чтобы получить список учетных записей Data Lake Store, используйте следующую команду:

   ```azurecli
   az dla account data-lake-store list --account "<Data Lake Analytics account name>"
   ```

Чтобы получить список учетных записей хранилища BLOB-объектов, используйте следующую команду:

   ```azurecli
   az dla account blob-storage list --account "<Data Lake Analytics account name>"
   ```

![Снимок экрана, показывающий Azure C L I с выделенной информацией "Даталакестореаккаунтс:".](./media/data-lake-analytics-manage-use-cli/data-lake-analytics-list-data-source.png)

### <a name="delete-data-sources"></a>Удаление источников данных

Удаление учетной записи хранения озера данных Azure

   ```azurecli
   az dla account data-lake-store delete --account "<Data Lake Analytics account name>" --data-lake-store-account-name "<Azure Data Lake Store account name>"
   ```

Удаление учетной записи хранения BLOB-объектов

   ```azurecli
   az dla account blob-storage delete --account "<Data Lake Analytics account name>" --storage-account-name "<Data Lake Store account name>"
   ```

## <a name="manage-jobs"></a>Управление заданиями

Для создания любого задания требуется учетная запись аналитики озера данных.  Дополнительные сведения см. в разделе [Управление учетными записями Data Lake Analytics](#manage-accounts).

### <a name="list-jobs"></a>Список заданий

   ```azurecli
   az dla job list --account "<Data Lake Analytics account name>"
   ```

   ![Источник данных списка аналитики озера данных](./media/data-lake-analytics-manage-use-cli/data-lake-analytics-list-jobs.png)

### <a name="get-job-details"></a>Получение сведений о задании

   ```azurecli
   az dla job show --account "<Data Lake Analytics account name>" --job-identity "<Job Id>"
   ```

### <a name="submit-jobs"></a>Отправка заданий

> [!NOTE]
> Приоритет задания по умолчанию — 1000, а степень параллелизма по умолчанию для задания — 1.
>
>    ```azurecli
>    az dla job submit --account "<Data Lake Analytics account name>" --job-name "<Name of your job>" --script "<Script to submit>"
>    ```

### <a name="cancel-jobs"></a>Отмена задания

Чтобы найти идентификатор задания, используйте команду list, а затем нажмите кнопку Отмена, чтобы отменить задание.

   ```azurecli
   az dla job cancel --account "<Data Lake Analytics account name>" --job-identity "<Job Id>"
   ```

## <a name="pipelines-and-recurrences"></a>Конвейеры и повторения

### <a name="get-information-about-pipelines-and-recurrences"></a>Получение сведений о конвейерах и повторениях

Используйте команды `az dla job pipeline`, чтобы получить сведения о конвейерах для ранее отправленных заданий.

```azurecli
az dla job pipeline list --account "<Data Lake Analytics Account Name>"

az dla job pipeline show --account "<Data Lake Analytics Account Name>" --pipeline-identity "<Pipeline ID>"
```

Используйте команды `az dla job recurrence`, чтобы получить сведения о повторениях для ранее отправленных заданий.

```azurecli
az dla job recurrence list --account "<Data Lake Analytics Account Name>"

az dla job recurrence show --account "<Data Lake Analytics Account Name>" --recurrence-identity "<Recurrence ID>"
```

## <a name="next-steps"></a>Дальнейшие действия

- [Обзор аналитики озера данных Microsoft Azure](data-lake-analytics-overview.md)
- [Приступая к работе с Data Lake Analytics с помощью портал Azure](data-lake-analytics-get-started-portal.md)
- [Управление Azure Data Lake Analytics с помощью портал Azure](data-lake-analytics-manage-use-portal.md)
- [Мониторинг и устранение неполадок Azure Data Lake Analytics заданий с помощью портал Azure](data-lake-analytics-monitor-and-troubleshoot-jobs-tutorial.md)
