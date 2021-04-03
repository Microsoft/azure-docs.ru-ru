---
title: Краткое руководство по резервному копированию виртуальной машины шаблона Resource Manager
description: Узнайте, как создавать резервные копии виртуальных машин с помощью шаблона Azure Resource Manager.
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 05/14/2019
ms.custom: mvc,subject-armqs
ms.openlocfilehash: e64aa10d8ddadd367d04b2b480770a99f3ece1dc
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "88826537"
---
# <a name="quickstart-back-up-a-virtual-machine-in-azure-with-an-arm-template"></a>Краткое руководство. Резервное копирование виртуальной машины в Azure с помощью шаблона ARM

[Azure Backup](backup-overview.md) создает резервные копии локальных компьютеров, приложений и виртуальных машин Azure. В этой статье описывается резервное копирование виртуальной машины Azure с помощью шаблона Azure Resource Manager (ARM) и Azure PowerShell. В данном кратком руководстве рассматривается преимущественно процесс развертывания шаблона ARM для создания хранилища Служб восстановления. Дополнительные сведения о разработке шаблонов ARM см. в разделе [Документация по Azure Resource Manager](../azure-resource-manager/index.yml) и в [справочнике по шаблонам](/azure/templates/microsoft.recoveryservices/allversions).

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

[Хранилище служб восстановления](backup-azure-recovery-services-vault-overview.md) — это логический контейнер, в котором хранятся данные резервного копирования защищенных ресурсов, например виртуальных машин Azure. Когда выполняется задание резервного копирования, в хранилище Служб восстановления создается точка восстановления. Позже вы сможете использовать одну из этих точек восстановления, чтобы восстановить данные до определенной точки во времени. Кроме того, вы можете выполнить резервное копирование виртуальной машины с помощью [Azure PowerShell](./quick-backup-vm-powershell.md), [Azure CLI](quick-backup-vm-cli.md) или [портала Azure](quick-backup-vm-portal.md).

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-recovery-services-create-vm-and-configure-backup%2Fazuredeploy.json)

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-recovery-services-create-vm-and-configure-backup/). Этот шаблон позволяет развернуть простую виртуальную машину Windows и хранилище Служб восстановления с конфигурацией DefaultPolicy, имеющей значение "Protection" (Защита).

:::code language="json" source="~/quickstart-templates/101-recovery-services-create-vm-and-configure-backup/azuredeploy.json":::

В шаблоне определены следующие ресурсы:

- [**Microsoft.Storage/storageAccounts**](/azure/templates/microsoft.storage/storageaccounts)
- [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicipaddresses)
- [**Microsoft.Network/networkSecurityGroups;**](/azure/templates/microsoft.network/networksecuritygroups)
- [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks);
- [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkinterfaces)
- [**Microsoft.Compute/virutalMachines**](/azure/templates/microsoft.compute/virtualmachines)
- [**Microsoft.RecoveryServices/vaults**](/azure/templates/microsoft.recoveryservices/2016-06-01/vaults)
- [**Microsoft.RecoveryServices/vaults/backupFabrics/protectionContainers/protectedItems**](/azure/templates/microsoft.recoveryservices/vaults/backupfabrics/protectioncontainers/protecteditems)

## <a name="deploy-the-template"></a>Развертывание шаблона

Для развертывания шаблона щелкните **Попробовать**, чтобы открыть Azure Cloud Shell, а затем вставьте следующий скрипт PowerShell в окно оболочки. Чтобы вставить код, щелкните окно оболочки правой кнопкой мыши, а затем выберите **Вставить**.

```azurepowershell-interactive
$projectName = Read-Host -Prompt "Enter a project name (limited to eight characters) that is used to generate Azure resource names"
$location = Read-Host -Prompt "Enter the location (for example, centralus)"
$adminUsername = Read-Host -Prompt "Enter the administrator username for the virtual machine"
$adminPassword = Read-Host -Prompt "Enter the administrator password for the virtual machine" -AsSecureString
$dnsPrefix = Read-Host -Prompt "Enter the unique DNS Name for the Public IP used to access the virtual machine"

$resourceGroupName = "${projectName}rg"
$templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-recovery-services-create-vm-and-configure-backup/azuredeploy.json"

New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri -projectName $projectName -adminUsername $adminUsername -adminPassword $adminPassword -dnsLabelPrefix $dnsPrefix
```

В этом кратком руководстве для развертывания шаблона ARM используется Azure PowerShell. [Портал Azure](../azure-resource-manager/templates/deploy-portal.md), [Azure CLI](../azure-resource-manager/templates/deploy-cli.md) и [REST API](../azure-resource-manager/templates/deploy-rest.md) также можно использовать для развертывания шаблонов.

## <a name="validate-the-deployment"></a>Проверка развертывания

### <a name="start-a-backup-job"></a>Запуск задания резервного копирования

Шаблон создает виртуальную машину и включает на ней резервное копирование. После развертывания шаблона необходимо запустить задание резервного копирования. Дополнительные сведения см. в разделе [Запуск задания резервного копирования](./quick-backup-vm-powershell.md#start-a-backup-job).

### <a name="monitor-the-backup-job"></a>Мониторинг задания резервного копирования

Мониторинг задания резервного копирования описывается в разделе [Мониторинг задания резервного копирования](./quick-backup-vm-powershell.md#monitor-the-backup-job).

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вам больше не требуется резервная копия виртуальной машины, ее можно удалить.

- Если вы хотите восстановить виртуальною машину, пропустите процедуру очистки.
- Если вы использовали имеющуюся виртуальную машину, можете пропустить последний командлет [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup), чтобы оставить группу ресурсов и виртуальную машину.

Отключите защиту, удалите точки восстановления и хранилище. Затем удалите группу ресурсов и связанные ресурсы виртуальной машины следующим образом:

```powershell
Disable-AzRecoveryServicesBackupProtection -Item $item -RemoveRecoveryPoints
$vault = Get-AzRecoveryServicesVault -Name "myRecoveryServicesVault"
Remove-AzRecoveryServicesVault -Vault $vault
Remove-AzResourceGroup -Name "myResourceGroup"
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы создали хранилище служб восстановления, включили защиту на виртуальной машине и создали начальную точку восстановления.

- [Узнайте, как](tutorial-backup-vm-at-scale.md) создать резервные копии виртуальных машин на портале Azure.
- [Узнайте, как](tutorial-restore-disk.md) быстро восстановить виртуальную машину.
- [Узнайте, как](../azure-resource-manager/templates/template-tutorial-create-first-template.md) создать шаблоны ARM.
