---
title: Устранение ошибок расширения виртуальной машины Windows
description: Узнайте об устранении неполадок в работе расширений виртуальной машины Windows в Azure.
ms.topic: article
ms.service: virtual-machines
ms.subservice: extensions
author: amjads1
ms.author: amjads
ms.collection: windows
ms.date: 03/29/2016
ms.openlocfilehash: 8cc8a0b5ae0a83a152168b14a2c5577f8f7b48ab
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102564804"
---
# <a name="troubleshooting-azure-windows-vm-extension-failures"></a>Troubleshooting Azure Windows VM extension failures
[!INCLUDE [virtual-machines-common-extensions-troubleshoot](../../../includes/virtual-machines-common-extensions-troubleshoot.md)]

## <a name="viewing-extension-status"></a>Просмотр состояния расширения
Шаблоны Azure Resource Manager можно выполнять из Azure PowerShell. После выполнения шаблона состояние расширения можно узнать в обозревателе ресурсов Azure или с помощью программ командной строки.

Пример:

Azure PowerShell:

```azurepowershell
Get-AzVM -ResourceGroupName $RGName -Name $vmName -Status
```

Пример выходных данных:

```output
Extensions:  {
  "ExtensionType": "Microsoft.Compute.CustomScriptExtension",
  "Name": "myCustomScriptExtension",
  "SubStatuses": [
    {
      "Code": "ComponentStatus/StdOut/succeeded",
      "DisplayStatus": "Provisioning succeeded",
      "Level": "Info",
      "Message": "    Directory: C:\\temp\\n\\n\\nMode                LastWriteTime     Length Name
          \\n----                -------------     ------ ----                              \\n-a---          9/1/2015   2:03 AM         11
          test.txt                          \\n\\n",
                  "Time": null
      },
    {
      "Code": "ComponentStatus/StdErr/succeeded",
      "DisplayStatus": "Provisioning succeeded",
      "Level": "Info",
      "Message": "",
      "Time": null
    }
  ]
}
```

## <a name="troubleshooting-extension-failures"></a>Устранение неполадок расширений

### <a name="rerun-the-extension-on-the-vm"></a>Повторное выполнение расширения на виртуальной машине
При выполнении сценариев на виртуальной машине с помощью расширения пользовательских сценариев может возникать ошибка, указывающая на то, что виртуальная машина создана успешно, но сценарий не выполнен. В этом случае рекомендуется удалить соответствующее расширение и выполнить шаблон еще раз.
Примечание. В будущем эта функция будет усовершенствована, что позволит устранить необходимость в удалении расширения.

#### <a name="remove-the-extension-from-azure-powershell"></a>Удаление расширения с помощью Azure PowerShell
```azurepowershell
Remove-AzVMExtension -ResourceGroupName $RGName -VMName $vmName -Name "myCustomScriptExtension"
```

После удаления расширения шаблон можно выполнить повторно, чтобы запустить скрипты на виртуальной машине.

### <a name="trigger-a-new-goalstate-to-the-vm"></a>Активация нового Гоалстате для виртуальной машины
Возможно, вы заметили, что расширение не было выполнено или не удается выполнить из-за отсутствия "генератора сертификатов Windows Azure CRP" (этот сертификат используется для защиты транспорта защищенных параметров расширения).
Этот сертификат будет автоматически создан повторно путем перезапуска гостевого агента Windows из виртуальной машины:
- Открытие диспетчера задач
- Перейдите на вкладку "сведения".
- Определение WindowsAzureGuestAgent.exe процесса
- Щелкните правой кнопкой мыши и выберите пункт "завершить задачу". Процесс будет автоматически перезапущен


Вы также можете активировать новый Гоалстате для виртуальной машины, выполнив "повторное применение ВМ". [Повторное применение](/rest/api/compute/virtualmachines/reapply) виртуальной машины — это API, появившийся в 2020 для повторного применения состояния виртуальной машины. Мы рекомендуем сделать это за раз, когда можно допустить короткий простой виртуальной машины. Хотя повторное применение не приводит к перезагрузке виртуальной машины, а подавляющее большинство раз повторного вызова не перезапускает виртуальную машину, существует очень небольшой риск, что некоторые другие ожидающие обновления модели виртуальной машины применяются, когда повторное применение активирует новое состояние цели, а другое изменение может потребовать перезагрузки. 

Портал Azure:

На портале выберите виртуальную машину и в левой области в разделе **Поддержка и устранение неполадок** выберите **Повторное развертывание и повторное применение**, а затем нажмите кнопку **повторно применить**.


Azure PowerShell *(замените имя RG и имя виртуальной машины значениями)*:

```azurepowershell
Set-AzVM -ResourceGroupName <RG Name> -Name <VM Name> -Reapply
```

Azure CLI *(замените имя RG и имя виртуальной машины значениями)*:

```azurecli
az vm reapply -g <RG Name> -n <VM Name>
```

Если "повторное применение виртуальной машины" не сработал, можно добавить новый пустой диск данных в виртуальную машину из портал управления Azure, а затем удалить его позже после того, как сертификат будет добавлен обратно.