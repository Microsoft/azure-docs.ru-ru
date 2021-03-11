---
title: Обработчик расширения настройки требуемого состояния Azure
description: С помощью расширения DSC вы можете отправлять и применять конфигурацию DSC PowerShell к виртуальной машине Azure.
ms.topic: article
ms.service: virtual-machines
ms.subservice: extensions
author: bobbytreed
ms.author: robreed
ms.collection: windows
ms.date: 03/26/2018
ms.openlocfilehash: 72f66aeee64133a13ce0e49155c4b2a90240a3fb
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102559993"
---
# <a name="powershell-dsc-extension"></a>Расширение PowerShell DSC

## <a name="overview"></a>Обзор

Расширение DSC PowerShell для Windows публикуется и поддерживается корпорацией Майкрософт. Расширение отправляет и применяет конфигурацию DSC PowerShell к виртуальной машине Azure. Расширение DSC вызывает DSC PowerShell, чтобы применить полученную конфигурацию DSC к виртуальной машине. В этом документе подробно описаны поддерживаемые платформы, конфигурации и параметры развертывания для расширения виртуальной машины DSC для Windows.

## <a name="prerequisites"></a>Предварительные требования

### <a name="operating-system"></a>Операционная система

Расширение DSC поддерживает следующие операционные системы:

Windows Server 2019, Windows Server 2016, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 с пакетом обновления 1 (SP1), Windows Client 7/8.1/10

### <a name="internet-connectivity"></a>Подключение к Интернету

Для расширения DSC для Windows требуется, чтобы Целевая виртуальная машина могла взаимодействовать с Azure и расположением пакета конфигурации (ZIP-файл), если он хранится в расположении за пределами Azure. 

## <a name="extension-schema"></a>Схема расширения

В приведенном ниже JSON-файле показана схема для части settings расширения DSC в шаблоне Azure Resource Manager. 

```json
{
  "type": "Microsoft.Compute/virtualMachines/extensions",
  "name": "Microsoft.Powershell.DSC",
  "apiVersion": "2018-10-01",
  "location": "<location>",
  "properties": {
    "publisher": "Microsoft.Powershell",
    "type": "DSC",
    "typeHandlerVersion": "2.77",
    "autoUpgradeMinorVersion": true,
    "settings": {
        "wmfVersion": "latest",
        "configuration": {
            "url": "http://validURLToConfigLocation",
            "script": "ConfigurationScript.ps1",
            "function": "ConfigurationFunction"
        },
        "configurationArguments": {
            "argument1": "Value1",
            "argument2": "Value2"
        },
        "configurationData": {
            "url": "https://foo.psd1"
        },
        "privacy": {
            "dataCollection": "enable"
        },
        "advancedOptions": {
            "forcePullAndApply": false,
            "downloadMappings": {
             "specificDependencyKey": "https://myCustomDependencyLocation"
            }
        } 
    },
    "protectedSettings": {
        "configurationArguments": {
            "parameterOfTypePSCredential1": {
                "userName": "UsernameValue1",
                "password": "PasswordValue1"
            },
            "parameterOfTypePSCredential2": {
                "userName": "UsernameValue2",
                "password": "PasswordValue2"
            }
        },
        "configurationUrlSasToken": "?g!bber1sht0k3n",
        "configurationDataUrlSasToken": "?dataAcC355T0k3N"
    }
  }
}
```

### <a name="property-values"></a>Значения свойств

| Имя | Значение и пример | Тип данных |
| ---- | ---- | ---- |
| версия_API | 2018-10-01 | Дата |
| publisher | Microsoft.Powershell.DSC | строка |
| type | DSC | строка |
| typeHandlerVersion | 2.77 | INT |

### <a name="settings-property-values"></a>Значения свойства параметров

| Имя | Тип данных | Описание
| ---- | ---- | ---- |
| settings.wmfVersion | строка | Указывает версию Windows Management Framework, которую необходимо установить на виртуальной машине. Если задать для этого свойства значение latest, будет установлена последняя версия Windows Management Framework. Для этого свойства доступны только такие значения: 4.0, 5.0 и latest. Возможные значения зависят от обновлений. По умолчанию используется значение latest. |
| settings.configuration.url | строка | Указывает URL-адрес расположения, из которого можно скачать ZIP-файл конфигурации DSC. Если для доступа к предоставленному URL-адресу требуется маркер SAS, для свойства protectedSettings.configurationUrlSasToken будет необходимо задать значение маркера SAS. Это свойство обязательное, если заданы свойства settings.configuration.script и (или) settings.configuration.function.
| settings.configuration.script | строка | Указывает имя файла скрипта, содержащего определение вашей конфигурации DSC. Этот скрипт должен находиться в корневом каталоге ZIP-файла, скачанного по URL-адресу, указанному в свойстве configuration.url. Это свойство обязательное, если заданы свойства settings.configuration.url и (или) settings.configuration.script.
| settings.configuration.function | строка | Указывает имя вашей конфигурации DSC. Указанную конфигурацию необходимо добавить в скрипт, заданный в свойстве configuration.script. Это свойство обязательное, если заданы свойства settings.configuration.url и (или) settings.configuration.function.
| settings.configurationArguments | Коллекция | Определяет параметры, которые необходимо передать в конфигурацию DSC. Это свойство не будет зашифровано.
| settings.configurationData.url | строка | Указывает URL-адрес расположения, из которого можно скачать файл данных конфигурации (в формате PDS1), используемый в качестве входных данных для вашей конфигурации DSC. Если для доступа к предоставленному URL-адресу требуется маркер SAS, для свойства protectedSettings.configurationDataUrlSasToken будет необходимо задать значение маркера SAS.
| settings.privacy.dataEnabled | строка | Включает или отключает сбор данных телеметрии. Для этого свойства доступны только такие значения: Enable, Disable, '' или $null. Если для этого свойства не задано значение или задано значение NULL, сбор данных телеметрии будет выполняться.
| settings.advancedOptions.forcePullAndApply | Bool | Этот параметр предназначен для улучшения работы с расширением для регистрации узлов в Automation DSC Azure.  Если значение равно `$true` , расширение будет ожидать первого запуска конфигурации, извлеченного из службы, прежде чем будет возвращено сообщение об успешном выполнении или сбое.  Если задано значение $false, то состояние, возвращаемое расширением, будет указывать только на то, был ли узел зарегистрирован в конфигурации состояния службы автоматизации Azure, и конфигурация узла не будет выполняться во время регистрации.
| settings.advancedOptions.downloadMappings | Коллекция | Определяет альтернативные расположения для скачивания зависимостей, таких как WMF и .NET.

### <a name="protected-settings-property-values"></a>Значения свойств защищенных параметров

| Имя | Тип данных | Описание
| ---- | ---- | ---- |
| protectedSettings.configurationArguments | строка | Определяет параметры, которые необходимо передать в конфигурацию DSC. Это свойство будет зашифровано. |
| protectedSettings.configurationUrlSasToken | строка | Указывает маркер SAS для доступа к URL-адресу, определенному в свойстве configuration.url. Это свойство будет зашифровано. |
| protectedSettings.configurationDataUrlSasToken | строка | Указывает маркер SAS для доступа к URL-адресу, определенному в свойстве configurationData.url. Это свойство будет зашифровано. |


## <a name="template-deployment"></a>Развертывание шаблона

Расширения виртуальной машины Azure можно развернуть с помощью шаблонов Azure Resource Manager.
Шаблоны идеально подходят для развертывания одной или нескольких виртуальных машин, требующих настройки после развертывания.
Пример шаблона диспетчер ресурсов, включающий расширение DSC для Windows, можно найти в [коллекции быстрое начало Azure](https://github.com/Azure/azure-quickstart-templates/blob/master/101-automation-configuration/nested/provisionServer.json#L91).

## <a name="troubleshoot-and-support"></a>Устранение неполадок и поддержка

### <a name="troubleshoot"></a>Диагностика

Данные о состоянии развертывания расширения можно получить на портале Azure, а также использовав Azure CLI. Чтобы просмотреть состояние развертывания расширений для определенной виртуальной машины, выполните следующую команду в Azure CLI.

```azurecli
az vm extension list --resource-group myResourceGroup --vm-name myVM -o table
```

Пакет расширения скачивается и развертывается в этом расположении на виртуальной машине Azure
```
C:\Packages\Plugins\{Extension_Name}\{Extension_Version}
```

В файле состояния расширения содержатся коды подсостояния и ошибок состояния, а также подробные сведения об ошибке и описание для каждого запуска расширения.
```
C:\Packages\Plugins\{Extension_Name}\{Extension_Version}\Status\{0}.Status  -> {0} being the sequence number
```

Журналы выходных данных расширения регистрируются в следующем каталоге:

```
C:\WindowsAzure\Logs\Plugins\{Extension_Name}\{Extension_Version}
```

### <a name="error-codes-and-their-meanings"></a>Коды ошибок и их описание

| Код ошибки | Значение | Возможное действие |
| :---: | --- | --- |
| 1000 | Общая ошибка | Сообщение об этой ошибке выдается при возникновении конкретного исключения и приводится в журналах расширения. |
| 52 | Ошибка установки расширения | Сообщение об этой ошибке выдается при возникновении конкретного исключения. |
| 1002 | Ошибка установки WMF | Произошла ошибка при установке WMF. |
| 1004 | Недопустимый ZIP-пакет | Недопустимый ZIP-файл. Ошибка при распаковке ZIP-файла. |
| 1100 | Ошибка аргумента | Указывает на проблему во входных данных, введенных пользователем. Сообщение об этой ошибке выдается при возникновении конкретного исключения|


### <a name="support"></a>Поддержка

Если в любой момент при изучении этой статьи вам потребуется дополнительная помощь, вы можете обратиться к экспертам по Azure на [форумах MSDN Azure и Stack Overflow](https://azure.microsoft.com/support/forums/). Кроме того, можно зарегистрировать обращение в службу поддержки Azure. Перейдите на [сайт поддержки Azure](https://azure.microsoft.com/support/options/) и щелкните "Получить поддержку". Дополнительные сведения об использовании службы поддержки Azure см. в статье [Часто задаваемые вопросы о поддержке Microsoft Azure](https://azure.microsoft.com/support/faq/).
