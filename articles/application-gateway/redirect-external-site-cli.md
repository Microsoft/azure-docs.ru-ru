---
title: Перенаправление внешнего трафика с помощью интерфейса командной строки (шлюз приложений Azure)
description: Узнайте, как создать шлюз приложений, который перенаправляет внешний веб-трафик в соответствующий пул, используя Azure CLI.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 09/24/2020
ms.author: victorh
ms.openlocfilehash: 838c2dc887790bb12b390261d94748595232d8b3
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94565863"
---
# <a name="create-an-application-gateway-with-external-redirection-using-the-azure-cli"></a>Создание шлюза приложений с перенаправлением внешнего трафика с помощью Azure CLI

С помощью Azure CLI можно настроить [перенаправление веб-трафика](multiple-site-overview.md) при создании [шлюза приложений](overview.md). В этом руководстве вы настроите прослушиватель и правило, которое перенаправляет веб-трафик, поступающий на шлюз приложений, на внешний сайт.

Вы узнаете, как выполнять следующие задачи:

* Настройка сети
* создание прослушивателя и правила перенаправления;
* Создание шлюза приложений

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

 - Для работы с этим учебником требуется Azure CLI версии 2.0.4 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Группа ресурсов — это логический контейнер, в котором происходит развертывание ресурсов Azure и управление ими. Создайте группу ресурсов, используя команду [az group create](/cli/azure/group).

В следующем примере создается группа ресурсов с именем *myResourceGroupAG* в расположении *eastus*.

```azurecli-interactive 
az group create --name myResourceGroupAG --location eastus
```

## <a name="create-network-resources"></a>Создание сетевых ресурсов 

Создайте виртуальную сеть с именем *myVNet* и подсеть *myAGSubnet* с помощью команды [az network vnet create](/cli/azure/network/vnet). Создайте общедоступный IP-адрес с именем *myAGPublicIPAddress*, используя команду [az network public-ip create](/cli/azure/network/public-ip). Эти ресурсы используются для обеспечения сетевого подключения к шлюзу приложений и связанным с ним ресурсам.

```azurecli-interactive
az network vnet create \
  --name myVNet \
  --resource-group myResourceGroupAG \
  --location eastus \
  --address-prefix 10.0.0.0/16 \
  --subnet-name myAGSubnet \
  --subnet-prefix 10.0.1.0/24
az network public-ip create \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress
```

## <a name="create-an-application-gateway"></a>Создание шлюза приложений

Выполните команду [az network application-gateway create](/cli/azure/network/application-gateway), чтобы создать шлюз приложений *myAppGateway*. При создании шлюза приложений с помощью Azure CLI укажите такие сведения о конфигурации, как емкость, номер SKU и параметры HTTP. Шлюз приложений назначается подсети *myAGSubnet* и адресу *myPublicIPAddressddress*, созданным ранее. 

```azurecli-interactive
az network application-gateway create \
  --name myAppGateway \
  --location eastus \
  --resource-group myResourceGroupAG \
  --vnet-name myVNet \
  --subnet myAGsubnet \
  --capacity 2 \
  --sku Standard_Medium \
  --http-settings-cookie-based-affinity Disabled \
  --frontend-port 8080 \
  --http-settings-port 80 \
  --http-settings-protocol Http \
  --public-ip-address myAGPublicIPAddress
```

Создание шлюза приложений может занять несколько минут. Когда шлюз приложений будет создан, вы увидите такие новые функции шлюза:

- *appGatewayBackendPool* — шлюз приложений должен иметь по крайней мере один внутренний пул адресов.
- *appGatewayBackendHttpSettings* — указывает, что для обмена данными используются порт 80 и протокол HTTP.
- *appGatewayHttpListener* — прослушиватель по умолчанию, связанный с *appGatewayBackendPool*.
- *appGatewayFrontendIP* — назначает адрес *myAGPublicIPAddress* для прослушивателя *appGatewayHttpListener*.
- *rule1* — правило маршрутизации по умолчанию, связанное с прослушивателем *appGatewayHttpListener*.

### <a name="add-the-redirection-configuration"></a>Добавление конфигурации перенаправления

Добавьте конфигурацию перенаправления, которая отправляет трафик из *www \. consoto.org* в прослушиватель для *www \. contoso.com* в шлюз приложений с помощью команды [AZ Network Application-Gateway Redirect-config Create](/cli/azure/network/application-gateway/redirect-config).

```azurecli-interactive
az network application-gateway redirect-config create \
  --name myredirect \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --type Temporary \
  --target-url "https://bing.com"
```

### <a name="add-a-listener-and-routing-rule"></a>Добавление прослушивателя и правила маршрутизации

Прослушиватель требуется для того, чтобы шлюз приложений правильно маршрутизировал трафик. Создайте прослушиватель, используя команду [az network application-gateway http-listener create](/cli/azure/network/application-gateway), с интерфейсным портом. Чтобы создать порт, используйте команду [az network application-gateway frontend-port create](/cli/azure/network/application-gateway). Правило требуется для того, чтобы указать прослушивателю, куда отправлять входящий трафик. Создайте базовое правило с именем *redirectRule*, используя команду [az network application-gateway rule create](/cli/azure/network/application-gateway).

```azurecli-interactive
az network application-gateway frontend-port create \
  --port 80 \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --name redirectPort
az network application-gateway http-listener create \
  --name redirectListener \
  --frontend-ip appGatewayFrontendIP \
  --frontend-port redirectPort \
  --resource-group myResourceGroupAG \
  --gateway-name myAppGateway
az network application-gateway rule create \
  --gateway-name myAppGateway \
  --name redirectRule \
  --resource-group myResourceGroupAG \
  --http-listener redirectListener \
  --rule-type Basic \
  --redirect-config myredirect
```

## <a name="test-the-application-gateway"></a>Тестирование шлюза приложений

Чтобы получить общедоступный IP-адрес шлюза приложений, используйте команду [az network public-ip show](/cli/azure/network/public-ip). Скопируйте общедоступный IP-адрес и вставьте его в адресную строку браузера.

В браузере должен открыться сайт *bing.com*.

## <a name="next-steps"></a>Дальнейшие действия

- [Создание шлюза приложений с перенаправлением внутреннего трафика при помощи Azure CLI](redirect-internal-site-cli.md)
