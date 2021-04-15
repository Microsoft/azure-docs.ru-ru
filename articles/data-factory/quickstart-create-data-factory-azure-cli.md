---
title: Краткое руководство. Создание фабрики данных Azure с помощью Azure CLI
description: В этом кратком руководстве создается фабрика данных Azure, включая связанную службу, наборы данных и конвейер. Вы можете запустить конвейер для выполнения операции копирования файлов.
author: linda33wj
ms.author: jingwang
ms.service: azure-cli
ms.topic: quickstart
ms.date: 03/24/2021
ms.custom:
- template-quickstart
- devx-track-azurecli
ms.openlocfilehash: 9af5f276e49e9eb2756dc544db353c75c99bc5a9
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105937958"
---
# <a name="quickstart-create-an-azure-data-factory-using-azure-cli"></a>Краткое руководство. Создание фабрики данных Azure с помощью Azure CLI

В этом кратком руководстве описано создание фабрики данных Azure с помощью Azure CLI. Конвейер, который вы создадите в этой фабрике данных, копирует данные из одной папки в другую в хранилище BLOB-объектов Azure. Сведения о преобразовании данных с помощью фабрики данных Azure см. в статье [Преобразование данных в фабрике данных Azure](transform-data.md).

Общие сведения о службе фабрики данных Azure см. в статье [Введение в фабрику данных Azure](introduction.md).

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/), прежде чем начинать работу.

[!INCLUDE [azure-cli-prepare-your-environment](../../includes/azure-cli-prepare-your-environment.md)]

> [!NOTE]
> Чтобы создать экземпляры фабрики данных, нужно назначить учетной записи пользователя, используемой для входа в Azure, роль участника, владельца либо администратора подписки Azure. Дополнительные сведения см. в разделе [Роли Azure](quickstart-create-data-factory-powershell.md#azure-roles).

## <a name="prepare-a-container-and-test-file"></a>Подготовка контейнера и тестового файла

В этом кратком руководстве используется учетная запись хранения Azure, которая содержит контейнер с файлом.

1. Чтобы создать группу ресурсов с именем `ADFQuickStartRG`, используйте команду [az group create](/cli/azure/group#az_group_create):

   ```azurecli
   az group create --name ADFQuickStartRG --location eastus
   ```

1. Создайте учетную запись хранения с помощью команды [az storage account create](/cli/azure/storage/container#az_storage_container_create):

   ```azurecli
   az storage account create --resource-group ADFQuickStartRG \
       --name adfquickstartstorage --location eastus
   ```

1. Создайте контейнер с именем `adftutorial` с помощью команды [az storage container create](/cli/azure/storage/container#az_storage_container_create):

   ```azurecli
   az storage container create --resource-group ADFQuickStartRG --name adftutorial \
       --account-name adfquickstartstorage --auth-mode key
   ```

1. В локальном каталоге создайте файл с именем `emp.txt` для отправки. Если вы работаете в Azure Cloud Shell, текущий рабочий каталог можно найти с помощью команды `echo $PWD` Bash. Для создания файла можно использовать стандартные команды Bash, например `cat`:

   ```console
   cat > emp.txt
   This is text.
   ```

   Чтобы сохранить новый файл, нажмите клавиши **CTRL+D**.

1. Чтобы отправить новый файл в контейнер хранилища Azure, используйте команду [az storage blob upload](/cli/azure/storage/blob#az_storage_blob_upload):

   ```azurecli
   az storage blob upload --account-name adfquickstartstorage --name input/emp.txt \
       --container-name adftutorial --file emp.txt --auth-mode key
   ```

   Эта команда отправляет файл в новую папку с именем `input`.

## <a name="create-a-data-factory"></a>Создание фабрики данных

Чтобы создать фабрику данных Azure, выполните команду [az datafactory factory create](/cli/azure/ext/datafactory/datafactory/factory#ext_datafactory_az_datafactory_factory_create):

```azurecli
az datafactory factory create --resource-group ADFQuickStartRG \
    --factory-name ADFTutorialFactory
```

> [!IMPORTANT]
> Замените `ADFTutorialFactory` глобально уникальным именем фабрики данных, например, ADFTutorialFactorySP1127.

Чтобы увидеть созданную фабрику данных, выполните команду [az datafactory factory show](/cli/azure/ext/datafactory/datafactory/factory#ext_datafactory_az_datafactory_factory_show):

```azurecli
az datafactory factory show --resource-group ADFQuickStartRG \
    --factory-name ADFTutorialFactory
```

## <a name="create-a-linked-service-and-datasets"></a>Создание связанной службы и наборов данных

Затем необходимо создать связанную службу и два набора данных.

1. Получите строку подключения для своей учетной записи хранения с помощью команды [az storage account show-connection-string](/cli/azure/ext/datafactory/datafactory/factory#ext_datafactory_az_datafactory_factory_show):

   ```azurecli
   az storage account show-connection-string --resource-group ADFQuickStartRG \
       --name adfquickstartstorage --key primary
   ```

1. В рабочем каталоге создайте JSON-файл с этим содержимым, который содержит вашу строку подключения из предыдущего шага. Присвойте файлу имя `AzureStorageLinkedService.json`:

   ```json
   {
       "type":"AzureStorage",
           "typeProperties":{
           "connectionString":{
           "type": "SecureString",
           "value":"DefaultEndpointsProtocol=https;AccountName=adfquickstartstorage;AccountKey=K9F4Xk/EhYrMBIR98rtgJ0HRSIDU4eWQILLh2iXo05Xnr145+syIKNczQfORkQ3QIOZAd/eSDsvED19dAwW/tw==;EndpointSuffix=core.windows.net"
           }
       }
   }
   ```

1. Создайте связанную службу с именем с `AzureStorageLinkedService` помощью команды [az datafactory linked-service create](/cli/azure/ext/datafactory/datafactory/linked-service#ext_datafactory_az_datafactory_linked_service_create):

   ```azurecli
   az datafactory linked-service create --resource-group ADFQuickStartRG \
       --factory-name ADFTutorialFactory --linked-service-name AzureStorageLinkedService \
       --properties @AzureStorageLinkedService.json
   ```

1. В рабочем каталоге создайте JSON-файл с этим содержимым с именем `InputDataset.json`:

   ```json
   {
       "type": 
           "AzureBlob",
           "linkedServiceName": {
               "type":"LinkedServiceReference",
               "referenceName":"AzureStorageLinkedService"
               },
           "annotations": [],
           "type": "Binary",
           "typeProperties": {
               "location": {
                   "type": "AzureBlobStorageLocation",
                   "fileName": "emp.txt",
                   "folderPath": "input",
                   "container": "adftutorial"
           }
       }
   }
   ```

1. Создайте входной набор данных с именем `InputDataset` с помощью команды [az datafactory dataset create](/cli/azure/ext/datafactory/datafactory/dataset#ext_datafactory_az_datafactory_dataset_create):

   ```azurecli
   az datafactory dataset create --resource-group ADFQuickStartRG \
       --dataset-name InputDataset --factory-name ADFQuickStartFactory \
       --properties @InputDataset.json
   ```

1. В рабочем каталоге создайте JSON-файл с этим содержимым с именем `OutputDataset.json`:

   ```json
   {
       "type": 
           "AzureBlob",
           "linkedServiceName": {
               "type":"LinkedServiceReference",
               "referenceName":"AzureStorageLinkedService"
               },
           "annotations": [],
           "type": "Binary",
           "typeProperties": {
               "location": {
                   "type": "AzureBlobStorageLocation",
                   "fileName": "emp.txt",
                   "folderPath": "output",
                   "container": "adftutorial"
           }
       }
   }
   ```

1. Создайте выходной набор данных с именем `OutputDataset` с помощью команды [az datafactory dataset create](/cli/azure/ext/datafactory/datafactory/dataset#ext_datafactory_az_datafactory_dataset_create):

   ```azurecli
   az datafactory dataset create --resource-group ADFQuickStartRG \
       --dataset-name OutputDataset --factory-name ADFQuickStartFactory \
       --properties @OutputDataset.json
   ```

## <a name="create-and-run-the-pipeline"></a>Создание и запуск конвейера

Наконец, создайте и запустите конвейер.

1. В рабочем каталоге создайте JSON-файл с этим содержимым с именем `Adfv2QuickStartPipeline.json`:

   ```json
   {
       "name": "Adfv2QuickStartPipeline",
       "properties": {
           "activities": [
               {
                   "name": "CopyFromBlobToBlob",
                   "type": "Copy",
                   "dependsOn": [],
                   "policy": {
                       "timeout": "7.00:00:00",
                       "retry": 0,
                       "retryIntervalInSeconds": 30,
                       "secureOutput": false,
                       "secureInput": false
                   },
                   "userProperties": [],
                   "typeProperties": {
                          "source": {
                           "type": "BinarySource",
                           "storeSettings": {
                               "type": "AzureBlobStorageReadSettings",
                               "recursive": true
                           }
                       },
                       "sink": {
                           "type": "BinarySink",
                           "storeSettings": {
                               "type": "AzureBlobStorageWriteSettings"
                           }
                       },
                       "enableStaging": false
                   },
                   "inputs": [
                       {
                           "referenceName": "InputDataset",
                           "type": "DatasetReference"
                       }
                   ],
                   "outputs": [
                       {
                           "referenceName": "OutputDataset",
                           "type": "DatasetReference"
                       }
                   ]
               }
           ],
           "annotations": []
       }
   }
   ```

1. Создайте конвейер с именем с `Adfv2QuickStartPipeline` помощью команды [az datafactory pipeline create](/cli/azure/ext/datafactory/datafactory/pipeline#ext_datafactory_az_datafactory_pipeline_create):

   ```azurecli
   az datafactory pipeline create --resource-group ADFQuickStartRG \
       --factory-name ADFTutorialFactory --name Adfv2QuickStartPipeline \
       --pipeline @Adfv2QuickStartPipeline.json
   ```

1. Запустите конвейер с помощью команды [az datafactory pipeline create-run](/cli/azure/ext/datafactory/datafactory/pipeline#ext_datafactory_az_datafactory_pipeline_create_run):

   ```azurecli
   az datafactory pipeline create-run --resource-group ADFQuickStartRG \
       --name Adfv2QuickStartPipeline --factory-name ADFTutorialFactory
   ```

   Эта команда возвращает ИД запуска. Скопируйте его для использования в следующей команде.

1. Убедитесь, что запуск конвейера успешно выполнен, с помощью команды [az datafactory pipeline-run show](/cli/azure/ext/datafactory/datafactory/pipeline-run#ext_datafactory_az_datafactory_pipeline_run_show):

   ```azurecli
   az datafactory pipeline-run show --resource-group ADFQuickStartRG \
       --factory-name ADFTutorialFactory --run-id 00000000-0000-0000-0000-000000000000
   ```

Чтобы убедиться, что конвейер был запущен правильно, можно использовать [портал Azure](https://portal.azure.com/). Дополнительные сведения см. в разделе [Просмотр развернутых ресурсов](quickstart-create-data-factory-powershell.md#review-deployed-resources).

## <a name="clean-up-resources"></a>Очистка ресурсов

Все ресурсы в этом кратком руководстве входят в состав одной группы ресурсов. Чтобы их удалить, используйте команду [az group delete](/cli/azure/group#az_group_delete):

```azurecli
az group delete --name ADFQuickStartRG
```

Если вы используете эту группу ресурсов для чего-то еще, удалите отдельные ресурсы. Например, чтобы удалить связанную службу, используйте команду [az datafactory linked-service delete](/cli/azure/ext/datafactory/datafactory/linked-service#ext_datafactory_az_datafactory_linked_service_delete).

В этом кратком руководстве вы создали следующие JSON-файлы:

- AzureStorageLinkedService.json
- InputDataset.json
- OutputDataset.json
- Adfv2QuickStartPipeline.json

Удалите их с помощью стандартных команд Bash.

## <a name="next-steps"></a>Дальнейшие действия

- [Конвейеры и действия в Фабрике данных Azure](concepts-pipelines-activities.md)
- [Связанные службы в Фабрике данных Azure](concepts-linked-services.md)
- [Наборы данных в фабрике данных Azure](concepts-datasets-linked-services.md)
- [Преобразование данных в фабрике данных Azure](transform-data.md)
