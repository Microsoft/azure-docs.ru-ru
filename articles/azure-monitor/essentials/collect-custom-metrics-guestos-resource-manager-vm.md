---
title: Сбор метрик виртуальной машины Windows в Azure Monitor с помощью шаблона
description: Отправка метрик гостевой ОС в хранилище базы метрик Azure Monitor с помощью шаблона Resource Manager для виртуальной машины Windows
author: anirudhcavale
services: azure-monitor
ms.topic: conceptual
ms.date: 05/04/2020
ms.author: bwren
ms.openlocfilehash: 8e510cf2e6fed9f9ffdec1dcc4dacf16a866d66b
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102049021"
---
# <a name="send-guest-os-metrics-to-the-azure-monitor-metric-store-by-using-an-azure-resource-manager-template-for-a-windows-virtual-machine"></a>Отправка метрик гостевой ОС в хранилище метрик Azure Monitor с помощью шаблона Azure Resource Manager для виртуальной машины Windows
Данные о производительности из гостевых ОС виртуальных машин Azure не собираются автоматически, как другие [метрики платформы](./monitor-azure-resource.md#monitoring-data). Установите [расширение диагностики](../agents/diagnostics-extension-overview.md) Azure Monitor, чтобы собирать метрики гостевой ОС в базу данных метрик и использовать их со всеми возможностями метрик Azure Monitor, включая оповещения почти в режиме реального времени, диаграммы, маршрутизацию и доступ из REST API. В этой статье описывается процесс отправки метрик производительности гостевой ОС с виртуальной машины под управлением Windows в базу данных метрик с использованием шаблона Resource Manager. 

> [!NOTE]
> Дополнительные сведения о настройке расширения диагностики для сбора метрик гостевой ОС с помощью портала Azure см. в разделе [Установка и настройка расширения системы диагностики Windows Azure (WAD)](../agents/diagnostics-extension-windows-install.md).


Если вы не знакомы с шаблонами Resource Manager, изучите сведения о [развертывании шаблонов](../../azure-resource-manager/management/overview.md), их структуре и синтаксисе.

## <a name="prerequisites"></a>Предварительные требования

- Подписку необходимо зарегистрировать в [Microsoft.Insights](../../azure-resource-manager/management/resource-providers-and-types.md).

- Необходимо установить [Azure PowerShell](/powershell/azure) или [Azure Cloud Shell](../../cloud-shell/overview.md).

- Ресурс виртуальной машины должен находиться в [регионе, поддерживающем пользовательские метрики](./metrics-custom-overview.md#supported-regions). 


## <a name="set-up-azure-monitor-as-a-data-sink"></a>Настройка Azure Monitor в качестве приемника данных
Расширение системы диагностики Azure использует функцию, называемую "приемники данных", для маршрутизации метрик и журналов в различные расположения. Ниже показано, как с помощью шаблона Resource Manager и PowerShell развернуть виртуальную машину, используя новый приемник данных Azure Monitor.

## <a name="author-resource-manager-template"></a>Создание шаблона Resource Manager
В этом примере можно использовать общедоступный пример шаблона. Начальные шаблоны можно найти по адресу https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-windows.

- **Azuredeploy.json** — это предварительно настроенный шаблон Resource Manager для развертывания виртуальной машины.

- **Azuredeploy.parameters.json** — это файл параметров, в котором хранятся сведения, такие как имя пользователя и пароль, которые вам нужно будет задать для виртуальной машины. Во время развертывания шаблона Resource Manager используются параметры, заданные в этом файле.

Скачайте и сохраните оба файла в локальном расположении.

### <a name="modify-azuredeployparametersjson"></a>Изменение файла azuredeploy.parameters.json
Откройте файл *azuredeploy.parameters.json*.

1. Введите значения **adminUsername** и **adminPassword** для виртуальной машины. Эти параметры используются для удаленного доступа к виртуальной машине. НЕ ИСПОЛЬЗУЙТЕ значения, указанные в этом шаблоне, во избежание перехвата виртуальной машины. Боты сканируют Интернет на наличие имен пользователей и паролей в общедоступных репозиториях GitHub. Вероятно, они будут тестировать виртуальные машины с этими значениями по умолчанию.

1. Создайте уникальный параметр dnsname для виртуальной машины.

### <a name="modify-azuredeployjson"></a>Изменение файла azuredeploy.json

Откройте файл *azuredeploy.json*.

Добавьте идентификатор учетной записи хранения в раздел **variables** шаблона после записи для **storageAccountName**.

```json
// Find these lines.
"variables": {
    "storageAccountName": "[concat(uniquestring(resourceGroup().id), 'sawinvm')]",

// Add this line directly below.
    "accountid": "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]",
```

Добавьте расширение "Управляемое удостоверение службы" (MSI) в шаблон в верхней части раздела **resources**. Это расширение гарантирует, что Azure Monitor принимает исходящие метрики.

```json
//Find this code.
"resources": [
// Add this code directly below.
    {
        "type": "Microsoft.Compute/virtualMachines/extensions",
        "name": "[concat(variables('vmName'), '/', 'WADExtensionSetup')]",
        "apiVersion": "2017-12-01",
        "location": "[resourceGroup().location]",
        "dependsOn": [
            "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]" ],
        "properties": {
            "publisher": "Microsoft.ManagedIdentity",
            "type": "ManagedIdentityExtensionForWindows",
            "typeHandlerVersion": "1.0",
            "autoUpgradeMinorVersion": true,
            "settings": {
                "port": 50342
            }
        }
    },
```

Добавьте конфигурацию **identity** к ресурсу виртуальной машины, чтобы убедиться, что Azure присваивает системное удостоверение расширению MSI. Этот шаг гарантирует, что виртуальная машина может передавать гостевые метрики о себе в Azure Monitor.

```json
// Find this section
                "subnet": {
            "id": "[variables('subnetRef')]"
            }
        }
        }
    ]
    }
},
{
    "apiVersion": "2017-03-30",
    "type": "Microsoft.Compute/virtualMachines",
    "name": "[variables('vmName')]",
    "location": "[resourceGroup().location]",
    // add these 3 lines below
    "identity": {
    "type": "SystemAssigned"
    },
    //end of added lines
    "dependsOn": [
    "[resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
    "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
    ],
    "properties": {
    "hardwareProfile": {
    ...
```

Добавьте следующую конфигурацию, чтобы включить расширение диагностики на виртуальной машине Windows. Для простой виртуальной машины на основе Resource Manager можно добавить конфигурацию расширения в массив ресурсов для соответствующей виртуальной машины. Строка sinks &mdash; AzMonSink и соответствующий SinksConfig, как показано далее в разделе &mdash; позволяет расширению выводить метрики напрямую в Azure Monitor. При желании счетчики производительности можно добавлять или удалять.


```json
        "networkProfile": {
            "networkInterfaces": [
            {
                "id": "[resourceId('Microsoft.Network/networkInterfaces',variables('nicName'))]"
            }
            ]
        },
"diagnosticsProfile": {
    "bootDiagnostics": {
    "enabled": true,
    "storageUri": "[reference(resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))).primaryEndpoints.blob]"
    }
}
},
//Start of section to add
"resources": [
{
            "type": "Microsoft.Compute/virtualMachines/extensions",
            "name": "[concat(variables('vmName'), '/', 'Microsoft.Insights.VMDiagnosticsSettings')]",
            "apiVersion": "2017-12-01",
            "location": "[resourceGroup().location]",
            "dependsOn": [
            "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
            ],
            "properties": {
            "publisher": "Microsoft.Azure.Diagnostics",
            "type": "IaaSDiagnostics",
            "typeHandlerVersion": "1.12",
            "autoUpgradeMinorVersion": true,
            "settings": {
                "WadCfg": {
                "DiagnosticMonitorConfiguration": {
    "overallQuotaInMB": 4096,
    "DiagnosticInfrastructureLogs": {
                    "scheduledTransferLogLevelFilter": "Error"
        },
                    "Directories": {
                    "scheduledTransferPeriod": "PT1M",
    "IISLogs": {
                        "containerName": "wad-iis-logfiles"
                    },
                    "FailedRequestLogs": {
                        "containerName": "wad-failedrequestlogs"
                    }
                    },
                    "PerformanceCounters": {
                    "scheduledTransferPeriod": "PT1M",
                    "sinks": "AzMonSink",
                    "PerformanceCounterConfiguration": [
                        {
                        "counterSpecifier": "\\Memory\\Available Bytes",
                        "sampleRate": "PT15S"
                        },
                        {
                        "counterSpecifier": "\\Memory\\% Committed Bytes In Use",
                        "sampleRate": "PT15S"
                        },
                        {
                        "counterSpecifier": "\\Memory\\Committed Bytes",
                        "sampleRate": "PT15S"
                        }
                    ]
                    },
                    "WindowsEventLog": {
                    "scheduledTransferPeriod": "PT1M",
                    "DataSource": [
                        {
                        "name": "Application!*"
                        }
                    ]
                    },
                    "Logs": {
                    "scheduledTransferPeriod": "PT1M",
                    "scheduledTransferLogLevelFilter": "Error"
                    }
                },
                "SinksConfig": {
                    "Sink": [
                    {
                        "name" : "AzMonSink",
                        "AzureMonitor" : {}
                    }
                    ]
                }
                },
                "StorageAccount": "[variables('storageAccountName')]"
            },
            "protectedSettings": {
                "storageAccountName": "[variables('storageAccountName')]",
                "storageAccountKey": "[listKeys(variables('accountid'),'2015-06-15').key1]",
                "storageAccountEndPoint": "https://core.windows.net/"
            }
            }
        }
        ]
//End of section to add
```


Сохраните и закройте оба файла.


## <a name="deploy-the-resource-manager-template"></a>Развертывание шаблона Resource Manager

> [!NOTE]
> Необходимо использовать расширение системы диагностики Azure версии 1.5 или более поздней, А ТАКЖЕ задать для свойства **autoUpgradeMinorVersion** значение true в шаблоне Resource Manager. Затем Azure загрузит нужное расширение при запуске виртуальной машины. Если этих параметров нет в шаблоне, внесите их и повторно разверните шаблон.


Для развертывания шаблона Resource Manager мы используем Azure PowerShell.

1. Запустите PowerShell.
1. Войдите в Azure, используя команду `Login-AzAccount`.
1. Получите список подписок с помощью командлета `Get-AzSubscription`.
1. Задайте подписку, которая используется для создания и (или) обновления виртуальной машины:

   ```powershell
   Select-AzSubscription -SubscriptionName "<Name of the subscription>"
   ```
1. Чтобы создать группу ресурсов для развертываемой виртуальной машины, выполните указанную ниже команду.

   ```powershell
    New-AzResourceGroup -Name "<Name of Resource Group>" -Location "<Azure Region>"
   ```
   > [!NOTE]
   > Не забывайте [использовать регион Azure, включенный для пользовательских метрик](./metrics-custom-overview.md).

1. Выполните следующие команды, чтобы развернуть виртуальную машину на основе шаблона Resource Manager.
   > [!NOTE]
   > Если вы хотите обновить имеющуюся виртуальную машину, просто добавьте *-Mode Incremental* в конце следующей команды.

   ```powershell
   New-AzResourceGroupDeployment -Name "<NameThisDeployment>" -ResourceGroupName "<Name of the Resource Group>" -TemplateFile "<File path of your Resource Manager template>" -TemplateParameterFile "<File path of your parameters file>"
   ```

1. После успешного завершения развертывания виртуальная машина должна появиться на портале Azure и начать передачу метрик в Azure Monitor.

   > [!NOTE]
   > С выбранным параметром vmSkuSize могут возникнуть ошибки. В этом случае вернитесь к файлу azuredeploy.json и измените значение по умолчанию для параметра vmSkuSize. В этом случае рекомендуем попробовать значение Standard_DS1_v2.

## <a name="chart-your-metrics"></a>Создание диаграммы метрик

1. Войдите на портал Azure.

2. В меню слева выберите **Монитор**.

3. На странице "Монитор" щелкните **Метрики**.

   ![Страница "Метрики"](media/collect-custom-metrics-guestos-resource-manager-vm/metrics.png)

4. Измените период агрегирования на **Последние 30 минут**.

5. В раскрывающемся списке ресурсов выберите только что созданную виртуальную машину. Если вы не изменяли имя в шаблоне, оно должно быть *SimpleWinVM2*.

6. В раскрывающемся меню пространств имен выберите **azure.vm.windows.guest**.

7. В раскрывающемся меню метрик выберите **Память > \%выделенных байт в использовании**.


## <a name="next-steps"></a>Дальнейшие действия
- Дополнительные сведения о настраиваемых метриках см. в [этой статье](./metrics-custom-overview.md).