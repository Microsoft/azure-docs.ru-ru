---
title: Создание кластера Service Fabric под управлением Windows в Azure
description: Из этого руководства вы узнаете, как развернуть кластер Service Fabric на платформе Windows в виртуальной сети и группе безопасности сети Azure с помощью PowerShell.
ms.topic: tutorial
ms.date: 07/22/2019
ms.custom: mvc
ms.openlocfilehash: a7390858e55a456ec5fb2f851be1a7443be97082
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86245064"
---
# <a name="tutorial-deploy-a-service-fabric-cluster-running-windows-into-an-azure-virtual-network"></a>Руководство по развертыванию кластера Service Fabric на платформе Windows в виртуальной сети Azure

Это руководство представляет первую часть цикла. Вы узнаете, как развернуть в Azure кластер Service Fabric на платформе Windows в [виртуальной сети Azure](../virtual-network/virtual-networks-overview.md) и [группе безопасности сети](../virtual-network/virtual-network-vnet-plan-design-arm.md) с помощью шаблона и PowerShell. Выполнив инструкции из этого учебника, вы получите кластер в облаке, в котором можно разворачивать приложения. Создание кластера Linux с помощью Azure CLI описывается в статье о [создании защищенного кластера Linux в Azure](service-fabric-tutorial-create-vnet-and-linux-cluster.md).

В этом руководстве описывается рабочий сценарий. Если вы хотите создать небольшой кластер для тестирования, ознакомьтесь с [этой статьей](./scripts/service-fabric-powershell-create-secure-cluster-cert.md).

В этом руководстве описано следующее:

> [!div class="checklist"]
> * создание виртуальной сети в Azure с помощью PowerShell;
> * создание хранилища ключей и передача сертификата;
> * настройка проверки подлинности Azure Active Directory;
> * настройка коллекции диагностики;
> * настройка службы EventStore;
> * настройка журналов Azure Monitor;
> * создание защищенного кластера Service Fabric в Azure с помощью PowerShell;
> * Защита кластера с помощью сертификата X.509
> * Подключение к кластеру с помощью PowerShell
> * Удаление кластера

Из этого цикла руководств вы узнаете, как выполнять следующие задачи:
> [!div class="checklist"]
> * создание защищенного кластера в Azure;
> * [Мониторинг кластера](service-fabric-tutorial-monitor-cluster.md)
> * [свертывание и развертывание кластера](service-fabric-tutorial-scale-cluster.md);
> * [Обновление среды выполнения кластера](service-fabric-tutorial-upgrade-cluster.md)
> * [Удаление кластера](service-fabric-tutorial-delete-cluster.md)


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные требования

Перед началом работы с этим руководством выполните следующие действия:

* Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Установите [пакет SDK для Service Fabric и модуль PowerShell](service-fabric-get-started.md).
* Установите [Azure PowerShell](/powershell/azure/install-az-ps).
* Изучите основные понятия для [кластеров Azure](service-fabric-azure-clusters-overview.md).
* [Спланируйте и подготовьте](service-fabric-cluster-azure-deployment-preparation.md) развертывание рабочего кластера.

Ниже приведены процедуры для создания кластера Service Fabric с семью узлами. Чтобы рассчитать затраты, связанные с выполнением кластера Service Fabric в Azure, используйте [калькулятор цен Azure](https://azure.microsoft.com/pricing/calculator/).

## <a name="download-and-explore-the-template"></a>Скачивание и изучение шаблона

Скачайте следующие файлы шаблона Azure Resource Manager:

* [azuredeploy.json][template]
* [azuredeploy.parameters.json][parameters]

Этот шаблон развертывает в виртуальной сети и группе безопасности сети защищенный кластер с семью виртуальными машинами и тремя типами узлов.  Другие примеры шаблонов можно найти на сайте [GitHub](https://github.com/Azure-Samples/service-fabric-cluster-templates). Файл [azuredeploy.json][template] развертывает ряд ресурсов, в том числе следующие.

### <a name="service-fabric-cluster"></a>Кластер Service Fabric

В ресурсе **Microsoft.ServiceFabric/clusters** кластер Windows настроен со следующими характеристиками:

* три типа узлов;
* пять узлов в первичном типе узла (это можно изменить в параметрах шаблона), и по одном узлу для двух других типов узла;
* ОС: Windows Server 2016 Datacenter с контейнерами (можно настроить в параметрах шаблона);
* с защитой сертификатом (можно настроить в параметрах шаблона);
* включен [обратный прокси-сервер](service-fabric-reverseproxy.md);
* включена [служба DNS](service-fabric-dnsservice.md);
* [уровень устойчивости](service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster) Bronze (можно настроить в параметрах шаблона);
* [уровень надежности](service-fabric-cluster-capacity.md#reliability-characteristics-of-the-cluster) Silver (можно настроить в параметрах шаблона);
* конечная точка подключения клиента: 19000 (можно настроить в параметрах шаблона);
* конечная точка HTTP-шлюза: 19080 (можно настроить в параметрах шаблона);

### <a name="azure-load-balancer"></a>Azure Load Balancer

В ресурсе **Microsoft.Network/loadBalancers** настроена подсистема балансировки нагрузки; Настроены пробы и правила для следующих портов:

* конечная точка подключения клиента: 19000;
* конечная точка HTTP-шлюза: 19080;
* порт приложения: 80
* порт приложения: 443
* обратный прокси-сервер Service Fabric: 19081.

Если вам нужны другие порты приложений, следует изменить ресурсы **Microsoft.Network/loadBalancers** и **Microsoft.Network/networkSecurityGroups**, чтобы разрешить входящий трафик.

### <a name="virtual-network-subnet-and-network-security-group"></a>Виртуальная сеть, подсеть и группа безопасности сети

Названия виртуальной сети, подсети и группы безопасности сети указаны в параметрах шаблона. Адресные пространства виртуальной сети и подсети также объявляются в параметрах шаблона и настраиваются в ресурсе **Microsoft.Network/virtualNetworks**:

* Диапазон адресов виртуальной сети: 172.16.0.0/20.
* Диапазон адресов подсети Service Fabric: 172.16.2.0/23.

Следующие правила входящего трафика включаются в ресурсе **Microsoft.Network/networkSecurityGroups** Значения порта можно изменить, изменив переменные шаблона.

* ClientConnectionEndpoint (протокол TCP): 19000;
* HttpGatewayEndpoint (HTTP/TCP): 19080;
* SMB: 445
* Межузловая передача сообщений: 1025, 1026, 1027;
* Диапазон временных портов: от 49152 до 65534 (требуется минимум 256 портов);
* Порты для использования в приложениях: 80 и 443
* Диапазон портов назначения: от 49152 до 65534 (используются для связи между службами, другие порты в подсистеме балансировки нагрузки не открываются).
* Блокировка остальных портов.

Если вам нужны другие порты приложений, следует изменить ресурсы **Microsoft.Network/loadBalancers** и **Microsoft.Network/networkSecurityGroups**, чтобы разрешить входящий трафик.

### <a name="windows-defender"></a>Защитник Windows
по умолчанию на Windows Server 2016 установлена и активирована [Антивирусная программа "Защитник Windows"](/windows/security/threat-protection/windows-defender-antivirus/windows-defender-antivirus-on-windows-server-2016); пользовательский интерфейс устанавливается по умолчанию на некоторых номерах SKU, но не является обязательным. Для каждого типа узла или масштабируемого набора виртуальной машины, объявленного в шаблоне, используется [расширение антивредоносного ПО для виртуальных машин Azure](../virtual-machines/extensions/iaas-antimalware-windows.md) для исключения каталогов и процессов Service Fabric:

```json
{
"name": "[concat('VMIaaSAntimalware','_vmNodeType0Name')]",
"properties": {
        "publisher": "Microsoft.Azure.Security",
        "type": "IaaSAntimalware",
        "typeHandlerVersion": "1.5",
        "settings": {
        "AntimalwareEnabled": "true",
        "Exclusions": {
                "Paths": "D:\\SvcFab;D:\\SvcFab\\Log;C:\\Program Files\\Microsoft Service Fabric",
                "Processes": "Fabric.exe;FabricHost.exe;FabricInstallerService.exe;FabricSetup.exe;FabricDeployer.exe;ImageBuilder.exe;FabricGateway.exe;FabricDCA.exe;FabricFAS.exe;FabricUOS.exe;FabricRM.exe;FileStoreService.exe"
        },
        "RealtimeProtectionEnabled": "true",
        "ScheduledScanSettings": {
                "isEnabled": "true",
                "scanType": "Quick",
                "day": "7",
                "time": "120"
        }
        },
        "protectedSettings": null
}
}
```

## <a name="set-template-parameters"></a>Установка параметров шаблона

В файле параметров [azuredeploy.parameters.json][parameters] объявляются многие значения, используемые для развертывания кластера и связанных ресурсов. Ниже приведены параметры, которые нужно изменить для конкретного развертывания.

**Параметр** | **Пример значения** | **Примечания** 
|---|---|---|
|adminUserName|vmadmin| Имя администратора кластера виртуальных машин. [Требования к имени пользователя для виртуальных машин](../virtual-machines/windows/faq.md#what-are-the-username-requirements-when-creating-a-vm). |
|adminPassword|Password#1234| Пароль администратора для кластера виртуальных машин. [Требования к паролю для виртуальных машин](../virtual-machines/windows/faq.md#what-are-the-password-requirements-when-creating-a-vm).|
|clusterName|mysfcluster123| Имя кластера. Может содержать только буквы и цифры. Длина имени должна составлять от 3 до 23 знаков.|
|location|southcentralus| Расположение кластера. |
|certificateThumbprint|| <p>Если создается самозаверяющий сертификат или указывается файл сертификата, значение должно быть пустым.</p><p>Если необходимо использовать имеющийся сертификат, который вы ранее передали в хранилище ключей, заполните значение отпечатка SHA1 сертификата. Например, 6190390162C988701DB5676EB81083EA608DCCF3.</p> |
|certificateUrlValue|| <p>Если создается самозаверяющий сертификат или указывается файл сертификата, значение должно быть пустым. </p><p>Если необходимо использовать имеющийся сертификат, который вы ранее передали в хранилище ключей, укажите URL-адрес сертификата. Например https:\//mykeyvault.vault.azure.net:443/secrets/mycertificate/02bea722c9ef4009a76c5052bcbf8346.</p>|
|sourceVaultValue||<p>Если создается самозаверяющий сертификат или указывается файл сертификата, значение должно быть пустым.</p><p>Если необходимо использовать имеющийся сертификат, который вы ранее передали в хранилище ключей, укажите сведения об исходном хранилище. Например, /subscriptions/333cc2c84-12fa-5778-bd71-c71c07bf873f/resourceGroups/MyTestRG/providers/Microsoft.KeyVault/vaults/MYKEYVAULT.</p>|

## <a name="set-up-azure-active-directory-client-authentication"></a>Настройка проверки подлинности клиента Azure Active Directory
Для кластеров Service Fabric, развернутых в общедоступной сети, размещенной в Azure, для взаимной проверки подлинности между клиентом и узлом рекомендуется:
* использование Azure Active Directory для удостоверения клиента;
* использование сертификата для удостоверения сервера и TLS-шифрования HTTP-подключений.

Настройку Azure Active Directory (AAD) для проверки подлинности клиентов для кластера Service Fabric нужно сделать перед [созданием кластера](#createvaultandcert). Azure AD позволяет организациям (известным как клиенты) управлять доступом пользователей к приложениям. 

Кластеры Service Fabric предлагают несколько точек входа для управления функциями кластеров, включая веб-интерфейс [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) и [Visual Studio](service-fabric-manage-application-in-visual-studio.md). В итоге вы создаете два приложения Azure AD для управления доступом к кластеру: одно веб-приложение и одно собственное приложение.  Создав приложения, сопоставьте пользователей с ролями, разрешающими доступ только для чтения и администрирования.

> [!NOTE]
> Перед созданием кластера необходимо выполнить следующие действия. Поскольку в сценариях предварительно заданы имена кластеров и конечные точки, эти имена нужно выбрать заблаговременно, при этом они должны отличаться от созданных ранее имен.

В этой статье предполагается, что клиент уже создан. Если это не так, обратитесь к статье [Краткое руководство. Настройка среды разработки](../active-directory/develop/quickstart-create-new-tenant.md).

Чтобы упростить некоторые шаги по настройке Azure AD с кластером Service Fabric, мы создали набор скриптов для Windows PowerShell. [Скачайте сценарии](https://github.com/Azure-Samples/service-fabric-aad-helpers) на компьютер.

### <a name="create-azure-ad-applications-and-assign-users-to-roles"></a>Создание приложения Azure AD и назначение ролей пользователям
Создайте два приложения Azure AD для управления доступом к кластеру: одно веб-приложение и одно собственное приложение. Создав приложения, представляющие кластер, назначьте пользователям [роли, поддерживаемые Service Fabric](service-fabric-cluster-security-roles.md): с правами только на чтение и администратора.

Запустите `SetupApplications.ps1` и укажите идентификатор клиента, имя кластера и URL-адрес ответа веб-приложения в качестве параметров. Укажите имена пользователей и пароли для пользователей. Пример:

```powershell
$Configobj = .\SetupApplications.ps1 -TenantId '<MyTenantID>' -ClusterName 'mysfcluster123' -WebApplicationReplyUrl 'https://mysfcluster123.eastus.cloudapp.azure.com:19080/Explorer/index.html' -AddResourceAccess
.\SetupUser.ps1 -ConfigObj $Configobj -UserName 'TestUser' -Password 'P@ssword!123'
.\SetupUser.ps1 -ConfigObj $Configobj -UserName 'TestAdmin' -Password 'P@ssword!123' -IsAdmin
```

> [!NOTE]
> Для национальных облаков (например, Azure для государственных организаций, Azure для Китая и Azure для Германии) укажите параметр `-Location`.

*TenantId* или идентификатор каталога вы можете найти на [портале Azure](https://portal.azure.com). Выберите **Azure Active Directory** > **Свойства** и скопируйте значение **идентификатора каталога**.

Значение *ClusterName* используется в качестве префикса для приложений Azure AD, создаваемых скриптом. Оно не должно совпадать с именем реального кластера. Оно нужно только для удобства сопоставления артефактов AAD с кластером Service Fabric.

*WebApplicationReplyUrl* является конечной точкой по умолчанию, которую Azure AD возвращает пользователям после завершения ими входа. Эту конечную точку следует назначить конечной точкой Service Fabric Explorer для кластера, по умолчанию это:

https://&lt;cluster_domain&gt;:19080/Explorer

Вам будет предложено войти в учетную запись с правами администратора для клиента Azure AD. После входа сценарий начнет создавать веб-приложение и собственное приложение для представления кластера Service Fabric. В списке приложений клиента на [портале Azure](https://portal.azure.com) вы увидите две новые записи:

   * *имя_кластера*\_Кластер
   * *имя_кластера*\_Клиент

Этот скрипт выводит код JSON, который потребуется для шаблона Azure Resource Manager при создании кластера. Мы рекомендуем пока не закрывать окно PowerShell.

```json
"azureActiveDirectory": {
  "tenantId":"<guid>",
  "clusterApplication":"<guid>",
  "clientApplication":"<guid>"
},
```

### <a name="add-azure-ad-configuration-to-use-azure-ad-for-client-access"></a>Добавление конфигурации Azure AD для клиентского доступа через Azure AD
В [azuredeploy.json][template] настройте Azure AD в разделе **Microsoft.ServiceFabric/clusters**. Добавьте параметры для идентификатора клиента, идентификатора приложения кластера и идентификатора клиентского приложения.  

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    ...

    "aadTenantId": {
      "type": "string",
      "defaultValue": "0e3d2646-78b3-4711-b8be-74a381d9890c"
    },
    "aadClusterApplicationId": {
      "type": "string",
      "defaultValue": "cb147d34-b0b9-4e77-81d6-420fef0c4180"
    },
    "aadClientApplicationId": {
      "type": "string",
      "defaultValue": "7a8f3b37-cc40-45cc-9b8f-57b8919ea461"
    }
  },

...

{
  "apiVersion": "2018-02-01",
  "type": "Microsoft.ServiceFabric/clusters",
  "name": "[parameters('clusterName')]",
  ...
  "properties": {
    ...
    "azureActiveDirectory": {
      "tenantId": "[parameters('aadTenantId')]",
      "clusterApplication": "[parameters('aadClusterApplicationId')]",
      "clientApplication": "[parameters('aadClientApplicationId')]"
    },
    ...
  }
}
```

Добавьте значения параметров в файл параметров [azuredeploy.parameters.json][parameters]. Пример:

```json
"aadTenantId": {
"value": "0e3d2646-78b3-4711-b8be-74a381d9890c"
},
"aadClusterApplicationId": {
"value": "cb147d34-b0b9-4e77-81d6-420fef0c4180"
},
"aadClientApplicationId": {
"value": "7a8f3b37-cc40-45cc-9b8f-57b8919ea461"
}
```
<a id="configurediagnostics" name="configurediagnostics_anchor"></a>

## <a name="configure-diagnostics-collection-on-the-cluster"></a>Настройка сбора данных диагностики для кластера
При использовании кластера Service Fabric очень полезно централизованно собирать журналы со всех узлов. Централизованное хранение журналов упрощает анализ и устранение неполадок в кластере, а также в приложениях и службах, работающих в этом кластере.

Один из способов отправки и сбора журналов заключается в использовании расширения "Диагностика Azure", которое отправляет журналы в службу хранилища Azure или (опционально) в Azure Application Insights или Центры событий Azure. Вы также можете использовать внешний процесс, чтобы считывать события из хранилища и передавать их в платформу обработки, например в журналы Azure Monitor или другое решение для анализа журналов.

Если вы строго придерживаетесь инструкций в этом руководстве, сбор диагностических сведений уже настроен в [шаблоне][template].

Если у вас есть кластер, в котором еще не развернута система диагностики, вы можете добавить или обновить систему диагностики с помощью шаблона кластера. Измените шаблон Resource Manager, который используется для создания существующего кластера, или скачайте шаблон на портале. Измените файл template.json, выполнив следующие действия.

Добавьте новый ресурс хранилища в раздел resources шаблона.
```json
"resources": [
...
{
  "apiVersion": "2015-05-01-preview",
  "type": "Microsoft.Storage/storageAccounts",
  "name": "[parameters('applicationDiagnosticsStorageAccountName')]",
  "location": "[parameters('computeLocation')]",
  "sku": {
    "accountType": "[parameters('applicationDiagnosticsStorageAccountType')]"
  },
  "tags": {
    "resourceType": "Service Fabric",
    "clusterName": "[parameters('clusterName')]"
  }
},
...
]
```

Теперь добавьте параметры для имени и типа учетной записи хранения в раздел parameters шаблона. Замените текст заполнителя "storage account name goes here" именем учетной записи хранения, которую вы намерены использовать.

```json
"parameters": {
...
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
...
}
```

Затем добавьте расширение **IaaSDiagnostics** в массив extensions для свойства **VirtualMachineProfile** в каждом ресурсе **Microsoft.Compute/virtualMachineScaleSets** этого кластера.  Если вы используете наш [пример шаблона][template], в нем описаны три масштабируемых набора виртуальных машин (по одному для каждого типа узла в кластере).

```json
"apiVersion": "2018-10-01",
"type": "Microsoft.Compute/virtualMachineScaleSets",
"name": "[variables('vmNodeType1Name')]",
"properties": {
    ...
    "virtualMachineProfile": {
        "extensionProfile": {
            "extensions": [
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
            ...
            ]
        }
    }
}
```
<a id="configureeventstore" name="configureeventstore_anchor"></a>

## <a name="configure-the-eventstore-service"></a>Настройка службы EventStore
Служба EventStore является одним из средств мониторинга для Service Fabric. EventStore предоставляет способ для оценки состояния кластера или рабочих нагрузок в определенный момент времени. EventStore — это служба Service Fabric с отслеживанием состояния, сохраняющая события из кластера. Событие предоставляется через Service Fabric Explorer, REST и API. EventStore отправляет запросы непосредственно к кластеру, чтобы получить данные диагностики по любой сущности в кластере. Их следует использовать, чтобы:

* Диагностировать проблемы при разработке, тестировании или во время использования конвейера для мониторинга.
* Проверить, правильно ли обрабатываются действия управления, предпринимаемые в кластере.
* Получить моментальный снимок взаимодействия Service Fabric с конкретной сущностью.



Чтобы включить службу EventStore в кластере, добавьте следующий код в свойство **fabricSettings** для ресурса **Microsoft.ServiceFabric/clusters**:

```json
"apiVersion": "2018-02-01",
"type": "Microsoft.ServiceFabric/clusters",
"name": "[parameters('clusterName')]",
"properties": {
    ...
    "fabricSettings": [
        ...
        {
            "name": "EventStoreService",
            "parameters": [
                {
                "name": "TargetReplicaSetSize",
                "value": "3"
                },
                {
                "name": "MinReplicaSetSize",
                "value": "1"
                }
            ]
        }
    ]
}
```
<a id="configureloganalytics" name="configureloganalytics_anchor"></a>

## <a name="set-up-azure-monitor-logs-for-the-cluster"></a>Настройка журналов Azure Monitor для кластера

Для мониторинга событий на уровне кластера мы рекомендуем использовать журналы Azure Monitor. Чтобы настроить журналы Azure Monitor для мониторинга кластера, следует [включить систему диагностики для просмотра событий уровня кластера](#configure-diagnostics-collection-on-the-cluster).  

Рабочая область должна быть подключена к данным диагностики, поступающим из кластера.  Эти данные журналов хранятся в учетной записи хранения *applicationDiagnosticsStorageAccountName* в таблицах WADServiceFabric*EventTable, WADWindowsEventLogsTable и WADETWEventTable.

Добавьте рабочую область Azure Log Analytics и добавьте решение в эту рабочую область:

```json
"resources": [
    ...
    {
        "apiVersion": "2015-11-01-preview",
        "location": "[parameters('omsRegion')]",
        "name": "[parameters('omsWorkspacename')]",
        "type": "Microsoft.OperationalInsights/workspaces",
        "properties": {
            "sku": {
                "name": "Free"
            }
        },
        "resources": [
            {
                "apiVersion": "2015-11-01-preview",
                "name": "[concat(variables('applicationDiagnosticsStorageAccountName'),parameters('omsWorkspacename'))]",
                "type": "storageinsightconfigs",
                "dependsOn": [
                    "[concat('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename'))]",
                    "[concat('Microsoft.Storage/storageAccounts/', variables('applicationDiagnosticsStorageAccountName'))]"
                ],
                "properties": {
                    "containers": [],
                    "tables": [
                        "WADServiceFabric*EventTable",
                        "WADWindowsEventLogsTable",
                        "WADETWEventTable"
                    ],
                    "storageAccount": {
                        "id": "[resourceId('Microsoft.Storage/storageaccounts/', variables('applicationDiagnosticsStorageAccountName'))]",
                        "key": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('applicationDiagnosticsStorageAccountName')),'2015-06-15').key1]"
                    }
                }
            },
            {
                "apiVersion": "2015-11-01-preview",
                "type": "datasources",
                "name": "sampleWindowsPerfCounter",
                "dependsOn": [
                    "[concat('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename'))]"
                ],
                "kind": "WindowsPerformanceCounter",
                "properties": {
                    "objectName": "Memory",
                    "instanceName": "*",
                    "intervalSeconds": 10,
                    "counterName": "Available MBytes"
                }
            },
            {
                "apiVersion": "2015-11-01-preview",
                "type": "datasources",
                "name": "sampleWindowsPerfCounter2",
                "dependsOn": [
                    "[concat('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename'))]"
                ],
                "kind": "WindowsPerformanceCounter",
                "properties": {
                    "objectName": "Service Fabric Service",
                    "instanceName": "*",
                    "intervalSeconds": 10,
                    "counterName": "Average milliseconds per request"
                }
            }
        ]
    },
    {
        "apiVersion": "2015-11-01-preview",
        "location": "[parameters('omsRegion')]",
        "name": "[variables('solution')]",
        "type": "Microsoft.OperationsManagement/solutions",
        "dependsOn": [
            "[concat('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename'))]"
        ],
        "properties": {
            "workspaceResourceId": "[resourceId('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename'))]"
        },
        "plan": {
            "name": "[variables('solution')]",
            "publisher": "Microsoft",
            "product": "[Concat('OMSGallery/', variables('solutionName'))]",
            "promotionCode": ""
        }
    }
]
```

Теперь добавьте параметры
```json
"parameters": {
    ...
    "omsWorkspacename": {
        "type": "string",
        "defaultValue": "mysfomsworkspace",
        "metadata": {
            "description": "Name of your OMS Log Analytics Workspace"
        }
    },
    "omsRegion": {
        "type": "string",
        "defaultValue": "West Europe",
        "allowedValues": [
            "West Europe",
            "East US",
            "Southeast Asia"
        ],
        "metadata": {
            "description": "Specify the Azure Region for your OMS workspace"
        }
    }
}
```

Теперь добавьте переменные
```json
"variables": {
    ...
    "solution": "[Concat('ServiceFabric', '(', parameters('omsWorkspacename'), ')')]",
    "solutionName": "ServiceFabric"
}
```

Добавьте расширение агента Log Analytics в каждый масштабируемый набор виртуальных машин в кластере и подключите агент к рабочей области Log Analytics. Это активирует сбор данных диагностики о контейнерах и приложениях, а также мониторинг производительности. Если вы добавите агент в качестве расширения ресурса масштабируемого набора виртуальных машин, Azure Resource Manager гарантирует его установку на каждом узле, даже при масштабировании кластера.

```json
"apiVersion": "2018-10-01",
"type": "Microsoft.Compute/virtualMachineScaleSets",
"name": "[variables('vmNodeType1Name')]",
"properties": {
    ...
    "virtualMachineProfile": {
        "extensionProfile": {
            "extensions": [
                {
                    "name": "[concat(variables('vmNodeType0Name'),'OMS')]",
                    "properties": {
                        "publisher": "Microsoft.EnterpriseCloud.Monitoring",
                        "type": "MicrosoftMonitoringAgent",
                        "typeHandlerVersion": "1.0",
                        "autoUpgradeMinorVersion": true,
                        "settings": {
                            "workspaceId": "[reference(resourceId('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename')), '2015-11-01-preview').customerId]"
                        },
                        "protectedSettings": {
                            "workspaceKey": "[listKeys(resourceId('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename')),'2015-11-01-preview').primarySharedKey]"
                        }
                    }
                }
            ...
            ]
        }
    }
}
```

<a id="createvaultandcert" name="createvaultandcert_anchor"></a>

## <a name="deploy-the-virtual-network-and-cluster"></a>Развертывание кластера и виртуальной сети

Настройте топологию сети и разверните кластер Service Fabric. Шаблон Resource Manager [azuredeploy.json][template] создает виртуальную сеть, а также подсеть и группу безопасности сети для Service Fabric. Шаблон также развертывает кластер с включенным сертификатом безопасности. Для рабочих кластеров в качестве сертификата нужно использовать сертификат из центра сертификации. Самозаверяющий сертификат можно использовать для защиты тестовых кластеров.

Шаблон в этой статье развертывает кластер, использующий отпечаток сертификата для идентификации сертификата кластера. Два сертификата не могут иметь один и тот же отпечаток. Это затрудняет управление сертификатами. Переключение развернутого кластера с отпечатков сертификата на общие имена сертификатов упрощает управление сертификатами. Сведения о том, как настроить для управления сертификатами кластера метод общих имен сертификатов, см. в [этой статье](service-fabric-cluster-change-cert-thumbprint-to-cn.md).

### <a name="create-a-cluster-by-using-an-existing-certificate"></a>Создание кластера с применением существующего сертификата

В приведенном ниже сценарии используется командлет [New-AzServiceFabricCluster](/powershell/module/az.servicefabric/New-azServiceFabricCluster) и шаблон, которые позволяют развернуть в Azure новый кластер. Этот командлет создает хранилище ключей в Azure и отправляет в него существующий сертификат.

```powershell
# Variables.
$groupname = "sfclustertutorialgroup"
$clusterloc="southcentralus"  # Must match the location parameter in the template
$templatepath="C:\temp\cluster"

$certpwd="q6D7nN%6ck@6" | ConvertTo-SecureString -AsPlainText -Force
$clustername = "mysfcluster123"  # Must match the clustername parameter in the template
$vaultname = "clusterkeyvault123"
$vaultgroupname="clusterkeyvaultgroup123"
$subname="$clustername.$clusterloc.cloudapp.azure.com"

# Sign in to your Azure account and select your subscription
Connect-AzAccount
Get-AzSubscription
Set-AzContext -SubscriptionId <guid>

# Create a new resource group for your deployment, and give it a name and a location.
New-AzResourceGroup -Name $groupname -Location $clusterloc

# Create the Service Fabric cluster.
New-AzServiceFabricCluster  -ResourceGroupName $groupname -TemplateFile "$templatepath\azuredeploy.json" `
-ParameterFile "$templatepath\azuredeploy.parameters.json" -CertificatePassword $certpwd `
-KeyVaultName $vaultname -KeyVaultResourceGroupName $vaultgroupname -CertificateFile $certpath
```

### <a name="create-a-cluster-by-using-a-new-self-signed-certificate"></a>Создание кластера с помощью нового самозаверяющего сертификата

В приведенном ниже сценарии используется командлет [New-AzServiceFabricCluster](/powershell/module/az.servicefabric/New-azServiceFabricCluster) и шаблон, которые позволяют развернуть в Azure новый кластер. Этот командлет создает хранилище ключей в Azure, добавляет в него самозаверяющий сертификат и скачивает файл сертификата на локальный компьютер.

```powershell
# Variables.
$groupname = "sfclustertutorialgroup"
$clusterloc="southcentralus"  # Must match the location parameter in the template
$templatepath="C:\temp\cluster"

$certpwd="q6D7nN%6ck@6" | ConvertTo-SecureString -AsPlainText -Force
$certfolder="c:\mycertificates\"
$clustername = "mysfcluster123"
$vaultname = "clusterkeyvault123"
$vaultgroupname="clusterkeyvaultgroup123"
$subname="$clustername.$clusterloc.cloudapp.azure.com"

# Sign in to your Azure account and select your subscription
Connect-AzAccount
Get-AzSubscription
Set-AzContext -SubscriptionId <guid>

# Create a new resource group for your deployment, and give it a name and a location.
New-AzResourceGroup -Name $groupname -Location $clusterloc

# Create the Service Fabric cluster.
New-AzServiceFabricCluster  -ResourceGroupName $groupname -TemplateFile "$templatepath\azuredeploy.json" `
-ParameterFile "$templatepath\azuredeploy.parameters.json" -CertificatePassword $certpwd `
-CertificateOutputFolder $certfolder -KeyVaultName $vaultname -KeyVaultResourceGroupName $vaultgroupname -CertificateSubjectName $subname

```

## <a name="connect-to-the-secure-cluster"></a>Подключение к безопасному кластеру

Подключитесь к кластеру с помощью модуля Service Fabric PowerShell, который устанавливается вместе с пакетом SDK для Service Fabric.  Сначала установите сертификат в личное хранилище (Мое хранилище) текущего пользователя компьютера. Выполните следующую команду PowerShell:

```powershell
$certpwd="q6D7nN%6ck@6" | ConvertTo-SecureString -AsPlainText -Force
Import-PfxCertificate -Exportable -CertStoreLocation Cert:\CurrentUser\My `
        -FilePath C:\mycertificates\mysfcluster20170531104310.pfx `
        -Password $certpwd
```

Теперь все готово для подключения к защищенному кластеру.

Модуль PowerShell **Service Fabric** предоставляет многие командлеты для управления кластерами, приложениями и службами Service Fabric. Для подключения к безопасному кластеру используйте командлет [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster). Сведения об отпечатке SHA1 сертификата и конечной точке подключения можно найти в выходных данных из предыдущего шага.

Если ранее вы настроили проверку подлинности клиента AAD, выполните следующую команду: 
```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint mysfcluster123.southcentralus.cloudapp.azure.com:19000 `
        -KeepAliveIntervalInSec 10 `
        -AzureActiveDirectory `
        -ServerCertThumbprint C4C1E541AD512B8065280292A8BA6079C3F26F10
```

Если вы не используете проверку подлинности клиента AAD, выполните следующую команду:
```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint mysfcluster123.southcentralus.cloudapp.azure.com:19000 `
          -KeepAliveIntervalInSec 10 `
          -X509Credential -ServerCertThumbprint C4C1E541AD512B8065280292A8BA6079C3F26F10 `
          -FindType FindByThumbprint -FindValue C4C1E541AD512B8065280292A8BA6079C3F26F10 `
          -StoreLocation CurrentUser -StoreName My
```

Убедитесь, что подключение успешно установлено и кластер находится в работоспособном состоянии, выполнив командлет [Get-ServiceFabricClusterHealth](/powershell/module/servicefabric/get-servicefabricclusterhealth).

```powershell
Get-ServiceFabricClusterHealth
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Кластер, который вы только что создали, используется в других статьях этого цикла учебников. Если вы не собираетесь немедленно приступить к следующей статье, то можете [удалить кластер](./service-fabric-tutorial-delete-cluster.md), чтобы за него не взималась плата.

## <a name="next-steps"></a>Дальнейшие действия

Перейдите к следующему учебнику, который посвящен масштабированию кластера.

> [!div class="checklist"]
> * создание виртуальной сети в Azure с помощью PowerShell;
> * создание хранилища ключей и передача сертификата;
> * настройка проверки подлинности Azure Active Directory;
> * настройка коллекции диагностики;
> * настройка службы EventStore;
> * настройка журналов Azure Monitor;
> * создание защищенного кластера Service Fabric в Azure с помощью PowerShell;
> * Защита кластера с помощью сертификата X.509
> * Подключение к кластеру с помощью PowerShell
> * Удаление кластера

Теперь перейдите к следующему учебнику, который посвящен мониторингу кластера.
> [!div class="nextstepaction"]
> [Мониторинг кластера](service-fabric-tutorial-monitor-cluster.md)

[template]:https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/7-VM-Windows-3-NodeTypes-Secure-NSG/AzureDeploy.json
[parameters]:https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/7-VM-Windows-3-NodeTypes-Secure-NSG/AzureDeploy.Parameters.json
