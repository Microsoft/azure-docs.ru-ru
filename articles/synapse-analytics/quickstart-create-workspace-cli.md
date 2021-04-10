---
title: Краткое руководство. Создание рабочей области Synapse с помощью Azure CLI
description: Создайте рабочую область Azure Synapse с помощью Azure CLI, выполнив действия, описанные в этом руководстве.
services: synapse-analytics
author: alehall
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: workspace
ms.date: 08/25/2020
ms.author: alehall
ms.reviewer: jrasnick
ms.openlocfilehash: 8a56b325dd5e1180b1229465965167241fab76a8
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101676468"
---
# <a name="quickstart-create-an-azure-synapse-workspace-with-azure-cli"></a>Краткое руководство. Создание рабочей области Azure Synapse с помощью Azure CLI

Azure CLI — это интерфейс командной строки Azure для управления ресурсами Azure. Вы можете использовать его в браузере с Azure Cloud Shell. Его также можно установить в macOS, Linux или Windows и запускать из командной строки.

В этом кратком руководстве вы узнаете, как создать рабочую область Synapse с помощью Azure CLI.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

- Скачайте и установите [jg](https://stedolan.github.io/jq/download/), простой и удобный процессор командной строки JSON
- [Учетная запись хранения Azure Data Lake Storage 2-го поколения](../storage/common/storage-account-create.md)

    > [!IMPORTANT]
    > В рабочей области Azure Synapse должна быть возможность считывать данные в выбранной учетной записи ADLS 2-го поколения и выполнять запись в нее. Кроме того, для любой учетной записи хранения, связываемой в качестве основной учетной записи хранения, при создании учетной записи хранения необходимо включить **иерархическое пространство имен**, как описано на странице [Создание учетной записи хранения](../storage/common/storage-account-create.md?tabs=azure-portal#create-a-storage-account). 

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

## <a name="create-an-azure-synapse-workspace-using-the-azure-cli"></a>Создание рабочей области Azure Synapse с помощью Azure CLI

1. Задайте необходимые переменные среды, чтобы создать ресурсы для рабочей области Azure Synapse.

    | Имя переменной среды | Описание |
    |---|---|---|
    |Имя_учетной_записи_хранения| Имя существующей учетной записи хранения ADLS 2-го поколения.|
    |StorageAccountResourceGroup| Имя существующей группы ресурсов учетной записи хранения ADLS 2-го поколения. |
    |FileShareName| Имя существующей файловой системы хранилища.|
    |SynapseResourceGroup| Выберите новое имя для группы ресурсов Azure Synapse. |
    |Регион| Выберите один из [регионов Azure](https://azure.microsoft.com/global-infrastructure/geographies/#overview). |
    |SynapseWorkspaceName| Выберите уникальное имя для новой рабочей области Azure Synapse. |
    |SqlUser| Выберите значение для нового имени пользователя.|
    |SqlPassword| Выберите надежный пароль.|
    |||

1. Создайте группу ресурсов в форме контейнера для рабочей области Azure Synapse:
    ```azurecli
    az group create --name $SynapseResourceGroup --location $Region
    ```

1. Создайте рабочую область Azure Synapse:
    ```azurecli
    az synapse workspace create \
      --name $SynapseWorkspaceName \
      --resource-group $SynapseResourceGroup \
      --storage-account $StorageAccountName \
      --file-system $FileShareName \
      --sql-admin-login-user $SqlUser \
      --sql-admin-login-password $SqlPassword \
      --location $Region
    ```

1. Получите URL-адрес Web и Dev для рабочей области Azure Synapse:
    ```azurecli
    WorkspaceWeb=$(az synapse workspace show --name $SynapseWorkspaceName --resource-group $SynapseResourceGroup | jq -r '.connectivityEndpoints | .web')

    WorkspaceDev=$(az synapse workspace show --name $SynapseWorkspaceName --resource-group $SynapseResourceGroup | jq -r '.connectivityEndpoints | .dev')
    ```

1. Создайте правило брандмауэра, которое разрешит доступ к рабочей области Azure Synapse с компьютера:

    ```azurecli
    ClientIP=$(curl -sb -H "Accept: application/json" "$WorkspaceDev" | jq -r '.message')
    ClientIP=${ClientIP##'Client Ip address : '}
    echo "Creating a firewall rule to enable access for IP address: $ClientIP"

    az synapse workspace firewall-rule create --end-ip-address $ClientIP --start-ip-address $ClientIP --name "Allow Client IP" --resource-group $SynapseResourceGroup --workspace-name $SynapseWorkspaceName
    ```

1. Чтобы получить доступ к рабочей области, откройте URL-адрес Web рабочей области Azure Synapse, который хранится в переменной среды`WorkspaceWeb`:

    ```azurecli
    echo "Open your Azure Synapse Workspace Web URL in the browser: $WorkspaceWeb"
    ```
    
    [ ![web для рабочей области Azure Synapse](media/quickstart-create-synapse-workspace-cli/create-workspace-cli-1.png) ](media/quickstart-create-synapse-workspace-cli/create-workspace-cli-1.png#lightbox)


## <a name="clean-up-resources"></a>Очистка ресурсов

Выполните действия ниже, чтобы удалить рабочую область Azure Synapse.
> [!WARNING]
> Удаление рабочей области Azure Synapse приведет к удалению модулей аналитики и данных, хранящихся в базе данных автономных пулов SQL и метаданных рабочей области. В этом случае вы больше не сможете подключиться к конечным точкам SQL и Apache Spark. Все артефакты кода будут удалены (запросы, записные книжки, определения заданий и конвейеры).
>
> Удаление рабочей области **не** влияет на данные в Azure Data Lake Store 2-го поколения, связанные с рабочей областью.

Если вы хотите удалить рабочую область Azure Synapse, выполните следующую команду:

```azurecli
az synapse workspace delete --name $SynapseWorkspaceName --resource-group $SynapseResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

Теперь можно создать пулы [SQL](quickstart-create-sql-pool-studio.md) или [Apache Spark](quickstart-create-apache-spark-pool-studio.md), чтобы начать анализ и изучение данных.