---
title: Создание и передача Флаткар контейнера Linux VHD для использования в Azure
description: Узнайте, как создать и передать виртуальный жесткий диск, содержащий Флаткар контейнер операционной системы Linux.
author: marga-kinvolk
ms.author: danis
ms.service: virtual-machines
ms.collection: linux
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 07/16/2020
ms.reviewer: cynthn
ms.openlocfilehash: 5d8be9493b7a312270301e3520f301f797fe2167
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102565297"
---
# <a name="using-a-prebuilt-flatcar-image-for-azure"></a>Использование предварительно созданного образа Флаткар для Azure

Вы можете скачать предварительно созданные образы виртуальных жестких дисков Azure для контейнера Флаткар в Linux для каждого из поддерживаемых каналов Флаткар:

- [стабильная](https://stable.release.flatcar-linux.net/amd64-usr/current/flatcar_production_azure_image.vhd.bz2)
- [бета-версия](https://beta.release.flatcar-linux.net/amd64-usr/current/flatcar_production_azure_image.vhd.bz2)
- [буквы](https://alpha.release.flatcar-linux.net/amd64-usr/current/flatcar_production_azure_image.vhd.bz2)
- [угловой](https://edge.release.flatcar-linux.net/amd64-usr/current/flatcar_production_azure_image.vhd.bz2)

Этот образ уже полностью настроен и оптимизирован для работы в Azure. Его необходимо распаковать.

## <a name="building-your-own-flatcar-based-virtual-machine-for-azure"></a>Создание виртуальной машины на основе Флаткар для Azure

Кроме того, можно создать собственный Флаткар контейнер с образом Linux.

На любом компьютере под управлением Linux выполните инструкции, описанные в разделе [Флаткар Container for Linux Developer SDK](https://docs.flatcar-linux.org/os/sdk-modifying-flatcar/). При выполнении `image_to_vm.sh` скрипта убедитесь, что вы прошли `--format=azure` Создание виртуального жесткого диска Azure.

## <a name="next-steps"></a>Дальнейшие действия

Создав VHD-файл, вы можете использовать полученный файл для создания новых виртуальных машин в Azure. Если вы отправляете VHD-файл в Azure в первый раз, см. статью [Создание виртуальной машины Linux на основе пользовательского диска](upload-vhd.md#option-1-upload-a-vhd).
