---
title: Изменить путь к BLOB-объекту из значения по умолчанию
description: Узнайте, как настроить функцию Azure для переименования пути к файлу большого двоичного объекта по умолчанию и перемещения больших двоичных объектов в другое расположение.
author: alkohli
ms.service: storsimple
ms.topic: how-to
ms.date: 01/16/2018
ms.author: alkohli
ms.openlocfilehash: 6af095c7abdb9aa61e57d543ff2ab2f9192dadc8
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96011444"
---
# <a name="change-a-blob-path-from-the-default-path"></a>Изменение пути к большому двоичному объекту по умолчанию

При преобразовании данных в службе диспетчера данных StorSimple по умолчанию преобразованные большие двоичные объекты помещаются в контейнер хранилища, указанный во время создания целевого репозитория. Пока большие двоичные объекты поступают в текущее расположение, может возникнуть необходимость переместить их в другое расположение. В этой статье описывается настройка функции Azure для переименования пути к файлу большого двоичного объекта по умолчанию и вытекающее отсюда перемещение больших двоичных объектов в другое расположение.

## <a name="prerequisites"></a>Предварительные условия

Убедитесь, что определение задания в службе диспетчера данных StorSimple настроено правильно.

## <a name="create-an-azure-function"></a>Создание функции Azure

Для создания функции Azure выполните следующие действия:

1. Перейдите на [портал Microsoft Azure](https://portal.azure.com/).

2. Щелкните **Создать ресурс**. В поле **Поиск** введите **приложение-функция** и нажмите клавишу **ВВОД**. В появившемся списке приложений выберите и щелкните **Приложение-функция**.

    ![Введение "Приложение-функция" поле поиска](./media/storsimple-data-manager-change-default-blob-path/search-function-app.png)

3. Нажмите кнопку **Создать**.

    ![Кнопка "Создать" в окне "Приложение-функция"](./media/storsimple-data-manager-change-default-blob-path/create-function-app.png)

4. В колонке настройки **Приложение-функция** выполните следующие действия:

    1. Укажите уникальное **имя приложения**.
    2. В раскрывающемся списке выберите **подписку**. Эта подписка должна быть такой же, как и та, что связана со службой диспетчера данных StorSimple.
    3. Выберите **Создание** для группы ресурсов.
    4. В раскрывающемся списке **План размещения** выберите **План потребления**.
    5. Укажите расположение выполняемой функции. Используйте тот же регион, в котором находятся служба диспетчера данных StorSimple и учетная запись хранения, связанная с определением задания.
    6. Выберите существующую учетную запись хранения или параметр «Создать новую учетную запись хранения». Учетная запись хранения используется внутренне для функции.

        ![Введение данных конфигурации нового приложения-функции](./media/storsimple-data-manager-change-default-blob-path/function-app-parameters.png)

    7. Нажмите кнопку **Создать**. Приложение-функция создано.
     
        ![Созданное приложение-функция](./media/storsimple-data-manager-change-default-blob-path/function-app-created.png)

5. Выберите **Функции** и щелкните **+ New function** (+ Создать функцию).

    ![Щелкните + New function (+ Создать функцию)](./media/storsimple-data-manager-change-default-blob-path/create-new-function.png)

6. В качестве языка выберите **C#**. В массиве элементов шаблонов выберите язык **C#** в элементе **QueueTrigger-CSharp**.

7. В окне **Queue trigger** (Триггер очередей):

    1. Введите **имя** функции.
    2. В поле **Имя очереди** введите имя определения задания преобразования данных.
    3. В разделе **Подключение к учетной записи хранения** щелкните **Создать**. В списке учетных записей хранения выберите учетную запись, связанную с определением задания. Запишите имя подключения (выделено). Оно понадобится позже в функции Azure.

        ![Создание функции C#](./media/storsimple-data-manager-change-default-blob-path/new-function-parameters.png)

    4. Нажмите кнопку **Создать**. **Функция** создана.

     
10. В окне функции выполните файл _CSX_.

    ![Создать новую функцию C# 2](./media/storsimple-data-manager-change-default-blob-path/new-function-run-csx.png)
    
    Выполните следующие действия.

    1. Вставьте следующий код:

        ```
        using System;
        using System.Configuration;
        using Microsoft.WindowsAzure.Storage.Blob;
        using Microsoft.WindowsAzure.Storage.Queue;
        using Microsoft.WindowsAzure.Storage;
        using System.Collections.Generic;
        using System.Linq;

        public static void Run(QueueItem myQueueItem, TraceWriter log)
        {
            CloudStorageAccount storageAccount = CloudStorageAccount.Parse(ConfigurationManager.AppSettings["STORAGE_CONNECTIONNAME"]);

            string storageAccUriEndswith = "windows.net/";
            string uri = myQueueItem.TargetLocation.Replace("%20", " ");
            log.Info($"Blob Uri: {uri}");

            // Remove storage account uri string
            uri = uri.Substring(uri.IndexOf(storageAccUriEndswith) + storageAccUriEndswith.Length);

            string containerName = uri.Substring(0, uri.IndexOf("/")); 

            // Remove container name string
            uri = uri.Substring(containerName.Length + 1);

            // Current blob path
            string blobName = uri; 

            string volumeName = uri.Substring(containerName.Length + 1);
            volumeName = uri.Substring(0, uri.IndexOf("/"));

            // Remove volume name string
            uri = uri.Substring(volumeName.Length + 1);

            string newContainerName = uri.Substring(0, uri.IndexOf("/")).ToLower();
            string newBlobName = uri.Substring(newContainerName.Length + 1);

            log.Info($"Container name: {containerName}");
            log.Info($"Volume name: {volumeName}");
            log.Info($"New container name: {newContainerName}");

            log.Info($"Blob name: {blobName}");
            log.Info($"New blob name: {newBlobName}");

            // Create the blob client.
            CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

            // Container reference
            CloudBlobContainer container = blobClient.GetContainerReference(containerName);
            CloudBlobContainer newContainer = blobClient.GetContainerReference(newContainerName);
            newContainer.CreateIfNotExists();

            if(!container.Exists())
            {
                log.Info($"Container - {containerName} not exists");
                return;
            }

            if(!newContainer.Exists())
            {
                log.Info($"Container - {newContainerName} not exists");
                return;
            }

            CloudBlockBlob blob = container.GetBlockBlobReference(blobName);
            if (!blob.Exists())
            {
                // Skip to copy the blob to new container, if source blob doesn't exist
                log.Info($"The specified blob does not exist.");
                log.Info($"Blob Uri: {blob.Uri}");
                return;
            }

            CloudBlockBlob blobCopy = newContainer.GetBlockBlobReference(newBlobName);
            if (!blobCopy.Exists())
            {
                blobCopy.StartCopy(blob);
                // Delete old blob, after copy to new container
                blob.DeleteIfExists();
                log.Info($"Blob file path renamed completed successfully");
            }
            else
            {
                log.Info($"Blob file path renamed already done");
                // Delete old blob, if already exists.
                blob.DeleteIfExists();
            }
        }

        public class QueueItem
        {
            public string SourceLocation {get;set;}
            public long SizeInBytes {get;set;}
            public string Status {get;set;}
            public string JobID {get;set;}
            public string TargetLocation {get; set;}
        }

        ```

    2. В строке 11 замените имя **STORAGE_CONNECTIONNAME** именем подключения к учетной записи хранения (см. шаг 7в).

        ![Копирование имени подключения к хранилищу](./media/storsimple-data-manager-change-default-blob-path/new-function-storage-connection-name.png)

    3. **Сохраните** функцию.

        ![Сохранение функции](./media/storsimple-data-manager-change-default-blob-path/save-function.png)

12. Чтобы завершить создание функции, добавьте еще один файл, сделав следующее:

    1. Щелкните **Просмотреть файлы**.

       ![Ссылка "Просмотреть файлы"](./media/storsimple-data-manager-change-default-blob-path/view-files.png)

    2. Выберите **+ Добавить**.
        
        ![Добавить новую функцию Добавить файл](./media/storsimple-data-manager-change-default-blob-path/new-function-add-file.png)
    
    3. Введите **project.js**, а затем нажмите клавишу **Ввод**. В файле **project.json** скопируйте следующий код:

        ```
        {
        "frameworks": {
            "net46":{
            "dependencies": {
                "windowsazure.storage": "8.1.1"
            }
            }
        }
        }

        ```

    
    4. Выберите команду **Сохранить**.

        ![Новый проект функции JSON](./media/storsimple-data-manager-change-default-blob-path/new-function-project-json.png)

Вы создали функцию Azure. Эта функция запускается каждый раз, когда задание преобразования данных создает большой двоичный объект.

## <a name="next-steps"></a>Дальнейшие действия

[Использование пользовательского интерфейса для службы диспетчера данных StorSimple (закрытая предварительная версия)](storsimple-data-manager-ui.md)
