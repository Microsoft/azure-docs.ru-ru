---
title: Перенаправление HTTP в HTTPS с помощью интерфейса командной строки
titleSuffix: Azure Application Gateway
description: Узнайте, как создать перенаправление HTTP-HTTPS и добавить сертификат для завершения TLS с помощью Azure CLI.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 09/24/2020
ms.author: victorh
ms.openlocfilehash: 0d56a1c46f251307755416ef44991ac6f809f330
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94566747"
---
# <a name="create-an-application-gateway-with-http-to-https-redirection-using-the-azure-cli"></a>Создание шлюза приложений с сертификатом с помощью Azure CLI

Вы можете использовать Azure CLI для создания [шлюза приложений](overview.md) с сертификатом для завершения TLS/SSL. Правило маршрутизации используется для перенаправления трафика HTTP в HTTPS-порт в шлюзе приложений. В этом примере также создается [масштабируемый набор виртуальных машин](../virtual-machine-scale-sets/overview.md) с двумя экземплярами виртуальных машин, предназначенный для внутреннего пула шлюза приложений.

Вы узнаете, как выполнять следующие задачи:

* Создание самозаверяющего сертификата
* настройка сети;
* создание шлюза приложений с сертификатом;
* добавление прослушивателя и правила перенаправления;
* создание масштабируемого набора виртуальных машин с серверным пулом, используемым по умолчанию.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

 - Для работы с этим учебником требуется Azure CLI версии 2.0.4 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-a-self-signed-certificate"></a>Создание самозаверяющего сертификата

Для использования в рабочей среде следует импортировать действительный сертификат, подписанный доверенным поставщиком. В этом руководстве для создания самозаверяющего сертификата и PFX-файла используется команда openssl.

```console
openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout privateKey.key -out appgwcert.crt
```

Введите значения, которые соответствуют вашему сертификату. Вы можете принять значения по умолчанию.

```console
openssl pkcs12 -export -out appgwcert.pfx -inkey privateKey.key -in appgwcert.crt
```

Введите пароль для сертификата. *Azure123456!* — пароль, который используется в этом примере.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Группа ресурсов — это логический контейнер, в котором происходит развертывание ресурсов Azure и управление ими. Создайте группу ресурсов, используя команду [az group create](/cli/azure/group).

В следующем примере создается группа ресурсов с именем *myResourceGroupAG* в расположении *eastus*.

```azurecli-interactive 
az group create --name myResourceGroupAG --location eastus
```

## <a name="create-network-resources"></a>Создание сетевых ресурсов

Создайте виртуальную сеть с именем *myVNet* и подсеть *myAGSubnet* с помощью команды [az network vnet create](/cli/azure/network/vnet). Затем добавьте подсеть с именем *myBackendSubnet*, необходимую для внутренних серверов, используя команду [az network vnet subnet create](/cli/azure/network/vnet/subnet). Создайте общедоступный IP-адрес с именем *myAGPublicIPAddress*, используя команду [az network public-ip create](/cli/azure/network/public-ip).

```azurecli-interactive
az network vnet create \
  --name myVNet \
  --resource-group myResourceGroupAG \
  --location eastus \
  --address-prefix 10.0.0.0/16 \
  --subnet-name myAGSubnet \
  --subnet-prefix 10.0.1.0/24
az network vnet subnet create \
  --name myBackendSubnet \
  --resource-group myResourceGroupAG \
  --vnet-name myVNet \
  --address-prefix 10.0.2.0/24
az network public-ip create \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress
```

## <a name="create-the-application-gateway"></a>Создание шлюза приложений

Выполните команду [az network application-gateway create](/cli/azure/network/application-gateway#az-network-application-gateway-create), чтобы создать шлюз приложений *myAppGateway*. При создании шлюза приложений с помощью Azure CLI укажите такие сведения о конфигурации, как емкость, номер SKU и параметры HTTP. 

Шлюз приложений назначается подсети *myAGSubnet* и адресу *myAGPublicIPAddress*, созданным ранее. В этом примере нужно связать созданный сертификат и пароль при создании шлюза приложений. 

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
  --frontend-port 443 \
  --http-settings-port 80 \
  --http-settings-protocol Http \
  --public-ip-address myAGPublicIPAddress \
  --cert-file appgwcert.pfx \
  --cert-password "Azure123456!"

```

 Создание шлюза приложений может занять несколько минут. Когда шлюз приложений будет создан, вы увидите такие новые функции шлюза:

- *appGatewayBackendPool* — шлюз приложений должен иметь по крайней мере один внутренний пул адресов.
- *appGatewayBackendHttpSettings* — указывает, что для обмена данными используются порт 80 и протокол HTTP.
- *appGatewayHttpListener* — прослушиватель по умолчанию, связанный с *appGatewayBackendPool*.
- *appGatewayFrontendIP* — назначает адрес *myAGPublicIPAddress* для прослушивателя *appGatewayHttpListener*.
- *rule1* — правило маршрутизации по умолчанию, связанное с прослушивателем *appGatewayHttpListener*.

## <a name="add-a-listener-and-redirection-rule"></a>добавление прослушивателя и правила перенаправления;

### <a name="add-the-http-port"></a>Добавление HTTP-порта

С помощью команды [az network application-gateway frontend-port create](/cli/azure/network/application-gateway/frontend-port#az-network-application-gateway-frontend-port-create) в шлюз приложений можно добавить HTTP-порт.

```azurecli-interactive
az network application-gateway frontend-port create \
  --port 80 \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --name httpPort
```

### <a name="add-the-http-listener"></a>Добавление прослушивателя HTTP

С помощью команды [az network application-gateway http-listener create](/cli/azure/network/application-gateway/http-listener#az-network-application-gateway-http-listener-create) в шлюз приложений можно добавить прослушиватель *myListener*.

```azurecli-interactive
az network application-gateway http-listener create \
  --name myListener \
  --frontend-ip appGatewayFrontendIP \
  --frontend-port httpPort \
  --resource-group myResourceGroupAG \
  --gateway-name myAppGateway
```

### <a name="add-the-redirection-configuration"></a>Добавление конфигурации перенаправления

Добавьте в шлюз приложений конфигурацию перенаправления HTTP в HTTPS с помощью команды [az network application-gateway redirect-config create](/cli/azure/network/application-gateway/redirect-config#az-network-application-gateway-redirect-config-create).

```azurecli-interactive
az network application-gateway redirect-config create \
  --name httpToHttps \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --type Permanent \
  --target-listener appGatewayHttpListener \
  --include-path true \
  --include-query-string true
```

### <a name="add-the-routing-rule"></a>Добавление правила маршрутизации

Добавьте в шлюз приложений правило маршрутизации *rule2* с помощью команды [az network application-gateway rule create](/cli/azure/network/application-gateway/rule#az-network-application-gateway-rule-create).

```azurecli-interactive
az network application-gateway rule create \
  --gateway-name myAppGateway \
  --name rule2 \
  --resource-group myResourceGroupAG \
  --http-listener myListener \
  --rule-type Basic \
  --redirect-config httpToHttps
```

## <a name="create-a-virtual-machine-scale-set"></a>создавать масштабируемый набор виртуальных машин;

В этом примере вы создаете масштабируемый набор виртуальных машин *myvmss*, чтобы предоставить серверы для внутреннего пула в шлюзе приложений. Виртуальные машины в масштабируемом наборе связаны с подсетью *myBackendSubnet* и пулом *appGatewayBackendPool*. Чтобы создать масштабируемый набор, выполните команду [az vmss create](/cli/azure/vmss#az-vmss-create).

```azurecli-interactive
az vmss create \
  --name myvmss \
  --resource-group myResourceGroupAG \
  --image UbuntuLTS \
  --admin-username azureuser \
  --admin-password Azure123456! \
  --instance-count 2 \
  --vnet-name myVNet \
  --subnet myBackendSubnet \
  --vm-sku Standard_DS2 \
  --upgrade-policy-mode Automatic \
  --app-gateway myAppGateway \
  --backend-pool-name appGatewayBackendPool
```

### <a name="install-nginx"></a>Установка nginx

```azurecli-interactive
az vmss extension set \
  --publisher Microsoft.Azure.Extensions \
  --version 2.0 \
  --name CustomScript \
  --resource-group myResourceGroupAG \
  --vmss-name myvmss \
  --settings '{ "fileUris": ["https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/application-gateway/iis/install_nginx.sh"],
  "commandToExecute": "./install_nginx.sh" }'
```

## <a name="test-the-application-gateway"></a>Тестирование шлюза приложений

Чтобы получить общедоступный IP-адрес шлюза приложений, используйте команду [az network public-ip show](/cli/azure/network/public-ip). Скопируйте общедоступный IP-адрес и вставьте его в адресную строку браузера.

```azurecli-interactive
az network public-ip show \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress \
  --query [ipAddress] \
  --output tsv
```

![Предупреждение системы безопасности](./media/redirect-http-to-https-cli/application-gateway-secure.png)

Чтобы принять предупреждение безопасности, если вы использовали самозаверяющий сертификат, выберите **сведения** , а затем перейдите на **веб-страницу**. На экране отобразится защищенный веб-сайт NGINX, как в показано следующем примере:

![Тестирование базового URL-адреса в шлюзе приложений](./media/redirect-http-to-https-cli/application-gateway-nginxtest.png)

## <a name="next-steps"></a>Дальнейшие действия

- [Создание шлюза приложений с перенаправлением внутреннего трафика при помощи Azure CLI](redirect-internal-site-cli.md)