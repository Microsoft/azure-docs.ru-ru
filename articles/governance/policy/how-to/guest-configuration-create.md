---
title: Создание политик гостевой конфигурации для Windows
description: Узнайте, как создать политику гостевой конфигурации в службе "Политика Azure" для Windows.
ms.date: 08/17/2020
ms.topic: how-to
ms.openlocfilehash: 72772743eba23ea7c2a93f5037ac84b671256a66
ms.sourcegitcommit: a67b972d655a5a2d5e909faa2ea0911912f6a828
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104887705"
---
# <a name="how-to-create-guest-configuration-policies-for-windows"></a>Создание политик гостевой конфигурации для Windows

Перед созданием определений пользовательских политик мы рекомендуем ознакомиться с общими сведениями, представленными на странице [гостевой конфигурации в службе "Политика Azure"](../concepts/guest-configuration.md).
 
Дополнительные сведения о создании политик гостевой конфигурации для Linux см. на странице [Создание политик гостевой конфигурации для Linux](./guest-configuration-create-linux.md).

При аудите системы Windows гостевая конфигурация применяет модуль ресурсов [Desired State Configuration (DSC)](/powershell/scripting/dsc/overview/overview) для создания файла конфигурации. Эта конфигурация DSC определяет состояние, в котором должен находиться компьютер. Если оценка конфигурации проходит неудачно, то срабатывает политика **auditIfNotExists** и компьютер считается **несоответствующим**.

[Гостевую конфигурацию в службе "Политика Azure"](../concepts/guest-configuration.md) можно использовать только для проверки параметров на компьютерах. Исправление параметров на компьютерах пока недоступно.

Используйте следующие действия, чтобы создать собственную конфигурацию для проверки состояния виртуальной машины Azure или виртуальной машины, отличной от Azure.

> [!IMPORTANT]
> Определения настраиваемых политик с гостевой конфигурацией в средах Azure для государственных организаций и Azure для Китая — это предварительная версия функции.
>
> Для аудита виртуальных машин Azure требуется расширение гостевой конфигурации.
> Чтобы развернуть расширение в масштабе на всех компьютерах Windows, назначьте следующие определения политик: `Deploy prerequisites to enable Guest Configuration Policy on Windows VMs`
> 
> Не используйте секреты или конфиденциальные сведения в пакетах с нестандартным содержимым.

## <a name="install-the-powershell-module"></a>Установка модуля PowerShell

Модуль гостевой конфигурации автоматизирует процесс создания пользовательского содержимого, в том числе:

- создание артефакта содержимого гостевой конфигурации (ZIP-файл);
- автоматическое тестирование артефакта;
- создание определения политики;
- публикация политики.

Модуль можно установить на компьютере под управлением ОС Windows, macOS или Linux с помощью локально установленной среды PowerShell 6.2 или более поздней версии, либо [Azure Cloud Shell](https://shell.azure.com), а также с помощью [основного образа Docker для Azure PowerShell](https://hub.docker.com/r/azuresdk/azure-powershell-core).

> [!NOTE]
> Компиляция конфигураций в Windows пока не поддерживается.

### <a name="base-requirements"></a>Основные требования

Операционные системы, в которых можно установить модуль:

- Linux
- macOS
- Windows

Для модуля ресурсов гостевой конфигурации требуется следующее программное обеспечение:

- PowerShell 6.2 или более поздней версии. Если она еще не установлена, выполните [эти инструкции](/powershell/scripting/install/installing-powershell).
- Azure PowerShell 1.5.0 или более поздней версии. Если она еще не установлена, выполните [эти инструкции](/powershell/azure/install-az-ps).
  - Требуются только модули AZ "az. Accounts" и "az. Resources".

### <a name="install-the-module"></a>Установка модуля

Порядок установки модуля **GuestConfiguration** в PowerShell

1. В командной строке PowerShell выполните следующую команду:

   ```azurepowershell-interactive
   # Install the Guest Configuration DSC resource module from PowerShell Gallery
   Install-Module -Name GuestConfiguration
   ```

1. Убедитесь, что модуль успешно импортирован.

   ```azurepowershell-interactive
   # Get a list of commands for the imported GuestConfiguration module
   Get-Command -Module 'GuestConfiguration'
   ```

## <a name="guest-configuration-artifacts-and-policy-for-windows"></a>Артефакты и политика гостевой конфигурации для Windows

Гостевая конфигурация использует PowerShell Desired State Configuration в качестве абстракции языка для описания правил аудита в Windows. Агент загружает отдельный экземпляр PowerShell 6,2, поэтому не возникает конфликтов с работой PowerShell DSC в Windows PowerShell 5.1 и нет необходимости предварительно устанавливать PowerShell 6.2 или более поздней версии.

Общие сведения о понятиях и терминологии DSC см. в статье [Общие сведения о службе настройки требуемого состояния Windows PowerShell](/powershell/scripting/dsc/overview/overview).

### <a name="how-guest-configuration-modules-differ-from-windows-powershell-dsc-modules"></a>Отличия модулей гостевой конфигурации от модулей Windows PowerShell DSC

Когда гостевая конфигурация выполняет аудит компьютера, последовательность событий отличается от последовательности в Windows PowerShell DSC.

1. Сначала агент выполняет `Test-TargetResource`, чтобы определить, находится ли конфигурация в правильном состоянии.
1. Логическое значение, возвращаемое этой функцией, определяет состояние соответствия для назначения гостя в Azure Resource Manager.
1. Поставщик выполняет `Get-TargetResource` для получения текущего состояния каждого параметра, чтобы получить сведения о том, почему компьютер не соответствует требованиям, и убедиться, что текущее состояние соответствует требованиям.

Параметры в политике Azure, которые передают значения для назначений гостевой конфигурации, должны иметь _строковый_ тип. Передача массивов через параметры невозможна, даже если ресурс DSC поддерживает массивы.

### <a name="get-targetresource-requirements"></a>Требования Get-TargetResource

Функция `Get-TargetResource` имеет особые требования к гостевой конфигурации, которые не действовали для Desired State Configuration в Windows.

- Возвращаемая хэш-таблица должна включать свойство с именем **Reasons**.
- Свойство Reasons должно быть массивом.
- Каждый элемент в этом массиве должен быть хэш-таблицей с именами ключей **Code** и **Phrase**.

Свойство Reasons используется службой для стандартизации представления информации о причинах несоответствия компьютера. Каждый элемент массива Reasons можно считать конкретной причиной, которая не позволяет считать ресурс соответствующим. Это свойство должно быть массивом, так как может существовать сразу несколько причин, из-за которых ресурс не соответствует требованиям.

Служба ожидает получить свойства **Code** и **Phrase**. При создании пользовательского ресурса введите текст (обычно используется вывод stdout), который будет отображаться в виде значения **Phrase** в качестве причины несоответствия для этого ресурса. **Code** имеет требования к форматированию, которые позволяют четко отображать в отчетах сведения о ресурсе, используемом для аудита. Это решение делает гостевую конфигурацию расширяемой. Вы можете выполнять любую команду, которая поддерживает вывод выходных данных в виде строкового значения для свойства **Phrase**.

- **Code** (строка). Имя ресурса, затем повтор имени и короткое имя без пробелов в качестве идентификатора причины. Все три значения должны разделяться двоеточиями и не содержать пробелов.
  - Например, так: `registry:registry:keynotpresent`
- **Phrase** (строка). Понятный текст, описывающий причину несоответствия для параметра.
  - Например, так: `The registry key $key is not present on the machine.`

```powershell
$reasons = @()
$reasons += @{
  Code = 'Name:Name:ReasonIdentifer'
  Phrase = 'Explain why the setting is not compliant'
}
return @{
    reasons = $reasons
}
```

Свойство reasons необходимо добавить в MOF-файл схемы для ресурса в качестве внедренного класса.

```mof
[ClassVersion("1.0.0.0")] 
class Reason
{
    [Read] String Phrase;
    [Read] String Code;
};

[ClassVersion("1.0.0.0"), FriendlyName("ResourceName")]
class ResourceName : OMI_BaseResource
{
    [Key, Description("Example description")] String Example;
    [Read, EmbeddedInstance("Reason")] String Reasons[];
};
```

Если ресурс содержит обязательные свойства, они также должны возвращаться `Get-TargetResource` в параллельном режиме с `reasons` классом. Если `reasons` он не включен, служба включает поведение "Catch-All", которое сравнивает входные значения со значениями `Get-TargetResource` и значения, возвращаемые методом `Get-TargetResource` , и предоставляет подробное сравнение в виде `reasons` .

### <a name="configuration-requirements"></a>Требования настройки

Имя настраиваемой конфигурации должно быть одинаковым везде. Имя ZIP-файла для пакета содержимого, имя конфигурации в MOF-файле и имя назначения гостя в шаблоне Azure Resource Manager (шаблон ARM) должны быть одинаковыми.

### <a name="policy-requirements"></a>Требования политики

Раздел определения политики `metadata` должен включать два свойства службы настройки гостевой системы для автоматизации подготовки и создания отчетов о назначениях гостевой конфигурации. `category`Для свойства необходимо задать значение "Гостевая конфигурация", а раздел с именем `Guest Configuration` должен содержать сведения о назначении гостевой конфигурации. `New-GuestConfigurationPolicy`Командлет создает этот текст автоматически.
См. пошаговые инструкции на этой странице.

В следующем примере демонстрируется `metadata` раздел.

```json
    "metadata": {
      "category": "Guest Configuration",
      "guestConfiguration": {
        "name": "test",
        "version": "1.0.0",
        "contentType": "Custom",
        "contentUri": "CUSTOM-URI-HERE",
        "contentHash": "CUSTOM-HASH-VALUE-HERE",
        "configurationParameter": {}
      }
    },
```

### <a name="scaffolding-a-guest-configuration-project"></a>Формирование шаблонов для проекта гостевой конфигурации

Разработчики, которые хотят ускорить процесс начала работы и применить готовый пример кода, могут установить проект, предоставленным сообществом, с именем **Guest Configuration Project** (Проект гостевой конфигурации). Этот проект устанавливает шаблон для модуля PowerShell [Plaster](https://github.com/powershell/plaster). Это средство можно использовать для формирования шаблона проекта, в который входят рабочая конфигурация, пример ресурса и набор тестов [Pester](https://github.com/pester/pester) для проверки проекта. Также шаблон включает запускатели задач для Visual Studio Code, позволяющие автоматизировать создание и проверку пакета гостевой конфигурации. Дополнительные сведения см. на странице проекта [Guest Configuration Project](https://github.com/microsoft/guestconfigurationproject) на сайте GitHub.

Дополнительные сведения о работе с конфигурациями в целом см. в статье [Создание, компиляция и применение конфигурации](/powershell/scripting/dsc/configurations/write-compile-apply-configuration).

### <a name="expected-contents-of-a-guest-configuration-artifact"></a>Ожидаемое содержимое артефакта гостевой конфигурации

Готовый пакет используется гостевой конфигурацией для создания определений в службе "Политика Azure". В этот пакет входит следующее:

- Скомпилированная конфигурация DSC в MOF-файле.
- Папка для модулей
  - модуль GuestConfiguration;
  - модуль DscNativeResources;
  - (Windows) модули ресурса DSC, которые требуются для MOF-файла.

В создании пакета вам помогут командлеты PowerShell.
Папка корневого уровня или папка версии не требуются.
Формат пакета должен быть ZIP-файлом и не может превышать общий размер 100 МБ при несжатом виде.

### <a name="storing-guest-configuration-artifacts"></a>Хранение артефактов гостевой конфигурации

ZIP-пакет должен храниться в расположении, доступном для управляемых виртуальных машин.
Это может быть репозиторий GitHub, репозиторий в Azure Repos или служба хранилища Azure. Если вы предпочитаете, чтобы этот пакет не был общедоступным, можно включить в URL-адрес [маркер SAS](../../../storage/common/storage-sas-overview.md). Можно также реализовать [конечную точку службы](../../../storage/common/storage-network-security.md#grant-access-from-a-virtual-network) для компьютеров в частной сети. Хотя такая конфигурация применяется только для доступа к пакету, а не для взаимодействия с самой службой.

## <a name="step-by-step-creating-a-custom-guest-configuration-audit-policy-for-windows"></a>Пошаговые инструкции по созданию пользовательской политики аудита гостевой конфигурации для Windows

Создайте конфигурацию DSC для аудита параметров. В следующем примере скрипта PowerShell создается конфигурация с именем **AuditBitLocker**, импортирует модуль ресурсов **PsDscResources** и применяет ресурс `Service` для аудита работающей службы. Скрипт конфигурации можно выполнить на компьютере под управлением Windows или macOS.

```powershell
# Add PSDscResources module to environment
Install-Module 'PSDscResources'

# Define the DSC configuration and import GuestConfiguration
Configuration AuditBitLocker
{
    Import-DscResource -ModuleName 'PSDscResources'

    Node AuditBitlocker {
      Service 'Ensure BitLocker service is present and running'
      {
          Name = 'BDESVC'
          Ensure = 'Present'
          State = 'Running'
      }
    }
}

# Compile the configuration to create the MOF files
AuditBitLocker
```

Запустите этот скрипт в терминале PowerShell или сохраните этот файл с именем `config.ps1` в папке проекта.
Запустите его в PowerShell, выполнив команду `./config.ps1` в терминале. Создастся MOF-файл.

Команда `Node AuditBitlocker` не является технически необходимой, но она создает файл с именем `AuditBitlocker.mof`, а не с именем по умолчанию `localhost.mof`. Если имя MOF-файла соответствует конфигурации, вам будет проще упорядочивать множество файлов при работе в большом масштабе.

После компиляции MOF-файла нужно упаковать вместе все вспомогательные файлы. Готовый пакет используется гостевой конфигурацией для создания определений в службе "Политика Azure".

Командлет `New-GuestConfigurationPackage` создает пакет. Модули, необходимые для этой конфигурации, должны быть доступны в `$Env:PSModulePath`. Командлету `New-GuestConfigurationPackage` при создании содержимого Windows передаются следующие параметры:

- **Name** (Имя). Имя пакета гостевой конфигурации.
- **Конфигурация.** Полный путь к скомпилированному документу конфигурации DSC.
- **Путь**. Путь к выходной папке. Это необязательный параметр. Если этот параметр не указан, пакет создается в текущем каталоге.

Выполните следующую команду, чтобы создать пакет с использованием конфигурации, заданной на предыдущем шаге.

```azurepowershell-interactive
New-GuestConfigurationPackage `
  -Name 'AuditBitlocker' `
  -Configuration './AuditBitlocker/AuditBitlocker.mof'
```

После создания пакета конфигурации, но перед его публикацией в Azure можно протестировать пакет с рабочей станции или среды непрерывной интеграции и непрерывного развертывания (CI/CD). Командлет гостевой конфигурации `Test-GuestConfigurationPackage` включает в среду разработки тот же агент, который используется на компьютерах Azure. С помощью этого решения можно выполнить локальное тестирование интеграции перед выпуском в платных облачных средах.

Поскольку агент фактически оценивает локальную среду, в большинстве случаев командлет тестирования следует выполнять на той же платформе операционной системы, на которой вы планируете проводить аудит. При тестировании используются только те модули, которые включены в пакет содержимого.

Командлет `Test-GuestConfigurationPackage` принимает следующие параметры:

- **Name** (Имя). Имя политики гостевой конфигурации.
- **Parameter**. Параметры политики, указанные в формате хэш-таблицы.
- **Путь**. Полный путь к пакету гостевой конфигурации.

Выполните следующую команду, чтобы протестировать пакет, созданный на предыдущем шаге.

```azurepowershell-interactive
Test-GuestConfigurationPackage `
  -Path ./AuditBitlocker.zip
```

Этот командлет также поддерживает ввод данных из конвейера PowerShell. Передайте выходные данные командлета `New-GuestConfigurationPackage` в командлет `Test-GuestConfigurationPackage`.

```azurepowershell-interactive
New-GuestConfigurationPackage -Name AuditBitlocker -Configuration ./AuditBitlocker/AuditBitlocker.mof | Test-GuestConfigurationPackage
```

Следующим шагом является публикация файла в хранилище BLOB-объектов Azure. Никаких особых требований к учетной записи хранения не существует, но рекомендуется размещать файл в регионе рядом с вашими компьютерами. Если у вас нет учетной записи хранения, используйте следующий пример. Для приведенных ниже команд, включая `Publish-GuestConfigurationPackage` , требуется `Az.Storage` модуль.

```azurepowershell-interactive
# Creates a new resource group, storage account, and container
New-AzResourceGroup -name myResourceGroupName -Location WestUS
New-AzStorageAccount -ResourceGroupName myResourceGroupName -Name myStorageAccountName -SkuName 'Standard_LRS' -Location 'WestUs' | New-AzStorageContainer -Name guestconfiguration -Permission Blob
```

Командлет `Publish-GuestConfigurationPackage` принимает следующие параметры:

- **Путь**: расположение публикуемого пакета
- **ResourceGroupName**: имя группы ресурсов, в которой находится учетная запись хранения.
- **StorageAccountName**: имя учетной записи хранения, в которую должен быть опубликован пакет
- **StorageContainerName**: (по умолчанию: *гуестконфигуратион*) имя контейнера хранилища в учетной записи хранения.
- **Force**: перезаписать существующий пакет в учетной записи хранения с тем же именем

В приведенном ниже примере пакет публикуется в имя контейнера хранилища "гуестконфигуратион".

```azurepowershell-interactive
Publish-GuestConfigurationPackage -Path ./AuditBitlocker.zip -ResourceGroupName myResourceGroupName -StorageAccountName myStorageAccountName
```

После создания и отправки пакета настраиваемой политики гостевой конфигурации создайте определение политики гостевой конфигурации. Командлет `New-GuestConfigurationPolicy` принимает пакет настраиваемой политики и создает определение политики.

Командлет `New-GuestConfigurationPolicy` принимает следующие параметры:

- **ContentUri**. Общедоступный URI HTTP(S) пакета содержимого гостевой конфигурации.
- **DisplayName**. Отображаемое имя политики.
- **Описание**: описание политики.
- **Parameter**. Параметры политики, указанные в формате хэш-таблицы.
- **Версия.** Версия политики.
- **Путь**. Целевой путь, по которому создаются определения политик.
- **Platform**. Целевая платформа (Windows или Linux) для политики гостевой конфигурации и пакета содержимого.
- **Tag** —добавляет один или несколько фильтров тегов к определению политики.
- **Category** — задает поле метаданных категории в определении политики.

В следующем примере создаются определения политики из пакета настраиваемой политики в папке, к которой указан путь.

```azurepowershell-interactive
New-GuestConfigurationPolicy `
    -ContentUri 'https://storageaccountname.blob.core.windows.net/packages/AuditBitLocker.zip?st=2019-07-01T00%3A00%3A00Z&se=2024-07-01T00%3A00%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=JdUf4nOCo8fvuflOoX%2FnGo4sXqVfP5BYXHzTl3%2BovJo%3D' `
    -DisplayName 'Audit BitLocker Service.' `
    -Description 'Audit if BitLocker is not enabled on Windows machine.' `
    -Path './policies' `
    -Platform 'Windows' `
    -Version 1.0.0 `
    -Verbose
```

Командлет `New-GuestConfigurationPolicy` создает следующие файлы:

- **auditIfNotExists.json**;

В выходных данных командлета возвращается объект, содержащий отображаемое имя инициативы и путь к файлам политики.

Опубликуйте определения политики с помощью командлета `Publish-GuestConfigurationPolicy`. Этот командлет принимает только параметр **Path** с расположением JSON-файлов, созданных `New-GuestConfigurationPolicy`.

Чтобы выполнить команду Publish, требуются права доступа для создания политик в Azure. Требования к авторизации описаны на странице [Общие сведения о Политике Azure](../overview.md). Наиболее подходящей встроенной ролью является **Участник политики ресурсов**.

```azurepowershell-interactive
Publish-GuestConfigurationPolicy -Path '.\policyDefinitions'
```

Командлет `Publish-GuestConfigurationPolicy` принимает путь из конвейера PowerShell. Эта функция позволяет создать файлы политики и опубликовать их в одном наборе последовательных команд.

```azurepowershell-interactive
New-GuestConfigurationPolicy `
 -ContentUri 'https://storageaccountname.blob.core.windows.net/packages/AuditBitLocker.zip?st=2019-07-01T00%3A00%3A00Z&se=2024-07-01T00%3A00%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=JdUf4nOCo8fvuflOoX%2FnGo4sXqVfP5BYXHzTl3%2BovJo%3D' `
  -DisplayName 'Audit BitLocker service.' `
  -Description 'Audit if the BitLocker service is not enabled on Windows machine.' `
  -Path './policies' `
 | Publish-GuestConfigurationPolicy
```

С помощью политики, созданной в Azure, последним шагом является назначение определения. См. раздел назначение определения с помощью [портала](../assign-policy-portal.md), [Azure CLI](../assign-policy-azurecli.md)и [Azure PowerShell](../assign-policy-powershell.md).

### <a name="filtering-guest-configuration-policies-using-tags"></a>Фильтрация политик гостевой конфигурации с помощью тегов

Определения политик, созданные командлетами в модуле гостевой конфигурации, при желании можно дополнить фильтром для тегов. Параметр **Tag** для командлета `New-GuestConfigurationPolicy` поддерживает массив хэш-таблиц, содержащих отдельные теги. Теги добавляются в `If` раздел определения политики и не могут быть изменены с помощью назначения политики.

Ниже приведен пример фрагмента определения политики, которая применяет фильтр по тегам.

```json
"if": {
  "allOf" : [
    {
      "allOf": [
        {
          "field": "tags.Owner",
          "equals": "BusinessUnit"
        },
        {
          "field": "tags.Role",
          "equals": "Web"
        }
      ]
    },
    {
      // Original Guest Configuration content
    }
  ]
}
```

### <a name="using-parameters-in-custom-guest-configuration-policy-definitions"></a>Использование параметров в определениях пользовательских политик гостевой конфигурации

Гостевая конфигурация поддерживает переопределение свойств конфигурации во время выполнения. Эта возможность позволяет использовать в MOF-файле пакета не только статические значения. Переопределяемые значения предоставляются с помощью Политики Azure и не влияют на то, как создаются или компилируются конфигурации.

Командлеты `New-GuestConfigurationPolicy` и `Test-GuestConfigurationPolicyPackage` включают параметр с именем **Parameter**. Этот параметр принимает определение хэш-таблицы с подробными сведениями о каждом параметре и создает все необходимые разделы файлов, используемых для определения в службе "Политика Azure".

В следующем примере создается определение политики для аудита службы, для которого пользователь выбирает значение из списка во время назначения политики.

```azurepowershell-interactive
# This DSC Resource text:
Service 'UserSelectedNameExample'
      {
          Name = 'ParameterValue'
          Ensure = 'Present'
          State = 'Running'
      }

# Would require the following hashtable:
$PolicyParameterInfo = @(
    @{
        Name = 'ServiceName'                                            # Policy parameter name (mandatory)
        DisplayName = 'windows service name.'                           # Policy parameter display name (mandatory)
        Description = "Name of the windows service to be audited."      # Policy parameter description (optional)
        ResourceType = "Service"                                        # DSC configuration resource type (mandatory)
        ResourceId = 'UserSelectedNameExample'                                   # DSC configuration resource id (mandatory)
        ResourcePropertyName = "Name"                                   # DSC configuration resource property name (mandatory)
        DefaultValue = 'winrm'                                          # Policy parameter default value (optional)
        AllowedValues = @('BDESVC','TermService','wuauserv','winrm')    # Policy parameter allowed values (optional)
    }
)

New-GuestConfigurationPolicy
    -ContentUri 'https://storageaccountname.blob.core.windows.net/packages/AuditBitLocker.zip?st=2019-07-01T00%3A00%3A00Z&se=2024-07-01T00%3A00%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=JdUf4nOCo8fvuflOoX%2FnGo4sXqVfP5BYXHzTl3%2BovJo%3D' `
    -DisplayName 'Audit Windows Service.' `
    -Description 'Audit if a Windows Service is not enabled on Windows machine.' `
    -Path '.\policyDefinitions' `
    -Parameter $PolicyParameterInfo `
    -Version 1.0.0
```

## <a name="extending-guest-configuration-with-third-party-tools"></a>Расширение гостевой конфигурации с помощью средств сторонних разработчиков

Пакеты артефактов для гостевой конфигурации можно расширить, включив средства сторонних разработчиков.
Для расширения гостевой конфигурации требуется создать два компонента.

- Ресурс Desired State Configuration, который обрабатывает все действия, связанные с управлением сторонним средством.
  - Установка
  - Invoke
  - Преобразование вывода
- Содержимое в правильном формате, который понимает это средство.

Ресурс DSC требует пользовательской разработки, если решение сообщества еще не существует.
Решения сообщества можно просмотреть, выполнив поиск в коллекции PowerShell по тегу [GuestConfiguration](https://www.powershellgallery.com/packages?q=Tags%3A%22GuestConfiguration%22).

> [!Note]
> Расширяемость гостевой конфигурации поддерживается в режиме "использование собственной лицензии". Перед ее использованием убедитесь, что вы соблюдаете все условия, применимые к средствам сторонних разработчиков.

После установки ресурса DSC в среде разработки примените параметр **FilesToInclude** для командлета `New-GuestConfigurationPackage`, чтобы включить содержимое платформы стороннего разработчика в артефакт содержимого.

## <a name="policy-lifecycle"></a>Жизненный цикл политики

Если вы хотите отпустить обновление политики, внесите изменения в пакет конфигурации гостя и сведения об определении политики Azure.

> [!NOTE]
> `version`Свойство назначения гостевой конфигурации влияет только на пакеты, размещенные корпорацией Майкрософт. Для управления версиями пользовательским содержимым рекомендуется включить версию в имя файла.

Во-первых, при запуске `New-GuestConfigurationPackage` Укажите имя пакета, которое делает его уникальным по отношению к предыдущим версиям. Номер версии можно включить в имя, например `PackageName_1.0.0` .
Число в этом примере используется только для того, чтобы сделать пакет уникальным, а не указывать, что пакет должен считаться более новым или более старым, чем другие пакеты.

Во вторых, обновите параметры, используемые с `New-GuestConfigurationPolicy` командлетом, после каждого из приведенных ниже объяснений.

- **Версия.** При запуске командлета `New-GuestConfigurationPolicy` необходимо указать номер версии, который больше текущей опубликованной версии.
- **контентури**: при выполнении `New-GuestConfigurationPolicy` командлета необходимо указать универсальный код ресурса (URI) для расположения пакета. Включение версии пакета в имя файла обеспечит изменение значения этого свойства в каждом выпуске.
- **contentHash**. Это свойство обновляется автоматически командлетом `New-GuestConfigurationPolicy`. Это значение хэша пакета, созданного командлетом `New-GuestConfigurationPackage`. Это свойство должно соответствовать публикуемому файлу `.zip`. Если обновляется только свойство **contentUri**, расширение не примет пакет содержимого.

Самый простой способ для выпуска обновленного пакета — повторить процедуру, описанную в этой статье, и указать новый номер версии. Этот позволит обеспечить правильное обновление всех свойств.

## <a name="optional-signing-guest-configuration-packages"></a>Необязательное действие: Подписывание пакетов гостевой конфигурации

Настраиваемые политики гостевой конфигурации используют хэш SHA256 для проверки того, что пакет политики не изменился.
При необходимости клиенты также могут использовать сертификат для подписывания пакетов и настроить расширение гостевой конфигурации на использование только подписанного содержимого.

Чтобы реализовать этот сценарий, необходимо выполнить два действия. Запустите командлет, чтобы подписать пакет содержимого, и добавьте тег на компьютеры, требующие подписания кода.

Чтобы использовать функцию проверки подписи, запустите командлет `Protect-GuestConfigurationPackage` для подписания пакета перед его публикацией. Для этого командлета требуется сертификат подписывания кода.

```azurepowershell-interactive
$Cert = Get-ChildItem -Path cert:\LocalMachine\My | Where-Object {($_.Subject-eq "CN=mycert") }
Protect-GuestConfigurationPackage -Path .\package\AuditWindowsService\AuditWindowsService.zip -Certificate $Cert -Verbose
```

Командлет `Protect-GuestConfigurationPackage` принимает следующие параметры:

- **Путь**. Полный путь к пакету гостевой конфигурации.
- **Certificate**. Сертификат подписывания кода для подписывания пакета приложения. Этот параметр поддерживается только при подписывании содержимого для Windows.

Агент гостевой конфигурации ожидает наличия открытого ключа сертификата в списке центров сертификации для доверенных корневых сертификатов на компьютерах Windows и в пути `/usr/local/share/ca-certificates/extra` на компьютерах Linux. Чтобы узел проверял подписанное содержимое, установите открытый ключ сертификата на компьютер перед применением пользовательской политики. Это можно выполнить любым обычным способом на виртуальной машине или с помощью службы "Политика Azure". Пример шаблона [приведен здесь](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-push-certificate-windows).
Политика доступа Key Vault должна разрешать поставщику вычислительных ресурсов доступ к сертификатам во время развертывания. Подробные инструкции см. в разделе [Настройка хранилища ключей для виртуальных машин в Azure Resource Manager](../../../virtual-machines/windows/key-vault-setup.md#use-templates-to-set-up-key-vault).

Ниже приводится пример экспорта открытого ключа из сертификата для подписи для последующего импорта на компьютер.

```azurepowershell-interactive
$Cert = Get-ChildItem -Path cert:\LocalMachine\My | Where-Object {($_.Subject-eq "CN=mycert3") } | Select-Object -First 1
$Cert | Export-Certificate -FilePath "$env:temp\DscPublicKey.cer" -Force
```

После публикации содержимого добавьте тег с именем `GuestConfigPolicyCertificateValidation` и значением `enabled` на все виртуальные машины, где требуется подписывание кода. См. [примеры тегов](../samples/built-in-policies.md#tags), чтобы узнать, как организовать доставку тегов в большом масштабе с помощью службы "Политика Azure". Когда тег будет создан, определение политики, созданное с помощью командлета `New-GuestConfigurationPolicy`, обеспечивает соблюдение требования за счет расширения гостевой конфигурации.

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте подробнее об аудите виртуальных машин с помощью [гостевой конфигурации](../concepts/guest-configuration.md).
- Узнайте о [программном создании политик](./programmatically-create.md).
- Узнайте, как [получать данные о соответствии](./get-compliance-data.md).
