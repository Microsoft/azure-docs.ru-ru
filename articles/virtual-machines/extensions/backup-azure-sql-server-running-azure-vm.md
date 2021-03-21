---
title: Azure Backup для SQL Server, работающих на виртуальной машине Azure
description: В этой статье вы узнаете, как зарегистрировать Azure Backup в SQL Server, работающем на виртуальной машине Azure.
ms.topic: article
ms.service: virtual-machines
ms.subservice: extensions
author: dcurwin
ms.author: dacurwin
ms.collection: windows
ms.date: 07/05/2019
ms.openlocfilehash: 46cc4b23ca7dd70da3f917a3399406480dc9fdbd
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102557103"
---
# <a name="azure-backup-for-sql-server-running-in-azure-vm"></a>Azure Backup для SQL Server, работающих на виртуальной машине Azure

Azure Backup, среди прочих предложений, обеспечивает поддержку резервного копирования рабочих нагрузок, таких как SQL Server, выполняемых на виртуальных машинах Azure. Так как приложение SQL выполняется на виртуальной машине Azure, службе архивации требуется разрешение на доступ к приложению и получение необходимых сведений.
Для этого Azure Backup устанавливает расширение **азуребаккупвиндовсворклоад** на виртуальной машине, в которой выполняется SQL Server, во время процесса регистрации, инициированного пользователем.

## <a name="prerequisites"></a>Предварительные условия

Список поддерживаемых сценариев см. в [матрице поддержки](../../backup/sql-support-matrix.md#scenario-support) , поддерживаемой Azure Backup.

## <a name="network-connectivity"></a>Возможность подключения к сети

Azure Backup поддерживает теги NSG, развертывая прокси-сервер или указанные диапазоны IP-адресов; Дополнительные сведения о каждом из методов см. в этой [статье](../../backup/backup-sql-server-database-azure-vms.md#establish-network-connectivity).

## <a name="extension-schema"></a>Схема расширения

Схема расширения и значения свойств — это значения конфигурации (параметры среды выполнения), которые служба передает в API CRP. Эти значения конфигурации используются во время регистрации и обновления. Расширение **азуребаккупвиндовсворклоад** также использует эту схему. Схема предварительно установлена; в поле Обжектстр можно добавить новый параметр.

  ```json
      "runtimeSettings": [{
      "handlerSettings": {
      "protectedSettingsCertThumbprint": "",
      "protectedSettings": {
      "objectStr": "",
      "logsBlobUri": "",
      "statusBlobUri": ""
      }
      },
      "publicSettings": {
      "locale": "en-us",
      "taskId": "1c0ae461-9d3b-418c-a505-bb31dfe2095d",
      "objectStr": "",
      "commandStartTimeUTCTicks": "636295005824665976",
      "vmType": "vmType"
      }
      }]
      }
  ```

В следующем JSON показана схема для расширения Ворклоадбаккуп.  

  ```json
  {
    "type": "extensions",
    "name": "WorkloadBackup",
    "location":"<myLocation>",
    "properties": {
      "publisher": "Microsoft.RecoveryServices",
      "type": "AzureBackupWindowsWorkload",
      "typeHandlerVersion": "1.1",
      "autoUpgradeMinorVersion": true,
      "settings": {
        "locale":"<location>",
        "taskId":"<TaskId used by Azure Backup service to communicate with extension>",

        "objectStr": "<The configuration passed by Azure Backup service to extension>",

        "commandStartTimeUTCTicks": "<Scheduled start time of registration or upgrade task>",
        "vmType": "<Type of VM where registration got triggered Eg. Compute or ClassicCompute>"
      },
      "protectedSettings": {
        "objectStr": "<The sensitive configuration passed by Azure Backup service to extension>",
        "logsBlobUri": "<blob uri where logs of command execution by extension are written to>",
        "statusBlobUri": "<blob uri where status of the command executed by extension is written>"
      }
    }
  }
  ```

### <a name="property-values"></a>Значения свойств

Имя | Значение/пример | Тип данных
 --- | --- | ---
локаль | ru-ru  |  строка
taskId | "1c0ae461-9d3b-418c-a505-bb31dfe2095d"  | строка
objectStr <br/> Значение publicsettings  | "eyJjb250YWluZXJQcm9wZXJ0aWVzIjp7IkNvbnRhaW5lcklEIjoiMzVjMjQxYTItOGRjNy00ZGE5LWI4NTMtMjdjYTJhNDZlM2ZkIiwiSWRNZ210Q29udGFpbmVySWQiOjM0NTY3ODg5LCJSZXNvdXJjZUlkIjoiMDU5NWIwOGEtYzI4Zi00ZmFlLWE5ODItOTkwOWMyMGVjNjVhIiwiU3Vic2NyaXB0aW9uSWQiOiJkNGEzOTliNy1iYjAyLTQ2MWMtODdmYS1jNTM5ODI3ZTgzNTQiLCJVbmlxdWVDb250 YWluZXJOYW1lIjoiODM4MDZjODUtNTQ4OS00NmNhLWEyZTctNWMzNzNhYjg3OTcyIn0sInN0YW1wTGlzdCI6W3siU2VydmljZU5hbWUiOjUsIlNlcnZpY2VTdGFtcFVybCI6Imh0dHA6XC9cL015V0xGYWJTdmMuY29tIn1dfQ = =" | строка
commandStartTimeUTCTicks | "636967192566036845"  | строка
вмтипе  | "Microsoft. COMPUTE/virtualmachines"  | строка
objectStr <br/> ProtectedSettings | "eyJjb250YWluZXJQcm9wZXJ0aWVzIjp7IkNvbnRhaW5lcklEIjoiMzVjMjQxYTItOGRjNy00ZGE5LWI4NTMtMjdjYTJhNDZlM2ZkIiwiSWRNZ210Q29udGFpbmVySWQiOjM0NTY3ODg5LCJSZXNvdXJjZUlkIjoiMDU5NWIwOGEtYzI4Zi00ZmFlLWE5ODItOTkwOWMyMGVjNjVhIiwiU3Vic2NyaXB0aW9uSWQiOiJkNGEzOTliNy1iYjAyLTQ2MWMtODdmYS1jNTM5ODI3ZTgzNTQiLCJVbmlxdWVDb250 YWluZXJOYW1lIjoiODM4MDZjODUtNTQ4OS00NmNhLWEyZTctNWMzNzNhYjg3OTcyIn0sInN0YW1wTGlzdCI6W3siU2VydmljZU5hbWUiOjUsIlNlcnZpY2VTdGFtcFVybCI6Imh0dHA6XC9cL015V0xGYWJTdmMuY29tIn1dfQ = =" | строка
logsBlobUri | <https://seapod01coord1exsapk732.blob.core.windows.net/bcdrextensionlogs-d45d8a1c-281e-4bc8-9d30-3b25176f68ea/sopattna-vmubuntu1404ltsc.v2.Logs.txt?sv=2014-02-14&sr=b&sig=DbwYhwfeAC5YJzISgxoKk%2FEWQq2AO1vS1E0rDW%2FlsBw%3D&st=2017-11-09T14%3A33%3A29Z&se=2017-11-09T17%3A38%3A29Z&sp=rw> | строка
statusBlobUri | <https://seapod01coord1exsapk732.blob.core.windows.net/bcdrextensionlogs-d45d8a1c-281e-4bc8-9d30-3b25176f68ea/sopattna-vmubuntu1404ltsc.v2.Status.txt?sv=2014-02-14&sr=b&sig=96RZBpTKCjmV7QFeXm5IduB%2FILktwGbLwbWg6Ih96Ao%3D&st=2017-11-09T14%3A33%3A29Z&se=2017-11-09T17%3A38%3A29Z&sp=rw> | строка

## <a name="template-deployment"></a>Развертывание шаблона

Мы рекомендуем добавить расширение Азуребаккупвиндовсворклоад в виртуальную машину, включив SQL Server резервное копирование на виртуальной машине. Это можно сделать с помощью [шаблона диспетчер ресурсов](https://github.com/Azure/azure-quickstart-templates/tree/master/101-recovery-services-vm-workload-backup) , предназначенного для автоматизации резервного копирования на виртуальной машине SQL Server.

## <a name="powershell-deployment"></a>Развертывание с помощью PowerShell

Необходимо зарегистрировать виртуальную машину Azure, содержащую приложение SQL, с хранилищем служб восстановления. Во время регистрации расширение Азуребаккупвиндовсворклоад устанавливается на виртуальной машине. Для регистрации виртуальной машины используйте командлет [Register-азрековерисервицесбаккупконтаинерпс](/powershell/module/az.recoveryservices/register-azrecoveryservicesbackupcontainer) .

```powershell
$myVM = Get-AzVM -ResourceGroupName <VMRG Name> -Name <VMName>
Register-AzRecoveryServicesBackupContainer -ResourceId $myVM.ID -BackupManagementType AzureWorkload -WorkloadType MSSQL -VaultId $targetVault.ID -Force
```

Команда вернет **контейнер резервного копирования** этого ресурса, и состояние будет **зарегистрировано**.

## <a name="next-steps"></a>Дальнейшие действия

- [Подробнее](../../backup/backup-sql-server-azure-troubleshoot.md) о рекомендациях по устранению неполадок при резервном копировании виртуальных машин Azure SQL Server
- [Распространенные вопросы](../../backup/faq-backup-sql-server.md) о резервном копировании SQL Server баз данных, работающих на виртуальных машинах Azure и использующих службу Azure Backup.
