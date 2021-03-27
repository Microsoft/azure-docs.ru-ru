---
title: Агрегирование событий с помощью система диагностики Azure Windows
description: Ознакомьтесь со сведениями об агрегации и сборе событий с использованием WAD для мониторинга и диагностики кластеров Azure Service Fabric.
ms.topic: conceptual
ms.date: 04/03/2018
ms.openlocfilehash: bbc8efcb2600e1832ad8a37560ab231a4a7f3185
ms.sourcegitcommit: a9ce1da049c019c86063acf442bb13f5a0dde213
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/27/2021
ms.locfileid: "105626735"
---
# <a name="event-aggregation-and-collection-using-windows-azure-diagnostics"></a>Агрегирование и сбор событий с помощью Диагностики Azure для Windows
> [!div class="op_single_selector"]
> * [Windows](service-fabric-diagnostics-event-aggregation-wad.md)
> * [Linux](service-fabric-diagnostics-event-aggregation-lad.md)
>
>

Во время работы кластера Azure Service Fabric рекомендуется централизованно собирать журналы со всех узлов. Централизованное хранение журналов упрощает анализ и устранение неполадок в кластере, а также в приложениях и службах, работающих в этом кластере.

Один из способов отправки и сбора журналов заключается в использовании расширения Диагностики Azure для Windows (WAD), которое отправляет журналы в службу хранилища Azure, а также может отправлять журналы в Azure Application Insights или Центры событий Azure. Вы также можете использовать внешний процесс для чтения событий из хранилища и их размещения в продукте платформы анализа, например [Azure Monitor журналов](./service-fabric-diagnostics-oms-setup.md) или другого решения для анализа журналов.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные условия
В этом руководстве используются инструменты, представленные ниже.

* [Azure Resource Manager](../azure-resource-manager/management/overview.md)
* [Azure PowerShell](/powershell/azure/)
* [Шаблон Azure Resource Manager](../virtual-machines/extensions/diagnostics-template.md?toc=/azure/virtual-machines/windows/toc.json)

## <a name="service-fabric-platform-events"></a>События платформы Service Fabric
Service Fabric настраивает несколько [стандартных каналов ведения журнала](service-fabric-diagnostics-event-generation-infra.md), и для нескольких из них расширение предварительно настраивает отправку данных мониторинга и диагностики в таблицу хранилища или в другое расположение.
  * [Операционные события](service-fabric-diagnostics-event-generation-operational.md). операции более высокого уровня, выполняемые платформой Service Fabric. Некоторые примеры: создание приложений и служб, изменение состояния узлов и сведения об обновлении. Они передаются в рамках журналов трассировки событий для Windows (ETW).
  * [События модели программирования на основе Reliable Actors](service-fabric-reliable-actors-diagnostics.md).
  * [События модели программирования на основе Reliable Services](service-fabric-reliable-services-diagnostics.md).

## <a name="deploy-the-diagnostics-extension-through-the-portal"></a>Развертывание расширения системы диагностики с помощью портала
Для сбора журналов прежде всего нужно развернуть расширение системы диагностики на каждом узле масштабируемого набора виртуальных машин в кластере Service Fabric. Расширение системы диагностики собирает журналы на каждой виртуальной машине и отправляет их в указанную учетную запись хранения. Ниже описано, как настроить это для новых и существующих кластеров с помощью портала Azure и шаблонов Azure Resource Manager.

### <a name="deploy-the-diagnostics-extension-as-part-of-cluster-creation-through-azure-portal"></a>Развертывание расширения системы диагностики в ходе создания кластера с помощью портала Azure
При создании кластера, на этапе конфигурации кластера, разверните дополнительные параметры и убедитесь, что здесь **включен** параметр диагностики (это значение по умолчанию).

![Параметры системы диагностики Azure на портале для создания кластера](media/service-fabric-diagnostics-event-aggregation-wad/azure-enable-diagnostics-new.png)

Мы настоятельно рекомендуем скачать шаблон **прежде, чем вы щелкнете "Создать"** на последнем шаге. Дополнительную информацию см. в статье [Создание кластера Service Fabric с помощью Azure Resource Manager](service-fabric-cluster-creation-via-arm.md). Этот шаблон потребуется для настройки каналов (перечисленных выше), из которых вы намерены собирать данные.

![Шаблон кластера](media/service-fabric-diagnostics-event-aggregation-wad/download-cluster-template.png)

Теперь, когда вы собираете события в службе хранилища Azure, [Настройте журналы Azure Monitor](service-fabric-diagnostics-oms-setup.md) , чтобы получить подробные сведения и запросить их на портале Azure Monitor журналов.

>[!NOTE]
>Сейчас не поддерживается возможность фильтрации или очистки событий, которые отправляются в таблицы. Если вы не реализуете метод для удаления событий из таблицы, она будет постоянно расти. По умолчанию действует ограничение в 50 ГБ. Вы можете изменить это ограничение, используя [инструкции, приведенные ниже в этой статье](service-fabric-diagnostics-event-aggregation-wad.md#update-storage-quota). Кроме того, там есть пример службы для очистки данных, выполняющийся в [примере модуля наблюдения](https://github.com/Azure-Samples/service-fabric-watchdog-service). Мы рекомендуем создать свою службу, если у вас нет веских причин хранить журналы дольше 30 или 90 дней.



## <a name="deploy-the-diagnostics-extension-through-azure-resource-manager"></a>Развертывание расширения системы диагностики с помощью Azure Resource Manager

### <a name="create-a-cluster-with-the-diagnostics-extension"></a>Создание кластера с расширением системы диагностики
Чтобы создать кластер с помощью Resource Manager, добавьте код JSON с конфигурацией системы диагностики в полный шаблон Resource Manager. Мы предоставляем пример шаблона диспетчера ресурсов для кластера из пяти виртуальных машин с конфигурацией системы диагностики (эта конфигурация входит в примеры шаблонов диспетчера ресурсов). Этот шаблон можно найти в коллекции примеров Azure. См. статью [Пример шаблона Resource Manager — кластер из пяти узлов с системой диагностики](https://azure.microsoft.com/resources/templates/service-fabric-secure-cluster-5-node-1-nodetype/).

Чтобы просмотреть параметр системы диагностики в шаблоне Resource Manager, откройте файл azuredeploy.json и выполните поиск **IaaSDiagnostics**. Чтобы создать кластер с помощью этого шаблона, нажмите кнопку **Развернуть в Azure**, которая доступна по ссылке выше.

Также можно скачать пример шаблона Resource Manager, внести в него изменения и создать кластер на основе измененного шаблона с помощью команды `New-AzResourceGroupDeployment` в окне Azure PowerShell. Параметры, передаваемые в команду, приведены в коде ниже. Дополнительные инструкции по развертыванию группы ресурсов с помощью PowerShell см. в статье [Развертывание ресурсов с использованием шаблонов Resource Manager и Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md).

### <a name="add-the-diagnostics-extension-to-an-existing-cluster"></a>Добавление расширения системы диагностики к существующему кластеру
Если у вас есть кластер, в котором еще не развернута система диагностики, вы можете добавить или обновить систему диагностики с помощью шаблона кластера. Измените шаблон Resource Manager, который используется для создания существующего кластера, или скачайте шаблон на портале, как описано выше. Измените файл template.json, выполнив следующие действия.

Добавьте новый ресурс хранилища в шаблон, внеся изменения в раздел resources.

```json
{
    "apiVersion": "2018-07-01",
    "type": "Microsoft.Storage/storageAccounts",
    "name": "[parameters('applicationDiagnosticsStorageAccountName')]",
    "location": "[parameters('computeLocation')]",
    "sku": {
    "name": "[parameters('applicationDiagnosticsStorageAccountType')]"
    "tier": "standard"
  },
    "tags": {
    "resourceType": "Service Fabric",
    "clusterName": "[parameters('clusterName')]"
  }
},
```

 Затем добавьте этот ресурс в раздел parameters сразу после определений учетной записи хранения между `supportLogStorageAccountName`. Замените *имя учетной записи хранения* текста заполнителя именем учетной записи хранения, которую вы хотите.

```json
    "applicationDiagnosticsStorageAccountType": {
      "type": "string",
      "allowedValues": [
        "Standard_LRS",
        "Standard_GRS"
      ],
      "defaultValue": "Standard_LRS",
      "metadata": {
        "description": "Replication option for the application diagnostics storage account"
      }
    },
    "applicationDiagnosticsStorageAccountName": {
      "type": "string",
      "defaultValue": "**STORAGE ACCOUNT NAME GOES HERE**",
      "metadata": {
        "description": "Name for the storage account that contains application diagnostics data from the cluster"
      }
    },
```
Затем добавьте в раздел `VirtualMachineProfile` файла template.json следующий код в массиве extensions. Обязательно добавьте запятую в конце или в начале в зависимости от места вставки.

```json
{
    "name": "[concat(parameters('vmNodeType0Name'),'_Microsoft.Insights.VMDiagnosticsSettings')]",
    "properties": {
        "type": "IaaSDiagnostics",
        "autoUpgradeMinorVersion": true,
        "protectedSettings": {
        "storageAccountName": "[parameters('applicationDiagnosticsStorageAccountName')]",
        "storageAccountKey": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', parameters('applicationDiagnosticsStorageAccountName')),'2015-05-01-preview').key1]",
        "storageAccountEndPoint": "https://core.windows.net/"
        },
        "publisher": "Microsoft.Azure.Diagnostics",
        "settings": {
        "WadCfg": {
            "DiagnosticMonitorConfiguration": {
            "overallQuotaInMB": "50000",
            "EtwProviders": {
                "EtwEventSourceProviderConfiguration": [
                {
                    "provider": "Microsoft-ServiceFabric-Actors",
                    "scheduledTransferKeywordFilter": "1",
                    "scheduledTransferPeriod": "PT5M",
                    "DefaultEvents": {
                    "eventDestination": "ServiceFabricReliableActorEventTable"
                    }
                },
                {
                    "provider": "Microsoft-ServiceFabric-Services",
                    "scheduledTransferPeriod": "PT5M",
                    "DefaultEvents": {
                    "eventDestination": "ServiceFabricReliableServiceEventTable"
                    }
                }
                ],
                "EtwManifestProviderConfiguration": [
                {
                    "provider": "cbd93bc2-71e5-4566-b3a7-595d8eeca6e8",
                    "scheduledTransferLogLevelFilter": "Information",
                    "scheduledTransferKeywordFilter": "4611686018427387904",
                    "scheduledTransferPeriod": "PT5M",
                    "DefaultEvents": {
                    "eventDestination": "ServiceFabricSystemEventTable"
                    }
                },
                {
                    "provider": "02d06793-efeb-48c8-8f7f-09713309a810",
                    "scheduledTransferLogLevelFilter": "Information",
                    "scheduledTransferKeywordFilter": "4611686018427387904",
                    "scheduledTransferPeriod": "PT5M",
                    "DefaultEvents": {
                    "eventDestination": "ServiceFabricSystemEventTable"
                    }
                }
                ]
            }
            }
        },
        "StorageAccount": "[parameters('applicationDiagnosticsStorageAccountName')]"
        },
        "typeHandlerVersion": "1.5"
    }
}
```

После изменения файла template.json (как описано выше) повторно опубликуйте шаблон Resource Manager. Если шаблон экспортирован, для повторной публикации шаблона выполните файл deploy.ps1. После развертывания убедитесь, что параметр **ProvisioningState** имеет значение **Succeeded**.

> [!TIP]
> Если планируется развернуть контейнеры в кластере, разрешите WAD получать статистику Docker, добавив в раздел **WadCfg > DiagnosticMonitorConfiguration** следующий код:

```json
"DockerSources": {
    "Stats": {
        "enabled": true,
        "sampleRate": "PT1M"
    }
},
```

### <a name="update-storage-quota"></a>Изменение квоты хранилища

Это расширение постоянно увеличивает размер заполняемых таблиц, пока не будет достигнута квота хранилища. Возможно, вы захотите уменьшить эту квоту. По умолчанию квота имеет значение 50 ГБ, а изменить ее можно в шаблоне, в поле `overallQuotaInMB` раздела `DiagnosticMonitorConfiguration`.

```json
"overallQuotaInMB": "50000",
```

## <a name="log-collection-configurations"></a>Настройка сбора журналов
Для сбора доступны журналы из нескольких дополнительных каналов, и здесь мы опишем несколько распространенных конфигураций, которые можно применить в шаблоне для кластеров, работающих в Azure.

* Операционный канал — базовый: включается по умолчанию высокоуровневые операции, выполняемые Service Fabric и кластером, включая события для узла, развертывание нового приложения, откат обновления и т. д. Список событий см. в статье [события рабочего канала](./service-fabric-diagnostics-event-generation-operational.md).
  
```json
      scheduledTransferKeywordFilter: "4611686018427387904"
  ```
* Операционный канал — подробные сведения. Включает отчеты о работоспособности и решения о балансировке нагрузки, а также все содержимое операционного канала базовых сведений. Эти события создаются системой или кодом через API отчетов о работоспособности и (или) нагрузке, такие как [ReportPartitionHealth](/previous-versions/azure/reference/mt645153(v=azure.100)) или [ReportLoad](/previous-versions/azure/reference/mt161491(v=azure.100)). Чтобы просмотреть эти события в окне просмотра событий диагностики Visual Studio, добавьте Microsoft-ServiceFabric:4:0x4000000000000008 в список поставщиков трассировки событий Windows.

```json
      scheduledTransferKeywordFilter: "4611686018427387912"
  ```

* Канал данных и обмена сообщениями — базовые сведения. Содержит критически важные журналы и события, создаваемые в системе обмена сообщениями (пока только ReverseProxy) и пути обработки данных, а также подробные журналы операционного канала. Эти события также включают сбои при обработке запросов и другие критические проблемы для ReverseProxy и обрабатываемых запросов. **Мы рекомендуем использовать этот уровень для ведения подробного журнала**. Чтобы просмотреть эти события в средстве просмотра диагностических событий Visual Studio, добавьте Microsoft-ServiceFabric:4:0x4000000000000010 в список поставщиков трассировки событий Windows.

```json
      scheduledTransferKeywordFilter: "4611686018427387928"
  ```

* Канал данных и обмена сообщениями — подробные сведения. Этот канал содержит развернутую информацию из некритических журналов для процессов обработки данных и обмена сообщениями в кластере, а также все содержимое операционного канала подробных сведений. Подробные сведения об устранении любых неполадок, связанных с событиями обратного прокси-сервера, приведены в [руководстве по диагностике обратного прокси-сервера](service-fabric-reverse-proxy-diagnostics.md).  Чтобы просмотреть эти события в средстве просмотра диагностических событий Visual Studio, добавьте Microsoft-ServiceFabric:4:0x4000000000000020 в список поставщиков трассировки событий Windows.

```json
      scheduledTransferKeywordFilter: "4611686018427387944"
  ```

>[!NOTE]
>Этот канал содержит огромный объем сведений о событиях, поэтому включение сбора этих данных приводит к быстрому созданию множества трассировок, которые могут потреблять емкость хранилища. Включайте этот режим только при крайней необходимости.


Чтобы включить рекомендуемый режим подробного ведения журналов с наименьшими помехами (**Операционный канал базовых сведений**), используйте следующее значение `EtwManifestProviderConfiguration` для `WadCfg` в шаблоне :

```json
  "WadCfg": {
        "DiagnosticMonitorConfiguration": {
          "overallQuotaInMB": "50000",
          "EtwProviders": {
            "EtwEventSourceProviderConfiguration": [
              {
                "provider": "Microsoft-ServiceFabric-Actors",
                "scheduledTransferKeywordFilter": "1",
                "scheduledTransferPeriod": "PT5M",
                "DefaultEvents": {
                  "eventDestination": "ServiceFabricReliableActorEventTable"
                }
              },
              {
                "provider": "Microsoft-ServiceFabric-Services",
                "scheduledTransferPeriod": "PT5M",
                "DefaultEvents": {
                  "eventDestination": "ServiceFabricReliableServiceEventTable"
                }
              }
            ],
            "EtwManifestProviderConfiguration": [
              {
                "provider": "cbd93bc2-71e5-4566-b3a7-595d8eeca6e8",
                "scheduledTransferLogLevelFilter": "Information",
                "scheduledTransferKeywordFilter": "4611686018427387904",
                "scheduledTransferPeriod": "PT5M",
                "DefaultEvents": {
                  "eventDestination": "ServiceFabricSystemEventTable"
                }
              },
              {
                "provider": "02d06793-efeb-48c8-8f7f-09713309a810",
                "scheduledTransferLogLevelFilter": "Information",
                "scheduledTransferKeywordFilter": "4611686018427387904",
                "scheduledTransferPeriod": "PT5M",
                "DefaultEvents": {
                "eventDestination": "ServiceFabricSystemEventTable"
                }
              }
            ]
          }
        }
      },
```

## <a name="collect-from-new-eventsource-channels"></a>Сбор из новых каналов EventSource

Чтобы обновить службу диагностики для сбора журналов из новых каналов EventSource, представляющих новое приложение, которое вы собираетесь развернуть, выполните шаги, описанные выше, чтобы настроить службу диагностики для имеющегося кластера.

В раздел `EtwEventSourceProviderConfiguration` файла template.json нужно добавить записи для новых каналов EventSource. Это следует сделать до обновления конфигурации с помощью команды PowerShell `New-AzResourceGroupDeployment`. Имя источника события определяется как часть кода в файле ServiceEventSource.cs, созданном в Visual Studio.

Например, если источнику события задано имя My-Eventsource, добавьте следующий код для помещения событий из источника My-Eventsource в таблицу MyDestinationTableName.

```json
        {
            "provider": "My-Eventsource",
            "scheduledTransferPeriod": "PT5M",
            "DefaultEvents": {
            "eventDestination": "MyDestinationTableName"
            }
        }
```

Чтобы собрать данные счетчиков производительности и журналов событий, измените шаблон Resource Manager, используя примеры в статье [Создание виртуальной машины Windows с мониторингом и диагностикой с использованием шаблона Azure Resource Manager](../virtual-machines/extensions/diagnostics-template.md?toc=/azure/virtual-machines/windows/toc.json). Затем опубликуйте шаблон Resource Manager.

## <a name="collect-performance-counters"></a>Сбор данных счетчиков производительности

Для сбора метрик производительности из кластера добавьте счетчики производительности в элемент WadCfg > DiagnosticMonitorConfiguration в шаблоне Resource Manager для кластера. Дополнительные сведения о том, как изменить `WadCfg`, чтобы собрать данные конкретных счетчиков производительности, см. в статье [Performance monitoring with Windows Azure Diagnostics extension](service-fabric-diagnostics-perf-wad.md) (Мониторинг производительности с помощью расширения системы диагностики Microsoft Azure). Список счетчиков производительности Service Fabric, которые мы рекомендуем собирать, см. в статье [Метрики производительности](service-fabric-diagnostics-event-generation-perf.md).
  
Если вы используете приемник Application Insights, как описано в разделе ниже, и вам нужно, чтобы эти метрики отображались в Application Insights, добавьте имя приемника в соответствующем разделе, как показано выше. Это позволит автоматически отправлять счетчики производительности, отдельно настроенные для ресурса Application Insights.


## <a name="send-logs-to-application-insights"></a>Отправка журналов в Application Insights

### <a name="configuring-application-insights-with-wad"></a>Настройка Application Insights с помощью WAD

>[!NOTE]
>В настоящее время это распространяется только на кластеры Windows.

Существует два основных способа отправки данных из WAD в Azure Application Insights, что достигается путем добавления Application Insights приемника в конфигурацию WAD с помощью портал Azure или с помощью шаблона Azure Resource Manager.

#### <a name="add-an-application-insights-instrumentation-key-when-creating-a-cluster-in-azure-portal"></a>При создании кластера на портале Azure добавьте ключ инструментирования Application Insights.

![Добавление AIKey](media/service-fabric-diagnostics-event-analysis-appinsights/azure-enable-diagnostics.png)

Если при создании кластера диагностика включена, появляется дополнительное поле для ввода ключа инструментирования Application Insights. Если вставить сюда ключ Application Insights, в шаблоне Resource Manager, используемом для развертывания кластера, будет автоматически настроен приемник Application Insights.

#### <a name="add-the-application-insights-sink-to-the-resource-manager-template"></a>Добавление приемника Application Insights в шаблон Resource Manager

В разделе WadCfg шаблона Resource Manager добавьте приемник, внеся два указанных ниже изменения.

1. Добавьте конфигурацию приемника сразу после объявления `DiagnosticMonitorConfiguration`:

    ```json
    "SinksConfig": {
        "Sink": [
            {
                "name": "applicationInsights",
                "ApplicationInsights": "***ADD INSTRUMENTATION KEY HERE***"
            }
        ]
    }

    ```

2. Включите приемник в `DiagnosticMonitorConfiguration`, добавив следующую строку в `DiagnosticMonitorConfiguration``WadCfg` (сразу перед объявлением `EtwProviders`):

    ```json
    "sinks": "applicationInsights"
    ```

В обоих приведенных выше фрагментах кода для указания приемника использовано имя applicationInsights. Это не является обязательным. Если имя приемника включено в элемент sinks, именем может быть любая строка.

Сейчас журналы кластера отображаются в средстве просмотра журналов Application Insights как **трассировки**. Так как большинство трассировок, поступающих от платформы, имеют уровень "информационное", можно также рассмотреть возможность изменения конфигурации приемника, чтобы она отправляла только журналы типа "warning" или "Error". Для этого в приемник можно добавить каналы, как показано в [этой статье](../azure-monitor/agents/diagnostics-extension-to-application-insights.md).

>[!NOTE]
>Если вы укажете на портале или в шаблоне Resource Manager неправильный ключ Application Insights, придется вручную заменить его, а затем обновить или повторно развернуть кластер.

## <a name="next-steps"></a>Дальнейшие шаги

Если вы правильно настроили диагностику Azure, данные из журнала трассировки событий Windows и журнала EventSource станут появляться в таблице хранилища. Если вы решили использовать Azure Monitor журналы, Kibana или любую другую платформу аналитики и визуализации данных, которая не настроена непосредственно в шаблоне диспетчер ресурсов, обязательно настройте платформу для чтения данных из этих таблиц хранилища. Сделать это для журналов Azure Monitor довольно тривиальное, и оно объясняется в [анализе событий и журнала](service-fabric-diagnostics-event-analysis-oms.md). В этом смысле Application Insights — это особый случай, так как это решение можно настроить при настройке расширения диагностики. Дополнительные сведения об Application Insights см. в [этой статье](service-fabric-diagnostics-event-analysis-appinsights.md).

>[!NOTE]
>Сегодня не существует способа фильтрации или очистки событий, которые отправляются в таблицу. Если не реализовать метод удаления событий из таблицы, она продолжит расти. Сейчас есть пример службы очистки данных, выполняющийся в [примере модуля наблюдения](https://github.com/Azure-Samples/service-fabric-watchdog-service). Мы также советуем написать собственный пример, если у вас нет веских причин для хранения журналов дольше 30 или 90 дней.

* [Узнайте, как собирать данные счетчиков производительности или журналы, используя расширения системы диагностики.](../virtual-machines/extensions/diagnostics-template.md?toc=/azure/virtual-machines/windows/toc.json)
* [Анализ событий и визуализация с помощью Application Insights](service-fabric-diagnostics-event-analysis-appinsights.md)
* [Анализ событий и визуализация с помощью журналов Azure Monitor](service-fabric-diagnostics-event-analysis-oms.md)
* [Анализ событий и визуализация с помощью Application Insights](service-fabric-diagnostics-event-analysis-appinsights.md)
* [Анализ событий и визуализация с помощью журналов Azure Monitor](service-fabric-diagnostics-event-analysis-oms.md)
