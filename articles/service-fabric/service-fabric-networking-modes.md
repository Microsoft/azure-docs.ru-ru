---
title: Настройка сетевых режимов для служб контейнеров
description: Узнайте, как настроить разные сетевые режимы, которые поддерживаются в Azure Service Fabric.
author: athinanthny
ms.topic: conceptual
ms.date: 2/23/2018
ms.author: atsenthi
ms.openlocfilehash: e6174f35bd54b3ca0b2c5240a663369350b30ce8
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86241902"
---
# <a name="service-fabric-container-networking-modes"></a>Сетевые режимы контейнеров Service Fabric

Кластер Azure Service Fabric для служб контейнеров по умолчанию поддерживает сетевой режим **NAT**. Если используется режим NAT и подключения к одному порту прослушивают несколько служб контейнеров, могут возникнуть ошибки развертывания. Чтобы несколько служб контейнеров можно было разместить на одном порту, Service Fabric поддерживает **открытый** сетевой режим (в версии 5.7 и более поздних). В открытом режиме каждая служба контейнеров динамически получает внутренний IP-адрес, что позволяет нескольким службам прослушивать один и тот же порт.  

Если в манифесте службы указана одна служба контейнеров со статической конечной точкой, вы можете создавать и удалять службы в открытом сетевом режиме без ошибок развертывания. При статическом сопоставлении портов один и тот же файл docker-compose.yml можно использовать для создания нескольких служб.

Когда служба контейнеров перезапускается или перемещается на другой узел в кластере, ее IP-адрес изменяется. По этой причине мы не рекомендуем использовать динамически назначаемый IP-адрес для обнаружения служб контейнеров. Для обнаружения служб следует использовать только службу именования Service Fabric или службу DNS. 

>[!WARNING]
>Azure позволяет всего 65 356 IP-адресов на виртуальную сеть. Сумма количества узлов и количества экземпляров службы контейнеров (использующих открытый режим) не может превышать 65 356 IP-адресов в виртуальной сети. Для сценариев высокой плотности мы рекомендуем применять сетевой режим NAT. Кроме того, другие зависимости, такие как балансировщик нагрузки, будут иметь другие [ограничения](../azure-resource-manager/management/azure-subscription-service-limits.md) , которые следует учитывать. В настоящее время до 50 IP-адресов на узел проверено и проверено стабильно. 
>

## <a name="set-up-open-networking-mode"></a>Настройка открытого сетевого режима

1. Подготовьте шаблон Azure Resource Manager. В разделе кластерного ресурса **fabricSettings** включите службу DNS и поставщик IP-адресов. 

    ```json
    "fabricSettings": [
                {
                    "name": "DnsService",
                    "parameters": [
                       {
                            "name": "IsEnabled",
                            "value": "true"
                      }
                    ]
                },
                {
                    "name": "Hosting",
                    "parameters": [
                      { 
                            "name": "IPProviderEnabled",
                            "value": "true"
                      }
                    ]
                },
                {
                    "name": "Setup",
                    "parameters": [
                    {
                            "name": "ContainerNetworkSetup",
                            "value": "true"
                    }
                    ]
                }
            ],
    ```
    
2. Настройте раздел сетевого профиля ресурса "Набор масштабирования виртуальной машины". Это разрешает настройку нескольких IP-адресов на каждом узле кластера. Следующий пример устанавливает пять IP-адресов на каждом узле для кластера Service Fabric на Windows или Linux. Это означает, что на каждом узле могут существовать по пять экземпляров службы, прослушивающих один порт. Чтобы пять IP-адресов были доступны из Azure Load Balancer, зарегистрируйте эти пять IP-адресов в пуле адресов серверной части Azure Load Balancer, как показано ниже.  Вам также необходимо добавить переменные в начало шаблона в разделе переменных.

    Добавьте этот раздел в "Переменные".

    ```json
    "variables": {
        "nicName": "NIC",
        "vmName": "vm",
        "virtualNetworkName": "VNet",
        "vnetID": "[resourceId('Microsoft.Network/virtualNetworks',variables('virtualNetworkName'))]",
        "vmNodeType0Name": "[toLower(concat('NT1', variables('vmName')))]",
        "subnet0Name": "Subnet-0",
        "subnet0Prefix": "10.0.0.0/24",
        "subnet0Ref": "[concat(variables('vnetID'),'/subnets/',variables('subnet0Name'))]",
        "lbID0": "[resourceId('Microsoft.Network/loadBalancers',concat('LB','-', parameters('clusterName'),'-',variables('vmNodeType0Name')))]",
        "lbIPConfig0": "[concat(variables('lbID0'),'/frontendIPConfigurations/LoadBalancerIPConfig')]",
        "lbPoolID0": "[concat(variables('lbID0'),'/backendAddressPools/LoadBalancerBEAddressPool')]",
        "lbProbeID0": "[concat(variables('lbID0'),'/probes/FabricGatewayProbe')]",
        "lbHttpProbeID0": "[concat(variables('lbID0'),'/probes/FabricHttpGatewayProbe')]",
        "lbNatPoolID0": "[concat(variables('lbID0'),'/inboundNatPools/LoadBalancerBEAddressNatPool')]"
    }
    ```
    
    Добавьте следующий раздел в "Набор масштабирования виртуальной машины".

    ```json   
    "networkProfile": {
                "networkInterfaceConfigurations": [
                  {
                    "name": "[concat(parameters('nicName'), '-0')]",
                    "properties": {
                      "ipConfigurations": [
                        {
                          "name": "[concat(parameters('nicName'),'-',0)]",
                          "properties": {
                            "primary": "true",
                            "loadBalancerBackendAddressPools": [
                              {
                                "id": "[variables('lbPoolID0')]"
                              }
                            ],
                            "loadBalancerInboundNatPools": [
                              {
                                "id": "[variables('lbNatPoolID0')]"
                              }
                            ],
                            "subnet": {
                              "id": "[variables('subnet0Ref')]"
                            }
                          }
                        },
                        {
                          "name": "[concat(parameters('nicName'),'-', 1)]",
                          "properties": {
                            "primary": "false",
                            "loadBalancerBackendAddressPools": [
                              {
                                "id": "[variables('lbPoolID0')]"
                              }
                            ],
                            "subnet": {
                              "id": "[variables('subnet0Ref')]"
                            }
                          }
                        },
                        {
                          "name": "[concat(parameters('nicName'),'-', 2)]",
                          "properties": {
                            "primary": "false",
                            "loadBalancerBackendAddressPools": [
                              {
                                "id": "[variables('lbPoolID0')]"
                              }
                            ],
                            "subnet": {
                              "id": "[variables('subnet0Ref')]"
                            }
                          }
                        },
                        {
                          "name": "[concat(parameters('nicName'),'-', 3)]",
                          "properties": {
                            "primary": "false",
                            "loadBalancerBackendAddressPools": [
                              {
                                "id": "[variables('lbPoolID0')]"
                              }
                            ],
                            "subnet": {
                              "id": "[variables('subnet0Ref')]"
                            }
                          }
                        },
                        {
                          "name": "[concat(parameters('nicName'),'-', 4)]",
                          "properties": {
                            "primary": "false",
                            "loadBalancerBackendAddressPools": [
                              {
                                "id": "[variables('lbPoolID0')]"
                              }
                            ],
                            "subnet": {
                              "id": "[variables('subnet0Ref')]"
                            }
                          }
                        },
                        {
                          "name": "[concat(parameters('nicName'),'-', 5)]",
                          "properties": {
                            "primary": "false",
                            "loadBalancerBackendAddressPools": [
                              {
                                "id": "[variables('lbPoolID0')]"
                              }
                            ],
                            "subnet": {
                              "id": "[variables('subnet0Ref')]"
                            }
                          }
                        }
                      ],
                      "primary": true
                    }
                  }
                ]
              }
   ```
 
3. Только для кластеров Windows: настройте правило группы безопасности сети (NSG) Azure, которое открывает порт UDP/53 для виртуальной сети, со следующими значениями:

   |Параметр |Значение |
   | --- | --- |
   |Приоритет |2000 |
   |Имя |Custom_Dns  |
   |Источник |Виртуальная сеть |
   |Назначение | VirtualNetwork |
   |Служба | DNS (UDP/53) |
   |Действие | Allow  |

4. Укажите сетевой режим в манифесте приложения для каждой службы: `<NetworkConfig NetworkType="Open">`. При использовании **открытого** сетевого режима служба получит выделенный IP-адрес. Если режим не указан, по умолчанию служба использует режим **NAT**. В следующем примере манифеста видно, что службы `NodeContainerServicePackage1` и `NodeContainerServicePackage2` могут ожидать передачи данных на одном и том же порту (обе службы прослушивают `Endpoint1`). Если установлен открытый режим сети, вы не сможете задать конфигурации `PortBinding`.

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <ApplicationManifest ApplicationTypeName="NodeJsApp" ApplicationTypeVersion="1.0" xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
      <Description>Calculator Application</Description>
      <Parameters>
        <Parameter Name="ServiceInstanceCount" DefaultValue="3"></Parameter>
        <Parameter Name="MyCpuShares" DefaultValue="3"></Parameter>
      </Parameters>
      <ServiceManifestImport>
        <ServiceManifestRef ServiceManifestName="NodeContainerServicePackage1" ServiceManifestVersion="1.0"/>
        <Policies>
          <ContainerHostPolicies CodePackageRef="NodeContainerService1.Code" Isolation="hyperv">
           <NetworkConfig NetworkType="Open"/>
          </ContainerHostPolicies>
        </Policies>
      </ServiceManifestImport>
      <ServiceManifestImport>
        <ServiceManifestRef ServiceManifestName="NodeContainerServicePackage2" ServiceManifestVersion="1.0"/>
        <Policies>
          <ContainerHostPolicies CodePackageRef="NodeContainerService2.Code" Isolation="default">
            <NetworkConfig NetworkType="Open"/>
          </ContainerHostPolicies>
        </Policies>
      </ServiceManifestImport>
    </ApplicationManifest>
    ```

    Вы можете комбинировать и сопоставлять разные сетевые режимы для использования со службами в рамках приложения для кластера Windows. Можно настроить систему так, что некоторые службы будут работать в открытом режиме, а другие — в режиме NAT. Если служба настроена для режима NAT, она может прослушивать только уникальный порт.

    >[!NOTE]
    >В кластерах Linux использование разных сетевых режимов для разных служб не поддерживается. 
    >

5. При выборе **открытого** режима определение **конечной точки** манифеста службы должно явно указывать на пакет кода, соответствующий этой конечной точке, даже если пакет службы содержит только один пакет кода. 
   
   ```xml
   <Resources>
     <Endpoints>
       <Endpoint Name="ServiceEndpoint" Protocol="http" Port="80" CodePackageRef="Code"/>
     </Endpoints>
   </Resources>
   ```
   
6. В Windows при перезагрузке виртуальной машины повторно создается открытая сеть. Это позволяет устранить первоначальную проблему в сетевом стеке. Сеть повторно создается по умолчанию. Если это поведение нужно отключить, воспользуйтесь указанной ниже конфигурацией, а затем обновите ее.

```json
"fabricSettings": [
                {
                    "name": "Setup",
                    "parameters": [
                    {
                            "name": "SkipContainerNetworkResetOnReboot",
                            "value": "true"
                    }
                    ]
                }
            ],          
 ``` 
 
## <a name="next-steps"></a>Дальнейшие действия
* [Моделирование приложения в Service Fabric](service-fabric-application-model.md)
* [Указание ресурсов в манифесте службы](./service-fabric-service-manifest-resources.md) для Service Fabric
* [Развертывание контейнера Windows в Service Fabric на платформе Windows Server 2016](service-fabric-get-started-containers.md)
* [Развертывание контейнера Docker в Service Fabric на платформе Linux](service-fabric-get-started-containers-linux.md)
