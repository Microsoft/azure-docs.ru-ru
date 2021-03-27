---
title: Требования к системе для Microsoft Azure Data Box | Документация Майкрософт
description: Сведения о важных системных требованиях для Azure Data Box и клиентов, подключающихся к Data Box.
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: article
ms.date: 03/25/2021
ms.author: alkohli
ms.openlocfilehash: 7f999dbf7a4e0262e36181a98560931d32d3296b
ms.sourcegitcommit: c94e282a08fcaa36c4e498771b6004f0bfe8fb70
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105612506"
---
# <a name="azure-data-box-system-requirements"></a>Требования к системе для Azure Data Box

В этой статье описываются важные требования к системе для Data Box Microsoft Azure и для клиентов, подключающихся к Data Box. Перед развертыванием Data Box рекомендуется внимательно просмотреть сведения, а затем обратиться к ней, когда потребуется выполнить развертывание и эксплуатацию.

В приведенных ниже разделах описаны требования к системе.

* **Требования к программному обеспечению:** Для узлов, подключающихся к Data Box, описываются поддерживаемые операционные системы, протоколы обмена файлами, учетные записи хранения, типы хранилищ и браузеры для локального веб-интерфейса.
* **Требования к сети:** В Data Box описаны требования к сетевым подключениям и портам для оптимальной работы Data Box.


## <a name="software-requirements"></a>Требования к программному обеспечению

Требования к программному обеспечению включают в себя Поддерживаемые операционные системы, протоколы обмена файлами, учетные записи хранения, типы хранилищ и браузеры для локального веб-интерфейса.

### <a name="supported-operating-systems-for-clients"></a>Поддерживаемые версии операционных систем для клиентов

[!INCLUDE [data-box-supported-os-clients](../../includes/data-box-supported-os-clients.md)]


### <a name="supported-file-transfer-protocols-for-clients"></a>Поддерживаемые протоколы обмена файлами для клиентов

[!INCLUDE [data-box-supported-file-systems-clients](../../includes/data-box-supported-file-systems-clients.md)]

> [!IMPORTANT] 
> Подключение к Data Boxным общим ресурсам не поддерживается через ОСТАВШУЮся для экспорта заказов.

### <a name="supported-storage-accounts"></a>Учетные записи хранилища BLOB-объектов

[!INCLUDE [data-box-supported-storage-accounts](../../includes/data-box-supported-storage-accounts.md)]

### <a name="supported-storage-types"></a>Поддерживаемые типы хранилища

[!INCLUDE [data-box-supported-storage-types](../../includes/data-box-supported-storage-types.md)]

### <a name="supported-web-browsers"></a>Поддерживаемые веб-браузеры

[!INCLUDE [data-box-supported-web-browsers](../../includes/data-box-supported-web-browsers.md)]

## <a name="networking-requirements"></a>Требования к сети

Центр обработки данных должен иметь высокоскоростную сеть. Настоятельно рекомендуем использовать по крайней мере 1 10-гигабитное подключение. Если 10-гигабитное подключение недоступно, можно использовать канал данных 1-GbE для копирования данных, но это повлияет на скорость копирования.

### <a name="port-requirements"></a>Требования к порту

В следующей таблице перечислены порты, которые необходимо открыть в брандмауэре, чтобы разрешить трафик SMB или NFS. В этой таблице *в* (*входящая*) указывается направление, от которого входящий клиент запрашивает доступ к устройству. *Out* (или *Outbound*) — это направление, в котором устройство Data Box отправляет данные извне, помимо развертывания. Например, данные могут быть исходящими в Интернете.

[!INCLUDE [data-box-port-requirements](../../includes/data-box-port-requirements.md)]


## <a name="next-steps"></a>Дальнейшие действия

* [Развертывание Azure Data Box](data-box-deploy-ordered.md)
