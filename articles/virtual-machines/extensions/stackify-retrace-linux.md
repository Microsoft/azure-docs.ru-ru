---
title: Stackify. перетрассировка расширения агента Linux для Azure
description: Разверните агент Stackify Retrace для Linux на виртуальной машине Linux.
ms.topic: article
ms.service: virtual-machines
ms.subservice: extensions
author: amjads1
ms.author: amjads
ms.collection: linux
ms.date: 04/12/2018
ms.openlocfilehash: 1fc437637fde524da125af9191bf9de79a2e9c37
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102559007"
---
# <a name="stackify-retrace-linux-agent-extension"></a>Расширение агента Stackify Retrace для Linux

## <a name="overview"></a>Обзор

Stackify предоставляет продукты, которые отслеживают сведения о приложении, чтобы ускорить поиск и устранение проблем. Для команд разработчиков Retrace представляет собой полностью интегрируемый сверхмощный инструмент отслеживания производительности приложений, совместимый с несколькими средами. Он объединяет в себе функции нескольких инструментов, необходимых любой группе разработки.

Retrace — это единственный инструмент, который обеспечивает приведенные ниже возможности во всех средах на одной платформе.

* Управление производительностью приложений (APM).
* Ведение журнала приложений и серверов.
* Отслеживание ошибок и мониторинг.
* Метрики сервера, приложения и настраиваемые метрики.

**О расширении агента Stackify Retrace для Linux**

Это расширение предоставляет путь установки агента Retrace для Linux. 

## <a name="prerequisites"></a>Предварительные требования

### <a name="operating-system"></a>Операционная система 

Агент Retrace можно выполнять с использованием следующих дистрибутивов Linux.

| Distribution | Версия |
|---|---|
| Ubuntu | 16.04 LTS, 14.04 LTS, 16.10 и 17.04 |
| Debian | 7.9 и выше, 8.2 и выше, 9 |
| Red Hat | 6.7 и выше, 7.1 и выше |
| CentOS | 6.3 и выше, 7.0 и выше |

### <a name="internet-connectivity"></a>Подключение к Интернету

Для расширения агента Stackify Retrace для Linux требуется, чтобы целевая виртуальная машина была подключена к Интернету. 

Может потребоваться настроить параметры сети, чтобы разрешить подключения к Stackify. Ознакомьтесь с разделом https://support.stackify.com/hc/en-us/articles/207891903-Adding-Exceptions-to-a-Firewall. 


## <a name="extension-schema"></a>Схема расширения

---

В следующем фрагменте JSON показана схема для расширения агента Stackify Retrace. Для работы расширения требуется `environment` и `activationKey`.

```json
    {
      "type": "extensions",
      "name": "StackifyExtension",
      "apiVersion": "[variables('apiVersion')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Compute/virtualMachines',variables('vmName'))]"
      ],
      "properties": {
        "publisher": "Stackify.LinuxAgent.Extension",
        "type": "StackifyLinuxAgentExtension",
        "typeHandlerVersion": "1.0",
        "autoUpgradeMinorVersion": true,
        "settings": {
          "environment": "myEnvironment"
        },
        "protectedSettings": {
          "activationKey": "myActivationKey"
        }
      }
    }      
```

## <a name="template-deployment"></a>Развертывание шаблона 

Расширения виртуальной машины Azure можно развернуть с помощью шаблонов Azure Resource Manager. Для запуска расширения агента Stackify Retrace для Linux во время развертывания шаблона Azure Resource Manager в нем можно использовать схему JSON, описанную в предыдущем разделе.  

JSON для расширения виртуальной машины можно вложить в ресурс виртуальной машины или поместить в корень или на верхний уровень JSON-файла шаблона Resource Manager. Размещение JSON влияет на значения имени и типа ресурса. Дополнительные сведения см. в разделе "Указание имени и типа дочернего ресурса в шаблоне Resource Manager".

В следующем примере предполагается, что расширение Stackify Retrace для Linux вложено в ресурс виртуальной машины. При вложении ресурса расширения JSON помещается в объект "resources": [] виртуальной машины.

Для работы расширения требуется `environment` и `activationKey`.

```json
    {
      "type": "extensions",
      "name": "StackifyExtension",
      "apiVersion": "[variables('apiVersion')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Compute/virtualMachines',variables('vmName'))]"
      ],
      "properties": {
        "publisher": "Stackify.LinuxAgent.Extension",
        "type": "StackifyLinuxAgentExtension",
        "typeHandlerVersion": "1.0",
        "autoUpgradeMinorVersion": true,
        "settings": {
          "environment": "myEnvironment"
        },
        "protectedSettings": {
          "activationKey": "myActivationKey"
        }
      }
    }      
```

При размещении JSON расширения в корне шаблона имя ресурса содержит ссылку на родительскую виртуальную машину, а тип отражает вложенную конфигурацию.

```json
    {
        "type": "Microsoft.Compute/virtualMachines/extensions",
        "name": "<parentVmResource>/StackifyExtension",
        "apiVersion": "[variables('apiVersion')]",
        "location": "[resourceGroup().location]",
        "dependsOn": [
            "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
        ],
        "properties": {
            "publisher": "Stackify.LinuxAgent.Extension",
            "type": "StackifyLinuxAgentExtension",
            "typeHandlerVersion": "1.0",
            "autoUpgradeMinorVersion": true,
            "settings": {
              "environment": "myEnvironment"
            },
            "protectedSettings": {
              "activationKey": "myActivationKey"
            }
        }
    }
```


## <a name="powershell-deployment"></a>Развертывание с помощью PowerShell

Команду `Set-AzVMExtension` можно использовать для развертывания расширения агента Stackify Retrace для Linux на существующей виртуальной машине. Перед выполнением команды необходимо сохранить открытые и закрытые конфигурации в хэш-таблице PowerShell.

Для работы расширения требуется `environment` и `activationKey`.

```powershell
$PublicSettings = @{"environment" = "myEnvironment"}
$ProtectedSettings = @{"activationKey" = "myActivationKey"}

Set-AzVMExtension -ExtensionName "Stackify.LinuxAgent.Extension" `
    -ResourceGroupName "myResourceGroup" `
    -VMName "myVM" `
    -Publisher "Stackify.LinuxAgent.Extension" `
    -ExtensionType "StackifyLinuxAgentExtension" `
    -TypeHandlerVersion 1.0 `
    -Settings $PublicSettings `
    -ProtectedSettings $ProtectedSettings `
    -Location WestUS `
```

## <a name="azure-cli-deployment"></a>Развертывание с помощью Azure CLI 

Инструмент Azure CLI можно использовать для развертывания расширения агента Stackify Retrace для Linux на существующей виртуальной машине.  

Для работы расширения требуется `environment` и `activationKey`.

```azurecli
az vm extension set --publisher 'Stackify.LinuxAgent.Extension' --version 1.0 --name 'StackifyLinuxAgentExtension' --protected-settings '{"activationKey":"myActivationKey"}' --settings '{"environment":"myEnvironment"}'  --resource-group 'myResourceGroup' --vm-name 'myVmName'
```

## <a name="troubleshoot-and-support"></a>Устранение неполадок и поддержка

### <a name="error-codes"></a>Коды ошибок

| Код ошибки | Значение | Возможное действие |
| :---: | --- | --- |
| 10 | Ошибка при установке | Требуется компонент wget. |
| 20 | Ошибка при установке | Требуется компонент Python. |
| 30 | Ошибка при установке | Требуется компонент sudo. |
| 40 | Ошибка при установке | Требуется ключ активации. |
| 51 | Ошибка при установке | Дистрибутив ОС не поддерживается. |
| 60 | Ошибка при установке | Требуется указать среду. |
| 70 | Ошибка при установке | Неизвестно |
| 80 | Ошибка при включении | Не удалось установить службу. |
| 90 | Ошибка при включении | Не удалось запустить службу. |
| 100 | Ошибка при отключении | Не удалось остановить службу. |
| 110 | Ошибка при отключении | Не удалось удалить службу. |
| 120 | Ошибка при удалении | Не удалось остановить службу. |

Если вам нужна дополнительная помощь, обращайтесь в службу поддержки Stackify: https://support.stackify.com.