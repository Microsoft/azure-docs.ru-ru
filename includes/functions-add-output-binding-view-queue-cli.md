---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 03/25/2020
ms.author: glenga
ms.openlocfilehash: 921127ffdd35007cc3fa2eaaa95cdb3fac8bbe15
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107782291"
---
Очередь можно просмотреть на [портале Azure](../articles/storage/queues/storage-quickstart-queues-portal.md) или в [Обозревателе службы хранилища Azure](https://storageexplorer.com/). Очередь также можно просмотреть в интерфейсе командной строки Azure. Для этого выполните приведенные ниже шаги:

1. Откройте файл *local.setting.json* проекта функций и скопируйте значение строки подключения. В окне терминала или командной строки выполните следующую команду, чтобы создать переменную среды с именем `AZURE_STORAGE_CONNECTION_STRING`. Вставьте конкретную строку подключения вместо `<MY_CONNECTION_STRING>`. (Эта переменная среды означает, что вам не нужно указывать строку подключения для каждой последующей команды с помощью аргумента `--connection-string`.)

    # <a name="bash"></a>[bash](#tab/bash)
    
    ```bash
    export AZURE_STORAGE_CONNECTION_STRING="<MY_CONNECTION_STRING>"
    ```
    
    # <a name="powershell"></a>[PowerShell](#tab/powershell)
    
    ```powershell
    $env:AZURE_STORAGE_CONNECTION_STRING = "<MY_CONNECTION_STRING>"
    ```
    
    # <a name="azure-cli"></a>[Azure CLI](#tab/cmd)
    
    ```azurecli
    set AZURE_STORAGE_CONNECTION_STRING="<MY_CONNECTION_STRING>"
    ```
    
    ---
    
1. (Дополнительно) Команду [`az storage queue list`](/cli/azure/storage/queue#az_storage_queue_list) можно использовать для просмотра очередей службы хранилища в учетной записи. В выходных данных этой команды должна быть очередь с именем `outqueue`, созданная при написании функцией первого сообщения в этой очереди.
    
    ```azurecli
    az storage queue list --output tsv
    ```

1. Используйте команду [`az storage message get`](/cli/azure/storage/message#az_storage_message_get), чтобы прочитать сообщение из этой очереди. Необходимо указывать первое имя, использованное ранее при проверке функции. Команда считывает и удаляет первое сообщение из очереди. 

    # <a name="bash"></a>[bash](#tab/bash)
    
    ```bash
    echo `echo $(az storage message get --queue-name outqueue -o tsv --query '[].{Message:content}') | base64 --decode`
    ```
    
    # <a name="powershell"></a>[PowerShell](#tab/powershell)
    
    ```powershell
    [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($(az storage message get --queue-name outqueue -o tsv --query '[].{Message:content}')))
    ```
    
    # <a name="azure-cli"></a>[Azure CLI](#tab/cmd)
    
    ```azurecli
    az storage message get --queue-name outqueue -o tsv --query [].{Message:content} > %TEMP%out.b64 && certutil -decode -f %TEMP%out.b64 %TEMP%out.txt > NUL && type %TEMP%out.txt && del %TEMP%out.b64 %TEMP%out.txt /q
    ```

    Этот сценарий использует certutil для декодирования коллекции сообщений в кодировке Base64 из локального временного файла. Если выходные данные отсутствуют, попробуйте удалить `> NUL` из сценария, чтобы отключить подавление выходных данных certutil, если возникает ошибка. 
    
    ---
    
    Так как текст сообщения хранится [в кодировке Base64](../articles/azure-functions/functions-bindings-storage-queue-trigger.md#encoding), перед отображением сообщения его необходимо декодировать. После выполнения `az storage message get` сообщение удаляется из очереди. Если в `outqueue` было только одно сообщение, вы не получите сообщение при повторном выполнении этой команды, вместо этого возникнет ошибка.
