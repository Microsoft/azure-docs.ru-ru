---
title: Общие сведения о настройке требуемого состояния для Azure
description: Узнайте, как использовать обработчик расширений Microsoft Azure для настройки требуемого состояния (DSC). В статье приведены необходимые компоненты, архитектура и командлеты.
services: virtual-machines
documentationcenter: ''
author: mgoedtel
manager: evansma
editor: ''
tags: azure-resource-manager
keywords: DSC
ms.assetid: bbacbc93-1e7b-4611-a3ec-e3320641f9ba
ms.service: virtual-machines
ms.subservice: extensions
ms.collection: windows
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: na
ms.date: 07/13/2020
ms.author: magoedte
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: dcdc325633aff5ab828cb1c82f4bb2d8becee967
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102560044"
---
# <a name="introduction-to-the-azure-desired-state-configuration-extension-handler"></a>Общие сведения об обработчике расширения Desired State Configuration в Azure

Агент виртуальной машины Azure и связанные расширения являются частью служб инфраструктуры Microsoft Azure. Расширения виртуальной машины — это программные компоненты, которые расширяют функциональные возможности виртуальной машины и упрощают различные операции управления ею.

Основным вариантом использования расширения Azure Desired State Configuration (DSC) является начальная загрузка виртуальной машины в [службу настройки состояния службы автоматизации Azure (DSC)](../../automation/automation-dsc-overview.md).
Служба предоставляет [преимущества](/powershell/scripting/dsc/managing-nodes/metaConfig#pull-service) , которые включают постоянное управление КОНФИГУРАЦИЕЙ виртуальной машины и интеграцию с другими операционными средствами, такими как мониторинг Azure.
Использование расширения для регистрации виртуальной машины в службе предоставляет гибкое решение, которое даже работает в рамках подписок Azure.

Расширение DSC можно использовать независимо от службы Automation DSC.
Тем не менее, это приведет к отправке конфигурации только на виртуальную машину.
На виртуальной машине не доступны текущие отчеты, кроме локального.

В этой статье предоставлены сведения для двух сценариев — расширения DSC для подключения службы автоматизации и использования расширения DSC как инструмента для назначения конфигураций виртуальным машинам с помощью пакета SDK Azure.

## <a name="prerequisites"></a>Предварительные условия

- **Локальный компьютер.** Для взаимодействия с расширением виртуальной машины Azure нужно использовать портал Azure или пакет SDK для Azure PowerShell.
- **Гостевой агент.** Виртуальная машина Azure, настроенная с помощью конфигурации DSC, должна работать под управлением ОС, которая поддерживает Windows Management Framework (WMF) версии 4.0 или более поздней. Полный список поддерживаемых версий ОС см. в [журнале версий расширения DSC](../../automation/automation-dsc-extension-history.md).

## <a name="terms-and-concepts"></a>Термины и основные понятия

Для изучения этого руководства нужно знать приведенные ниже понятия.

- **Конфигурация**. Документ конфигурации DSC.
- **Узел**. Целевой объект для конфигурации DSC. В этом документе *узел* всегда ссылается на виртуальную машину Azure.
- **Данные конфигурации**. PSD1-файл, содержащий данные среды для конфигурации.

## <a name="architecture"></a>Архитектура

Расширение DSC Azure использует платформу агента Azure, чтобы доставлять и применять конфигурации DSC виртуальных машин Azure, а также сообщать об этих конфигурациях. Расширение DSC принимает документ конфигурации и набор параметров. Если файл не предоставлен, внедряется [скрипт конфигурации по умолчанию](#default-configuration-script) с расширением. Скрипт конфигурации по умолчанию используется только для задания метаданных в [локальном диспетчере конфигураций](/powershell/scripting/dsc/managing-nodes/metaConfig).

Когда расширение вызывается впервые, с его помощью устанавливается определенная версия WMF, следуя приведенной ниже логике:

- Если виртуальная машина Azure работает под управлением Windows Server 2016, никакие действия не требуются, потому что в Windows Server 2016 уже установлена последняя версия PowerShell.
- Если указано свойство **wmfVersion**, то устанавливается соответствующая версия WMF (за исключением случаев, когда эта версия несовместима с ОС виртуальной машины).
- Если свойство **wmfVersion** не указано, то устанавливается последняя применимая версия WMF.

Для установки WMF требуется перезагрузка. После перезапуска расширения скачивается ZIP-файл, указанный в свойстве **modulesUrl**, если оно предоставлено. Если это расположение находится в хранилище BLOB-объектов Azure, то для доступа к файлу в свойстве **sasToken** можно указать маркер SAS. После скачивания и распаковки ZIP-файла функция настройки, определенная в **configurationFunction** , запускает файл. mof ([MOF-файл](/windows/win32/wmisdk/managed-object-format--mof-)). Затем расширение выполняет командлет `Start-DscConfiguration -Force` с созданным MOF-файлом. Расширение фиксирует выходные данные и записывает их в канал состояний Azure.

### <a name="default-configuration-script"></a>Скрипт конфигурации по умолчанию

Расширение DSC Azure включает скрипт конфигурации по умолчанию, предназначенный для использования при подключении виртуальной машины к DSC службы автоматизации Azure. Параметры сценария сопоставляются с настраиваемыми свойствами [локального диспетчера конфигураций](/powershell/scripting/dsc/managing-nodes/metaConfig). Параметры скрипта см. в разделе [Скрипт конфигурации по умолчанию](dsc-template.md#default-configuration-script) статьи [Расширение Desired State Configuration (DSC) с использованием шаблонов Azure Resource Manager](dsc-template.md). Полный скрипт см. в [шаблоне быстрого запуска Azure на GitHub](https://github.com/Azure/azure-quickstart-templates/blob/master/dsc-extension-azure-automation-pullserver/UpdateLCMforAAPull.zip?raw=true).

## <a name="information-for-registering-with-azure-automation-state-configuration-dsc-service"></a>Сведения для регистрации в службе настройки состояния службы автоматизации Azure (DSC)

При использовании расширения DSC для регистрации узла в службе настройки состояния потребуется указать три значения.

- Регистратионурл — HTTPS-адрес учетной записи службы автоматизации Azure.
- RegistrationKey — общий секрет, используемый для регистрации узлов в службе.
- NodeConfigurationName — имя конфигурации узла (MOF), которую необходимо извлечь из службы для настройки роли сервера.

Эти сведения можно просмотреть в портал Azure или с помощью PowerShell.

```powershell
(Get-AzAutomationRegistrationInfo -ResourceGroupName <resourcegroupname> -AutomationAccountName <accountname>).Endpoint
(Get-AzAutomationRegistrationInfo -ResourceGroupName <resourcegroupname> -AutomationAccountName <accountname>).PrimaryKey
```

В качестве имени конфигурации узла убедитесь, что конфигурация узла существует в конфигурации состояния Azure.  В противном случае развертывание расширения вернет ошибку.  Также убедитесь, что вы используете имя *конфигурации узла* , а не конфигурацию.
Конфигурация определяется в скрипте, который используется [для компиляции конфигурации узла (MOF-файл)](../../automation/automation-dsc-compile.md).
Имя всегда будет содержать конфигурацию, за которой следует точка `.` , а `localhost` также либо конкретное имя компьютера.

## <a name="dsc-extension-in-resource-manager-templates"></a>Использование расширения DSC в шаблонах Resource Manager

В большинстве сценариев для работы с расширением DSC используются шаблоны развертывания Azure Resource Manager. Ознакомиться с дополнительными сведениями и примерами включения расширения DSC в шаблоны развертывания Azure Resource Manager можно с помощью статьи [Расширение Desired State Configuration (DSC) с использованием шаблонов Azure Resource Manager](dsc-template.md).

## <a name="dsc-extension-powershell-cmdlets"></a>Командлеты PowerShell для расширения DSC

Командлеты PowerShell, которые используются для управления расширением DSC, лучше подходят для сценариев интерактивного устранения неполадок и сбора данных. Эти командлеты можно использовать для упаковки, публикации и отслеживания развертываний расширения DSC. Командлеты для расширения DSC еще не обновлены для работы со [сценарием конфигурации по умолчанию](#default-configuration-script).

Командлет **Publish-AzVMDscConfiguration** принимает файл конфигурации, сканирует его на наличие ресурсов, зависимых от DSC, а затем создает ZIP-файл. ZIP-файл содержит конфигурацию и ресурсы DSC, которые необходимы для применения конфигурации. Командлет также может создать пакет локально, используя параметр *-OutputArchivePath*, или опубликовать ZIP-файл в хранилище BLOB-объектов Azure, а затем защитить его маркером SAS.

На корневом уровне папки архива у ZIP-файла, созданного этим командлетом, есть скрипт конфигурации в формате PS1. Предназначенная для ресурсов папка модуля находится в папке архива.

Командлет **Set-AzVMDscExtension** внедряет параметры, которые необходимы расширению DSC PowerShell, в объект конфигурации виртуальной машины.

Командлет **Get-AzVMDscExtension** извлекает состояние расширения DSC определенной виртуальной машины.

Командлет **Get-AzVMDscExtensionStatus** извлекает состояние конфигурации DSC, которое применил обработчик расширения DSC. Это действие может выполняться с одной виртуальной машиной или с группой виртуальных машин.

Командлет **Remove-AzVMDscExtension** удаляет обработчик расширения из определенной виртуальной машины. Этот командлет *не* приводит к удалению конфигурации, удалению WMF или изменению примененных параметров на виртуальной машине. Удаляется только обработчик расширений.

Важная информация о командлетах расширения DSC для Resource Manager:

- Командлеты Azure Resource Manager выполняются синхронно.
- *ResourceGroupName*, *VMName*, *ArchiveStorageAccountName*, *Version*, *Location* — обязательные параметры.
- *ArchiveResourceGroupName* — необязательный параметр. Этот параметр можно указать, если ваша учетная запись хранения не принадлежит к той группе ресурсов, в которой создана виртуальная машина.
- Параметр **AutoUpdate** позволяет автоматически обновлять обработчик расширений до последней версии сразу, когда она становится доступной. После выпуска новой версии WMF возможна перезагрузка виртуальной машины из-за данного параметра.

### <a name="get-started-with-cmdlets"></a>Начало работы с командлетами

Расширение DSC Azure может использовать документы конфигурации DSC для непосредственной настройки виртуальных машин Azure во время развертывания. Этот шаг не регистрирует узел для службы автоматизации. Этот узел *не* управляется централизовано.

Ниже показан пример простой конфигурации. Сохраните конфигурацию локально как iisInstall.ps1.

```powershell
configuration IISInstall
{
    node "localhost"
    {
        WindowsFeature IIS
        {
            Ensure = "Present"
            Name = "Web-Server"
        }
    }
}
```

Следующие команды размещают скрипт iisInstall.ps1 на указанной виртуальной машине. а также выполняют конфигурацию и возвращают сведения о состоянии.

```powershell
$resourceGroup = 'dscVmDemo'
$vmName = 'myVM'
$storageName = 'demostorage'
#Publish the configuration script to user storage
Publish-AzVMDscConfiguration -ConfigurationPath .\iisInstall.ps1 -ResourceGroupName $resourceGroup -StorageAccountName $storageName -force
#Set the VM to run the DSC configuration
Set-AzVMDscExtension -Version '2.76' -ResourceGroupName $resourceGroup -VMName $vmName -ArchiveStorageAccountName $storageName -ArchiveBlobName 'iisInstall.ps1.zip' -AutoUpdate -ConfigurationName 'IISInstall'
```

## <a name="azure-cli-deployment"></a>Развертывание с помощью Azure CLI

Azure CLI можно использовать для развертывания расширения DSC на существующей виртуальной машине.

Для виртуальной машины под Windows:

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name Microsoft.Powershell.DSC \
  --publisher Microsoft.Powershell \
  --version 2.77 --protected-settings '{}' \
  --settings '{}'
```

Для виртуальной машины под управлением Linux:

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name DSCForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 2.7 --protected-settings '{}' \
  --settings '{}'
```

## <a name="azure-portal-functionality"></a>Использование портала Azure

Для настройки DSC на портале:

1. Перейдите к виртуальной машине.
2. В разделе **Параметры** выберите **Расширения**.
3. На новой странице выберите **+Добавить**, а затем — **Настройка требуемого состояния Windows PowerShell**.
4. В нижней части страницы сведений о расширении нажмите кнопку **Создать**.

Портал собирает следующие входные данные.

- **Configuration Modules or Script** (Модули конфигурации или скрипт). Это поле является обязательным (для [скрипта конфигурации по умолчанию](#default-configuration-script) форма не была обновлена). Модулям и скриптам конфигурации требуется PS1-файл, содержащий скрипт конфигурации, или ZIP-файл со скриптом конфигурации PS1 в корне. При использовании ZIP-файла все зависимые ресурсы должны быть включены в его папки модуля. ZIP-файл можно создать с помощью командлета **Publish-AzureVMDscConfiguration -OutputArchivePath**, включенного в пакет SDK для Azure PowerShell. Этот ZIP-файл передается в хранилище BLOB-объектов пользователя и защищается маркером SAS.

- **Полное имя модуля конфигурации**. в PS1-файл можно включить несколько функций настройки. Введите имя сценария конфигурации (в формате PS1), суффикс \\, а затем — имя функции конфигурации. Например, если PS1-файл скрипта называется configuration.ps1, а имя конфигурации — **IisInstall**, введите **configuration.ps1\IisInstall**.

- **Configuration Arguments** (Аргументы конфигурации). Если функция конфигурации принимает аргументы, введите их здесь в формате **argumentName1=value1,argumentName2=value2**. Этот формат отличается от того, в котором аргументы конфигурации принимаются в командлетах PowerShell или шаблонах Resource Manager.

- **Файл PSD1 данных конфигурации**: Если для вашей конфигурации требуется файл данных конфигурации в `.psd1` , используйте это поле, чтобы выбрать файл данных и передать его в хранилище BLOB-объектов пользователя. где он будет защищен маркером SAS.

- **WMF Version** (Версия WMF). Указывает версию Windows Management Framework (WMF), которую необходимо установить на виртуальной машине. Если задать для этого свойства значение latest, будет установлена последняя версия WMF. В настоящее время для этого свойства доступны только такие значения: 4.0, 5.0, 5.1 и latest. Возможные значения зависят от обновлений. По умолчанию используется значение **latest**.

- **Data Collection** (Сбор данных). Определяет, собирает ли расширение данные телеметрии. Дополнительные сведения см. в записи блога [Azure DSC Extension Data Collection](https://devblogs.microsoft.com/powershell/azure-dsc-extension-data-collection-2/) (Коллекция данных расширения DSC Azure).

- **Version** (Версия). Указывает версию устанавливаемого расширения DSC. Сведения о версиях приведены в [журнале версий расширения DSC](/powershell/scripting/dsc/getting-started/azuredscexthistory).

- **Auto Upgrade Minor Version** (Автоматическое обновление до дополнительного номера версии). Это поле сопоставляется с параметром **AutoUpdate** в командлетах и позволяет расширению выполнять автоматическое обновление до последней версии во время установки. Если указать значение **Yes** (Да), то обработчик расширений будет использовать последнюю версию. Если указать значение **No** (Нет), то будет установлена версия, указанная в свойстве **Version** (Версия). Если не выбрано ни значение **Yes** (Да), ни значение **No** (Нет), это аналогично выбору значения **No** (Нет).

## <a name="logs"></a>Журналы

Журналы для расширения хранятся в следующем расположении: `C:\WindowsAzure\Logs\Plugins\Microsoft.Powershell.DSC\<version number>`.

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о PowerShell DSC см. в [центре документации по PowerShell](/powershell/scripting/dsc/overview/overview).
- Изучите [шаблон Resource Manager для расширения DSC](dsc-template.md).
- Дополнительные функциональные возможности, которыми можно управлять с помощью PowerShell DSC, и ресурсы по DSC см. в [коллекции PowerShell](https://www.powershellgallery.com/packages?q=DscResource&x=0&y=0).
- Сведения о передаче конфиденциальных параметров в конфигурации см. в статье о [безопасном управлении учетными данными с помощью обработчика расширения DSC](dsc-credentials.md).
