---
title: Использовать частный IP-адрес для внутренней маршрутизации входной конечной точки
description: В этой статье содержатся сведения о том, как использовать частные IP-адреса для внутренней маршрутизации и таким образом предоставлять конечную точку входящих данных в кластере остальной части виртуальной сети.
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/4/2019
ms.author: caya
ms.openlocfilehash: 8be37ed1da0da4da3db43ef4c1cd01ed962f24ed
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93397314"
---
# <a name="use-private-ip-for-internal-routing-for-an-ingress-endpoint"></a>Использовать частный IP-адрес для внутренней маршрутизации для конечной точки входящего трафика 

Эта функция позволяет предоставлять конечную точку входящих данных в `Virtual Network` с помощью частного IP-адреса.

## <a name="pre-requisites"></a>Предварительные требования  
Шлюз приложений с [частной IP-конфигурацией](./configure-application-gateway-with-private-frontend-ip.md)

Существует два способа настройки контроллера для использования частного IP-адреса для входящего трафика,

## <a name="assign-to-a-particular-ingress"></a>Назначить определенному входящему
Чтобы предоставить определенный входящий трафик через частный IP-адрес, используйте аннотацию [`appgw.ingress.kubernetes.io/use-private-ip`](./ingress-controller-annotations.md#use-private-ip) в качестве входных данных.

### <a name="usage"></a>Использование
```yaml
appgw.ingress.kubernetes.io/use-private-ip: "true"
```

Для шлюзов приложений без частного IP-адреса передает с заметками `appgw.ingress.kubernetes.io/use-private-ip: "true"` будет игнорироваться. Это будет указано в журнале входящих событий и АГИК Pod.

* Ошибка, как указано в событии входящего трафика

    ```bash
    Events:
    Type     Reason       Age               From                                                                     Message
    ----     ------       ----              ----                                                                     -------
    Warning  NoPrivateIP  2m (x17 over 2m)  azure/application-gateway, prod-ingress-azure-5c9b6fcd4-bctcb  Ingress default/hello-world-ingress requires Application Gateway 
    applicationgateway3026 has a private IP address
    ```

* Ошибка, как указано в журналах АГИК

    ```bash
    E0730 18:57:37.914749       1 prune.go:65] Ingress default/hello-world-ingress requires Application Gateway applicationgateway3026 has a private IP address
    ```


## <a name="assign-globally"></a>Назначить глобально
В этом случае необходимо ограничить доступ всех передает к частному IP-адресу, используя `appgw.usePrivateIP: true` в `helm` конфигурации.

### <a name="usage"></a>Использование
```yaml
appgw:
    subscriptionId: <subscriptionId>
    resourceGroup: <resourceGroupName>
    name: <applicationGatewayName>
    usePrivateIP: true
```

Это сделает контроллер входящего трафика фильтром IP-адресов для частного IP-адреса при настройке интерфейсных прослушивателей в шлюзе приложений.
АГИК выдаст сбой, если не `usePrivateIP: true` назначен частный IP-адрес.

> [!NOTE]
> Для SKU шлюза приложений версии 2 требуется общедоступный IP-адрес. Если требуется, чтобы шлюз приложений был частным, подключите его [`Network Security Group`](../virtual-network/network-security-groups-overview.md) к подсети шлюза приложений, чтобы ограничить трафик.