---
title: включить файл
description: включить файл
services: storage
author: tamram
ms.service: storage
ms.topic: include
ms.date: 01/19/2021
ms.author: tamram
ms.custom: include file
ms.openlocfilehash: 0914cf9515930e23e4134181ffe8332e36eacffe
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98612938"
---
Хранилище BLOB-объектов Azure — это решение корпорации Майкрософт для хранения объектов в облаке. Хранилище BLOB-объектов оптимизировано для хранения больших объемов неструктурированных данных. Неструктурированные данные — это данные, которые не соответствуют определенной модели данных или определению, например текстовых или двоичных данных.

## <a name="about-blob-storage"></a>Сведения о хранилище BLOB-объектов

Хранилище BLOB-объектов предназначено для следующих задач:

* Обслуживание изображений или документов непосредственно в браузере.
* Хранение файлов для распределенного доступа.
* Потоковая передача видео и звука.
* Запись в файлы журнала.
* Хранение резервных копий и восстановление данных, аварийное восстановление и архивация.
* Хранение данных для анализа локальной службой или службой, размещенной в Azure.

Пользователи или клиентские приложения могут получить доступ к объектам в хранилище BLOB-объектов через HTTP/HTTPS из любой точки мира. К объектам в хранилище BLOB-объектов можно обращаться через [REST API службы хранилища Azure](/rest/api/storageservices/blob-service-rest-api), [Azure PowerShell](/powershell/module/az.storage), [Azure CLI](/cli/azure/storage) или клиентскую библиотеку службы хранилища Azure. Клиентские библиотеки доступны для разных языков, включая:

* [.NET](/dotnet/api/overview/azure/storage)
* [Java](/java/api/overview/azure/storage)
* [Node.js](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/storage)
* [Python](../articles/storage/blobs/storage-quickstart-blobs-python.md)
* [GO](https://github.com/azure/azure-storage-blob-go/)
* [PHP](https://azure.github.io/azure-storage-php/)
* [Ruby](https://azure.github.io/azure-storage-ruby)

## <a name="about-azure-data-lake-storage-gen2"></a>Сведения об Azure Data Lake Storage 2-го поколения

Хранилище BLOB-объектов поддерживает Azure Data Lake Storage 2-го поколения, решение аналитики больших данных корпорации Майкрософт, предназначенное для облака. Azure Data Lake Storage 2-го поколения предлагает иерархическую файловую систему, а также преимущества хранилища BLOB-объектов:

* экономичное многоуровневое хранилище;
* Высокий уровень доступности
* Строгая согласованность
* возможности аварийного восстановления.

Общие сведения об Azure Data Lake Storage 2-го поколения см. в [этой статье](../articles/storage/blobs/data-lake-storage-introduction.md).