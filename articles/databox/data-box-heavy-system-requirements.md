---
title: Microsoft Azure Data Box Heavy требования к системе | Документация Майкрософт
description: Сведения о требованиях к программному обеспечению и сети для Azure Data Box Heavy
services: databox
author: alkohli
ms.service: databox
ms.subservice: heavy
ms.topic: article
ms.date: 07/03/2019
ms.author: alkohli
ms.openlocfilehash: 6097dba9193fce4c8158137b488080107881994a
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "84707758"
---
# <a name="azure-data-box-heavy-system-requirements"></a>Azure Data Box Heavy требования к системе

В этой статье описываются важные требования к системе для устройства Azure Data Box Heavy и для клиентов, подключающихся к устройству. Перед развертыванием Data Box Heavy рекомендуется внимательно просмотреть сведения, а затем при развертывании и последующей операции обращаться к ней по мере необходимости.

В приведенных ниже разделах описаны требования к системе.

* **Требования к программному обеспечению для узлов, подключающихся к Data Box Heavy** — описание поддерживаемых платформ, браузеров для локального пользовательского веб-интерфейса, клиентов SMB и дополнительных требований для узлов, которые могут подключаться к Data Box.
* **Требования к сети для Data Box Heavy** . предоставляет сведения о требованиях к сети для оптимальной работы устройства Data Box Heavy.

## <a name="software-requirements"></a>Требования к программному обеспечению

Требования к программному обеспечению включают в себя информацию о поддерживаемых операционных системах, веб-браузерах для локального пользовательского веб-интерфейса и клиентах SMB.

### <a name="supported-operating-systems-for-clients"></a>Поддерживаемые версии операционных систем для клиентов

[!INCLUDE [data-box-supported-os-clients](../../includes/data-box-supported-os-clients.md)]

### <a name="supported-file-systems-for-linux-clients"></a>Поддерживаемые файловые системы для клиентов Linux

[!INCLUDE [data-box-supported-file-systems-clients](../../includes/data-box-supported-file-systems-clients.md)]

### <a name="supported-storage-accounts"></a>Учетные записи хранилища BLOB-объектов

[!INCLUDE [data-box-supported-storage-accounts](../../includes/data-box-supported-storage-accounts.md)]

### <a name="supported-storage-types"></a>Поддерживаемые типы хранилища

[!INCLUDE [data-box-supported-storage-types](../../includes/data-box-supported-storage-types.md)]

### <a name="supported-web-browsers"></a>Поддерживаемые веб-браузеры

[!INCLUDE [data-box-supported-web-browsers](../../includes/data-box-supported-web-browsers.md)]

## <a name="networking-requirements"></a>Требования к сети

Центр обработки данных должен иметь высокоскоростную сеть. Для ускорения копирования 2 40-гигабитные подключения можно использовать параллельно (по одному на узел). Если у вас нет 40-GbE, рекомендуется иметь по крайней мере 2 10-GbE подключений (по одному на узел).

### <a name="port-requirements"></a>Требования к порту

В следующей таблице перечислены порты, которые необходимо открыть в брандмауэре, чтобы разрешить трафик SMB или NFS. В этой таблице значение *в* или *входящий* относится к направлению, из которого клиент запрашивает доступ к вашему устройству. *Out* или *Outbound* — это направление, в котором устройство Data Box Heavy отправляет данные извне, помимо развертывания: например, исходящий трафик в Интернете.

[!INCLUDE [data-box-port-requirements](../../includes/data-box-port-requirements.md)]

## <a name="next-steps"></a>Дальнейшие действия

* [Развертывание Azure Data Box](data-box-deploy-ordered.md)
