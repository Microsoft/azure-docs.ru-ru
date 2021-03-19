---
title: Устранение неполадок при репликации между регионами для Azure NetApp Files | Документация Майкрософт
description: Описывает сообщения об ошибках и способы их устранения, которые могут помочь при устранении неполадок с репликацией в разных регионах для Azure NetApp Files.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 03/10/2021
ms.author: b-juche
ms.openlocfilehash: d3d944646689e9e6189b0343e8bf67c8fb0abcbd
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104590931"
---
# <a name="troubleshoot-cross-region-replication"></a>Устранение неполадок с репликацией между регионами

В этой статье описываются сообщения об ошибках и способы их устранения, которые могут помочь при устранении неполадок с репликацией в разных регионах для Azure NetApp Files. 

## <a name="errors-creating-replication"></a>Ошибки при создании репликации  

|     Сообщение об ошибке    |     Решение    |
|-|-|
|     `Volume {0} cannot   be used as source because it is already in replication`    |     Невозможно создать репликацию с исходным томом, который уже находится в отношении репликации данных.    |
|     `Peered region   '{0}' is not accepted`    |     Вы пытаетесь создать репликацию между неравноправными регионами.    |
|     `RemoteVolumeResource   '{0}' of wrong type '{1}'`    |     Проверьте, что идентификатор удаленного ресурса является ИДЕНТИФИКАТОРом ресурса тома.    |

## <a name="errors-authorizing-volume"></a>Ошибки авторизации тома  

|     Сообщение об ошибке    |     Решение    |
|-|-|
|     `Missing value   for 'AuthorizeSourceReplication'`    |     В   `RemoteResourceID` пользовательском интерфейсе или запросе API отсутствует или является недопустимым (исправьте сообщение об ошибке).    |
|     `Missing value   for 'RemoteVolumeResourceId'`    |     В   `RemoteResourceID` пользовательском интерфейсе или запросе API отсутствует или является недопустимым.    |
|     `Data   Protection volume not found for RemoteVolumeResourceId: {remoteResourceId}`    |     Проверьте   `RemoteResourceID` правильность или наличие для пользователя.    |
|     `Remote volume   '{0}' is not configured for replication`    |     Целевой том не является томом защиты данных.    |
|     `Remote volume   '{0}' does not have source volume '{1}' as RemoteVolumeResourceId`    |     На томе защиты данных отсутствует этот исходный том в ИДЕНТИФИКАТОРе удаленного ресурса (указан недопустимый идентификатор источника).    |
|     `The   destination volume replication creation failed (message: {0})`    |     Эта ошибка указывает на ошибку сервера. Обратитесь в службу поддержки.    |

## <a name="errors-deleting-replication"></a>Ошибки при удалении репликации

|     Сообщение об ошибке    |     Решение    |
|-|-|
|     `Replication   cannot be deleted, mirror state needs to be in status: Broken before deleting`    |     Проверьте, что какая-либо репликация была разорвана или не была инициализирована и неактивна (сбой инициализации).    |
|     `Cannot delete   source replication`    |     Удаление репликации с исходной стороны не допускается. Убедитесь, что вы удаляете репликацию с конечной стороны.    |

## <a name="errors-deleting-volume"></a>Ошибки при удалении тома

|     Сообщение об ошибке    |     Решение    |
|-|-|
| `Volume is a member of an active volume replication relationship`  |  Удалите репликацию перед удалением тома. См. раздел [Удаление репликации](cross-region-replication-delete.md). Для этой операции необходимо разорвать пиринг перед удалением репликации для тома. |
| `Volume with replication cannot be deleted`  |  Удалите репликацию перед удалением тома. См. раздел [Удаление репликации](cross-region-replication-delete.md). Для этой операции необходимо разорвать пиринг перед удалением репликации для тома. 

## <a name="errors-resyncing-volume"></a>Ошибки при повторной синхронизации тома

|     Сообщение об ошибке    |     Решение    |
|-|-|
|     `Volume Replication is in invalid status: (Mirrored|Uninitialized) for operation: 'ResyncReplication'`     |     Проверьте, что репликация тома находится в состоянии "нарушено".    |

## <a name="errors-deleting-snapshot"></a>Ошибки при удалении моментального снимка 

|     Сообщение об ошибке    |     Решение    |
|-|-|
|     `Snapshot   cannot be deleted, parent volume is a Data Protection volume with a   replication object`    |     Убедитесь, что репликация тома была нарушена, если вы хотите удалить этот моментальный снимок.    |
|     `Cannot delete   volume replication generated snapshot`    |     Удаление базовых моментальных снимков репликации запрещено.    |

## <a name="errors-resizing-volumes"></a>Ошибки при изменении размера томов

|     Сообщение об ошибке    |     Решение    |
|-|-|
|   Попытка изменить размер исходного тома завершилась ошибкой `"PoolSizeTooSmall","message":"Pool size too small for total volume size."`  |  Убедитесь в наличии достаточного запаса в пулах ресурсов для исходного и целевого томов репликации между регионами. При изменении размера исходного тома целевой том автоматически изменяется. Но если пул емкости, в котором размещен целевой том, не имеет достаточного запаса, изменение размера исходного и целевого томов завершится ошибкой. Дополнительные сведения см. [в разделе изменение размера тома назначения репликации в разных регионах](azure-netapp-files-resize-capacity-pools-or-volumes.md#resize-a-cross-region-replication-destination-volume) .   |

## <a name="next-steps"></a>Дальнейшие действия  

* [Репликация между регионами](cross-region-replication-introduction.md)
* [Требования и рекомендации по использованию репликации между регионами](cross-region-replication-requirements-considerations.md)
* [Создание репликации тома](cross-region-replication-create-peering.md)
* [Отображение состояния работоспособности отношения репликации](cross-region-replication-display-health-status.md)
* [Управление аварийным восстановлением](cross-region-replication-manage-disaster-recovery.md)
* [Изменение размера тома назначения репликации между регионами](azure-netapp-files-resize-capacity-pools-or-volumes.md#resize-a-cross-region-replication-destination-volume)
* [Устранение неполадок с репликацией между регионами](troubleshoot-cross-region-replication.md)
