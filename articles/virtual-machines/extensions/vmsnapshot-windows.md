---
title: Расширение Windows "Моментальный снимок виртуальной машины" для службы Azure Backup
description: С помощью расширения моментального снимка виртуальной машины вы можете создать согласованную с приложением резервную копию виртуальной машины из службы Azure Backup.
services: backup, virtual-machines
documentationcenter: ''
author: trinadhkotturu
manager: gwallace
ms.service: virtual-machines
ms.subservice: extensions
ms.collection: windows
ms.topic: article
ms.date: 10/15/2020
ms.author: trinadhk
ms.openlocfilehash: 834c886c15e50870ec008dbb5ceab38132cc4649
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102561077"
---
# <a name="vm-snapshot-windows-extension-for-azure-backup"></a>Расширение Windows "Моментальный снимок виртуальной машины" для службы Azure Backup

Служба Azure Backup поддерживает резервное копирование рабочих нагрузок из локальной среды в облако и архивацию облачных ресурсов в хранилище служб восстановления. Служба Azure Backup использует расширение моментального снимка виртуальной машины для создания согласованной с приложением резервной копии виртуальной машины Azure без завершения работы виртуальной машины. Корпорация Майкрософт публикует и поддерживает расширение моментального снимка виртуальной машины в составе службы Azure Backup. Служба Azure Backup установит расширение в рамках первого запланированного задания резервного копирования, запущенного после включения резервного копирования. В этом документе подробно описаны поддерживаемые платформы, конфигурации и параметры развертывания для расширения моментального снимка виртуальной машины.

Расширение VMSnapshot отображается в портал Azure только для виртуальных машин без управления.

## <a name="prerequisites"></a>Предварительные требования

### <a name="operating-system"></a>Операционная система
Список поддерживаемых операционных систем см. в разделе об [операционных системах, поддерживаемых службой Azure Backup](../../backup/backup-azure-arm-vms-prepare.md#before-you-start).

## <a name="extension-schema"></a>Схема расширения

В следующем JSON-файле показана схема для расширения моментального снимка виртуальной машины. Для расширения требуется идентификатор задачи — он идентифицирует задание резервного копирования, которое инициировало создание моментального снимка на виртуальной машине, URI BLOB-объекта состояния, в который записывается состояние операции создания моментального снимка, запланированное время начала создания моментального снимка, URI BLOB-объекта журналов, в который записываются журналы, соответствующие задаче моментального снимка, objstr-представление дисков виртуальных машин и метаданных.  Так как эти параметры должны рассматриваться в качестве конфиденциальных данных, их следует хранить в защищенной конфигурации параметров. Данные защищенных параметров расширения виртуальной машины Azure зашифрованы. Они расшифровываются только на целевой виртуальной машине. Обратите внимание, что эти параметры рекомендуется передавать из службы Azure Backup только в рамках задания резервного копирования.

```json
{
  "type": "extensions",
  "name": "VMSnapshot",
  "location":"<myLocation>",
  "properties": {
    "publisher": "Microsoft.Azure.RecoveryServices",
    "type": "VMSnapshot",
    "typeHandlerVersion": "1.9",
    "autoUpgradeMinorVersion": true,
    "settings": {
      "locale":"<location>",
      "taskId":"<taskId used by Azure Backup service to communicate with extension>",
      "commandToExecute": "snapshot",
      "commandStartTimeUTCTicks": "<scheduled start time of the snapshot task>",
      "vmType": "microsoft.compute/virtualmachines"
    },
    "protectedSettings": {
      "objectStr": "<blob SAS uri representation of VM sent by Azure Backup service to extension>",
      "logsBlobUri": "<blob uri where logs of command execution by extension are written to>",
      "statusBlobUri": "<blob uri where status of the command executed by extension is written>"
    }
  }
}
```

### <a name="property-values"></a>Значения свойств

| Имя | Значение и пример | Тип данных |
| ---- | ---- | ---- |
| версия_API | 2015-06-15 | Дата |
| taskId | e07354cf-041e-4370-929f-25a319ce8933_1 | строка |
| commandStartTimeUTCTicks | 6.36458E+17 | строка |
| локаль | ru-ru | строка |
| objectStr | Encoding of sas uri array- "blobSASUri": ["https:\/\/sopattna5365.blob.core.windows.net\/vhds\/vmwin1404ltsc201652903941.vhd?sv=2014-02-14&sr=b&sig=TywkROXL1zvhXcLujtCut8g3jTpgbE6JpSWRLZxAdtA%3D&st=2017-11-09T14%3A23%3A28Z&se=2017-11-09T17%3A38%3A28Z&sp=rw", "https:\/\/sopattna8461.blob.core.windows.net\/vhds\/vmwin1404ltsc-20160629-122418.vhd?sv=2014-02-14&sr=b&sig=5S0A6YDWvVwqPAkzWXVy%2BS%2FqMwzFMbamT5upwx05v8Q%3D&st=2017-11-09T14%3A23%3A28Z&se=2017-11-09T17%3A38%3A28Z&sp=rw", "https:\/\/sopattna8461.blob.core.windows.net\/bootdiagnostics-vmwintu1-deb58392-ed5e-48be-9228-ff681b0cd3ee\/vmubuntu1404ltsc-20160629-122541.vhd?sv=2014-02-14&sr=b&sig=X0Me2djByksBBMVXMGIUrcycvhQSfjYvqKLeRA7nBD4%3D&st=2017-11-09T14%3A23%3A28Z&se=2017-11-09T17%3A38%3A28Z&sp=rw", "https:\/\/sopattna5365.blob.core.windows.net\/vhds\/vmwin1404ltsc-20160701-163922.vhd?sv=2014-02-14&sr=b&sig=oXvtK2IXCNqWv7fpjc7TAzFDpc1GoXtT7r%2BC%2BNIAork%3D&st=2017-11-09T14%3A23%3A28Z&se=2017-11-09T17%3A38%3A28Z&sp=rw", "https:\/\/sopattna5365.blob.core.windows.net\/vhds\/vmwin1404ltsc-20170705-124311.vhd?sv=2014-02-14&sr=b&sig=ZUM9d28Mvvm%2FfrhJ71TFZh0Ni90m38bBs3zMl%2FQ9rs0%3D&st=2017-11-09T14%3A23%3A28Z&se=2017-11-09T17%3A38%3A28Z&sp=rw"] | строка |
| logsBlobUri | https://seapod01coord1exsapk732.blob.core.windows.net/bcdrextensionlogs-d45d8a1c-281e-4bc8-9d30-3b25176f68ea/sopattna-vmubuntu1404ltsc.v2.Logs.txt?sv=2014-02-14&sr=b&sig=DbwYhwfeAC5YJzISgxoKk%2FEWQq2AO1vS1E0rDW%2FlsBw%3D&st=2017-11-09T14%3A33%3A29Z&se=2017-11-09T17%3A38%3A29Z&sp=rw | строка |
| statusBlobUri | https://seapod01coord1exsapk732.blob.core.windows.net/bcdrextensionlogs-d45d8a1c-281e-4bc8-9d30-3b25176f68ea/sopattna-vmubuntu1404ltsc.v2.Status.txt?sv=2014-02-14&sr=b&sig=96RZBpTKCjmV7QFeXm5IduB%2FILktwGbLwbWg6Ih96Ao%3D&st=2017-11-09T14%3A33%3A29Z&se=2017-11-09T17%3A38%3A29Z&sp=rw | строка |



## <a name="template-deployment"></a>Развертывание шаблона

Расширения виртуальной машины Azure можно развернуть с помощью шаблонов Azure Resource Manager. Однако добавлять расширение моментального снимка виртуальной машины рекомендуется путем включения резервного копирования на виртуальной машине. Это можно сделать с помощью шаблона Resource Manager.  Пример шаблона Resource Manager, включающего резервное копирование на виртуальной машине, можно найти в [коллекции быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-recovery-services-backup-vms/).


## <a name="azure-cli-deployment"></a>Развертывание с помощью Azure CLI

Включить резервное копирование на виртуальной машине можно с помощью Azure CLI. После включения первое запланированное задание резервного копирования установит на виртуальной машине расширение моментального снимка виртуальной машины.

```azurecli
az backup protection enable-for-vm \
    --resource-group myResourceGroup \
    --vault-name myRecoveryServicesVault \
    --vm myVM \
    --policy-name DefaultPolicy
```

## <a name="troubleshoot-and-support"></a>Устранение неполадок и поддержка

### <a name="troubleshoot"></a>Диагностика

Данные о состоянии развертывания расширения можно получить на портале Azure, а также использовав Azure CLI. Чтобы просмотреть состояние развертывания расширений для определенной виртуальной машины, выполните следующую команду в Azure CLI.

```azurecli
az vm extension list --resource-group myResourceGroup --vm-name myVM -o table
```

Выходные данные выполнения расширения регистрируются в следующем файле:

```
C:\Packages\Plugins\Microsoft.Azure.RecoveryServices.VMSnapshot
```

### <a name="error-codes-and-their-meanings"></a>Коды ошибок и их описание

Дополнительные сведения об устранении неполадок см. в [руководстве по устранению неполадок при резервном копировании ВМ Azure](../../backup/backup-azure-vms-troubleshoot.md).

### <a name="support"></a>Поддержка

Если в любой момент при изучении этой статьи вам потребуется дополнительная помощь, вы можете обратиться к экспертам по Azure на [форумах MSDN Azure и Stack Overflow](https://azure.microsoft.com/support/forums/). Кроме того, можно зарегистрировать обращение в службу поддержки Azure. Перейдите на [сайт поддержки Azure](https://azure.microsoft.com/support/options/) и щелкните "Получить поддержку". Дополнительные сведения об использовании службы поддержки Azure см. в статье [Часто задаваемые вопросы о поддержке Microsoft Azure](https://azure.microsoft.com/support/faq/).
