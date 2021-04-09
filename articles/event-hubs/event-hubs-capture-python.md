---
title: Чтение собранных данных в службе "Центры событий Azure" из приложения Python (последняя версия)
description: В этой статье показано, как написать код Python для сбора данных, отправляемых в концентратор событий и считывания собранных данных о событиях из учетной записи хранения Azure.
ms.topic: quickstart
ms.date: 01/04/2021
ms.openlocfilehash: acc2ce04add5fd837e9edc789e9616a9f04fb4b9
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97883203"
---
# <a name="capture-event-hubs-data-in-azure-storage-and-read-it-by-using-python-azure-eventhub"></a>Запись данных Центров событий в службу хранилища Azure и их чтение с помощью Python (azure-eventhub)

Концентратор событий можно настроить таким образом, чтобы данные, отправляемые в него, собирались в учетной записи хранения Azure либо в Azure Data Lake Storage 1-го поколения или 2-го поколения. В этой статье показано, как написать код Python, чтобы отправлять события в концентратор событий и считывать собранные данные из **хранилища BLOB-объектов Azure**. Дополнительные сведения об этой функции см. [Capture events through Azure Event Hubs in Azure Blob Storage or Azure Data Lake Storage](event-hubs-capture-overview.md) (Сбор событий из Центров событий Azure в хранилище BLOB-объектов Azure или Azure Data Lake Storage).

В приведенном кратком руководстве для демонстрации функции сбор используется [пакет SDK для Azure Python](https://azure.microsoft.com/develop/python/). Приложение *sender.py* отправляет имитацию телеметрии среды в концентраторы событий в формате JSON. В настройках концентратора событий предусмотрено использование функции записи для пакетной записи этих данных в хранилище BLOB-объектов. Приложение *capturereader.py* считывает эти большие двоичные объекты и создает для каждого устройства дополнительный файл. Затем приложение записывает данные в CSV-файлы.

В этом кратком руководстве вы: 

> [!div class="checklist"]
> * Создайте учетную запись хранилища BLOB-объектов Azure и контейнер на портале Azure.
> * Создание пространства имен Центров событий с помощью портала Azure.
> * Создайте концентратор событий с включенной функцией "Сбор" и подключите его к учетной записи хранения.
> * Отправьте данные в концентратор событий с помощью скрипта Python.
> * Прочитайте и обработайте файлы, полученные от функции "Сбор" службы "Центры событий", с помощью другого скрипта Python.

## <a name="prerequisites"></a>Предварительные требования

- Должны быть установлены Python, PIP и следующие пакеты. Для тестирования кода из этой статьи использовались такие версии: 
    - Python 3.7
    - azure-eventhub 5.2.0;
    - azure-storage-blob 12.6.0;
    - avro-python3 1.10.1.
- Подписка Azure. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/), прежде чем начать работу.  
- Активное пространство имен Центров событий и концентратор событий.
Выполните действия в статье [Краткое руководство. Создание концентратора событий с помощью портала Azure](event-hubs-create.md). Запишите имя пространства имен Центров событий, имя концентратора событий и первичный ключ доступа для пространства имен. Чтобы получить ключ доступа см. раздел [Get connection string from the portal](event-hubs-get-connection-string.md#get-connection-string-from-the-portal) (Получение строки подключения на портале). По умолчанию для ключа используется имя *RootManageSharedAccessKey*. Для этого краткого руководства нужен только первичный ключ. Вам не нужна строка подключения.  
- Учетная запись хранения Azure, контейнер больших двоичных объектов в учетной записи хранения и строка подключения к учетной записи хранения. Выполните действия в приведенных ниже ресурсах, если эти элементы отсутствуют.  
    1. [Create an Azure Storage account](../storage/common/storage-account-create.md?tabs=azure-portal) (Создание учетной записи хранения Azure)  
    1. [Создание контейнера](../storage/blobs/storage-quickstart-blobs-portal.md#create-a-container)  
    1. [Получение строки подключения к учетной записи хранения](../storage/common/storage-configure-connection-string.md).

    Обязательно запишите строку подключения и имя контейнера для последующего использования в кратком руководстве.  
- Включите функцию "Сбор" для концентратора событий. Для этого следуйте инструкциям в разделе [Enable capturing of events streaming through Azure Event Hubs](event-hubs-capture-enable-through-portal.md) (Включение функции "Сбор" в Центрах событий с помощью портала Azure). Выберите учетную запись хранения и контейнер BLOB-объектов, созданные на предыдущем шаге. Эту функцию можно включить при создании концентратора событий.  

## <a name="create-a-python-script-to-send-events-to-your-event-hub"></a>Создание сценария Python для отправки событий в концентратор событий
В этом разделе вы создадите скрипт Python, который отправляет 200 событий (10 устройств * 20 событий) в концентратор событий. Эти события являются примером чтения среды, отправляемого в формате JSON. 

1. Откройте предпочитаемый редактор Python, например [Visual Studio Code][Visual Studio Code].
2. Создайте сценарий с именем *sender.py*. 
3. Скопируйте приведенный ниже код в файл *sender.py*. 
   
    ```python
    import time
    import os
    import uuid
    import datetime
    import random
    import json
    
    from azure.eventhub import EventHubProducerClient, EventData
    
    # This script simulates the production of events for 10 devices.
    devices = []
    for x in range(0, 10):
        devices.append(str(uuid.uuid4()))
    
    # Create a producer client to produce and publish events to the event hub.
    producer = EventHubProducerClient.from_connection_string(conn_str="EVENT HUBS NAMESAPCE CONNECTION STRING", eventhub_name="EVENT HUB NAME")
    
    for y in range(0,20):    # For each device, produce 20 events. 
        event_data_batch = producer.create_batch() # Create a batch. You will add events to the batch later. 
        for dev in devices:
            # Create a dummy reading.
            reading = {'id': dev, 'timestamp': str(datetime.datetime.utcnow()), 'uv': random.random(), 'temperature': random.randint(70, 100), 'humidity': random.randint(70, 100)}
            s = json.dumps(reading) # Convert the reading into a JSON string.
            event_data_batch.add(EventData(s)) # Add event data to the batch.
        producer.send_batch(event_data_batch) # Send the batch of events to the event hub.
    
    # Close the producer.    
    producer.close()
    ```
4. Замените в сценарии следующие значения.  
    * Замените `EVENT HUBS NAMESPACE CONNECTION STRING` строками подключения для вашего пространства имен Центров событий.  
    * Замените переменную `EVENT HUB NAME` именем вашего концентратора событий.  
5. Запустите скрипт для отправки событий в концентратор событий.  
6. Вы можете проверить получение сообщения концентратором событий на портале Azure. Переключитесь на представление **Сообщения** в разделе **Метрики**. Обновите страницу, чтобы обновить диаграмму. На отображение полученных сообщений на странице может уйти несколько секунд. 

    [![Проверка получения сообщения концентратором событий](./media/get-started-capture-python-v2/messages-portal.png)](./media/get-started-capture-python-v2/messages-portal.png#lightbox)

## <a name="create-a-python-script-to-read-your-capture-files"></a>Создание скрипта Python для чтения файлов записи
В этом примере записанные данные хранятся в хранилище BLOB-объектов Azure. В этом разделе скрипт считывает записанные файлы данных из учетной записи хранения Azure и создает CSV-файлы, которые можно легко открыть и просмотреть. В текущем рабочем каталоге приложения вы увидите 10 файлов. Эти файлы будут содержать данные о среде для 10 устройств. 

1. В редакторе Python создайте скрипт с именем *capturereader.py*. Он считывает собранные файлы и создает для каждого устройства отдельный файл для записи данных только по этому устройству.
2. Скопируйте приведенный ниже код и вставьте его в файл *capturereader.py*. 
   
    ```python
    import os
    import string
    import json
    import uuid
    import avro.schema
    
    from azure.storage.blob import ContainerClient, BlobClient
    from avro.datafile import DataFileReader, DataFileWriter
    from avro.io import DatumReader, DatumWriter
    
    
    def processBlob2(filename):
        reader = DataFileReader(open(filename, 'rb'), DatumReader())
        dict = {}
        for reading in reader:
            parsed_json = json.loads(reading["Body"])
            if not 'id' in parsed_json:
                return
            if not parsed_json['id'] in dict:
                list = []
                dict[parsed_json['id']] = list
            else:
                list = dict[parsed_json['id']]
                list.append(parsed_json)
        reader.close()
        for device in dict.keys():
            filename = os.getcwd() + '\\' + str(device) + '.csv'
            deviceFile = open(filename, "a")
            for r in dict[device]:
                deviceFile.write(", ".join([str(r[x]) for x in r.keys()])+'\n')
    
    def startProcessing():
        print('Processor started using path: ' + os.getcwd())
        # Create a blob container client.
        container = ContainerClient.from_connection_string("AZURE STORAGE CONNECTION STRING", container_name="BLOB CONTAINER NAME")
        blob_list = container.list_blobs() # List all the blobs in the container.
        for blob in blob_list:
            # Content_length == 508 is an empty file, so process only content_length > 508 (skip empty files).        
            if blob.size > 508:
                print('Downloaded a non empty blob: ' + blob.name)
                # Create a blob client for the blob.
                blob_client = ContainerClient.get_blob_client(container, blob=blob.name)
                # Construct a file name based on the blob name.
                cleanName = str.replace(blob.name, '/', '_')
                cleanName = os.getcwd() + '\\' + cleanName 
                with open(cleanName, "wb+") as my_file: # Open the file to write. Create it if it doesn't exist. 
                    my_file.write(blob_client.download_blob().readall()) # Write blob contents into the file.
                processBlob2(cleanName) # Convert the file into a CSV file.
                os.remove(cleanName) # Remove the original downloaded file.
                # Delete the blob from the container after it's read.
                container.delete_blob(blob.name)
    
    startProcessing()    
    ```
3. Замените `AZURE STORAGE CONNECTION STRING` строками подключения для вашей учетной записи хранения Azure. Имя контейнера, созданного в этом кратком руководстве — *сбор*. Если вы использовали другое имя для контейнера, замените *сбор* именем контейнера в учетной записи хранения. 

## <a name="run-the-scripts"></a>Выполнение сценариев
1. Откройте окно командной строки с Python в соответствующем расположении, а затем выполните следующие команды для установки пакетов необходимых компонентов Python.
   
   ```
   pip install azure-storage-blob
   pip install azure-eventhub
   pip install avro-python3
   ```

    > [!NOTE]
    > Для тестирования кода из этой статьи использовались такие версии: 
    > - Python 3.7
    > - azure-eventhub 5.2.0;
    > - azure-storage-blob 12.6.0;
    > - avro-python3 1.10.1.
2. Замените вашу папку на папку, в которой были сохранены скрипты *sender.py* и *capturereader.py*, и выполните следующую команду.
   
   ```
   python sender.py
   ```
   
   Эта команда запускает новый процесс Python для выполнения скрипта sender.
3. Подождите несколько минут, чтобы запустить сбор, а затем введите следующую команду в исходном окне командной строки:
   
   ```
   python capturereader.py
   ```

   Этот обработчик записи загружает все большие двоичные объекты из учетной записи хранения или контейнера в локальный каталог. Он обрабатывает все непустые файлы и записывает результаты в CSV-файлы в локальном каталоге.

## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь с [примерами для Python на сайте GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/eventhub/azure-eventhub/samples) 


[Azure portal]: https://portal.azure.com/
[Overview of Event Hubs Capture]: event-hubs-capture-overview.md
[1]: ./media/event-hubs-archive-python/event-hubs-python1.png
[About Azure storage accounts]:../storage/common/storage-create-storage-account.md
[Visual Studio Code]: https://code.visualstudio.com/
[Event Hubs overview]: ./event-hubs-about.md
