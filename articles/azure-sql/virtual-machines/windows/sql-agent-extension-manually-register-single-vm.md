---
title: Регистрация с расширением SQL IaaS Agent
description: Для включения функций виртуальных машин SQL Server, развернутых за пределами Azure Marketplace, а также для обеспечения соответствия и улучшения управляемости следует провести регистрацию виртуальной машины Azure SQL Server с расширением SQL IaaS Agent.
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
ms.openlocfilehash: e34876c76259b8274e0b0ef9059659802eb55cf1
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107765455"
---
# <a name="register-sql-server-vm-with-sql-iaas-agent-extension"></a>Регистрация виртуальной машины SQL Server в Azure с расширением SQL IaaS Agent
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Регистрация виртуальной машины SQL Server с [расширением SQL IaaS Agent](sql-server-iaas-agent-extension-automate-management.md) позволяет открыть множество преимуществ для SQL Server на виртуальной машине Azure. 

В данной статье описывается, как зарегистрировать одиночную виртуальную машину SQL Server с расширением SQL IaaS Agent. Кроме того, можно зарегистрировать все виртуальные машины SQL Server [автоматически](sql-agent-extension-automatic-registration-all-vms.md) или [занести несколько виртуальных машин в пакетную операцию](sql-agent-extension-manually-register-vms-bulk.md).


## <a name="overview"></a>Обзор

При регистрации с [расширением SQL IaaS Agent](sql-server-iaas-agent-extension-automate-management.md) в рамках вашей подписки создается **ресурс**  _виртуальной машины SQL_, который становится _независимым_ от самой виртуальной машины ресурсом. При снятии регистрации виртуальной машины SQL Server с расширения будет удален только сам _ресурс_ **виртуальной машины SQL**, виртуальная машина фактически удалена не будет.

Развертывание образа виртуальной машины SQL Server из Azure Marketplace через портал Azure предусматривает автоматическую регистрацию виртуальной машины SQL Server с расширением. Однако, если вы выбрали самостоятельную установку SQL Server на виртуальной машине Azure или подготовили виртуальную машину Azure с пользовательского виртуального жесткого диска, то виртуальную машину SQL Server с расширением SQL IaaS Agent необходимо зарегистрировать, чтобы открыть все имеющиеся преимущества и управляемость. 

Для использования расширения SQL IaaS Agent необходимо сначала [зарегистрировать подписку с помощью поставщика **Microsoft.SqlVirtualMachine**](#register-subscription-with-resource-provider). После этого поставщик ресурсов будет иметь возможность создавать ресурсы в пределах данной конкретной подписки.

> [!IMPORTANT]
> Расширение SQL IaaS Agent собирает данные для предоставления клиентам дополнительных преимуществ по использованию SQL Server в виртуальных машинах Azure. Корпорация Майкрософт не намерена использовать эти данные для аудита лицензирования без согласия клиента. Дополнительные сведения см. в разделе [Приложение о конфиденциальности SQL Server](/sql/sql-server/sql-server-privacy#non-personal-data).

## <a name="prerequisites"></a>Предварительные требования

Для регистрации виртуальной машины SQL Server с расшиением понадобится: 

- [Подписка Azure](https://azure.microsoft.com/free/).
- Модель ресурсов Azure: [Виртуальная машина Windows Server 2008 (или более поздняя)](../../../virtual-machines/windows/quick-create-portal.md) с [SQL Server 2008 (или более поздней версией)](https://www.microsoft.com/sql-server/sql-server-downloads), развернутая в общедоступном облаке или в Azure для государственных организаций. 
- Последняя версия [Azure CLI](/cli/azure/install-azure-cli) или [Azure PowerShell (5.0 минимум)](/powershell/azure/install-az-ps). 


## <a name="register-subscription-with-resource-provider"></a>Регистрация подписки с помощью поставщика ресурсов

Чтобы зарегистрировать виртуальную машину SQL IaaS с расширением SQL IaaS Agent, необходимо сначала зарегистрировать подписку с помощью поставщика ресурсов **Microsoft.SqlVirtualMachine**. Это предоставляет расширению SQL IaaS Agent возможность создавать ресурсы в вашей подписке.  Данную процедуру можно проделать с помощью портала Azure, Azure CLI или Azure PowerShell.

### <a name="azure-portal"></a>Портал Azure

1. Войдите на портал Azure и откройте раздел **Все службы**. 
1. Перейдите к разделу **Подписки** и выберите нужную подписку.  
1. На странице **Подписки** перейдите к элементу **extensions** (расширения). 
1. Введите в фильтр значение **sql**, чтобы отсортировать расширения, имеющие отношение к SQL. 
1. Выберите действие **Зарегистрировать**, **Перерегистрировать** или **Отменить регистрацию** для поставщика **Microsoft.SqlVirtualMachine** в зависимости от ваших намерений. 

   ![Изменение поставщика](./media/sql-agent-extension-manually-register-single-vm/select-resource-provider-sql.png)


### <a name="command-line"></a>Командная строка

Зарегистрируйте подписку Azure в поставщике **Microsoft.SqlVirtualMachine** с помощью Azure CLI или Azure PowerShell. 

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

## <a name="register-with-extension"></a>Зарегистрировать с расширением

Существует три режима управления для [расширения SQL IaaS Agent](sql-server-iaas-agent-extension-automate-management.md#management-modes). 

Регистрация расширения в режиме полного управления перезапускает службу SQL Server, поэтому рекомендуется сначала зарегистрировать расширение в упрощенном режиме, а затем выполнить [обновление полностью](#upgrade-to-full) в период обслуживания. 

### <a name="lightweight-management-mode"></a>Режим упрощенного управления

Используйте Azure CLI или Azure PowerShell, чтобы зарегистрировать виртуальную машину SQL Server с расширением в упрощенном режиме. Это не приведет к перезапуску службы SQL Server. Затем в любое время можно выполнить обновление до полного режима, но это приведет к перезапуску службы SQL Server. Поэтому рекомендуется дождаться периода запланированного обслуживания. 

Укажите тип лицензии SQL Server: "С оплатой по мере использования" (`PAYG`), чтобы платить только за потребление, "Преимущество гибридного использования Azure" (`AHUB`), чтобы использовать собственную лицензию, или "Аварийное восстановление" (`DR`), чтобы активировать [лицензию бесплатной реплики аварийного восстановления](business-continuity-high-availability-disaster-recovery-hadr-overview.md#free-dr-replica-in-azure).

Экземпляры отказоустойчивого кластера и многоэкземплярные развертывания можно зарегистрировать только с расширением SQL IaaS Agent в упрощенном режиме. 

# <a name="azure-cli"></a>[Azure CLI](#tab/bash);

Зарегистрировать виртуальную машину SQL Server в упрощенном режиме с помощью Azure CLI: 

  ```azurecli-interactive
  # Register Enterprise or Standard self-installed VM in Lightweight mode
  az sql vm create --name <vm_name> --resource-group <resource_group_name> --location <vm_location> --license-type <license_type> 
  ```


# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

Зарегистрировать виртуальную машину SQL Server в упрощенном режиме с помощью Azure PowerShell:  


  ```powershell-interactive
  # Get the existing compute VM
  $vm = Get-AzVM -Name <vm_name> -ResourceGroupName <resource_group_name>
          
  # Register SQL VM with 'Lightweight' SQL IaaS agent
  New-AzSqlVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName -Location $vm.Location `
    -LicenseType <license_type>  -SqlManagementType LightWeight  
  ```

---

### <a name="full-management-mode"></a>Режим полного управления

Регистрация в полном режиме приведет к перезапуску службы SQL Server. Выполняйте дальнейшие действия с осторожностью. 

Чтобы зарегистрировать виртуальную машину SQL Server сразу в полном режиме (и, возможно, перезапустить службу SQL Server), используйте следующую команду Azure PowerShell: 

  ```powershell-interactive
  # Get the existing  Compute VM
  $vm = Get-AzVM -Name <vm_name> -ResourceGroupName <resource_group_name>
        
  # Register with SQL IaaS Agent extension in full mode
  New-AzSqlVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName -SqlManagementType Full
  ```

### <a name="noagent-management-mode"></a>Режим управления NoAgent 

SQL Server 2008 и 2008 R2, установленные в Windows Server 2008 (_не R2_), можно зарегистрировать с расширением SQL IaaS Agent в [режиме NoAgent](sql-server-iaas-agent-extension-automate-management.md#management-modes). Этот вариант обеспечивает соответствие и позволяет отслеживать виртуальную машину SQL Server на портале Azure с ограниченными функциональными возможностями.


В **типе лицензии**: укажите один из следующих типов: `AHUB`, `PAYG` или `DR`. В **предложении образа**: укажите либо `SQL2008-WS2008`, либо `SQL2008R2-WS2008`

Чтобы зарегистрировать SQL Server 2008 (`SQL2008-WS2008`) или 2008 R2 (`SQL2008R2-WS2008`) в экземпляре Windows Server 2008, используйте следующий фрагмент кода Azure CLI или Azure PowerShell: 


# <a name="azure-cli"></a>[Azure CLI](#tab/bash);

Зарегистрировать виртуальную машину SQL Server в режиме NoAgent с помощью Azure CLI: 

  ```azurecli-interactive
   az sql vm create -n sqlvm -g myresourcegroup -l eastus |
   --license-type <license type>  --sql-mgmt-type NoAgent 
   --image-sku Enterprise --image-offer <image offer> 
 ```
 

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)


Зарегистрировать виртуальную машину SQL Server в режиме NoAgent с помощью Azure PowerShell: 

  ```powershell-interactive
  # Get the existing compute VM
  $vm = Get-AzVM -Name <vm_name> -ResourceGroupName <resource_group_name>
          
  New-AzSqlVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName -Location $vm.Location `
    -LicenseType <license type> -SqlManagementType NoAgent -Sku Standard -Offer <image offer>
  ```

---

## <a name="verify-mode"></a>Режим проверки

Текущий режим SQL IaaS Agent можно просмотреть с помощью Azure PowerShell: 

```powershell-interactive
# Get the SqlVirtualMachine
$sqlvm = Get-AzSqlVM -Name $vm.Name  -ResourceGroupName $vm.ResourceGroupName
$sqlvm.SqlManagementType
```

## <a name="upgrade-to-full"></a>Обновление до полного режима  

Виртуальные машины SQL Server, которые зарегистрировали расширение в *упрощенном* режиме, могут обновить режим до _полного_ с помощью портала Azure, Azure CLI или Azure PowerShell. Виртуальные машины SQL Server в режиме _NoAgent_ можно обновить до _полного_ режима после обновления операционной системы до Windows 2008 R2 или выше. Переход на использование более ранней версии невозможен. Чтобы сделать это, необходимо [снять регистрацию](#unregister-from-extension) виртуальной машины SQL Server с расширения SQL IaaS Agent. При этом будет удален _ресурс_ **виртуальной машины SQL**, но не фактическая виртуальная машина. 

> [!NOTE]
> При полном обновлении режима управления для расширения SQL IaaS будет перезапущена служба SQL Server. В некоторых случаях перезапуск может привести к тому, что имена субъектов-служб (SPN), связанные со службой SQL Server, будут переключены не на ту учетную запись пользователя. При наличии проблем с подключением после обновления режима управления до полного [отмените регистрацию и повторно зарегистрируйте имена участников-служб](/sql/database-engine/configure-windows/register-a-service-principal-name-for-kerberos-connections).


### <a name="azure-portal"></a>портал Azure;

Чтобы обновить расширение до полного режима с помощью портала Azure, выполните следующие действия. 

1. Войдите на [портал Azure](https://portal.azure.com).
1. Перейдите к ресурсу [виртуальных машин SQL](manage-sql-vm-portal.md#access-the-sql-virtual-machines-resource). 
1. Выберите необходимую вам виртуальную машину SQL Server и щелкните **Обзор**. 
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
Вы можете проверить, зарегистрирована ли ваша виртуальная машина SQL Server с расширением SQL IaaS Agent, используя портал Azure, Azure CLI или Azure PowerShell. 

### <a name="azure-portal"></a>портал Azure; 

Чтобы проверить состояние регистрации с помощью портал Azure, выполните следующие действия: 

1. Войдите на [портал Azure](https://portal.azure.com). 
1. Перейдите на ваши виртуальные машины [SQL Server](manage-sql-vm-portal.md).
1. Выберите свою виртуальную машину SQL Server из списка. Если вашей виртуальной машины SQL Server в списке нет, скорее всего, она не была зарегистрирована с расширением SQL IaaS Agent. 
1. Просмотрите значение в разделе **Состояние**. Если **Состояние** имеет значение **Успешно**, то виртуальная машина SQL Server с расширением SQL IaaS Agent зарегистрирована успешно. 

   ![Проверка состояния при регистрации с помощью поставщика ресурсов SQL](./media/sql-agent-extension-manually-register-single-vm/verify-registration-status.png)

### <a name="command-line"></a>Командная строка

Проверьте текущее состояние регистрации виртуальной машины SQL Server с помощью Azure CLI или Azure PowerShell. Параметр `ProvisioningState` отображает состояние `Succeeded`, если регистрация прошла успешно. 

# <a name="azure-cli"></a>[Azure CLI](#tab/bash);

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

Ошибка указывает на то, что виртуальная машина SQL Server не была зарегистрирована с расширением. 


## <a name="unregister-from-extension"></a>Снять регистрацию с расширения

Чтобы снять регистрацию виртуальной машины SQL Server с расширения SQL IaaS Agent, удалите *ресурс* виртуальной машины SQL, используя портал Azure или Azure CLI. При удалении *ресурса* виртуальной машины SQL виртуальная машина SQL Server не удаляется. Тем не менее будьте внимательны и осторожны при выполнении этих действий, так как при попытке удалить *ресурс* можно случайно удалить виртуальную машину. 

Снятие регистрации виртуальной машины SQL с расширения SQL IaaS Agent необходима для того, чтобы перейти с полного режима управления на другой. 

### <a name="azure-portal"></a>портал Azure;

Чтобы снять регистрацию виртуальной машины SQL Server с расширения на портале Azure, выполните следующие действия.

1. Войдите на [портал Azure](https://portal.azure.com).
1. Перейдите к ресурсу виртуальной машины SQL. 
  
   ![Ресурс "Виртуальные машины SQL"](./media/sql-agent-extension-manually-register-single-vm/sql-vm-manage.png)

1. Выберите команду **Удалить**. 

   ![Щелкните "Удалить" в меню навигации сверху](./media/sql-agent-extension-manually-register-single-vm/delete-sql-vm-resource.png)

1. Введите имя виртуальной машины SQL и **снимите флажок рядом с этой виртуальной машиной**.

   ![Флажок виртуальной машины необходимо снять, чтобы предотвратить удаление реальной виртуальной машины. После этого нажмите кнопку "Удалить", чтобы продолжить удаление ресурса виртуальной машины SQL](./media/sql-agent-extension-manually-register-single-vm/confirm-delete-of-resource-uncheck-box.png)

   >[!WARNING]
   > Если не снять флажок рядом с именем виртуальной машины, то она будет *удалена* полностью. Снимите флажок, чтобы снять регистрацию виртуальной машины SQL Server с расширения и *не удалять виртуальную машину фактически*. 

1. Выберите **Удалить**, чтобы подтвердить удаление *ресурса* виртуальной машины SQL, а не самой виртуальной машины SQL Server. 

### <a name="command-line"></a>Командная строка

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы снять регистрацию виртуальной машины SQL Server с расширения с помощью Azure CLI, выполните команду [az sql vm delete](/cli/azure/sql/vm#az_sql_vm_delete). Удалится *ресурс* виртуальной машины SQL Server, но не сама виртуальная машина. 


```azurecli-interactive
az sql vm delete 
  --name <SQL VM resource name> |
  --resource-group <Resource group name> |
  --yes 
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Чтобы снять регистрацию SQL Server виртуальной машины с расширения с помощью Azure PowerShell, используйте команду [Remove-AzSqlVM](/powershell/module/az.sqlvirtualmachine/remove-azsqlvm). Удалится *ресурс* виртуальной машины SQL Server, но не сама виртуальная машина. 

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
