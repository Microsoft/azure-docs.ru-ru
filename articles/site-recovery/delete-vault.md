---
title: Удаление хранилища Azure Site Recovery
description: Узнайте, как удалить хранилище Служб восстановления, настроенного для Azure Site Recovery
author: sideeksh
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 11/05/2019
ms.author: sideeksh
ms.openlocfilehash: a33e04a24013d5450c98b91048fa418958d16886
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89426390"
---
# <a name="delete-a-site-recovery-services-vault"></a>Удаление хранилища Служб восстановления для Site Recovery

В этой статье описывается, как удалить хранилище служб восстановления для Site Recovery. Чтобы удалить хранилище, используемое в Azure Backup, см. сведения в статье [Удаление резервного хранилища в Azure](../backup/backup-azure-delete-vault.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]


## <a name="before-you-start"></a>Перед началом работы

Прежде чем можно будет удалить хранилище, необходимо удалить зарегистрированные серверы и элементы в хранилище. То, что необходимо удалить, зависит от развернутых сценариев репликации. 


## <a name="delete-a-vault-azure-vm-to-azure"></a>Удаление хранилища виртуальной машины Azure в Azure

1. Выполните [эти инструкции](site-recovery-manage-registration-and-protection.md#disable-protection-for-a-azure-vm-azure-to-azure) , чтобы удалить все защищенные виртуальные машины.
2. Затем удалите хранилище.

## <a name="delete-a-vault-vmware-vm-to-azure"></a>Удаление хранилища виртуальной машины VMware в Azure

1. Выполните [эти инструкции](site-recovery-manage-registration-and-protection.md#disable-protection-for-a-vmware-vm-or-physical-server-vmware-to-azure) , чтобы удалить все защищенные виртуальные машины.
2. Выполните следующие [действия](vmware-azure-set-up-replication.md#disassociate-or-delete-a-replication-policy) , чтобы удалить все политики репликации.
3. Удалите ссылки на vCenter, выполнив [следующие действия](vmware-azure-manage-vcenter.md#delete-a-vcenter-server).
4. Выполните [эти инструкции](vmware-azure-manage-configuration-server.md#delete-or-unregister-a-configuration-server) , чтобы списать сервер конфигурации.
5. Затем удалите хранилище.


## <a name="delete-a-vault-hyper-v-vm-with-vmm-to-azure"></a>Удаление хранилища виртуальной машины Hyper-V (с VMM) в Azure

1. Выполните следующие [действия](site-recovery-manage-registration-and-protection.md#disable-protection-for-a-hyper-v-virtual-machine-replicating-to-azure-using-the-system-center-vmm-to-azure-scenario) , чтобы удалить виртуальные машины Hyper-V под управлением System Center VMM.
2. Отменяет связь и удаляет все политики репликации. Это необходимо сделать в хранилище > **Site Recovery инфраструктуры**  >  **для**  >  **политик репликации** VMM System Center.
3. Выполните следующие [действия](site-recovery-manage-registration-and-protection.md#unregister-a-vmm-server) , чтобы отменить регистрацию подключенного сервера VMM.
4. Затем удалите хранилище.

## <a name="delete-a-vault-hyper-v-vm-to-azure"></a>Удаление хранилища виртуальной машины Hyper-V в Azure

1. Выполните следующие [действия](site-recovery-manage-registration-and-protection.md#disable-protection-for-a-hyper-v-virtual-machine-hyper-v-to-azure) , чтобы удалить все защищенные виртуальные машины.
2. Отменяет связь и удаляет все политики репликации. Это необходимо сделать в хранилище > **Site Recovery инфраструктуры**  >  **для политик репликации сайтов Hyper-V**  >  .
3. Выполните [эти инструкции](site-recovery-manage-registration-and-protection.md#unregister-a-hyper-v-host-in-a-hyper-v-site) , чтобы отменить регистрацию узла Hyper-V.
4. Удалите сайт Hyper-V.
5. Затем удалите хранилище.


## <a name="use-powershell-to-force-delete-the-vault"></a>Принудительное удаление хранилища с помощью PowerShell 

> [!Important]
> Если вы тестируете продукт и вас не беспокоит возможная потеря данных, используйте метод принудительного удаления для быстрого удаления хранилища и всех его зависимостей.
> Команда PowerShell удаляет все содержимое хранилища и **не является обратимой**.

Используйте приведенные ниже команды, чтобы удалить хранилище Site Recovery, даже если оно содержит защищенные элементы.

```azurepowershell
Connect-AzAccount

Select-AzSubscription -SubscriptionName "XXXXX"

$vault = Get-AzRecoveryServicesVault -Name "vaultname"

Remove-AzRecoveryServicesVault -Vault $vault
```

Дополнительные сведения о [Get-азрековерисервицесваулт](/powershell/module/az.recoveryservices/get-azrecoveryservicesvault)и [Remove-азрековерисервицесваулт](/powershell/module/az.recoveryservices/remove-azrecoveryservicesvault).
