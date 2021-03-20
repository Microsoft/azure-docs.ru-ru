---
title: Ограничения Azure Data Box Heavy | Документация Майкрософт
description: Описание системных ограничений и рекомендуемых размеров для Microsoft Azure Data Box Heavy компонентов и подключений.
services: databox
author: alkohli
ms.service: databox
ms.subservice: heavy
ms.topic: article
ms.date: 08/28/2019
ms.author: alkohli
ms.openlocfilehash: 76e543a5bec749db55eb77649d2f6e0526f2cb1d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "84705565"
---
# <a name="azure-data-box-heavy-limits"></a>Ограничения Azure Data Box Heavy

Учитывайте эти ограничения при развертывании и эксплуатации устройства Azure Data Box Heavy. Ограничения для Data Box описаны в следующей таблице.


## <a name="data-box-heavy-service-limits"></a>Ограничения службы Data Box Heavy

[!INCLUDE [data-box-service-limits](../../includes/data-box-service-limits.md)]

## <a name="data-box-heavy-limits"></a>Ограничения Data Box Heavy

- Data Box Heavy может хранить не более 1 000 000 000 файлов на каждом узле.
- Data Box Heavy поддерживает не более 512 контейнеров или общих папок для каждого узла в облаке. Каталоги верхнего уровня в общей папке пользователя преобразуются в контейнеры или общие файловые ресурсы Azure в облаке.

## <a name="azure-storage-limits"></a>Ограничения хранилища Azure

[!INCLUDE [data-box-storage-limits](../../includes/data-box-storage-limits.md)]

## <a name="data-upload-caveats"></a>Предупреждения, связанные с передачей данных

[!INCLUDE [data-box-data-upload-caveats](../../includes/data-box-data-upload-caveats.md)]

## <a name="azure-storage-account-size-limits"></a>Ограничения размера для учетной записи хранения Azure

[!INCLUDE [data-box-storage-account-size-limits](../../includes/data-box-storage-account-size-limits.md)]

## <a name="azure-object-size-limits"></a>Ограничения размера для объектов Azure

[!INCLUDE [data-box-object-size-limits](../../includes/data-box-object-size-limits.md)]

## <a name="azure-block-blob-page-blob-and-file-naming-conventions"></a>Соглашения об именовании для файлов, блочных и страничных BLOB-объектов Azure

[!INCLUDE [data-box-naming-conventions](../../includes/data-box-naming-conventions.md)]
