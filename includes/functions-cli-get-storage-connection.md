---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 03/25/2020
ms.author: glenga
ms.openlocfilehash: 5cb345ef2d20f75066e90f9e6478be27f925b1b0
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "80673384"
---
### <a name="retrieve-the-azure-storage-connection-string"></a>Получение строки подключения к Службе хранилища Azure

Ранее вы создали учетную запись хранения Azure для использования приложением-функции. Строка подключения данной учетной записи надежно хранится в параметрах приложения в Azure. Скачав параметр в файл *local.settings.json*, вы можете использовать это подключение для записи данных в очередь службы хранилища в той же учетной записи при локальном запуске функции. 

1. В корневом каталоге проекта выполните следующую команду, заменив `<app_name>` именем приложения-функции из предыдущего краткого руководства. Эта команда перезапишет все существующие значения в файле.

    ```
    func azure functionapp fetch-app-settings <app_name>
    ```
    
1. Откройте файл *local.settings.json* и найдите значение `AzureWebJobsStorage`, которое является строкой подключения к учетной записи хранения. В других разделах этой статьи вы будете использовать имя `AzureWebJobsStorage` и строку подключения.

> [!IMPORTANT]
> Файл *local.settings.json* содержит секреты, скачанные из Azure, поэтому всегда исключайте этот файл из системы управления версиями. Файл *.gitignore*, созданный с помощью локального проекта функций, по умолчанию исключает файл.