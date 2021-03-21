---
title: Доступные размеры для облачных служб Azure (Расширенная поддержка)
description: Доступные размеры для развертываний облачных служб Azure (Расширенная поддержка)
ms.topic: article
ms.service: cloud-services-extended-support
author: gachandw
ms.author: gachandw
ms.reviewer: mimckitt
ms.date: 10/13/2020
ms.custom: ''
ms.openlocfilehash: cd8011782d134031393731a29594d44aba41b2ef
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101713309"
---
# <a name="available-sizes-for-azure-cloud-services-extended-support"></a>Доступные размеры для облачных служб Azure (Расширенная поддержка)

В этой статье описываются доступные размеры виртуальных машин для экземпляров облачных служб (Расширенная поддержка).   

| Семейство SKU |  ACU/Core | 
|---|---|
| [A5–7](../virtual-machines/sizes-previous-gen.md?bc=%2fazure%2fvirtual-machines%2flinux%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json#a-series)| 100 |
|[A8-A11](../virtual-machines/sizes-previous-gen.md?bc=%2fazure%2fvirtual-machines%2flinux%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json#a-series---compute-intensive-instances) | 225* |
|[Av2](../virtual-machines/av2-series.md) | 100 | 
|[D](../virtual-machines/sizes-previous-gen.md?bc=%2fazure%2fvirtual-machines%2flinux%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json#d-series) | 160 | 
|[Dv2](../virtual-machines/dv2-dsv2-series.md) | 160–190* |
|[Dv3](../virtual-machines/dv3-dsv3-series.md) | 160–190* |
|[Ev3](../virtual-machines/ev3-esv3-series.md) | 160–190*
|[G](../virtual-machines/sizes-previous-gen.md?bc=%2fazure%2fvirtual-machines%2flinux%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json#g-series) | 180-240 * |
|[H](../virtual-machines/h-series.md) | 290–300* | 

>[!NOTE]
> Единицы ACU, помеченные * , основаны на технологии Intel® Turbo для увеличения частоты ЦП и значительного повышения производительности. Степень увеличения производительности может различаться в зависимости от размера виртуальной машины, рабочей нагрузки и других рабочих нагрузок, выполняющихся на том же узле.

## <a name="configure-sizes-for-cloud-services-extended-support"></a>Настройка размеров для облачных служб (Расширенная поддержка)

Можно указать размер виртуальной машины экземпляра роли как часть модели службы в файле определения службы. Размер роли определяет количество ядер ЦП, объем памяти и размер локальной файловой системы.

Например, установка размера экземпляра веб-роли `Standard_D2` : 

```xml
<WorkerRole name="Worker1" vmsize="Standard_D2"> 
</WorkerRole> 
```

## <a name="change-the-size-of-an-existing-role"></a>Изменение размера существующей роли

Чтобы изменить размер существующей роли, измените размер виртуальной машины в файле определения службы (CSDEF), переупакуйте облачную службу и повторно разверните ее. 

## <a name="get-a-list-of-available-sizes"></a>Получение списка доступных размеров 

Чтобы получить список доступных размеров, ознакомьтесь со [списком SKU ресурсов](/rest/api/compute/resourceskus/list) и примените следующие фильтры:


`ResourceType = virtualMachines ` <br>
`VMDeploymentTypes = PaaS `


## <a name="next-steps"></a>Дальнейшие действия 
- Ознакомьтесь с [предварительными требованиями для развертывания](deploy-prerequisite.md) облачных служб (Расширенная поддержка).
- Ознакомьтесь с [часто задаваемыми вопросами об Облачных службах (расширенная поддержка)](faq.md).
- Разверните Облачную службу (расширенная поддержка) с помощью [портала Azure](deploy-portal.md), [PowerShell](deploy-powershell.md), [шаблона](deploy-template.md) или [Visual Studio](deploy-visual-studio.md).