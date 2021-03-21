---
title: Ограничения R для Azure Stack ребра Pro | Документация Майкрософт
description: Описание системных ограничений и рекомендуемых размеров для Azure Stack пограничных Pro R.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: conceptual
ms.date: 10/13/2020
ms.author: alkohli
ms.openlocfilehash: dfff3bdd716c54a6c83dbc9fec63c794c1fba85b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96467630"
---
# <a name="azure-stack-edge-pro-r-limits"></a>Ограничения R для Azure Stack ребра Pro

Эти ограничения следует учитывать при развертывании и эксплуатации решения R для Azure Stack пограничной версии Pro.

## <a name="azure-stack-edge-pro-r-service-limits"></a>Ограничения службы R для Azure Stack ребра Pro

[!INCLUDE [azure-stack-edge-gateway-service-limits](../../includes/azure-stack-edge-gateway-service-limits.md)]

## <a name="azure-stack-edge-pro-r-device-limits"></a>Ограничения устройства R для Azure Stack ребра Pro

В следующей таблице описаны ограничения для устройства R Azure Stack ребра.

| Описание | Значение |
|---|---|
|Нет. файлов на каждом устройстве |100 млн |
|Нет. общих папок в каждом контейнере |1 |
|Максимальное число. общих конечных точек и конечных точек RESTFUL на устройство| 24 |
|Максимальное число. многоуровневые учетные записи хранения на устройство| 24|
|Максимальный размер файла, записанного в общую папку| 5 ТБ |
|Максимальное число групп ресурсов на устройство| 800 |

## <a name="azure-storage-limits"></a>Ограничения службы хранилища Azure

[!INCLUDE [azure-stack-edge-gateway-storage-limits](../../includes/azure-stack-edge-gateway-storage-limits.md)]

## <a name="data-upload-caveats"></a>Предупреждения, связанные с передачей данных

[!INCLUDE [azure-stack-edge-gateway-storage-data-upload-caveats](../../includes/azure-stack-edge-gateway-storage-data-upload-caveats.md)]

## <a name="azure-storage-account-size-and-object-size-limits"></a>Ограничения размера для объектов и учетной записи хранения Azure

[!INCLUDE [azure-stack-edge-gateway-storage-acct-limits](../../includes/azure-stack-edge-gateway-storage-acct-limits.md)]


## <a name="azure-object-size-limits"></a>Ограничения размера для объектов Azure

[!INCLUDE [azure-stack-edge-gateway-storage-object-limits](../../includes/azure-stack-edge-gateway-storage-object-limits.md)]

## <a name="next-steps"></a>Дальнейшие действия

- [Подготовка к развертыванию Azure Stack пограничных Pro R](azure-stack-edge-pro-r-deploy-prep.md)
