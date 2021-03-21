---
title: Ограничения Azure Data Box | Документация Майкрософт
description: В этой статье описаны ограничения системы и рекомендуемые размеры для компонентов и подключений Microsoft Azure Data Box.
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: article
ms.date: 01/05/2021
ms.author: alkohli
ms.openlocfilehash: 97d8da86565db73aa9a3866f39f793aaf0905470
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97900167"
---
# <a name="azure-data-box-limits"></a>Ограничения Azure Data Box

Учитывайте эти ограничения, когда развертываете и используете Microsoft Azure Data Box. Ограничения для Data Box описаны в следующей таблице.

## <a name="data-box-service-limits"></a>Ограничения службы Data Box

[!INCLUDE [data-box-service-limits](../../includes/data-box-service-limits.md)]

## <a name="data-box-limits"></a>Ограничения Data Box

- Data Box может хранить не более 500 000 000 файлов как для импорта, так и для экспорта.
- Data Box поддерживает не более 512 контейнеров или общих папок в облаке. Каталоги верхнего уровня в общей папке пользователя преобразуются в контейнеры или общие файловые ресурсы Azure в облаке. 
- Емкость Data Box использования может быть меньше 80 ТБ из-за использования пространства метаданных ReFS.
- Data Box поддерживает не более 10 клиентских подключений за раз в общем ресурсе NFS.

## <a name="azure-storage-limits"></a>Ограничения службы хранилища Azure

[!INCLUDE [data-box-storage-limits](../../includes/data-box-storage-limits.md)]

## <a name="data-upload-caveats"></a>Предупреждения, связанные с передачей данных


### <a name="for-import-order"></a>Порядок импорта

Data Box предостережений для заказа на импорт:

[!INCLUDE [data-box-data-upload-caveats](../../includes/data-box-data-upload-caveats.md)]

### <a name="for-export-order"></a>Для порядка экспорта

Data Box предостережений для порядка экспорта:

- Data Box является устройством на основе Windows и не поддерживает имена файлов с учетом регистра. Например, в Azure можно использовать два разных файла с именами, которые просто различаются по регистру. Не используйте Data Box для экспорта таких файлов, так как файлы будут перезаписаны на устройстве.
- При наличии повторяющихся тегов во входных файлах или тегах, ссылающихся на одни и те же данные, Data Box экспорта может пропустить или перезаписать файлы. Количество файлов и размер данных, отображаемых портал Azure, может отличаться от фактического размера данных на устройстве. 
- Data Box экспорт данных в систему на основе Windows через SMB и ограничивается ограничениями SMB для файлов и папок. Файлы и папки с неподдерживаемыми именами не экспортируются.
- Существует сопоставление 1:1 из префикса в контейнер.
- Максимальный размер имени файла — 1024 символов. Имена файлов, превышающие эту длину, не экспортируются.
- Выполняется экспорт повторяющихся префиксов в *XML-* файл (переданный во время создания заказа). Дубликаты префиксов не пропускаются.
- Страничные BLOB-объекты и имена контейнеров учитывают регистр. Если регистр не совпадает, большой двоичный объект и контейнер не будут найдены.
 

## <a name="azure-storage-account-size-limits"></a>Ограничения размера для учетной записи хранения Azure

[!INCLUDE [data-box-storage-account-size-limits](../../includes/data-box-storage-account-size-limits.md)]

## <a name="azure-object-size-limits"></a>Ограничения размера для объектов Azure

[!INCLUDE [data-box-object-size-limits](../../includes/data-box-object-size-limits.md)]

## <a name="azure-block-blob-page-blob-and-file-naming-conventions"></a>Соглашения об именовании для файлов, блочных и страничных BLOB-объектов Azure

[!INCLUDE [data-box-naming-conventions](../../includes/data-box-naming-conventions.md)]
