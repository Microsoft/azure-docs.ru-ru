---
title: Регистрация нескольких виртуальных машин SQL в Azure с помощью расширения агента IaaS SQL
description: Для повышения управляемости можно выполнить сбор SQL Server виртуальных машин с расширением агента IaaS SQL.
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
ms.openlocfilehash: 558daede55f6563155d3f54e97d77c0a3ca4de59
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97357234"
---
# <a name="register-multiple-sql-vms-in-azure-with-the-sql-iaas-agent-extension"></a>Регистрация нескольких виртуальных машин SQL в Azure с помощью расширения агента IaaS SQL
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

В этой статье описывается, как зарегистрировать SQL Server виртуальные машины (ВМ) в Azure с [расширением агента IaaS SQL](sql-server-iaas-agent-extension-automate-management.md) с помощью `Register-SqlVMs` командлета Azure PowerShell. 


В этой статье содержатся инструкции по регистрации SQL Server виртуальных машин вручную. Кроме того, можно вручную зарегистрировать [все SQL Server виртуальные машины](sql-agent-extension-automatic-registration-all-vms.md) или [отдельные SQL Server виртуальные машины](sql-agent-extension-manually-register-single-vm.md). 

## <a name="overview"></a>Обзор

Командлет `Register-SqlVMs` можно использовать для регистрации всех виртуальных машин в заданном списке подписок, групп ресурсов или в списке конкретных виртуальных машин. Командлет зарегистрирует виртуальные машины в [режиме управления lightweight_](sql-server-iaas-agent-extension-automate-management.md#management-modes), а затем создаст [отчет и файл журнала](#output-description). 

Процесс регистрации не несет риска, не имеет простоев и не перезапускает службу SQL Server или виртуальную машину. 

## <a name="prerequisites"></a>Предварительные условия

Чтобы зарегистрировать виртуальную машину SQL Server с расширением, вам потребуется следующее: 

- [Подписка Azure](https://azure.microsoft.com/free/) , [зарегистрированная в поставщике **Microsoft. склвиртуалмачине**](sql-agent-extension-manually-register-single-vm.md#register-subscription-with-rp) и содержащая незарегистрированные SQL Server виртуальные машины. 
- Учетные данные клиента, используемые для регистрации виртуальных машин, существуют в любой из следующих ролей Azure: **участник виртуальной машины**, **участник** или **владелец**. 
- Последняя версия [AZ PowerShell (5,0 минимум)](/powershell/azure/new-azureps-module-az). 


## <a name="get-started"></a>Начало работы

Прежде чем продолжить, необходимо создать локальную копию скрипта, импортировать ее в качестве модуля PowerShell и подключиться к Azure. 

### <a name="create-the-script"></a>Создание скрипта

Чтобы создать скрипт, скопируйте [полный скрипт](#full-script) в конце этой статьи и сохраните его локально как `RegisterSqlVMs.psm1`. 

### <a name="import-the-script"></a>Импорт скрипта

После создания скрипта его можно импортировать как модуль в терминал PowerShell. 

Откройте административный терминал PowerShell и перейдите к сохраненному файлу `RegisterSqlVMs.psm1`. Чтобы импортировать скрипт как модуль, выполните следующий командлет PowerShell: 

```powershell-interactive
Import-Module .\RegisterSqlVMs.psm1
```

### <a name="connect-to-azure"></a>Подключение к Azure

Для подключения к Azure используйте следующий командлет PowerShell:

```powershell-interactive
Connect-AzAccount
```


## <a name="all-vms-in-a-list-of-subscriptions"></a>Все виртуальные машины в списке подписок 

Используйте следующий командлет, чтобы зарегистрировать все виртуальные машины SQL Server в списке подписок:

```powershell-interactive
Register-SqlVMs -SubscriptionList SubscriptionId1,SubscriptionId2
```

Выходные данные примера: 

```
Number of subscriptions registration failed for 
because you do not have access or credentials are wrong: 1
Total VMs Found: 10
VMs Already registered: 1
Number of VMs registered successfully: 4
Number of VMs failed to register due to error: 1
Number of VMs skipped as VM or the guest agent on VM is not running: 3
Number of VMs skipped as they are not running SQL Server On Windows: 1

Please find the detailed report in file RegisterSqlVMScriptReport1571314821.txt
Please find the error details in file VMsNotRegisteredDueToError1571314821.log
```

## <a name="all-vms-in-a-single-subscription"></a>Все виртуальные машины в одной подписке

Используйте следующий командлет, чтобы зарегистрировать все виртуальные машины SQL Server в одной подписке: 

```powershell-interactive
Register-SqlVMs -Subscription SubscriptionId1
```

Выходные данные примера:

```
Total VMs Found: 10
VMs Already registered: 1
Number of VMs registered successfully: 5
Number of VMs failed to register due to error: 1
Number of VMs skipped as VM or the  guest agent on VM is not running: 2
Number of VMs skipped as they are not running SQL Server On Windows: 1

Please find the detailed report in file RegisterSqlVMScriptReport1571314821.txt
Please find the error details in file VMsNotRegisteredDueToError1571314821.log
```

## <a name="all-vms-in-multiple-resource-groups"></a>Все виртуальные машины в нескольких группах ресурсов

Используйте следующий командлет, чтобы зарегистрировать все виртуальные машины SQL Server в нескольких группах ресурсов в пределах одной подписки:

```powershell-interactive
Register-SqlVMs -Subscription SubscriptionId1 -ResourceGroupList ResourceGroup1,ResourceGroup2
```

Выходные данные примера:

```
Total VMs Found: 4
VMs Already registered: 1
Number of VMs registered successfully: 1
Number of VMs failed to register due to error: 1
Number of VMs skipped as they are not running SQL Server On Windows: 1

Please find the detailed report in file RegisterSqlVMScriptReport1571314821.txt
Please find the error details in file VMsNotRegisteredDueToError1571314821.log
```

## <a name="all-vms-in-a-resource-group"></a>Все виртуальные машины в группе ресурсов

Используйте следующий командлет, чтобы зарегистрировать все виртуальные машины SQL Server в одной группе ресурсов: 

```powershell-interactive
Register-SqlVMs -Subscription SubscriptionId1 -ResourceGroupName ResourceGroup1
```

Выходные данные примера:

```
Total VMs Found: 4
VMs Already registered: 1
Number of VMs registered successfully: 1
Number of VMs failed to register due to error: 1
Number of VMs skipped as VM or the guest agent on VM is not running: 1

Please find the detailed report in file RegisterSqlVMScriptReport1571314821.txt
Please find the error details in file VMsNotRegisteredDueToError1571314821.log
```

## <a name="specific-vms-in-a-single-resource-group"></a>Определенные виртуальные машины в одной группе ресурсов

Используйте следующий командлет, чтобы зарегистрировать конкретные виртуальные машины SQL Server в одной группе ресурсов:

```powershell-interactive
Register-SqlVMs -Subscription SubscriptionId1 -ResourceGroupName ResourceGroup1 -VmList VM1,VM2,VM3
```

Выходные данные примера:

```
Total VMs Found: 3
VMs Already registered: 0
Number of VMs registered successfully: 1
Number of VMs skipped as VM or the guest agent on VM is not running: 1
Number of VMs skipped as they are not running SQL Server On Windows: 1

Please find the detailed report in file RegisterSqlVMScriptReport1571314821.txt
Please find the error details in file VMsNotRegisteredDueToError1571314821.log
```

## <a name="a-specific-vm"></a>Определенная виртуальная машина

Используйте следующий командлет, чтобы зарегистрировать конкретные виртуальные машины SQL Server: 

```powershell-interactive
Register-SqlVMs -Subscription SubscriptionId1 -ResourceGroupName ResourceGroup1 -Name VM1
```

Выходные данные примера:

```
Total VMs Found: 1
VMs Already registered: 0
Number of VMs registered successfully: 1

Please find the detailed report in  file RegisterSqlVMScriptReport1571314821.txt
```


## <a name="output-description"></a>Описание выходных данных

Отчет и файл журнала создаются каждый раз при использовании командлета `Register-SqlVMs`. 

### <a name="report"></a>Report

Отчет создается как файл `.txt` с именем `RegisterSqlVMScriptReport<Timestamp>.txt`, в котором время начала выполнения командлета используется в качестве метки времени. В отчете перечислены следующие сведения:

| **Выходное значение** | **Описание** |
| :--------------  | :-------------- | 
| Количество подписок, которые не удалось зарегистрировать, так как у вас нет доступа или учетные данные неверны | Предоставляет число и список подписок, в которых возникли проблемы с указанной проверкой подлинности. Подробные сведения об ошибке можно найти в журнале, выполнив поиск по идентификатору подписки. | 
| Число подписок, которые не удалось попытаться выполнить, так как они не зарегистрированы в поставщике ресурсов | В этом разделе содержится количество и список подписок, которые не были зарегистрированы в расширении агента IaaS SQL. |
| Общее число найденных виртуальных машин | Число виртуальных машин, обнаруженных в области параметров, переданных в командлет. | 
| Количество уже зарегистрированных виртуальных машин | Число виртуальных машин, которые были пропущены, так как они уже зарегистрированы в расширении. |
| Количество успешно зарегистрированных виртуальных машин | Количество виртуальных машин, успешно зарегистрированных после выполнения командлета. Список зарегистрированных виртуальных машин в формате `SubscriptionID, Resource Group, Virtual Machine`. | 
| Количество виртуальных машин, которые не удалось зарегистрировать из-за ошибки | Количество виртуальных машин, которые не удалось зарегистрировать из-за ошибки. Подробные сведения об ошибке можно найти в файле журнала. | 
| Количество виртуальных машин, пропущенных из-за того, что не запущена виртуальная машина или расположенный на ней гостевой агент | Количество и список виртуальных машин, которые не удалось зарегистрировать, так как не запущена виртуальная машина или расположенный на ней гостевой агент. Регистрацию можно повторить после запуска виртуальной машины или гостевого агента. Дополнительные сведения см. в файле журнала. |
| Количество пропущенных виртуальных машин, на которых не запущен SQL Server в Windows | Количество виртуальных машин, которые были пропущены, так как на них не запущен SQL Server или они не являются виртуальными машинами Windows. Виртуальные машины перечислены в формате `SubscriptionID, Resource Group, Virtual Machine`. | 
| &nbsp; | &nbsp; |

### <a name="log"></a>Журнал 

Ошибки записываются в файл журнала с именем `VMsNotRegisteredDueToError<Timestamp>.log`, в котором время запуска скрипта используется в качестве метки времени. Если ошибка возникла на уровне подписки, в журнале содержится идентификатор подписки и сообщение об ошибке (с разделителями-запятыми). Если ошибка связана с регистрацией виртуальной машины, в журнале содержится идентификатор подписки, имя группы ресурсов, имя виртуальной машины, код ошибки и сообщение, разделяемые запятыми. 

## <a name="remarks"></a>Remarks

При регистрации SQL Server виртуальных машин с расширением с помощью предоставленного скрипта учитывайте следующее.

- Для регистрации с помощью расширения требуется Гостевой агент, работающий на виртуальной машине SQL Server. Образы Windows Server 2008 не имеют гостевого агента, поэтому эти виртуальные машины завершаются сбоем и должны быть зарегистрированы вручную с помощью [режима управления NoAgent](sql-server-iaas-agent-extension-automate-management.md#management-modes).
- Существует встроенная логика повторных попыток для устранения прозрачных ошибок. Если виртуальная машина успешно зарегистрирована, эта операция выполняется быстро. Однако в случае сбоя регистрации каждая виртуальная машина будет выполнена повторно.  Таким образом, для завершения процесса регистрации необходимо значительное время, хотя требования к фактическому времени зависят от типа и количества ошибок. 

## <a name="full-script"></a>Полный сценарий

Полный сценарий на сайте GitHub см. в статье [Пакетная регистрация SQL Server виртуальных машин с помощью команды AZ PowerShell](https://github.com/Azure/azure-docs-powershell-samples/blob/master/sql-virtual-machine/register-sql-vms/RegisterSqlVMs.psm1). 

Скопируйте полный скрипт и сохраните его как файл `RegisterSqLVMs.psm1`.

[!code-powershell-interactive[main](../../../../powershell_scripts/sql-virtual-machine/register-sql-vms/RegisterSqlVMs.psm1 "Bulk register SQL Server virtual machines")]

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в следующих статьях: 

* [Обзор SQL Server на виртуальных машинах Windows](sql-server-on-azure-vm-iaas-what-is-overview.md).
* [Вопросы и ответы об SQL Server на виртуальных машинах Windows](frequently-asked-questions-faq.md)
* [Руководство по ценам для виртуальных машин SQL Server в Azure](pricing-guidance.md)
* [Заметки о выпуске SQL Server на виртуальных машинах Windows](../../database/doc-changes-updates-release-notes.md)
