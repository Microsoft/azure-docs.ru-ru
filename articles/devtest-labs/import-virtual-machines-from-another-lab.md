---
title: Импорт виртуальных машин из другой лаборатории в Azure DevTest Labs
description: В этой статье описывается, как импортировать виртуальные машины из другой лабораторной среды в текущую лабораторную работу в Azure DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 0f664a0ae399575ee936565adaf7364fd1c5ce5c
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85475940"
---
# <a name="import-virtual-machines-from-another-lab-in-azure-devtest-labs"></a>Импорт виртуальных машин из другой лаборатории в Azure DevTest Labs
В этой статье содержатся сведения о том, как импортировать виртуальные машины из другой лабораторной среды в лабораторию.

## <a name="scenarios"></a>Сценарии
Ниже приведены некоторые сценарии, в которых необходимо импортировать виртуальные машины из одной лабораторной среды в другую.

- Человек в группе переходит на другую группу в рамках предприятия и хочет перейти на Рабочий стол разработчика в DevTest Labs новой группы.
- Группа достигла [квоты уровня подписки](../azure-resource-manager/management/azure-subscription-service-limits.md) и хочет разделить группы на несколько подписок.
- Компания переходит на Express Route (или другую новую сетевую топологию), и группа хочет переместить виртуальные машины для использования этой новой инфраструктуры.

## <a name="solution-and-constraints"></a>Решение и ограничения
Эта функция позволяет импортировать виртуальные машины в одной лаборатории (источнике) в другую лабораторию (назначение). При необходимости можно задать новое имя для целевой виртуальной машины в процессе. Процесс импорта включает все зависимости, такие как диски, расписания, параметры сети и т. д.

Процесс занимает некоторое время и влияет на следующие факторы:

- Число или размер дисков, подключенных к исходному компьютеру (так как это операция копирования, а не операция перемещения)
- Расстояние до назначения (например, регион "Восточная часть США" — Юго-Восточная Азия).

После завершения процесса исходная виртуальная машина останется выключенной, а новая — в целевой лаборатории.

Существует два ключевых ограничения, которые следует учитывать при планировании импорта виртуальных машин из одной лаборатории в другую лабораторию.

- Импорты виртуальных машин в подписках и в разных регионах поддерживаются, но подписки должны быть связаны с одним и тем же клиентом Azure Active Directory.
- Виртуальные машины не должны находиться в состоянии, доступном в исходной лаборатории.
- Вы являетесь владельцем виртуальной машины в исходной лаборатории и владельцем лаборатории в целевой лаборатории.
- Сейчас эта функция поддерживается только с помощью PowerShell и REST API.

## <a name="use-powershell"></a>Использование PowerShell
Скачайте файл ImportVirtualMachines.ps1 из [GitHub](https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/Scripts/ImportVirtualMachines). С помощью скрипта можно импортировать одну виртуальную машину или все виртуальные машины в исходной лаборатории в целевую лабораторию.

### <a name="use-powershell-to-import-a-single-vm"></a>Импорт одной виртуальной машины с помощью PowerShell
Для выполнения этого скрипта PowerShell необходимо определить исходную и целевую виртуальные машины, а также при необходимости указать новое имя для конечного компьютера:

```powershell
./ImportVirtualMachines.ps1 -SourceSubscriptionId "<ID of the subscription that contains the source lab>" `
                            -SourceDevTestLabName "<Name of the source lab>" `
                            -SourceVirtualMachineName "<Name of the VM to be imported from the source lab> " `
                            -DestinationSubscriptionId "<ID of the subscription that contians the destination lab>" `
                            -DestinationDevTestLabName "<Name of the destination lab>" `
                            -DestinationVirtualMachineName "<Optional: specify a new name for the imported VM in the destination lab>"
```

### <a name="use-powershell-to-import-all-vms-in-the-source-lab"></a>Импорт всех виртуальных машин в исходной лаборатории с помощью PowerShell
Если исходная виртуальная машина не указана, скрипт автоматически импортирует все виртуальные машины в DevTest Labs.  Пример:

```powershell
./ImportVirtualMachines.ps1 -SourceSubscriptionId "<ID of the subscription that contains the source lab>" `
                            -SourceDevTestLabName "<Name of the source lab>" `
                            -DestinationSubscriptionId "<ID of the subscription that contians the destination lab>" `
                            -DestinationDevTestLabName "<Name of the destination lab>"
```

## <a name="use-http-rest-to-import-a-vm"></a>Использование протокола HTTP RESTFUL для импорта виртуальной машины
Вызов RESTFUL прост. Вы получите достаточно информации для обнаружения исходных и целевых ресурсов. Помните, что операция выполняется в целевом ресурсе лаборатории.

```REST
POST https://management.azure.com/subscriptions/<DestinationSubscriptionID>/resourceGroups/<DestinationResourceGroup>/providers/Microsoft.DevTestLab/labs/<DestinationLab>/ImportVirtualMachine?api-version=2017-04-26-preview
{
   sourceVirtualMachineResourceId: "/subscriptions/<SourceSubscriptionID>/resourcegroups/<SourceResourceGroup>/providers/microsoft.devtestlab/labs/<SourceLab>/virtualmachines/<NameofVMTobeImported>",
   destinationVirtualMachineName: "<NewNameForImportedVM>"
}
```

## <a name="next-steps"></a>Дальнейшие действия
См. следующие статьи:

- [Управление базовыми политиками лаборатории в Azure DevTest Labs](devtest-lab-set-lab-policy.md)
- [Часто задаваемые вопросы по Аналитике компьютеров](devtest-lab-faq.md)
