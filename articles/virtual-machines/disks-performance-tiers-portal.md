---
title: Изменение производительности управляемых дисков Azure с помощью портал Azure
description: Узнайте, как изменить уровни производительности для новых и существующих управляемых дисков с помощью портал Azure.
author: roygara
ms.service: virtual-machines
ms.topic: how-to
ms.date: 01/05/2021
ms.author: rogarana
ms.subservice: disks
ms.custom: references_regions
ms.openlocfilehash: 625fb1e3dd0b433da6b60f995aa6b380c23ec9ce
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97901040"
---
# <a name="change-your-performance-tier-using-the-azure-portal"></a>Измените уровень производительности с помощью портал Azure

[!INCLUDE [virtual-machines-disks-performance-tiers-intro](../../includes/virtual-machines-disks-performance-tiers-intro.md)]

## <a name="restrictions"></a>Ограничения

[!INCLUDE [virtual-machines-disks-performance-tiers-restrictions](../../includes/virtual-machines-disks-performance-tiers-restrictions.md)]

## <a name="getting-started"></a>Начало работы

### <a name="new-disks"></a>Новые диски

Ниже показано, как изменить уровень производительности диска при первом создании диска.

1. Войдите на [портал Azure](https://portal.azure.com/).
1. Перейдите к виртуальной машине, для которой вы хотите создать новый диск.
1. При выборе нового диска сначала нужно выбрать необходимый размер диска.
1. Выбрав размер, выберите другой уровень производительности, чтобы изменить его производительность.
1. Нажмите кнопку **ОК** , чтобы создать диск.

:::image type="content" source="media/disks-performance-tiers-portal/new-disk-change-performance-tier.png" alt-text="Снимок экрана: колонка создания диска, выделена дискета, и выделена раскрывающийся список уровня производительности." lightbox="media/disks-performance-tiers-portal/performance-tier-settings.png":::


## <a name="existing-disks"></a>Существующие диски

Ниже описаны действия по изменению уровня производительности существующего диска.

1. Войдите на [портал Azure](https://portal.azure.com/).
1. Перейдите к виртуальной машине, содержащей диск, который вы хотите изменить.
1. Либо Освободите виртуальную машину, либо отсоедините диск.
1. Выбор диска
1. Выберите **размер и производительность**.
1. В раскрывающемся списке **уровень производительности** выберите уровень, отличный от текущего уровня производительности диска.
1. Выберите **Изменить размер**.

:::image type="content" source="media/disks-performance-tiers-portal/change-tier-existing-disk.png" alt-text="Снимок экрана: колонка &quot;размер + производительность&quot;, уровень производительности выделена." lightbox="media/disks-performance-tiers-portal/performance-tier-settings.png":::

## <a name="next-steps"></a>Дальнейшие действия

Если необходимо изменить размер диска, чтобы воспользоваться преимуществами более высоких уровней производительности, см. следующие статьи:

- [Расширение виртуальных жестких дисков на виртуальной машине Linux с помощью Azure CLI](linux/expand-disks.md)
- [Расширение управляемого диска, подключенного к виртуальной машине Windows](windows/expand-os-disk.md)
