---
title: Перемещение внешней подсистемы балансировки нагрузки Azure в другой регион Azure с помощью портал Azure
description: Используйте шаблон Azure Resource Manager, чтобы переместить внешнюю подсистему балансировки нагрузки из одного региона Azure в другой с помощью портал Azure.
author: asudbring
ms.service: load-balancer
ms.topic: how-to
ms.date: 09/17/2019
ms.author: allensu
ms.openlocfilehash: f83ff3d1d03354daef3466c1f48eaa505e378634
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94693755"
---
# <a name="move-an-external-load-balancer-to-another-region-by-using-the-azure-portal"></a>Перемещение внешней подсистемы балансировки нагрузки в другой регион с помощью портал Azure

Существует несколько сценариев, в которых необходимо переместить внешнюю подсистему балансировки нагрузки из одного региона в другой. Например, может потребоваться создать другую внешнюю подсистему балансировки нагрузки с той же конфигурацией для тестирования. Также может потребоваться переместить внешнюю подсистему балансировки нагрузки в другой регион в рамках планирования аварийного восстановления.

В литеральном смысле вы не можете переместить внешнюю подсистему балансировки нагрузки Azure из одного региона в другой. Но вы можете использовать шаблон Azure Resource Manager для экспорта существующей конфигурации и общедоступного IP-адреса внешней подсистемы балансировки нагрузки. Затем можно разместить ресурс в другом регионе, экспортировав подсистему балансировки нагрузки и общедоступный IP-адрес в шаблон, изменив параметры в соответствии с регионом назначения, а затем развернув шаблон в новом регионе. Дополнительные сведения о диспетчер ресурсов и шаблонах см. [в разделе Экспорт групп ресурсов в шаблоны](../azure-resource-manager/management/manage-resource-groups-powershell.md#export-resource-groups-to-templates).


## <a name="prerequisites"></a>Предварительные условия

- Убедитесь, что внешняя подсистема балансировки нагрузки Azure находится в регионе Azure, из которого вы хотите переместиться.

- Внешние подсистемы балансировки нагрузки Azure нельзя перемещать между регионами. Необходимо связать новую подсистему балансировки нагрузки с ресурсами в целевом регионе.

- Чтобы экспортировать конфигурацию внешней подсистемы балансировки нагрузки и развернуть шаблон для создания внешней подсистемы балансировки нагрузки в другом регионе, необходимо назначить роль "участник сети" или выше.

- Определите структуру сети в исходном регионе и все ресурсы, которые вы сейчас используете. Этот макет включает, но не ограничен подсистемами балансировки нагрузки, группами безопасности сети, общедоступными IP-адресами и виртуальными сетями.

- Убедитесь, что подписка Azure позволяет создавать внешние подсистемы балансировки нагрузки в целевом регионе. Свяжитесь со службой поддержки, чтобы включить необходимые квоты.

- Убедитесь, что у вашей подписки достаточно ресурсов для поддержки добавления подсистем балансировки нагрузки. Ознакомьтесь со статьей [Подписка Azure, границы, квоты и ограничения службы](../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits).

## <a name="prepare-and-move"></a>Подготовка и перемещение
В следующих процедурах показано, как подготовить внешнюю подсистему балансировки нагрузки для перемещения с помощью шаблона диспетчер ресурсов и переместить конфигурацию внешней подсистемы балансировки нагрузки в целевой регион с помощью портал Azure. Сначала необходимо экспортировать общедоступную IP-конфигурацию внешней подсистемы балансировки нагрузки.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

### <a name="export-the-public-ip-template-and-deploy-the-public-ip-from-the-portal"></a>Экспорт шаблона общедоступного IP-адреса и его развертывание на портале

1. Войдите в портал [Azure](https://portal.azure.com) и выберите **Группы ресурсов**.
2. Выберите группу ресурсов, содержащую общедоступный IP-адрес источника, и щелкните ее.
3. Выберите **Параметры**  >  **Экспорт шаблона**.
4. Выберите **развертывание** в разделе **Экспорт шаблона**.
5. Выберите **шаблон**  >  **изменить параметры** , чтобы открыть parameters.jsв файле в интерактивном редакторе.
8. Чтобы изменить параметр имени общедоступного IP-адреса, измените свойство **value** в разделе **Параметры** с ОБЩЕдоступного IP-адреса на имя целевого общедоступного IP-адреса. Заключите имя в кавычки.

    ```json
            {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "publicIPAddresses_myVM1pubIP_name": {
            "value": "<target-publicip-name>"
              }
             }
            }

    ```

    Выберите **сохранить** в редакторе.

9.  Выберите **шаблон**  >  **изменить шаблон** , чтобы открыть template.jsв файле в интерактивном редакторе.

10. Чтобы изменить целевой регион, в который будет перемещен общедоступный IP-адрес, измените свойство **Location** в разделе **ресурсы**:

    ```json
            "resources": [
            {
            "type": "Microsoft.Network/publicIPAddresses",
            "apiVersion": "2019-06-01",
            "name": "[parameters('publicIPAddresses_myPubIP_name')]",
            "location": "<target-region>",
            "sku": {
                "name": "Basic",
                "tier": "Regional"
            },
            "properties": {
                "provisioningState": "Succeeded",
                "resourceGuid": "7549a8f1-80c2-481a-a073-018f5b0b69be",
                "ipAddress": "52.177.6.204",
                "publicIPAddressVersion": "IPv4",
                "publicIPAllocationMethod": "Dynamic",
                "idleTimeoutInMinutes": 4,
                "ipTags": []
               }
               }
             ]
    ```
  
    Чтобы получить коды расположения регионов, см. раздел [расположения Azure](https://azure.microsoft.com/global-infrastructure/locations/). Код для региона — это имя региона без пробелов. Например, код для Центральной нас — **centralus**.
    
12. Кроме того, в зависимости от требований можно изменить другие параметры в шаблоне.

    * **Номер SKU**. Вы можете изменить номер SKU общедоступного IP-адреса в конфигурации с Standard на базовый или с Basic на Standard, изменив свойство **Name** в разделе **SKU** в template.jsв файле:

        ```json
          "resources": [
         {
            "type": "Microsoft.Network/publicIPAddresses",
            "apiVersion": "2019-06-01",
            "name": "[parameters('publicIPAddresses_myPubIP_name')]",
            "location": "<target-region>",
            "sku": {
                "name": "Basic",
                "tier": "Regional"
            },
        ```

        Сведения о различиях между общедоступными IP-адресами уровня "базовый" и "Стандартный" см. [в статье Создание, изменение или удаление общедоступного адреса](../virtual-network/virtual-network-public-ip-address.md).

    * **Метод выделения общедоступного IP-адреса** и **время ожидания простоя**. Можно изменить метод выделения общедоступного IP-адреса, изменив свойство **публиЦипаллокатионмесод** с **dynamic** на **static** или **static на** **dynamic**. Вы можете изменить время ожидания простоя, изменив свойство **идлетимеаутинминутес** на нужное значение. Значение по умолчанию — **4**.

        ```json
          "resources": [
         {
            "type": "Microsoft.Network/publicIPAddresses",
            "apiVersion": "2019-06-01",
            "name": "[parameters('publicIPAddresses_myPubIP_name')]",
            "location": "<target-region>",
            "sku": {
                "name": "Basic",
                "tier": "Regional"
            },
            "properties": {
                "provisioningState": "Succeeded",
                "resourceGuid": "7549a8f1-80c2-481a-a073-018f5b0b69be",
                "ipAddress": "52.177.6.204",
                "publicIPAddressVersion": "IPv4",
                "publicIPAllocationMethod": "Dynamic",
                "idleTimeoutInMinutes": 4,
                "ipTags": []

        ```

        Сведения о методах выделения и значениях времени ожидания простоя см. [в разделе Создание, изменение или удаление общедоступного IP-адреса](../virtual-network/virtual-network-public-ip-address.md).

 
13. Выберите **сохранить** в интерактивном редакторе.

14. Выберите **основные**  >  **подписки** , чтобы выбрать подписку, в которой будет развернут целевой общедоступный IP-адрес.

15. Выберите **основные**  >  **группы ресурсов** , чтобы выбрать группу ресурсов, в которой будет развернут целевой общедоступный IP-адрес. Можно выбрать **создать** , чтобы создать новую группу ресурсов для целевого общедоступного IP-адреса. Убедитесь, что имя не совпадает с исходной группой ресурсов существующего исходного общедоступного IP-адреса.

16. Убедитесь, что в качестве **основы**  >  **расположения** выбрано целевое расположение, в котором должен быть развернут общедоступный IP-адрес.

17. В разделе **Параметры** убедитесь, что имя совпадает с именем, введенным ранее в редакторе параметров.

18. Установите флажок **условия** .

19. Выберите **приобрести** , чтобы развернуть целевой общедоступный IP-адрес.

20. Если у вас есть другой общедоступный IP-адрес, используемый для исходящего трафика NAT для перемещаемой подсистемы балансировки нагрузки, повторите предыдущие шаги для экспорта и развертывания второго исходящего общедоступного IP-адреса в целевом регионе.

### <a name="export-the-external-load-balancer-template-and-deploy-the-load-balancer-from-the-azure-portal"></a>Экспортируйте шаблон внешней подсистемы балансировки нагрузки и разверните подсистему балансировки нагрузки из портал Azure

1. Войдите в портал [Azure](https://portal.azure.com) и выберите **Группы ресурсов**.
2. Выберите группу ресурсов, содержащую источник внешней подсистемы балансировки нагрузки, и щелкните ее.
3. Выберите **Параметры**  >  **Экспорт шаблона**.
4. Выберите **развертывание** в разделе **Экспорт шаблона**.
5. Выберите **шаблон**  >  **изменить параметры** , чтобы открыть parameters.jsв файле в интерактивном редакторе.

5. Чтобы изменить параметр имени внешней подсистемы балансировки нагрузки, измените свойство **value** исходного имени внешней подсистемы балансировки нагрузки на имя целевой внешней подсистемы балансировки нагрузки. Заключите имя в кавычки.

    ```json
       "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
       "contentVersion": "1.0.0.0",
       "parameters": {
          "loadBalancers_myLoadbalancer_ext_name": {
          "value": "<target-external-lb-name>"
    },
          "publicIPAddresses_myPubIP_in_externalid": {
          "value": "<target-publicIP-resource-ID>"
    },

    ```

6.  Чтобы изменить значение целевого общедоступного IP-адреса, перемещенного на предыдущих шагах, сначала необходимо получить идентификатор ресурса, а затем вставить его в parameters.jsв файле. Чтобы получить идентификатор:

    1. На другом вкладке или окне браузера Войдите в [портал Azure](https://portal.azure.com) и выберите **группы ресурсов**.
    2. Откройте целевую группу ресурсов, содержащую общедоступный IP-адрес, который вы переместили на предыдущих шагах. Выберите ее.
    3. Выберите **Параметры** > **Свойства**.
    4. В колонке справа выделите **идентификатор ресурса** и скопируйте его в буфер обмена. Кроме того, можно выбрать пункт **Копировать в буфер обмена** справа от параметра путь к **идентификатору ресурса** .
    5. Вставьте идентификатор ресурса в свойство **значение** в редакторе **изменения параметров** , которое открыто в другом окне или на вкладке браузера:

        ```json
           ```json
           "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
           "contentVersion": "1.0.0.0",
           "parameters": {
              "loadBalancers_myLoadbalancer_ext_name": {
              "value": "<target-external-lb-name>"
        },
              "publicIPAddresses_myPubIP_in_externalid": {
              "value": "<target-publicIP-resource-ID>"
        },

        ```
    6. Выберите **сохранить** в интерактивном редакторе.


7.  Если вы настроили исходящие правила NAT и исходящего трафика для балансировщика нагрузки, вы увидите третью запись в этом файле для внешнего идентификатора исходящего общедоступного IP-адреса. Повторите предыдущие шаги в **целевом регионе** , чтобы получить идентификатор для исходящего общедоступного IP-адреса. Вставьте этот идентификатор в parameters.jsфайла:

    ```json
            "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
            "parameters": {
                "loadBalancers_myLoadbalancer_ext_name": {
                "value": "<target-external-lb-name>",

            },
                "publicIPAddresses_myPubIP_in_externalid": {
                "value": "<target-publicIP-resource-ID>",

            },
                "publicIPAddresses_myPubIP_out_externalid": {
                "defaultValue": "<target-publicIP-outbound-resource-ID>",

            }
        },
    ```

8.  Выберите **шаблон**  >  **изменить шаблон** , чтобы открыть template.jsв файле в интерактивном редакторе.
9.  Чтобы изменить целевой регион, в который будет перемещена конфигурация внешней подсистемы балансировки нагрузки, измените свойство **Location** в разделе **ресурсы** в template.jsфайл:

    ```json
        "resources": [
            {
                "type": "Microsoft.Network/loadBalancers",
                "apiVersion": "2019-06-01",
                "name": "[parameters('loadBalancers_myLoadBalancer_name')]",
                "location": "<target-external-lb-region>",
                "sku": {
                    "name": "Standard",
                    "tier": "Regional"
                },
    ```

10. Чтобы получить коды расположения регионов, см. раздел [расположения Azure](https://azure.microsoft.com/global-infrastructure/locations/). Код для региона — это имя региона без пробелов. Например, код для Центральной нас — **centralus**.

11. Кроме того, в зависимости от требований можно изменить другие параметры в шаблоне.

    * **Номер SKU**. Номер SKU внешней подсистемы балансировки нагрузки можно изменить в конфигурации с Standard на Basic или с Basic на Standard, изменив свойство **Name** в разделе **SKU** в template.jsв файле:

        ```json
        "resources": [
        {
            "type": "Microsoft.Network/loadBalancers",
            "apiVersion": "2019-06-01",
            "name": "[parameters('loadBalancers_myLoadBalancer_name')]",
            "location": "<target-external-lb-region>",
            "sku": {
                "name": "Standard",
                "tier": "Regional"
            },
        ```
      Сведения о различиях между подсистемами балансировки нагрузки уровня "базовый" и "Стандартный" см. в статье [обзор Load Balancer (цен. Категория "Стандартный") Azure](./load-balancer-overview.md).

    * **Правила балансировки нагрузки**. Вы можете добавить или удалить правила балансировки нагрузки в конфигурации, добавив или удалив записи в разделе **лоадбаланЦингрулес** template.jsв файле:

        ```json
        "loadBalancingRules": [
                    {
                        "name": "myInboundRule",
                        "etag": "W/\"39e5e9cd-2d6d-491f-83cf-b37a259d86b6\"",
                        "properties": {
                            "provisioningState": "Succeeded",
                            "frontendIPConfiguration": {
                                "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/frontendIPConfigurations/myfrontendIPinbound')]"
                            },
                            "frontendPort": 80,
                            "backendPort": 80,
                            "enableFloatingIP": false,
                            "idleTimeoutInMinutes": 4,
                            "protocol": "Tcp",
                            "enableTcpReset": false,
                            "loadDistribution": "Default",
                            "disableOutboundSnat": true,
                            "backendAddressPool": {
                                "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/backendAddressPools/myBEPoolInbound')]"
                            },
                            "probe": {
                                "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/probes/myHTTPProbe')]"
                            }
                        }
                    }
                ]
        ```
       Сведения о правилах балансировки нагрузки см. в разделе [что такое Azure Load Balancer?](https://docs.microsoft.com/azure/load-balancer/load-balancer-overview).

    * **Зонды**. Вы можете добавить или удалить пробу для балансировщика нагрузки в конфигурации, добавив или удалив записи в разделе **зонды** template.jsв файле:

        ```json
        "probes": [
                    {
                        "name": "myHTTPProbe",
                        "etag": "W/\"39e5e9cd-2d6d-491f-83cf-b37a259d86b6\"",
                        "properties": {
                            "provisioningState": "Succeeded",
                            "protocol": "Http",
                            "port": 80,
                            "requestPath": "/",
                            "intervalInSeconds": 15,
                            "numberOfProbes": 2
                        }
                    }
                ],
        ```
       Дополнительные сведения см. в разделе [Load Balancer проверки работоспособности](https://docs.microsoft.com/azure/load-balancer/load-balancer-custom-probe-overview).

    * **Правила NAT для входящего трафика**. Вы можете добавить или удалить правила NAT для входящего трафика для подсистемы балансировки нагрузки, добавив или удалив записи в разделе **inboundNatRules** template.jsв файле:

        ```json
        "inboundNatRules": [
                    {
                        "name": "myInboundNATRule",
                        "etag": "W/\"39e5e9cd-2d6d-491f-83cf-b37a259d86b6\"",
                        "properties": {
                            "provisioningState": "Succeeded",
                            "frontendIPConfiguration": {
                                "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/frontendIPConfigurations/myfrontendIPinbound')]"
                            },
                            "frontendPort": 4422,
                            "backendPort": 3389,
                            "enableFloatingIP": false,
                            "idleTimeoutInMinutes": 4,
                            "protocol": "Tcp",
                            "enableTcpReset": false
                        }
                    }
                ]
        ```
        Чтобы завершить добавление или удаление правила NAT для входящего трафика, оно должно присутствовать или удалено в качестве свойства **типа** в конце template.jsв файле:

        ```json
        {
            "type": "Microsoft.Network/loadBalancers/inboundNatRules",
            "apiVersion": "2019-06-01",
            "name": "[concat(parameters('loadBalancers_myLoadBalancer_name'), '/myInboundNATRule')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name'))]"
            ],
            "properties": {
                "provisioningState": "Succeeded",
                "frontendIPConfiguration": {
                    "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/frontendIPConfigurations/myfrontendIPinbound')]"
                },
                "frontendPort": 4422,
                "backendPort": 3389,
                "enableFloatingIP": false,
                "idleTimeoutInMinutes": 4,
                "protocol": "Tcp",
                "enableTcpReset": false
            }
        }
        ```
        Сведения о правилах NAT для входящего трафика см. в разделе [что такое Azure Load Balancer?](https://docs.microsoft.com/azure/load-balancer/load-balancer-overview).

    * **Правила исходящего трафика**. Вы можете добавить или удалить правила исходящего трафика в конфигурации, изменив свойство **аутбаундрулес** в template.jsв файле:

        ```json
        "outboundRules": [
                    {
                        "name": "myOutboundRule",
                        "etag": "W/\"39e5e9cd-2d6d-491f-83cf-b37a259d86b6\"",
                        "properties": {
                            "provisioningState": "Succeeded",
                            "allocatedOutboundPorts": 10000,
                            "protocol": "All",
                            "enableTcpReset": false,
                            "idleTimeoutInMinutes": 15,
                            "backendAddressPool": {
                                "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/backendAddressPools/myBEPoolOutbound')]"
                            },
                            "frontendIPConfigurations": [
                                {
                                    "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/frontendIPConfigurations/myfrontendIPoutbound')]"
                                }
                            ]
                        }
                    }
                ]
        ```

         Дополнительные сведения см. в разделе [Load Balancer правила исходящего трафика](./load-balancer-outbound-connections.md#outboundrules).

12. Выберите **сохранить** в интерактивном редакторе.

13. Выберите **основные**  >  **подписки** , чтобы выбрать подписку, в которой будет развернута Целевая внешняя подсистема балансировки нагрузки.

15. Выберите **основные**  >  **группы ресурсов** , чтобы выбрать группу ресурсов, в которой будет развернута Целевая подсистема балансировки нагрузки. Чтобы создать группу ресурсов для целевой внешней подсистемы балансировки нагрузки, можно выбрать **создать новый** . Или можно выбрать существующую группу ресурсов, созданную ранее для общедоступного IP-адреса. Убедитесь, что имя не совпадает с исходной группой ресурсов существующего источника внешняя подсистема балансировки нагрузки.

16. Убедитесь, что в качестве **основы**  >  **расположения** выбрано целевое расположение, в котором должна быть развернута внешняя подсистема балансировки нагрузки.

17. В разделе **Параметры** убедитесь, что имя совпадает с именем, введенным ранее в редакторе параметров. Убедитесь, что идентификаторы ресурсов заполнены для всех общедоступных IP-адресов в конфигурации.

18. Установите флажок **условия** .

19. Выберите **приобрести** , чтобы развернуть целевой общедоступный IP-адрес.

## <a name="discard"></a>Игнорировать

Если вы хотите отменить целевой общедоступный IP-адрес и внешнюю подсистему балансировки нагрузки, удалите группу ресурсов, которая содержит их. Для этого выберите группу ресурсов на панели мониторинга на портале, а затем выберите **Удалить** в верхней части страницы обзора.

## <a name="clean-up"></a>Очистка

Чтобы зафиксировать изменения и завершить перемещение общедоступного IP-адреса и внешней подсистемы балансировки нагрузки, удалите исходный общедоступный IP-адрес и внешнюю подсистему балансировки нагрузки или группу ресурсов. Для этого выберите группу ресурсов на панели мониторинга на портале, а затем выберите **Удалить** в верхней части каждой страницы.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы переместили внешнюю подсистему балансировки нагрузки Azure из одного региона в другой и очистили исходные ресурсы. Дополнительные сведения о перемещении ресурсов между регионами и аварийным восстановлением в Azure см. в следующих статьях:


- [Перемещение ресурсов в новую группу ресурсов или подписку](../azure-resource-manager/management/move-resource-group-and-subscription.md)
- [Перенос виртуальных машин Azure в другой регион](../site-recovery/azure-to-azure-tutorial-migrate.md)