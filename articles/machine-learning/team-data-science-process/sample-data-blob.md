---
title: Пример данных в хранилище BLOB-объектов Azure. процесс обработки и анализа данных группы
description: Данные выборки, хранящиеся в хранилище BLOB-объектов Azure, загружаются программно, а затем изменяются с помощью процедур, написанных на Python
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
ms.openlocfilehash: c5827a0e07e537b66684f852d8f3e1500cd9febb
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98788847"
---
# <a name="sample-data-in-azure-blob-storage"></a><a name="heading"></a>Пример данных в хранилище BLOB-объектов Azure

В этой статье рассматривается выборка данных, хранящихся в хранилище BLOB-объектов Azure, путем их загрузки программным способом и последующей их выборки с помощью процедур, написанных на

**Для чего нужна выборка данных?**
Если размер набора данных, который планируется проанализировать, слишком большой, обычно рекомендуется уменьшить выборку данных до размера, который останется репрезентативным и будет более управляемым. Выборка упрощает понимание данных, исследование и проектирование признаков. Роль этой операции в процессе аналитики Кортаны заключается в том, чтобы сделать возможным быстрое прототипирование функций обработки данных и моделей машинного обучения.

Эта задача выборки является одним из этапов [процесса обработки и анализа данных группы (TDSP)](./index.yml).

## <a name="download-and-down-sample-data"></a>Скачать данные и уменьшить выборку данных
1. Скачайте данные из хранилища BLOB-объектов Azure с помощью службы BLOB-объектов из следующего примера кода Python: 

    ```python
    from azure.storage.blob import BlobService
    import tables

    STORAGEACCOUNTNAME= <storage_account_name>
    STORAGEACCOUNTKEY= <storage_account_key>
    LOCALFILENAME= <local_file_name>        
    CONTAINERNAME= <container_name>
    BLOBNAME= <blob_name>

    #download from blob
    t1=time.time()
    blob_service=BlobService(account_name=STORAGEACCOUNTNAME,account_key=STORAGEACCOUNTKEY)
    blob_service.get_blob_to_path(CONTAINERNAME,BLOBNAME,LOCALFILENAME)
    t2=time.time()
    print(("It takes %s seconds to download "+blobname) % (t2 - t1))
    ```

2. Считать данные во фрейм данных Pandas из файла, загруженного выше.

    ```python
    import pandas as pd

    #directly ready from file on disk
    dataframe_blobdata = pd.read_csv(LOCALFILE)
    ```

3. Уменьшить выборку данных с помощью `random.choice``numpy`:

    ```python
    # A 1 percent sample
    sample_ratio = 0.01 
    sample_size = np.round(dataframe_blobdata.shape[0] * sample_ratio)
    sample_rows = np.random.choice(dataframe_blobdata.index.values, sample_size)
    dataframe_blobdata_sample = dataframe_blobdata.ix[sample_rows]
    ```

Теперь вы можете работать с приведенным выше кадром данных с примером "один процент" для дальнейшего исследования и создания компонентов.

## <a name="upload-data-and-read-it-into-azure-machine-learning"></a><a name="heading"></a>Передача данных и их считывание в службу машинного обучения Azure
С помощью следующего образца кода можно уменьшить выборку данных и использовать их непосредственно в Машинном обучении Azure.

1. Запись фрейма данных в локальный файл

    ```python
    dataframe.to_csv(os.path.join(os.getcwd(),LOCALFILENAME), sep='\t', encoding='utf-8', index=False)
    ```

2. Передача локального файла в BLOB-объект Azure с помощью следующего образца кода:

    ```python
    from azure.storage.blob import BlobService
    import tables

    STORAGEACCOUNTNAME= <storage_account_name>
    LOCALFILENAME= <local_file_name>
    STORAGEACCOUNTKEY= <storage_account_key>
    CONTAINERNAME= <container_name>
    BLOBNAME= <blob_name>

    output_blob_service=BlobService(account_name=STORAGEACCOUNTNAME,account_key=STORAGEACCOUNTKEY)    
    localfileprocessed = os.path.join(os.getcwd(),LOCALFILENAME) #assuming file is in current working directory

    try:

    #perform upload
    output_blob_service.put_block_blob_from_path(CONTAINERNAME,BLOBNAME,localfileprocessed)

    except:            
        print ("Something went wrong with uploading to the blob:"+ BLOBNAME)
    ```

3. Считывание данных из большого двоичного объекта Azure с помощью модуля [Импорт данных](/azure/machine-learning/studio-module-reference/import-data) Машинного обучения Azure, как показано на рисунке ниже.

![большой двоичный объект считывателя](./media/sample-data-blob/reader_blob.png)
