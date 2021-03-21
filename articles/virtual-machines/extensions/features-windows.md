---
title: Расширения и компоненты виртуальной машины Azure для Windows
description: Узнайте о расширениях, доступных для виртуальных машин Azure. Расширения сгруппированы по предоставляемым функциям или улучшениям.
ms.topic: article
ms.service: virtual-machines
ms.subservice: extensions
author: amjads1
ms.author: amjads
ms.collection: windows
ms.date: 03/30/2018
ms.openlocfilehash: b9b10e2ed58a41cee99e7e6ef1c3994035460407
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104601866"
---
# <a name="virtual-machine-extensions-and-features-for-windows"></a>Обзор расширений и компонентов виртуальной машины под управлением Windows

Расширения виртуальных машин Azure — это небольшие приложения, которые выполняют задачи настройки и автоматизации после развертывания виртуальных машин Azure. Например, если на виртуальную машину требуется установить программное обеспечение, антивирусную защиту или выполнить сценарий внутри него, можно использовать расширение виртуальной машины. Расширения виртуальных машин Azure можно запускать через интерфейс командной строки Azure, PowerShell, шаблоны Azure Resource Manager или портал Azure. Их можно включить в пакет для развертывания виртуальной машины или запустить на любой из уже установленных систем.

Эта статья содержит обзор расширений виртуальной машины Azure, предварительные требования для их использования, рекомендации по их обнаружению, удалению и администрированию. Этот документ содержит только общие сведения, так как существует множество расширений виртуальных машин с разными параметрами настройки. Информацию о нужном расширении можно найти в соответствующей документации.

 

## <a name="use-cases-and-samples"></a>Варианты использования и примеры

Существует несколько разных расширений ВМ Azure, которые используются в определенных сценариях. Некоторые примеры:

- Применение к виртуальной машине конфигураций требуемого состояния PowerShell с помощью расширения DSC для Windows. Подробнее см. [Общие сведения об обработчике расширения Desired State Configuration в Azure](dsc-overview.md);
- Настройте мониторинг виртуальной машины с помощью расширения виртуальной машины агента Log Analytics. Дополнительные сведения см. [в статье подключение виртуальных машин Azure к Azure Monitor журналах](../../azure-monitor/vm/quick-collect-azurevm.md).
- Настройка виртуальной машины Azure с помощью Chef. Дополнительные сведения см. в статье [Автоматизация развертывания виртуальной машины Azure с помощью Chef](/azure/developer/chef/windows-vm-configure).
- настройка мониторинга инфраструктуры Azure с помощью расширения Datadog. Дополнительную информацию см. в [блоге Datadog](https://www.datadoghq.com/blog/introducing-azure-monitoring-with-one-click-datadog-deployment/);


Помимо расширений, созданных для конкретных процессов, существует расширение пользовательских сценариев для виртуальных машин под управлением Windows и Linux. Расширение пользовательских скриптов для Windows позволяет запустить на виртуальной машине любой скрипт PowerShell. Пользовательские сценарии могут пригодиться при проектировании развертывания Azure, для которого требуется дополнительная настройка, ее невозможно выполнить собственными средствами Azure. Подробнее см. [Использование расширений пользовательских сценариев для виртуальной машины Windows с шаблонами Azure Resource Manager](custom-script-windows.md).

## <a name="prerequisites"></a>Предварительные требования

Чтобы расширения работали на виртуальной машине, необходимо установить агент Windows для Azure. Некоторые расширения имеют дополнительные требования, например доступ к определенным ресурсам или зависимости.

### <a name="azure-vm-agent"></a>Агент виртуальной машины Azure

Агент виртуальной машины Azure управляет взаимодействием виртуальной машины с контроллером структуры Azure. Агент отвечает за многие функциональные аспекты развертывания виртуальных машин Azure и управления ими, включая запуск расширений виртуальной машины. Агент виртуальной машины Azure предварительно установлен во всех образах Azure Marketplace. Его также можно установить вручную в поддерживаемых операционных системах. Агент виртуальных машин Azure для Windows часто называют гостевым агентом Windows.

Сведения о поддерживаемых операционных системах и инструкции по установке см. в статье [Агент виртуальной машины Azure](agent-windows.md).

#### <a name="supported-agent-versions"></a>Поддерживаемые версии агента

Чтобы обеспечить максимальное удобство, мы предоставили минимальные версии агента. Дополнительные сведения см. в [этой статье](https://support.microsoft.com/help/4049215/extensions-and-virtual-machine-agent-minimum-version-support).

#### <a name="supported-oses"></a>Поддерживаемые операционные системы

Гостевой агент Windows работает на многих операционных системах, но платформа расширений налагает некоторые ограничения на совместимость расширений с операционными системами. Дополнительные сведения см. в [этой статье](https://support.microsoft.com/help/4078134/azure-extension-supported-operating-systems
).

Некоторые расширения поддерживаются не на всех операционных системах и вы можете столкнуться с *ошибкой 51 "Неподдерживаемая ОС"*. Сведения о совместимости приводятся в документации по отдельным расширениям.

#### <a name="network-access"></a>Доступ к сети

Пакеты расширений можно скачать из репозитория расширений в службе хранилища Azure. В ней также публикуются новости об актуальном состоянии расширений. Если вы используете [поддерживаемую](https://support.microsoft.com/help/4049215/extensions-and-virtual-machine-agent-minimum-version-support) версию агентов, вам не нужно разрешать доступ к службе хранилища Azure в регионе виртуальной машины, так как с помощью агента можно перенаправить подключение к контроллеру структуры Azure для связи с агентом (компонент хостгаплугин через привилегированный канал на частном IP- [168.63.129.16](../../virtual-network/what-is-ip-address-168-63-129-16.md)). Если вы используете неподдерживаемую версию агента, необходимо разрешить исходящий доступ от виртуальной машины к службе хранилище Azure в том же регионе.

> [!IMPORTANT]
> Если вы заблокировали доступ к *168.63.129.16* с помощью гостевого брандмауэра или прокси-сервера, расширения будут завершаться сбоем независимо от описанного выше. Требуются порты 80, 443 и 32526.

Агенты можно использовать только для скачивания пакетов расширений и отправки отчетов о состоянии. Например, если для установки расширения требуется скачать сценарий из GitHub (пользовательский сценарий) или требуется доступ к службе хранилища Azure (Azure Backup), откройте дополнительные порты в брандмауэре или в группе безопасности сети (NSG). Разные расширения имеют разные требования, ведь они по сути являются самостоятельными приложениями. Для расширений, которым требуется доступ к службе хранилища Azure или Azure Active Directory, можно разрешить доступ с помощью [тегов службы Azure NSG](../../virtual-network/network-security-groups-overview.md#service-tags) к хранилищу или AzureActiveDirectory.

Гостевой агент Windows не поддерживает прокси-сервер для перенаправления запросов трафика агента через. Это означает, что гостевой агент Windows будет использовать пользовательский прокси-сервер (если он есть) для доступа к ресурсам в Интернете или на узле через IP-168.63.129.16.

## <a name="discover-vm-extensions"></a>Поиск расширений ВМ

Существует множество разных расширений виртуальных машин, которые можно использовать с виртуальными машинами Azure. Их полный список можно получить с помощью командлета [Get-AzVMExtensionImage](/powershell/module/az.compute/get-azvmextensionimage). В следующем примере перечислены все расширения, доступные в расположении *WestUS*:

```powershell
Get-AzVmImagePublisher -Location "WestUS" | `
Get-AzVMExtensionImageType | `
Get-AzVMExtensionImage | Select Type, Version
```

## <a name="run-vm-extensions"></a>Запуск расширений ВМ

Расширения виртуальных машин Azure можно запускать на имеющихся виртуальных машинах, например для изменения конфигурации или восстановления подключения на ранее развернутой виртуальной машине. Также расширения можно использовать при развертывании с помощью шаблонов Azure Resource Manager. Включив расширения в шаблоны Resource Manager, вы сможете развернуть виртуальные машины Azure сразу со всеми нужными настройками, без дополнительных действий после развертывания.

Запустить расширение на имеющейся виртуальной машине можно несколькими способами.

### <a name="powershell"></a>PowerShell

Есть несколько команд PowerShell, позволяющих выполнять отдельные модули. Чтобы просмотреть список, используйте [Get-Command](/powershell/module/microsoft.powershell.core/get-command) с фильтрацией по значению *Extension*:

```powershell
Get-Command Set-Az*Extension* -Module Az.Compute
```

Вы увидите примерно такой результат:

```powershell
CommandType     Name                                          Version    Source
-----------     ----                                          -------    ------
Cmdlet          Set-AzVMAccessExtension                       4.5.0      Az.Compute
Cmdlet          Set-AzVMADDomainExtension                     4.5.0      Az.Compute
Cmdlet          Set-AzVMAEMExtension                          4.5.0      Az.Compute
Cmdlet          Set-AzVMBackupExtension                       4.5.0      Az.Compute
Cmdlet          Set-AzVMBginfoExtension                       4.5.0      Az.Compute
Cmdlet          Set-AzVMChefExtension                         4.5.0      Az.Compute
Cmdlet          Set-AzVMCustomScriptExtension                 4.5.0      Az.Compute
Cmdlet          Set-AzVMDiagnosticsExtension                  4.5.0      Az.Compute
Cmdlet          Set-AzVMDiskEncryptionExtension               4.5.0      Az.Compute
Cmdlet          Set-AzVMDscExtension                          4.5.0      Az.Compute
Cmdlet          Set-AzVMExtension                             4.5.0      Az.Compute
Cmdlet          Set-AzVMSqlServerExtension                    4.5.0      Az.Compute
Cmdlet          Set-AzVmssDiskEncryptionExtension             4.5.0      Az.Compute
```

В следующем примере расширение пользовательских сценариев загружает скрипт из репозитория GitHub на целевую виртуальную машину и запускает этот скрипт. Дополнительные сведения об использовании расширения пользовательских сценариев см. в статье [Использование расширений пользовательских сценариев для виртуальной машины Windows с шаблонами Azure Resource Manager](custom-script-windows.md).

```powershell
Set-AzVMCustomScriptExtension -ResourceGroupName "myResourceGroup" `
    -VMName "myVM" -Name "myCustomScript" `
    -FileUri "https://raw.githubusercontent.com/neilpeterson/nepeters-azure-templates/master/windows-custom-script-simple/support-scripts/Create-File.ps1" `
    -Run "Create-File.ps1" -Location "West US"
```

В следующем примере расширение доступа к виртуальной машине сбрасывает текущий и устанавливает новый временный пароль администратора на виртуальной машине Windows. Дополнительные сведения о расширении доступа к виртуальной машине см. в статье [Сброс службы удаленных рабочих столов или ее пароля для входа в систему на виртуальной машине под управлением Windows](/troubleshoot/azure/virtual-machines/reset-rdp). После этой операции следует сбросить пароль при первом входе в систему:

```powershell
$cred=Get-Credential

Set-AzVMAccessExtension -ResourceGroupName "myResourceGroup" -VMName "myVM" -Name "myVMAccess" `
    -Location WestUS -UserName $cred.GetNetworkCredential().Username `
    -Password $cred.GetNetworkCredential().Password -typeHandlerVersion "2.0"
```

С помощью команды `Set-AzVMExtension` можно запустить любое расширение виртуальной машины. Дополнительные сведения см. в разделе справки [о командлете Set-AzVMExtension](/powershell/module/az.compute/set-azvmextension).


### <a name="azure-portal"></a>Портал Azure

Расширения виртуальных машин можно устанавливать на имеющиеся виртуальные машины через портал Azure. Выберите на портале нужную виртуальную машину, затем щелкните **Расширения** и **Добавить**. Выберите нужное расширение из списка доступных расширений и следуйте инструкциям мастера.

В следующем примере показана установка расширения Microsoft Antimalware с помощью портала Azure.

![Установка расширения защиты от вредоносных программ](./media/features-windows/installantimalwareextension.png)

### <a name="azure-resource-manager-templates"></a>Шаблоны Azure Resource Manager

Расширения ВМ можно добавлять в шаблоны Azure Resource Manager. Так они смогут выполняться при развертывании соответствующего шаблона. Развертывая расширение вместе с шаблоном, можно создать полностью настроенное развертывание Azure. Например, следующий код JSON берется из шаблона диспетчер ресурсов развертывает набор виртуальных машин с балансировкой нагрузки и базу данных SQL Azure, а затем устанавливает приложение .NET Core на каждой виртуальной машине. Расширение ВМ отвечает за установку программного обеспечения.

Дополнительные сведения см. в разделе [шаблон полного диспетчер ресурсов](https://github.com/Microsoft/dotnet-core-sample-templates/tree/master/dotnet-core-music-windows).

```json
{
    "apiVersion": "2015-06-15",
    "type": "extensions",
    "name": "config-app",
    "location": "[resourceGroup().location]",
    "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'),copyindex())]",
    "[variables('musicstoresqlName')]"
    ],
    "tags": {
    "displayName": "config-app"
    },
    "properties": {
    "publisher": "Microsoft.Compute",
    "type": "CustomScriptExtension",
    "typeHandlerVersion": "1.9",
    "autoUpgradeMinorVersion": true,
    "settings": {
        "fileUris": [
        "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-windows/scripts/configure-music-app.ps1"
        ]
    },
    "protectedSettings": {
        "commandToExecute": "[concat('powershell -ExecutionPolicy Unrestricted -File configure-music-app.ps1 -user ',parameters('adminUsername'),' -password ',parameters('adminPassword'),' -sqlserver ',variables('musicstoresqlName'),'.database.windows.net')]"
    }
    }
}
```

Дополнительные сведения о создании шаблонов Azure Resource Manager см. в [этом разделе](../windows/template-description.md#extensions).

## <a name="secure-vm-extension-data"></a>Защита данных в расширениях ВМ

Для запуска расширений виртуальных машин может потребоваться конфиденциальная информация, например учетные данные, имена учетных записей хранения или ключи доступа к учетной записи хранения. Многие расширения виртуальных машин поддерживают защищенную конфигурацию, при использовании которой данные хранятся в зашифрованном виде и расшифровываются только на целевой виртуальной машине. Каждое расширение имеет собственную схему защищенной конфигурации, которая описывается в документации по этим расширениям.

В следующем примере показан экземпляр расширения пользовательских сценариев для Windows. Выполняемая команда содержит набор учетных данных. В этом примере выполняемая команда не зашифрована.

```json
{
    "apiVersion": "2015-06-15",
    "type": "extensions",
    "name": "config-app",
    "location": "[resourceGroup().location]",
    "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'),copyindex())]",
    "[variables('musicstoresqlName')]"
    ],
    "tags": {
    "displayName": "config-app"
    },
    "properties": {
    "publisher": "Microsoft.Compute",
    "type": "CustomScriptExtension",
    "typeHandlerVersion": "1.9",
    "autoUpgradeMinorVersion": true,
    "settings": {
        "fileUris": [
        "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-windows/scripts/configure-music-app.ps1"
        ],
        "commandToExecute": "[concat('powershell -ExecutionPolicy Unrestricted -File configure-music-app.ps1 -user ',parameters('adminUsername'),' -password ',parameters('adminPassword'),' -sqlserver ',variables('musicstoresqlName'),'.database.windows.net')]"
    }
    }
}
```

Переместив свойство **commandToExecute** в конфигурацию **protected**, вы защитите данные из строки выполнения, как показано в следующем примере:

```json
{
    "apiVersion": "2015-06-15",
    "type": "extensions",
    "name": "config-app",
    "location": "[resourceGroup().location]",
    "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'),copyindex())]",
    "[variables('musicstoresqlName')]"
    ],
    "tags": {
    "displayName": "config-app"
    },
    "properties": {
    "publisher": "Microsoft.Compute",
    "type": "CustomScriptExtension",
    "typeHandlerVersion": "1.9",
    "autoUpgradeMinorVersion": true,
    "settings": {
        "fileUris": [
        "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-windows/scripts/configure-music-app.ps1"
        ]
    },
    "protectedSettings": {
        "commandToExecute": "[concat('powershell -ExecutionPolicy Unrestricted -File configure-music-app.ps1 -user ',parameters('adminUsername'),' -password ',parameters('adminPassword'),' -sqlserver ',variables('musicstoresqlName'),'.database.windows.net')]"
    }
    }
}
```

На виртуальной машине Azure IaaS, использующей расширения, в консоли сертификатов могут отображаться сертификаты с темой " **_генератор сертификатов Microsoft Azure CRP_**". На классической виртуальной машине RDFE эти сертификаты имеют имя субъекта **_Управление службами Windows Azure для расширений_**.

Эти сертификаты обеспечивают безопасность обмена данными между виртуальной машиной и ее узлом во время передачи защищенных параметров (пароля, других учетных данных), используемых расширениями. Сертификаты создаются контроллером структуры Azure и передаются агенту виртуальной машины. Если вы останавливаете и запускаете виртуальную машину каждый день, контроллер структуры может создавать сертификат. Сертификат хранится в хранилище личных сертификатов компьютера. Эти сертификаты можно удалять. При необходимости агент виртуальной машины повторно создает сертификаты.

### <a name="how-do-agents-and-extensions-get-updated"></a>Как обновляются агенты и расширения?

Агенты и расширения используют общий механизм обновления. Некоторые обновления не требуют дополнительных правил брандмауэра.

Когда обновление становится доступно, оно устанавливается на виртуальных машинах только при внесении изменений в расширения или другие элементы модели виртуальной машины, как например:

- Диски данных
- Расширения
- контейнер диагностики загрузки;
- секреты гостевой ОС;
- Размер виртуальной машины
- сетевой профиль.

Издатели предоставляют обновления для разных регионов в разное время, а значит виртуальные машины в нескольких регионах могут одновременно использовать разные версии.

#### <a name="listing-extensions-deployed-to-a-vm"></a>Получение списка расширений, развернутых на виртуальной машине

```powershell
$vm = Get-AzVM -ResourceGroupName "myResourceGroup" -VMName "myVM"
$vm.Extensions | select Publisher, VirtualMachineExtensionType, TypeHandlerVersion
```

```powershell
Publisher             VirtualMachineExtensionType          TypeHandlerVersion
---------             ---------------------------          ------------------
Microsoft.Compute     CustomScriptExtension                1.9
```

#### <a name="agent-updates"></a>Обновления агентов

Гостевой агент Windows содержит только *код обработки расширений*, а *код подготовки Windows* отделен от него. Гостевой агент Windows можно удалить. Автоматическое обновление гостевого агента Windows отключить невозможно.

*Код обработки расширений* отвечает за взаимодействие со структурой Azure и обработку операций расширения виртуальных машин, такие как установка, отправка отчетов о состоянии, обновление и удаление отдельных расширений. Обновления содержат исправления безопасности, исправления ошибок и улучшения в *коде обработки расширения*.

Чтобы узнать установленную версию, используйте инструкции из раздела [Detect the VM Agent](agent-windows.md#detect-the-vm-agent) (Обнаружение агента виртуальной машины).

#### <a name="extension-updates"></a>Обновление расширений

Когда становится доступно обновление расширения, гостевой агент Windows скачивает его и обновляет расширение. Автоматические обновления расширений классифицируются как *дополнения* или *исправления*. При подготовке расширения вы можете согласиться на установку обновлений категории *Дополнения* либо отказаться от них. В следующем примере показано, как можно автоматически обновлять дополнительные версии в шаблоне Resource Manager с помощью параметра *autoUpgradeMinorVersion": true,*:

```json
    "properties": {
    "publisher": "Microsoft.Compute",
    "type": "CustomScriptExtension",
    "typeHandlerVersion": "1.9",
    "autoUpgradeMinorVersion": true,
    "settings": {
        "fileUris": [
        "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-windows/scripts/configure-music-app.ps1"
        ]
    },
```

Мы настоятельно рекомендуем включить автоматическое обновление для развертываний расширений, чтобы своевременно получать дополнительные исправления ошибок. У вас нет возможности отказаться от обновлений, которые содержат исправления безопасности или значительных ошибок.

### <a name="how-to-identify-extension-updates"></a>Как определить параметры обновления расширений

#### <a name="identifying-if-the-extension-is-set-with-autoupgrademinorversion-on-a-vm"></a>Определение значения параметра autoUpgradeMinorVersion для расширения на виртуальной машине

Модель виртуальной машины позволяет узнать, был ли установлен параметр autoUpgradeMinorVersion при подготовке расширения. Для этого запустите командлет [Get-AzVm](/powershell/module/az.compute/get-azvm), указав группу ресурсов и имя виртуальной машины следующим образом:

```powerShell
 $vm = Get-AzVm -ResourceGroupName "myResourceGroup" -VMName "myVM"
 $vm.Extensions
```

Выходные данные в следующем примере указывают, что параметр *autoUpgradeMinorVersion* имеет значение *true*:

```powershell
ForceUpdateTag              :
Publisher                   : Microsoft.Compute
VirtualMachineExtensionType : CustomScriptExtension
TypeHandlerVersion          : 1.9
AutoUpgradeMinorVersion     : True
```

#### <a name="identifying-when-an-autoupgrademinorversion-occurred"></a>Определение наличия установленного обновления autoUpgradeMinorVersion

Чтобы узнать, когда становится доступно обновление для расширения, изучите журналы агента на виртуальной машине, расположенные по адресу *C:\WindowsAzure\Logs\WaAppAgent.log*

В следующем примере мы видим, что на виртуальной машине установлена версия *Microsoft.Compute.CustomScriptExtension 1.8*. И для этого расширения доступно обновление категории "Исправление" до версии *1.9*:

```powershell
[INFO]  Getting plugin locations for plugin 'Microsoft.Compute.CustomScriptExtension'. Current Version: '1.8', Requested Version: '1.9'
[INFO]  Auto-Upgrade mode. Highest public version for plugin 'Microsoft.Compute.CustomScriptExtension' with requested version: '1.9', is: '1.9'
```

## <a name="agent-permissions"></a>Разрешения для агента

Чтобы агент работал должным образом, он должен запускаться от имени учетной записи *локальной системы*.

## <a name="troubleshoot-vm-extensions"></a>Устранение неполадок с расширениями ВМ

Для каждого расширения ВМ предусмотрена своя процедура устранения неполадок. Например, если вы используете расширение пользовательских сценариев, сведения о выполнении сценария можно найти на той виртуальной машине, где запускалось расширение. Действия по устранению неполадок для конкретных расширений описаны в документации по соответствующим расширениям.

Следующие инструкции по устранению неполадок применимы ко всем расширениям виртуальных машин.

1. Чтобы проверить журнал гостевого агента Windows, просмотрите действие при подготовке расширения в *к:\виндовсазуре\логс\вааппажент.лог* .

2. Дополнительные сведения см. в журналах фактического расширения в *к:\виндовсазуре\логс\плугинс \\ <extensionName>*

3. Коды ошибок, известные проблемы и другие данные для устранения неполадок вы найдете в документации по конкретному расширению.

4. Просмотрите системные журналы. Проверьте наличие других операций, которые могут мешать работе расширения, например длительные установки других приложений, требующие монопольного доступа к диспетчеру пакетов.

### <a name="common-reasons-for-extension-failures"></a>Распространенные причины сбоев расширений

1. Расширению предоставляется на выполнение всего 20 минут (за исключением расширения пользовательских сценариев, Chef и DSC, которые могут работать 90 минут). Если в вашем развертывании требуется больше времени, для расширения фиксируется превышение времени ожидания. Это может быть вызвано нехваткой ресурсов на виртуальной машине или выполнением на той же виртуальной машине во время работы приложения других задач конфигурации или запуска, которые потребляют большие объемы ресурсов.

2. Не соблюдаются минимальные требования. Некоторые расширения имеют зависимость от номера SKU виртуальной машины, например от образа HPC. Расширения могут иметь требования к сетевому доступу, такие как наличие связи со службой хранилища Azure или другими общедоступными службами. Работе также могут помешать отсутствие доступа к репозиторию пакетов, нехватка места на диске или ограничения безопасности.

3. Монопольный доступ к диспетчеру пакетов. В некоторых случаях длительная настройка виртуальной машины может конфликтовать с установкой расширения, если оба этих процесса требуют монопольный доступ к диспетчеру пакетов.

### <a name="view-extension-status"></a>Просмотр состояния расширения

После запуска расширения ВИРТУАЛЬНОЙ машины для виртуальной машины используйте команду [Get-AzVM](/powershell/module/az.compute/get-azvm) , чтобы вернуть состояние расширения. *Substatuses[0]* указывает, что подготовка расширения прошла успешно, а значит оно правильно развернуто на виртуальной машине. Но *Substatuses[1]* указывает, что выполнение расширения на виртуальной машине завершилось сбоем.

```powershell
Get-AzVM -ResourceGroupName "myResourceGroup" -VMName "myVM" -Status
```

Вы должны увидеть результат, аналогичный приведенному ниже.

```powershell
Extensions[0]           :
  Name                  : CustomScriptExtension
  Type                  : Microsoft.Compute.CustomScriptExtension
  TypeHandlerVersion    : 1.9
  Substatuses[0]        :
    Code                : ComponentStatus/StdOut/succeeded
    Level               : Info
    DisplayStatus       : Provisioning succeeded
    Message             : Windows PowerShell \nCopyright (C) Microsoft Corporation. All rights reserved.\n
  Substatuses[1]        :
    Code                : ComponentStatus/StdErr/succeeded
    Level               : Info
    DisplayStatus       : Provisioning succeeded
    Message             : The argument 'cseTest%20Scriptparam1.ps1' to the -File parameter does not exist. Provide the path to an existing '.ps1' file as an argument to the

-File parameter.
  Statuses[0]           :
    Code                : ProvisioningState/failed/-196608
    Level               : Error
    DisplayStatus       : Provisioning failed
    Message             : Finished executing command
```

Кроме того, состояние выполнения расширения можно найти на портале Azure. Чтобы просмотреть состояние расширения, выберите виртуальную машину, затем щелкните **Расширения** и найдите нужное расширение.

### <a name="rerun-vm-extensions"></a>Повторный запуск расширений виртуальной машины

В некоторых случаях может потребоваться перезапуск расширения виртуальной машины. Можно сначала удалить расширение, затем повторно запустить его любым удобным способом. Чтобы удалить расширение, выполните командлет [Remove-AzVMExtension](/powershell/module/az.compute/remove-azvmextension) следующим образом:

```powershell
Remove-AzVMExtension -ResourceGroupName "myResourceGroup" -VMName "myVM" -Name "myExtensionName"
```

Кроме того, вы можете удалить расширение через портал Azure, сделав следующее:

1. Выберите виртуальную машину.
2. Выберите **Расширения**.
3. Выберите нужное расширение.
4. Выберите **Удалить**.

## <a name="common-vm-extensions-reference"></a>Справочники по распространенным расширениям виртуальной машины
| Имя расширения | Описание | Дополнительные сведения |
| --- | --- | --- |
| Расширение Custom Script в ОС Windows |Выполняет сценарии на виртуальных машинах Azure. |[Расширение Custom Script в ОС Windows](custom-script-windows.md) |
| Расширение DSC в ОС Windows |Расширение PowerShell DSC (настройка требуемого состояния) |[Расширение DSC в ОС Windows](dsc-overview.md) |
| Расширение Диагностики Azure |Управляет системой диагностики Azure |[Расширение система диагностики Azure](https://azure.microsoft.com/blog/windows-azure-virtual-machine-monitoring-with-wad-extension/) |
| Расширение Azure VM Access |Управляет пользователями и учетными данными. |[Расширение для доступа к виртуальной машине для Linux](https://azure.microsoft.com/blog/using-vmaccess-extension-to-reset-login-credentials-for-linux-vm/) |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о расширениях виртуальных машин см. в статье c [обзором расширений и компонентов виртуальной машины Azure](overview.md).