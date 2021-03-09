---
title: Создание шаблона Конструктора образов виртуальных машин Azure (предварительная версия)
description: Узнайте, как создать шаблон для использования с Конструктором образов виртуальных машин Azure.
author: danielsollondon
ms.author: danis
ms.date: 03/02/2021
ms.topic: reference
ms.service: virtual-machines
ms.subservice: image-builder
ms.collection: linux
ms.reviewer: cynthn
ms.openlocfilehash: a3138da0ecbcabaeb7ef910975afc3b7005e5b50
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102519713"
---
# <a name="preview-create-an-azure-image-builder-template"></a>Предварительный просмотр: Создание шаблона Конструктора образов виртуальных машин Azure 

Конструктор образов виртуальных машин Azure использует JSON-файл для передачи сведений в службу "Конструктор образов". В этой статье мы рассмотрим разделы JSON-файла, чтобы вы могли создать свой собственный. Примеры полных JSON-файлов см. в разделе [GitHub, посвященном Конструктору образов виртуальных машин Azure](https://github.com/Azure/azvmimagebuilder/tree/main/quickquickstarts).

Ниже приведен базовый формат шаблона.

```json
 { 
    "type": "Microsoft.VirtualMachineImages/imageTemplates", 
    "apiVersion": "2020-02-14", 
    "location": "<region>", 
    "tags": {
        "<name": "<value>",
        "<name>": "<value>"
     },
    "identity":{},           
    "dependsOn": [], 
    "properties": { 
        "buildTimeoutInMinutes": <minutes>, 
        "vmProfile": 
            {
            "vmSize": "<vmSize>",
            "osDiskSizeGB": <sizeInGB>,
            "vnetConfig": {
                "subnetId": "/subscriptions/<subscriptionID>/resourceGroups/<vnetRgName>/providers/Microsoft.Network/virtualNetworks/<vnetName>/subnets/<subnetName>"
                }
            },
        "source": {}, 
        "customize": {}, 
        "distribute": {} 
      } 
 } 
```



## <a name="type-and-api-version"></a>Тип и версия API

`type` — это тип ресурса, который должен быть `"Microsoft.VirtualMachineImages/imageTemplates"`. `apiVersion` со временем будет меняться по мере изменений API, но для предварительной версии должно быть указано значение `"2020-02-14"`.

```json
    "type": "Microsoft.VirtualMachineImages/imageTemplates",
    "apiVersion": "2020-02-14",
```

## <a name="location"></a>Расположение

Расположение — это регион, в котором будет создан пользовательский образ. В предварительной версии Конструктора образов поддерживаются следующие регионы.

- Восточная часть США
- восточная часть США 2
- центрально-западная часть США
- западная часть США
- западная часть США 2
- Северная Европа
- Западная Европа


```json
    "location": "<region>",
```
## <a name="vmprofile"></a>vmProfile
По умолчанию Конструктор образов будет использовать виртуальную машину сборки "Standard_D1_v2". Эту виртуальную машину можно переопределить; например, если вы хотите настроить образ для виртуальной машины GPU, потребуется размер виртуальной машины GPU. Водить описание не обязательно.

```json
 {
    "vmSize": "Standard_D1_v2"
 },
```

## <a name="osdisksizegb"></a>osDiskSizeGB

По умолчанию Конструктор образов не будет изменять размер образа, а будет использовать размер из исходного образа. Можно увеличить **только** размер диска операционной системы (Win и Linux), это необязательный параметр, а значение 0 означает, что размер исходного образа не изменится. Размер диска ОС нельзя уменьшить до размера, меньшего, чем размер исходного образа.

```json
 {
    "osDiskSizeGB": 100
 },
```

## <a name="vnetconfig"></a>vnetConfig
Если вы не укажете свойства виртуальной сети, Конструктор образов создаст собственную виртуальную сеть, общедоступный IP-адрес и NSG. Общедоступный IP-адрес используется для взаимодействия службы с виртуальной машиной сборки, однако если вы не хотите общедоступный IP-адрес или хотите, чтобы Конструктор образов имел доступ к существующим ресурсам вашей виртуальной сети, таким как серверы конфигурации (DSC, Chef, Puppet, Ansible), общие папки и т. д., то можете указать виртуальную сеть. Дополнительные сведения можно найти в [документации по сетям](image-builder-networking.md).

```json
    "vnetConfig": {
        "subnetId": "/subscriptions/<subscriptionID>/resourceGroups/<vnetRgName>/providers/Microsoft.Network/virtualNetworks/<vnetName>/subnets/<subnetName>"
        }
    }
```
## <a name="tags"></a>Теги

Это пары "ключ-значение", которые вы можете указать для создаваемого образа.

## <a name="depends-on-optional"></a>dependsOn (необязательно)

Этот необязательный раздел можно использовать, чтобы обеспечить завершение зависимостей перед продолжением. 

```json
    "dependsOn": [],
```

Дополнительные сведения см. в разделе [Определение зависимостей ресурсов](../../azure-resource-manager/templates/define-resource-dependency.md#dependson).

## <a name="identity"></a>Идентификация

Обязательный. для построителя образов необходимо иметь разрешения на чтение и запись образов, чтение в скриптах из службы хранилища Azure. необходимо создать удостоверение User-Assigned Azure с разрешениями для отдельных ресурсов. Дополнительные сведения о работе с разрешениями для работы с Image Builder и соответствующих действиях см. в [документации](image-builder-user-assigned-identity.md).


```json
    "identity": {
    "type": "UserAssigned",
          "userAssignedIdentities": {
            "<imgBuilderId>": {}
        }
        },
```


Поддержка построителя изображений для User-Assigned удостоверения:
* Поддерживает только одно удостоверение
* Не поддерживает пользовательские доменные имена

Дополнительные сведения см. в разделе [Что такое управляемые удостоверения для ресурсов Azure](../../active-directory/managed-identities-azure-resources/overview.md).
Дополнительные сведения о развертывании этого компонента см. в разделе [Настройка управляемых удостоверений для ресурсов Azure на виртуальной машине Azure с помощью Azure CLI](../../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md#user-assigned-managed-identity).

## <a name="properties-source"></a>Свойства: source

В разделе `source` содержатся сведения об исходном образе, который будет использоваться Конструктором образов. В настоящее время построитель образов поддерживает создание образов Hyper-V (GEN1) 1 в галерее образов (SIG) или управляемом образе Azure. Если вы хотите создать образы Gen2, необходимо использовать исходный образ Gen2 и распространить его на виртуальный жесткий диск. После этого потребуется создать управляемый образ из виртуального жесткого диска и вставить его в подпись в качестве образа Gen2.

Для API должен быть задан тип источника, определяющий источник для сборки образа. В настоящее время существует три типа:
- PlatformImage — указывает, что источником является образ Marketplace;
- ManagedImage — используется при запуске из обычного управляемого образа;
- SharedImageVersion —указывается, когда в качестве источника используется версия образа из Общей коллекции образов.


> [!NOTE]
> При использовании существующих пользовательских образов Windows можно выполнить команду Sysprep до 8 раз в одном образе Windows. Дополнительные сведения см. в документации по [Sysprep](/windows-hardware/manufacture/desktop/sysprep--generalize--a-windows-installation#limits-on-how-many-times-you-can-run-sysprep) .

### <a name="platformimage-source"></a>Источник PlatformImage 
Конструктор образов Azure поддерживает образы Windows Server, клиента Windows и Linux Azure Marketplace. Полный список см. [здесь](../image-builder-overview.md#os-support). 

```json
        "source": {
            "type": "PlatformImage",
                "publisher": "Canonical",
                "offer": "UbuntuServer",
                "sku": "18.04-LTS",
                "version": "latest"
        },
```


Здесь используются те же свойства, что и при создании виртуальных машин. Чтобы получить список свойств, выполните в интерфейсе командной строки Azure следующую команду. 
 
```azurecli-interactive
az vm image list -l westus -f UbuntuServer -p Canonical --output table –-all 
```

В качестве значения версии можно указать "latest", так как версия определяется при сборке образа, а не при отправке шаблона. Если эта функциональная возможность используется с назначением "Общая коллекция образов", можно избежать повторной отправки шаблона и перезапускать сборку образа через определенные интервалы, чтобы ваши образы воссоздавались из самых последних образов.

#### <a name="support-for-market-place-plan-information"></a>Поддержка сведений о плане размещения на рынке
Можно также указать сведения о плане, например:
```json
    "source": {
        "type": "PlatformImage",
        "publisher": "RedHat",
        "offer": "rhel-byos",
        "sku": "rhel-lvm75",
        "version": "latest",
        "planInfo": {
            "planName": "rhel-lvm75",
            "planProduct": "rhel-byos",
            "planPublisher": "redhat"
       }
```
### <a name="managedimage-source"></a>Источник ManagedImage

Задает в качестве исходного образа существующий управляемый образ универсального виртуального жесткого диска или виртуальной машины.

> [!NOTE]
> Исходный управляемый образ должен быть поддерживаемой ОС, а образ должен быть в том же регионе, что и шаблон Azure Image Builder. 

```json
        "source": { 
            "type": "ManagedImage", 
                "imageId": "/subscriptions/<subscriptionId>/resourceGroups/{destinationResourceGroupName}/providers/Microsoft.Compute/images/<imageName>"
        }
```

Идентификатор `imageId` должен быть идентификатором ResourceId управляемого образа. Для получения списка доступных образов используйте команду `az image list`.


### <a name="sharedimageversion-source"></a>Источник SharedImageVersion
Задает в качестве исходного образа существующую версию образа в Общей коллекции образов.

> [!NOTE]
> Исходный управляемый образ должен быть поддерживаемой ОС, а образ должен быть в том же регионе, что и шаблон Azure Image Builder. в противном случае следует реплицировать версию образа в область шаблона построителя образов.


```json
        "source": { 
            "type": "SharedImageVersion", 
            "imageVersionID": "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/p  roviders/Microsoft.Compute/galleries/<sharedImageGalleryName>/images/<imageDefinitionName/versions/<imageVersion>" 
   } 
```

Идентификатор `imageVersionId` должен быть идентификатором ResourceId версии образа. Для вывода списка версий образа используйте команду [az sig image-version list](/cli/azure/sig/image-version#az-sig-image-version-list).


## <a name="properties-buildtimeoutinminutes"></a>Свойства: buildTimeoutInMinutes

По умолчанию Конструктор образов будет выполняться в течение 240 минут. После этого наступает блокировка по времени, и Конструктор образов остановится, независимо от того, завершена ли сборка образа. При возникновении блокировки по времени появится сообщение об ошибке, подобное приведенному ниже.

```text
[ERROR] Failed while waiting for packerizer: Timeout waiting for microservice to
[ERROR] complete: 'context deadline exceeded'
```

Если вы не указали значение свойства buildTimeoutInMinutes или задали значение 0, будет использоваться значение по умолчанию. Это значение можно увеличивать или уменьшать; максимальное значение составляет 960 минут (16 часов). Для Windows не рекомендуется устанавливать значение меньше 60 минут. Если вы обнаружили, что возникла пауза, просмотрите [журналы](image-builder-troubleshoot.md#customization-log) — возможно, на этом этапе настройки ожидается какое-нибудь действие, например ввод пользователя. 

Если оказывается, что для завершения настройки требуется больше времени, задайте нужное время с небольшим запасом. Но не задавайте слишком высокое значение, так как, возможно, вам придется дожидаться, пока это время истечет, прежде чем появится сообщение об ошибке. 


## <a name="properties-customize"></a>Свойства: customize

Конструктор образов поддерживает несколько "настройщиков". Настройщики — это функции, используемые для настройки образа. Это может быть, например, выполнение сценариев или перезагрузка серверов. 

Применяя `customize`, помните следующие правила. 
- Можно использовать несколько настройщиков, но каждый из них должен иметь уникальное `name`.
- Настройщики выполняются в порядке, указанном в шаблоне.
- В случае сбоя одного из настройщиков происходит сбой всего компонента настройки и выводится сообщение об ошибке.
- Настоятельно рекомендуется тщательно протестировать скрипт, прежде чем использовать его в шаблоне. Гораздо проще отладить скрипт на собственной виртуальной машине.
- Не помещайте в скрипты конфиденциальные данные. 
- Расположения скриптов должны быть общедоступными, если не используется [MSI](https://docs.microsoft.com/azure/virtual-machines/linux/image-builder-user-assigned-identity).

```json
        "customize": [
            {
                "type": "Shell",
                "name": "<name>",
                "scriptUri": "<path to script>",
                "sha256Checksum": "<sha256 checksum>"
            },
            {
                "type": "Shell",
                "name": "<name>",
                "inline": [
                    "<command to run inline>",
                ]
            }

        ],
```     

 
Раздел настройки представляет собой массив. Конструктор образов виртуальных машин Azure будет выполнять настройщики в последовательном порядке. Любой сбой в каком-либо из настройщиков приведет к сбою всего процесса сборки. 

> [!NOTE]
> Встроенные команды можно просмотреть в определении шаблона образа и служба поддержки Майкрософт при помощи в обращении в службу поддержки. Если у вас есть конфиденциальная информация, ее следует переместить в скрипты в службе хранилища Azure, где доступ требует проверки подлинности.
 
### <a name="shell-customizer"></a>Настройщик Shell

Настройщик оболочки (Shell) поддерживает выполнение скриптов оболочки, которые должны быть общедоступными, чтобы Конструктор образов мог к ним обращаться.

```json
    "customize": [ 
        { 
            "type": "Shell", 
            "name": "<name>", 
            "scriptUri": "<link to script>",
            "sha256Checksum": "<sha256 checksum>"       
        }, 
    ], 
        "customize": [ 
        { 
            "type": "Shell", 
            "name": "<name>", 
            "inline": "<commands to run>"
        }, 
    ], 
```

Поддержка ОС: Linux 
 
Свойства раздела customize:

- **type** — Shell. 
- **name** — имя для отслеживания настройки. 
- **scriptUri** — универсальный код ресурса (URI) для расположения файла 
- **inline** — массив команд оболочки, разделенных запятыми.
- **sha256Checksum** — значение контрольной суммы sha256 файла; вы формируете ее локально, а Конструктор образов затем проверяет ее.
    * Чтобы сформировать sha256Checksum, откройте окно терминала в Mac/Linux и выполните следующую команду: `sha256sum <fileName>`

> [!NOTE]
> Встроенные команды хранятся как часть определения шаблона изображения, они могут отображаться при создании дампа определения образа, и они также видимы для служба поддержки Майкрософт в случае обращения в службу поддержки для устранения неполадок. Если у вас есть важные команды или значения, настоятельно рекомендуется переместить их в скрипты и использовать удостоверение пользователя для проверки подлинности в службе хранилища Azure.

#### <a name="super-user-privileges"></a>Привилегии суперпользователя
Чтобы команды выполнялись с привилегиями суперпользователя, их необходимо `sudo` Добавить в скрипты или использовать встроенные команды, например:
```json
                "type": "Shell",
                "name": "setupBuildPath",
                "inline": [
                    "sudo mkdir /buildArtifacts",
                    "sudo cp /tmp/index.html /buildArtifacts/index.html"
```
Пример скрипта с использованием sudo, на который можно сослаться с помощью Скриптури:
```bash
#!/bin/bash -e

echo "Telemetry: creating files"
mkdir /myfiles

echo "Telemetry: running sudo 'as-is' in a script"
sudo touch /myfiles/somethingElevated.txt
```

### <a name="windows-restart-customizer"></a>Настройщик перезапуска Windows 
С помощью настройщика перезапуска можно перезапустить виртуальную машину Windows и дождаться ее возвращения в сеть. Это позволяет устанавливать программное обеспечение, требующее перезагрузки.  

```json 
     "customize": [ 

            {
                "type": "WindowsRestart",
                "restartCommand": "shutdown /r /f /t 0", 
                "restartCheckCommand": "echo Azure-Image-Builder-Restarted-the-VM  > c:\\buildArtifacts\\azureImageBuilderRestart.txt",
                "restartTimeout": "5m"
            }
  
        ],
```

Поддержка ОС: Windows
 
Свойства раздела customize:
- **Тип**. WindowsRestart
- **restartCommand** — команда для выполнения перезапуска (необязательно). Значение по умолчанию — `'shutdown /r /f /t 0 /c \"packer restart\"'`.
- **restartCheckCommand** — команда для проверки успешности перезапуска (необязательно). 
- **restartTimeout** — время ожидания перезапуска, указанное в виде строки, состоящей из величины и единицы измерения. Например, `5m` (5 минут) или `2h` (2 часа). Значение по умолчанию: '5m'

### <a name="linux-restart"></a>Перезапуск Linux  
Настройщик перезапуска Linux не предусмотрен. Однако если вы устанавливаете драйверы или компоненты, требующие перезагрузки, то вы можете установить их и вызвать перезагрузку с помощью настройщика оболочки. Для виртуальной машины сборки существует время ожидания SSH 20 минут.

### <a name="powershell-customizer"></a>Настройщик PowerShell 
Настройщик оболочки (Shell) выполняет скрипты и встроенные команды. Скрипты должны быть общедоступными, чтобы Конструктор образов мог к ним обращаться.

```json 
     "customize": [
        { 
             "type": "PowerShell",
             "name":   "<name>",  
             "scriptUri": "<path to script>",
             "runElevated": "<true false>",
             "sha256Checksum": "<sha256 checksum>" 
        },  
        { 
             "type": "PowerShell", 
             "name": "<name>", 
             "inline": "<PowerShell syntax to run>", 
             "validExitCodes": "<exit code>",
             "runElevated": "<true or false>" 
         } 
    ], 
```

Поддержка ОС: Windows и Linux

Свойства раздела customize:

- **type** — PowerShell.
- **scriptUri** — универсальный код ресурса (URI) для расположения файла скрипта PowerShell. 
- **inline** — встроенные команды, разделенные запятыми.
- **validExitCodes** — необязательные допустимые коды, которые можно возвращать из скрипта или встроенной команды. Это позволит избежать возврата ошибки скриптом или встроенной командой.
- **runElevated** — необязательное логическое значение для поддержки выполнения команд и скриптов с повышенными привилегиями.
- **sha256Checksum** — значение контрольной суммы sha256 файла; вы формируете ее локально, а Конструктор образов затем проверяет ее.
    * Для формирования sha256Checksum используйте командлет PowerShell в Windows [Get-Hash](/powershell/module/microsoft.powershell.utility/get-filehash)


### <a name="file-customizer"></a>Настройщик File

Настройщик File позволяет Конструктору образов скачивать файлы из GitHub или службы хранилища Azure. Если у вас есть конвейер сборки образа, который зависит от артефактов сборки, можно установить настройщик File для загрузки из общей папки сборки и переместить артефакты в образ.  

```json
     "customize": [ 
         { 
            "type": "File", 
             "name": "<name>", 
             "sourceUri": "<source location>",
             "destination": "<destination>",
             "sha256Checksum": "<sha256 checksum>"
         }
     ]
```

Поддержка ОС: Linux и Windows 

Свойства настройщика File:

- **sourceUri** — доступная конечная точка хранилища. Это может быть GitHub или служба хранилища Azure. Вы можете загружать только один файл, а не весь каталог. Если необходимо загрузить каталог, используйте сжатый файл, а затем распакуйте его с помощью настройщиков Shell или PowerShell. 

> [!NOTE]
> Если sourceUri является учетной записью хранения Azure, независимо от того, помечен ли этот большой двоичный объект общедоступным, вы будете предоставлять управляемому удостоверению пользователя разрешения на чтение BLOB-объекта. Ознакомьтесь с этим [примером](https://docs.microsoft.com/azure/virtual-machines/linux/image-builder-user-assigned-identity#create-a-resource-group) , чтобы задать разрешения для хранилища.

- **destination** — полный путь к целевому файлу и имя файла. Все указанные пути и подкаталоги должны существовать. Для их предварительной настройки используйте настройщики Shell или PowerShell. Вы можете использовать настройщики скриптов для создания пути. 

Каталоги Windows и пути Linux это поддерживают, но с некоторыми отличиями. 
- В ОС Linux Конструктор образов может осуществлять запись только по пути /tmp.
- В Windows нет ограничений на используемый путь, но этот путь должен существовать.
 
 
Если при попытке загрузить файл или разместить его в указанном каталоге произошла ошибка, то этап настройки завершится ошибкой, которая будет зарегистрирована в журнале customization.log.

> [!NOTE]
> Настройщик File подходит только для загрузки небольших файлов, размер которых менее 20 МБ. Для загрузки файлов большего размера используйте скрипт или встроенную команду с кодом для загрузки файлов, например `wget` или `curl` в Linux, `Invoke-WebRequest` в Windows.

### <a name="windows-update-customizer"></a>Настройщик WindowsUpdate
Этот настройщик построен на [средстве подготовки Центра обновления Windows](https://packer.io/docs/provisioners/community-supported.html) для Packer. Это проект с открытым исходным кодом, поддерживаемый сообществом Packer. Корпорация Майкрософт тестирует и проверяет это средство подготовки с помощью службы "Конструктор образов", а также будет поддерживать исследования проблем этого средства и работу по их устранению, однако этот проект с открытым исходным кодом официально не поддерживается Майкрософт. Подробную документацию и справку по этому средству подготовки Центра обновления Windows см. в репозитории проекта.

```json
     "customize": [
            {
                "type": "WindowsUpdate",
                "searchCriteria": "IsInstalled=0",
                "filters": [
                    "exclude:$_.Title -like '*Preview*'",
                    "include:$true"
                            ],
                "updateLimit": 20
            }
               ], 
OS support: Windows
```

Свойства раздела customize:
- **type** — WindowsUpdate.
- **searchcriteria** — необязательное свойство, определяющее тип устанавливаемых обновлений (Recommended, Important и т. д.). По умолчанию установлено BrowseOnly=0 и IsInstalled=0 (Recommended).
- **filters**— необязательное свойство, позволяющее указать фильтр для включения или исключения обновлений.
- **updateLimit** — необязательное свойство; задает, сколько обновлений можно установить (по умолчанию 1000).
 
> [!NOTE]
> Настройка Центр обновления Windows может завершиться ошибкой при наличии невыполненных перезапусков Windows или при выполнении установки приложения. обычно эта ошибка может появиться в журнале настройки. log `System.Runtime.InteropServices.COMException (0x80240016): Exception from HRESULT: 0x80240016` . Мы настоятельно рекомендуем рассмотреть возможность добавления в перезагрузку Windows и/или предоставления приложениям достаточного времени для завершения установки с помощью команд [Sleep](https://docs.microsoft.com/powershell/module/microsoft.powershell.utility/start-sleep) или wait во встроенных командах или сценариях перед запуском центр обновления Windows.

### <a name="generalize"></a>Generalize 
По умолчанию Конструктор образов виртуальных машин Azure также выполняет код отзыва в конце каждого этапа настройки образа, чтобы подготовить образ к использованию. Подготовка к использованию — это процесс, в котором образ настраивается для многократного использования в целях создания нескольких виртуальных машин. Для виртуальных машин Windows Конструктор образов виртуальных машин Azure использует команду Sysprep. Для Linux Конструктор образов выполняет команду "waagent -deprovision". 

Команды, которые Конструктор образов использует для подготовки к использованию, могут не подойти для каких-то ситуаций, поэтому Конструктор образов позволяет вам при необходимости настроить эту команду. 

Если выполняется миграция существующей настройки, и вы используете разные команды Sysprep и waagent, вы можете применять универсальные команды Конструктора образов, а в случае сбоя создания виртуальной машины — свои собственные команды Sysprep или waagent.

Если Конструктор образов виртуальных машин Azure успешно создает пользовательский образ Windows, а вы создаете на его основе виртуальную машину и затем обнаруживаете, что произошла ошибка или создание завершилось неудачно, вам следует просмотреть документацию по Windows Server Sysprep или отправить запрос в группу поддержки клиентов Windows Server Sysprep, где вам помогут устранить неполадки и дадут рекомендации по правильному использованию Sysprep.


#### <a name="default-sysprep-command"></a>Команда Sysprep по умолчанию
```powershell
Write-Output '>>> Waiting for GA Service (RdAgent) to start ...'
while ((Get-Service RdAgent).Status -ne 'Running') { Start-Sleep -s 5 }
Write-Output '>>> Waiting for GA Service (WindowsAzureTelemetryService) to start ...'
while ((Get-Service WindowsAzureTelemetryService) -and ((Get-Service WindowsAzureTelemetryService).Status -ne 'Running')) { Start-Sleep -s 5 }
Write-Output '>>> Waiting for GA Service (WindowsAzureGuestAgent) to start ...'
while ((Get-Service WindowsAzureGuestAgent).Status -ne 'Running') { Start-Sleep -s 5 }
Write-Output '>>> Sysprepping VM ...'
if( Test-Path $Env:SystemRoot\system32\Sysprep\unattend.xml ) {
  Remove-Item $Env:SystemRoot\system32\Sysprep\unattend.xml -Force
}
& $Env:SystemRoot\System32\Sysprep\Sysprep.exe /oobe /generalize /quiet /quit
while($true) {
  $imageState = (Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Setup\State).ImageState
  Write-Output $imageState
  if ($imageState -eq 'IMAGE_STATE_GENERALIZE_RESEAL_TO_OOBE') { break }
  Start-Sleep -s 5
}
Write-Output '>>> Sysprep complete ...'
```
#### <a name="default-linux-deprovision-command"></a>Команда Linux deprovision по умолчанию

```bash
/usr/sbin/waagent -force -deprovision+user && export HISTSIZE=0 && sync
```

#### <a name="overriding-the-commands"></a>Переопределение команд
Чтобы переопределить команды, с помощью средств подготовки скриптов PowerShell или Shell создайте файлы команд со строго соответствующими именами файлов и разместите их в правильных каталогах:

* Windows: c:\DeprovisioningScript.ps1
* Linux: /tmp/DeprovisioningScript.sh

Конструктор образов будет считывать эти команды, и они записываются в журналы "customization.log" Конструктора образов Azure. Сведения о сборе журналов см. в разделе, посвященном [устранению неполадок](image-builder-troubleshoot.md#customization-log).
 
## <a name="properties-distribute"></a>Свойства: distribute

Конструктор образов Azure поддерживает три целевых объекта распространения: 

- **managedImage** — управляемый образ;
- **sharedImage** — Общая коллекция образов;
- **VHD** — VHD в учетной записи хранения.

Вы можете распространить изображение на оба типа целевого объекта в той же конфигурации.

> [!NOTE]
> Команда Sysprep по умолчанию AIB не включает "/Mode: VM", но это может быть необходимо при создании образов, на которых будет установлена роль HyperV. Если необходимо добавить этот аргумент команды, необходимо переопределить команду Sysprep.

Так как у вас может быть несколько целевых объектов для распространения образа, Конструктор образов поддерживает состояние для каждого целевого объекта распространения, которое можно получить, запросив `runOutputName`.  `runOutputName` — объект, который можно запросить после распространения, чтобы получить сведения об этом распространении. Например, можно запросить расположение VHD, регионы, в которые была реплицирована версия образа, или версию созданного образа SIG. Это свойство имеет каждый целевой объект распространения. Имя `runOutputName` должно быть уникальным для каждого целевого объекта распространения. Ниже приведен пример, в котором запрашивается распространение Общей коллекции образов.

```bash
subscriptionID=<subcriptionID>
imageResourceGroup=<resourceGroup of image template>
runOutputName=<runOutputName>

az resource show \
        --ids "/subscriptions/$subscriptionID/resourcegroups/$imageResourceGroup/providers/Microsoft.VirtualMachineImages/imageTemplates/ImageTemplateLinuxRHEL77/runOutputs/$runOutputName"  \
        --api-version=2020-02-14
```

Выходные данные:
```json
{
  "id": "/subscriptions/xxxxxx/resourcegroups/rheltest/providers/Microsoft.VirtualMachineImages/imageTemplates/ImageTemplateLinuxRHEL77/runOutputs/rhel77",
  "identity": null,
  "kind": null,
  "location": null,
  "managedBy": null,
  "name": "rhel77",
  "plan": null,
  "properties": {
    "artifactId": "/subscriptions/xxxxxx/resourceGroups/aibDevOpsImg/providers/Microsoft.Compute/galleries/devOpsSIG/images/rhel/versions/0.24105.52755",
    "provisioningState": "Succeeded"
  },
  "resourceGroup": "rheltest",
  "sku": null,
  "tags": null,
  "type": "Microsoft.VirtualMachineImages/imageTemplates/runOutputs"
}
```

### <a name="distribute-managedimage"></a>Распространение: managedImage

Выходным результатом образа будет ресурс управляемого образа.

```json
{
       "type":"managedImage",
       "imageId": "<resource ID>",
       "location": "<region>",
       "runOutputName": "<name>",
       "artifactTags": {
            "<name": "<value>",
            "<name>": "<value>"
        }
}
```
 
Свойства распространения:
- **type** — managedImage. 
- **имажеид** — идентификатор ресурса конечного образа, ожидаемый формат:/Subscriptions/ \<subscriptionId> /resourceGroups/ \<destinationResourceGroupName> /провидерс/Микрософт.компуте/имажес/\<imageName>
- **location** — расположение управляемого образа.  
- **runOutputName** — уникальное имя для идентификации распространения.  
- **artifactTags** — необязательные задаваемые пользователем теги пар "ключ-значение".
 
 
> [!NOTE]
> Целевая группа ресурсов должна существовать.
> Если вы хотите распространить образ в другой регион, это увеличит время развертывания. 

### <a name="distribute-sharedimage"></a>Распространение: sharedImage 
Общая коллекция образов Azure — это новая служба управления образами, которая позволяет управление репликацией региона образа, управление версиями и предоставление общего доступа к пользовательским образам. Конструктор образов Azure поддерживает распространение с помощью этой службы, так что вы можете распространять образы в регионы, поддерживаемые общими коллекциями образов. 
 
В Общую коллекцию образов входят следующие компоненты. 
 
- Коллекция — контейнер для нескольких общих образов. Коллекция развертывается в одном регионе.
- Определения образов — концептуальное группирование образов. 
- Версии образов — это тип образа, используемый для развертывания виртуальной машины или масштабируемого набора. Версии образов можно реплицировать в другие регионы, где требуется развернуть виртуальные машины.
 
Прежде чем можно будет распространять коллекцию образов, необходимо создать коллекцию и определения образов (см. раздел [общие образы](../shared-images-cli.md)). 

```json
{
    "type": "SharedImage",
    "galleryImageId": "<resource ID>",
    "runOutputName": "<name>",
    "artifactTags": {
        "<name>": "<value>",
        "<name>": "<value>"
    },
    "replicationRegions": [
        "<region where the gallery is deployed>",
        "<region>"
    ]
}
``` 

Свойства распространения для общих коллекций образов:

- **type** — sharedImage.  
- **галлеримажеид** — идентификатор общей коллекции образов, может быть указан в двух форматах:
    * Автоматическое управление версиями. в построителе образов создается монотонный номер версии. Это полезно при необходимости сохранения перестроения образов из одного шаблона: `/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Compute/galleries/<sharedImageGalleryName>/images/<imageGalleryName>` .
    * Явная версия. Вы можете передать номер версии, который должен использовать построитель изображений. Формат: `/subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.Compute/galleries/<sharedImageGalName>/images/<imageDefName>/versions/<version e.g. 1.1.1>`

- **runOutputName** — уникальное имя для идентификации распространения.  
- **artifactTags** — необязательные задаваемые пользователем теги пар "ключ-значение".
- **replicationRegions** — массив регионов для репликации. Один из регионов должен быть регионом, в котором развернута коллекция. Добавление регионов приведет к увеличению времени сборки, так как сборка не завершится до завершения репликации.
- **ексклудефромлатест** (необязательно). Это позволяет пометить созданную версию образа как последнюю версию в определении SIG, значение по умолчанию — false.
- **storageAccountType** (необязательно) AIB поддерживает указание следующих типов хранилища для создаваемой версии образа:
    * "Standard_LRS"
    * "Standard_ZRS"


> [!NOTE]
> Если шаблон изображения и ссылка на него `image definition` находятся в разных расположениях, вы увидите дополнительное время для создания изображений. В настоящее время построитель образов не имеет `location` параметра для ресурса версии изображения, он переведется из родительского объекта `image definition` . Например, если определение изображения находится в westus и требуется реплицировать версию образа в eastus, большой двоичный объект копируется в westus, из этого создается ресурс версии образа в westus, а затем репликация в eastus. Чтобы избежать дополнительного времени репликации, убедитесь, что `image definition` шаблон образа и находится в одном расположении.


### <a name="distribute-vhd"></a>Распространение: VHD  
Вы можете записать результат на виртуальный жесткий диск. Затем этот виртуальный жесткий диск можно копировать, использовать для публикации в Azure MarketPlace или использовать с Azure Stack.  

```json
{ 
    "type": "VHD",
    "runOutputName": "<VHD name>",
    "tags": {
        "<name": "<value>",
        "<name>": "<value>"
    }
}
```
 
Поддержка ОС: Windows и Linux

Параметры распространения на VHD:

- **type** — VHD.
- **runOutputName** — уникальное имя для идентификации распространения.  
- **tags** — необязательные задаваемые пользователем теги пар "ключ-значение".
 
Конструктор образов Azure не разрешает пользователю указывать расположение учетной записи хранения, но вы можете запросить состояние `runOutputs`, чтобы получить это расположение.  

```azurecli-interactive
az resource show \
   --ids "/subscriptions/$subscriptionId/resourcegroups/<imageResourceGroup>/providers/Microsoft.VirtualMachineImages/imageTemplates/<imageTemplateName>/runOutputs/<runOutputName>"  | grep artifactUri 
```

> [!NOTE]
> После создания VHD необходимо как можно скорее скопировать его в другое расположение. VHD хранится в учетной записи хранения во временной группе ресурсов, созданной при отправке шаблона образа в службу "Конструктор образов виртуальных машин Azure". Если вы удалите этот шаблон образа, VHD будет утерян. 

## <a name="image-template-operations"></a>Операции с шаблонами изображений

### <a name="starting-an-image-build"></a>Запуск сборки образа
Чтобы запустить сборку, необходимо вызвать командлет Run в ресурсе шаблона Image, примеры `run` команд:

```PowerShell
Invoke-AzResourceAction -ResourceName $imageTemplateName -ResourceGroupName $imageResourceGroup -ResourceType Microsoft.VirtualMachineImages/imageTemplates -ApiVersion "2020-02-14" -Action Run -Force
```


```bash
az resource invoke-action \
     --resource-group $imageResourceGroup \
     --resource-type  Microsoft.VirtualMachineImages/imageTemplates \
     -n helloImageTemplateLinux01 \
     --action Run 
```

### <a name="cancelling-an-image-build"></a>Отмена сборки образа
Если вы используете сборку образа, которую вы считаете неправильной, ожидаете ввода данных пользователем, или вы никогда не завершите работу, вы можете отменить сборку.

Сборку можно отменить в любое время. Если фаза распространения запустилась, вы все равно можете отменить, но вам потребуется очистить все образы, которые могут быть не завершены. Команда Cancel не ждет завершения команды Cancel. для этого выполните мониторинг `lastrunstatus.runstate` , чтобы отменить ход выполнения, используя эти [команды](image-builder-troubleshoot.md#customization-log)состояния.


Примеры `cancel` команд:

```powerShell
Invoke-AzResourceAction -ResourceName $imageTemplateName -ResourceGroupName $imageResourceGroup -ResourceType Microsoft.VirtualMachineImages/imageTemplates -ApiVersion "2020-02-14" -Action Cancel -Force
```

```bash
az resource invoke-action \
     --resource-group $imageResourceGroup \
     --resource-type  Microsoft.VirtualMachineImages/imageTemplates \
     -n helloImageTemplateLinux01 \
     --action Cancel 
```

## <a name="next-steps"></a>Дальнейшие действия

В библиотеке [GitHub в разделе Конструктора образов Azure](https://github.com/azure/azvmimagebuilder) имеются примеры JSON-файлов для разных сценариев.
