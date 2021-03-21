---
title: Поддерживаемые ресурсы для Azure Red Hat OpenShift 3.11
description: Узнайте, какие регионы Azure и размеры виртуальных машин поддерживаются Microsoft Azure Red Hat OpenShift.
author: jimzim
ms.author: jzim
ms.service: azure-redhat-openshift
ms.topic: conceptual
ms.date: 05/15/2019
ms.openlocfilehash: ad0837ae110b84cdff690fe76e13923a0ab60996
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100635614"
---
# <a name="azure-red-hat-openshift-resources"></a>Ресурсы Azure Red Hat OpenShift

В этом разделе перечислены регионы Azure и размеры виртуальных машин, поддерживаемые Microsoft Azure службы Red Hat OpenShift 3,11.

## <a name="azure-regions"></a>Регионы Azure

Текущий список регионов, в которых можно развернуть кластеры OpenShift Azure Red Hat, см. в разделе [Доступные продукты по регионам](https://azure.microsoft.com/global-infrastructure/services/?products=openshift&regions=all) .

## <a name="virtual-machine-sizes"></a>Размер виртуальных машин

Ниже приведены поддерживаемые размеры виртуальных машин, которые можно указать для вычислений узлов в кластере Azure Red Hat OpenShift.

> [!Important]
> Каждая виртуальная машина имеет различное количество дисков, которые можно подключить. Возможно, это не будет сразу же очищать память или размер ЦП.
> Не во всех регионах доступны виртуальные машины всех размеров. Даже если API поддерживает указанный размер, может возникнуть ошибка, если размер недоступен в указанном регионе.
> Дополнительные сведения см. [в текущем списке поддерживаемых размеров виртуальных машин на регион](https://azure.microsoft.com/global-infrastructure/services/?products=virtual-machines) .

## <a name="compute-node-sizes"></a>Размеры узлов вычислений

REST API Azure Red Hat OpenShift поддерживают следующие размеры расчетных узлов:

|Размер|vCPU|ОЗУ|
|-|-|-|
|Стандартный D4s v3|4|16 ГБ|
|Стандартный D8s v3|8|32 ГБ|
|Стандартный D16s v3|16|64 ГБ|
|Стандартный D32s v3|32|128 ГБ|
|-|-|-|
|Стандартный E4s v3|4|32 ГБ|
|Стандартный E8s v3|8|64 ГБ|
|Стандартный E16s v3|16|128 ГБ|
|Стандартный E32s v3|32|256 ГБ|
|-|-|-|
|Стандартный F8s v2|8|16 ГБ|
|Стандартный F16s v2|16|32 ГБ|
|Стандартный F32s v2|32|64 ГБ|

## <a name="master-node-sizes"></a>Размеры главного узла

В REST API Azure Red Hat OpenShift поддерживаются следующие размеры узлов главной и основной инфраструктуры:

|Размер|vCPU|ОЗУ|
|-|-|-|
|Стандартный D4s v3|4|16 ГБ|
|Стандартный D8s v3|8|32 ГБ|
|Стандартный D16s v3|16|64 ГБ|
|Стандартный D32s v3|32|128 ГБ|

## <a name="next-steps"></a>Дальнейшие действия

Воспользуйтесь учебником [Создание кластера Azure Red Hat OpenShift](tutorial-create-cluster.md) .
