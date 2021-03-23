---
title: IP-диапазоны авторизации сервера API в службе Kubernetes Azure (AKS)
description: Узнайте, как защитить кластер с помощью диапазона IP-адресов для доступа к серверу API в службе Kubernetes Azure (AKS).
services: container-service
ms.topic: article
ms.date: 09/21/2020
ms.openlocfilehash: 20c3d46e9a19c66c727e5a21bccd6caa0dc5f0ba
ms.sourcegitcommit: 2c1b93301174fccea00798df08e08872f53f669c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104771929"
---
# <a name="secure-access-to-the-api-server-using-authorized-ip-address-ranges-in-azure-kubernetes-service-aks"></a>Безопасный доступ к серверу API с помощью диапазонов полномочных IP-адресов в службе Kubernetes Azure (AKS)

В Kubernetes сервер API получает запросы на выполнение действий в кластере, например для создания ресурсов или масштабирования количества узлов. Сервер API является основным способом взаимодействия с кластером и управления им. Чтобы повысить безопасность кластера и сократить число атак, сервер API должен быть доступен только из ограниченного набора диапазонов IP-адресов.

В этой статье показано, как использовать диапазоны разрешенных IP-адресов сервера API для ограничения IP-адресов и Цидрс для доступа к плоскости управления.

## <a name="before-you-begin"></a>Перед началом

В этой статье показано, как создать кластер AKS с помощью Azure CLI.

Вам необходимо установить и настроить службу Azure CLI версии 2.0.76 или более поздней. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][install-azure-cli].

### <a name="limitations"></a>Ограничения

Функция диапазона IP-адресов для авторизации сервера API имеет следующие ограничения.
- В кластерах, созданных после перегрузки диапазонов IP-адресов, разрешенных сервером API, из предварительной версии в октябре 2019, разрешенные диапазоны IP-адресов сервера API поддерживаются только для балансировщика нагрузки " *стандартный* ". Существующие кластеры с подсистемой балансировки нагрузки уровня " *базовый* " и "ПОЛНОМОЧНЫЙ IP-адрес сервера API" будут продолжать работать, как есть, но не могут быть перенесены в *стандартный* балансировщик нагрузки SKU. Эти существующие кластеры также продолжат работать при обновлении Kubernetes версии или плоскости управления. Разрешенные диапазоны IP-адресов сервера API не поддерживаются для частных кластеров.
- Эта функция несовместима с кластерами, которые используют [общедоступный IP-адрес для каждого узла](use-multiple-node-pools.md#assign-a-public-ip-per-node-for-your-node-pools).

## <a name="overview-of-api-server-authorized-ip-ranges"></a>Общие сведения о IP-адресах полномочных серверов API

Сервер API Kubernetes — это то, как предоставляются базовые API-интерфейсы Kubernetes. Этот компонент поддерживает взаимодействие со средствами управления, например с `kubectl` или панелью мониторинга Kubernetes. AKS предоставляет плоскость управления кластером с одним клиентом, с выделенным сервером API. По умолчанию серверу API назначается общедоступный IP-адрес, и вы должны контролировать доступ с помощью управления доступом на основе ролей Kubernetes (Kubernetes RBAC) или Azure RBAC.

Чтобы защитить доступ к общедоступной плоскости управления AKS или серверу API, можно включить и использовать разрешенные IP-диапазоны. Эти диапазоны разрешенных IP-адресов разрешают обмен данными только с сервером API. Запрос к серверу API из IP-адреса, который не является частью этих разрешенных диапазонов IP-адресов, блокируется. Продолжайте использовать Kubernetes RBAC или Azure RBAC для авторизации пользователей и действий, которые они запрашивают.

Дополнительные сведения о сервере API и других компонентах кластера см. в разделе [Основные понятия Kubernetes Core для AKS][concepts-clusters-workloads].

## <a name="create-an-aks-cluster-with-api-server-authorized-ip-ranges-enabled"></a>Создание кластера AKS с включенными IP-диапазонами разрешенных серверов API

Создайте кластер с помощью команды [AZ AKS Create][az-aks-create] и укажите *`--api-server-authorized-ip-ranges`* параметр, чтобы предоставить список допустимых диапазонов IP-адресов. Эти диапазоны IP-адресов обычно являются диапазонами адресов, используемыми локальными сетями или общедоступными IP-адресами. При указании диапазона CIDR начинается с первого IP-адреса в диапазоне. Например, *137.117.106.90/29* является допустимым диапазоном, но убедитесь, что вы указали первый IP-адрес в диапазоне, например *137.117.106.88/29*.

> [!IMPORTANT]
> По умолчанию в кластере используется [балансировщик нагрузки стандартного SKU][standard-sku-lb] , который можно использовать для настройки исходящего шлюза. При включении разрешенных диапазонов IP-адресов сервера API во время создания кластера общедоступный IP-адрес кластера также разрешается по умолчанию в дополнение к указанным диапазонам. Если указано значение *""* или нет значения для *`--api-server-authorized-ip-ranges`* , разрешенные диапазоны IP-адресов сервера API будут отключены. Обратите внимание, что если вы используете PowerShell, используйте *`--api-server-authorized-ip-ranges=""`* (со знаком равенства), чтобы избежать проблем с синтаксическим анализом.

В следующем примере создается кластер с одним узлом с именем *myAKSCluster* в группе ресурсов с именем *myResourceGroup* с включенными диапазонами разрешенных IP-адресов сервера API. Допустимые диапазоны IP-адресов: *73.140.245.0/24*.

```azurecli-interactive
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-count 1 \
    --vm-set-type VirtualMachineScaleSets \
    --load-balancer-sku standard \
    --api-server-authorized-ip-ranges 73.140.245.0/24 \
    --generate-ssh-keys
```

> [!NOTE]
> Эти диапазоны следует добавить в список разрешений:
> - Общедоступный IP-адрес брандмауэра
> - Любой диапазон, представляющий сети, из которых будет осуществляться администрирование кластера.
> - Если вы используете Azure Dev Spaces в кластере AKS, необходимо разрешить [Дополнительные диапазоны на основе вашего региона][dev-spaces-ranges].
>
> Верхний предел числа диапазонов IP-адресов, которые можно указать, — 200.
>
> Для распространения правил может потребоваться до 2min. Подождите, пока не протестируется подключение.

### <a name="specify-the-outbound-ips-for-the-standard-sku-load-balancer"></a>Укажите исходящие IP-адреса для стандартного балансировщика нагрузки SKU

При создании кластера AKS, если указаны исходящие IP-адреса или префиксы для кластера, эти адреса или префиксы также разрешены. Пример:

```azurecli-interactive
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-count 1 \
    --vm-set-type VirtualMachineScaleSets \
    --load-balancer-sku standard \
    --api-server-authorized-ip-ranges 73.140.245.0/24 \
    --load-balancer-outbound-ips <publicIpId1>,<publicIpId2> \
    --generate-ssh-keys
```

В приведенном выше примере все IP-адреса, указанные в параметре, *`--load-balancer-outbound-ip-prefixes`* разрешены вместе с IP-адресами в *`--api-server-authorized-ip-ranges`* параметре.

Вместо этого можно указать параметр, *`--load-balancer-outbound-ip-prefixes`* разрешающий IP-префиксы исходящей подсистемы балансировки нагрузки.

### <a name="allow-only-the-outbound-public-ip-of-the-standard-sku-load-balancer"></a>Разрешить только исходящий общедоступный IP-адрес SKU "Стандартный" балансировщика нагрузки

При включении допустимых диапазонов IP-адресов для сервера API во время создания кластера в дополнение к указанным диапазонам также разрешены исходящие общедоступные IP-адреса для стандартного балансировщика нагрузки SKU для кластера. Чтобы разрешить только исходящий общедоступный IP-адрес в подсистеме балансировки нагрузки уровня "Стандартный", используйте значение *0.0.0.0/32* при указании *`--api-server-authorized-ip-ranges`* параметра.

В следующем примере разрешен только исходящий общедоступный IP-адрес подсистемы балансировки нагрузки уровня "Стандартный", и доступ к серверу API можно получить только с узлов в кластере.

```azurecli-interactive
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-count 1 \
    --vm-set-type VirtualMachineScaleSets \
    --load-balancer-sku standard \
    --api-server-authorized-ip-ranges 0.0.0.0/32 \
    --generate-ssh-keys
```

## <a name="update-a-clusters-api-server-authorized-ip-ranges"></a>Обновление диапазонов IP-адресов сервера API кластера

Чтобы обновить разрешенные IP-адреса сервера API в существующем кластере, выполните команду [AZ AKS Update][az-aks-update] и используйте *`--api-server-authorized-ip-ranges`* Параметры,--Load-балансировщика-Outbound-IP-префиксы *, *`--load-balancer-outbound-ips`* или--Load-балансировщик-Outbound-IP-префиксы* .

В следующем примере выполняется обновление диапазонов IP-адресов сервера API в кластере с именем *myAKSCluster* в группе ресурсов с именем *myResourceGroup*. Диапазон IP-адресов для авторизации — *73.140.245.0/24*:

```azurecli-interactive
az aks update \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --api-server-authorized-ip-ranges  73.140.245.0/24
```

Можно также использовать *0.0.0.0/32* при указании *`--api-server-authorized-ip-ranges`* параметра, чтобы разрешить только общедоступный IP-адрес стандартного балансировщика нагрузки SKU.

## <a name="disable-authorized-ip-ranges"></a>Отключение диапазонов IP-адресов с правом доступа

Чтобы отключить допустимые диапазоны IP-адресов, выполните команду [AZ AKS Update][az-aks-update] и укажите пустой диапазон, чтобы отключить полномочные IP-адреса сервера API. Пример:

```azurecli-interactive
az aks update \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --api-server-authorized-ip-ranges ""
```

## <a name="find-existing-authorized-ip-ranges"></a>Найти существующие диапазоны IP-адресов с полномочиями

Чтобы найти допустимые IP-диапазоны, выполните команду [AZ AKS][az-aks-show] и укажите имя кластера и группу ресурсов. Пример:

```azurecli-interactive
az aks show \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --query apiServerAccessProfile.authorizedIpRanges'
```

## <a name="update-disable-and-find-authorized-ip-ranges-using-azure-portal"></a>Обновление, отключение и поиск допустимых диапазонов IP-адресов с помощью портал Azure

Приведенные выше операции добавления, обновления, поиска и отключения разрешенных IP-диапазонов также можно выполнить в портал Azure. Чтобы получить доступ, перейдите к **сети** в разделе **Параметры** в колонке меню ресурса кластера.

:::image type="content" source="media/api-server-authorized-ip-ranges/ip-ranges-specified.PNG" alt-text="В браузере отображаются сетевые параметры ресурса кластера портал Azure странице. Параметры &quot;задать указанный диапазон IP-адресов&quot; и &quot;указанные диапазоны IP-адресов&quot; выделены.":::

## <a name="how-to-find-my-ip-to-include-in---api-server-authorized-ip-ranges"></a>Как найти IP-адрес для включения в `--api-server-authorized-ip-ranges` ?

Чтобы получить доступ к серверу API, необходимо добавить компьютеры разработки, Инструментарий или IP-адреса автоматизации в список разрешенных диапазонов IP-адресов для кластера AKS. 

Кроме того, можно настроить инсталляционный сервер с необходимым инструментарием внутри отдельной подсети в виртуальной сети брандмауэра. При этом предполагается, что в вашей среде есть брандмауэр с соответствующей сетью, и вы добавили IP-адреса брандмауэра в диапазоны авторизации. Аналогичным образом, если вы выполнили принудительное туннелирование из подсети AKS в подсети с брандмауэром, это тоже хорошо, чем наличие Jumpbox в подсети кластера.

Добавьте другой IP-адрес в утвержденные диапазоны с помощью следующей команды.

```bash
# Retrieve your IP address
CURRENT_IP=$(dig @resolver1.opendns.com ANY myip.opendns.com +short)
# Add to AKS approved list
az aks update -g $RG -n $AKSNAME --api-server-authorized-ip-ranges $CURRENT_IP/32
```

>> [!NOTE]
> В приведенном выше примере в кластер добавляются IP-диапазоны авторизации сервера API. Чтобы отключить допустимые диапазоны IP-адресов, выполните команду AZ AKS Update и укажите пустой диапазон "". 

Другой вариант — использовать приведенную ниже команду в системах Windows для получения общедоступного IPv4-адреса или выполнить действия, описанные в разделе [Поиск IP-адреса](https://support.microsoft.com/en-gb/help/4026518/windows-10-find-your-ip-address).

```azurepowershell-interactive
Invoke-RestMethod http://ipinfo.io/json | Select -exp ip
```

Этот адрес можно также найти, выполнив поиск по слову "адрес моего IP-адреса" в Интернет-браузере.

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы включили IP-диапазоны разрешенных серверов API. Этот подход является одной из частей того, как можно запустить защищенный кластер AKS.

Дополнительные сведения см. в статьях [Основные понятия безопасности для приложений и кластеров в AKS][concepts-security] и рекомендации [по обеспечению безопасности и обновлениям кластера в AKS][operator-best-practices-cluster-security].

<!-- LINKS - external -->
[cni-networking]: https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md
[dev-spaces-ranges]: ../dev-spaces/configure-networking.md#aks-cluster-network-requirements
[kubenet]: https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/#kubenet

<!-- LINKS - internal -->
[az-aks-update]: /cli/azure/ext/aks-preview/aks#ext-aks-preview-az-aks-update
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-aks-show]: /cli/azure/aks#az_aks_show
[az-network-public-ip-list]: /cli/azure/network/public-ip#az-network-public-ip-list
[concepts-clusters-workloads]: concepts-clusters-workloads.md
[concepts-security]: concepts-security.md
[install-azure-cli]: /cli/azure/install-azure-cli
[operator-best-practices-cluster-security]: operator-best-practices-cluster-security.md
[route-tables]: ../virtual-network/manage-route-table.md
[standard-sku-lb]: load-balancer-standard.md
