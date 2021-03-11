---
title: Расширение Desired State Configuration (DSC) с использованием шаблонов Azure Resource Manager
description: В этой статье описывается шаблон Resource Manager для расширения Desired State Configuration в Azure.
services: virtual-machines
author: mgoedtel
tags: azure-resource-manager
keywords: DSC
ms.assetid: b5402e5a-1768-4075-8c19-b7f7402687af
ms.service: virtual-machines
ms.subservice: extensions
ms.collection: windows
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: na
ms.date: 02/09/2021
ms.author: magoedte
ms.openlocfilehash: 6d22e93fe8bd36e24520108bc310d5b08c25120c
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102560027"
---
# <a name="desired-state-configuration-extension-with-azure-resource-manager-templates"></a>Расширение Desired State Configuration (DSC) с использованием шаблонов Azure Resource Manager

В этой статье описывается шаблон Azure Resource Manager для [обработчика расширения Desired State Configuration (DSC)](dsc-overview.md). Во многих примерах используется **регистратионурл** (представленная в виде строки) и **RegistrationKey** (предоставляется как [PSCredential](/dotnet/api/system.management.automation.pscredential) для подключения к службе автоматизации Azure. Подробнее о получении этих значений см. в разделе [Подключение компьютеров для управления с помощью настройки состояния службы автоматизации Azure (Безопасная регистрация)](../../automation/automation-dsc-onboarding.md#enable-machines-securely-using-registration).

> [!NOTE]
> Вы можете встретить немного отличающиеся примеры схемы. Изменение в схеме произошло в выпуске за октябрь 2016 года. Дополнительные сведения см. в разделе об [обновлении предыдущего формата](#update-from-a-previous-format).

## <a name="template-example-for-a-windows-vm"></a>Пример шаблона для виртуальной машины Windows

В приведенном ниже фрагменте кода показан раздел **Resource** шаблона.
Расширение DSC наследует свойства расширения по умолчанию.
Дополнительные сведения см. в статье [о классе VirtualMachineExtension](/dotnet/api/microsoft.azure.management.compute.models.virtualmachineextension).

```json
{
  "type": "Microsoft.Compute/virtualMachines/extensions",
  "name": "[concat(parameters('VMName'), '/Microsoft.Powershell.DSC')]",
  "apiVersion": "2018-06-01",
  "location": "[parameters('location')]",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', parameters('VMName'))]"
  ],
  "properties": {
    "publisher": "Microsoft.Powershell",
    "type": "DSC",
    "typeHandlerVersion": "2.77",
    "autoUpgradeMinorVersion": true,
    "protectedSettings": {
      "Items": {
        "registrationKeyPrivate": "[listKeys(resourceId('Microsoft.Automation/automationAccounts/', parameters('automationAccountName')), '2018-06-30').Keys[0].value]"
      }
    },
    "settings": {
      "Properties": [
        {
          "Name": "RegistrationKey",
          "Value": {
            "UserName": "PLACEHOLDER_DONOTUSE",
            "Password": "PrivateSettingsRef:registrationKeyPrivate"
          },
          "TypeName": "System.Management.Automation.PSCredential"
        },
        {
          "Name": "RegistrationUrl",
          "Value": "[reference(concat('Microsoft.Automation/automationAccounts/', parameters('automationAccountName'))).registrationUrl]",
          "TypeName": "System.String"
        },
        {
          "Name": "NodeConfigurationName",
          "Value": "[parameters('nodeConfigurationName')]",
          "TypeName": "System.String"
        }
      ]
    }
  }
}
```

## <a name="template-example-for-windows-virtual-machine-scale-sets"></a>Пример шаблона для масштабируемого набора виртуальных машин Windows

Узел масштабируемого набора виртуальных машин содержит раздел **properties** с атрибутами **VirtualMachineProfile и extensionProfile**.
В разделе **extensions** добавьте данные о расширении DSC.

Расширение DSC наследует свойства расширения по умолчанию.
Дополнительные сведения см. в статье [о классе VirtualMachineScaleSetExtension](/dotnet/api/microsoft.azure.management.compute.models.virtualmachinescalesetextension).

```json
"extensionProfile": {
    "extensions": [
      {
        "name": "Microsoft.Powershell.DSC",
        "properties": {
          "publisher": "Microsoft.Powershell",
          "type": "DSC",
          "typeHandlerVersion": "2.77",
          "autoUpgradeMinorVersion": true,
          "protectedSettings": {
            "Items": {
              "registrationKeyPrivate": "[listKeys(resourceId('Microsoft.Automation/automationAccounts/', parameters('automationAccountName')), '2018-06-30').Keys[0].value]"
            }
          },
          "settings": {
            "Properties": [
              {
                "Name": "RegistrationKey",
                "Value": {
                  "UserName": "PLACEHOLDER_DONOTUSE",
                  "Password": "PrivateSettingsRef:registrationKeyPrivate"
                },
                "TypeName": "System.Management.Automation.PSCredential"
              },
              {
                "Name": "RegistrationUrl",
                "Value": "[reference(concat('Microsoft.Automation/automationAccounts/', parameters('automationAccountName'))).registrationUrl]",
                "TypeName": "System.String"
              },
              {
                "Name": "NodeConfigurationName",
                "Value": "[parameters('nodeConfigurationName')]",
                "TypeName": "System.String"
              }
            ]
          }
        }
      }
    ]
  }
```

## <a name="detailed-settings-information"></a>Подробные сведения о разделе settings

Используйте указанную ниже схему в разделе **settings** расширения Azure DSC в шаблоне Resource Manager.

Список доступных аргументов для скрипта конфигурации по умолчанию см. в [этом разделе](#default-configuration-script).

```json
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
        "downloadMappings": {
            "customWmfLocation": "http://myWMFlocation"
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
```

## <a name="details"></a>Сведения

| Имя свойства | Тип | Описание |
| --- | --- | --- |
| settings.wmfVersion |строка |Указывает версию Windows Management Framework (WMF), которую необходимо установить на виртуальной машине. Если задать для этого свойства значение **latest**, будет установлена последняя версия Windows Management Framework. В настоящее время для этого свойства доступны только такие значения: **4.0**, **5.0**, **5.1** и **latest**. Возможные значения зависят от обновлений. По умолчанию используется значение **latest**. |
| settings.configuration.url |строка |Указывает URL-адрес расположения, из которого можно скачать ZIP-файл конфигурации DSC. Если для доступа к предоставленному URL-адресу требуется маркер SAS, задайте для свойства **protectedSettings.configurationUrlSasToken** значение маркера SAS. Это свойство обязательное, если заданы свойства **settings.configuration.script** или **settings.configuration.function**. Если для этих свойств значение не задано, необходимо предоставить расширение, которое вызывает скрипт конфигурации по умолчанию для установки метаданных диспетчера конфигурации расположения (LCM) и аргументов. |
| settings.configuration.script |строка |Указывает имя файла скрипта, содержащего определение вашей конфигурации DSC. Этот скрипт должен находиться в корневом каталоге ZIP-файла, скачанного по URL-адресу, указанному в свойстве **settings.configuration.url**. Это свойство обязательное, если заданы свойства **settings.configuration.url** или **settings.configuration.script**. Если для этих свойств значение не задано, необходимо предоставить расширение, которое вызывает скрипт конфигурации по умолчанию для установки метаданных LCM и аргументов. |
| settings.configuration.function |строка |Указывает имя вашей конфигурации DSC. Указанную конфигурацию необходимо добавить в скрипт, который определяет свойство **settings.configuration.script**. Это свойство обязательное, если заданы свойства **settings.configuration.url** или **settings.configuration.function**. Если для этих свойств значение не задано, необходимо предоставить расширение, которое вызывает скрипт конфигурации по умолчанию для установки метаданных LCM и аргументов. |
| settings.configurationArguments |Коллекция |Определяет параметры, которые необходимо передать в конфигурацию DSC. Это свойство не зашифровано. |
| settings.configurationData.url |строка |Указывает URL-адрес расположения, из которого можно скачать файл данных конфигурации (в формате PDS1), используемый в качестве входных данных для вашей конфигурации DSC. Если для доступа к предоставленному URL-адресу требуется маркер SAS, задайте для свойства **protectedSettings.configurationDataUrlSasToken** значение маркера SAS. |
| settings.privacy.dataCollection |строка |Включает или отключает сбор данных телеметрии. Для этого свойства доступны только такие значения: **Enable**, **Disable**, **''** или **$null**. Если для этого свойства не задано значение или задано значение null, сбор данных телеметрии будет выполняться. Значение по умолчанию — **''** . Дополнительные сведения см. в записи блога [Azure DSC Extension Data Collection](https://devblogs.microsoft.com/powershell/azure-dsc-extension-data-collection-2/) (Коллекция данных расширения DSC Azure). |
| settings.advancedOptions.downloadMappings |Коллекция |Определяет альтернативные расположения для скачивания Windows Management Framework. Дополнительные сведения см. в записи блога [Azure DSC Extension 2.8 & How to map downloads of the extension dependencies to your own location](https://devblogs.microsoft.com/powershell/azure-dsc-extension-2-8-how-to-map-downloads-of-the-extension-dependencies-to-your-own-location/) (Расширение DSC Azure версии 2.8. Сведения о сопоставлении загрузок зависимостей расширения с вашим расположением). |
| protectedSettings.configurationArguments |Коллекция |Определяет параметры, которые необходимо передать в конфигурацию DSC. Это свойство зашифровано. |
| protectedSettings.configurationUrlSasToken |строка |Указывает маркер SAS для доступа к URL-адресу, который определяет свойство **settings.configuration.url**. Это свойство зашифровано. |
| protectedSettings.configurationDataUrlSasToken |строка |Указывает маркер SAS для доступа к URL-адресу, который определяет свойство **settings.configurationData.url**. Это свойство зашифровано. |

## <a name="default-configuration-script"></a>Скрипт конфигурации по умолчанию

Дополнительные сведения об указанных ниже значениях см. в разделе [Базовые параметры](/powershell/scripting/dsc/managing-nodes/metaConfig#basic-settings).
С помощью скрипта конфигурации по умолчанию расширения DSC можно настроить только свойства LCM, указанные в следующей таблице.

| Имя свойства | Тип | Описание |
| --- | --- | --- |
| protectedSettings.configurationArguments.RegistrationKey |PSCredential |Обязательное свойство. Определяет ключ, используемый узлом для регистрации в службе автоматизации Azure в качестве пароля для объекта учетных данных PowerShell. Это значение можно автоматически обнаружить с помощью метода **listkeys** в учетной записи службы автоматизации.  См. [пример](#example-using-referenced-azure-automation-registration-values). |
| settings.configurationArguments.RegistrationUrl |строка |Обязательное свойство. Указывает URL-адрес конечной точки службы автоматизации, в которой пытается зарегистрироваться узел. Это значение можно автоматически обнаружить с помощью метода **reference** в учетной записи службы автоматизации. |
| settings.configurationArguments.NodeConfigurationName |строка |Обязательное свойство. Указывает конфигурацию узла в учетной записи службы автоматизации, которая будет присвоена узлу. |
| settings.configurationArguments.ConfigurationMode |строка |Указывает режим для LCM. Допустимые значения: **ApplyOnly**, **ApplyandMonitior** и **ApplyandAutoCorrect**.  Значение по умолчанию — **ApplyAndMonitor**. |
| settings.configurationArguments.RefreshFrequencyMins | uint32 | Указывает, как часто LCM пытается проверить учетную запись службы автоматизации на наличие обновлений.  Значение по умолчанию — **30**.  Минимальное значение — **15**. |
| settings.configurationArguments.ConfigurationModeFrequencyMins | uint32 | Указывает, как часто LCM проверяет текущую конфигурацию. Значение по умолчанию — **15**. Минимальное значение — **15**. |
| settings.configurationArguments.RebootNodeIfNeeded | Логическое | Указывает, может ли узел автоматически перезагружаться по запросу операции DSC. Значение по умолчанию — **false**. |
| settings.configurationArguments.ActionAfterReboot | строка | Указывает, что происходит после перезагрузки при применении конфигурации. Допустимые значения: **ContinueConfiguration** и **StopConfiguration**. Значение по умолчанию: **ContinueConfiguration**. |
| settings.configurationArguments.AllowModuleOverwrite | Логическое | Указывает, перезаписывает ли LCM имеющиеся модули на узле. Значение по умолчанию — **false**. |

## <a name="settings-vs-protectedsettings"></a>Сравнение разделов settings и protectedSettings

Все параметры сохраняются в текстовом файле параметров на виртуальной машине.
Свойства, указанные в разделе **settings**, общедоступные.
Общедоступные свойства не зашифрованы в текстовом файле параметров.
Свойства, указанные в разделе **protectedSettings**, зашифрованы с помощью сертификата, а это значит, что они не отображаются как обычный текст в файле параметров на виртуальной машине.

Если для конфигурации нужны учетные данные, их можно добавить в раздел **protectedSettings**.

```json
"protectedSettings": {
    "configurationArguments": {
        "parameterOfTypePSCredential1": {
               "userName": "UsernameValue1",
               "password": "PasswordValue1"
        }
    }
}
```

## <a name="example-configuration-script"></a>Пример скрипта конфигурации

В указанном ниже примере показано поведение по умолчанию расширения DSC, которое заключается в предоставлении параметров метаданных LCM и регистрации в службе автоматизации DSC.
Аргументы конфигурации необходимы
и будут переданы в скрипт конфигурации по умолчанию, чтобы задать метаданные LCM.

```json
"settings": {
    "configurationArguments": {
        "RegistrationUrl" : "[parameters('registrationUrl1')]",
        "NodeConfigurationName" : "nodeConfigurationNameValue1"
    }
},
"protectedSettings": {
    "configurationArguments": {
        "RegistrationKey": {
            "userName": "NOT_USED",
            "Password": "registrationKey"
        }
    }
}
```

## <a name="example-using-the-configuration-script-in-azure-storage"></a>Пример с использованием скрипта конфигурации в службе хранилища Azure

Указанный ниже пример основан на примере конфигурации в статье [Общие сведения об обработчике расширения Desired State Configuration в Azure](dsc-overview.md).
В этом примере для развертывания расширения используются шаблоны Resource Manager, а не командлеты.
Сохраните конфигурацию IisInstall.ps1, добавьте ее в ZIP-файл (пример: `iisinstall.zip`) и передайте файл на доступный URL-адрес.
В этом примере используется хранилище BLOB-объектов Azure, но ZIP-файл можно скачать из любого произвольного расположения.

В шаблоне Resource Manager следующий код указывает виртуальной машине скачать правильный файл и выполнить соответствующую функцию PowerShell.

```json
"settings": {
    "configuration": {
        "url": "https://demo.blob.core.windows.net/iisinstall.zip",
        "script": "IisInstall.ps1",
        "function": "IISInstall"
    }
},
"protectedSettings": {
    "configurationUrlSasToken": "odLPL/U1p9lvcnp..."
}
```

## <a name="example-using-referenced-azure-automation-registration-values"></a>Пример использования ссылок на значения регистрации в службе автоматизации Azure

Следующий пример получает **RegistrationUrl** и **RegistrationKey**, ссылаясь на свойства учетной записи службы автоматизации Azure, используя метод **listkeys** для извлечения первичного ключа (0).  В этом примере параметры **automationAccountName** и **NodeConfigName** были предоставлены в шаблоне.

```json
"settings": {
    "RegistrationUrl" : "[reference(concat('Microsoft.Automation/automationAccounts/', parameters('automationAccountName'))).registrationUrl]",
    "NodeConfigurationName" : "[parameters('NodeConfigName')]"
},
"protectedSettings": {
    "configurationArguments": {
        "RegistrationKey": {
            "userName": "NOT_USED",
            "Password": "[listKeys(resourceId('Microsoft.Automation/automationAccounts/', parameters('automationAccountName')), '2018-01-15').Keys[0].value]"
        }
    }
}
```

## <a name="update-from-a-previous-format"></a>Обновление из предыдущего формата

Все параметры в предыдущем формате расширения (содержащие общедоступные свойства **ModulesUrl**, **ModuleSource**, **ModuleVersion**, **ConfigurationFunction**, **SasToken** или **Properties**) автоматически адаптируются к текущему формату расширения.
и выполняются в обычном режиме.

Раньше схема settings выглядела следующим образом:

```json
"settings": {
    "WMFVersion": "latest",
    "ModulesUrl": "https://UrlToZipContainingConfigurationScript.ps1.zip",
    "SasToken": "SAS Token if ModulesUrl points to private Azure Blob Storage",
    "ConfigurationFunction": "ConfigurationScript.ps1\\ConfigurationFunction",
    "Properties": {
        "ParameterToConfigurationFunction1": "Value1",
        "ParameterToConfigurationFunction2": "Value2",
        "ParameterOfTypePSCredential1": {
            "UserName": "UsernameValue1",
            "Password": "PrivateSettingsRef:Key1"
        },
        "ParameterOfTypePSCredential2": {
            "UserName": "UsernameValue2",
            "Password": "PrivateSettingsRef:Key2"
        }
    }
},
"protectedSettings": {
    "Items": {
        "Key1": "PasswordValue1",
        "Key2": "PasswordValue2"
    },
    "DataBlobUri": "https://UrlToConfigurationDataWithOptionalSasToken.psd1"
}
```

Вот как меняется предыдущий формат:

| Имя текущего свойства | Предыдущий эквивалент схемы |
| --- | --- |
| settings.wmfVersion |settings.wmfVersion |
| settings.configuration.url |settings.ModulesUrl |
| settings.configuration.script |Первая часть свойства settings.ConfigurationFunction (перед \\\\) |
| settings.configuration.function |Вторая часть свойства settings.ConfigurationFunction (после \\\\) |
| settings.configuration.module.name | settings.ModuleSource |
| settings.configuration.module.version | settings.ModuleVersion |
| settings.configurationArguments |settings.Properties |
| settings.configurationData.url |protectedSettings.DataBlobUri (без маркера SAS) |
| settings.privacy.dataCollection |settings.Privacy.dataCollection |
| settings.advancedOptions.downloadMappings |settings.advancedOptions.downloadMappings |
| protectedSettings.configurationArguments |protectedSettings.Properties |
| protectedSettings.configurationUrlSasToken |settings.SasToken |
| protectedSettings.configurationDataUrlSasToken |Маркер SAS из свойства protectedSettings.DataBlobUri |

## <a name="troubleshooting"></a>Устранение неполадок

Ниже приведены некоторые общие ошибки и способы их устранения.

### <a name="invalid-values"></a>Недопустимые значения

"Privacy.dataCollection имеет значение "{0}".
Единственные возможные значения это "Enable" и "Disable".
"WmfVersion имеет значение "{0}".
Возможные значения: … и "latest".

**Проблема.** Указано недопустимое значение.

**Решение** Замените недопустимое значение допустимым.
Дополнительные сведения см. в таблице раздела [Сведения](#details).

### <a name="invalid-url"></a>Недопустимый URL-адрес

"ConfigurationData.url имеет значение "{0}". Это недопустимый URL-адрес." "DataBlobUri имеет значение "{0}". Это недопустимый URL-адрес." "Configuration.url имеет значение "{0}". This is not a valid URL" (Значение свойства Configuration.url — "{0}". Это недопустимый URL-адрес).

**Проблема.** Указан недопустимый URL-адрес.

**Решение** Проверьте все указанные URL-адреса.
Убедитесь, что все URL-адреса разрешаются в допустимые расположения, к которым расширение может получить доступ на удаленном компьютере.

### <a name="invalid-registrationkey-type"></a>Недопустимый тип RegistrationKey

"Недопустимый тип параметра RegistrationKey типа PSCredential."

**Проблема.** Значение *RegistrationKey* в protectedSettings.configurationArguments не может быть предоставлено как любой тип, отличный от PSCredential.

**Решение** Измените запись protectedSettings.configurationArguments для RegisterKey на тип PSCredential, используя следующий формат:

```json
"configurationArguments": {
    "RegistrationKey": {
        "userName": "NOT_USED",
        "Password": "RegistrationKey"
    }
}
```

### <a name="invalid-configurationargument-type"></a>Недопустимый тип свойства ConfigurationArgument

"Недопустимый тип ConfigurationArgument: {0}"

**Проблема.** Невозможно разрешить свойство *ConfigurationArguments* в объект **Хэш-таблица**.

**Решение** Задайте для свойства *ConfigurationArguments* тип **Хэш-таблица**.
Следуйте формату из приведенного выше примеров. Обращайте внимание на кавычки, запятые и скобки.

### <a name="duplicate-configurationarguments"></a>Повторяющееся свойство ConfigurationArguments

"В общедоступном и защищенном свойстве configurationArguments найден повторяющийся аргумент "{0}"

**Проблема.** *ConfigurationArguments* из общедоступных параметров и *ConfigurationArguments* из защищенных параметров имеют свойства с одинаковыми именами.

**Решение** Удалите одно из повторяющихся свойств.

### <a name="missing-properties"></a>Отсутствующие свойства

"settings.сonfiguration.function requires that configuration.url or settings.configuration.module is specified" (Для параметра settings.configuration.function требуется указать свойство settings.configuration.url или configuration.module)

"settings.сonfiguration.url requires that settings.сonfiguration.script is specified" (Для параметра settings.сonfiguration.url требуется указать свойство settings.сonfiguration.script)

"settings.сonfiguration.script requires that settings.сonfiguration.url is specified" (Для параметра settings.сonfiguration.script требуется указать свойство settings.сonfiguration.url)

"settings.сonfiguration.url requires that settings.сonfiguration.function is specified" (Для параметра settings.сonfiguration.function требуется указать свойство settings.сonfiguration.url)

"protectedSettings.ConfigurationUrlSasToken.url requires that settings.сonfiguration.function is specified" (Для параметра protectedSettings.ConfigurationUrlSasToken.url требуется указать свойство settings.сonfiguration.url)

"protectedSettings.ConfigurationDataUrlSasToken.url requires that settings.сonfiguration.function is specified" (Для параметра protectedSettings.ConfigurationDataUrlSasToken.url требуется указать свойство settings.сonfiguration.url)

**Проблема.** Для заданного свойства требуется другое свойство, которое отсутствует.

**Решения**:

- Укажите отсутствующее свойство.
- Удалите свойство, требующее отсутствующего свойства.

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте больше об [использовании наборов масштабирования виртуальных машин с помощью расширения Azure DSC](../../virtual-machine-scale-sets/virtual-machine-scale-sets-dsc.md).
- Получите дополнительные сведения о [безопасном управлении учетными данными посредством DSC](dsc-credentials.md).
- Ознакомьтесь с [общими сведениями об обработчике расширения Desired State Configuration в Azure](dsc-overview.md).
- Дополнительные сведения о PowerShell DSC см. в [центре документации по PowerShell](/powershell/scripting/dsc/overview/overview).
