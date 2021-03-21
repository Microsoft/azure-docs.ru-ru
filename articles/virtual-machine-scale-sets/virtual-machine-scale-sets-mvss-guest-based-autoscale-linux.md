---
title: Использование автомасштабирования Azure с метриками гостей в шаблоне масштабируемого набора Linux
description: Узнайте, как выполнять автоматическое масштабирование, используя метрики гостевой виртуальной машины в шаблоне масштабируемого набора виртуальных машин Linux
author: ju-shim
ms.author: jushiman
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.subservice: autoscale
ms.date: 04/26/2019
ms.reviewer: avverma
ms.custom: avverma
ms.openlocfilehash: 0605780651e1a3c54ae53d13a3f99e1124fa76db
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100585006"
---
# <a name="autoscale-using-guest-metrics-in-a-linux-scale-set-template"></a>Автоматическое масштабирование с помощью метрик гостевой виртуальной машины в шаблоне масштабируемого набора Linux

В Azure есть два основных типа метрик, которые собираются из виртуальных машин и масштабируемых наборов: метрики узла и гостевые метрики. На высоком уровне, если вы хотите использовать стандартные метрики ЦП, диска и сети, вполне подходит метрики узла. Однако если вам нужны более крупные метрики, необходимо выполнить поиск метрик гостей.

Метрики узла не нуждаются в дополнительной настройке, так как они собираются виртуальной машиной узла, а для метрик гостя требуется установить [расширение система диагностики Azure Windows](../virtual-machines/extensions/diagnostics-template.md) или [расширение система диагностики Azure Linux](../virtual-machines/extensions/diagnostics-linux.md) на гостевой виртуальной машине. Одна из основных причин использования метрик гостевой виртуальной машины вместо метрик узла заключается в том, что метрики гостевой виртуальной машины предоставляют больший набор показателей, чем метрики узла. Один из таких примеров — это метрики потребления памяти, которые доступны только через метрики гостевой виртуальной машины. Поддерживаемые метрики узла перечислены [здесь](../azure-monitor/essentials/metrics-supported.md), а часто используемые метрики гостевой виртуальной машины — [здесь](../azure-monitor/autoscale/autoscale-common-metrics.md). В этой статье показано, как изменить [шаблон базового приемлемого масштабируемого набора](virtual-machine-scale-sets-mvss-start.md) , чтобы использовать правила автомасштабирования на основе метрик гостевой виртуальной машины для масштабируемых наборов Linux.

## <a name="change-the-template-definition"></a>Изменение определения шаблона

В [предыдущей статье](virtual-machine-scale-sets-mvss-start.md) мы создали шаблон базового масштабируемого набора. Теперь мы будем использовать этот шаблон и изменим его, чтобы создать шаблон, который развертывает масштабируемый набор Linux с автомасштабированием на основе метрик гостя.

Сначала добавьте параметры для `storageAccountName` и `storageAccountSasToken`. Агент диагностики хранит данные метрик в [таблице](../cosmos-db/tutorial-develop-table-dotnet.md) в этой учетной записи хранения. Начиная с агента диагностики Linux версии 3.0 использование ключа доступа к хранилищу больше не поддерживается. Вместо этого используйте [маркер SAS](../storage/common/storage-sas-overview.md).

```diff
     },
     "adminPassword": {
       "type": "securestring"
+    },
+    "storageAccountName": {
+      "type": "string"
+    },
+    "storageAccountSasToken": {
+      "type": "securestring"
     }
   },
```

Затем измените масштабируемый набор `extensionProfile`, чтобы включить расширение диагностики. В этой конфигурации укажите идентификатор ресурса масштабируемого набора, который будет использоваться для сбора метрик, а также учетную запись хранения и маркер SAS для хранения метрик. Укажите то, как часто будут вычисляться метрики (в данном случае — каждую минуту) и какие метрики нужно отслеживать (в этом случае — это процент используемой памяти). Дополнительные сведения об этой конфигурации и других метриках см. в [этой документации](../virtual-machines/extensions/diagnostics-linux.md).

```diff
                 }
               }
             ]
+          },
+          "extensionProfile": {
+            "extensions": [
+              {
+                "name": "LinuxDiagnosticExtension",
+                "properties": {
+                  "publisher": "Microsoft.Azure.Diagnostics",
+                  "type": "LinuxDiagnostic",
+                  "typeHandlerVersion": "3.0",
+                  "settings": {
+                    "StorageAccount": "[parameters('storageAccountName')]",
+                    "ladCfg": {
+                      "diagnosticMonitorConfiguration": {
+                        "performanceCounters": {
+                          "sinks": "WADMetricJsonBlob",
+                          "performanceCounterConfiguration": [
+                            {
+                              "unit": "percent",
+                              "type": "builtin",
+                              "class": "memory",
+                              "counter": "percentUsedMemory",
+                              "counterSpecifier": "/builtin/memory/percentUsedMemory",
+                              "condition": "IsAggregate=TRUE"
+                            }
+                          ]
+                        },
+                        "metrics": {
+                          "metricAggregation": [
+                            {
+                              "scheduledTransferPeriod": "PT1M"
+                            }
+                          ],
+                          "resourceId": "[resourceId('Microsoft.Compute/virtualMachineScaleSets', 'myScaleSet')]"
+                        }
+                      }
+                    }
+                  },
+                  "protectedSettings": {
+                    "storageAccountName": "[parameters('storageAccountName')]",
+                    "storageAccountSasToken": "[parameters('storageAccountSasToken')]",
+                    "sinksConfig": {
+                      "sink": [
+                        {
+                          "name": "WADMetricJsonBlob",
+                          "type": "JsonBlob"
+                        }
+                      ]
+                    }
+                  }
+                }
+              }
+            ]
           }
         }
       }
```

Наконец, добавьте ресурс `autoscaleSettings`, чтобы настроить автоматическое масштабирование на основе этих метрик. Для ресурса указано предложение `dependsOn`, которое ссылается на масштабируемый набор, чтобы убедиться, что масштабируемый набор имеется, прежде чем пытаться его масштабировать. Если бы для автоматического масштабирования была выбрана другая метрика, вы бы использовали `counterSpecifier` из конфигурации расширения диагностики в качестве `metricName` в конфигурации автоматического масштабирования. Дополнительные сведения о настройке автомасштабирования см. в статье [Рекомендации по автомасштабированию](../azure-monitor/autoscale/autoscale-best-practices.md) и [Создание или обновление параметров автоматического масштабирования](/rest/api/monitor/autoscalesettings).

```diff
+    },
+    {
+      "type": "Microsoft.Insights/autoscaleSettings",
+      "apiVersion": "2015-04-01",
+      "name": "guestMetricsAutoscale",
+      "location": "[resourceGroup().location]",
+      "dependsOn": [
+        "Microsoft.Compute/virtualMachineScaleSets/myScaleSet"
+      ],
+      "properties": {
+        "name": "guestMetricsAutoscale",
+        "targetResourceUri": "[resourceId('Microsoft.Compute/virtualMachineScaleSets', 'myScaleSet')]",
+        "enabled": true,
+        "profiles": [
+          {
+            "name": "Profile1",
+            "capacity": {
+              "minimum": "1",
+              "maximum": "10",
+              "default": "3"
+            },
+            "rules": [
+              {
+                "metricTrigger": {
+                  "metricName": "/builtin/memory/percentUsedMemory",
+                  "metricNamespace": "",
+                  "metricResourceUri": "[resourceId('Microsoft.Compute/virtualMachineScaleSets', 'myScaleSet')]",
+                  "timeGrain": "PT1M",
+                  "statistic": "Average",
+                  "timeWindow": "PT5M",
+                  "timeAggregation": "Average",
+                  "operator": "GreaterThan",
+                  "threshold": 60
+                },
+                "scaleAction": {
+                  "direction": "Increase",
+                  "type": "ChangeCount",
+                  "value": "1",
+                  "cooldown": "PT1M"
+                }
+              },
+              {
+                "metricTrigger": {
+                  "metricName": "/builtin/memory/percentUsedMemory",
+                  "metricNamespace": "",
+                  "metricResourceUri": "[resourceId('Microsoft.Compute/virtualMachineScaleSets', 'myScaleSet')]",
+                  "timeGrain": "PT1M",
+                  "statistic": "Average",
+                  "timeWindow": "PT5M",
+                  "timeAggregation": "Average",
+                  "operator": "LessThan",
+                  "threshold": 30
+                },
+                "scaleAction": {
+                  "direction": "Decrease",
+                  "type": "ChangeCount",
+                  "value": "1",
+                  "cooldown": "PT1M"
+                }
+              }
+            ]
+          }
+        ]
+      }
     }
   ]
 }
```





## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [mvss-next-steps-include](../../includes/mvss-next-steps.md)]
