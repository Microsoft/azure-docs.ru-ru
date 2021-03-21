---
title: Создание среды Gen2 "аналитика временных рядов Azure" с помощью Azure CLI "Gen2" аналитика временных рядов Azure "| Документация Майкрософт
description: Узнайте, как настроить среду в Gen2 "аналитика временных рядов Azure" с помощью Azure CLI.
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: how-to
ms.date: 03/15/2021
ms.custom: seodec18
ms.openlocfilehash: ed185413cff155610b2b088b1791169e33f6ce7a
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103464484"
---
# <a name="create-an-azure-time-series-insights-gen2-environment-using-the-azure-cli"></a>Создание среды Gen2 "аналитика временных рядов Azure" с помощью Azure CLI

В этом документе вы узнаете, как создать среду Gen2 "аналитика временных рядов".

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="prerequisites"></a>Предварительные требования

* Создание учетной записи хранения Azure для [холодного хранилища ](./concepts-storage.md#cold-store) в вашей среде. Эта учетная запись предназначена для долгосрочного хранения и анализа исторических данных.

> [!NOTE]
> Замените в коде значение `mytsicoldstore` уникальным именем учетной записи для холодного хранилища.

Сначала создайте учетную запись хранения:

```azurecli-interactive
storage=mytsicoldstore
rg=-my-resource-group-name
az storage account create -g $rg -n $storage --https-only
key=$(az storage account keys list -g $rg -n $storage --query [0].value --output tsv
```

## <a name="creating-the-environment"></a>Создание среды

Теперь, когда учетная запись хранения создана, ее имя и ключ управления назначены переменным, выполните следующую команду, чтобы создать среду "аналитика временных рядов Azure":

> [!NOTE]
> В коде замените следующие уникальные имена для сценария:
>
> * `my-tsi-env` с именем среды.
> * `my-ts-id-prop` с именем свойства идентификатора временного ряда.

> [!IMPORTANT]
> Идентификатор временных рядов в вашей среде подобен ключу секции базы данных. Идентификатор временных рядов также выступает в качестве первичного ключа для модели временных рядов.
>
> Дополнительные сведения см [. в разделе рекомендации по выбору идентификатора временного ряда.](./how-to-select-tsid.md)

```azurecli-interactive
az tsi environment gen2 create --name "my-tsi-env" --location eastus2 --resource-group $rg --sku name="L1" capacity=1 --time-series-id-properties name=my-ts-id-prop type=String --warm-store-configuration data-retention=P7D --storage-configuration account-name=$storage management-key=$key
```

## <a name="remove-an-azure-time-series-insights-environment"></a>Удаление среды службы "аналитика временных рядов Azure"

Azure CLI можно использовать для удаления отдельного ресурса, например среды "аналитика временных рядов", или удаления группы ресурсов и всех ее ресурсов, включая все среды аналитики временных рядов.

Чтобы [Удалить среды "аналитика временных рядов](/cli/azure/ext/timeseriesinsights/tsi/environment?view=azure-cli-latest#ext_timeseriesinsights_az_tsi_environment_delete)", выполните следующую команду:

```azurecli-interactive
az tsi environment delete --name "my-tsi-env" --resource-group $rg
```

Чтобы [удалить учетную запись хранения](/cli/azure/storage/account?view=azure-cli-latest#az_storage_account_delete), выполните следующую команду:

```azurecli-interactive
az storage account delete --name $storage --resource-group $rg
```

Чтобы [Удалить группу ресурсов](/cli/azure/group#az-group-delete) и все ее ресурсы, выполните следующую команду:

```azurecli-interactive
az group delete --name $rg
```

## <a name="next-steps"></a>Следующие шаги

* Сведения о [источниках событий приема потоковой передачи](./concepts-streaming-ingestion-event-sources.md) для среды Gen2 "аналитика временных рядов Azure".
* Узнайте, как подключиться к центру [Интернета вещей](./how-to-ingest-data-iot-hub.md)
