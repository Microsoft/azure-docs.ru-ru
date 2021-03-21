---
title: Развертывание и настройка брандмауэра Azure с помощью Azure CLI
description: Из этой статьи вы узнаете, как развернуть и настроить брандмауэр Azure с помощью Azure CLI.
services: firewall
author: vhorne
ms.service: firewall
ms.date: 08/29/2019
ms.author: victorh
ms.topic: how-to
ms.openlocfilehash: 4d611a6d575fbc94a555006882f77e5a31753164
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98051468"
---
# <a name="deploy-and-configure-azure-firewall-using-azure-cli"></a>Развертывание и настройка брандмауэра Azure с помощью Azure CLI

Управление доступом исходящих сетевых подключений является важной частью общего плана безопасности сети. Например, вы можете ограничить доступ к веб-сайтам или исходящим IP-адресам и портам, которые могут быть доступными.

Вы можете управлять доступом к исходящей сети из подсети Azure только с помощью Брандмауэра Azure. Брандмауэр Azure позволяет настроить:

* Правила приложений, определяющие полные доменные имена (FQDN), к которым можно получить доступ из подсети. Полное доменное имя может также [содержать экземпляры SQL](sql-fqdn-filtering.md).
* Правила сети, определяющие адрес источника, протокол, порт назначения и адрес назначения.

При маршрутизации трафика на брандмауэр, используемый в качестве шлюза по умолчанию, для подсети к трафику применяются настроенные правила брандмауэра.

В этой статье вы создадите упрощенную отдельную виртуальную сеть с тремя подсетями для простоты развертывания. Для развертываний в рабочей среде рекомендуется использовать [звездообразную модель](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke). Брандмауэр находится в собственной виртуальной сети. Серверы рабочей нагрузки размещены в одноранговых виртуальных сетях в одном регионе с одной или несколькими подсетями.

* **AzureFirewallSubnet** — в этой подсети находится брандмауэр.
* **Workload-SN** — в этой подсети находится сервер рабочей нагрузки. Трафик этой подсети проходит через брандмауэр.
* **Jump-SN** — в этой подсети находится сервер перехода. Сервер перехода имеет общедоступный IP-адрес, к которому можно подключиться с помощью удаленного рабочего стола. Затем вы можете подключиться к серверу рабочей нагрузки (используя другой удаленный рабочий стол).

![Инфраструктура сети, используемая в руководстве](media/tutorial-firewall-rules-portal/Tutorial_network.png)

Вы узнаете, как выполнять следующие задачи:

* настройка тестовой сетевой среды;
* развертывание брандмауэра;
* создание маршрута по умолчанию;
* настройка правила приложения для предоставления доступа к [www.google.com]\(www.google.com);
* настройка сетевых правил для предоставления доступа к внешним DNS-серверам;
* тестирование брандмауэра.

При желании эту процедуру можно выполнить с помощью [портал Azure](tutorial-firewall-deploy-portal.md) или [Azure PowerShell](deploy-ps.md).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0.4 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="set-up-the-network"></a>Настройка сети

Сначала создайте группу ресурсов, необходимых для развертывания брандмауэра. Затем создайте виртуальную сеть, подсети и тестовые серверы.

### <a name="create-a-resource-group"></a>Создание группы ресурсов

Группа ресурсов содержит все ресурсы для развертывания.

```azurecli-interactive
az group create --name Test-FW-RG --location eastus
```

### <a name="create-a-vnet"></a>Создание виртуальной сети

Эта виртуальная сеть имеет три подсети.

> [!NOTE]
> Размер подсети AzureFirewallSubnet равен /26. Дополнительные сведения о размере подсети см. в статье с [часто задаваемыми вопросами о Брандмауэре Azure](firewall-faq.yml#why-does-azure-firewall-need-a--26-subnet-size).

```azurecli-interactive
az network vnet create \
  --name Test-FW-VN \
  --resource-group Test-FW-RG \
  --location eastus \
  --address-prefix 10.0.0.0/16 \
  --subnet-name AzureFirewallSubnet \
  --subnet-prefix 10.0.1.0/26
az network vnet subnet create \
  --name Workload-SN \
  --resource-group Test-FW-RG \
  --vnet-name Test-FW-VN   \
  --address-prefix 10.0.2.0/24
az network vnet subnet create \
  --name Jump-SN \
  --resource-group Test-FW-RG \
  --vnet-name Test-FW-VN   \
  --address-prefix 10.0.3.0/24
```

### <a name="create-virtual-machines"></a>Создание виртуальных машин

Теперь создайте виртуальные машины для перехода и рабочей нагрузки и поместите их в соответствующие подсети.
При появлении запроса введите пароль для виртуальной машины.

Создайте Srv-Jump виртуальную машину.

```azurecli-interactive
az vm create \
    --resource-group Test-FW-RG \
    --name Srv-Jump \
    --location eastus \
    --image win2016datacenter \
    --vnet-name Test-FW-VN \
    --subnet Jump-SN \
    --admin-username azureadmin
az vm open-port --port 3389 --resource-group Test-FW-RG --name Srv-Jump
```



Создайте сетевую карту для Srv-Work с конкретными IP-адресами DNS-сервера и без общедоступного IP-адреса для тестирования.

```azurecli-interactive
az network nic create \
    -g Test-FW-RG \
    -n Srv-Work-NIC \
   --vnet-name Test-FW-VN \
   --subnet Workload-SN \
   --public-ip-address "" \
   --dns-servers 209.244.0.3 209.244.0.4
```

Теперь создайте виртуальную машину рабочей нагрузки.
При появлении запроса введите пароль для виртуальной машины.

```azurecli-interactive
az vm create \
    --resource-group Test-FW-RG \
    --name Srv-Work \
    --location eastus \
    --image win2016datacenter \
    --nics Srv-Work-NIC \
    --admin-username azureadmin
```

## <a name="deploy-the-firewall"></a>Развертывание брандмауэра

Теперь разверните брандмауэр в виртуальной сети.

```azurecli-interactive
az network firewall create \
    --name Test-FW01 \
    --resource-group Test-FW-RG \
    --location eastus
az network public-ip create \
    --name fw-pip \
    --resource-group Test-FW-RG \
    --location eastus \
    --allocation-method static \
    --sku standard
az network firewall ip-config create \
    --firewall-name Test-FW01 \
    --name FW-config \
    --public-ip-address fw-pip \
    --resource-group Test-FW-RG \
    --vnet-name Test-FW-VN
az network firewall update \
    --name Test-FW01 \
    --resource-group Test-FW-RG 
az network public-ip show \
    --name fw-pip \
    --resource-group Test-FW-RG
fwprivaddr="$(az network firewall ip-config list -g Test-FW-RG -f Test-FW01 --query "[?name=='FW-config'].privateIpAddress" --output tsv)"
```

Запишите частный IP-адрес. Вы будете использовать его позже при создании маршрута по умолчанию.

## <a name="create-a-default-route"></a>создание маршрута по умолчанию;

Создание таблицы с отключенным распространением маршрута BGP

```azurecli-interactive
az network route-table create \
    --name Firewall-rt-table \
    --resource-group Test-FW-RG \
    --location eastus \
    --disable-bgp-route-propagation true
```

Создайте маршрут.

```azurecli-interactive
az network route-table route create \
  --resource-group Test-FW-RG \
  --name DG-Route \
  --route-table-name Firewall-rt-table \
  --address-prefix 0.0.0.0/0 \
  --next-hop-type VirtualAppliance \
  --next-hop-ip-address $fwprivaddr
```

Связывание таблицы маршрутов с подсетью

```azurecli-interactive
az network vnet subnet update \
    -n Workload-SN \
    -g Test-FW-RG \
    --vnet-name Test-FW-VN \
    --address-prefixes 10.0.2.0/24 \
    --route-table Firewall-rt-table
```

## <a name="configure-an-application-rule"></a>Настройка правила приложения

Правило приложения разрешает исходящий доступ к www.google.com.

```azurecli-interactive
az network firewall application-rule create \
   --collection-name App-Coll01 \
   --firewall-name Test-FW01 \
   --name Allow-Google \
   --protocols Http=80 Https=443 \
   --resource-group Test-FW-RG \
   --target-fqdns www.google.com \
   --source-addresses 10.0.2.0/24 \
   --priority 200 \
   --action Allow
```

Брандмауэр Azure содержит встроенную коллекцию правил для целевых полных доменных имен инфраструктуры, которые разрешены по умолчанию. Эти доменные имена предназначены для платформы и не могут использоваться для других целей. См. дополнительные сведения об [FQDN инфраструктуры](infrastructure-fqdns.md).

## <a name="configure-a-network-rule"></a>Настройка правила сети

Правило сети разрешает исходящий доступ к двум IP-адресам через порт 53 (DNS).

```azurecli-interactive
az network firewall network-rule create \
   --collection-name Net-Coll01 \
   --destination-addresses 209.244.0.3 209.244.0.4 \
   --destination-ports 53 \
   --firewall-name Test-FW01 \
   --name Allow-DNS \
   --protocols UDP \
   --resource-group Test-FW-RG \
   --priority 200 \
   --source-addresses 10.0.2.0/24 \
   --action Allow
```

## <a name="test-the-firewall"></a>тестирование брандмауэра.

Теперь проверьте брандмауэр, чтобы убедиться, что он работает должным образом.

1. Запишите частный IP-адрес для виртуальной машины **SRV-работы** :

   ```azurecli-interactive
   az vm list-ip-addresses \
   -g Test-FW-RG \
   -n Srv-Work
   ```

1. Подключите удаленный рабочий стол к виртуальной машине **Srv-Jump** и выполните вход. После этого откройте подключение к удаленному рабочему столу к частному IP-адресу **SRV** и выполните вход.

3. На **SRV-работе** откройте окно PowerShell и выполните следующие команды:

   ```
   nslookup www.google.com
   nslookup www.microsoft.com
   ```

   Обе команды должны возвращать ответы, показывая, что запросы DNS проходят через брандмауэр.

1. Выполните следующие команды:

   ```
   Invoke-WebRequest -Uri https://www.google.com
   Invoke-WebRequest -Uri https://www.google.com

   Invoke-WebRequest -Uri https://www.microsoft.com
   Invoke-WebRequest -Uri https://www.microsoft.com
   ```

   `www.google.com`Запросы должны выполняться успешно, и `www.microsoft.com` запросы должны завершаться ошибкой. Это показывает, что правила брандмауэра работают должным образом.

Итак, теперь вы убедились в том, что правила брандмауэра работают:

* Вы можете разрешать имена DNS с помощью настроенного внешнего DNS-сервера.
* Вы можете перейти только к одному разрешенному имени FQDN.

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете разместить ресурсы брандмауэра для следующего руководства или, если они больше не нужны, удалить группу ресурсов **Test-FW-RG** , чтобы удалить все ресурсы, связанные с брандмауэром:

```azurecli-interactive
az group delete \
  -n Test-FW-RG
```

## <a name="next-steps"></a>Дальнейшие действия

* [Руководство. Мониторинг журналов и метрик Брандмауэра Azure](./firewall-diagnostics.md)