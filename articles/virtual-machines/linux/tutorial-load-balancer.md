---
title: Учебник. Балансировка нагрузки виртуальных машин Linux в Azure
description: В этом руководстве описано, как с помощью Azure CLI создать подсистему балансировки нагрузки для высокодоступного безопасного приложения, выполняемого на трех виртуальных машинах Linux.
services: virtual-machines
documentationcenter: virtual-machines
author: cynthn
manager: gwallace
tags: azure-resource-manager
ms.subservice: networking
ms.assetid: ''
ms.service: virtual-machines
ms.collection: linux
ms.devlang: azurecli
ms.topic: tutorial
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 11/13/2017
ms.author: cynthn
ms.custom: mvc, devx-track-js, devx-track-azurecli
ms.openlocfilehash: 433bbd51618cfb5624c8ed2c549e1793488f0e81
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102553771"
---
# <a name="tutorial-load-balance-linux-virtual-machines-in-azure-to-create-a-highly-available-application-with-the-azure-cli"></a>Руководство по Балансировка нагрузки виртуальных машин Linux в Azure для создания приложения высокого уровня доступности с помощью Azure CLI

Балансировка нагрузки обеспечивает более высокий уровень доступности за счет распределения входящих запросов между несколькими виртуальными машинами. В этом руководстве вы узнаете о различных компонентах балансировщика нагрузки Azure Load Balancer, распределяющего трафик и обеспечивающего высокую доступность. Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Создание Azure Load Balancer
> * Создание пробы работоспособности балансировщика нагрузки
> * создавать правила трафика подсистемы балансировки нагрузки;
> * создавать базовое приложение Node.js с помощью файла конфигурации cloud-init;
> * создавать виртуальные машины и присоединять их к подсистеме балансировки нагрузки;
> * Просмотр балансировщика нагрузки в действии
> * добавлять и удалять виртуальные машины из подсистемы балансировки нагрузки.

При работе с этим руководством используется интерфейс командной строки (CLI) в [Azure Cloud Shell](../../cloud-shell/overview.md), который всегда обновлен до последней версии. Чтобы открыть Cloud Shell, выберите **Попробовать** в верхнем углу любого блока кода.

Если вы решили установить и использовать интерфейс командной строки локально, то для работы с этим руководством вам понадобится Azure CLI 2.0.30 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0]( /cli/azure/install-azure-cli).

## <a name="azure-load-balancer-overview"></a>Обзор Azure Load Balancer
Azure Load Balancer представляет собой балансировщик нагрузки уровня 4 (TCP, UDP), который обеспечивает высокий уровень доступности, распределяя входящий трафик между работоспособными виртуальными машинами. Проба работоспособности позволяет отслеживать данный порт на каждой виртуальной машине и передавать трафик только в рабочую виртуальную машину.

Вы определяете интерфейсную конфигурацию IP-адресов, содержащую один или несколько общедоступных IP-адресов. Такая интерфейсная конфигурация IP-адресов предоставляет доступ к балансировщику нагрузки и приложениям через Интернет. 

Виртуальные машины подключаются к балансировщику нагрузки с помощью виртуальной сетевой карты. Для распределения трафика между виртуальными машинами внутренний пул адресов содержит IP-адреса виртуальных сетевых карт, подключенных к балансировщику нагрузки.

Чтобы управлять потоком трафика, нужно определить правила балансировщика нагрузки для определенных портов и протоколов, сопоставленных с виртуальными машинами.

Если в предыдущем руководстве вы дошли до шага [Создание набора для масштабирования виртуальной машины](tutorial-create-vmss.md), балансировщик нагрузки должен быть уже создан. Все эти компоненты были настроены в составе масштабируемого набора.


## <a name="create-azure-load-balancer"></a>Создание Azure Load Balancer
Этот раздел подробно описывает, как создать и настроить каждый из компонентов балансировщика нагрузки. Прежде чем создать балансировщик нагрузки, выполните команду [az group create](/cli/azure/group) для создания группы ресурсов. В следующем примере создается группа ресурсов с именем *myResourceGroupLoadBalancer* в расположении *eastus*.

```azurecli-interactive
az group create --name myResourceGroupLoadBalancer --location eastus
```

### <a name="create-a-public-ip-address"></a>Создание общедоступного IP-адреса
Для доступа к приложению через Интернет требуется общедоступный IP-адрес для балансировщика нагрузки. Создайте общедоступный IP-адрес с помощью команды [az network public-ip create](/cli/azure/network/public-ip). В следующем примере создается общедоступный IP-адрес *myPublicIP* в группе ресурсов *myResourceGroupLoadBalancer*.

```azurecli-interactive
az network public-ip create \
    --resource-group myResourceGroupLoadBalancer \
    --name myPublicIP
```

### <a name="create-a-load-balancer"></a>Создание балансировщика нагрузки
Создайте балансировщик нагрузки с помощью команды [az network lb create](/cli/azure/network/lb). В следующем примере создается подсистема балансировки нагрузки *myLoadBalancer*, а адрес *myPublicIP* назначается внешней IP-конфигурации.

```azurecli-interactive
az network lb create \
    --resource-group myResourceGroupLoadBalancer \
    --name myLoadBalancer \
    --frontend-ip-name myFrontEndPool \
    --backend-pool-name myBackEndPool \
    --public-ip-address myPublicIP
```

### <a name="create-a-health-probe"></a>Создание пробы работоспособности
Чтобы балансировщик нагрузки мог следить за состоянием приложения, необходимо настроить пробу работоспособности. Проба работоспособности динамически добавляет или удаляет виртуальные машины из балансировщика нагрузки на основе их ответа на проверки работоспособности. По умолчанию виртуальная машина удаляется из числа машин, на которые балансировщик распределяет нагрузку, после двух последовательных сбоев с интервалом в 15 секунд. Пробу работоспособности можно создать на основе протокола или конкретной страницы проверки работоспособности приложения. 

В следующем примере создается проба TCP. Вы также можете создать настраиваемую пробу HTTP для более детальных проверок. При использовании настраиваемой пробы HTTP нужно создать страницу проверки работоспособности, например *healthcheck.js*. Чтобы обеспечить работоспособность узла, проба должна возвращать ответ **HTTP 200 OK** для балансировщика нагрузки.

Чтобы создать пробу работоспособности TCP, используйте команду [az network lb probe create](/cli/azure/network/lb/probe). В следующем примере создается проба TCP *myHealthProbe*.

```azurecli-interactive
az network lb probe create \
    --resource-group myResourceGroupLoadBalancer \
    --lb-name myLoadBalancer \
    --name myHealthProbe \
    --protocol tcp \
    --port 80
```

### <a name="create-a-load-balancer-rule"></a>Создание правила балансировщика нагрузки
Правило балансировщика нагрузки позволяет определить распределение трафика между виртуальными машинами. Вы определяете интерфейсную конфигурацию IP-адресов для входящего трафика и внутренний пул IP-адресов для приема трафика, а также требуемый порт источника и назначения. Чтобы обеспечить получение трафика только работоспособными виртуальными машинами, можно также определить используемую пробу работоспособности.

Создайте правило балансировщика нагрузки с помощью команды [az network lb rule create](/cli/azure/network/lb/rule). В следующем примере создается правило *myLoadBalancerRule*, используется проба работоспособности *myHealthProbe* и трафик распределяется через порт *80*.

```azurecli-interactive
az network lb rule create \
    --resource-group myResourceGroupLoadBalancer \
    --lb-name myLoadBalancer \
    --name myLoadBalancerRule \
    --protocol tcp \
    --frontend-port 80 \
    --backend-port 80 \
    --frontend-ip-name myFrontEndPool \
    --backend-pool-name myBackEndPool \
    --probe-name myHealthProbe
```


## <a name="configure-virtual-network"></a>Настройка виртуальной сети
Прежде чем развертывать виртуальные машины и тестировать балансировщик нагрузки, создайте вспомогательные ресурсы виртуальной сети. Дополнительные сведения о виртуальных сетях см. в руководстве [Управление виртуальными сетями Azure](tutorial-virtual-network.md).

### <a name="create-network-resources"></a>Создание сетевых ресурсов
Создайте виртуальную сеть с помощью команды [az network vnet create](/cli/azure/network/vnet). В следующем примере создается виртуальная сеть *myVnet* с подсетью *mySubnet*.

```azurecli-interactive
az network vnet create \
    --resource-group myResourceGroupLoadBalancer \
    --name myVnet \
    --subnet-name mySubnet
```

Чтобы добавить группу безопасности сети, используйте команду [az network nsg create](/cli/azure/network/nsg). В следующем примере создается группа безопасности сети *myNetworkSecurityGroup*.

```azurecli-interactive
az network nsg create \
    --resource-group myResourceGroupLoadBalancer \
    --name myNetworkSecurityGroup
```

Создайте правило группы безопасности сети с помощью команды [az network nsg rule create](/cli/azure/network/nsg/rule). В следующем примере создается правило группы безопасности сети *myNetworkSecurityGroupRule*.

```azurecli-interactive
az network nsg rule create \
    --resource-group myResourceGroupLoadBalancer \
    --nsg-name myNetworkSecurityGroup \
    --name myNetworkSecurityGroupRule \
    --priority 1001 \
    --protocol tcp \
    --destination-port-range 80
```

Для создания виртуальных сетевых карт используется команда [az network nic create](/cli/azure/network/nic). В следующем примере создаются три виртуальных сетевых адаптера (по одной виртуальной сетевой карте для каждой виртуальной машины, используемой приложением). Вы можете в любое время создать дополнительные виртуальные сетевые карты и виртуальные машины и добавить их в балансировщик нагрузки:

```azurecli
for i in `seq 1 3`; do
    az network nic create \
        --resource-group myResourceGroupLoadBalancer \
        --name myNic$i \
        --vnet-name myVnet \
        --subnet mySubnet \
        --network-security-group myNetworkSecurityGroup \
        --lb-name myLoadBalancer \
        --lb-address-pools myBackEndPool
done
```

Когда все три виртуальных сетевых адаптера будут созданы, переходите к следующему шагу.


## <a name="create-virtual-machines"></a>Создание виртуальных машин

### <a name="create-cloud-init-config"></a>Создание конфигурации cloud-init
В предыдущем руководстве [Как настроить виртуальную машину Linux при первой загрузке](tutorial-automate-vm-deployment.md) вы узнали, как автоматизировать настройку виртуальной машины с помощью cloud-init. Этот же файл конфигурации cloud-init можно использовать для установки NGINX, а также для запуска простого приложения Node.js "Hello World" на следующем шаге. Чтобы увидеть подсистему балансировки нагрузки в действии, в конце руководства мы откроем это простое приложение в веб-браузере.

В текущей оболочке создайте файл *cloud-init.txt* и вставьте в него следующую конфигурацию. Например, создайте файл в Cloud Shell, не на локальном компьютере. Введите `sensible-editor cloud-init.txt`, чтобы создать файл и просмотреть список доступных редакторов. Убедитесь, что весь файл cloud-init скопирован правильно, особенно первая строка:

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

### <a name="create-virtual-machines"></a>Создание виртуальных машин
Чтобы улучшить высокую доступность приложения, поместите виртуальные машины в группу доступности. Дополнительные сведения о группах доступности см. в руководстве [Создание высокодоступных виртуальных машин](tutorial-availability-sets.md).

Создайте группу доступности с помощью команды [az vm availability-set create](/cli/azure/vm/availability-set). В следующем примере создается группа доступности *myAvailabilitySet*.

```azurecli-interactive 
az vm availability-set create \
    --resource-group myResourceGroupLoadBalancer \
    --name myAvailabilitySet
```

Теперь вы можете создать виртуальные машины с помощью команды [az vm create](/cli/azure/vm). В следующем примере создаются три виртуальные машины и ключи SSH, если они не существуют.

```azurecli
for i in `seq 1 3`; do
    az vm create \
        --resource-group myResourceGroupLoadBalancer \
        --name myVM$i \
        --availability-set myAvailabilitySet \
        --nics myNic$i \
        --image UbuntuLTS \
        --admin-username azureuser \
        --generate-ssh-keys \
        --custom-data cloud-init.txt \
        --no-wait
done
```

Некоторые фоновые задачи продолжают работу после возврата к командной строке в Azure CLI. Параметр `--no-wait` не ожидает завершения всех задач. Прежде чем вы получите доступ к приложению, может пройти несколько минут. Проба работоспособности балансировщика нагрузки автоматически определяет, когда приложение работает на каждой виртуальной машине. Как только приложение будет запущено, правило балансировщика нагрузки начнет распределение трафика.


## <a name="test-load-balancer"></a>Проверка балансировщика нагрузки
Получите общедоступный IP-адрес балансировщика нагрузки с помощью команды [az network public-ip show](/cli/azure/network/public-ip). Следующий пример позволяет получить IP-адрес для созданного ранее *myPublicIP*.

```azurecli-interactive
az network public-ip show \
    --resource-group myResourceGroupLoadBalancer \
    --name myPublicIP \
    --query [ipAddress] \
    --output tsv
```

После этого можно ввести общедоступный IP-адрес в веб-браузер. Прежде чем подсистема балансировки нагрузки начнет распределять трафик между виртуальными машинами, виртуальные машины должны быть подготовлены. Эта подготовка может занять несколько минут. Отображается приложение, а также имя узла виртуальной машины, на которую балансировщик нагрузки направил трафик, как показано в следующем примере:

![Запуск приложения Node.js](./media/tutorial-load-balancer/running-nodejs-app.png)

Чтобы увидеть, как балансировщик нагрузки распределяет трафик между тремя виртуальными машинами, на которых выполняется приложение, принудительно обновите веб-браузер.


## <a name="add-and-remove-vms"></a>Добавление и удаление виртуальных машин
На виртуальных машинах, выполняющих приложение, может потребоваться выполнить обслуживание, например установить обновления операционной системы. Для обработки большего объема трафика в приложении необходимо добавить дополнительные виртуальные машины. В этом разделе показано, как удалить или добавить виртуальную машину из балансировщика нагрузки.

### <a name="remove-a-vm-from-the-load-balancer"></a>Удаление виртуальной машины из балансировщика нагрузки
Вы можете удалить виртуальную машину из внутреннего пула адресов с помощью команды [az network nic ip-config address-pool remove](/cli/azure/network/nic/ip-config/address-pool). В следующем примере из *myLoadBalancer* удаляется виртуальная сетевая карта для **myVM2**.

```azurecli-interactive
az network nic ip-config address-pool remove \
    --resource-group myResourceGroupLoadBalancer \
    --nic-name myNic2 \
    --ip-config-name ipConfig1 \
    --lb-name myLoadBalancer \
    --address-pool myBackEndPool 
```

Чтобы увидеть, как балансировщик нагрузки распределяет трафик между оставшимися двумя виртуальными машинами, на которых выполняется приложение, принудительно обновите веб-браузер. Теперь можно выполнить обслуживание на виртуальной машине, например установить обновления операционной системы или перезагрузить виртуальную машину.

Чтобы просмотреть список виртуальных машин с виртуальными сетевыми адаптерами, подключенными к подсистеме балансировки нагрузки, используйте команду [az network lb address-pool show](/cli/azure/network/lb/address-pool). Запросите и отфильтруйте идентификатор виртуального сетевого адаптера, как показано ниже.

```azurecli-interactive
az network lb address-pool show \
    --resource-group myResourceGroupLoadBalancer \
    --lb-name myLoadBalancer \
    --name myBackEndPool \
    --query backendIpConfigurations \
    --output tsv | cut -f4
```

Ниже показан пример выходных данных, из которого следует, что виртуальный сетевой адаптер для виртуальной машины 2 больше не является частью серверного пула адресов.

```output
/subscriptions/<guid>/resourceGroups/myResourceGroupLoadBalancer/providers/Microsoft.Network/networkInterfaces/myNic1/ipConfigurations/ipconfig1
/subscriptions/<guid>/resourceGroups/myResourceGroupLoadBalancer/providers/Microsoft.Network/networkInterfaces/myNic3/ipConfigurations/ipconfig1
```

### <a name="add-a-vm-to-the-load-balancer"></a>Добавление виртуальной машины в балансировщик нагрузки
После выполнения обслуживания виртуальной машины, или если необходимо расширить емкость, можно добавить виртуальную машину во внутренний пул адресов с помощью команды [az network nic ip-config address-pool add](/cli/azure/network/nic/ip-config/address-pool). В следующем примере в *myLoadBalancer* добавляется виртуальная сетевая карта для **myVM2**.

```azurecli-interactive
az network nic ip-config address-pool add \
    --resource-group myResourceGroupLoadBalancer \
    --nic-name myNic2 \
    --ip-config-name ipConfig1 \
    --lb-name myLoadBalancer \
    --address-pool myBackEndPool
```

Чтобы убедиться, что виртуальный сетевой адаптер подключен к серверному пулу адресов, еще раз выполните команду [az network lb address-pool show](/cli/azure/network/lb/address-pool).


## <a name="next-steps"></a>Дальнейшие действия
В рамках этого руководства вы создали балансировщик нагрузки и присоединили к нему виртуальные машины. Вы ознакомились с выполнением следующих задач:

> [!div class="checklist"]
> * Создание Azure Load Balancer
> * Создание пробы работоспособности балансировщика нагрузки
> * создавать правила трафика подсистемы балансировки нагрузки;
> * создавать базовое приложение Node.js с помощью файла конфигурации cloud-init;
> * создавать виртуальные машины и присоединять их к подсистеме балансировки нагрузки;
> * Просмотр балансировщика нагрузки в действии
> * добавлять и удалять виртуальные машины из подсистемы балансировки нагрузки.

Для получения дополнительных сведений о компонентах виртуальных сетей Azure перейдите к указанному ниже руководству.

> [!div class="nextstepaction"]
> [Управление виртуальными сетями и виртуальными машинами](tutorial-virtual-network.md)
