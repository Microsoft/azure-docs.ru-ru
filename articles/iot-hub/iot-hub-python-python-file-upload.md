---
title: Отправка файлов с устройств в Центр Интернета вещей с помощью Python | Документация Майкрософт
description: Сведения об отправке файлов с устройства в облако с помощью пакета SDK для устройств Azure IoT для Python. Отправленные файлы хранятся в контейнере больших двоичных объектов службы хранилища Azure.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: python
ms.topic: conceptual
ms.date: 03/31/2020
ms.author: robinsh
ms.custom: mqtt, devx-track-python
ms.openlocfilehash: 77d51b2c839a64567838fa4d6308d203a6bb8b82
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102501147"
---
# <a name="upload-files-from-your-device-to-the-cloud-with-iot-hub-python"></a>Передача файлов с устройства в облако с помощью центра Интернета вещей (Python)

[!INCLUDE [iot-hub-file-upload-language-selector](../../includes/iot-hub-file-upload-language-selector.md)]

В этой статье описано, как использовать [возможности передачи файлов Центра Интернета вещей](iot-hub-devguide-file-upload.md), чтобы передать файл в [хранилище BLOB-объектов Azure](../storage/index.yml). В этом руководстве описаны следующие процедуры.

* Безопасное предоставление контейнера хранилища для передачи файла.

* Использование клиента Python для передачи файла через Центр Интернета вещей.

В кратком руководстве [Отправка данных телеметрии из устройства в Центр Интернета вещей](quickstart-send-telemetry-python.md) демонстрируется базовая функциональность Центра Интернета вещей при передаче сообщений с устройства в облако. Тем не менее в некоторых случаях не просто сопоставить данные, отправляемые устройством, с относительно небольшими сообщениями, отправляемыми из устройства в облако, которые принимает Центр Интернета вещей. При передаче файлов с устройства вы можете рассчитывать на безопасность и надежность Центра Интернета вещей.

По завершении работы с этим руководством вы запустите консольное приложение Python:

* **FileUpload.py**, которое отправляет файл в хранилище с помощью пакета SDK для устройств для Python.

[!INCLUDE [iot-hub-include-python-sdk-note](../../includes/iot-hub-include-python-sdk-note.md)]

[!INCLUDE [iot-hub-include-x509-ca-signed-file-upload-support-note](../../includes/iot-hub-include-x509-ca-signed-file-upload-support-note.md)]

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [iot-hub-include-python-v2-async-installation-notes](../../includes/iot-hub-include-python-v2-async-installation-notes.md)]

* Убедитесь, что в брандмауэре открыт порт 8883. Пример устройства в этой статье использует протокол MQTT, который передает данные через порт 8883. В некоторых корпоративных и академических сетях этот порт может быть заблокирован. Дополнительные сведения и способы устранения этой проблемы см. в разделе о [подключении к Центру Интернета вещей по протоколу MQTT](iot-hub-mqtt-support.md#connecting-to-iot-hub).

[!INCLUDE [iot-hub-associate-storage](../../includes/iot-hub-associate-storage.md)]

## <a name="upload-a-file-from-a-device-app"></a>Передача файла из приложения устройства

В этом разделе вы создадите приложение для устройства, чтобы отправлять файлы в Центр Интернета вещей.

1. В командной строке выполните следующую команду, чтобы установить пакет **Azure-IOT-Device** . Этот пакет используется для координации передачи файлов в центр Интернета вещей.

    ```cmd/sh
    pip install azure-iot-device
    ```

1. В командной строке выполните следующую команду, чтобы установить пакет [**Azure. Storage. BLOB**](https://pypi.org/project/azure-storage-blob/) . Этот пакет используется для отправки файла.

    ```cmd/sh
    pip install azure.storage.blob
    ```

1. Создайте тестовый файл, который будет отправлен в хранилище BLOB-объектов.

1. В текстовом редакторе создайте в рабочей папке файл **FileUpload.py**.

1. Добавьте следующие инструкции `import` и переменные в начало файла **FileUpload.py**.

    ```python
    import os
    import asyncio
    from azure.iot.device.aio import IoTHubDeviceClient
    from azure.core.exceptions import AzureError
    from azure.storage.blob import BlobClient

    CONNECTION_STRING = "[Device Connection String]"
    PATH_TO_FILE = r"[Full path to local file]"
    ```

1. В файле замените `[Device Connection String]` строкой подключения устройства Центра Интернета вещей. Замените на `[Full path to local file]` путь к созданному файлу теста или на любой файл на устройстве, который требуется передать.

1. Создайте функцию для передачи файла в хранилище BLOB-объектов:

    ```python
    async def store_blob(blob_info, file_name):
        try:
            sas_url = "https://{}/{}/{}{}".format(
                blob_info["hostName"],
                blob_info["containerName"],
                blob_info["blobName"],
                blob_info["sasToken"]
            )

            print("\nUploading file: {} to Azure Storage as blob: {} in container {}\n".format(file_name, blob_info["blobName"], blob_info["containerName"]))

            # Upload the specified file
            with BlobClient.from_blob_url(sas_url) as blob_client:
                with open(file_name, "rb") as f:
                    result = blob_client.upload_blob(f, overwrite=True)
                    return (True, result)

        except FileNotFoundError as ex:
            # catch file not found and add an HTTP status code to return in notification to IoT Hub
            ex.status_code = 404
            return (False, ex)

        except AzureError as ex:
            # catch Azure errors that might result from the upload operation
            return (False, ex)
    ```

    Эта функция анализирует структуру *blob_info* , переданную в нее, чтобы создать URL-адрес, используемый для инициализации объекта [Azure. Storage. BLOB. блобклиент](/python/api/azure-storage-blob/azure.storage.blob.blobclient). Затем он отправляет файл в хранилище BLOB-объектов Azure с помощью этого клиента.

1. Добавьте следующий код, чтобы подключить клиент и передать файл:

    ```python
    async def main():
        try:
            print ( "IoT Hub file upload sample, press Ctrl-C to exit" )

            conn_str = CONNECTION_STRING
            file_name = PATH_TO_FILE
            blob_name = os.path.basename(file_name)

            device_client = IoTHubDeviceClient.create_from_connection_string(conn_str)

            # Connect the client
            await device_client.connect()

            # Get the storage info for the blob
            storage_info = await device_client.get_storage_info_for_blob(blob_name)

            # Upload to blob
            success, result = await store_blob(storage_info, file_name)

            if success == True:
                print("Upload succeeded. Result is: \n") 
                print(result)
                print()

                await device_client.notify_blob_upload_status(
                    storage_info["correlationId"], True, 200, "OK: {}".format(file_name)
                )

            else :
                # If the upload was not successful, the result is the exception object
                print("Upload failed. Exception is: \n") 
                print(result)
                print()

                await device_client.notify_blob_upload_status(
                    storage_info["correlationId"], False, result.status_code, str(result)
                )

        except Exception as ex:
            print("\nException:")
            print(ex)

        except KeyboardInterrupt:
            print ( "\nIoTHubDeviceClient sample stopped" )

        finally:
            # Finally, disconnect the client
            await device_client.disconnect()


    if __name__ == "__main__":
        asyncio.run(main())
        #loop = asyncio.get_event_loop()
        #loop.run_until_complete(main())
        #loop.close()
    ```

    Этот код создает асинхронный **иосубдевицеклиент** и использует следующие API для управления передачей файлов в центр Интернета вещей:

    * **get_storage_info_for_blob** получает сведения из центра Интернета вещей о связанной учетной записи хранения, созданной ранее. Эти сведения включают имя узла и контейнера, имя большого двоичного объекта и маркер SAS. Сведения о хранилище передаются функции **store_blob** (созданной на предыдущем шаге), поэтому **блобклиент** в этой функции может пройти проверку подлинности в службе хранилища Azure. Метод **get_storage_info_for_blob** также возвращает correlation_id, который используется в методе **notify_blob_upload_status** . Correlation_id — это способ маркировки большого двоичного объекта, над которым вы работаете.

    * **notify_blob_upload_status** уведомляет центр Интернета вещей о состоянии операции хранилища BLOB-объектов. Он передается correlation_id, полученным методом **get_storage_info_for_blob** . Он используется центром Интернета вещей для уведомления любой службы, которая может прослушивать уведомление о состоянии задачи передачи файлов.

1. Сохраните и закройте файл **UploadFile.py**.

## <a name="run-the-application"></a>Выполнение приложения

Теперь все готово для запуска приложения.

1. В командной строке в рабочей папке выполните следующую команду.

    ```cmd/sh
    python FileUpload.py
    ```

2. На следующем снимке экрана показаны выходные данные приложения **FileUpload**.

    ![Выходные данные приложения simulated-device](./media/iot-hub-python-python-file-upload/run-device-app.png)

3. На портале можно просмотреть отправленный файл в контейнере хранилища, который вы настроили.

    ![Отправленный файл](./media/iot-hub-python-python-file-upload/view-blob.png)

## <a name="next-steps"></a>Дальнейшие действия

В этом руководство показано, как использовать возможности передачи файлов Центра Интернета вещей, чтобы упростить передачу файлов из устройств. Изучение функций и сценариев Центра Интернета вещей можно продолжить в следующих руководствах:

* [Создание Центра Интернета вещей с помощью шаблона Azure Resource Manager (PowerShell)](iot-hub-rm-template-powershell.md)

* [Пакет SDK для устройств Azure IoT для C](iot-hub-device-sdk-c-intro.md)

* [Пакеты SDK для Центра Интернета вещей Azure](iot-hub-devguide-sdks.md)

Дополнительные сведения о хранилище BLOB-объектов Azure см. по следующим ссылкам:

* [Документация по хранилищу BLOB-объектов Azure](../storage/blobs/index.yml)

* [Документация по службе хранилища BLOB-объектов Azure для API Python](/python/api/overview/azure/storage-blob-readme)