---
title: Краткое руководство. Направление веб-трафика с помощью CLI
titleSuffix: Azure Application Gateway
description: В этом кратком руководстве показано, как с помощью Azure CLI создать Шлюз приложений Azure, который направляет веб-трафик к виртуальным машинам в серверном пуле.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: quickstart
ms.date: 01/19/2021
ms.author: victorh
ms.custom: mvc, devx-track-js, devx-track-azurecli
ms.openlocfilehash: 1a691d8ab212dd436b0dc9f7aafbc19a406b12b7
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98601776"
---
# <a name="quickstart-direct-web-traffic-with-azure-application-gateway---azure-cli"></a>Краткое руководство. Направление веб-трафика с помощью Шлюза приложений Azure и Azure CLI.

В этом кратком руководстве рассказывается о том, как создать шлюз приложений с помощью Azure CLI, а затем протестировать его, чтобы убедиться в его правильной работе. 

Шлюз приложений направляет веб-трафик приложений на определенные ресурсы в серверном пуле. Вы назначаете прослушиватели портам, создаете правила и добавляете ресурсы в серверный пул. Для наглядности в этой статье применяются: простая настройка с открытым интерфейсным IP-адресом, базовый прослушиватель для размещения одного сайта на шлюзе приложений, базовое правило маршрутизации запросов и две виртуальные машины, используемые для серверного пула.

Инструкции в этом кратком руководстве можно также выполнить с помощью [Azure PowerShell](quick-create-powershell.md) или [портала Azure](quick-create-portal.md).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0.4 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-resource-group"></a>Создать группу ресурсов

В Azure выделите связанные ресурсы группе ресурсов. Создайте группу ресурсов с помощью `az group create`. 

В следующем примере создается группа ресурсов с именем *myResourceGroupAG* в расположении *eastus*.

```azurecli-interactive 
az group create --name myResourceGroupAG --location eastus
```

## <a name="create-network-resources"></a>Создание сетевых ресурсов 

В Azure для обмена между создаваемыми ресурсами необходима виртуальная сеть.  Подсеть шлюза приложений может содержать только шлюзы приложений. Другие ресурсы запрещены.  Вы можете создать новую подсеть для шлюза приложений или использовать уже существующую. В этом примере создаются две подсети: одна — для шлюза приложений, а вторая — для внутренних серверов. Вы можете настроить интерфейсный IP-адрес как общедоступный или частный, в зависимости от варианта использования шлюза приложений. В этом примере мы будем использовать общедоступный интерфейсный IP-адрес.

Чтобы создать виртуальную сеть и подсеть, используйте `az network vnet create`. Воспользуйтесь командой `az network public-ip create`, чтобы создать общедоступный IP-адрес.

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
  --vnet-name myVNet   \
  --address-prefix 10.0.2.0/24
az network public-ip create \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress \
  --allocation-method Static \
  --sku Standard
```

## <a name="create-the-backend-servers"></a>Создайте внутренние серверы.

Серверная часть может содержать сетевые адаптеры, масштабируемые наборы виртуальных машин, общедоступные IP-адреса, внутренние IP-адреса, полные доменные имена и такие мультитенантые серверные части, как Служба приложений Azure. В этом примере вы создадите две виртуальные машины, которые будут использоваться как внутренние серверы для шлюза приложений. Вы также установите службы IIS на виртуальных машинах для проверки шлюза приложений.

#### <a name="create-two-virtual-machines"></a>Создание двух виртуальных машин

Чтобы убедиться в успешном создании шлюза приложения, установите веб-сервер NGINX на виртуальные машины. Тот же самый файл конфигурации cloud-init можно использовать и для установки NGINX, а также для запуска простого приложения Node.js "Hello World" на виртуальной машине Linux. Дополнительные сведения о cloud-init см. в статье [Поддержка cloud-init для виртуальных машин в Azure](../virtual-machines/linux/using-cloud-init.md).

В Azure Cloud Shell создайте файл *cloud-init.txt* и вставьте в него следующую конфигурацию. Чтобы создать файл, введите *editor cloud-init.txt*.

```yaml
#cloud-config
package_upgrade: true
packages:
  - nginx
  - nodejs
  - npm
write_files:
  - owner: www-data:www-data
  - path: /etc/nginx/sites-available/default
    content: |
      server {
        listen 80;
        location / {
          proxy_pass http://localhost:3000;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection keep-alive;
          proxy_set_header Host $host;
          proxy_cache_bypass $http_upgrade;
        }
      }
  - owner: azureuser:azureuser
  - path: /home/azureuser/myapp/index.js
    content: |
      var express = require('express')
      var app = express()
      var os = require('os');
      app.get('/', function (req, res) {
        res.send('Hello World from host ' + os.hostname() + '!')
      })
      app.listen(3000, function () {
        console.log('Hello world app listening on port 3000!')
      })
runcmd:
  - service nginx restart
  - cd "/home/azureuser/myapp"
  - npm init
  - npm install express -y
  - nodejs index.js
```

Создайте сетевые интерфейсы с помощью `az network nic create`. Чтобы создать виртуальные машины, используйте команду `az vm create`.

```azurecli-interactive
for i in `seq 1 2`; do
  az network nic create \
    --resource-group myResourceGroupAG \
    --name myNic$i \
    --vnet-name myVNet \
    --subnet myBackendSubnet
  az vm create \
    --resource-group myResourceGroupAG \
    --name myVM$i \
    --nics myNic$i \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys \
    --custom-data cloud-init.txt
done
```

## <a name="create-the-application-gateway"></a>Создание шлюза приложений

Создайте шлюз приложений с помощью `az network application-gateway create`. При создании шлюза приложения с помощью Azure CLI укажите такие сведения о конфигурации, как емкость, номер SKU и параметры HTTP. Azure добавляет частные IP-адреса сетевых интерфейсов в качестве серверов внутреннего пула шлюза приложения.

```azurecli-interactive
address1=$(az network nic show --name myNic1 --resource-group myResourceGroupAG | grep "\"privateIpAddress\":" | grep -oE '[^ ]+$' | tr -d '",')
address2=$(az network nic show --name myNic2 --resource-group myResourceGroupAG | grep "\"privateIpAddress\":" | grep -oE '[^ ]+$' | tr -d '",')
az network application-gateway create \
  --name myAppGateway \
  --location eastus \
  --resource-group myResourceGroupAG \
  --capacity 2 \
  --sku Standard_v2 \
  --http-settings-cookie-based-affinity Enabled \
  --public-ip-address myAGPublicIPAddress \
  --vnet-name myVNet \
  --subnet myAGSubnet \
  --servers "$address1" "$address2"
```

Чтобы создать шлюз приложения в Azure, может понадобиться до 30 минут. После его создания в разделе **Параметры** страницы **Шлюз приложений** можно просматривать следующие параметры:

- **appGatewayBackendPool**: отображается на странице **Серверные пулы**. Он задает требуемые значения для пулов серверной части.
- **appGatewayBackendHttpSettings**: отображается на странице **Параметры HTTP**. Он указывает, что для связи шлюз приложений использует протокол HTTP и 80 порт.
- **appGatewayHttpListener**: отображается на **Странице слушателей**. Задает прослушиватель по умолчанию, связанный с **appGatewayBackendPool**.
- **appGatewayFrontendIP**: отображается на странице **Конфигурации IP внешнего интерфейса**. Он назначает адрес *myAGPublicIPAddress* для прослушивателя **appGatewayHttpListener**.
- **rule1**: отображается на странице **Правила**. Указывает правило маршрутизации по умолчанию, связанное с **appGatewayHttpListener**.

## <a name="test-the-application-gateway"></a>Тестирование шлюза приложений

Несмотря на то что Azure не требует устанавливать веб-сервер NGINX для создания шлюза приложения, в рамках этого руководства он устанавливается, чтобы проверить, создан ли шлюз приложений. Чтобы получить общедоступный IP-адрес нового шлюза приложений, используйте `az network public-ip show`. 

```azurecli-interactive
az network public-ip show \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress \
  --query [ipAddress] \
  --output tsv
```

Скопируйте и вставьте общедоступный IP-адрес в адресную строку браузера.
    
![Тестирование шлюза приложений](./media/quick-create-cli/application-gateway-nginxtest.png)

Когда вы обновляете браузер, должно отобразиться имя второй виртуальной машины. Это означает, что шлюз приложений успешно создан и может подключиться к серверу.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если ресурсы, которые были созданы с помощью шлюза приложения, больше не нужны, воспользуйтесь командой `az group delete`, чтобы удалить группу ресурсов. Удалив ее, вы также удалите шлюз приложений и все связанные с ним ресурсы.

```azurecli-interactive 
az group delete --name myResourceGroupAG
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Руководство по управлению веб-трафиком с помощью шлюза приложений и Azure CLI](./tutorial-manage-web-traffic-cli.md)

