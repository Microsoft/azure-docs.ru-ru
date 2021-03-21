---
title: Ограничения для мини-R Azure Stack пограничных устройств | Документация Майкрософт
description: Описывает ограничения системы и рекомендованные размеры для Azure Stack пограничных Мини-R.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: conceptual
ms.date: 10/13/2020
ms.author: alkohli
ms.openlocfilehash: 6f01d62d1d11f5ff90661482ffd2db112657eee5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96467849"
---
# <a name="azure-stack-edge-mini-r-limits"></a>Ограничения для мини-R Azure Stack ребра


Эти ограничения следует учитывать при развертывании и эксплуатации решения Mini R на Azure Stack пограничных устройств.

## <a name="azure-stack-edge-service-limits"></a>Ограничения службы Azure Stack ребра

[!INCLUDE [azure-stack-edge-gateway-service-limits](../../includes/azure-stack-edge-gateway-service-limits.md)]

## <a name="azure-stack-edge-mini-r-device-limits"></a>Ограничения для устройств Mini R на Azure Stack пограничных устройствах

В следующей таблице описаны ограничения для устройства Mini R Azure Stack пограничных устройств.

| Описание | Ограничение|
|---|---:|
|Нет. файлов на каждом устройстве | 100 млн <!--check with devs-->|
|Нет. общих папок в каждом контейнере | 1|
|Максимальное число. общих конечных точек и конечных точек RESTFUL на устройство| 24 |
|Максимальное число. многоуровневые учетные записи хранения на устройство| 24|
|Максимальный размер файла, записанного в общую папку| 500 ГБ|
|Максимальное число групп ресурсов на устройство| 800|

## <a name="azure-storage-limits"></a>Ограничения службы хранилища Azure

[!INCLUDE [azure-stack-edge-gateway-storage-limits](../../includes/azure-stack-edge-gateway-storage-limits.md)]

## <a name="data-upload-caveats"></a>Предупреждения, связанные с передачей данных

[!INCLUDE [azure-stack-edge-gateway-storage-data-upload-caveats](../../includes/azure-stack-edge-gateway-storage-data-upload-caveats.md)]

## <a name="azure-storage-account-size-and-object-size-limits"></a>Ограничения размера для объектов и учетной записи хранения Azure

[!INCLUDE [azure-stack-edge-gateway-storage-acct-limits](../../includes/azure-stack-edge-gateway-storage-acct-limits.md)]

## <a name="azure-object-size-limits"></a>Ограничения размера для объектов Azure

[!INCLUDE [azure-stack-edge-gateway-storage-object-limits](../../includes/azure-stack-edge-gateway-storage-object-limits.md)]

## <a name="next-steps"></a>Дальнейшие действия

- [Подготовка к развертыванию Azure Stack пограничных устройств](azure-stack-edge-gpu-deploy-prep.md)
