---
title: Расширение Azure DSC для Linux
description: Установка пакетов OMI и DSC для обеспечения возможности настройки виртуальной машины Linux в Azure с помощью Desired State Configuration.
ms.topic: article
ms.service: virtual-machines
ms.subservice: extensions
author: mgoedtel
ms.author: magoedte
ms.collection: linux
ms.date: 06/12/2018
ms.openlocfilehash: 3a9478f93a2a6f8e31be39a58a18e0e394b9d5c2
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102560112"
---
# <a name="dsc-extension-for-linux-microsoftostcextensionsdscforlinux"></a>Расширение DSC для Linux (Microsoft. OSTCExtensions. Дскфорлинукс)

Настройка требуемого состояния (DSC) — это платформа управления, которую можно использовать для управления инфраструктурой ИТ и разработки с помощью конфигурации в виде кода.

> [!NOTE]
> Расширение DSC для Linux и [log Analytics расширение виртуальной машины для Linux](./oms-linux.md) в настоящее время представляют конфликт и не поддерживаются в параллельной конфигурации. Не используйте эти два решения на одной виртуальной машине.

Расширение Дскфорлинукс публикуется и поддерживается корпорацией Майкрософт. Оно устанавливает агенты OMI и DSC на виртуальных машинах Azure. Расширение DSC также может выполнять следующие действия:

- Зарегистрируйте виртуальную машину Linux в учетной записи службы автоматизации Azure, чтобы извлекать конфигурации в службе автоматизации Azure (Register Екстенсионактион).
- Принудительная отправка конфигураций MOF на виртуальную машину Linux (Push Екстенсионактион).
- Примените конфигурацию meta MOF к виртуальной машине Linux, чтобы настроить опрашивающий сервер для настройки опрашивающего узла (Pull Екстенсионактион).
- Установите пользовательские модули DSC на виртуальную машину Linux (установите Екстенсионактион).
- Удалите пользовательские модули DSC с виртуальной машины Linux (удалите Екстенсионактион).

## <a name="prerequisites"></a>Предварительные требования

### <a name="operating-system"></a>Операционная система

Для узлов под управлением Linux расширение DSC для Linux поддерживает все дистрибутивы Linux, перечисленные в [документации по DSC PowerShell](/powershell/scripting/dsc/getting-started/lnxgettingstarted).

### <a name="internet-connectivity"></a>Подключение к Интернету

Для расширения Дскфорлинукс требуется, чтобы Целевая виртуальная машина была подключена к Интернету. Например, для расширения Register требуется подключение к службе автоматизации.
Для других действий, таких как извлечение, извлечение, требуется подключение к службе хранилища Azure и GitHub. Это зависит от параметров, предоставленных клиентом.

## <a name="extension-schema"></a>Схема расширения

### <a name="public-configuration"></a>Открытая конфигурация

Ниже приведены все поддерживаемые параметры открытой конфигурации:

* `FileUri`: (необязательно, строка) URI MOF-файла, файла meta MOF или ZIP-файла настраиваемого ресурса.
* `ResourceName`— (необязательно, строка) имя пользовательского модуля ресурсов.
* `ExtensionAction` (необязательный, строчный): указывает, что делает расширения. Допустимые значения: Register, Push, Pull, install и Remove. Если он не указан, по умолчанию он считается действием Push.
* `NodeConfigurationName`: (необязательный, строка) имя конфигурации узла, которую необходимо применить.
* `RefreshFrequencyMins`: (необязательный, int) указывает периодичность (в минутах), с которой DSC пытается получить конфигурацию с опрашивающего сервера.
       Если конфигурация на опрашивающем сервере отличается от текущей на целевом узле, она копируется в ожидающее хранилище и применяется.
* `ConfigurationMode` (необязательный, строчный): указывает, каким образом DSC следует применить конфигурацию. Допустимые значения: ApplyOnly, ApplyAndMonitor и ApplyAndAutoCorrect.
* `ConfigurationModeFrequencyMins` (необязательный, целочисленный): указывает интервал (в минутах), с которым DSC проверяет, что конфигурация находится в требуемом состоянии.

> [!NOTE]
> Если вы используете более раннюю версию, чем 2,3, параметр Mode будет таким же, как и Екстенсионактион. Похоже, что режим является перегруженным термином. Чтобы избежать путаницы, Екстенсионактион используется в версии 2,3. Для обеспечения обратной совместимости, расширение поддерживает как mode, так и ExtensionAction.
>

### <a name="protected-configuration"></a>Защищенная конфигурация

Ниже приведены все поддерживаемые параметры защищенной конфигурации:

* `StorageAccountName`— (необязательно, строка) имя учетной записи хранения, содержащей файл
* `StorageAccountKey`: (необязательный, строковый) ключ учетной записи хранения, содержащей файл
* `RegistrationUrl`(необязательно, строка) URL-адрес учетной записи службы автоматизации Azure.
* `RegistrationKey`(необязательно, строка) ключ доступа учетной записи службы автоматизации Azure.

## <a name="scenarios"></a>Сценарии

### <a name="register-an-azure-automation-account"></a>Регистрация учетной записи службы автоматизации Azure

protected.json
```json
{
  "RegistrationUrl": "<azure-automation-account-url>",
  "RegistrationKey": "<azure-automation-account-key>"
}
```
public.json
```json
{
  "ExtensionAction" : "Register",
  "NodeConfigurationName" : "<node-configuration-name>",
  "RefreshFrequencyMins" : "<value>",
  "ConfigurationMode" : "<ApplyAndMonitor | ApplyAndAutoCorrect | ApplyOnly>",
  "ConfigurationModeFrequencyMins" : "<value>"
}
```

Формат PowerShell
```powershell
$privateConfig = '{
  "RegistrationUrl": "<azure-automation-account-url>",
  "RegistrationKey": "<azure-automation-account-key>"
}'

$publicConfig = '{
  "ExtensionAction" : "Register",
  "NodeConfigurationName": "<node-configuration-name>",
  "RefreshFrequencyMins": "<value>",
  "ConfigurationMode": "<ApplyAndMonitor | ApplyAndAutoCorrect | ApplyOnly>",
  "ConfigurationModeFrequencyMins": "<value>"
}'
```

### <a name="apply-an-mof-configuration-file-in-an-azure-storage-account-to-the-vm"></a>Применение файла конфигурации MOF (в учетной записи хранения Azure) к виртуальной машине

protected.json
```json
{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}
```

public.json
```json
{
  "FileUri": "<mof-file-uri>",
  "ExtensionAction": "Push"
}
```

Формат PowerShell
```powershell
$privateConfig = '{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}'

$publicConfig = '{
  "FileUri": "<mof-file-uri>",
  "ExtensionAction": "Push"
}'
```

### <a name="apply-an-mof-configuration-file-in-public-storage-to-the-vm"></a>Применение файла конфигурации MOF (в общедоступном хранилище) к виртуальной машине

public.json
```json
{
  "FileUri": "<mof-file-uri>"
}
```

Формат PowerShell
```powershell
$publicConfig = '{
  "FileUri": "<mof-file-uri>"
}'
```

### <a name="apply-a-meta-mof-configuration-file-in-an-azure-storage-account-to-the-vm"></a>Применение файла конфигурации meta MOF (в учетной записи хранения Azure) к виртуальной машине

protected.json
```json
{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}
```

public.json
```json
{
  "ExtensionAction": "Pull",
  "FileUri": "<meta-mof-file-uri>"
}
```

Формат PowerShell
```powershell
$privateConfig = '{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}'

$publicConfig = '{
  "ExtensionAction": "Pull",
  "FileUri": "<meta-mof-file-uri>"
}'
```

### <a name="apply-a-meta-mof-configuration-file-in-public-storage-to-the-vm"></a>Применение файла конфигурации Meta MOF (в общедоступном хранилище) к виртуальной машине

public.json

```json
{
  "FileUri": "<meta-mof-file-uri>",
  "ExtensionAction": "Pull"
}
```

Формат PowerShell

```powershell
$publicConfig = '{
  "FileUri": "<meta-mof-file-uri>",
  "ExtensionAction": "Pull"
}'
```

### <a name="install-a-custom-resource-module-a-zip-file-in-an-azure-storage-account-to-the-vm"></a>Установка настраиваемого модуля ресурсов (ZIP-файл в учетной записи хранения Azure) на виртуальную машину

protected.json

```json
{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}
```

public.json

```json
{
  "ExtensionAction": "Install",
  "FileUri": "<resource-zip-file-uri>"
}
```

Формат PowerShell
```powershell
$privateConfig = '{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}'

$publicConfig = '{
  "ExtensionAction": "Install",
  "FileUri": "<resource-zip-file-uri>"
}'
```

### <a name="install-a-custom-resource-module-a-zip-file-in-public-storage-to-the-vm"></a>Установка настраиваемого модуля ресурсов (ZIP-файл в общедоступном хранилище) на виртуальную машину

public.json

```json
{
  "ExtensionAction": "Install",
  "FileUri": "<resource-zip-file-uri>"
}

```

Формат PowerShell

```powershell
$publicConfig = '{
  "ExtensionAction": "Install",
  "FileUri": "<resource-zip-file-uri>"
}'
```

### <a name="remove-a-custom-resource-module-from-the-vm"></a>Удаление пользовательского модуля ресурсов из виртуальной машины

public.json

```json
{
  "ResourceName": "<resource-name>",
  "ExtensionAction": "Remove"
}
```

Формат PowerShell

```powershell
$publicConfig = '{
  "ResourceName": "<resource-name>",
  "ExtensionAction": "Remove"
}'
```

## <a name="template-deployment"></a>Развертывание шаблона

Расширения виртуальной машины Azure можно развернуть с помощью шаблонов Azure Resource Manager. Шаблоны идеально подходят при развертывании одной или нескольких виртуальных машин, для которых требуется настройка после развертывания, например для адаптации к службе автоматизации Azure.

Пример шаблона Resource Manager — [201-dsc-linux-azure-storage-on-ubuntu](https://github.com/Azure/azure-quickstart-templates/tree/master/201-dsc-linux-azure-storage-on-ubuntu) и [201-dsc-linux-public-storage-on-ubuntu](https://github.com/Azure/azure-quickstart-templates/tree/master/201-dsc-linux-public-storage-on-ubuntu).

Дополнительные сведения о шаблоне Azure Resource Manager см. в разделе [authoring Azure Resource Manager Templates](../../azure-resource-manager/templates/template-syntax.md).

## <a name="azure-cli-deployment"></a>Развертывание с помощью Azure CLI

### <a name="use-azure-cliazure-cli"></a>Использование [Azure CLI] [Azure-CLI]

Перед развертыванием расширения Дскфорлинукс настройте `public.json` и в соответствии с `protected.json` различными сценариями в разделе 3.

#### <a name="classic"></a>Классический

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]

Классический режим развертывания также называется режимом управления службами Azure. Его можно включить с помощью команды:

```
$ azure config mode asm
```

Расширение Дскфорлинукс можно развернуть, выполнив команду:

```
$ azure vm extension set <vm-name> DSCForLinux Microsoft.OSTCExtensions <version> \
--private-config-path protected.json --public-config-path public.json
```

Следующая команда позволяет узнать последнюю версию расширения:

```
$ azure vm extension list
```

#### <a name="resource-manager"></a>Resource Manager

Чтобы переключиться в режим Azure Resource Manager, выполните команду:

```
$ azure config mode arm
```

Расширение Дскфорлинукс можно развернуть, выполнив команду:

```
$ azure vm extension set <resource-group> <vm-name> \
DSCForLinux Microsoft.OSTCExtensions <version> \
--private-config-path protected.json --public-config-path public.json
```

> [!NOTE]
> В Azure Resource Manager режиме `azure vm extension list` сейчас недоступен.
>

### <a name="use-azure-powershellazure-powershell"></a>Использование [Azure PowerShell] [Azure-PowerShell]

#### <a name="classic"></a>Классический

Вы можете войти в учетную запись Azure в режиме управления службами Azure, выполнив команду:

```powershell
Add-AzureAccount
```

И разверните расширение Дскфорлинукс, выполнив команду:

```powershell
$vmname = '<vm-name>'
$vm = Get-AzureVM -ServiceName $vmname -Name $vmname
$extensionName = 'DSCForLinux'
$publisher = 'Microsoft.OSTCExtensions'
$version = '< version>'
```

Измените содержимое $privateConfig и $publicConfig в соответствии с различными сценариями, описанными в предыдущем разделе.

```
$privateConfig = '{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}'
```

```
$publicConfig = '{
  "ExtensionAction": "Push",
  "FileUri": "<mof-file-uri>"
}'
```

```powershell
Set-AzureVMExtension -ExtensionName $extensionName -VM $vm -Publisher $publisher `
  -Version $version -PrivateConfiguration $privateConfig `
  -PublicConfiguration $publicConfig | Update-AzureVM
```

#### <a name="resource-manager"></a>Resource Manager

Вы можете войти в учетную запись Azure в Azure Resource Managerном режиме, выполнив команду:

```powershell
Login-AzAccount
```

Дополнительные сведения об использовании Azure PowerShell с Azure Resource Manager см. в статье [Управление ресурсами Azure с помощью Azure PowerShell](../../azure-resource-manager/management/manage-resources-powershell.md).

Расширение Дскфорлинукс можно развернуть, выполнив команду:

```powershell
$rgName = '<resource-group-name>'
$vmName = '<vm-name>'
$location = '< location>'
$extensionName = 'DSCForLinux'
$publisher = 'Microsoft.OSTCExtensions'
$version = '< version>'
```

Измените содержимое $privateConfig и $publicConfig в соответствии с различными сценариями, описанными в предыдущем разделе.

```
$privateConfig = '{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}'
```

```
$publicConfig = '{
  "ExtensionAction": "Push",
  "FileUri": "<mof-file-uri>"
}'
```

```powershell
Set-AzVMExtension -ResourceGroupName $rgName -VMName $vmName -Location $location `
  -Name $extensionName -Publisher $publisher -ExtensionType $extensionName `
  -TypeHandlerVersion $version -SettingString $publicConfig -ProtectedSettingString $privateConfig
```

## <a name="troubleshoot-and-support"></a>Устранение неполадок и поддержка

### <a name="troubleshoot"></a>Диагностика

Данные о состоянии развертывания расширения можно получить на портале Azure, а также с помощью Azure CLI. Чтобы просмотреть состояние развертывания расширений для данной виртуальной машины, выполните следующую команду с помощью Azure CLI.

```azurecli
az vm extension list --resource-group myResourceGroup --vm-name myVM -o table
```

Выходные данные выполнения расширения регистрируются в следующем файле:

```
/var/log/azure/<extension-name>/<version>/extension.log file.
```

Код ошибки: 51 представляет неподдерживаемое распространение или неподдерживаемое действие расширения.
В некоторых случаях расширение DSC для Linux не может установить OMI, если на компьютере уже существует более поздняя версия OMI. [error response: (000003)Downgrade not allowed] (сообщение об ошибке: (000003) откат на более старую версию не разрешен)

### <a name="support"></a>Поддержка

Если в любой момент при изучении этой статьи вам потребуется дополнительная помощь, обратитесь к экспертам по Azure на [форумах MSDN Azure и Stack Overflow](https://azure.microsoft.com/support/community/). Кроме того, можно зафайлировать обращение в службу поддержки Azure. Перейдите на [сайт поддержки Azure](https://azure.microsoft.com/support/options/)и выберите **получить поддержку**. Сведения об использовании службы поддержки Azure см. в статье [часто задаваемые вопросы о поддержке Microsoft Azure](https://azure.microsoft.com/support/faq/).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о расширениях виртуальных машин см. в обзоре [расширений и компонентов виртуальной машины для Linux](features-linux.md).
