---
title: Установка драйвера GPU NVIDIA для Azure серии N для Windows
description: Как установить драйверы NVIDIA GPU для виртуальных машин серии N под управлением Windows Server или Windows в Azure
author: vikancha-MSFT
manager: jkabat
ms.service: virtual-machines
ms.subservice: vm-sizes-gpu
ms.collection: windows
ms.topic: how-to
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 09/24/2018
ms.author: vikancha
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 05487db1d0522634eda606dcb2ba22876fc341db
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102557443"
---
# <a name="install-nvidia-gpu-drivers-on-n-series-vms-running-windows"></a>Установка драйверов GPU NVIDIA на виртуальные машины серии N под управлением Windows 

Чтобы воспользоваться всеми преимуществами GPU виртуальных машин Azure серии N, необходимо установить драйверы GPU NVIDIA. [Расширение драйвера GPU NVIDIA](../extensions/hpccompute-gpu-windows.md) устанавливает необходимые драйверы CUDA или GRID NVIDIA на виртуальную машину серии N. Для установки расширения и управления им можно использовать портал Azure или такие инструменты, как Azure PowerShell и шаблоны Azure Resource Manager. Сведения о поддерживаемых операционных системах и этапах развертывания см. в [документации по расширению драйвера GPU NVIDIA](../extensions/hpccompute-gpu-windows.md).

Если вы решили установить драйверы NVIDIA GPU вручную, в этой статье содержатся Поддерживаемые операционные системы, драйверы, а также действия по установке и проверке. Сведения о ручной установке драйверов также доступны для [виртуальных машин Linux](../linux/n-series-driver-setup.md).

Основные характеристики, сведения о дисках и объеме памяти см. в статье [Графический процессор](../sizes-gpu.md?toc=/azure/virtual-machines/windows/toc.json). 

[!INCLUDE [virtual-machines-n-series-windows-support](../../../includes/virtual-machines-n-series-windows-support.md)]

## <a name="driver-installation"></a>Установка драйвера

1. Подключитесь к каждой виртуальной машине серии N с помощью удаленного рабочего стола.

2. Скачайте, извлеките и установите поддерживаемый драйвер для своей операционной системы Windows.

После установки драйвера GRID на виртуальной машине ее требуется перезагрузить. После установки драйвера CUDA перезапуск не требуется.

## <a name="verify-driver-installation"></a>Проверка установки драйверов

Обратите внимание, что панель управления NVIDIA доступна только при установке драйвера сетки. Если установлены драйверы CUDA, панель управления NVIDIA отображаться не будет.

Установку драйвера можно проверить в диспетчере устройств. В следующем примере показана успешная конфигурация карты Tesla K80 на виртуальной машине Azure серии NC.

![Свойства драйвера GPU](./media/n-series-driver-setup/GPU_driver_properties.png)

Чтобы запросить состояние устройства GPU, выполните служебную программу командной строки [nvidia smi](https://developer.nvidia.com/nvidia-system-management-interface), установленную вместе с драйвером.

1. Откройте командную строку и измените каталог на **:\Program Files\NVIDIA Corporation\NVSMI**.

2. Выполните `nvidia-smi`. Если драйвер установлен, то отобразятся выходные данные, аналогичные приведенным ниже. **GPU-Util** отобразит **0 %**, если только в этот момент графический процессор не выполняет рабочую нагрузку на виртуальной машине. Версия драйвера и сведения о GPU могут отличаться от показанных на изображении.

![Состояние устройства NVIDIA](./media/n-series-driver-setup/smi.png)  

## <a name="rdma-network-connectivity"></a>Сетевое подключение RDMA

Сетевое подключение RDMA можно включить на виртуальных машинах серии N с поддержкой RDMA, таких как NC24r, развернутых в одной группе доступности или в одной группе размещения в масштабируемом наборе виртуальных машин. Необходимо добавить расширение HpcVmDrivers для установки драйверов сетевых устройств Windows, обеспечивающих подключения RDMA. Чтобы в виртуальную машину серии N с поддержкой RDMA добавить расширение виртуальной машины, используйте командлеты [Azure PowerShell](/powershell/azure/) для Azure Resource Manager.

Чтобы установить последнюю версию расширения HpcVMDrivers 1.1 на существующую виртуальную машину myVM с поддержкой RDMA, размещенную в регионе "Западная часть США", выполните следующее.
  ```powershell
  Set-AzVMExtension -ResourceGroupName "myResourceGroup" -Location "westus" -VMName "myVM" -ExtensionName "HpcVmDrivers" -Publisher "Microsoft.HpcCompute" -Type "HpcVmDrivers" -TypeHandlerVersion "1.1"
  ```
  Дополнительные сведения см. в разделе [Обзор расширений и компонентов виртуальной машины под управлением Windows](../extensions/features-windows.md).

Сеть RDMA поддерживает трафик MPI (Message Passing Interface) для приложений, использующих [Microsoft MPI](/message-passing-interface/microsoft-mpi) или Intel MPI 5.x. 


## <a name="next-steps"></a>Дальнейшие действия

* Разработчики приложений с ускорением GPU, предназначенных для графических процессоров NVIDIA Tesla, могут также скачать и установить последнюю версию [CUDA Toolkit](https://developer.nvidia.com/cuda-downloads). Дополнительные сведения см. в [руководстве по установке CUDA](https://docs.nvidia.com/cuda/cuda-installation-guide-microsoft-windows/index.html#axzz4ZcwJvqYi).
