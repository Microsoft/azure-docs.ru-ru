---
title: Шаблоны Azure Resource Manager
description: Шаблоны Azure Resource Manager для хранилищ Служб восстановления и функций Azure Backup.
ms.topic: sample
ms.date: 01/31/2019
ms.custom: mvc
ms.openlocfilehash: a4c2f444cb821f7979571b9d777895a59450e7c2
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96309585"
---
# <a name="azure-resource-manager-templates-for-azure-backup"></a>Шаблоны Azure Resource Manager для Azure Backup

В приведенной ниже таблице представлены ссылки на шаблоны Azure Resource Manager для хранилищ служб восстановления и функций Azure Backup. Сведения о синтаксисе и свойствах JSON см. в статье о [типах ресурсов Microsoft.RecoveryServices](/azure/templates/microsoft.recoveryservices/allversions).

| Шаблон | Описание |
|---|---|
|**Хранилище служб восстановления** | |
| [создание хранилища служб восстановления](https://github.com/Azure/azure-quickstart-templates/tree/master/101-recovery-services-vault-create);| Создайте хранилище служб восстановления, Хранилище можно использовать для Azure Backup и Azure Site Recovery. |
|**Настройка виртуальных машин**| |
| [Backup Resource Manager VMs to Recovery Services Vault](https://github.com/Azure/azure-quickstart-templates/tree/master/101-recovery-services-backup-vms) (Резервное копирование виртуальных машин диспетчера ресурсов в хранилище служб восстановления) | Используйте имеющееся хранилище служб восстановления и политику Backup для резервного копирования виртуальных машин диспетчера ресурсов в той же группе ресурсов.|
| [Backup ARM and Classic IaaS VMs to Recovery Services Vault](https://github.com/Azure/azure-quickstart-templates/tree/master/201-recovery-services-backup-classic-resource-manager-vms) (Резервное копирование виртуальных машин ARM и классических виртуальных машин IaaS в хранилище служб восстановления) | Шаблон для резервного копирования классических виртуальных машин и виртуальных машин диспетчера ресурсов. |
| [Create Weekly Backup Policy for Recovery Services Vault to protect Azure IaaS VMs](https://github.com/Azure/azure-quickstart-templates/tree/master/101-recovery-services-weekly-backup-policy-create) (Создание политики еженедельного резервного копирования для хранилища служб восстановления для защиты виртуальных машин Azure IaaS) | Шаблон создает хранилище Служб восстановления и политику еженедельного резервного копирования, которая используется для резервного копирования классических виртуальных машин и виртуальных машин, развернутых с помощью Resource Manager.|
| [Create Daily Backup Policy for Recovery Services Vault to protect Azure IaaS VMs](https://github.com/Azure/azure-quickstart-templates/tree/master/101-recovery-services-daily-backup-policy-create) (Создание политики ежедневного резервного копирования для хранилища служб восстановления для защиты виртуальных машин Azure IaaS) | Шаблон создает хранилище Служб восстановления и политику ежедневного резервного копирования, которая используется для резервного копирования классических виртуальных машин и виртуальных машин, развернутых с помощью Resource Manager.|
| [Create a simple windows VM and configure backup](https://github.com/Azure/azure-quickstart-templates/tree/master/101-recovery-services-create-vm-and-configure-backup) (Создание простой виртуальной машины Windows и настройка резервного копирования) | Шаблон создает хранилище виртуальных машин и хранилищ Windows Server с включенной политикой резервного копирования по умолчанию.|
|**Мониторинг заданий службы архивации** |  |
| [Использование журналов Azure Monitor с Azure Backup](https://github.com/Azure/azure-quickstart-templates/tree/master/101-backup-oms-monitoring) | Шаблон развертывает журналы Azure Monitor с Azure Backup, что позволяет отслеживать задания резервного копирования и восстановления, оповещения о резервном копировании и облачное хранилище, используемое в хранилищах Служб восстановления.|  
|**Резервное копирование SQL Server на виртуальной машине Azure** |  |
| [Резервное копирование SQL Server на виртуальной машине Azure](https://github.com/Azure/azure-quickstart-templates/tree/master/101-recovery-services-vm-workload-backup) | Шаблон создает хранилище Служб восстановления и политику резервного копирования для определенной рабочей нагрузки. Он регистрирует виртуальную машину с помощью службы Azure Backup и настраивает защиту для этой виртуальной машины. Сейчас это работает только для образов из коллекции SQL. |
|**Резервное копирование файловых ресурсов Azure** |  |
| [Резервное копирование файловых ресурсов Azure](https://github.com/Azure/azure-quickstart-templates/tree/master/101-recovery-services-backup-file-share) | С помощью этого шаблона можно настроить защиту для существующей общей папки Azure, указав соответствующие сведения для хранилища Служб восстановления и политику резервного копирования. При необходимости шаблон создает новое хранилище Служб восстановления и политику резервного копирования, а также регистрирует учетную запись хранения, содержащую общую папку, в хранилище Служб восстановления. |
|   |   |
