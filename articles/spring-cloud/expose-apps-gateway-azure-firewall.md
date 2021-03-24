---
title: Предоставление доступа к приложениям через Интернет с помощью шлюза приложений и брандмауэра Azure
description: Как предоставить приложениям доступ к Интернету с помощью шлюза приложений и брандмауэра Azure
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: how-to
ms.date: 11/17/2020
ms.custom: devx-track-java
ms.openlocfilehash: 6c22d1bae4f1d116aa52846880498c7c2a425174
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104878583"
---
# <a name="expose-applications-to-the-internet-using-application-gateway-and-azure-firewall"></a>Предоставление доступа к приложениям через Интернет с помощью шлюза приложений и брандмауэра Azure

В этом документе объясняется, как предоставить приложениям доступ к Интернету с помощью шлюза приложений и брандмауэра Azure. При развертывании экземпляра облачной службы Azure весны в виртуальной сети приложения в экземпляре службы доступны только в частной сети. Чтобы сделать приложения доступными в Интернете, необходимо интегрировать его со **шлюзом приложений Azure** и, при необходимости, с **брандмауэром Azure**.

## <a name="prerequisites"></a>Предварительные требования

- [Azure CLI версии 2.0.4 или более поздней](/cli/azure/install-azure-cli).

## <a name="define-variables"></a>Определение переменных

Определите переменные для группы ресурсов и виртуальной сети, которые вы создали в соответствии с инструкциями по [развертыванию Azure веснного облака в виртуальной сети Azure (внедрение в виртуальную сеть)](spring-cloud-tutorial-deploy-in-azure-virtual-network.md). Настройте значения на основе реальной среды.

```
SUBSCRIPTION='subscription-id'
RESOURCE_GROUP='my-resource-group'
LOCATION='eastus'
SPRING_APP_PRIVATE_FQDN='my-azure-spring-cloud-hello-vnet.private.azuremicroservices.io'
VIRTUAL_NETWORK_NAME='azure-spring-cloud-vnet'
APPLICATION_GATEWAY_SUBNET_NAME='app-gw-subnet'
APPLICATION_GATEWAY_SUBNET_CIDR='10.1.2.0/24'
```

## <a name="login-to-azure"></a>Вход в Azure

Войдите в Azure CLI и выберите активную подписку.

```
az login
az account set --subscription ${SUBSCRIPTION}
```

## <a name="create-network-resources"></a>Создание сетевых ресурсов

Создаваемый **шлюз приложений Azure** будет присоединяться к той же виртуальной сети, что и-или одноранговая виртуальная сеть, в качестве экземпляра облачной службы Azure весны. Сначала создайте новую подсеть для шлюза приложений в виртуальной сети с помощью `az network vnet subnet create` , а также создайте общедоступный IP-адрес в качестве внешнего интерфейса шлюза приложений с помощью `az network public-ip create` .

```
APPLICATION_GATEWAY_PUBLIC_IP_NAME='app-gw-public-ip'
az network vnet subnet create \
    --name ${APPLICATION_GATEWAY_SUBNET_NAME} \
    --resource-group ${RESOURCE_GROUP} \
    --vnet-name ${VIRTUAL_NETWORK_NAME} \
    --address-prefix ${APPLICATION_GATEWAY_SUBNET_CIDR}
az network public-ip create \
    --resource-group ${RESOURCE_GROUP} \
    --location ${LOCATION} \
    --name ${APPLICATION_GATEWAY_PUBLIC_IP_NAME} \
    --allocation-method Static \
    --sku Standard
```

## <a name="create-application-gateway"></a>Создание шлюза приложений

Создайте шлюз приложений с помощью `az network application-gateway create` и укажите частное полное доменное имя (FQDN) приложения в качестве серверов во внутреннем пуле. Затем обновите параметр HTTP, используя `az network application-gateway http-settings update` , чтобы использовать имя узла из серверного пула.

```
APPLICATION_GATEWAY_NAME='my-app-gw'
az network application-gateway create \
    --name ${APPLICATION_GATEWAY_NAME} \
    --resource-group ${RESOURCE_GROUP} \
    --location ${LOCATION} \
    --capacity 2 \
    --sku Standard_v2 \
    --http-settings-cookie-based-affinity Enabled \
    --http-settings-port 443 \
    --http-settings-protocol Https \
    --public-ip-address ${APPLICATION_GATEWAY_PUBLIC_IP_NAME} \
    --vnet-name ${VIRTUAL_NETWORK_NAME} \
    --subnet ${APPLICATION_GATEWAY_SUBNET_NAME} \
    --servers ${SPRING_APP_PRIVATE_FQDN}
az network application-gateway http-settings update \
    --gateway-name ${APPLICATION_GATEWAY_NAME} \
    --resource-group ${RESOURCE_GROUP} \
    --name appGatewayBackendHttpSettings \
    --host-name-from-backend-pool true
```

Чтобы создать шлюз приложения в Azure, может понадобиться до 30 минут. После создания проверьте работоспособность серверной части с помощью `az network application-gateway show-backend-health` .  Будет рассмотрено, достиг ли шлюз приложений вашего приложения через его частное полное доменное имя.

```
az network application-gateway show-backend-health \
    --name ${APPLICATION_GATEWAY_NAME} \
    --resource-group ${RESOURCE_GROUP}
```

Выходные данные показывают состояние работоспособности серверного пула.

```
{
  "backendAddressPools": [
    {
      "backendHttpSettingsCollection": [
        {
          "servers": [
            {
              "address": "my-azure-spring-cloud-hello-vnet.private.azuremicroservices.io",
              "health": "Healthy",
              "healthProbeLog": "Success. Received 200 status code",
              "ipConfiguration": null
            }
          ]
        }
      ]
    }
  ]
}
```

## <a name="access-your-application-using-the-frontend-public-ip-of-the-application-gateway"></a>Доступ к приложению с помощью общедоступного IP-адреса шлюза приложений

Получите общедоступный IP-адрес шлюза приложений с помощью `az network public-ip show` .

```
az network public-ip show \
    --resource-group ${RESOURCE_GROUP} \
    --name ${APPLICATION_GATEWAY_PUBLIC_IP_NAME} \
    --query [ipAddress] \
    --output tsv
```

Скопируйте и вставьте общедоступный IP-адрес в адресную строку браузера.

  ![Приложение в общедоступном IP-адресе](media/spring-cloud-expose-apps-gateway-az-firewall/app-gateway-public-ip.png)

## <a name="see-also"></a>См. также раздел

- [Устранение неполадок Azure Spring Cloud в виртуальной сети](spring-cloud-troubleshooting-vnet.md)
- [Обязательства клиентов в отношении запуска Azure Spring Cloud в виртуальной сети](spring-cloud-vnet-customer-responsibilities.md)