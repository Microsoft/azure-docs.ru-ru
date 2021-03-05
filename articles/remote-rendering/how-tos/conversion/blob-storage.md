---
title: Использование хранилища BLOB-объектов Azure для преобразования модели
description: Описание общих действий по настройке и использованию хранилища BLOB-объектов для преобразования модели.
author: jakrams
ms.author: jakras
ms.date: 02/04/2020
ms.topic: how-to
ms.openlocfilehash: 889c25fd1402c4438ff648afce64905de112c8db
ms.sourcegitcommit: f7eda3db606407f94c6dc6c3316e0651ee5ca37c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102215550"
---
# <a name="use-azure-blob-storage-for-model-conversion"></a>Использование хранилища BLOB-объектов Azure для преобразования модели

Службе [преобразования модели](model-conversion.md) требуется доступ к хранилищу BLOB-объектов Azure, чтобы он мог получать входные данные и сохранять выходные данные. В этой статье описывается, как выполнять самые распространенные действия.

## <a name="prepare-azure-storage-accounts"></a>Подготовка учетных записей хранения Azure

- Создание учетной записи хранения (StorageV2)
- Создайте входной контейнер больших двоичных объектов в учетной записи хранения (например, с именем "арринпут").
- Создайте выходной контейнер больших двоичных объектов в учетной записи хранения (например, с именем "арраутпут").

> [!TIP]
> Пошаговые инструкции по настройке учетной записи хранения см [. в кратком руководстве по преобразованию модели для подготовки к просмотру.](../../quickstarts/convert-model.md)

Создание учетной записи хранения и контейнеров больших двоичных объектов можно выполнить с помощью одного из следующих средств:

- [Портал Azure](https://portal.azure.com)
- [AZ — Командная строка](/cli/azure/install-azure-cli)
- [Обозреватель службы хранилища Azure](https://azure.microsoft.com/features/storage-explorer/)
- Пакеты SDK (C#, Python...)

## <a name="ensure-azure-remote-rendering-can-access-your-storage-account"></a>Убедитесь, что служба удаленной подготовки Azure может получить доступ к вашей учетной записи хранения.

Служба удаленного отрисовки Azure должна получить данные модели из учетной записи хранения и записать в нее данные.

Доступ к учетной записи хранения можно предоставить для удаленной отрисовки Azure следующими двумя способами.

### <a name="connect-your-azure-storage-account-with-your-azure-remote-rendering-account"></a>Подключение учетной записи хранения Azure к учетной записи удаленной подготовки Azure

Выполните действия, указанные в разделе [Создание учетной записи](../create-an-account.md#link-storage-accounts) .

### <a name="retrieve-sas-for-the-storage-containers"></a>Получение SAS для контейнеров хранилища

Сохраненные подписи доступа (SAS) используются для предоставления доступа для чтения входных данных и доступа на запись для выходных данных. Мы советуем создавать новые коды URI при каждом преобразовании модели. Так как URI истекает через некоторое время, сохранение их в течение более длительного времени может привести к неожиданному повреждению приложения.

Сведения о SAS см. в [документации SAS](../../../storage/common/storage-sas-overview.md).

URI SAS можно создать с помощью одного из следующих средств:

- AZ PowerShell Module
  - см. [примеры сценариев PowerShell](../../samples/powershell-example-scripts.md) .
- [AZ — Командная строка](/cli/azure/install-azure-cli)
- [Обозреватель службы хранилища Azure](https://azure.microsoft.com/features/storage-explorer/)
  - Щелкните правой кнопкой мыши контейнер "получить подписанный URL-доступ" (чтение, получение списка доступа для входного контейнера, доступ на запись для контейнера выходных данных).
- Пакеты SDK (C#, Python...)

Пример использования подписанных URL-символов в преобразовании ресурса приведен в Conversion.ps1 [примеров сценариев PowerShell](../../samples/powershell-example-scripts.md#script-conversionps1).

## <a name="upload-an-input-model"></a>Отправка входной модели

Чтобы начать преобразование модели, необходимо передать его с помощью одного из следующих параметров:

- [Обозреватель службы хранилища Azure](https://azure.microsoft.com/features/storage-explorer/) — удобный пользовательский интерфейс для отправки, скачивания и управления файлами в хранилище BLOB-объектов Azure.
- [Командная строка Azure](../../../storage/blobs/storage-quickstart-blobs-cli.md)
- [Overview of Azure PowerShell](/powershell/azure/install-az-ps?view=azps-2.2.0) (Общие сведения об Azure PowerShell)
  - см. [примеры сценариев PowerShell](../../samples/powershell-example-scripts.md) .
- [Использование пакета SDK для хранилища (Python, C#...)](../../../storage/index.yml)
- [Использование API-интерфейсов службы хранилища Azure](/rest/api/storageservices/blob-service-rest-api)

Пример передачи данных для преобразования см. в Conversion.ps1 [примеров сценариев PowerShell](../../samples/powershell-example-scripts.md#script-conversionps1).

## <a name="get-a-sas-uri-for-the-converted-model"></a>Получение URI SAS для преобразованной модели

Этот шаг аналогичен [извлечению SAS для контейнеров хранилища](#retrieve-sas-for-the-storage-containers). Однако на этот раз необходимо получить URI SAS для файла модели, который был записан в контейнер выходных данных.

Например, чтобы получить URI SAS через [Обозреватель службы хранилища Azure](https://azure.microsoft.com/features/storage-explorer/), щелкните правой кнопкой мыши файл модели и выберите "получить подпись общего доступа".

Подписанный URL-адрес (SAS) требуется для загрузки моделей, если учетная запись хранения не была подключена к учетной записи удаленной отрисовки Azure. Вы можете узнать, как подключить учетную запись в статье [Создание учетной записи](../create-an-account.md#link-storage-accounts).

## <a name="next-steps"></a>Дальнейшие действия

- [Настройка преобразования модели](configure-model-conversion.md)
- [Преобразование модели REST API](conversion-rest-api.md)
