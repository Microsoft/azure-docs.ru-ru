---
title: Перемещение внутренних Load Balancer Azure в другой регион Azure с помощью портал Azure
description: Используйте шаблон Azure Resource Manager, чтобы переместить внутренние Load Balancer Azure из одного региона Azure в другой с помощью портал Azure
author: asudbring
ms.service: load-balancer
ms.topic: how-to
ms.date: 09/18/2019
ms.author: allensu
ms.openlocfilehash: 68a2cb6926cb41956711a9e3c15d21c250d27f0b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94698499"
---
# <a name="move-azure-internal-load-balancer-to-another-region-using-the-azure-portal"></a>Перемещение внутренних Load Balancer Azure в другой регион с помощью портал Azure

Существуют различные сценарии, в которых необходимо переместить имеющуюся внутреннюю подсистему балансировки нагрузки из одного региона в другой. Например, может потребоваться создать внутреннюю подсистему балансировки нагрузки с той же конфигурацией для тестирования. Также может потребоваться переместить внутреннюю подсистему балансировки нагрузки в другой регион в рамках планирования аварийного восстановления.

Внутренние подсистемы балансировки нагрузки Azure невозможно переместить из одного региона в другой. Однако можно использовать шаблон Azure Resource Manager для экспорта существующей конфигурации и виртуальной сети внутренней подсистемы балансировки нагрузки.  Затем можно разместить ресурс в другом регионе, экспортировав подсистему балансировки нагрузки и виртуальную сеть в шаблон, изменив параметры в соответствии с регионом назначения, а затем развернув шаблоны в новом регионе.  Дополнительные сведения о Resource Manager и шаблонах см. в [Кратком руководстве по созданию и развертыванию шаблонов Azure Resource Manager с помощью портала Azure](../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md).


## <a name="prerequisites"></a>Предварительные условия

- Убедитесь, что внутренний балансировщик нагрузки Azure находится в регионе Azure, из которого вы хотите переместиться.

- Внутренние подсистемы балансировки нагрузки Azure нельзя перемещать между регионами.  Необходимо связать новую подсистему балансировки нагрузки с ресурсами в целевом регионе.

- Чтобы экспортировать конфигурацию внутренней подсистемы балансировки нагрузки и развернуть шаблон для создания внутренней подсистемы балансировки нагрузки в другом регионе, вам потребуется роль "участник сети" или выше.

- Определите структуру сети в исходном регионе и все ресурсы, которые вы сейчас используете. Этот макет включает, но не ограничен подсистемами балансировки нагрузки, группами безопасности сети, виртуальными машинами и виртуальными сетями.

- Убедитесь, что подписка Azure позволяет создавать внутренние подсистемы балансировки нагрузки в используемом целевом регионе. Свяжитесь со службой поддержки, чтобы включить необходимые квоты.

- Убедитесь, что у вашей подписки достаточно ресурсов для поддержки добавления подсистем балансировки нагрузки для этого процесса.  См. раздел [Подписка Azure, пределы, квоты и ограничения службы](../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits) .


## <a name="prepare-and-move"></a>Подготовка и перемещение
Ниже описано, как подготовить внутреннюю подсистему балансировки нагрузки для перемещения с помощью шаблона диспетчер ресурсов и переместить конфигурацию внутренней подсистемы балансировки нагрузки в целевую область с помощью портал Azure.  В рамках этого процесса необходимо добавить конфигурацию виртуальной сети для внутреннего балансировщика нагрузки и выполнить ее сначала перед перемещением внутренней подсистемы балансировки нагрузки.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

### <a name="export-the-virtual-network-template-and-deploy-from-the-azure-portal"></a>Экспортируйте шаблон виртуальной сети и разверните его из портал Azure

1. Войдите в [портал Azure](https://portal.azure.com)  >  **группы ресурсов**.
2. Выберите группу ресурсов, содержащую исходную виртуальную сеть, и щелкните ее.
3. Выберите **Параметры**>  >  **Экспорт шаблона**.
4. Выберите **развернуть** в колонке **Экспорт шаблона** .
5. Щелкните **шаблон**  >  **изменить параметры** , чтобы открыть **parameters.jsв** файле в интерактивном редакторе.
6. Чтобы изменить параметр имени виртуальной сети, измените свойство **value** в разделе **Параметры**.

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "virtualNetworks_myVNET1_name": {
                "value": "<target-virtual-network-name>"
            }
        }
    }
    ```
7. Измените значение параметра имя исходной виртуальной сети в редакторе на выбранное имя для целевой сети. Обязательно заключите имя в кавычки.

8. Нажмите кнопку **сохранить** в редакторе.

9. Щелкните **шаблон**  >  **изменить шаблон** , чтобы открыть **template.jsв** файле в интерактивном редакторе.

10. Чтобы изменить целевой регион, в который будет перемещена виртуальная сеть, измените свойство **Location** в разделе ресурсы:

    ```json
    "resources": [
                {
                    "type": "Microsoft.Network/virtualNetworks",
                    "apiVersion": "2019-06-01",
                    "name": "[parameters('virtualNetworks_myVNET1_name')]",
                    "location": "<target-region>",
                    "properties": {
                        "provisioningState": "Succeeded",
                        "resourceGuid": "6e2652be-35ac-4e68-8c70-621b9ec87dcb",
                        "addressSpace": {
                            "addressPrefixes": [
                                "10.0.0.0/16"
                            ]
                        },

    ```

11. Чтобы получить коды расположения регионов, см. раздел [расположения Azure](https://azure.microsoft.com/global-infrastructure/locations/).  Код для региона — это имя региона без пробелов, **Центральная американская**  =  **centralus**.

12. Кроме того, вы можете изменить другие параметры в **template.js** файле, если выбрать, и они необязательны в зависимости от требований:

    * **Адресное пространство** . адресное пространство виртуальной сети можно изменить перед сохранением, изменив раздел **Resources**  >  **аддрессспаце** и изменив свойство **аддресспрефиксес** в **template.jsв** файле:

        ```json
                "resources": [
                    {
                    "type": "Microsoft.Network/virtualNetworks",
                    "apiVersion": "2019-06-01",
                    "name": "[parameters('virtualNetworks_myVNET1_name')]",
                    "location": "<target-region",
                    "properties": {
                    "provisioningState": "Succeeded",
                    "resourceGuid": "6e2652be-35ac-4e68-8c70-621b9ec87dcb",
                    "addressSpace": {
                        "addressPrefixes": [
                        "10.0.0.0/16"
                        ]
                    },

        ```

    * **Подсеть** — имя подсети и адресное пространство подсети можно изменить или добавить в, изменив раздел **подсети** **template.jsв** файле. Имя подсети можно изменить, изменив свойство **Name** . Адресное пространство подсети можно изменить, изменив свойство **addressPrefix** в **template.jsв** файле:

        ```json
                "subnets": [
                    {
                    "name": "subnet-1",
                    "etag": "W/\"d9f6e6d6-2c15-4f7c-b01f-bed40f748dea\"",
                    "properties": {
                    "provisioningState": "Succeeded",
                    "addressPrefix": "10.0.0.0/24",
                    "delegations": [],
                    "privateEndpointNetworkPolicies": "Enabled",
                    "privateLinkServiceNetworkPolicies": "Enabled"
                    }
                    },
                    {
                    "name": "GatewaySubnet",
                    "etag": "W/\"d9f6e6d6-2c15-4f7c-b01f-bed40f748dea\"",
                    "properties": {
                    "provisioningState": "Succeeded",
                    "addressPrefix": "10.0.1.0/29",
                    "serviceEndpoints": [],
                    "delegations": [],
                    "privateEndpointNetworkPolicies": "Enabled",
                    "privateLinkServiceNetworkPolicies": "Enabled"
                    }
                    }

                ]
        ```

         В файле **template.js** для изменения префикса адреса его необходимо изменить в двух местах: в разделе, приведенном выше, и в разделе **Type** , приведенном ниже.  Измените свойство **addressPrefix** , чтобы оно совпадало с указанным выше:

        ```json
         "type": "Microsoft.Network/virtualNetworks/subnets",
           "apiVersion": "2019-06-01",
           "name": "[concat(parameters('virtualNetworks_myVNET1_name'), '/GatewaySubnet')]",
              "dependsOn": [
                 "[resourceId('Microsoft.Network/virtualNetworks', parameters('virtualNetworks_myVNET1_name'))]"
                   ],
              "properties": {
                 "provisioningState": "Succeeded",
                 "addressPrefix": "10.0.1.0/29",
                 "serviceEndpoints": [],
                 "delegations": [],
                 "privateEndpointNetworkPolicies": "Enabled",
                 "privateLinkServiceNetworkPolicies": "Enabled"
                  }
                 },
                  {
                  "type": "Microsoft.Network/virtualNetworks/subnets",
                  "apiVersion": "2019-06-01",
                  "name": "[concat(parameters('virtualNetworks_myVNET1_name'), '/subnet-1')]",
                     "dependsOn": [
                        "[resourceId('Microsoft.Network/virtualNetworks', parameters('virtualNetworks_myVNET1_name'))]"
                          ],
                     "properties": {
                        "provisioningState": "Succeeded",
                        "addressPrefix": "10.0.0.0/24",
                        "delegations": [],
                        "privateEndpointNetworkPolicies": "Enabled",
                        "privateLinkServiceNetworkPolicies": "Enabled"
                         }
                  }
         ]
        ```

13. Нажмите кнопку **сохранить** в интерактивном редакторе.

14. Щелкните **основные**  >  **подписки** , чтобы выбрать подписку, в которой будет развернута Целевая виртуальная сеть.

15. Щелкните **основные**  >  **ресурсы группа ресурсов** , чтобы выбрать группу ресурсов, в которой будет развернута Целевая виртуальная сеть.  Можно щелкнуть **создать** , чтобы создать новую группу ресурсов для целевой виртуальной сети.  Убедитесь, что имя не совпадает с исходной группой ресурсов существующей виртуальной сети.

16. Убедитесь, что в качестве **основы**  >  **расположения** задано целевое расположение, в котором требуется развернуть виртуальную сеть.

17. Проверьте в разделе **Параметры** , имя которых совпадает с именем, введенным в редакторе параметров выше.

18. Установите флажок в разделе **условия**.

19. Нажмите кнопку **приобрести** , чтобы развернуть целевую виртуальную сеть.

### <a name="export-the-internal-load-balancer-template-and-deploy-from-azure-powershell"></a>Экспорт шаблона внутренней подсистемы балансировки нагрузки и развертывание из Azure PowerShell

1. Войдите в [портал Azure](https://portal.azure.com)  >  **группы ресурсов**.
2. Выберите группу ресурсов, содержащую внутреннюю подсистему балансировки нагрузки источника, и щелкните ее.
3. Выберите **Параметры**>  >  **Экспорт шаблона**.
4. Выберите **развернуть** в колонке **Экспорт шаблона** .
5. Щелкните **шаблон**  >  **изменить параметры** , чтобы открыть **parameters.jsв** файле в интерактивном редакторе.

6. Чтобы изменить параметр имени внутренней подсистемы балансировки нагрузки, измените значение **DefaultValue** для свойства "имя внутренней подсистемы балансировки нагрузки" на имя целевой внутренней подсистемы балансировки нагрузки и убедитесь, что имя находится в кавычках:

    ```json
         "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
         "contentVersion": "1.0.0.0",
         "parameters": {
            "loadBalancers_myLoadBalancer_name": {
            "defaultValue": "<target-internal-lb-name>",
            "type": "String"
             },
            "virtualNetworks_myVNET2_internalid": {
             "defaultValue": "<target-vnet-resource-ID>",
             "type": "String"
             }
    ```

6. Чтобы изменить целевую виртуальную сеть, которая была перемещена выше, сначала необходимо получить идентификатор ресурса, а затем скопировать и вставить его в **parameters.jsв** файле. Чтобы получить идентификатор:

    1. Войдите в [портал Azure](https://portal.azure.com)  >  **группы ресурсов** на другой вкладке или в окне браузера.
    2. Выберите целевую группу ресурсов, содержащую перемещенную виртуальную сеть, из описанных выше действий и щелкните ее.
    3. Выберите **Параметры**>  >  **Свойства**.
    4. В колонке справа выделите **идентификатор ресурса** и скопируйте его в буфер обмена.  Кроме того, можно щелкнуть кнопку **Копировать в буфер обмена** справа от параметра путь к **идентификатору ресурса** .
    5. Вставьте идентификатор ресурса в свойство **DefaultValue** в редакторе **изменение параметров** , открываемом в другом окне браузера или на вкладке:

        ```json
         "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
         "contentVersion": "1.0.0.0",
         "parameters": {
            "loadBalancers_myLoadBalancer_name": {
            "defaultValue": "<target-internal-lb-name>",
            "type": "String"
             },
            "virtualNetworks_myVNET2_internalid": {
             "defaultValue": "<target-vnet-resource-ID>",
             "type": "String"
             }
        ```
    6. Нажмите кнопку **сохранить** в интерактивном редакторе.

7. Щелкните **шаблон**  >  **изменить шаблон** , чтобы открыть **template.jsв** файле в интерактивном редакторе.
8. Чтобы изменить целевой регион, в который будет перемещена конфигурация внутренней подсистемы балансировки нагрузки, измените свойство **Location** в разделе **ресурсы** в **template.js** файл:

    ```json
        "resources": [
            {
                "type": "Microsoft.Network/loadBalancers",
                "apiVersion": "2019-06-01",
                "name": "[parameters('loadBalancers_myLoadBalancer_name')]",
                "location": "<target-internal-lb-region>",
                "sku": {
                    "name": "Standard",
                    "tier": "Regional"
                },
    ```

9.  Чтобы получить коды расположения регионов, см. раздел [расположения Azure](https://azure.microsoft.com/global-infrastructure/locations/).  Код для региона — это имя региона без пробелов, **Центральная американская**  =  **centralus**.

10. Кроме того, при необходимости можно изменить другие параметры в шаблоне:

    * **SKU** . номер SKU внутренней подсистемы балансировки нагрузки можно изменить в конфигурации с Standard на Basic или Basic на Standard, изменив   >  свойство **имя** SKU в **template.jsв** файле:

        ```json
        "resources": [
        {
            "type": "Microsoft.Network/loadBalancers",
            "apiVersion": "2019-06-01",
            "name": "[parameters('loadBalancers_myLoadBalancer_name')]",
            "location": "<target-internal-lb-region>",
            "sku": {
                "name": "Standard",
                "tier": "Regional"
            },
        ```
      Дополнительные сведения о различиях между подсистемами балансировки нагрузки уровня "базовый" и "Стандартный" см. в статье [обзор Load Balancer (цен. Категория "Стандартный") Azure](./load-balancer-overview.md) .

    * **Правила балансировки нагрузки** . в конфигурацию можно добавить или удалить правила балансировки нагрузки, добавив или удалив записи в разделе **лоадбаланЦингрулес** **template.jsв** файле:

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
       Дополнительные сведения о правилах балансировки нагрузки см [. в разделе что такое Azure Load Balancer?](./load-balancer-overview.md)

    * **Пробы** . Вы можете добавить или удалить пробу для балансировщика нагрузки в конфигурации, добавив или удалив записи в разделе **зонды** **template.jsв** файле:

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
       Дополнительные сведения о Azure Load Balancer зондах работоспособности см. в разделе [проверки работоспособности Load Balancer](./load-balancer-custom-probe-overview.md) .

    * **Правила NAT для входящего трафика** . Вы можете добавить или удалить правила NAT для входящего трафика для подсистемы балансировки нагрузки, добавив или удалив записи в разделе **inboundNatRules** **template.jsв** файле:

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
        Чтобы завершить добавление или удаление правила NAT для входящего трафика, оно должно присутствовать или удалено в качестве свойства **типа** в конце **template.jsв** файле:

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
        Дополнительные сведения о правилах NAT для входящего трафика см. в разделе [что такое Azure Load Balancer?](./load-balancer-overview.md)

12. Нажмите кнопку **сохранить** в интерактивном редакторе.

13. Щелкните **основные**  >  **подписки** , чтобы выбрать подписку, в которой будет развернута Целевая внутренняя подсистема балансировки нагрузки.

15. Щелкните **основные**  >  **ресурсы группа ресурсов** , чтобы выбрать группу ресурсов, в которой будет развернута Целевая подсистема балансировки нагрузки.  Можно щелкнуть **создать** , чтобы создать новую группу ресурсов для целевого внутреннего балансировщика нагрузки, или выбрать существующую группу ресурсов, созданную ранее для виртуальной сети.  Убедитесь, что имя не совпадает с исходной группой ресурсов существующего источника внутренняя подсистема балансировки нагрузки.

16. Убедитесь, что в качестве **основы**  >  **расположения** выбрано целевое расположение, в котором требуется развернуть внутреннюю подсистему балансировки нагрузки.

17. Проверьте в разделе **Параметры** , имя которых совпадает с именем, введенным в редакторе параметров выше.  Убедитесь, что идентификаторы ресурсов заполнены для всех виртуальных сетей в конфигурации.

18. Установите флажок в разделе **условия**.

19. Нажмите кнопку **приобрести** , чтобы развернуть целевую виртуальную сеть.

## <a name="discard"></a>Игнорировать

Если вы хотите отменить целевую виртуальную сеть и внутреннюю подсистему балансировки нагрузки, удалите группу ресурсов, содержащую целевую виртуальную сеть и внутреннюю подсистему балансировки нагрузки.  Для этого выберите группу ресурсов на панели мониторинга на портале и щелкните **Удалить** в верхней части страницы обзора.

## <a name="clean-up"></a>Очистка

Чтобы зафиксировать изменения и завершить перемещение виртуальной сети и внутренней подсистемы балансировки нагрузки, удалите исходную виртуальную сеть и внутреннюю подсистему балансировки нагрузки или группу ресурсов. Для этого выберите виртуальную сеть, внутреннюю подсистему балансировки нагрузки или группу ресурсов на панели мониторинга на портале и щелкните **Удалить** в верхней части каждой страницы.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы переместили внутренний балансировщик нагрузки Azure из одного региона в другой и очистили исходные ресурсы.  Дополнительные сведения о перемещении ресурсов между регионами и аварийном восстановлении в Azure см. по следующей ссылке:


- [Перемещение ресурсов в новую группу ресурсов или подписку](../azure-resource-manager/management/move-resource-group-and-subscription.md)
- [Перенос виртуальных машин Azure в другой регион](../site-recovery/azure-to-azure-tutorial-migrate.md)