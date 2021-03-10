---
title: Расширение Azure Custom Script в ОС Windows
description: Сведения об автоматизации задач настройки виртуальных машин Windows с помощью расширения пользовательских сценариев
ms.topic: article
ms.service: virtual-machines
ms.subservice: extensions
ms.author: amjads
author: amjads1
ms.collection: windows
ms.date: 08/31/2020
ms.openlocfilehash: 13b4c4ef50ea37cabe30474d339acb19176cef97
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102553907"
---
# <a name="custom-script-extension-for-windows"></a>Расширение Custom Script в ОС Windows

Расширение пользовательских сценариев загружает и запускает сценарии на виртуальных машинах Azure. Это расширение можно использовать для настройки после развертывания, установки программного обеспечения и других задач настройки или управления. Сценарии можно скачать из службы хранилища Azure или GitHub или передать на портал Azure во время выполнения расширения. Расширение пользовательских сценариев интегрируется с шаблонами Azure Resource Manager, а также его можно запустить с помощью интерфейса командной строки Azure, PowerShell, портала Azure или API REST виртуальной машины Azure.

В этом документе объясняется, как использовать расширение пользовательских сценариев с помощью модуля Azure PowerShell и шаблонов Azure Resource Manager, а также подробно описываются действия по устранению неполадок в системах Windows.

## <a name="prerequisites"></a>Предварительные требования

> [!NOTE]  
> Не используйте расширение пользовательских сценариев для выполнения команды Update-AzVM на той же виртуальной машине, которая использовалась для параметра, так как команда выполнится через определенный период времени.  

### <a name="operating-system"></a>Операционная система

Расширение настраиваемых скриптов для Linux будет работать в поддерживаемых им ОС.

### <a name="windows"></a>Windows

* Windows Server 2008 R2
* Windows Server 2012
* Windows Server 2012 R2
* Windows 10
* Windows Server 2016
* Windows Server 2016 Core
* Windows Server 2019
* Windows Server 2019 Core

### <a name="script-location"></a>Расположение сценария

Вы можете настроить расширение, чтобы получить доступ к хранилищу BLOB-объектов Azure с помощью учетных данных службы хранилища BLOB-объектов Azure. Скрипт может находиться в любом расположении при условии, что виртуальная машина может осуществлять маршрутизацию в конечную точку, такую как GitHub или внутренний файловый сервер.

### <a name="internet-connectivity"></a>Подключение к Интернету

Чтобы скачать скрипт из внешнего источника, например с сайта GitHub или из службы хранилища Azure, откройте дополнительные порты брандмауэра или группы безопасности сети. Например, если скрипт находится в службе хранилища Azure, вы можете разрешить доступ с помощью тегов службы Azure NSG для [хранилища](../../virtual-network/network-security-groups-overview.md#service-tags).

Обратите внимание, что у расширения CustomScript нет способа обойти проверку сертификата. Поэтому при скачивании из защищенного расположения с помощью например. самозаверяющий сертификат, возможно, в итоге возникнут такие ошибки, как *"удаленный сертификат является недопустимым в соответствии с процедурой проверки"*. Убедитесь, что сертификат правильно установлен в хранилище *"Доверенные корневые центры сертификации"* на виртуальной машине.

Если скрипт расположен на локальном сервере, вам по-прежнему может потребоваться открыть дополнительные порты брандмауэра и группы безопасности сети.

### <a name="tips-and-tricks"></a>Советы и рекомендации

* Высокий процент сбоев этого расширения связан с синтаксическими ошибками в скрипте. Чтобы упростить поиск точки сбоя, протестируйте запуски скрипта без ошибок и включите в скрипт дополнительные возможности ведения журнала.
* Рекомендуется писать идемпотентные скрипты. Это гарантирует, что если они снова будут выполняться случайно, это не приведет к изменениям системы.
* Выполняемые скрипты не должны запрашивать ввод данных пользователем.
* На выполнение скрипта отводится 90 минут. Более продолжительное действие вызовет сбой подготовки расширения.
* Не следует помещать перезагрузки в скрипт, так как будут возникать проблемы с другими устанавливаемыми расширениями. После перезагрузки расширение прекратить работать.
* Если есть сценарий, который вызывает перезагрузку, а затем устанавливает приложения и запускает сценарии, можно запланировать перезагрузку с помощью запланированной задачи Windows или использовать такие средства, как расширения DSC, Chef или Puppet.
* Не рекомендуется запускать сценарий, который приведет к сбою или обновлению агента виртуальной машины. Это позволяет оставить расширение в состоянии перехода, что приведет к превышению времени ожидания.
* Расширение позволяет запустить скрипт только один раз. Если нужно выполнять скрипт при каждой загрузке, создайте запланированную задачу Windows с помощью расширения.
* Чтобы запланировать время выполнения скрипта, необходимо создать запланированную задачу Windows с помощью расширения.
* Во время выполнения скрипта вы увидите на портале Microsoft Azure или в CLI только переходное состояние расширения. Если требуются более частые обновления состояния выполняющегося скрипта, следует создать собственное решение.
* Расширение пользовательских сценариев изначально не поддерживает прокси-серверы, однако можно использовать средство для обмена файлами, которое поддерживает прокси-серверы в скрипте, например *Invoke-WebRequest* .
* Следует учитывать настраиваемые расположения каталогов, которые могут использоваться скриптами или командами, и иметь в распоряжении логику для обработки таких ситуаций.
* Расширение пользовательских скриптов будет выполняться под учетной записью LocalSystem
* Если вы планируете использовать свойства *storageAccountName* и *storageAccountKey* , эти свойства должны быть выровнены в *protectedSettings*.

## <a name="extension-schema"></a>Схема расширения

В конфигурации расширения пользовательских сценариев указываются такие параметры, как расположение сценария и команда для его выполнения. Эту конфигурацию можно хранить в файлах конфигурации, указать в командной строке или в шаблоне Azure Resource Manager.

Конфиденциальные данные можно хранить в зашифрованном виде в защищенной конфигурации, которая расшифровывается только внутри виртуальной машины. Защищенную конфигурацию можно использовать для команд, которые содержат секретные данные, например пароли.

Эти элементы следует рассматривать в качестве конфиденциальных данных. Их нужно задать в защищенной конфигурации параметров расширений. Данные защищенных параметров расширения виртуальной машины Azure зашифрованы. Они расшифровываются только на целевой виртуальной машине.

```json
{
    "apiVersion": "2018-06-01",
    "type": "Microsoft.Compute/virtualMachines/extensions",
    "name": "virtualMachineName/config-app",
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
        "typeHandlerVersion": "1.10",
        "autoUpgradeMinorVersion": true,
        "settings": {
            "fileUris": [
                "script location"
            ],
            "timestamp":123456789
        },
        "protectedSettings": {
            "commandToExecute": "myExecutionCommand",
            "storageAccountName": "myStorageAccountName",
            "storageAccountKey": "myStorageAccountKey",
            "managedIdentity" : {}
        }
    }
}
```

> [!NOTE]
> Свойство managedIdentity **не должны** использоваться в сочетании со свойствами storageAccountName или storageAccountKey

> [!NOTE]
> На виртуальной машине нельзя устанавливать больше одной версии расширения одновременно. Если указать пользовательский скрипт дважды в одном и том же шаблоне Resource Manager для той же виртуальной машины, возникнет ошибка.

> [!NOTE]
> Эту схему можно использовать в ресурсе VirtualMachine или в качестве автономного ресурса. Имя ресурса должно быть в формате "virtualMachineName/extensionName", если это расширение используется в качестве автономного ресурса в шаблоне Resource Manager.

### <a name="property-values"></a>Значения свойств

| Имя | Значение и пример | Тип данных |
| ---- | ---- | ---- |
| версия_API | 2015-06-15 | Дата |
| publisher | Microsoft.Compute; | строка |
| type | CustomScriptExtension | строка |
| typeHandlerVersion | 1,10 | INT |
| fileUris (пример) | https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-windows/scripts/configure-music-app.ps1 | массиве |
| метка времени (например) | 123456789 | 32-разрядное целое число |
| commandToExecute (пример) | powershell -ExecutionPolicy Unrestricted -File configure-music-app.ps1 | строка |
| storageAccountName (пример) | examplestorageacct | строка |
| storageAccountKey (пример) | TmJK/1N3AbAZ3q/+hOXoi/l73zOqsaxXDhqa9Y83/v5UpXQp2DQIBuv2Tifp60cE/OaHsJZmQZ7teQfczQj8hg== | строка |
| managedIdentity (пример) | {} или {"clientId": "31b403aa-c364-4240-A7FF-d85fb6cd7232"} или {"objectId": "12dd289c-0583-46e5-b9b4-115d5c19ef4b" } | Объект JSON |

>[!NOTE]
>В именах свойств учитывается регистр. Чтобы избежать проблем с развертыванием, используйте имена, как показано ниже.

#### <a name="property-value-details"></a>Сведения о значениях свойств

* `commandToExecute`: (**обязательное**, строка) выполняемый сценарий точки входа. Если команда содержит секретные данные, например пароли, или если значения fileUri являются конфиденциальным, используйте взамен это поле.
* `fileUris`: (необязательное, строковый массив) URL-адреса файлов для скачивания.
* `timestamp` (необязательный, 32-битное целое число) используется только для повторного запуска скрипта при изменении значения этого поля.  Любое целочисленное значение допустимо, оно только должно отличаться от предыдущего значения.
* `storageAccountName`: (необязательное, строка) имя учетной записи хранения. Если указаны учетные данные хранилища, все значения `fileUris` должны иметь формат URL-адресов для BLOB-объектов Azure.
* `storageAccountKey`: (необязательное, строка) ключ доступа для учетной записи хранения.
* `managedIdentity`: (необязательный, объект JSON) [управляемое удостоверение](../../active-directory/managed-identities-azure-resources/overview.md) для скачивания файлов
  * `clientId`: (необязательный, строка) идентификатор клиента управляемого удостоверения
  * `objectId`: (необязательный, строка) идентификатор объекта управляемого удостоверения

Указанные ниже значения можно задавать либо в открытых, либо в защищенных параметрах. Расширение отклонит любую конфигурацию, в которой приведенные ниже значения будут указаны в открытых и защищенных параметрах одновременно.

* `commandToExecute`

Открытые параметры могут быть полезны для отладки, но рекомендуется использовать защищенные параметры.

Открытые параметры отправляются в виде открытого текста на виртуальную машину, где будет выполняться скрипт.  Защищенные параметры шифруются с помощью ключа, который известен только Azure и виртуальной машине. Параметры сохраняются на виртуальной машине в том виде, в каком они были отправлены, т. е. если параметры были зашифрованы, они сохраняются на виртуальной машине зашифрованными. Сертификат, используемый для расшифровки зашифрованных значений, хранится на виртуальной машине и применяется для расшифровки параметров (при необходимости) во время выполнения.

####  <a name="property-managedidentity"></a>Свойство: managedIdentity
> [!NOTE]
> Это свойство **должно** быть задано только в защищенных параметрах.

CustomScript (начиная с версии 1.10) поддерживает [управляемое удостоверение](../../active-directory/managed-identities-azure-resources/overview.md) для скачивания файлов с URL-адресов, указанных в параметре "fileUris". Это предоставляет CustomScript доступ к частным BLOB-объектам или контейнерам службы хранилища Azure без необходимости передавать секреты, такие как маркеры SAS или ключи учетной записи хранения.

Чтобы использовать эту функцию, пользователь должен добавить [назначенное системой](../../app-service/overview-managed-identity.md?tabs=dotnet#add-a-system-assigned-identity) или [назначенное пользователем](../../app-service/overview-managed-identity.md?tabs=dotnet#add-a-user-assigned-identity) удостоверение на виртуальную машину или VMSS, где ожидается выполнение CustomScript, и [предоставить управляемому удостоверению доступ к контейнеру службы хранилища Azure или BLOB-объектов](../../active-directory/managed-identities-azure-resources/tutorial-vm-windows-access-storage.md#grant-access).

Чтобы использовать назначенное системой удостоверение на целевой виртуальной машине или VMSS, задайте в поле "managedidentity" пустой объект JSON. 

> Пример
>
> ```json
> {
>   "fileUris": ["https://mystorage.blob.core.windows.net/privatecontainer/script1.ps1"],
>   "commandToExecute": "powershell.exe script1.ps1",
>   "managedIdentity" : {}
> }
> ```

Чтобы использовать назначенное пользователем удостоверение на целевой виртуальной машине или VMSS, настройте в поле "managedidentity" идентификатор клиента или идентификатор объекта управляемого удостоверения.

> Примеры:
>
> ```json
> {
>   "fileUris": ["https://mystorage.blob.core.windows.net/privatecontainer/script1.ps1"],
>   "commandToExecute": "powershell.exe script1.ps1",
>   "managedIdentity" : { "clientId": "31b403aa-c364-4240-a7ff-d85fb6cd7232" }
> }
> ```
> ```json
> {
>   "fileUris": ["https://mystorage.blob.core.windows.net/privatecontainer/script1.ps1"],
>   "commandToExecute": "powershell.exe script1.ps1",
>   "managedIdentity" : { "objectId": "12dd289c-0583-46e5-b9b4-115d5c19ef4b" }
> }
> ```

> [!NOTE]
> Свойство managedIdentity **не должны** использоваться в сочетании со свойствами storageAccountName или storageAccountKey

## <a name="template-deployment"></a>Развертывание шаблона

Расширения виртуальной машины Azure можно развернуть с помощью шаблонов Azure Resource Manager. Схема JSON, подробно описанная в предыдущем разделе, может использоваться в шаблоне Azure Resource Manager для запуска расширения пользовательского скрипта во время развертывания. Приведенные ниже примеры демонстрируют использование расширения пользовательских сценариев.

* [Руководство. Развертывание расширений виртуальной машины с помощью шаблонов Azure Resource Manager](../../azure-resource-manager/templates/template-tutorial-deploy-vm-extensions.md).
* [Развертывание двухуровневого приложения в Windows и базе данных Azure SQL](https://github.com/Microsoft/dotnet-core-sample-templates/tree/master/dotnet-core-music-windows)

## <a name="powershell-deployment"></a>Развертывание с помощью PowerShell

Выполнив команду `Set-AzVMCustomScriptExtension`, расширение пользовательских сценариев можно добавить на существующую виртуальную машину. Дополнительные сведения см. в статье о [Set-AzVMCustomScriptExtension](/powershell/module/az.compute/set-azvmcustomscriptextension).

```powershell
Set-AzVMCustomScriptExtension -ResourceGroupName <resourceGroupName> `
    -VMName <vmName> `
    -Location myLocation `
    -FileUri <fileUrl> `
    -Run 'myScript.ps1' `
    -Name DemoScriptExtension
```

## <a name="additional-examples"></a>Дополнительные примеры

### <a name="using-multiple-scripts"></a>Использование нескольких скриптов

В этом примере три скрипта используются для построения сервера. **commandToExecute** вызывает первый скрипт, таким образом у вас есть параметры вызова других. Например, можно создать главный сценарий, который управляет выполнением с правильной обработкой ошибок, ведением журнала и управлением состоянием. Скрипты загружаются на локальный компьютер для выполнения. Например, в `1_Add_Tools.ps1` можно вызвать `2_Add_Features.ps1`, добавив `.\2_Add_Features.ps1` в скрипт, и повторить эту процедуру для других скриптов, определенных в `$settings`.

```powershell
$fileUri = @("https://xxxxxxx.blob.core.windows.net/buildServer1/1_Add_Tools.ps1",
"https://xxxxxxx.blob.core.windows.net/buildServer1/2_Add_Features.ps1",
"https://xxxxxxx.blob.core.windows.net/buildServer1/3_CompleteInstall.ps1")

$settings = @{"fileUris" = $fileUri};

$storageAcctName = "xxxxxxx"
$storageKey = "1234ABCD"
$protectedSettings = @{"storageAccountName" = $storageAcctName; "storageAccountKey" = $storageKey; "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File 1_Add_Tools.ps1"};

#run command
Set-AzVMExtension -ResourceGroupName <resourceGroupName> `
    -Location <locationName> `
    -VMName <vmName> `
    -Name "buildserver1" `
    -Publisher "Microsoft.Compute" `
    -ExtensionType "CustomScriptExtension" `
    -TypeHandlerVersion "1.10" `
    -Settings $settings `
    -ProtectedSettings $protectedSettings;
```

### <a name="running-scripts-from-a-local-share"></a>Запуск скриптов из локальной общей папки

В этом примере может потребоваться использовать локальный сервер SMB в качестве расположения скрипта. Таким образом, вам не нужно указывать другие параметры, кроме **commandToExecute**.

```powershell
$protectedSettings = @{"commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File \\filesvr\build\serverUpdate1.ps1"};

Set-AzVMExtension -ResourceGroupName <resourceGroupName> `
    -Location <locationName> `
    -VMName <vmName> `
    -Name "serverUpdate"
    -Publisher "Microsoft.Compute" `
    -ExtensionType "CustomScriptExtension" `
    -TypeHandlerVersion "1.10" `
    -ProtectedSettings $protectedSettings

```

### <a name="how-to-run-custom-script-more-than-once-with-cli"></a>Многократное выполнение пользовательского скрипта с помощью CLI

Чтобы выполнить расширение пользовательского скрипта более одного раза, должны соблюдаться следующие условия:

* Параметр **Name** расширения совпадает с указанным при предыдущем развертывании расширения.
* Необходимо обновить конфигурацию, в противном случае команда не будет выполнена повторно. Можно добавить в команду динамическое свойство, например метку времени.

Кроме того, можно задать для свойства [ForceUpdateTag](/dotnet/api/microsoft.azure.management.compute.models.virtualmachineextension.forceupdatetag) значение **true**.

### <a name="using-invoke-webrequest"></a>Использование Invoke-WebRequest

Если вы используете [Invoke-WebRequest](/powershell/module/microsoft.powershell.utility/invoke-webrequest) в скрипте, необходимо указать параметр `-UseBasicParsing`, иначе при проверке подробного состояния появится следующая ошибка:

```error
The response content cannot be parsed because the Internet Explorer engine is not available, or Internet Explorer's first-launch configuration is not complete. Specify the UseBasicParsing parameter and try again.
```
## <a name="virtual-machine-scale-sets"></a>Масштабируемые наборы виртуальных машин Microsoft Azure

Сведения о развертывании расширения пользовательских скриптов в масштабируемом наборе см. в [Add-AzVmssExtension](/powershell/module/az.compute/add-azvmssextension)

## <a name="classic-vms"></a>классические виртуальные машины;

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]

Чтобы развернуть расширение пользовательских сценариев на классических виртуальных машинах, можно использовать портал Azure или классические командлеты Azure PowerShell.

### <a name="azure-portal"></a>Портал Azure

Перейдите к классическому ресурсу виртуальной машины. Выберите **Расширения** в разделе **Параметры**.

Щелкните **+ Добавить** и в списке ресурсов выберите **Расширение пользовательских скриптов**.

На странице **Установка расширения** выберите локальный файл PowerShell, заполните все аргументы и нажмите кнопку **ОК**.

### <a name="powershell"></a>PowerShell

Выполнив команду [Set-AzureVMCustomScriptExtension](/powershell/module/servicemanagement/azure.service/set-azurevmcustomscriptextension), вы сможете добавить расширение пользовательских сценариев на существующую виртуальную машину.

```powershell
# define your file URI
$fileUri = 'https://xxxxxxx.blob.core.windows.net/scripts/Create-File.ps1'

# create vm object
$vm = Get-AzureVM -Name <vmName> -ServiceName <cloudServiceName>

# set extension
Set-AzureVMCustomScriptExtension -VM $vm -FileUri $fileUri -Run 'Create-File.ps1'

# update vm
$vm | Update-AzureVM
```

## <a name="troubleshoot-and-support"></a>Устранение неполадок и поддержка

### <a name="troubleshoot"></a>Диагностика

Данные о состоянии развертывания расширения можно получить на портале Azure, а также с помощью модуля Azure PowerShell. Чтобы просмотреть состояние развертывания расширений для определенной виртуальной машины, выполните следующую команду:

```powershell
Get-AzVMExtension -ResourceGroupName <resourceGroupName> -VMName <vmName> -Name myExtensionName
```

Выходные данные расширения регистрируются в файле, расположенном в следующей папке на целевой виртуальной машине.

```cmd
C:\WindowsAzure\Logs\Plugins\Microsoft.Compute.CustomScriptExtension
```

Указанные файлы скачиваются в указанную ниже папку на целевой виртуальной машине.

```cmd
C:\Packages\Plugins\Microsoft.Compute.CustomScriptExtension\1.*\Downloads\<n>
```

`<n>` — десятичное целое число, которое может измениться между выполнениями расширения.  Значение `1.*` соответствует фактическому текущему значению `typeHandlerVersion` расширения.  Например, фактическим каталогом может быть `C:\Packages\Plugins\Microsoft.Compute.CustomScriptExtension\1.8\Downloads\2`.  

При выполнении команды `commandToExecute` расширение устанавливает этот каталог (например, `...\Downloads\2`) в качестве текущего рабочего каталога. Этот процесс позволяет использовать относительные пути для поиска файлов, скачанных с помощью свойства `fileURIs`. Примеры приведены в следующей таблице.

Так как со временем абсолютный путь для скачивания может измениться, в строке `commandToExecute` лучше указывать относительные пути к сценариям или файлам, когда это возможно. Пример:

```json
"commandToExecute": "powershell.exe . . . -File \"./scripts/myscript.ps1\""
```

Для файлов, скачанных с помощью списка свойств `fileUris`, сохраняются данные пути после первого сегмента универсального кода ресурса (URI).  Как показано в следующей таблице, скачанные файлы распределяются по подкаталогам в соответствии со структурой значений `fileUris`.  

#### <a name="examples-of-downloaded-files"></a>Примеры скачанных файлов

| Универсальный код ресурса (URI) в fileUris | Относительное расположение скачанных файлов | Абсолютное расположение скачанных файлов<sup>1</sup> |
| ---- | ------- |:--- |
| `https://someAcct.blob.core.windows.net/aContainer/scripts/myscript.ps1` | `./scripts/myscript.ps1` |`C:\Packages\Plugins\Microsoft.Compute.CustomScriptExtension\1.8\Downloads\2\scripts\myscript.ps1`  |
| `https://someAcct.blob.core.windows.net/aContainer/topLevel.ps1` | `./topLevel.ps1` | `C:\Packages\Plugins\Microsoft.Compute.CustomScriptExtension\1.8\Downloads\2\topLevel.ps1` |

<sup>1</sup>Абсолютные пути каталогов изменяются в течение времени существования виртуальной машины, но не в рамках одного выполнения расширения CustomScript.

### <a name="support"></a>Поддержка

Если в любой момент при изучении этой статьи вам потребуется дополнительная помощь, вы можете обратиться к экспертам по Azure на [форумах MSDN Azure и Stack Overflow](https://azure.microsoft.com/support/forums/). Можно также зарегистрировать обращение в службу поддержки Azure. Перейдите на [сайт поддержки Azure](https://azure.microsoft.com/support/options/) и щелкните "Получить поддержку". Дополнительные сведения об использовании службы поддержки Azure см. в статье [Часто задаваемые вопросы о поддержке Microsoft Azure](https://azure.microsoft.com/support/faq/).
