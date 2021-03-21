---
title: Подключение файлового ресурса Azure NFS к службе файлов Azure
description: Узнайте, как подключить общую сетевую папку файловой системы.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 12/04/2020
ms.author: rogarana
ms.subservice: files
ms.custom: references_regions
ms.openlocfilehash: 8a993d9c1de35132198de5e3becc4f16d6a2a437
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96621303"
---
# <a name="how-to-mount-an-nfs-file-share"></a>Как подключить файловый ресурс NFS

[Файлы Azure](storage-files-introduction.md) — это простая в использовании облачная файловая система от Майкрософт. Файловые ресурсы Azure можно подключить в дистрибутивах Linux с помощью протокола SMB или протокола сетевой файловой системы (NFS). Эта статья посвящена подключению к NFS. Дополнительные сведения о подключении с помощью SMB см. в статье [Использование службы файлов Azure с Linux](storage-how-to-use-files-linux.md). Дополнительные сведения о каждом из доступных протоколов см. в статье [протоколы файловых ресурсов Azure](storage-files-compare-protocols.md).

## <a name="limitations"></a>Ограничения

[!INCLUDE [files-nfs-limitations](../../../includes/files-nfs-limitations.md)]

### <a name="regional-availability"></a>Доступность по регионам

[!INCLUDE [files-nfs-regional-availability](../../../includes/files-nfs-regional-availability.md)]

## <a name="prerequisites"></a>Предварительные требования

- [Создайте общую папку NFS](storage-files-how-to-create-nfs-shares.md).

    > [!IMPORTANT]
    > Доступ к общим папкам NFS можно получить только из доверенных сетей. Подключения к общему ресурсу NFS должны исходить из одного из следующих источников:

- Используйте одно из следующих сетевых решений.
    - Либо [Создайте закрытую конечную точку](storage-files-networking-endpoints.md#create-a-private-endpoint) (рекомендуется), либо [Ограничьте доступ к общедоступной конечной точке](storage-files-networking-endpoints.md#restrict-public-endpoint-access).
    - [Настройте VPN-подключение типа "точка — сеть" (P2S) в Linux для использования с файлами Azure](storage-files-configure-p2s-vpn-linux.md).
    - [Настройте VPN типа "сеть — сеть" для использования с файлами Azure](storage-files-configure-s2s-vpn.md).
    - Настройте [ExpressRoute](../../expressroute/expressroute-introduction.md).

## <a name="disable-secure-transfer"></a>Отключить безопасную пересылку

1. Войдите в портал Azure и получите доступ к учетной записи хранения, содержащей созданный общий ресурс NFS.
1. Щелкните **Конфигурация**.
1. Для **обязательного безопасного перемещения** выберите **отключено** .
1. Щелкните **Сохранить**.

    :::image type="content" source="media/storage-files-how-to-mount-nfs-shares/storage-account-disable-secure-transfer.png" alt-text="Снимок экрана с экраном настройки учетной записи хранения с отключенной безопасной защитой.":::

## <a name="mount-an-nfs-share"></a>Подключение общего ресурса NFS

1. После создания общей папки выберите общую папку и выберите **подключить из Linux**.
1. Введите путь подключения, который вы хотите использовать, а затем скопируйте сценарий.
1. Подключитесь к клиенту и используйте указанный сценарий подключения.

    :::image type="content" source="media/storage-files-how-to-create-mount-nfs-shares/mount-nfs-file-share-script.png" alt-text="Снимок экрана: колонка &quot;подключение к общей папке&quot;":::

Вы подключили общий ресурс NFS.

### <a name="validate-connectivity"></a>Проверка подключения

В случае сбоя подключения частная конечная точка может быть настроена неправильно или недоступна. Дополнительные сведения о подтверждении подключения см. в разделе [Проверка подключения](storage-files-networking-endpoints.md#verify-connectivity) конечных точек сети.

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о службе файлов Azure см. в статье [Планирование развертывания файлов Azure](storage-files-planning.md).
- Если возникнут проблемы, см. раздел [Устранение неполадок файловых ресурсов NFS Azure](storage-troubleshooting-files-nfs.md).