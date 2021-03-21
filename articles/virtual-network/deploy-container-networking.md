---
title: Развертывание сетевого интерфейса контейнера в виртуальной сети Azure | Документы Майкрософт
description: Узнайте, как развернуть подключаемый модуль сетевого интерфейса контейнера виртуальной сети Azure (CNI) для кластеров Kubernetes.
services: virtual-network
documentationcenter: na
author: aanandr
manager: NarayanAnnamalai
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 9/18/2018
ms.author: aanandr
ms.custom: ''
ms.openlocfilehash: b95b3cfdf8fea6e31015d945566803569b4ba064
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98222927"
---
# <a name="deploy-the-azure-virtual-network-container-network-interface-plug-in"></a>Развертывание подключаемого модуля сетевого интерфейса контейнера виртуальной сети Azure

Подключаемый модуль сетевого интерфейса контейнера (CNI) виртуальной сети Azure устанавливается на виртуальной машине Azure и предоставляет возможности виртуальной сети для модулей pod Kubernetes и контейнеров Docker. Дополнительные сведения о подключаемом модуле см. в разделе [Включение контейнеров для использования возможностей виртуальной сети Azure](container-networking-overview.md). Кроме того, подключаемый модуль можно использовать со службой Azure Kubernetes (AKS), выбрав параметр [Расширенное сетевое подключение](../aks/configure-azure-cni.md?toc=%2fazure%2fvirtual-network%2ftoc.json), который автоматически размещает контейнеры AKS в виртуальной сети.

## <a name="deploy-plug-in-for-acs-engine-kubernetes-cluster"></a>Развертывание подключаемого модуля для кластера Kubernetes с обработчиком ACS

Обработчик ACS развертывает кластер Kubernetes с помощью шаблона Azure Resource Manager. Конфигурация кластера указана в файле JSON, который передается в средство при создании шаблона. Дополнительные сведения о полном списке поддерживаемых параметров кластера и их описания см. в разделе [Microsoft Azure контейнер службы контейнеров — определение кластера](https://github.com/Azure/acs-engine/blob/master/docs/clusterdefinition.md). Подключаемый модуль — это подключаемый модуль сети по умолчанию для кластеров, созданных с использованием обработчика ACS. Следующие параметры конфигурации сети важны при настройке подключаемого модуля:

  | Параметр                              | Описание                                                                                                           |
  |--------------------------------------|------------------------------------------------------------------------------------------------------                 |
  | firstConsecutiveStaticIP             | IP-адрес, выделенный для главного узла. Это обязательный параметр.                                     |
  | clusterSubnet в kubernetesConfig | CIDR подсети виртуальной сети, где развернут кластер, из которого IP-адреса выделяются для pod   |
  | vnetSubnetId в masterProfile     | Указывает идентификатор ресурса Azure Resource Manager для подсети, где будет развертываться кластер                    |
  | vnetCidr                             | CIDR виртуальной сети, где развернут кластер                                                             |
  | max-Pods в kubeletConfig         | Максимальное количество pod на каждой виртуальной машине агента. Для подключаемого модуля значение по умолчанию — 30. Можно указать количество до 250  |

### <a name="example-configuration"></a>Пример конфигурации

Следующий пример json приведен для кластера со следующими свойствами:
-   1 главный узел и 2 узла агентов 
-   Развертывается в подсети с именем *KubeClusterSubnet* (10.0.0.0/20) с главным узлом и узлами агентов.

```json
{
  "apiVersion": "vlabs",
  "properties": {
    "orchestratorProfile": {
      "orchestratorType": "Kubernetes",
      "kubernetesConfig": {
        "clusterSubnet": "10.0.0.0/20" --> Subnet allocated for the cluster
      }
    },
    "masterProfile": {
      "count": 1,
      "dnsPrefix": "ACSKubeMaster",
      "vmSize": "Standard_A2",
      "vnetSubnetId": "/subscriptions/<subscription ID>/resourceGroups/<Resource Group Name>/providers/Microsoft.Network/virtualNetworks/<Vnet Name>/subnets/KubeClusterSubnet",
      "firstConsecutiveStaticIP": "10.0.1.50", --> IP address allocated to the Master node
"vnetCidr": "10.0.0.0/16" --> Virtual network address space
    },
    "agentPoolProfiles": [
      {
        "name": "k8sagentpoo1",
        "count": 2,
        "vmSize": "Standard_A2_v2",
"vnetSubnetId": "/subscriptions/<subscription ID>/resourceGroups/<Resource Group Name>/providers/Microsoft.Network/virtualNetworks/<VNet Name>/subnets/KubeClusterSubnet",
        "availabilityProfile": "AvailabilitySet"
      }
    ],
    "linuxProfile": {
      "adminUsername": "KubeServerAdmin",
      "ssh": {
        "publicKeys": [
          {…}
        ]
      }
    },
    "servicePrincipalProfile": {
      "clientId": "dd438987-aa12-4754-b47d-375811889714",
      "secret": "azure123"
    }
  }
}
```

## <a name="deploy-plug-in-for-a-kubernetes-cluster"></a>Развертывание подключаемого модуля для кластера Kubernetes

Выполните следующие действия, чтобы установить подключаемый модуль на каждой виртуальной машине Azure в кластере Kubernetes:

1. [Скачайте и установите подключаемый модуль](#download-and-install-the-plug-in).
2. Предварительно выделите пул IP-адресов виртуальной сети на каждой виртуальной машине, из которой IP-адреса будут назначаться для модулей pod. Каждая виртуальная машина Azure имеет основной частный IP-адрес виртуальной сети для каждого сетевого интерфейса. Пул IP-адресов для модулей pod добавляется в качестве дополнительных адресов (*ipconfigs*) в сетевом интерфейсе виртуальной машины с помощью одного из следующих вариантов:

   - **CLI**. [Назначение нескольких IP-адресов с помощью Azure CLI](virtual-network-multiple-ip-addresses-cli.md)
   - **PowerShell**. [Назначение нескольких IP-адресов с помощью PowerShell](virtual-network-multiple-ip-addresses-powershell.md)
   - **Портал**. [Назначение нескольких IP-адресов с помощью портала Azure](virtual-network-multiple-ip-addresses-portal.md)
   - **Шаблон Resource Manager**. [Назначение нескольких IP-адресов с помощью шаблонов](./template-samples.md)

   Убедитесь, что добавили достаточно IP-адресов для всех модулей pod, которые планируется использовать на виртуальной машине.

3. Выберите подключаемый модуль для предоставления сетей для кластера, передав Kubelet параметр командной строки `–network-plugin=cni` во время создания кластера. Kubernetes по умолчанию ищет подключаемый модуль и файл конфигурации в каталогах, где они уже установлены.
4. Если вы хотите, чтобы у модулей pod был доступ к Интернету, добавьте следующее правило *iptables* на виртуальных машинах Linux для предоставления Интернет-трафика через source NAT. В следующем примере указанный диапазон IP-адресов — 10.0.0.0/8.

   ```bash
   iptables -t nat -A POSTROUTING -m iprange ! --dst-range 168.63.129.16 -m
   addrtype ! --dst-type local ! -d 10.0.0.0/8 -j MASQUERADE
   ```

   Правила преобразуют сетевые адреса для трафика, не предназначенного для указанных диапазонов IP-адресов. Предполагается, что весь трафик за пределами предыдущих диапазонов является интернет-трафиком. Вы можете указать диапазон IP-адресов виртуальной сети виртуальной машины, выбрав диапазон одноранговых виртуальных сетей и локальных сетей.

   Виртуальные машины Windows автоматически используют source NAT для трафика, направленного за пределы подсети, к которой принадлежит виртуальная машина. Невозможно указать пользовательские диапазоны IP-адресов.

После завершения предыдущих шагов модулям pod, созданным на виртуальных машинах агента Kubernetes, автоматически назначаются частные IP-адреса из виртуальной сети.

## <a name="deploy-plug-in-for-docker-containers"></a>Развертывание подключаемого модуля для контейнеров Docker

1. [Скачайте и установите подключаемый модуль](#download-and-install-the-plug-in).
2. Чтобы создать контейнеры Docker, выполните следующую команду:

   ```
   ./docker-run.sh \<container-name\> \<container-namespace\> \<image\>
   ```

Контейнеры автоматически начнут получать IP-адреса из выделенного пула. Если вы хотите распределить нагрузку трафика для контейнеров Docker, их необходимо разместить за программной подсистемой балансировки нагрузки, и вы должны настроить пробу подсистемы балансировки нагрузки так же, как вы создаете политику и пробы для виртуальной машины.

### <a name="cni-network-configuration-file"></a>Файл конфигурации сети CNI

Файл конфигурации сети CNI описан в формате JSON. По умолчанию он находится в `/etc/cni/net.d` для Linux и `c:\cni\netconf` для Windows. Файл указывает конфигурацию подключаемого модуля и отличается для Windows и Linux. Ниже приведен пример файла конфигурации JSON для Linux и объяснения для некоторых ключевых параметров. В файл не требуется вносить изменения:

```json
{
       "cniVersion":"0.3.0",
       "name":"azure",
       "plugins":[
          {
             "type":"azure-vnet",
             "mode":"bridge",
             "bridge":"azure0",
             "ipam":{
                "type":"azure-vnet-ipam"
             }
          },
          {
             "type":"portmap",
             "capabilities":{
                "portMappings":true
             },
             "snat":true
          }
       ]
}
```

#### <a name="settings-explanation"></a>Объяснение параметров

- **cniVersion**: подключаемые модули CNI виртуальной сети Azure поддерживают версии 0.3.0 и 0.3.1 из [спецификации CNI](https://github.com/containernetworking/cni/blob/master/SPEC.md).
- **name**: имя сети. Это свойство может иметь любое уникальное значение.
- **type**: имя подключаемого модуля сети. Установлено *azure-vnet*.
- **mode**: режим работы. Это поле является необязательным. Поддерживается только режим bridge. Для получения дополнительных сведений см. раздел о [режимах работы](https://github.com/Azure/azure-container-networking/blob/master/docs/network.md).
- **bridge**: имя моста, который будет использоваться для подключения контейнеров к виртуальной сети. Это поле является необязательным. Если этот параметр опущен, подключаемый модуль автоматически выбирает уникальное имя на основе главного индекса интерфейса.
- **ipam type**: имя подключаемого модуля IPAM. Всегда установлено *azure-vnet-ipam*.

## <a name="download-and-install-the-plug-in"></a>Загрузка и установка подключаемого модуля

Загрузите подключаемый модуль с [GitHub](https://github.com/Azure/azure-container-networking/releases). Скачайте последнюю версию для платформы, которую вы используете:

- **Linux**: [Azure-vnet-CNI-Linux-amd64- \<version no.\> . tgz](https://github.com/Azure/azure-container-networking/releases/download/v1.0.12-rc3/azure-vnet-cni-linux-amd64-v1.0.12-rc3.tgz)
- **Windows**: [Azure-vnet-CNI-Windows-amd64- \<version no.\> . zip](https://github.com/Azure/azure-container-networking/releases/download/v1.0.12-rc3/azure-vnet-cni-windows-amd64-v1.0.12-rc3.zip)

Скопируйте скрипт установки для [Linux](https://github.com/Azure/azure-container-networking/blob/master/scripts/install-cni-plugin.sh) или [Windows](https://github.com/Azure/azure-container-networking/blob/master/scripts/Install-CniPlugin.ps1) на компьютер. Сохраните скрипт в каталог `scripts` на компьютере и назовите файл `install-cni-plugin.sh` для Linux или `install-cni-plugin.ps1` для Windows. Чтобы установить подключаемый модуль, запустите соответствующий скрипт для вашей платформы, указав версию используемого подключаемого модуля. Например, укажите *v1.0.12-rc3*:

   ```bash
   \$scripts/install-cni-plugin.sh [version]
   ```

   ```powershell
   scripts\\ install-cni-plugin.ps1 [version]
   ```

Скрипт автоматически установит подключаемый модуль в `/opt/cni/bin` для Linux и `c:\cni\bin` для Windows. Установленный подключаемый модуль поставляется с простым файлом конфигурации сети, который работает после установки. Его не нужно обновлять. Дополнительные сведения о параметрах в файле см. в разделе [Файл конфигурации сети CNI](#cni-network-configuration-file).