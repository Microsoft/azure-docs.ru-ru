---
title: Использование расширений с пулами пакетной службы
description: Расширения — это небольшие приложения, которые упрощают настройку после подготовки и настройки на вычислительных узлах пакетной службы.
ms.topic: how-to
ms.date: 02/10/2021
ms.openlocfilehash: 1bf9847af57347c143ee3d790d89988ba7cd48e4
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100417945"
---
# <a name="use-extensions-with-batch-pools"></a>Использование расширений с пулами пакетной службы

Расширения — это небольшие приложения, которые упрощают настройку после подготовки и настройки на вычислительных узлах пакетной службы. Вы можете выбрать любое из расширений, разрешенных пакетной службой Azure, и установить их на расчетные узлы по мере их подготовки. После этого расширение может выполнить требуемую операцию.

Вы можете проверить динамическое состояние используемых расширений и получить сведения, которые они возвращают, чтобы использовать любые возможности обнаружения, исправления или диагностики.

## <a name="prerequisites"></a>Предварительные условия

- Пулы с расширениями должны использовать [конфигурацию виртуальной машины](nodes-and-pools.md#virtual-machine-configuration).
- Тип расширения CustomScript зарезервирован для пакетной службы Azure и не может быть переопределен.

### <a name="supported-extensions"></a>Поддерживаемые расширения

При создании пула пакетной службы в настоящее время можно установить следующие расширения. 

- Расширение Azure Key Vault для [Linux](../virtual-machines/extensions/key-vault-linux.md) и [Windows](../virtual-machines/extensions/key-vault-windows.md)
- Расширение log Analytics и мониторинг для [Linux](../virtual-machines/extensions/oms-linux.md) и [Windows](../virtual-machines/extensions/oms-windows.md)
- Пакет безопасности Azure

Вы можете запросить поддержку дополнительных издателей или типов расширений, открыв запрос в службу поддержки.

## <a name="create-a-pool-with-extensions"></a>Создание пула с расширениями

В приведенном ниже примере создается пул пакетной службы для узлов Linux, использующий расширение Azure Key Vault.

Универсальный код ресурса (URI) REST API

```http
 PUT https://management.azure.com/subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/Microsoft.Batch/batchAccounts/<batchaccountName>/pools/<batchpoolName>?api-version=2021-01-01
```

Текст запроса

```json
{
    "name": "test1",
    "type": "Microsoft.Batch/batchAccounts/pools",
    "properties": {
        "vmSize": "STANDARD_DS2_V2",
        "taskSchedulingPolicy": {
            "nodeFillType": "Pack"
        },
        "deploymentConfiguration": {
            "virtualMachineConfiguration": {
                "imageReference": {
                    "publisher": "canonical",
                    "offer": "ubuntuserver",
                    "sku": "18.04-lts",
                    "version": "latest"
                },
                "nodeAgentSkuId": "batch.node.ubuntu 18.04",
                "extensions": [
                    {
                        "name": "secretext",
                        "type": "KeyVaultForLinux",
                        "publisher": "Microsoft.Azure.KeyVault",
                        "typeHandlerVersion": "1.0",
                        "autoUpgradeMinorVersion": true,
                        "settings": {
                            "secretsManagementSettings": {
                                "pollingIntervalInS": "300",
                                "certificateStoreLocation": "/var/lib/waagent/Microsoft.Azure.KeyVault",
                                "requireInitialSync": true,
                                "observedCertificates": [
                                    "https://testkvwestus2.vault.azure.net/secrets/authsecreat"
                                ]
                            },
                            "authenticationSettings": {
                                "msiEndpoint": "http://169.254.169.254/metadata/identity",
                                "msiClientId": "885b1a3d-f13c-4030-afcf-9f05044d78dc"
                            }
                        },
                        "protectedSettings":{}
                    }
                ]
            }
        },
        "scaleSettings": {
            "fixedScale": {
                "targetDedicatedNodes": 1,
                "targetLowPriorityNodes": 0,
                "resizeTimeout": "PT15M"
            }
        }
```

## <a name="get-extension-data-from-a-pool"></a>Получение данных расширения из пула

В приведенном ниже примере данные извлекаются из расширения Azure Key Vault.

Универсальный код ресурса (URI) REST API

```http
 GET https://<accountname>.<region>.batch.azure.com/pools/test3/nodes/tvmps_a3ce79db285d6c124399c5bd3f3cf308d652c89675d9f1f14bfc184476525278_d/extensions/secretext?api-version=2010-01-01
```

Текст ответа

```json
{
  "odata.metadata":"https://testwestus2batch.westus2.batch.azure.com/$metadata#extensions/@Element","instanceView":{
    "name":"secretext","statuses":[
      {
        "code":"ProvisioningState/succeeded","level":0,"displayStatus":"Provisioning succeeded","message":"Successfully started Key Vault extension service. 2021-02-08T19:49:39Z"
      }
    ]
  },"vmExtension":{
    "name":"KVExtensions","publisher":"Microsoft.Azure.KeyVault","type":"KeyVaultForLinux","typeHandlerVersion":"1.0","autoUpgradeMinorVersion":true,"settings":"{\r\n  \"secretsManagementSettings\": {\r\n    \"pollingIntervalInS\": \"300\",\r\n    \"certificateStoreLocation\": \"/var/lib/waagent/Microsoft.Azure.KeyVault\",\r\n    \"requireInitialSync\": true,\r\n    \"observedCertificates\": [\r\n      \"https://testkvwestus2.vault.azure.net/secrets/testumi\"\r\n    ]\r\n  },\r\n  \"authenticationSettings\": {\r\n    \"msiEndpoint\": \"http://169.254.169.254/metadata/identity\",\r\n    \"msiClientId\": \"885b1a3d-f13c-4030-afcf-922f05044d78dc\"\r\n  }\r\n}"
  }
}

```

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте о различных способах [копирования приложений и данных на узлы пула](batch-applications-to-pool-nodes.md).
- Дополнительные сведения о работе с [узлами и пулами](nodes-and-pools.md).
