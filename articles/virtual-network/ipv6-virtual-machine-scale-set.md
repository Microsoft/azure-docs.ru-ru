---
title: Развертывание масштабируемых наборов виртуальных машин с помощью IPv6 в Azure
titlesuffix: Azure Virtual Network
description: В этой статье показано, как развертывать масштабируемые наборы виртуальных машин с IPv6 в виртуальной сети Azure.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/31/2020
ms.author: kumud
ms.openlocfilehash: f969d7edc22c9c36481ca42449193af5f8c7b0d9
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "84710003"
---
# <a name="deploy-virtual-machine-scale-sets-with-ipv6-in-azure"></a>Развертывание масштабируемых наборов виртуальных машин с помощью IPv6 в Azure

В этой статье показано, как развернуть масштабируемый набор виртуальных машин с двумя стеками (IPv4 + IPv6) с помощью внешней подсистемы балансировки нагрузки в виртуальной сети Azure. Процесс создания масштабируемого набора виртуальных машин с поддержкой IPv6 практически идентичен процессу создания отдельных виртуальных машин, описанных [здесь](ipv6-configure-standard-load-balancer-template-json.md). Начнем с действий, которые похожи на процедуры, описанные для отдельных виртуальных машин.
1.    Создайте общедоступные IP-адреса IPv4 и IPv6.
2.    Создайте подсистему балансировки нагрузки с двумя стеками.  
3.    Создание правил группы безопасности сети (NSG).  

Единственным шагом, который отличается от отдельных виртуальных машин, является создание конфигурации сетевого интерфейса (NIC), использующей ресурс масштабируемого набора виртуальных машин: networkProfile/networkInterfaceConfigurations. Структура JSON аналогична структуре объекта Microsoft. Network/networkInterfaces, которая используется для отдельных виртуальных машин с добавлением сетевой карты и IP-конфигурации IPv4 в качестве основного интерфейса с помощью атрибута **PRIMARY: true**  , как показано в следующем примере:

```json
          "networkProfile": {
            "networkInterfaceConfigurations": [
              {
                "name": "[variables('nicName')]",
                "properties": {
                  "primary": true,
          "networkSecurityGroup": {
            "id": "[resourceId('Microsoft.Network/networkSecurityGroups','VmssNsg')]"
          },                  
                  "ipConfigurations": [
                    {
                      "name": "[variables('ipConfigName')]",
                      "properties": {
                        "primary": true,
                        "subnet": {
                          "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', 'MyvirtualNetwork','Mysubnet')]"
                        },
                        "privateIPAddressVersion":"IPv4",                       
                        "publicipaddressconfiguration": {
                          "name": "pub1",
                          "properties": {
                            "idleTimeoutInMinutes": 15
                          }
                        },
                        "loadBalancerBackendAddressPools": [
                          {
                            "id": "[resourceId('Microsoft.Network/loadBalancers/backendAddressPools', 'loadBalancer', 'bePool'))]"
                          }
                        ],
                        "loadBalancerInboundNatPools": [
                          {
                            "id": "[resourceId('Microsoft.Network/loadBalancers/inboundNatPools', 'loadBalancer', 'natPool')]"
                          }
                        ]
                      }
                    },
                    {
                      "name": "[variables('ipConfigNameV6')]",
                      "properties": {
                        "subnet": {
                          "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets','MyvirtualNetwork','Mysubnet')]"
                        },
                        "privateIPAddressVersion":"IPv6",
                        "loadBalancerBackendAddressPools": [
                          {
                            "id": "[resourceId('Microsoft.Network/loadBalancers/backendAddressPools', 'loadBalancer','bePoolv6')]"
                          }
                        ],                        
                      }
                    }
                  ]
                }
              }
            ]
          }

```


## <a name="sample-virtual-machine-scale-set-template-json"></a>Пример шаблона масштабируемого набора виртуальных машин в формате JSON

[Вот](https://azure.microsoft.com/resources/templates/ipv6-in-vnet-vmss/)как можно развернуть масштабируемый набор виртуальных машин с двойным стеком (IPv4 + IPv6) с двумя внешними Load Balancer и примером шаблона представления виртуальной сети.
## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о поддержке IPv6 в виртуальных сетях Azure см. в статье [что такое IPv6 для виртуальной сети Azure?](ipv6-overview.md).
