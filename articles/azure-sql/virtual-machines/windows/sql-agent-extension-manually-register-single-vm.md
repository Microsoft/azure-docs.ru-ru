---
title: Регистрация с помощью расширения агента IaaS для SQL
description: Зарегистрируйте виртуальную машину Azure SQL Server с расширением агента IaaS SQL, чтобы включить функции для SQL Server виртуальных машин, развернутых за пределами Azure Marketplace, а также для обеспечения соответствия и улучшенной управляемости.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
tags: azure-resource-manager
ms.service: virtual-machines-sql
ms.subservice: management
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 11/07/2020
ms.author: mathoma
ms.reviewer: jroth
ms.custom: devx-track-azurecli, devx-track-azurepowershell, contperf-fy21q2
ms.openlocfilehash: 983cdab0c5f5b856537c661c7427a83099f30ed4
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102181437"
---
# <a name="register-sql-server-vm-with-sql-iaas-agent-extension"></a>Регистрация SQL Server виртуальной машины с помощью расширения агента IaaS SQL
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Регистрация виртуальной машины SQL Server с [расширением агента IaaS SQL](sql-server-iaas-agent-extension-automate-management.md) позволяет разблокировать множество преимуществ для SQL Server на виртуальной машине Azure. 

В этой статье описывается, как зарегистрировать одну виртуальную машину SQL Server с расширением агента IaaS SQL. Кроме того, можно зарегистрировать все SQL Server виртуальные машины [автоматически](sql-agent-extension-automatic-registration-all-vms.md) или [несколько виртуальных машин, которые были внесены в пакетную](sql-agent-extension-manually-register-vms-bulk.md)операцию.


## <a name="overview"></a>Обзор

При регистрации в [расширении агента SQL Server IaaS](sql-server-iaas-agent-extension-automate-management.md) в подписке создается _ресурс_ **виртуальной машины SQL** , который является _отдельным_ ресурсом из ресурса виртуальной машины. При отмене регистрации ВИРТУАЛЬНОЙ машины SQL Server в расширении _ресурс_ **виртуальной машины SQL** будет удален, но фактические виртуальные машины не будут удалены.

При развертывании образа виртуальной машины Azure Marketplace SQL Server с помощью портал Azure автоматически регистрирует виртуальную машину SQL Server с расширением. Однако, если вы решили самостоятельно установить SQL Server на виртуальной машине Azure или подготовили виртуальную машину Azure из пользовательского виртуального жесткого диска, то необходимо зарегистрировать ВИРТУАЛЬную машину SQL Server с расширением агента SQL IaaS, чтобы разблокировать все преимущества функций и управляемость. 

Чтобы использовать расширение агента IaaS SQL, необходимо сначала [зарегистрировать подписку с помощью поставщика **Microsoft. склвиртуалмачине**](#register-subscription-with-rp), который предоставляет расширению SQL IaaS возможность создавать ресурсы в рамках этой конкретной подписки.

> [!IMPORTANT]
> Расширение агента IaaS SQL собирает данные для предоставления клиентам дополнительных преимуществ при использовании SQL Server в виртуальных машинах Azure. Корпорация Майкрософт не будет использовать эти данные для аудита лицензирования без согласия клиента. Дополнительные сведения см. в разделе [SQL Server о конфиденциальности](/sql/sql-server/sql-server-privacy#non-personal-data) .

## <a name="prerequisites"></a>Предварительные условия

Чтобы зарегистрировать виртуальную машину SQL Server с расширением, вам потребуется: 

- [Подписка Azure](https://azure.microsoft.com/free/).
- Модель ресурсов Azure — это [Виртуальная машина Windows Server 2008 (или более поздняя)](../../../virtual-machines/windows/quick-create-portal.md) с [SQL Server 2008 (или более поздней)](https://www.microsoft.com/sql-server/sql-server-downloads) , развернутая в общедоступном облаке или Azure для государственных организаций 
- Последняя версия [Azure CLI](/cli/azure/install-azure-cli) или [Azure PowerShell (5,0 минимум)](/powershell/azure/install-az-ps). 


## <a name="register-subscription-with-rp"></a>Регистрация подписки с помощью поставщика ресурсов

Чтобы зарегистрировать виртуальную машину SQL Server с расширением агента IaaS SQL, сначала необходимо зарегистрировать подписку с помощью поставщика **Microsoft. склвиртуалмачине** . Это дает расширению агента SQL IaaS возможность создавать ресурсы в подписке.  Это можно сделать с помощью портал Azure, Azure CLI или Azure PowerShell.

### <a name="azure-portal"></a>Портал Azure

1. Войдите на портал Azure и откройте раздел **Все службы**. 
1. Перейдите к разделу **Подписки** и выберите нужную подписку.  
1. На странице **подписки** перейдите к разделу **расширения**. 
1. В фильтре введите **SQL** , чтобы открыть расширения, связанные с SQL. 
1. Выберите действие **Зарегистрировать**, **Перерегистрировать** или **Отменить регистрацию** для поставщика **Microsoft.SqlVirtualMachine** в зависимости от ваших намерений. 

   ![Изменение поставщика](./media/sql-agent-extension-manually-register-single-vm/select-resource-provider-sql.png)


### <a name="command-line"></a>Командная строка

Зарегистрируйте подписку Azure в поставщике **Microsoft. склвиртуалмачине** , используя Azure CLI или Azure PowerShell. 

# <a name="azure-cli"></a>[Azure CLI](#tab/bash)

```azurecli-interactive
# Register the SQL IaaS Agent extension to your subscription 
az provider register --namespace Microsoft.SqlVirtualMachine 
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

```powershell-interactive
# Register the SQL IaaS Agent extension to your subscription
Register-AzResourceProvider -ProviderNamespace Microsoft.SqlVirtualMachine
```

---

## <a name="register-with-extension"></a>Регистрация с помощью расширения

Существует три режима управления для [расширения агента SQL Server IaaS](sql-server-iaas-agent-extension-automate-management.md#management-modes). 

Регистрация расширения в режиме полного управления перезапускает службу SQL Server, поэтому рекомендуется сначала зарегистрировать расширение в упрощенном режиме, а затем выполнить [обновление до полного](#upgrade-to-full) периода обслуживания. 

### <a name="lightweight-management-mode"></a>Режим упрощенного управления

Используйте Azure CLI или Azure PowerShell, чтобы зарегистрировать виртуальную машину SQL Server с расширением в упрощенном режиме. Это не приведет к перезапуску службы SQL Server. Затем в любое время можно выполнить обновление до полного режима, но это приведет к перезапуску службы SQL Server. Поэтому рекомендуется дождаться периода запланированного обслуживания. 

Укажите тип лицензии SQL Server как оплата по мере `PAYG` использования () для оплаты за использование, преимущество гибридного использования Azure ( `AHUB` ) для использования собственной лицензии или аварийное восстановление ( `DR` ) для активации [бесплатной лицензии на реплику аварийного](business-continuity-high-availability-disaster-recovery-hadr-overview.md#free-dr-replica-in-azure)восстановления.

Экземпляры отказоустойчивого кластера и развертывания с несколькими экземплярами могут быть зарегистрированы только с расширением агента IaaS SQL в упрощенном режиме. 

# <a name="azure-cli"></a>[Azure CLI](#tab/bash)

Зарегистрируйте SQL Serverную виртуальную машину в упрощенном режиме с Azure CLI: 

  ```azurecli-interactive
  # Register Enterprise or Standard self-installed VM in Lightweight mode
  az sql vm create --name <vm_name> --resource-group <resource_group_name> --location <vm_location> --license-type <license_type> 
  ```


# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

Зарегистрируйте SQL Serverную виртуальную машину в упрощенном режиме с Azure PowerShell:  


  ```powershell-interactive
  # Get the existing compute VM
  $vm = Get-AzVM -Name <vm_name> -ResourceGroupName <resource_group_name>
          
  # Register SQL VM with 'Lightweight' SQL IaaS agent
  New-AzSqlVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName -Location $vm.Location `
    -LicenseType <license_type>  -SqlManagementType LightWeight  
  ```

---

### <a name="full-management-mode"></a>Режим полного управления

Регистрация виртуальной машины SQL Server в полном режиме приведет к перезапуску службы SQL Server. Будьте внимательны. 

Чтобы зарегистрировать виртуальную машину SQL Server напрямую в полном режиме (и, возможно, перезапустить службу SQL Server), используйте следующую команду Azure PowerShell: 

  ```powershell-interactive
  # Get the existing  Compute VM
  $vm = Get-AzVM -Name <vm_name> -ResourceGroupName <resource_group_name>
        
  # Register with SQL IaaS Agent extension in full mode
  New-AzSqlVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName -SqlManagementType Full
  ```

### <a name="noagent-management-mode"></a>Режим управления NoAgent 

SQL Server 2008 и 2008 R2, установленные на Windows Server 2008 (_не R2_), можно зарегистрировать с расширением агента IaaS SQL в режиме "без [агента](sql-server-iaas-agent-extension-automate-management.md#management-modes)". Этот вариант обеспечивает соответствие и позволяет отслеживать виртуальную машину SQL Server на портале Azure с ограниченными функциональными возможностями.


Для **типа лицензии** укажите один из следующих типов: `AHUB` , `PAYG` или `DR` . Для **предложения образа** `SQL2008-WS2008` Укажите либо `SQL2008R2-WS2008`

Чтобы зарегистрировать SQL Server 2008 ( `SQL2008-WS2008` ) или 2008 R2 ( `SQL2008R2-WS2008` ) в экземпляре Windows Server 2008, используйте следующий Azure CLI или Azure PowerShell фрагмент кода: 


# <a name="azure-cli"></a>[Azure CLI](#tab/bash)

Зарегистрируйте виртуальную машину SQL Server в режиме агента с Azure CLI: 

  ```azurecli-interactive
   az sql vm create -n sqlvm -g myresourcegroup -l eastus |
   --license-type <license type>  --sql-mgmt-type NoAgent 
   --image-sku Enterprise --image-offer <image offer> 
 ```
 

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)


Зарегистрируйте SQL Server виртуальную машину в режиме "без агента" с Azure PowerShell: 

  ```powershell-interactive
  # Get the existing compute VM
  $vm = Get-AzVM -Name <vm_name> -ResourceGroupName <resource_group_name>
          
  New-AzSqlVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName -Location $vm.Location `
    -LicenseType <license type> -SqlManagementType NoAgent -Sku Standard -Offer <image offer>
  ```

---

## <a name="verify-mode"></a>Режим проверки

Текущий режим агента SQL Server IaaS можно просмотреть с помощью Azure PowerShell. 

```powershell-interactive
# Get the SqlVirtualMachine
$sqlvm = Get-AzSqlVM -Name $vm.Name  -ResourceGroupName $vm.ResourceGroupName
$sqlvm.SqlManagementType
```

## <a name="upgrade-to-full"></a>Обновить до полной версии  

SQL Server виртуальные машины, которые зарегистрировали расширение в *упрощенном* режиме, могут выполнить обновление до _полной_ версии с помощью портал Azure, Azure CLI или Azure PowerShell. Виртуальные машины SQL Server в режиме _NoAgent_ можно обновить до _полного_ режима после обновления операционной системы до Windows 2008 R2 или выше. Переход на использование более ранней версии невозможен. вам потребуется [отменить регистрацию](#unregister-from-extension) SQL Server виртуальной машины из расширения агента IaaS SQL. При этом будет удален _ресурс_ **виртуальной машины SQL**, но не фактическая виртуальная машина. 

> [!NOTE]
> При обновлении режима управления для расширения SQL IaaS до полного перезапуска будет перезапущена служба SQL Server. В некоторых случаях перезагрузка может привести к тому, что имена субъектов-служб (SPN), связанные со службой SQL Server, будут изменены на неправильную учетную запись пользователя. При наличии проблем с подключением после обновления режима управления до полной [Отмените регистрацию и повторно зарегистрируйте имена участников-служб](/sql/database-engine/configure-windows/register-a-service-principal-name-for-kerberos-connections).


### <a name="azure-portal"></a>Портал Azure

Чтобы обновить расширение до полного режима с помощью портал Azure, выполните следующие действия. 

1. Войдите на [портал Azure](https://portal.azure.com).
1. Перейдите к ресурсу [виртуальных машин SQL](manage-sql-vm-portal.md#access-the-sql-virtual-machines-resource). 
1. Выберите SQL Server виртуальную машину и щелкните **Обзор**. 
1. Для виртуальных машин IaaS для SQL Server с упрощенным режимом IaaS или NoAgent выберите сообщение **Only license type and edition updates are available with the SQL IaaS extension** (Для режима расширения SQL IaaS доступно только изменение типа лицензии и выпуска).

   ![Выбранные элементы изменения режима на портале](./media/sql-agent-extension-manually-register-single-vm/change-sql-iaas-mode-portal.png)

1. Установите флажок **Я соглашаюсь перезапустить службу SQL Server на виртуальной машине**, а затем выберите **Подтвердить**, чтобы обновить режим IaaS до полного. 

    ![Флажок, подтверждающий согласие перезапуска службы SQL Server на виртуальной машине](./media/sql-agent-extension-manually-register-single-vm/enable-full-mode-iaas.png)

### <a name="command-line"></a>Командная строка

# <a name="azure-cli"></a>[Azure CLI](#tab/bash)

Чтобы обновить расширение до полного режима, выполните следующий фрагмент кода Azure CLI:

  ```azurecli-interactive
  # Update to full mode
  az sql vm update --name <vm_name> --resource-group <resource_group_name> --sql-mgmt-type full  
  ```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

Чтобы обновить расширение до полного режима, выполните следующий фрагмент кода Azure PowerShell:

  ```powershell-interactive
  # Get the existing  Compute VM
  $vm = Get-AzVM -Name <vm_name> -ResourceGroupName <resource_group_name>
        
  # Register with SQL IaaS Agent extension in full mode
  Update-AzSqlVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName -SqlManagementType Full
  ```

---

## <a name="verify-registration-status"></a>Проверка состояния регистрации
Вы можете проверить, зарегистрирована ли SQL Server виртуальная машина с расширением агента IaaS SQL с помощью портал Azure, Azure CLI или Azure PowerShell. 

### <a name="azure-portal"></a>Портал Azure 

Чтобы проверить состояние регистрации с помощью портал Azure, выполните следующие действия. 

1. Войдите на [портал Azure](https://portal.azure.com). 
1. Перейдите на [SQL Server виртуальные машины](manage-sql-vm-portal.md).
1. Выберите свою виртуальную машину SQL Server из списка. Если ваша SQL Server виртуальная машина не указана здесь, скорее всего, она не была зарегистрирована в расширении агента IaaS SQL. 
1. Просмотрите значение в разделе **Состояние**. Если **состояние** — " **успешно**", SQL Server виртуальная машина успешно зарегистрирована в расширении агента IaaS SQL. 

   ![Проверка состояния при регистрации с помощью поставщика ресурсов SQL](./media/sql-agent-extension-manually-register-single-vm/verify-registration-status.png)

### <a name="command-line"></a>Командная строка

Проверьте текущее состояние регистрации виртуальной машины SQL Server с помощью Azure CLI или Azure PowerShell. Параметр `ProvisioningState` отображает состояние `Succeeded`, если регистрация прошла успешно. 

# <a name="azure-cli"></a>[Azure CLI](#tab/bash)

Чтобы проверить состояние регистрации с помощью Azure CLI, выполните следующий фрагмент кода:  

  ```azurecli-interactive
  az sql vm show -n <vm_name> -g <resource_group>
 ```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

Чтобы проверить состояние регистрации с помощью Azure PowerShell, выполните следующий фрагмент кода:

  ```powershell-interactive
  Get-AzSqlVM -Name <vm_name> -ResourceGroupName <resource_group>
  ```

---

Ошибка означает, что SQL Serverная виртуальная машина не зарегистрирована в расширении. 


## <a name="unregister-from-extension"></a>Отменить регистрацию в расширении

Чтобы отменить регистрацию SQL Server виртуальной машины с расширением агента IaaS SQL, удалите *ресурс* ВИРТУАЛЬНОЙ машины SQL с помощью портал Azure или Azure CLI. При удалении *ресурса* ВИРТУАЛЬНОЙ машины SQL SQL Server виртуальная машина не удаляется. Тем не менее будьте внимательны и осторожны при выполнении этих действий, так как при попытке удалить *ресурс* можно случайно удалить виртуальную машину. 

Отмена регистрации виртуальной машины SQL с расширением агента IaaS SQL необходима для перехода в режим управления с полной на более раннюю версию. 

### <a name="azure-portal"></a>Портал Azure

Чтобы отменить регистрацию SQL Server виртуальной машины из расширения с помощью портал Azure, выполните следующие действия.

1. Войдите на [портал Azure](https://portal.azure.com).
1. Перейдите к ресурсу виртуальной машины SQL. 
  
   ![Ресурс "Виртуальные машины SQL"](./media/sql-agent-extension-manually-register-single-vm/sql-vm-manage.png)

1. Выберите команду **Удалить**. 

   ![Выберите Удалить в верхней области навигации.](./media/sql-agent-extension-manually-register-single-vm/delete-sql-vm-resource.png)

1. Введите имя виртуальной машины SQL и **снимите флажок рядом с этой виртуальной машиной**.

   ![Снимите флажок для виртуальной машины, чтобы предотвратить удаление реальной виртуальной машины, а затем нажмите кнопку Удалить, чтобы продолжить удаление ресурса виртуальной машины SQL.](./media/sql-agent-extension-manually-register-single-vm/confirm-delete-of-resource-uncheck-box.png)

   >[!WARNING]
   > Если не снять флажок рядом с именем виртуальной машины, то она будет *удалена* полностью. Снимите флажок, чтобы отменить регистрацию SQL Server виртуальной машины из расширения, но *не удаляйте фактическую виртуальную машину*. 

1. Выберите **Удалить** , чтобы подтвердить удаление *ресурса* виртуальной машины SQL, а не SQL Server виртуальной машины. 

### <a name="command-line"></a>Командная строка

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы отменить регистрацию SQL Server виртуальной машины из расширения с Azure CLI, используйте команду [AZ SQL VM Delete](/cli/azure/sql/vm#az-sql-vm-delete) . Это приведет к удалению *ресурса* виртуальной машины SQL Server, но не удалит виртуальную машину. 


```azurecli-interactive
az sql vm delete 
  --name <SQL VM resource name> |
  --resource-group <Resource group name> |
  --yes 
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Чтобы отменить регистрацию SQL Server виртуальной машины из расширения с Azure PowerShell, используйте команду [Remove-азсклвм](/powershell/module/az.sqlvirtualmachine/remove-azsqlvm). Это приведет к удалению *ресурса* виртуальной машины SQL Server, но не удалит виртуальную машину. 

```powershell-interactive
Remove-AzSqlVM -ResourceGroupName <resource_group_name> -Name <VM_name>
```

---


## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в следующих статьях: 

* [Обзор SQL Server на виртуальных машинах Windows](sql-server-on-azure-vm-iaas-what-is-overview.md).
* [Вопросы и ответы об SQL Server на виртуальных машинах Windows](frequently-asked-questions-faq.md)  
* [Руководство по ценам для виртуальных машин SQL Server в Azure](pricing-guidance.md)
* [Заметки о выпуске SQL Server на виртуальных машинах Windows](../../database/doc-changes-updates-release-notes.md)
