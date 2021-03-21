---
title: Маршрутизация веб-трафика на основе URL-адресов с помощью Azure CLI
description: В этой статье вы узнаете, как маршрутизировать веб-трафик на основе URL-адреса в определенные масштабируемые пулы серверов с помощью Azure CLI.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 08/01/2019
ms.author: victorh
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 8e8fed99fe0b1de52d2e2d0018dfd8867b54b63b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94566526"
---
# <a name="route-web-traffic-based-on-the-url-using-the-azure-cli"></a>Маршрутизация веб-трафика на основе URL-адресов с помощью Azure CLI

Как ИТ-администратор, управляющий веб-трафиком, вы хотите предоставить своим клиентам или пользователям возможность получения необходимой информации как можно быстрее. Одним из способов оптимизации их условий работы является маршрутизация различных видов веб-трафика к другим ресурсам сервера. В этой статье показано, как с помощью Azure CLI установить и настроить маршрутизацию в Шлюзе приложений для различных типов трафика приложения. После этого трафик будет направляться в другие пулы серверов в зависимости от URL-адреса.

![Пример маршрутизации для URL-адресов](./media/tutorial-url-route-cli/scenario.png)

Вы узнаете, как выполнять следующие задачи:

* Создание группы ресурсов для необходимых сетевых ресурсов.
* Создание сетевых ресурсов.
* Создание шлюза приложений для трафика, поступающего из приложения.
* Указание пулов серверов и правил маршрутизации для различных типов трафика.
* Создание масштабируемого набора для каждого пула, чтобы пул мог автоматически масштабироваться.
* Выполнение теста, чтобы проверить, что разные типы трафика направляются в необходимый пул.

Вы также можете выполнить инструкции из этой процедуры, используя [Azure PowerShell](tutorial-url-route-powershell.md) или [портал Azure](create-url-route-portal.md).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

 - Для работы с этим учебником требуется Azure CLI версии 2.0.4 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Группа ресурсов — это логический контейнер, в котором выполняется развертывание ресурсов Azure и управление ими. Создайте группу ресурсов с помощью команды `az group create`.

В следующем примере создается группа ресурсов с именем *myResourceGroupAG* в расположении *eastus*.

```azurecli-interactive
az group create --name myResourceGroupAG --location eastus
```

## <a name="create-network-resources"></a>Создание сетевых ресурсов

Создайте виртуальную сеть с именем *myVNet* и подсеть *myAGSubnet* с помощью команды `az network vnet create`. Затем добавьте подсеть с именем *myBackendSubnet*, необходимую для внутренних серверов, используя команду `az network vnet subnet create`. Создайте общедоступный IP-адрес *myAGPublicIPAddress*, используя команду `az network public-ip create`.

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

## <a name="create-the-app-gateway-with-a-url-map"></a>Создание шлюза приложений путем сопоставления URL-адресов

Используйте команду `az network application-gateway create` для создания шлюза приложений с именем *myAppGateway*. При создании шлюза приложений с помощью Azure CLI укажите такие сведения о конфигурации, как емкость, номер SKU и параметры HTTP. Шлюз приложений назначен на адреса *myAGSubnet* и *myAGPublicIPAddress*.

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

 Создание шлюза приложений может занять несколько минут. Когда шлюз приложений будет создан, вы увидите такие новые функции:


|Компонент  |Описание  |
|---------|---------|
|appGatewayBackendPool     |Для шлюза приложений необходимо наличие по крайней мере одного внутреннего пула адресов.|
|appGatewayBackendHttpSettings     |Указывает, что для обмена данными используются порт 80 и протокол HTTP.|
|appGatewayHttpListener     |Прослушиватель по умолчанию, связанный с appGatewayBackendPool.|
|appGatewayFrontendIP     |Назначает адрес myAGPublicIPAddress для прослушивателя appGatewayHttpListener.|
|rule1     |Правило маршрутизации по умолчанию, связанное с прослушивателем appGatewayHttpListener.|

### <a name="add-image-and-video-backend-pools-and-a-port"></a>Добавление внутреннего порта и внутренних пулов для изображений и видео

Добавьте внутренние пулы с именами *imagesBackendPool* и *videoBackendPool* в шлюз приложений с помощью команды `az network application-gateway address-pool create`. Добавьте серверный порт для пулов, используя команду `az network application-gateway frontend-port create`.

```azurecli-interactive
az network application-gateway address-pool create \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --name imagesBackendPool

az network application-gateway address-pool create \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --name videoBackendPool

az network application-gateway frontend-port create \
  --port 8080 \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --name port8080
```

### <a name="add-a-backend-listener"></a>Добавление серверного прослушивателя

Добавьте серверный прослушиватель с именем *backendListener*, необходимый для маршрутизации трафика, используя команду `az network application-gateway http-listener create`.


```azurecli-interactive
az network application-gateway http-listener create \
  --name backendListener \
  --frontend-ip appGatewayFrontendIP \
  --frontend-port port8080 \
  --resource-group myResourceGroupAG \
  --gateway-name myAppGateway
```

### <a name="add-a-url-path-map"></a>Добавление сопоставления URL-путей

Сопоставления URL-путей гарантируют, что трафик с определенных URL-адресов маршрутизируется в определенные внутренние пулы. Создайте сопоставление URL-путей с именем *imagePathRule* и *videoPathRule*, используя команды `az network application-gateway url-path-map create` и `az network application-gateway url-path-map rule create`.

```azurecli-interactive
az network application-gateway url-path-map create \
  --gateway-name myAppGateway \
  --name myPathMap \
  --paths /images/* \
  --resource-group myResourceGroupAG \
  --address-pool imagesBackendPool \
  --default-address-pool appGatewayBackendPool \
  --default-http-settings appGatewayBackendHttpSettings \
  --http-settings appGatewayBackendHttpSettings \
  --rule-name imagePathRule

az network application-gateway url-path-map rule create \
  --gateway-name myAppGateway \
  --name videoPathRule \
  --resource-group myResourceGroupAG \
  --path-map-name myPathMap \
  --paths /video/* \
  --address-pool videoBackendPool
```

### <a name="add-a-routing-rule"></a>Добавление правила маршрутизации

Правило маршрутизации связывает сопоставления URL-адресов с созданным прослушивателем. Добавьте правило с именем *rule2*, используя команду `az network application-gateway rule create`.

```azurecli-interactive
az network application-gateway rule create \
  --gateway-name myAppGateway \
  --name rule2 \
  --resource-group myResourceGroupAG \
  --http-listener backendListener \
  --rule-type PathBasedRouting \
  --url-path-map myPathMap \
  --address-pool appGatewayBackendPool
```

## <a name="create-virtual-machine-scale-sets"></a>Создание масштабируемых наборов виртуальных машин

В рамках этой статьи вы создадите три масштабируемых набора виртуальных машин, которые поддерживают три созданных внутренних пула. Вы создадите масштабируемые наборы с именами *myvmss1*, *myvmss2* и *myvmss3*. Каждый масштабируемый набор содержит два экземпляра виртуальной машины, на которых устанавливаются службы NGINX.

```azurecli-interactive
for i in `seq 1 3`; do

  if [ $i -eq 1 ]
  then
    poolName="appGatewayBackendPool" 
  fi

  if [ $i -eq 2 ]
  then
    poolName="imagesBackendPool"
  fi

  if [ $i -eq 3 ]
  then
    poolName="videoBackendPool"
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
for i in `seq 1 3`; do
  az vmss extension set \
    --publisher Microsoft.Azure.Extensions \
    --version 2.0 \
    --name CustomScript \
    --resource-group myResourceGroupAG \
    --vmss-name myvmss$i \
    --settings '{ "fileUris": ["https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/application-gateway/iis/install_nginx.sh"], "commandToExecute": "./install_nginx.sh" }'
done
```

## <a name="test-the-application-gateway"></a>Тестирование шлюза приложений

Чтобы получить общедоступный IP-адрес шлюза приложений, используйте команду az network public-ip show. Скопируйте общедоступный IP-адрес и вставьте его в адресную строку браузера. Например, `http://40.121.222.19`, `http://40.121.222.19:8080/images/test.htm` или `http://40.121.222.19:8080/video/test.htm`.

```azurecli-interactive
az network public-ip show \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress \
  --query [ipAddress] \
  --output tsv
```

![Тестирование базового URL-адреса в шлюзе приложений](./media/tutorial-url-route-cli/application-gateway-nginx.png)

Измените URL-адрес на http://&lt;ip-address&gt;:8080/images/test.html, указав вместо заполнителя &lt;ip-address&gt; фактический IP-адрес. Результат должен быть примерно таким:

![Тестирование URL-адреса изображений в шлюзе приложений](./media/tutorial-url-route-cli/application-gateway-nginx-images.png)

Измените URL-адрес на http:// &lt; IP-address &gt; : 8080/видео/test.html, заменив IP &lt; -адрес IP &gt; , и вы увидите нечто вроде следующего.

![Тестирование URL-адреса видео в шлюзе приложений](./media/tutorial-url-route-cli/application-gateway-nginx-video.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

При необходимости вы можете удалить группу ресурсов, шлюз приложений и все связанные ресурсы.

```azurecli-interactive
az group delete --name myResourceGroupAG
```

## <a name="next-steps"></a>Дальнейшие действия

[Создание шлюза приложений с перенаправлением на основе URL-пути](./tutorial-url-redirect-cli.md)
