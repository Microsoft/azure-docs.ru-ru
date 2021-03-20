---
title: Azure Service Fabric настроить обратный прокси-сервер
description: Узнайте, как настроить и настроить службу обратных прокси-серверов для приложения Azure Service Fabric.
ms.topic: conceptual
ms.date: 11/13/2018
ms.author: pepogors
ms.openlocfilehash: f8a9025a50b2815f0e6030e7baf317b261c8c462
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86256338"
---
# <a name="set-up-and-configure-reverse-proxy-in-azure-service-fabric"></a>Установка и настройка обратного прокси-сервера в Azure Service Fabric
Необязательный обратный прокси-сервер в службе Azure Service Fabric помогает микрослужбам, работающим в кластере Service Fabric, обнаруживать другие службы с конечными точками HTTP и обмениваться данными с этими службами. См. дополнительные сведения об [обратном прокси-сервере в Azure Service Fabric](service-fabric-reverseproxy.md). В этой статье показано, как установить и настроить обратный прокси-сервер в кластере. 

## <a name="enable-reverse-proxy-using-azure-portal"></a>Включение обратного прокси-сервера с помощью портала Azure

Обратный прокси-сервер можно включить во время создания кластера Service Fabric на портале Azure. На портале Azure нельзя добавить обратный прокси-сервер к существующему кластеру. 

Чтобы настроить обратный прокси-сервер при [создании кластера с помощью портала Azure](./service-fabric-cluster-creation-via-portal.md), сделайте следующее:

1. На **шаге 2 (настройка кластера)** выберите для параметра **Конфигурация типа узла** значение **Включить обратный прокси-сервер**.

   ![Включение обратного прокси-сервера на портале](./media/service-fabric-reverseproxy-setup/enable-rp-portal.png)
2. Используемых Чтобы настроить безопасный обратный прокси-сервер, необходимо настроить сертификат TLS/SSL. На **шаге 3 (безопасность)** в разделе **Настройка параметров безопасности кластера** выберите для параметра **Тип конфигурации** значение **Пользовательский**. Затем в разделе **SSL-сертификат обратного прокси-сервера** установите флажок **Включить SSL-сертификат для обратного прокси-сервера** и введите данные сертификата.

   ![Настройка защищенного обратного прокси-сервера на портале](./media/service-fabric-reverseproxy-setup/configure-rp-certificate-portal.png)

   Если вы не хотите настраивать сертификат для обратного прокси-сервера при создании кластера, это можно сделать позже с помощью шаблона Resource Manager для группы ресурсов кластера. См. дополнительные сведения о [включении обратного прокси-сервера с помощью шаблонов Azure Resource Manager](#enable-reverse-proxy-via-azure-resource-manager-templates).

## <a name="enable-reverse-proxy-via-azure-resource-manager-templates"></a>Включение обратного прокси-сервера с помощью шаблонов Azure Resource Manager

Обратный прокси-сервер в Service Fabric можно включить для кластера Azure с помощью шаблона Azure Resource Manager. Вы можете включить обратный прокси-сервер при создании кластера или позже, обновив параметры кластера. 

Для нового кластера вы можете [создать пользовательский шаблон Resource Manager](service-fabric-cluster-creation-via-arm.md) или использовать готовый пример шаблона. 

Примеры шаблонов Resource Manager, которые помогут вам настроить защищенный обратный прокси-сервер для кластера Azure, см. на [GitHub](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/Reverse-Proxy-Sample). В руководстве по [настройке обратного прокси-сервера HTTPS в защищенном кластере](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/Reverse-Proxy-Sample/README.md#configure-https-reverse-proxy-in-a-secure-cluster) README-файла приведены примеры шаблонов и инструкции по настройке защищенного обратного прокси-сервера с сертификатом и обновлению сертификатов.

Если кластер уже существует, вы можете экспортировать шаблон Resource Manager для группы кластерных ресурсов с помощью [портала Azure](../azure-resource-manager/templates/export-template-portal.md), [PowerShell](../azure-resource-manager/management/manage-resources-powershell.md) или [Azure CLI](../azure-resource-manager/management/manage-resources-cli.md).

Подготовив шаблон Resource Manager, вы можете включить обратный прокси-сервер:

1. Определите порт обратного прокси-сервера в разделе [Parameters](../azure-resource-manager/templates/template-syntax.md) шаблона.

    ```json
    "SFReverseProxyPort": {
        "type": "int",
        "defaultValue": 19081,
        "metadata": {
            "description": "Endpoint for Service Fabric Reverse proxy"
        }
    },
    ```
2. Укажите порт для каждого из объектов NodeType в [разделе Тип ресурса](../azure-resource-manager/templates/template-syntax.md) [**Microsoft. ServiceFabric/Clusters**](/azure/templates/microsoft.servicefabric/clusters) .

    Порт идентифицируется по имени параметра reverseProxyEndpointPort.

    ```json
    {
        "apiVersion": "2016-09-01",
        "type": "Microsoft.ServiceFabric/clusters",
        "name": "[parameters('clusterName')]",
        "location": "[parameters('clusterLocation')]",
        ...
       "nodeTypes": [
          {
           ...
           "reverseProxyEndpointPort": "[parameters('SFReverseProxyPort')]",
           ...
          },
        ...
        ],
        ...
    }
    ```
3. Чтобы настроить сертификаты TLS/SSL в порте для обратных прокси-серверов, добавьте сертификат в свойство ***reverseProxyCertificate** _ в разделе Тип ресурса _ *Microsoft. ServiceFabric/Clusters* *  [](../azure-resource-manager/templates/template-syntax.md).

    ```json
    {
        "apiVersion": "2016-09-01",
        "type": "Microsoft.ServiceFabric/clusters",
        "name": "[parameters('clusterName')]",
        "location": "[parameters('clusterLocation')]",
        "dependsOn": [
            "[concat('Microsoft.Storage/storageAccounts/', parameters('supportLogStorageAccountName'))]"
        ],
        "properties": {
            ...
            "reverseProxyCertificate": {
                "thumbprint": "[parameters('sfReverseProxyCertificateThumbprint')]",
                "x509StoreName": "[parameters('sfReverseProxyCertificateStoreName')]"
            },
            ...
            "clusterState": "Default",
        }
    }
    ```

### <a name="supporting-a-reverse-proxy-certificate-thats-different-from-the-cluster-certificate"></a>Поддержка сертификата обратного прокси-сервера, отличного от сертификата кластера
 Если сертификат обратного прокси-сервера отличается от сертификата, который защищает кластер, то указанный ранее сертификат необходимо установить на виртуальную машину и добавить в список управления доступом (ACL), чтобы предоставить Service Fabric к нему доступ. Это можно сделать в [разделе типа ресурсов](../azure-resource-manager/templates/template-syntax.md) [**Microsoft. COMPUTE/virtualMachineScaleSets**](/azure/templates/microsoft.compute/virtualmachinescalesets) . Чтобы установить сертификат, добавьте его в osProfile. Раздел extension шаблона позволяет обновить сертификат в списке управления доступом.

  ```json
  {
    "apiVersion": "[variables('vmssApiVersion')]",
    "type": "Microsoft.Compute/virtualMachineScaleSets",
    ....
      "osProfile": {
          "adminPassword": "[parameters('adminPassword')]",
          "adminUsername": "[parameters('adminUsername')]",
          "computernamePrefix": "[parameters('vmNodeType0Name')]",
          "secrets": [
            {
              "sourceVault": {
                "id": "[parameters('sfReverseProxySourceVaultValue')]"
              },
              "vaultCertificates": [
                {
                  "certificateStore": "[parameters('sfReverseProxyCertificateStoreValue')]",
                  "certificateUrl": "[parameters('sfReverseProxyCertificateUrlValue')]"
                }
              ]
            }
          ]
        }
   ....
   "extensions": [
          {
              "name": "[concat(parameters('vmNodeType0Name'),'_ServiceFabricNode')]",
              "properties": {
                      "type": "ServiceFabricNode",
                      "autoUpgradeMinorVersion": false,
                      ...
                      "publisher": "Microsoft.Azure.ServiceFabric",
                      "settings": {
                        "clusterEndpoint": "[reference(parameters('clusterName')).clusterEndpoint]",
                        "nodeTypeRef": "[parameters('vmNodeType0Name')]",
                        "dataPath": "D:\\\\SvcFab",
                        "durabilityLevel": "Bronze",
                        "testExtension": true,
                        "reverseProxyCertificate": {
                          "thumbprint": "[parameters('sfReverseProxyCertificateThumbprint')]",
                          "x509StoreName": "[parameters('sfReverseProxyCertificateStoreValue')]"
                        },
                  },
                  "typeHandlerVersion": "1.0"
              }
          },
      ]
    }
  ```
> [!NOTE]
> Чтобы в существующем кластере включить обратный прокси-сервер с сертификатом, отличающимся от сертификата кластера, необходимо сначала установить сертификат обратного прокси-сервера и обновить список управления доступом в кластере. Завершите развертывание [шаблона Azure Resource Manager](service-fabric-cluster-creation-via-arm.md), используя описанные выше параметры, прежде чем начинать развертывание для включения обратного прокси-сервера с помощью шагов 1–3.

## <a name="enable-reverse-proxy-on-standalone-clusters"></a>Включение обратного прокси-сервера в автономных кластерах

Для автономных кластеров обратный прокси-сервер включается в файле ClusterConfig.json. Вы можете включить обратный прокси-сервер при создании кластера или позже, обновив конфигурацию существующего кластера. Дополнительные сведения о параметрах в файле ClusterConfig.json см. в [описании параметров автономного кластера](./service-fabric-cluster-manifest.md).

Следующие шаги описывают настройки, позволяющие включить обратный прокси-сервер и, если потребуется, защитить его с помощью сертификата X.509. 

1. Чтобы включить обратный прокси-сервер, задайте значение **reverseProxyEndpointPort** для типа узла в разделе **свойства** в конфигурации кластера. Следующий код JSON показывает, как задать порт конечной точки обратный прокси-сервера равным 19081 для узлов с типом "NodeType0":

   ```json
       "properties": {
          ... 
           "nodeTypes": [
               {
                   "name": "NodeType0",
                   ...
                   "reverseProxyEndpointPort": "19081",
                   ...
               }
           ],
          ...
       }
   ```
2. (Необязательно.) Для защиты обратного прокси-сервера настройте сертификат в разделе **безопасности** на странице **свойств**. 
   - Для среды разработки или тестирования можно использовать параметр **ReverseProxyCertificate**:

      ```json
          "properties": {
              ...
              "security": {
                  ...
                  "CertificateInformation": {
                      ...
                      "ReverseProxyCertificate": {
                          "Thumbprint": "[Thumbprint]",
                          "ThumbprintSecondary": "[Thumbprint]",
                          "X509StoreName": "My"
                      },
                      ...
                  }
              },
              ...
          }
      ```
   - Для рабочей среды мы рекомендуем использовать параметр **ReverseProxyCertificateCommonNames**:

      ```json
          "properties": {
              ...
              "security": {
                  ...
                  "CertificateInformation": {
                      ...
                      "ReverseProxyCertificateCommonNames": {
                        "CommonNames": [
                            {
                              "CertificateCommonName": "[CertificateCommonName]"
                            }
                          ],
                          "X509StoreName": "My"
                      },
                      ...
                  }
              },
              ...
          }
      ```

   Дополнительные сведения о настройке и управлении сертификатами для изолированного кластера, а также подробные сведения о настройке сертификатов для защиты обратного прокси-сервера, см. в руководстве по [обеспечению защиты с использованием сертификата X509](./service-fabric-windows-cluster-x509-security.md).

Изменив настройки в файле ClusterConfig.json для включения обратного прокси, выполните инструкции из руководства по [обновлению конфигурации кластера](service-fabric-cluster-config-upgrade-windows-server.md), чтобы применить эти настройки к кластеру.


## <a name="expose-reverse-proxy-on-a-public-port-through-azure-load-balancer"></a>Предоставление обратного прокси-сервера на общедоступном порту с помощью Azure Load Balancer

Чтобы обращаться к обратному прокси-серверу за пределами кластера Azure, настройте правила Azure Load Balancer и зонд работоспособности Azure для порта обратного прокси-сервера. Эти действия можно выполнить с помощью портала Azure или шаблона Resource Manager в любое время после создания кластера. 

> [!WARNING]
> Настройка порта обратного прокси-сервера в подсистеме балансировки нагрузки обеспечит адресацию извне кластера всех микрослужб в этом кластере, которые предоставляют конечную точку HTTP. Это означает, пользователь-злоумышленник может обнаружить внутренние микрослужбы. Это представляет серьезную уязвимость, например:
>
> * Пользователь-злоумышленник может многократно выполнять атаку типа "отказ в обслуживании", вызывая внутреннюю службу, у которой нет достаточно защищенного направления атак.
> * Пользователь-злоумышленник может доставлять неправильно сформированные пакеты во внутреннюю службу, вызывая непредвиденное поведение.
> * Внутренняя служба может возвращать персональные или конфиденциальные сведения, не предназначенные для предоставления службам вне кластера, в результате чего пользователь-злоумышленник может завладеть подобной информацией. 
>
> Вам следует тщательно изучить и устранить потенциальные угрозы безопасности для кластера и выполняющихся в нем приложений, прежде чем делать порт обратного прокси-сервера общедоступным. 
>

Чтобы обратный прокси-сервер для изолированного кластера был общедоступным, правильный способ реализации будет зависеть от системы, в которой размещен кластер. Описание этих способов выходит за рамки этой статьи. Но при этом по-прежнему остается актуальным предупреждение о предоставлении общего доступа к обратному прокси-серверу.

### <a name="expose-the-reverse-proxy-using-azure-portal"></a>Предоставление обратного прокси-сервера с помощью портала Azure 

1. На портале Azure щелкните группу ресурсов для кластера, затем щелкните подсистему балансировки нагрузки для кластера.
2. Чтобы добавить зонд работоспособности для порта обратного прокси-сервера, в левой части окна подсистемы балансировки нагрузки в разделе **Параметры** щелкните **Зонды работоспособности**. Затем щелкните **Добавить** в верхней части окна зондов работоспособности, введите сведения о порте обратного прокси-сервера, а затем щелкните **ОК**. По умолчанию для обратного прокси-сервера используется порт 19081, если вы не указали другое значение при создании кластера.

   ![Настройка зонда работоспособности для обратного прокси-сервера](./media/service-fabric-reverseproxy-setup/lb-rp-probe.png)
3. Чтобы добавить зонд работоспособности для порта обратного прокси-сервера, в разделе **Параметры** в левой части окна подсистемы балансировки нагрузки щелкните **Зонды работоспособности**. Затем щелкните **Добавить** в верхней части окна "Правила балансировка нагрузки" и введите сведения о порте обратного прокси-сервера. В поле **Порт** задайте номер порта, на котором будет предоставляться обратный прокси-сервер, в поле **Внутренний порт** — значение, указанное при настройке обратного прокси-сервера, а в поле **Зонд работоспособности** — значение для зонда работоспособности, которое вы настроили на предыдущем шаге. Задайте нужные значения для других полей и щелкните **ОК**.

   ![Настройка правила подсистемы балансировки нагрузки для обратного прокси-сервера](./media/service-fabric-reverseproxy-setup/lb-rp-rule.png)

### <a name="expose-the-reverse-proxy-via-resource-manager-templates"></a>Предоставление обратного прокси-сервера через шаблоны Resource Manager

Приведенный ниже код JSON использует тот же шаблон, что и в разделе [Включение обратного прокси-сервера с помощью шаблонов Azure Resource Manager](#enable-reverse-proxy-via-azure-resource-manager-templates). Изучите этот раздел, если вам нужны сведения о создании шаблона Resource Manager или экспорте шаблона для существующего кластера.  Изменения вносятся в [раздел типа ресурсов](../azure-resource-manager/templates/template-syntax.md) [**Microsoft. Network/loadBalancers**](/azure/templates/microsoft.network/loadbalancers) .

```json
{
    "apiVersion": "[variables('lbApiVersion')]",
    "type": "Microsoft.Network/loadBalancers",
    ...
    ...
    "loadBalancingRules": [
        ...
        {
            "name": "LBSFReverseProxyRule",
            "properties": {
                "backendAddressPool": {
                    "id": "[variables('lbPoolID0')]"
                },
                "backendPort": "[parameters('SFReverseProxyPort')]",
                "enableFloatingIP": "false",
                "frontendIPConfiguration": {
                    "id": "[variables('lbIPConfig0')]"
                },
                "frontendPort": "[parameters('SFReverseProxyPort')]",
                "idleTimeoutInMinutes": "5",
                "probe": {
                    "id": "[concat(variables('lbID0'),'/probes/SFReverseProxyProbe')]"
                },
                "protocol": "tcp"
            }
        }
    ],
    "probes": [
        ...
        {
            "name": "SFReverseProxyProbe",
            "properties": {
                "intervalInSeconds": 5,
                "numberOfProbes": 2,
                "port":     "[parameters('SFReverseProxyPort')]",
                "protocol": "tcp"
            }
        }  
    ]
}
```


## <a name="customize-reverse-proxy-behavior-using-fabric-settings"></a>Настройка поведения для обратного прокси-сервера с помощью параметров структуры

Вы можете изменять поведение обратного прокси-сервера, используя параметры структуры в шаблоне Resource Manager для кластеров, размещенных в Azure, или в файле ClusterConfig.json для автономных кластеров. Параметры, управляющие поведением обратного прокси-сервера, находятся в разделе [**ApplicationGateway/Http**](./service-fabric-cluster-fabric-settings.md#applicationgatewayhttp), который находится в разделе **fabricSettings** в **свойствах** кластера. 

Например, вы можете задать значение **DefaultHttpRequestTimeout**, чтобы настроить время ожидания запросов на обратном прокси-сервере. Следующий фрагмент JSON устанавливает в этом параметре значение 180 секунд:

   ```json
   {
   "fabricSettings": [
             ...
             {
               "name": "ApplicationGateway/Http",
               "parameters": [
                 {
                   "name": "DefaultHttpRequestTimeout",
                   "value": "180"
                 }
               ]
             }
           ],
           ...
   }
   ``` 

Дополнительные сведения об обновлении параметров структуры для кластеров Azure см. в руководстве по [настройке параметров кластера с помощью шаблонов Resource Manager](service-fabric-cluster-config-upgrade-azure.md). Для автономных кластеров см. руководство по [настройке параметров кластера для автономных кластеров](service-fabric-cluster-config-upgrade-windows-server.md). 

Несколько параметров структуры используются для создания защищенного подключения между обратным прокси-сервером и службами. Подробные сведения об этих параметрах см. [в статье подключение к безопасной службе с помощью обратных прокси-серверов](service-fabric-reverseproxy-configure-secure-communication.md).

## <a name="next-steps"></a>Дальнейшие действия
* [Подключение к защищенной службе с помощью обратного прокси-сервера](service-fabric-reverseproxy-configure-secure-communication.md)
* Параметры конфигурации обратного прокси-сервера описаны в статье [Настройка параметров кластера Service Fabric и политики обновления структур](service-fabric-cluster-fabric-settings.md#applicationgatewayhttp).
