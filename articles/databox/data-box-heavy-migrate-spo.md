---
title: Использование Azure Data Box Heavy для перемещения содержимого файлового ресурса в SharePoint Online
description: В этом руководстве описывается, как перенести содержимое общей папки в SharePoint Online с помощью Azure Data Box Heavy.
services: databox
author: alkohli
ms.service: databox
ms.subservice: heavy
ms.topic: how-to
ms.date: 07/18/2019
ms.author: alkohli
ms.openlocfilehash: b00a944b69b8492d4aa00258154bf00f1c66ad33
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92127120"
---
# <a name="use-the-azure-data-box-heavy-to-migrate-your-file-share-content-to-sharepoint-online"></a>Использование Azure Data Box Heavy для переноса содержимого общей папки в SharePoint Online

С помощью Azure Data Box Heavy и средства миграции SharePoint (SPMT) легко перенести содержимое общей папки в SharePoint Online и OneDrive. С помощью Data Box Heavy можно устранить зависимость от канала глобальной сети, используемого для передачи данных.

Microsoft Azure Data Box — это служба, которая позволяет заказать устройство на портале Microsoft Azure. Затем вы можете скопировать на это устройство терабайты данных со своих серверов. После отправки устройства обратно в корпорацию Майкрософт данные будут скопированы в Azure. В зависимости от объема данных, которые планируется перенести, можно выбрать:

- [Диск Data Box](./data-box-disk-overview.md) с 35 ТБ используемой емкости на заказ, предназначенный для малых и средних наборов данных;
- [Data Box](./data-box-overview.md) с 80 ТБ используемой емкости на устройство, предназначенный для средних и крупных наборов данных;
- [Data Box Heavy](./data-box-heavy-overview.md) с 770 ТБ используемой емкости на устройство, предназначенный для крупных наборов данных.

В этой статье рассказывается о том, как использовать Data Box Heavy для переноса содержимого общей папки в SharePoint Online.

## <a name="requirements-and-costs"></a>Требования и затраты

### <a name="for-data-box-heavy"></a>Data Box Heavy

- Data Box Heavy доступен только для подписок с Соглашением Enterprise (EA), поставщика облачных решений (CSP) или в рамках спонсорских предложений Azure. Если вы используете подписку другого типа, обратитесь в службу поддержки Майкрософт и обновите подписку. Вы можете также ознакомиться с [ценами на подписки Azure](https://azure.microsoft.com/pricing/).
- За использование Data Box Heavy взимается плата. Обязательно ознакомьтесь с [ценами на Data Box Heavy](https://azure.microsoft.com/pricing/details/databox/heavy/).


### <a name="for-sharepoint-online"></a>SharePoint Online

- Просмотрите [минимальные требования для средства миграции SharePoint (SPMT)](/sharepointmigration/how-to-use-the-sharepoint-migration-tool).

## <a name="workflow-overview"></a>Обзор рабочего процесса

Этот рабочий процесс включает в себя действия с Azure Data Box Heavy и SharePoint Online.
Следующие действия относятся к Azure Data Box Heavy.

1. Закажите Azure Data Box Heavy.
2. Получите и настройте устройство.
3. Скопируйте данные из локальной общей папки в папку для Файлов Azure на устройстве.
4. После завершения копирования отправьте устройство обратно согласно инструкциям.
5. Дождитесь полной передачи данных в Azure.

Следующие действия относятся к SharePoint Online.

6. Создайте виртуальную машину на портале Azure и подключите к ней общую папку Azure.
7. Установите инструмент SPMT на виртуальной машине Azure.
8. Запустите инструмент SPMT, указав общую папку Azure *источник*.
9. Выполните завершающие действия в инструменте.
10. Проверьте и подтвердите свои данные.

## <a name="use-data-box-heavy-to-copy-data"></a>Копирование данных в Data Box Heavy

Чтобы скопировать данные в Data Box Heavy, сделайте следующее.

1. [Закажите Data Box Heavy](data-box-heavy-deploy-ordered.md).
2. После получения Data Box Heavy [настройте это устройство](data-box-heavy-deploy-set-up.md). Следует подсоединить кабели к обоим узлам на устройстве и настроить их.
3. [Копирование данных в Azure Data Box Heavy](data-box-heavy-deploy-copy-data.md). При копировании соблюдайте следующие указания.

    - Для копирования данных используйте только папку *StorageAccountName_AzFile* в Data Box Heavy. Таким образом ваши данные окажутся в общей папке Azure, а не в блочных или страничных BLOB-объектах.
    - Скопируйте файлы в папку в папке *StorageAccountName_AzFile*. Вложенная папка в папке *StorageAccountName_AzFile* будет преобразована в общую папку. При копировании всех файлов непосредственно в папку *StorageAccountName_AzFile* возникает ошибка, после чего они отправляются как блочные BLOB-объекты. Эта общая папка будет подключена к виртуальной машине на следующем шаге.
    - Скопируйте данные на оба узла Data Box Heavy.
3. Выполните [подготовку к отправке](data-box-heavy-deploy-picked-up.md#prepare-to-ship) на своем устройстве. Успешная подготовка к отправке гарантирует передачу файлов в Azure.
4. [Верните устройство](data-box-heavy-deploy-picked-up.md#ship-data-box-heavy-back).
5. [Проверьте передачу данных в Azure](data-box-heavy-deploy-picked-up.md#verify-data-upload-to-azure).

## <a name="use-spmt-to-migrate-data"></a>Перенос данных с помощью SPMT

Получив от группы обработки данных Azure подтверждение того, что копирование данных завершено, приступайте к их переносу в SharePoint Online.

Чтобы обеспечить оптимальную производительность и использование сети, рекомендуется создать виртуальную машину Azure.

1. Войдите на портал Azure и [создайте виртуальную машину](../virtual-machines/windows/quick-create-portal.md).
2. [Подключите общую папку Azure к этой виртуальной машине](../storage/files/storage-how-to-use-files-windows.md#mount-the-azure-file-share-with-file-explorer).
3. [Скачайте средство миграции SharePoint](https://spmtreleasescus.blob.core.windows.net/install/default.htm) и установите его на виртуальной машине Azure.
4. Запустите средство миграции SharePoint. Щелкните **Вход** и введите имя пользователя и пароль для рабочей или учебной учетной записи.
5. При появлении запроса **Где находятся ваши данные?** выберите **Общая папка**. Введите путь к общей папке Azure, в которой находятся ваши данные.
6. Следуйте указаниям в остальных запросах как обычно, включая выбор целевого расположения. Дополнительные сведения см. в разделе [Использование средства миграции SharePoint](/sharepointmigration/how-to-use-the-sharepoint-migration-tool).

> [!IMPORTANT]
> - Скорость, с которой данные принимаются в SharePoint Online, зависит от нескольких факторов, независимо от того, находятся ли ваши данные в Azure. Понимание этих факторов поможет вам спланировать перенос и повысить его эффективность.  Дополнительные сведения см. в разделе [Рекомендации по улучшению производительности миграции в SharePoint и OneDrive](/sharepointmigration/sharepoint-online-and-onedrive-migration-speed).
> - При переносе данных в SharePoint Online существует риск потери установленных разрешений для файлов. Кроме того, возможна потеря части метаданных, таких как значения *Кем создано* и *Дата изменения*.

## <a name="next-steps"></a>Дальнейшие действия

[Закажите Data Box Heavy](./data-box-heavy-deploy-ordered.md)