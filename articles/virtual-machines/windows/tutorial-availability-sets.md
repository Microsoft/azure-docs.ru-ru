---
title: Развертывание виртуальных машин в группе доступности с помощью Azure PowerShell
description: Узнайте, как развертывать высокодоступные виртуальные машины в группах доступности с помощью Azure PowerShell
services: virtual-machines
author: mimckitt
ms.service: virtual-machines
ms.topic: how-to
ms.date: 3/8/2021
ms.author: mimckitt
ms.reviewer: cynthn
ms.custom: mvc
ms.openlocfilehash: 178a29ea37195ddd2013ca5220663a75132beb24
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102555913"
---
# <a name="create-and-deploy-virtual-machines-in-an-availability-set-using-azure-powershell"></a>Создание и развертывание виртуальных машин в группе доступности с помощью Azure PowerShell

В этом руководстве показано, как повысить доступность и надежность виртуальных машин с помощью групп доступности. Группы доступности распределяют развернутые в Azure виртуальные машины между несколькими изолированными аппаратными узлами в кластере. 

В этом руководстве описано следующее:

> [!div class="checklist"]
> * "Создать группу доступности"
> * Создание виртуальной машины в группе доступности
> * Проверка доступных размеров виртуальных машин.
> * Проверка службы "Помощник по Azure"


## <a name="launch-azure-cloud-shell"></a>Запуск Azure Cloud Shell

Azure Cloud Shell — это бесплатная интерактивная оболочка, с помощью которой можно выполнять действия, описанные в этой статье. Она включает предварительно установленные общие инструменты Azure и настроена для использования с вашей учетной записью. 

Чтобы открыть Cloud Shell, просто выберите **Попробовать** в правом верхнем углу блока кода. Cloud Shell можно также запустить в отдельной вкладке браузера, перейдя на страницу [https://shell.azure.com/powershell](https://shell.azure.com/powershell). Нажмите кнопку **Копировать**, чтобы скопировать блоки кода. Вставьте код в Cloud Shell и нажмите клавишу "ВВОД", чтобы выполнить его.

## <a name="create-an-availability-set"></a>"Создать группу доступности"

Оборудование в расположении делится на несколько доменов обновления и доменов сбоя. **Домен обновления** — это группа виртуальных машин и базового физического оборудования, которую можно перезагрузить одновременно. Виртуальные машины в одном **домене сбоя** совместно используют общее хранилище, а также общий источник питания и сетевой коммутатор.  

Создать группу доступности можно с помощью командлета [New-AzAvailabilitySet](/powershell/module/az.compute/new-azavailabilityset). В этом примере количество доменов сбоя и обновления — *2*, а группа доступности называется *myAvailabilitySet*.

Создайте группу ресурсов.

```azurepowershell-interactive
New-AzResourceGroup `
   -Name myResourceGroupAvailability `
   -Location EastUS
```

Создайте управляемую группу доступности с помощью командлета [New-AzAvailabilitySet](/powershell/module/az.compute/new-azavailabilityset) с параметром `-sku aligned`.

```azurepowershell-interactive
New-AzAvailabilitySet `
   -Location "EastUS" `
   -Name "myAvailabilitySet" `
   -ResourceGroupName "myResourceGroupAvailability" `
   -Sku aligned `
   -PlatformFaultDomainCount 2 `
   -PlatformUpdateDomainCount 2
```

## <a name="create-vms-inside-an-availability-set"></a>Создание виртуальных машин в группе доступности
Чтобы виртуальные машины правильно распределялись по оборудованию, их нужно создать внутри группы доступности. Существующую виртуальную машину невозможно добавить в группу доступности после создания. 


При создании конфигурации виртуальной машины с помощью командлета [New-AzVM](/powershell/module/az.compute/new-azvm) можно использовать параметр `-AvailabilitySetName`, чтобы указать имя группы доступности.

Сначала укажите имя и пароль администратора для виртуальной машины с помощью командлета [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential):

```azurepowershell-interactive
$cred = Get-Credential
```

Создайте две виртуальные машины в группе доступности с помощью командлета [New-AzVM](/powershell/module/az.compute/new-azvm).

```azurepowershell-interactive
for ($i=1; $i -le 2; $i++)
{
    New-AzVm `
        -ResourceGroupName "myResourceGroupAvailability" `
        -Name "myVM$i" `
        -Location "East US" `
        -VirtualNetworkName "myVnet" `
        -SubnetName "mySubnet" `
        -SecurityGroupName "myNetworkSecurityGroup" `
        -PublicIpAddressName "myPublicIpAddress$i" `
        -AvailabilitySetName "myAvailabilitySet" `
        -Credential $cred
}
```

Создание и настройка двух виртуальных машин занимает несколько минут. По завершении у вас будут две виртуальные машины, распределенные по базовому оборудованию. 

Если посмотреть на группу доступности на портале, перейдя в раздел **Группы ресурсов** > **myResourceGroupAvailability** > **myAvailabilitySet**, можно увидеть, как виртуальные машины распределены между двумя доменами сбоя и обновления.

![Группа доступности на портале](./media/tutorial-availability-sets/fd-ud.png)

## <a name="check-for-available-vm-sizes"></a>Знакомство с доступными размерами виртуальной машины 

При добавлении виртуальной машины в группу доступности следует знать, какие размеры виртуальных машин доступны на оборудовании. Используйте команду [Get-AzVMSize](/powershell/module/az.compute/get-azvmsize), чтобы получить все доступные размеры для виртуальных машин, которые можно развернуть в группе доступности.

```azurepowershell-interactive
Get-AzVMSize `
   -ResourceGroupName "myResourceGroupAvailability" `
   -AvailabilitySetName "myAvailabilitySet"
```

## <a name="check-azure-advisor"></a>Проверка службы "Помощник по Azure" 

Службу "Помощник по Azure" можно также использовать для получения дополнительных сведений о том, как повысить доступность виртуальных машин. Помощник по Azure анализирует конфигурацию и данные телеметрии использования и рекомендует решения, которые помогут повысить эффективность затрат, производительность, уровень доступности и безопасности ресурсов Azure.

Войдите на [портал Azure](https://portal.azure.com), выберите **Все службы** и введите **Помощник**. Панель мониторинга службы "Помощник" отображает персонализированные рекомендации для выбранной подписки. Дополнительные сведения можно найти в разделе [Приступая к работе с Azure Advisor](../../advisor/advisor-get-started.md).


## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как выполнять следующие задачи:

> [!div class="checklist"]
> * "Создать группу доступности"
> * Создание виртуальной машины в группе доступности
> * Проверка доступных размеров виртуальных машин.
> * Проверка службы "Помощник по Azure"

Перейдите к следующему руководству, чтобы узнать о масштабируемых наборах виртуальных машин.

> [!div class="nextstepaction"]
> [Создание масштабируемого набора виртуальной машины](tutorial-create-vmss.md)
