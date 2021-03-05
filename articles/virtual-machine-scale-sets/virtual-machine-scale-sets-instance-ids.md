---
title: Общие сведения об идентификаторах экземпляров для виртуальных машин масштабируемого набора Azure
description: Общие сведения о идентификаторах экземпляров для масштабируемых наборов виртуальных машин Azure и различных способах их поверхности.
author: mimckitt
ms.author: mimckitt
ms.topic: conceptual
ms.service: virtual-machine-scale-sets
ms.subservice: management
ms.date: 02/22/2018
ms.reviewer: jushiman
ms.custom: mimckitt
ms.openlocfilehash: a62c9bbde0726c8dec8fba1f69e221bd4e4b63bc
ms.sourcegitcommit: f7eda3db606407f94c6dc6c3316e0651ee5ca37c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102209855"
---
# <a name="understand-instance-ids-for-azure-vm-scale-set-vms"></a>Общие сведения об идентификаторах экземпляров для виртуальных машин масштабируемого набора Azure
В этой статье описаны идентификаторы экземпляров для масштабируемых наборов и связанные с ними возможности.

## <a name="scale-set-instance-ids"></a>Идентификаторы экземпляров масштабируемого набора

Каждой виртуальной машине в масштабируемом наборе присваивается определяющий ее идентификатор экземпляра. Он используется в программных интерфейсах (API) масштабируемого набора для выполнения операций на определенной виртуальной машине в масштабируемом наборе. Например, при использовании API пересоздания образа можно указать определенный идентификатор экземпляра:

REST API: `POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}/virtualmachines/{instanceId}/reimage?api-version={apiVersion}` (дополнительные сведения см. в [документации по REST API](/rest/api/compute/virtualmachinescalesetvms/reimage)).

PowerShell: `Set-AzVmssVM -ResourceGroupName {resourceGroupName} -VMScaleSetName {vmScaleSetName} -InstanceId {instanceId} -Reimage` (дополнительные сведения см. в [документации по PowerShell](/powershell/module/az.compute/set-azvmssvm)).

CLI: `az vmss reimage -g {resourceGroupName} -n {vmScaleSetName} --instance-id {instanceId}` (Дополнительные сведения см. в [документации по CLI](/cli/azure/vmss)).

Список всех идентификаторов экземпляров можно получить, запросив список всех экземпляров в наборе масштабирования:

REST API: `GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}/virtualMachines?api-version={apiVersion}` (дополнительные сведения см. в [документации по REST API](/rest/api/compute/virtualmachinescalesetvms/list)).

PowerShell: `Get-AzVmssVM -ResourceGroupName {resourceGroupName} -VMScaleSetName {vmScaleSetName}` (дополнительные сведения см. в [документации по PowerShell](/powershell/module/az.compute/get-azvmssvm)).

CLI: `az vmss list-instances -g {resourceGroupName} -n {vmScaleSetName}` (Дополнительные сведения см. в [документации по CLI](/cli/azure/vmss)).

Для получения списка виртуальных машин масштабируемого набора также можно использовать сайт [resources.azure.com](https://resources.azure.com) или [пакеты SDK Azure](https://azure.microsoft.com/downloads/).

Точное представление выходных данных зависит от параметров, которые вы предоставляете команде. Ниже приведены несколько примеров выходных данных CLI:

```azurecli
az vmss show -g {resourceGroupName} -n {vmScaleSetName}
```

```output
[
  {
    "instanceId": "85",
    "latestModelApplied": true,
    "location": "westus",
    "name": "nsgvmss_85",
    .
    .
    .
```

Как видно выше, свойство instanceId является десятичным числом. После удаления старых экземпляров идентификаторы можно повторно использовать для новых экземпляров.

>[!NOTE]
> Нет **гарантии** относительно способа присваивания идентификаторов экземпляров виртуальным машинам в масштабируемом наборе. Может показаться, что они периодически увеличиваются, но это не всегда так. Не имеет значения, каким образом идентификаторы экземпляров присваиваются виртуальным машинам.

## <a name="scale-set-vm-names"></a>Имена виртуальных машин масштабируемого набора

В приведенном выше примере выходных данных также имеется свойство name для виртуальной машины. Это имя имеет форму {имя_масштабируемого_набора}_{ИД_экземпляра}. Оно отображается на портале Azure при отображении списка экземпляров в масштабируемом наборе:

![Снимок экрана, показывающий список экземпляров в масштабируемом наборе виртуальных машин в портал Azure.](./media/virtual-machine-scale-sets-instance-ids/vmssInstances.png)

Часть имени {ИД_экземпляра} — то же десятичное число, что и в свойстве instanceId, указанном выше.

## <a name="instance-metadata-vm-name"></a>Имя виртуальной машины метаданных экземпляра

Если вы запрашиваете [метаданные экземпляра](../virtual-machines/windows/instance-metadata-service.md) из виртуальной машины масштабируемого набора, в выходных данных отображается свойство name:

```output
{
  "compute": {
    "location": "westus",
    "name": "nsgvmss_85",
    .
    .
    .
```

Это имя совпадает с именем, указанным выше.

## <a name="scale-set-vm-computer-name"></a>Имя компьютера виртуальной машины масштабируемого набора

Каждой виртуальной машины в масштабируемом наборе также присваивается имя компьютера. Имя этого компьютера — имя узла виртуальной машины в [предоставленном системой Azure разрешении DNS-имени в виртуальной сети](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md). Это имя компьютера имеет форму {префикс_имени_компьютера}{ИД_экземпляра_base36}.

{ИД_экземпляра_base36} представляется в кодировке [base36](https://en.wikipedia.org/wiki/Base36) и всегда состоит из шести цифр. Если представление в кодировке base36 состоит меньше, чем из шести цифр, часть имени {ИД_экземпляра_base36} дополняется нулями (до шести цифр). Например, экземпляр с частью имени {префикс_имени_компьютера} nsgvmss и идентификатором экземпляра 85 будет иметь имя компьютера nsgvmss00002D.

>[!NOTE]
> Префикс имени компьютера — свойство модели масштабируемого набора, задаваемое пользователем. Он может отличаться от имени масштабируемого набора.
