---
title: Использование графического процессора ускорения вычислений или ВПУ на Azure Stack пограничных устройствах для развертываний Kubernetes | Документация Майкрософт
description: Описывает Azure Stack Azure Stack, как использовать процессоры ускорения вычислений GPU или ВПУ на графическом процессоре Azure Stack.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 02/22/2021
ms.author: alkohli
ms.openlocfilehash: 3eb648af60a7be62d08f6b172347778d2358643c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102440128"
---
# <a name="use-compute-acceleration-on-azure-stack-edge-pro-gpu-for-kubernetes-deployment"></a>Использование ускорения вычислений для развертывания Kubernetes в графическом процессоре Pro Azure Stack

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

В этой статье описывается, как использовать ускорение вычислений на Azure Stack пограничных устройствах при использовании развертываний Kubernetes. Эта статья относится к Azure Stackным графическим процессорам Pro, Azure Stackм пограничных Pro R и Azure Stack пограничных устройств R.


## <a name="about-compute-acceleration"></a>О ускорении вычислений 

Центральный процессор (CPU) — это общее назначение по умолчанию для большинства процессов, выполняемых на компьютере. Часто специализированное оборудование компьютера используется для более эффективного выполнения некоторых функций, чем запуск программного обеспечения в ЦП. Например, графический процессор (GPU) можно использовать для ускорения обработки данных пикселей.  

Ускорение вычислений — это термин, применяемый специально для Azure Stack пограничных устройств, где для аппаратного ускорения используются графические единицы обработки (GPU), вычислительный блок (ВПУ) или поле с программируемым массивом шлюзов (FPGA). Большинство рабочих нагрузок, развернутых на Azure Stack пограничном устройстве, относятся к критическим времени, нескольким потокам камеры и (или) высокой частоте кадров, для которых требуется определенное аппаратное ускорение.

В этой статье будет обсуждаться ускорение вычислений с использованием GPU или ВПУ для следующих устройств:

- **Графический процессор Azure Stack с пограничными краями** . Эти устройства могут иметь 1 или 2 GPU NVIDIA T4 тензорные Core. Дополнительные сведения см. в разделе [NVIDIA T4](https://www.nvidia.com/en-us/data-center/tesla-t4/).
- **Azure Stack пограничной Pro R** — эти устройства имеют один GPU NVIDIA T4 тензорные Core. Дополнительные сведения см. в разделе [NVIDIA T4](https://www.nvidia.com/en-us/data-center/tesla-t4/).
- **Azure Stack пограничной мини-R** . Эти устройства имеют 1 Intel Мовидиус множество X ВПУ. Дополнительные сведения см. в разделе [Intel Мовидиус множество X ВПУ](https://www.movidius.com/MyriadX).


## <a name="use-gpu-for-kubernetes-deployment"></a>Использование GPU для развертывания Kubernetes

Следующий пример `yaml` можно использовать для графического процессора Azure Stack ребра Pro или устройства с Azure Stack ребра R с графическим процессором.

<!--In a production scenario, Pods are not used directly and these are wrapped around higher level constructs like Deployment, ReplicaSet which maintain the desired state in case of pod restarts, failures.-->

```yml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
spec:
  containers:
    - name: cuda-container
      image: nvidia/cuda:9.0-devel
      resources:
        limits:
          nvidia.com/gpu: 1 # requesting 1 GPU
    - name: digits-container
      image: nvidia/digits:6.0
      resources:
        limits:
          nvidia.com/gpu: 1 # requesting 1 GPU
```


## <a name="use-vpu-for-kubernetes-deployment"></a>Использование ВПУ для развертывания Kubernetes

Следующий пример `yaml` можно использовать для устройства мини R с Azure Stack ребра с ВПУ.

```yml
apiVersion: batch/v1
kind: Job
metadata:
 name: intelvpu-demo-job
 labels:
   jobgroup: intelvpu-demo
spec:
 template:
   metadata:
     labels:
       jobgroup: intelvpu-demo
   spec:
     restartPolicy: Never
     containers:
       -
         name: intelvpu-demo-job-1
         image: ubuntu-demo-openvino:devel
         imagePullPolicy: IfNotPresent
         command: [ "/do_classification.sh" ]
         resources:
           limits:
             vpu.intel.com/hddl: 1
```


## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как [использовать kubectl для запуска приложения Kubernetes с отслеживанием состояния с помощью персистентволуме на устройстве с Azure Stack ребра Pro GPU](azure-stack-edge-gpu-deploy-stateful-application-static-provision-kubernetes.md).
