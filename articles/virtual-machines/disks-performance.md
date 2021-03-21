---
title: Производительность диска и виртуальной машины
description: Узнайте больше о том, как виртуальные машины и их подключенные диски работают в сочетании с производительностью.
author: albecker1
ms.author: albecker
ms.date: 10/12/2020
ms.topic: conceptual
ms.service: virtual-machines
ms.subservice: disks
ms.openlocfilehash: d3a89e7733cc033792056b8de5232232b8327025
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100580403"
---
# <a name="virtual-machine-and-disk-performance"></a>Производительность диска и виртуальной машины
[!INCLUDE [VM and Disk Performance](../../includes/virtual-machine-disk-performance.md)]

## <a name="virtual-machine-uncached-vs-cached-limits"></a>Кэшированные ограничения виртуальной машины в сравнении с кэшем
Для виртуальных машин, включенных как для хранилища класса Premium, так и для кэширования хранилища класса Premium, существует два разных ограничения пропускной способности хранилища. Рассмотрим Standard_D8s_v3 виртуальную машину в качестве примера. Ниже приведена документация по [серии Dsv3](dv3-dsv3-series.md) и Standard_D8s_v3:

[!INCLUDE [VM and Disk Performance](../../includes/virtual-machine-disk-performance-2.md)]

