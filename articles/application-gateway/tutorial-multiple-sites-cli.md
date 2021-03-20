---
title: Размещение нескольких веб-сайтов с помощью интерфейса командной строки
titleSuffix: Azure Application Gateway
description: Узнайте, как создать шлюз приложений, на котором размещено несколько веб-сайтов, с помощью Azure CLI.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/13/2019
ms.author: victorh
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 350962aed89d04c5508e7b2c50e8a838cd5a7174
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94566152"
---
# <a name="create-an-application-gateway-that-hosts-multiple-web-sites-using-the-azure-cli"></a>Создание шлюза приложений, на котором размещено несколько веб-сайтов, с помощью Azure CLI

Чтобы настроить [размещение нескольких веб-сайтов](multiple-site-overview.md) при создании [шлюза приложений](overview.md), можно использовать Azure CLI. В этой статье описано, как определить серверные пулы адресов с помощью масштабируемых наборов виртуальных машин. Затем вы настроите прослушиватели и правила на основе принадлежащих вам доменов, чтобы обеспечить передачу веб-трафика на соответствующие серверы в пулах. В этой статье предполагается, что вы владеете несколькими доменами и в них используются примеры *www \. contoso.com* и *www \. Fabrikam.com*.

Вы узнаете, как выполнять следующие задачи:

* Настройка сети
* Создание шлюза приложений
* Создание серверных прослушивателей
* Создание правил маршрутизации
* создание масштабируемых наборов виртуальных машин с внутренними пулами.
* создание записи CNAME в домене.

:::image type="content" source="./media/tutorial-multiple-sites-cli/scenario.png" alt-text="Многосайтовый шлюз приложений":::

При необходимости эти инструкции можно выполнить с помощью [Azure PowerShell](tutorial-multiple-sites-powershell.md).

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

Создайте виртуальную сеть и подсеть с именем *myAGSubnet* с помощью команды [az network vnet create](/cli/azure/network/vnet). Затем добавьте подсеть, требуемую для внутренних серверов, используя команду [az network vnet subnet create](/cli/azure/network/vnet/subnet). Создайте общедоступный IP-адрес с именем *myAGPublicIPAddress*, используя команду [az network public-ip create](/cli/azure/network/public-ip).

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
  --name myAGPublicIPAddress \
  --allocation-method Static \
  --sku Standard
```

## <a name="create-the-application-gateway"></a>Создание шлюза приложений

Шлюз приложений можно создать с помощью команды [az network application-gateway create](/cli/azure/network/application-gateway#az-network-application-gateway-create). При создании шлюза приложений с помощью Azure CLI укажите такие сведения о конфигурации, как емкость, номер SKU и параметры HTTP. Шлюз приложений назначается подсети *myAGSubnet* и адресу *myAGPublicIPAddress*, созданным ранее. 

```azurecli-interactive
az network application-gateway create \
  --name myAppGateway \
  --location eastus \
  --resource-group myResourceGroupAG \
  --vnet-name myVNet \
  --subnet myAGsubnet \
  --capacity 2 \
  --sku Standard_v2 \
  --http-settings-cookie-based-affinity Disabled \
  --frontend-port 80 \
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

### <a name="add-the-backend-pools"></a>Добавление пулов серверной части

Добавьте серверные пулы, которые необходимы для размещения внутренних серверов, с помощью команды [AZ Network приложение-шлюз адрес — создание пула](/cli/azure/network/application-gateway/address-pool#az-network-application-gateway-address-pool-create) .
```azurecli-interactive
az network application-gateway address-pool create \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --name contosoPool

az network application-gateway address-pool create \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --name fabrikamPool
```

### <a name="add-listeners"></a>Добавление прослушивателей

Добавьте прослушиватели, необходимые для маршрутизации трафика, с помощью команды [AZ Network Application-Gateway HTTP-прослушиватель Create](/cli/azure/network/application-gateway/http-listener#az-network-application-gateway-http-listener-create).

>[!NOTE]
> С помощью шлюза приложений или SKU WAF v2 можно также настроить до 5 имен узлов на прослушиватель, а в имени узла можно использовать подстановочные знаки. Дополнительные сведения см. [в разделе имена узлов с подстановочными знаками в прослушивателе](multiple-site-overview.md#wildcard-host-names-in-listener-preview) .
>Чтобы использовать несколько имен узлов и подстановочных знаков в прослушивателе с помощью Azure CLI, необходимо использовать `--host-names` вместо `--host-name` . При использовании имен узлов можно указать до пяти имен узлов в виде значений, разделенных пробелами. Например: `--host-names "*.contoso.com *.fabrikam.com"`

```azurecli-interactive
az network application-gateway http-listener create \
  --name contosoListener \
  --frontend-ip appGatewayFrontendIP \
  --frontend-port appGatewayFrontendPort \
  --resource-group myResourceGroupAG \
  --gateway-name myAppGateway \
  --host-name www.contoso.com

az network application-gateway http-listener create \
  --name fabrikamListener \
  --frontend-ip appGatewayFrontendIP \
  --frontend-port appGatewayFrontendPort \
  --resource-group myResourceGroupAG \
  --gateway-name myAppGateway \
  --host-name www.fabrikam.com   
  ```

### <a name="add-routing-rules"></a>Добавление правил маршрутизации

Правила обрабатываются в том порядке, в котором они перечислены. Трафик направляется с использованием первого правила, которое соответствует независимым условиям. Например, если для одного порта имеется правило с базовым прослушивателем и правило с многосайтовым прослушивателем, для обеспечения правильной работы правило с многосайтовым прослушивателем должно быть указано перед правилом с базовым прослушивателем. 

В этом примере создаются два новых правила и удаляется правило по умолчанию, созданное при развертывании шлюза приложений. Вы можете добавить правило с помощью команды [az network application-gateway rule create](/cli/azure/network/application-gateway/rule#az-network-application-gateway-rule-create).

```azurecli-interactive
az network application-gateway rule create \
  --gateway-name myAppGateway \
  --name contosoRule \
  --resource-group myResourceGroupAG \
  --http-listener contosoListener \
  --rule-type Basic \
  --address-pool contosoPool

az network application-gateway rule create \
  --gateway-name myAppGateway \
  --name fabrikamRule \
  --resource-group myResourceGroupAG \
  --http-listener fabrikamListener \
  --rule-type Basic \
  --address-pool fabrikamPool

az network application-gateway rule delete \
  --gateway-name myAppGateway \
  --name rule1 \
  --resource-group myResourceGroupAG
```

## <a name="create-virtual-machine-scale-sets"></a>Создание масштабируемых наборов виртуальных машин

В этом примере вы создадите три масштабируемых набора виртуальных машин, которые поддерживают три пула серверной части в шлюзе приложений. Имена создаваемых масштабируемых наборов — *myvmss1*, *myvmss2* и *myvmss3*. Каждый масштабируемый набор содержит два экземпляра виртуальной машины, на которых устанавливаются службы IIS.

```azurecli-interactive
for i in `seq 1 2`; do

  if [ $i -eq 1 ]
  then
    poolName="contosoPool"
  fi

  if [ $i -eq 2 ]
  then
    poolName="fabrikamPool"
  fi

  az vmss create \
    --name myvmss$i \
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
    --backend-pool-name $poolName
done
```

### <a name="install-nginx"></a>Установка nginx

```azurecli-interactive
for i in `seq 1 2`; do

  az vmss extension set \
    --publisher Microsoft.Azure.Extensions \
    --version 2.0 \
    --name CustomScript \
    --resource-group myResourceGroupAG \
    --vmss-name myvmss$i \
    --settings '{ "fileUris": ["https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/application-gateway/iis/install_nginx.sh"],
  "commandToExecute": "./install_nginx.sh" }'

done
```

## <a name="create-a-cname-record-in-your-domain"></a>создание записи CNAME в домене.

После создания шлюза приложений с общедоступным IP-адресом можно получить DNS-адрес и использовать его для создания записи CNAME в своем домене. С помощью команды [az network public-ip show](/cli/azure/network/public-ip#az-network-public-ip-show) можно получить DNS-адрес шлюза приложений. Скопируйте значение *fqdn* для DNSSettings и используйте его в качестве значения создаваемой записи CNAME. 

```azurecli-interactive
az network public-ip show \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress \
  --query [dnsSettings.fqdn] \
  --output tsv
```

Использование записей A не рекомендуется, так как виртуальный IP-адрес может измениться при перезапуске шлюза приложений.

## <a name="test-the-application-gateway"></a>Тестирование шлюза приложений

В адресной строке браузера введите имя домена. Например, http:\//www.contoso.com.

![Проверка сайта contoso в шлюзе приложений](./media/tutorial-multiple-sites-cli/application-gateway-nginxtest1.png)

Введите адрес другого домена. Результат должен быть примерно таким:

![Проверка сайта fabrikam в шлюзе приложений](./media/tutorial-multiple-sites-cli/application-gateway-nginxtest2.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

При необходимости вы можете удалить группу ресурсов, шлюз приложений и все связанные ресурсы.

```azurecli-interactive
az group delete --name myResourceGroupAG
```

## <a name="next-steps"></a>Дальнейшие действия

[Создание шлюза приложений с правилами маршрутизации на основе URL-путей](./tutorial-url-route-cli.md)
