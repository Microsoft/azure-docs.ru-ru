---
title: Установка и настройка расширения "Диагностика Microsoft Azure" (WAD)
description: Сведения об установке и настройке расширения системы диагностики Windows. Кроме того, ознакомьтесь с описанием того, как хранятся данные в учетной записи хранения Azure и.
services: azure-monitor
author: bwren
ms.topic: conceptual
ms.date: 02/17/2020
ms.author: bwren
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 174f372f9dbe8dc0449c7f9b9f5b34c6206f92de
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101708566"
---
# <a name="install-and-configure-windows-azure-diagnostics-extension-wad"></a>Установка и настройка расширения "Диагностика Microsoft Azure" (WAD)
[Расширение "Диагностика Microsoft Azure"](diagnostics-extension-overview.md) — это агент в Azure Monitor, собирающий данные мониторинга из операционной системы на виртуальной машине и рабочих нагрузок виртуальных машин Azure и других вычислительных ресурсов. В этой статье приводятся сведения об установке и настройке расширения диагностики Windows, а также описание того, как данные хранятся в учетной записи хранения и Azure.

Расширение диагностики реализовано как [расширение виртуальной машины](../../virtual-machines/extensions/overview.md) в Azure, поэтому оно поддерживает те же параметры установки с помощью шаблонов Resource Manager, PowerShell и CLI. Сведения об установке и обслуживании расширений виртуальных машин см. в статье [Обзор расширений и компонентов виртуальной машины под управлением Windows](../../virtual-machines/extensions/features-windows.md)

## <a name="overview"></a>Обзор
При настройке расширения "Диагностика Microsoft Azure" необходимо указать учетную запись хранения, в которую будут отправляться все указанные данные. При необходимости можно добавить один или несколько *приемников данных* для отправки данных в разные расположения.

- Приемник Azure Monitor служит для отправки данных о производительности операционной системы на виртуальной машине в метрики Azure Monitor.
- Приемник концентратора событий служит для отправки данных о производительности и журналах операционной системы на виртуальной машине в концентраторы событий Azure для пересылки за пределы Azure. Этот приемник можно настроить на портале Azure.


## <a name="install-with-azure-portal"></a>Установка с помощью портала Azure
Вы можете установить и настроить расширение диагностики на отдельной виртуальной машине на портале Azure, который служит интерфейсом, вместо работы напрямую с конфигурацией. При включении расширения диагностики оно будет автоматически использовать конфигурацию по умолчанию с наиболее распространенными счетчиками производительности и событиями. Эту конфигурацию по умолчанию можно изменить в соответствии с конкретными требованиями.

> [!NOTE]
> Ниже описаны наиболее распространенные параметры расширения диагностики. Дополнительные сведения обо всех параметрах конфигурации см. в статье [Схема расширения диагностики Windows](diagnostics-extension-schema-windows.md).

1. Откройте меню виртуальной машины на портале Azure.

2. В меню виртуальной машины в разделе **Мониторинг** выберите **Параметры диагностики**.

3. Щелкните **включить мониторинг на гостевом уровне**, если расширение диагностики еще не включено.

   ![Включение мониторинга](media/diagnostics-extension-windows-install/enable-monitoring.png)

4. Для виртуальной машины будет создана новая учетная запись службы хранилища Azure с именем, которое будет основываться на имени группы ресурсов для виртуальной машины, а также будет выбран набор журналов производительности операционной системы на виртуальной машине и счетчиков по умолчанию.

   ![Параметры диагностики](media/diagnostics-extension-windows-install/diagnostic-settings.png)

5. На вкладке **Счетчики производительности** выберите метрики гостевой ОС, которые вы хотите получать с этой виртуальной машины. Для более расширенного выбора используйте параметр **Другие**.

   ![Счетчики производительности](media/diagnostics-extension-windows-install/performance-counters.png)

6. На вкладке **Журналы** выберите журналы, которые нужно получить с виртуальной машины. Журналы можно отправлять в хранилище или в концентраторы событий, но не в Azure Monitor. Используйте [агент Log Analytics](../agents/log-analytics-agent.md), чтобы получить журналы гостевой ОС в Azure Monitor.

   ![На снимке экрана показана вкладка журналы с разными журналами, выбранными для виртуальной машины.](media/diagnostics-extension-windows-install/logs.png)

7. На вкладке **Аварийные дампы** укажите процессы, позволяющие получать дампы памяти после сбоя. Данные будут записаны в учетную запись хранения для параметра диагностики. При необходимости можно указать контейнер больших двоичных объектов.

   ![Аварийные дампы](media/diagnostics-extension-windows-install/crash-dumps.png)

8. На вкладке **Приемники** укажите, следует ли передавать данные в расположения, отличные от службы хранилища Azure. Если выбрать **Azure Monitor**, данные о производительности гостевой ОС будут отправляться в метрики Azure Monitor. Приемник концентраторов событий невозможно настроить с помощью портала Azure.

   ![На снимке экрана показана вкладка приемники с включенным параметром отправить диагностические данные в Azure Monitor.](media/diagnostics-extension-windows-install/sinks.png)
   
   Если вы не включили назначенное системой удостоверение, настроенное для виртуальной машины, при сохранении конфигурации с приемником Azure Monitor может отобразиться следующее предупреждение. Щелкните баннер, чтобы включить назначенное системой удостоверение.
   
   ![Управляемый объект](media/diagnostics-extension-windows-install/managed-entity.png)

9. На вкладке **Агент** можно изменить учетную запись хранения, задать квоту диска и указать, следует ли выполнять сбор журналов инфраструктуры диагностики.  

   ![На снимке экрана показана вкладка агент с параметром для задания учетной записи хранения.](media/diagnostics-extension-windows-install/agent.png)

10. Чтобы сохранить конфигурацию, нажмите кнопку **Сохранить**. 

> [!NOTE]
> Несмотря на то, что конфигурация для расширения диагностики может быть в формате JSON или XML, любая конфигурация, настроенная на портале Azure, всегда будет храниться в формате JSON. Если вы используете формат XML для другого метода конфигурации, а затем изменяете конфигурацию с помощью портала Azure, параметры будут заменены на JSON.

## <a name="resource-manager-template"></a>Шаблон Resource Manager
Сведения о развертывании расширения диагностики с помощью шаблонов Azure Resource Manager см. в статье [Использование мониторинга и системы диагностики с виртуальной машиной Windows и шаблонами Azure Resource Manager](../../virtual-machines/extensions/diagnostics-template.md). 

## <a name="azure-cli-deployment"></a>Развертывание с помощью Azure CLI
Azure CLI можно использовать для развертывания расширения системы диагностики Azure на существующей виртуальной машине с помощью [az vm extension set](/cli/azure/vm/extension#az-vm-extension-set), как в примере ниже. 

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name IaaSDiagnostics \
  --publisher Microsoft.Azure.Diagnostics \
  --protected-settings protected-settings.json \
  --settings public-settings.json 
```

Защищенные параметры определяются в элементе [PrivateConfig](diagnostics-extension-schema-windows.md#privateconfig-element) схемы конфигурации. Ниже приведен минимальный пример файла защищенных параметров, который определяет учетную запись хранения. Подробные сведения о частных параметрах см. в разделе [Пример конфигурации](diagnostics-extension-schema-windows.md#privateconfig-element).

```JSON
{
    "storageAccountName": "mystorageaccount",
    "storageAccountKey": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "storageAccountEndPoint": "https://mystorageaccount.blob.core.windows.net"
}
```

Общедоступные параметры определяются в элементе [Public](diagnostics-extension-schema-windows.md#publicconfig-element) схемы конфигурации. Ниже приведен минимальный пример файла общедоступных параметров, который включает сбор журналов инфраструктуры диагностики, один счетчик производительности и один журнал событий. Подробные сведения об общедоступных параметрах см. в разделе [Пример конфигурации](diagnostics-extension-schema-windows.md#publicconfig-element).

```JSON
{
  "StorageAccount": "mystorageaccount",
  "WadCfg": {
    "DiagnosticMonitorConfiguration": {
      "overallQuotaInMB": 5120,
      "PerformanceCounters": {
        "scheduledTransferPeriod": "PT1M",
        "PerformanceCounterConfiguration": [
          {
            "counterSpecifier": "\\Processor Information(_Total)\\% Processor Time",
            "unit": "Percent",
            "sampleRate": "PT60S"
          }
        ]
      },
      "WindowsEventLog": {
        "scheduledTransferPeriod": "PT1M",
        "DataSource": [
          {
            "name": "Application!*[System[(Level=1 or Level=2 or Level=3)]]"
          }
        ]
      }
    }
  }
}
```



## <a name="powershell-deployment"></a>Развертывание с помощью PowerShell
PowerShell можно использовать для развертывания расширения системы диагностики Azure на существующей виртуальной машине с помощью [Set-AzVMDiagnosticsExtension](/powershell/module/servicemanagement/azure.service/set-azurevmdiagnosticsextension), как в примере ниже. 

```powershell
Set-AzVMDiagnosticsExtension -ResourceGroupName "myvmresourcegroup" `
  -VMName "myvm" `
  -DiagnosticsConfigurationPath "DiagnosticsConfiguration.json"
```

Частные параметры определяются в элементе [PrivateConfig](diagnostics-extension-schema-windows.md#privateconfig-element), а общедоступные параметры определяются в элементе [Public](diagnostics-extension-schema-windows.md#publicconfig-element) схемы конфигурации. Также можно указать сведения об учетной записи хранения в качестве параметров командлета Set-AzVMDiagnosticsExtension, а не включать их в частные параметры.

Ниже приведен минимальный пример файла конфигурации, который включает сбор журналов инфраструктуры диагностики, один счетчик производительности и один журнал событий. Подробные сведения об общедоступных и частных параметрах см. в разделе [Пример конфигурации](diagnostics-extension-schema-windows.md#publicconfig-element). 

```JSON
{
    "PublicConfig": {
        "WadCfg": {
            "DiagnosticMonitorConfiguration": {
                "overallQuotaInMB": 10000,
                "DiagnosticInfrastructureLogs": {
                    "scheduledTransferLogLevelFilter": "Error"
                },
                "PerformanceCounters": {
                    "scheduledTransferPeriod": "PT1M",
                    "PerformanceCounterConfiguration": [
                        {
                            "counterSpecifier": "\\Processor(_Total)\\% Processor Time",
                            "sampleRate": "PT3M",
                            "unit": "percent"
                        }
                    ]
                },
                "WindowsEventLog": {
                    "scheduledTransferPeriod": "PT1M",
                        "DataSource": [
                        {
                            "name": "Application!*[System[(Level=1 or Level=2 or Level=3)]]"
                        }
                    ]
                }
            }
        },
        "StorageAccount": "mystorageaccount",
        "StorageType": "TableAndBlob"
    },
    "PrivateConfig": {
        "storageAccountName": "mystorageaccount",
        "storageAccountKey": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
        "storageAccountEndPoint": "https://mystorageaccount.blob.core.windows.net"
    }
}
```

См. также [Включение системы диагностики Azure на виртуальной машине под управлением Windows с помощью PowerShell](../../virtual-machines/extensions/diagnostics-windows.md).

## <a name="data-storage"></a>Хранилище данных
В следующей таблице перечислены различные типы данных, собираемых из расширения диагностики, а также сведения о том, хранятся ли они в виде таблицы или большого двоичного объекта. Данные, хранящиеся в таблицах, также могут храниться в больших двоичных объектах в зависимости от [параметра StorageType](diagnostics-extension-schema-windows.md#publicconfig-element) в общедоступной конфигурации.


| Данные | Тип хранилища | Описание |
|:---|:---|:---|
| WADDiagnosticInfrastructureLogsTable | Таблица | Сведения об изменениях в мониторе диагностики и конфигурации. |
| WADDirectoriesTable | Таблица | Сведения о каталогах, которые отслеживает монитор диагностики.  Сюда входят журналы IIS, журналы неудачно завершенных запросов IIS и пользовательские папки.  Расположение файла журнала BLOB-объекта указано в поле Container, а имя большого двоичного объекта — в поле RelativePath.  В поле AbsolutePath указано расположение и имя файла на виртуальной машине Azure. |
| WadLogsTable | Таблица | Включенные в код журналы с данными прослушивателя трассировки. |
| WADPerformanceCountersTable | Таблица | Счетчики производительности. |
| WADWindowsEventLogsTable | Таблица | Журналы событий Windows. |
| wad-iis-failedreqlogfiles | BLOB-объект | Содержит сведения журналов неудачно завершенных запросов IIS. |
| wad-iis-logfiles | BLOB-объект | Содержит сведения о журналах IIS. |
| "custom" | BLOB-объект | Пользовательский контейнер на основе каталогов конфигурации, которые отслеживаются монитором диагностики.  Имя этого контейнера BLOB-объектов указывается в элементе WADDirectoriesTable. |

## <a name="tools-to-view-diagnostic-data"></a>Средства для просмотра диагностических данных
Данные после их передачи в хранилище можно просмотреть с помощью нескольких средств. Пример:

* Обозреватель сервера в Visual Studio. Если установлены инструменты Azure для Microsoft Visual Studio, можно использовать узел хранилища Azure в обозревателе сервера для просмотра больших двоичных объектов, доступных только для чтения, и табличных данных из учетных записей хранения Azure. Вы можете отображать данные из своей локальной учетной записи эмулятора хранения, а также из учетных записей хранения, созданных для Azure. Дополнительные сведения см. в статье [Обзор ресурсов хранилища с помощью обозревателя сервера и управление ими](/visualstudio/azure/vs-azure-tools-storage-resources-server-explorer-browse-manage).
* [Обозреватель службы хранилища Microsoft Azure](../../vs-azure-tools-storage-manage-with-storage-explorer.md) — это автономное приложение, которое упрощает работу с данными из службы хранилища Azure на платформе Windows, OSX и Linux.
* [Azure Management Studio](https://www.cerebrata.com/products/azure-management-studio/introduction) содержит диспетчер диагностики Azure, который позволяет просматривать и скачивать данные диагностики, которые собирают выполняемые в Azure приложения, а также управлять этими данными.

## <a name="next-steps"></a>Дальнейшие действия
- Дополнительные сведения о пересылке данных мониторинга в Центры событий Azure см. в статье [Отправка данных из расширения "Диагностика Microsoft Azure" в Центры событий Azure](diagnostics-extension-stream-event-hubs.md).
