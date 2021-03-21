---
title: Создание и Просмотр разделов службы "Сетка событий Azure", а также управление ими с помощью интерфейса командной строки
description: В этой статье показано, как использовать Azure CLI для создания, просмотра и удаления системных разделов.
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: c1c847c7f25e3a656b798e186a408e560b9ee9e6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98633228"
---
# <a name="create-view-and-manage-event-grid-system-topics-using-azure-cli"></a>Создание, просмотр и управление системными разделами сетки событий с помощью Azure CLI
В этой статье показано, как создавать системные разделы и управлять ими с помощью Azure CLI. Общие сведения о системных разделах см. в разделе [системные разделы](system-topics.md).

## <a name="install-extension-for-azure-cli"></a>Установка расширения для Azure CLI
Для Azure CLI вам потребуется [расширение Сетки событий](/cli/azure/azure-cli-extensions-list).

В Cloud Shell:

- Если расширение было установлено ранее, обновите его: `az extension update -n eventgrid`
- Если расширение ранее не устанавливалось, установите его:  `az extension add -n eventgrid`

При локальной установке:

1. [Установка Azure CLI](/cli/azure/install-azure-cli). С помощью `az --version` проверьте, что установлена последняя версия.
2. Удалите предыдущие версии расширения: `az extension remove -n eventgrid`
3. Установите расширение eventgrid с помощью `az extension add -n eventgrid`

## <a name="create-a-system-topic"></a>Создание системного раздела

- Чтобы сначала создать системный раздел в источнике Azure, а затем создать подписку на события для этого раздела, см. следующие справочные разделы.
    - [AZ eventgrid System — создание раздела](/cli/azure/ext/eventgrid/eventgrid/system-topic#ext-eventgrid-az-eventgrid-system-topic-create)

        ```azurecli-interactive
        # Get the ID of the Azure source (for example: Azure Storage account)
        storageid=$(az storage account show \
                --name <AZURE STORAGE ACCOUNT NAME> \
                --resource-group <AZURE RESOURCE GROUP NAME> \
                    --query id --output tsv)
    
        # Create the system topic on the Azure source (example: Azure Storage account)
        az eventgrid system-topic create \
            -g <AZURE RESOURCE GROUP NAME> \
            --name <SPECIFY SYSTEM TOPIC NAME> \
            --location <LOCATION> \
            --topic-type microsoft.storage.storageaccounts \
            --source $storageid
        ```           

        Чтобы получить список `topic-type` значений, которые можно использовать для создания системного раздела, выполните следующую команду. Эти значения типа раздела представляют источники событий, которые поддерживают создание системных разделов. Пропустите `Microsoft.EventGrid.Topics` и оставьте `Microsoft.EventGrid.Domains` список. 

        ```azurecli-interactive
        az eventgrid topic-type  list --output json | grep -w id
        ```
    - [AZ eventgrid системное событие-тема — Создание подписки](/cli/azure/ext/eventgrid/eventgrid/system-topic/event-subscription#ext-eventgrid-az-eventgrid-system-topic-event-subscription-create)

        ```azurecli-interactive
        az eventgrid system-topic event-subscription create --name <SPECIFY EVENT SUBSCRIPTION NAME> \
            -g rg1 --system-topic-name <SYSTEM TOPIC NAME> \
            --endpoint <ENDPOINT URL>         
        ```
- Чтобы создать системный раздел (неявно) при создании подписки на события для источника Azure, используйте метод [AZ eventgrid Event-Subscription Create](/cli/azure/ext/eventgrid/eventgrid/event-subscription#ext-eventgrid-az-eventgrid-event-subscription-create) . Приведем пример:
    
    ```azurecli-interactive
    storageid=$(az storage account show --name <AZURE STORAGE ACCOUNT NAME> --resource-group <AZURE RESOURCE GROUP NAME> --query id --output tsv)
    endpoint=<ENDPOINT URL>

    az eventgrid event-subscription create \
      --source-resource-id $storageid \
      --name <EVENT SUBSCRIPTION NAME> \
      --endpoint $endpoint
    ```
    Пошаговые инструкции см. в статье [Подписка на учетную запись хранения](../storage/blobs/storage-blob-event-quickstart.md?toc=%2Fazure%2Fevent-grid%2Ftoc.json#subscribe-to-your-storage-account).

## <a name="view-all-system-topics"></a>Просмотреть все разделы системы
Чтобы просмотреть все разделы и сведения о выбранном системном разделе, используйте следующие команды:

- [AZ eventgrid System — список разделов](/cli/azure/ext/eventgrid/eventgrid/system-topic#ext-eventgrid-az-eventgrid-system-topic-list)

    ```azurecli-interactive
    az eventgrid system-topic list   
     ```
- [AZ eventgrid System — вывод раздела](/cli/azure/ext/eventgrid/eventgrid/system-topic#ext-eventgrid-az-eventgrid-system-topic-show)

    ```azurecli-interactive
    az eventgrid system-topic show -g <AZURE RESOURCE GROUP NAME> -n <SYSTEM TOPIC NAME>     
     ```

## <a name="delete-a-system-topic"></a>Удаление системного раздела
Чтобы удалить системный раздел, используйте следующую команду: 

- [AZ eventgrid System — удаление раздела](/cli/azure/ext/eventgrid/eventgrid/system-topic#ext-eventgrid-az-eventgrid-system-topic-delete)

    ```azurecli-interactive
    az eventgrid system-topic delete -g <AZURE RESOURCE GROUP NAME> --name <SYSTEM TOPIC NAME>   
     ```

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о системных разделах и типах разделов, поддерживаемых службой "Сетка событий Azure", см. в разделе [системные разделы в службе "Сетка событий Azure](system-topics.md) ". 