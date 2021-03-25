---
title: Краткое руководство. Создание масштабируемого набора виртуальных машин Windows с помощью шаблона Azure
description: Узнайте, как быстро создать масштабируемый набор виртуальных машин Windows с помощью шаблона Azure Resource Manager, который позволяет развернуть пример приложения и настроить правила автоматического масштабирования.
author: ju-shim
ms.author: jushiman
ms.topic: quickstart
ms.service: virtual-machine-scale-sets
ms.subservice: windows
ms.date: 03/27/2020
ms.reviewer: mimckitt
ms.custom: mimckitt, subject-armqs
ms.openlocfilehash: 03863171cb05c0557611fb8cd3c0ade667e7d79e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88650083"
---
# <a name="quickstart-create-a-windows-virtual-machine-scale-set-with-an-arm-template"></a>Краткое руководство. Создание масштабируемого набора виртуальных машин Windows с помощью шаблона ARM

Масштабируемый набор виртуальных машин позволяет развернуть набор виртуальных машин с поддержкой автомасштабирования и управлять этим набором. Вы можете вручную изменить число виртуальных машин в масштабируемом наборе или определить правила для автоматического масштабирования в зависимости от использования ЦП, объема памяти или сетевого трафика. После этого Azure Load Balancer будет распределять трафик между экземплярами виртуальных машин в масштабируемом наборе. Выполнив инструкции из этого руководства, вы создадите масштабируемый набор виртуальных машин и развернете пример приложения с помощью шаблона Azure Resource Manager (ARM).

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Шаблоны ARM позволяют развертывать группы связанных ресурсов. С помощью одного шаблона можно создать масштабируемый набор виртуальных машин, установить приложения и настроить правила автоматического масштабирования. Вы можете повторно использовать этот шаблон, применив переменные и параметры, чтобы обновить существующие или создать дополнительные масштабируемые наборы. Шаблоны можно развернуть с помощью портала Azure, Azure CLI, Azure PowerShell либо же на основе конвейеров непрерывной интеграции и непрерывной поставки (CI/CD).

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-vmss-windows-webapp-dsc-autoscale%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/201-vmss-windows-webapp-dsc-autoscale/).

:::code language="json" source="~/quickstart-templates/201-vmss-windows-webapp-dsc-autoscale/azuredeploy.json":::

В этих шаблонах определены следующие ресурсы:

- [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks);
- [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicipaddresses)
- [**Microsoft.Network/loadBalancers**](/azure/templates/microsoft.network/loadbalancers);
- [**Microsoft.Compute/virtualMachineScaleSets**](/azure/templates/microsoft.compute/virtualmachinescalesets);
- [**Microsoft.Insights/autoscaleSettings**](/azure/templates/microsoft.insights/autoscalesettings).

### <a name="define-a-scale-set"></a>Определение масштабируемого набора

Чтобы создать масштабируемый набор с помощью шаблона, определите необходимые ресурсы. Ниже перечислены основные элементы и типы ресурсов для масштабируемого набора виртуальных машин.

| Свойство                     | Описание свойства                                  | Пример значения в шаблоне                    |
|------------------------------|----------------------------------------------------------|-------------------------------------------|
| type                         | Создаваемый тип ресурса Azure                            | Microsoft.Compute/virtualMachineScaleSets; |
| name                         | Имя масштабируемого набора                                       | myScaleSet                                |
| location                     | Расположение для создания масштабируемого набора                     | Восточная часть США                                   |
| sku.name                     | Размер виртуальной машины для каждого экземпляра в масштабируемом наборе                  | Standard_A1                               |
| sku.capacity                 | Количество экземпляров виртуальных машин, которое требуется изначально создать           | 2                                         |
| upgradePolicy.mode           | Режим обновления экземпляра виртуальной машины при внесении изменений              | Автоматически                                 |
| imageReference               | Платформа или пользовательский образ, используемые для экземпляров виртуальных машин | Microsoft Windows Server 2016 Datacenter  |
| osProfile.computerNamePrefix | Префикс имени для каждого экземпляра виртуальной машины                     | myvmss                                    |
| osProfile.adminUsername      | Имя пользователя для каждого экземпляра виртуальной машины                        | azureuser                                 |
| osProfile.adminPassword      | Пароль для каждого экземпляра виртуальной машины                        | P@ssw0rd!                                 |

Чтобы настроить шаблон масштабируемого набора, вы можете изменить размер виртуальной машины или начальную емкость. Еще один вариант — использовать другую платформу или пользовательский образ.

### <a name="add-a-sample-application"></a>Добавление примера приложения

Для проверки масштабируемого набора установите базовое веб-приложение. Когда вы развертываете масштабируемый набор, расширения виртуальных машин могут предусматривать задачи автоматизации и настройки после развертывания, например задачи установки приложения. Сценарии можно скачать из службы хранилища Azure или GitHub или передать на портал Azure во время выполнения расширения. Чтобы применить расширение к масштабируемому набору, добавьте раздел *extensionProfile* в предыдущий пример ресурса. Как правило, профиль расширения определяет следующие свойства:

- тип расширения;
- издатель расширения;
- версия расширения;
- расположение скриптов настройки или установки;
- команды для выполнения на экземплярах виртуальных машин.

В примере шаблона используется расширение PowerShell DSC для установки приложения ASP.NET MVC, выполняющегося в IIS.

Скрипт установки скачивается с сайта GitHub, как определено в свойстве *url*. Затем расширение запускает *InstallIIS* из скрипта *IISInstall.ps1*, как определено в свойствах *function* и *script*. Само приложение ASP.NET предоставляется в виде пакета веб-развертывания, который также скачивается с сайта GitHub, как определено в свойстве *WebDeployPackagePath*:

## <a name="deploy-the-template"></a>Развертывание шаблона

Вы можете развернуть шаблон, нажав кнопку **Развертывание в Azure**. После ее нажатия откроется портал Azure, загрузится весь шаблон и появится диалог для ввода параметров, таких как имя масштабируемого набора, число экземпляров и учетные данные администратора.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-vmss-windows-webapp-dsc-autoscale%2Fazuredeploy.json)

Вы также можете развернуть шаблон Resource Manager с помощью Azure PowerShell:

```azurepowershell-interactive
# Create a resource group
New-AzResourceGroup -Name myResourceGroup -Location EastUS

# Deploy template into resource group
New-AzResourceGroupDeployment `
    -ResourceGroupName myResourceGroup `
    -TemplateURI https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vmss-windows-webapp-dsc-autoscale/azuredeploy.json

# Update the scale set and apply the extension
Update-AzVmss `
    -ResourceGroupName myResourceGroup `
    -VmScaleSetName myVMSS `
    -VirtualMachineScaleSet $vmssConfig
```

По запросу укажите имя масштабируемого набора и учетные данные администратора для экземпляров виртуальных машин. Чтобы создать масштабируемый набор и применить расширение для настройки приложения, может потребоваться 10–15 минут.

## <a name="validate-the-deployment"></a>Проверка развертывания

Чтобы увидеть, как работает масштабируемый набор, откройте пример веб-приложения в браузере. Получите общедоступный IP-адрес своей подсистемы балансировки нагрузки с помощью командлета [Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress), как показано ниже:

```azurepowershell-interactive
Get-AzPublicIpAddress -ResourceGroupName myResourceGroup | Select IpAddress
```

Введите в браузере общедоступный IP-адрес своей подсистемы балансировки нагрузки в формате *http:\//publicIpAddress/MyApp*. Подсистема балансировки нагрузки передаст запрос на один из экземпляров виртуальной машины, как показано в следующем примере:

![Выполнение сайта IIS](./media/virtual-machine-scale-sets-create-powershell/running-iis-site.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить ненужную группу ресурсов и масштабируемый набор с помощью командлета [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup). Параметр `-Force` подтверждает, что вы хотите удалить ресурсы без дополнительного запроса. При использовании параметра `-AsJob` управление возвращается в командную строку без ожидания завершения операции.

```azurepowershell-interactive
Remove-AzResourceGroup -Name "myResourceGroup" -Force -AsJob
```

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого краткого руководства вы создали масштабируемый набор Windows на основе шаблона ARM, а также установили простое приложение ASP.NET на экземплярах виртуальных машин с помощью расширения PowerShell DSC. Чтобы получить дополнительные сведения, продолжите роботу с руководством по созданию масштабируемых наборов виртуальных машин Azure и управлению ими.

> [!div class="nextstepaction"]
> [Создание масштабируемых наборов виртуальных машин Azure и управление ими](tutorial-create-and-manage-powershell.md)
