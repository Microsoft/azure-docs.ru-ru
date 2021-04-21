---
title: включить файл
description: включить файл
services: virtual-machines-windows
author: cynthn
ms.service: virtual-machines-windows
ms.topic: include
ms.date: 02/11/2019
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: b3e097f1c41f3047dc4e9d6cae2a05a6b19dea9d
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/07/2021
ms.locfileid: "106554250"
---
## <a name="supported-operating-systems-and-drivers"></a>Поддерживаемые операционные системы и драйверы

### <a name="nvidia-tesla-cuda-drivers"></a>Драйверы NVIDIA Tesla (CUDA)

Драйверы NVIDIA Tesla (CUDA) для виртуальных машин серий NC, NCv2, NCv3, NCasT4_v3, ND и NDv2-series (необязательно для серии NV) поддерживаются только в операционных системах, перечисленных в следующей таблице. Ссылки для скачивания драйверов актуальны на момент публикации. Последние версии драйверов можно получить на веб-сайте [NVIDIA](https://www.nvidia.com/).

> [!TIP]
> Вместо ручной установки драйвера CUDA на виртуальной машине Windows Server можно развернуть образ [виртуальной машины для обработки и анализа данных](../articles/machine-learning/data-science-virtual-machine/overview.md) Azure. Выпуски DSVM Windows Server 2016 предварительно устанавливают драйверы NVIDIA CUDA, библиотеку глубокой нейронной сети CUDA и другие средства.


| OS | Драйвер |
| -------- |------------- |
| Windows Server 2019 | [451.82](http://us.download.nvidia.com/tesla/451.82/451.82-tesla-desktop-winserver-2019-2016-international.exe) (EXE-файл) |
| Windows Server 2016 | [451.82](http://us.download.nvidia.com/tesla/451.82/451.82-tesla-desktop-winserver-2019-2016-international.exe) (EXE-файл) |

### <a name="nvidia-grid-drivers"></a>Драйверы NVIDIA GRID

Корпорация Майкрософт распространяет установщики драйверов NVIDIA GRID для виртуальных машин серии NV и NVv3, используемых в качестве виртуальных рабочих станций или виртуальных приложений. Эти драйверы GRID следует устанавливать только на виртуальные машины Azure серии NV под управлением операционных систем, перечисленных в следующей таблице. Эти драйверы содержат лицензии на ПО виртуального графического процессора GRID в Azure. Вам не нужно настраивать сервер лицензий программного обеспечения vGPU NVIDIA.

Драйверы сетки, распространяемые Azure, не работают на виртуальных машинах серии, отличных от NV, таких как NCv2, NCv3, ND и NDv2-series. Единственным исключением является серия виртуальных машин NCas_T4_V3, в которой драйверы GRID будут включать графические функции, аналогичные возможностям серии NV.

NC-Series с графическими процессорами NVIDIA K80 не поддерживают графические приложения и приложения GRID.  

Обратите внимание, что расширение NVIDIA всегда устанавливает последнюю версию драйвера. Мы предоставляем ссылки на предыдущую версию для клиентов, которые вынуждены использовать более старую версию.

Для Windows Server 2019, Windows Server 2016 1607, 1709 и Windows 10 (до сборки 20H2):
- [GRID 12.1 (461.33)](https://go.microsoft.com/fwlink/?linkid=874181) (EXE-файл)
- [GRID 12.0 (461.09)](https://download.microsoft.com/download/4/8/d/48d2d45b-bebc-44ad-9c58-e0b79a9d4ff2/461.09_grid_win10_server2016_server2019_64bit_azure_swl.exe) (EXE-файл) 

Для Windows Server 2012 R2: 
- [GRID 12.1 (461.33)](https://download.microsoft.com/download/9/9/c/99caf5c6-af9f-48b2-bcb0-af5ec64b8592/461.33_grid_server2012R2_64bit_azure_swl.exe) (EXE-файл)
- [GRID 12.0 (461.09)](https://download.microsoft.com/download/c/5/e/c5e7df99-364d-45f5-bff7-c253d59121f1/461.09_grid_server2012R2_64bit_azure_swl.exe) (EXE-файл) 


Полный список всех ссылок на все драйверы GRID предыдущих версий компании NVIDIA см. на сайте [GitHub](https://github.com/Azure/azhpc-extensions/blob/master/NvidiaGPU/resources.json)
