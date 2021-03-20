---
title: Настройка приватного канала
description: Настройте закрытую конечную точку в реестре контейнеров и разрешите доступ через частную ссылку в локальной виртуальной сети. Доступ к частным ссылкам является компонентом уровня служб Premium.
ms.topic: article
ms.date: 10/01/2020
ms.openlocfilehash: 3193c65a2021d29f03bd9ae6cbc00fd6c349d9bf
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93342306"
---
# <a name="connect-privately-to-an-azure-container-registry-using-azure-private-link"></a>Подключение в частном порядке к реестру контейнеров Azure с помощью частной ссылки Azure


Ограничьте доступ к реестру, назначив частные IP-адреса виртуальной сети конечным точкам реестра и используя [частную связь Azure](../private-link/private-link-overview.md). Сетевой трафик между клиентами в виртуальной сети и частными конечными точками реестра проходит по виртуальной сети и частной связи в магистральной сети Майкрософт, что устраняет уязвимость в общедоступном Интернете. Частная ссылка также обеспечивает доступ к частному реестру из локальной среды через частный пиринг [Azure ExpressRoute](../expressroute/expressroute-introduction.MD) или [VPN-шлюз](../vpn-gateway/vpn-gateway-about-vpngateways.md).

Вы можете [настроить параметры DNS](../private-link/private-endpoint-overview.md#dns-configuration) для частных конечных точек реестра, чтобы параметры разрешили доступ к выделенному ЧАСТНОМУ IP-адресу реестра. При такой конфигурации DNS клиенты и службы в сети смогут по-прежнему обращаться к реестру по полному доменному имени реестра, например *myregistry.azurecr.io*. 

Эта возможность доступна на уровне **Премиум** службы реестра контейнеров. В настоящее время для реестра может быть настроено не более 10 частных конечных точек. Сведения об уровнях служб реестра и ограничениях см. в статье [Уровни службы Реестра контейнеров Azure](container-registry-skus.md).

[!INCLUDE [container-registry-scanning-limitation](../../includes/container-registry-scanning-limitation.md)]

## <a name="prerequisites"></a>Предварительные требования

* Для выполнения действий с Azure CLI, описанных в этой статье, рекомендуется использовать Azure CLI версии 2.6.0 или более поздней. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][azure-cli]. Кроме того, можно воспользоваться запуском в [Azure Cloud Shell](../cloud-shell/quickstart.md).
* Если у вас еще нет реестра контейнеров, создайте его (требуется уровень Premium) и [импортируйте](container-registry-import-images.md) пример общедоступного образа, например `mcr.microsoft.com/hello-world` из реестра контейнеров (Майкрософт). Создать реестр можно на [портале Azure][quickstart-portal] или с помощью [Azure CLI][quickstart-cli].
* Чтобы настроить доступ к реестру, используя приватный канал из другой подписки Azure, необходимо зарегистрировать в такой подписке поставщик ресурсов для Реестра контейнеров Azure. Пример:

  ```azurecli
  az account set --subscription <Name or ID of subscription of private link>

  az provider register --namespace Microsoft.ContainerRegistry
  ``` 

В примерах Azure CLI в этой статье используются следующие переменные среды. Подставьте соответствующие значения для вашей среды. Все примеры форматированы для оболочки Bash.

```bash
REGISTRY_NAME=<container-registry-name>
REGISTRY_LOCATION=<container-registry-location> # Azure region such as westeurope where registry created
RESOURCE_GROUP=<resource-group-name>
VM_NAME=<virtual-machine-name>
```

[!INCLUDE [Set up Docker-enabled VM](../../includes/container-registry-docker-vm-setup.md)]

## <a name="set-up-private-link---cli"></a>Настройка приватного канала (интерфейс командной строки)

### <a name="get-network-and-subnet-names"></a>Получение имен сети и подсети

Если вам неизвестны имена виртуальной сети и подсети, вам потребуется их уточнить для настройки приватного канала. В этом примере используется одна и та же подсеть для виртуальной машины и частной конечной точки реестра. Однако во многих сценариях конечная точка настраивается в отдельной подсети. 

При создании виртуальной машины Azure по умолчанию создает в той же группе ресурсов виртуальную сеть. Имя для этой виртуальной сети создается автоматически на основе имени виртуальной машины. Например, если вы присвоите виртуальной машине имя *myDockerVM*, то созданная по умолчанию виртуальная сеть будет называться *myDockerVMVNET*, а подсеть в ней — *myDockerVMSubnet*. Задайте эти значения в переменных среды с помощью команды [az network vnet list][az-network-vnet-list].

```azurecli
NETWORK_NAME=$(az network vnet list \
  --resource-group $RESOURCE_GROUP \
  --query '[].{Name: name}' --output tsv)

SUBNET_NAME=$(az network vnet list \
  --resource-group $RESOURCE_GROUP \
  --query '[].{Subnet: subnets[0].name}' --output tsv)

echo NETWORK_NAME=$NETWORK_NAME
echo SUBNET_NAME=$SUBNET_NAME
```

### <a name="disable-network-policies-in-subnet"></a>Отключение сетевых политик в подсети

[Отключите сетевые политики](../private-link/disable-private-endpoint-network-policy.md), например группы безопасности сети в подсети для частной конечной точки. Обновите конфигурацию подсети с помощью команды [az network vnet subnet update][az-network-vnet-subnet-update].

```azurecli
az network vnet subnet update \
 --name $SUBNET_NAME \
 --vnet-name $NETWORK_NAME \
 --resource-group $RESOURCE_GROUP \
 --disable-private-endpoint-network-policies
```

### <a name="configure-the-private-dns-zone"></a>Настройка частной зоны DNS

Создайте [закрытую зону DNS](../dns/private-dns-privatednszone.md) для закрытого домена реестра контейнеров Azure. На следующих шагах вы создадите записи DNS для вашего домена реестра в этой зоне DNS.

Чтобы использовать частную зону для переопределения стандартного разрешения DNS для реестра контейнеров Azure, зона должна иметь имя **privatelink.azurecr.io**. Чтобы создать частную зону, выполните следующую команду [az network private-dns zone create][az-network-private-dns-zone-create].

```azurecli
az network private-dns zone create \
  --resource-group $RESOURCE_GROUP \
  --name "privatelink.azurecr.io"
```

### <a name="create-an-association-link"></a>Создание канала связи

Воспользуйтесь командой [az network private-dns link vnet create][az-network-private-dns-link-vnet-create], чтобы связать частную зону с виртуальной сетью. В этом примере создается канал с именем *myDNSLink*.

```azurecli
az network private-dns link vnet create \
  --resource-group $RESOURCE_GROUP \
  --zone-name "privatelink.azurecr.io" \
  --name MyDNSLink \
  --virtual-network $NETWORK_NAME \
  --registration-enabled false
```

### <a name="create-a-private-registry-endpoint"></a>Создание частной конечной точки реестра

В этом разделе рассказывается о том, как создать частную конечную точку реестра в виртуальной сети. Сначала получите идентификатор ресурса реестра.

```azurecli
REGISTRY_ID=$(az acr show --name $REGISTRY_NAME \
  --query 'id' --output tsv)
```

Чтобы создать частную конечную точку реестра, воспользуйтесь командой [az network private-endpoint create][az-network-private-endpoint-create].

В следующем примере создается конечная точка *myPrivateEndpoint* и подключение к службе *myConnection*. Чтобы указать ресурс реестра контейнеров для конечной точки, передайте параметр `--group-ids registry`.

```azurecli
az network private-endpoint create \
    --name myPrivateEndpoint \
    --resource-group $RESOURCE_GROUP \
    --vnet-name $NETWORK_NAME \
    --subnet $SUBNET_NAME \
    --private-connection-resource-id $REGISTRY_ID \
    --group-ids registry \
    --connection-name myConnection
```

### <a name="get-private-ip-addresses"></a>Получение частных IP-адресов

Выполните запрос конечной точки [az network private-endpoint show][az-network-private-endpoint-show] для идентификатора сетевого интерфейса:

```azurecli
NETWORK_INTERFACE_ID=$(az network private-endpoint show \
  --name myPrivateEndpoint \
  --resource-group $RESOURCE_GROUP \
  --query 'networkInterfaces[0].id' \
  --output tsv)
```

В этом примере с сетевым интерфейсом связаны два частных IP-адреса для реестра контейнеров: один для самого реестра, а другой для конечной точки данных реестра. Следующие команды [az resource show][az-resource-show] получают частные IP-адреса для реестра контейнеров и конечной точки данных реестра.

```azurecli
PRIVATE_IP=$(az resource show \
  --ids $NETWORK_INTERFACE_ID \
  --api-version 2019-04-01 \
  --query 'properties.ipConfigurations[1].properties.privateIPAddress' \
  --output tsv)

DATA_ENDPOINT_PRIVATE_IP=$(az resource show \
  --ids $NETWORK_INTERFACE_ID \
  --api-version 2019-04-01 \
  --query 'properties.ipConfigurations[0].properties.privateIPAddress' \
  --output tsv)
```

> [!NOTE]
> Если реестр [геореплицирован](container-registry-geo-replication.md), запросите дополнительные конечные точки данных для каждой его реплики.

### <a name="create-dns-records-in-the-private-zone"></a>Создание записей DNS в частной зоне

Следующие команды создают записи DNS в частной зоне для конечной точки реестра и ее конечной точки данных. Например, если у вас есть реестр с именем *myregistry* в регионе *westeurope*, именами конечных точек будут: `myregistry.azurecr.io` и `myregistry.westeurope.data.azurecr.io`. 

> [!NOTE]
> Если реестр [геореплицирован](container-registry-geo-replication.md), создайте дополнительные записи DNS для IP-адреса конечной точки данных каждой реплики.

Сначала воспользуйтесь командой [az network private-dns record-set a create][az-network-private-dns-record-set-a-create], чтобы создать пустой набор записей A для конечной точки реестра и конечной точки данных.

```azurecli
az network private-dns record-set a create \
  --name $REGISTRY_NAME \
  --zone-name privatelink.azurecr.io \
  --resource-group $RESOURCE_GROUP

# Specify registry region in data endpoint name
az network private-dns record-set a create \
  --name ${REGISTRY_NAME}.${REGISTRY_LOCATION}.data \
  --zone-name privatelink.azurecr.io \
  --resource-group $RESOURCE_GROUP
```

Чтобы создать записи A для конечной точки реестра и конечной точки данных, воспользуйтесь командой [az network private-dns record-set a add-record][az-network-private-dns-record-set-a-add-record].

```azurecli
az network private-dns record-set a add-record \
  --record-set-name $REGISTRY_NAME \
  --zone-name privatelink.azurecr.io \
  --resource-group $RESOURCE_GROUP \
  --ipv4-address $PRIVATE_IP

# Specify registry region in data endpoint name
az network private-dns record-set a add-record \
  --record-set-name ${REGISTRY_NAME}.${REGISTRY_LOCATION}.data \
  --zone-name privatelink.azurecr.io \
  --resource-group $RESOURCE_GROUP \
  --ipv4-address $DATA_ENDPOINT_PRIVATE_IP
```

Приватный канал настроен и готов к использованию.

## <a name="set-up-private-link---portal"></a>Настройка приватного канала (портал)

Настройте приватный канал при создании реестра или добавьте его в имеющийся реестр. На следующих шагах предполагается, что вы уже настроили виртуальную сеть и подсеть с виртуальной машиной для тестирования. Вы также можете [создать новую виртуальную сеть и подсеть](../virtual-network/quick-create-portal.md).

### <a name="create-a-private-endpoint---new-registry"></a>Создание частной конечной точки — новый реестр

1. При создании реестра на портале на вкладке **Основные сведения** в разделе **SKU** выберите **Премиум**.
1. Перейдите на вкладку **Сеть**.
1. В разделе **Сетевое подключение** выберите **Частная конечная точка** >  **+ Добавить**.
1. Введите или выберите следующие сведения.

    | Параметр | Значение |
    | ------- | ----- |
    | Подписка | Выберите свою подписку. |
    | Группа ресурсов | Введите имя имеющейся группы или создайте новую.|
    | Имя | Укажите уникальное имя. |
    | Подресурс |Выберите **реестр**.|
    | **Сеть** | |
    | Виртуальная сеть| Выберите виртуальную сеть, в которой развернута виртуальная машина, например *myDockerVMVNET*. |
    | Подсеть | Выберите подсеть, например *myDockerVMSubnet*, где развернута виртуальная машина. |
    |**Интеграция с Частной зоной DNS**||
    |Интеграция с частной зоной DNS |Выберите **Да**. |
    |Частная зона DNS |Выберите *(Создать) privatelink.azurecr.io*. |
    |||
1. Настройте остальные параметры реестра, а затем нажмите кнопку **Проверить и создать**.

  ![Создание реестра с частной конечной точкой](./media/container-registry-private-link/private-link-create-portal.png)

### <a name="create-a-private-endpoint---existing-registry"></a>Создание частной конечной точки — имеющийся реестр

1. На портале перейдите к нужному реестру контейнеров.
1. В разделе **Параметры** выберите **Сеть**.
1. На вкладке **Частные конечные точки** выберите **+ Частная конечная точка**.
1. На вкладке **Основные сведения** введите или выберите следующие значения параметров.

    | Параметр | Значение |
    | ------- | ----- |
    | **Сведения о проекте** | |
    | Подписка | Выберите свою подписку. |
    | Группа ресурсов | Введите имя имеющейся группы или создайте новую.|
    | **Сведения об экземпляре** |  |
    | Имя | Введите имя. |
    |Регион|Выберите регион.|
    |||
5. По завершении выберите **Next: Ресурс**.
6. Введите или выберите следующие сведения.

    | Параметр | Значение |
    | ------- | ----- |
    |Метод подключения  | Выберите **Подключиться к ресурсу Azure в моем каталоге**.|
    | Подписка| Выберите свою подписку. |
    | Тип ресурса | Выберите **Microsoft.ContainerRegistry/registries**. |
    | Ресурс |Выберите имя реестра.|
    |Целевой подресурс |Выберите **реестр**.|
    |||
7. По завершении выберите **Next: Конфигурация**.
8. Введите или выберите указанные ниже сведения.

    | Параметр | Значение |
    | ------- | ----- |
    |**Сеть**| |
    | Виртуальная сеть| Выберите виртуальную сеть, в которой развернута виртуальная машина, например *myDockerVMVNET*. |
    | Подсеть | Выберите подсеть, например *myDockerVMSubnet*, где развернута виртуальная машина. |
    |**Интеграция с Частной зоной DNS**||
    |Интеграция с частной зоной DNS |Выберите **Да**. |
    |Частная зона DNS |Выберите *(Создать) privatelink.azurecr.io*. |
    |||

1. Выберите **Review + create** (Просмотреть и создать). Вы будете перенаправлены на страницу **Просмотр и создание**, где Azure проверит вашу конфигурацию. 
2. При появлении сообщения **Проверка пройдена** нажмите кнопку **Создать**.

После создания частной конечной точки параметры DNS в частной зоне отображаются на портале на странице **Частные конечные точки**.

1. На портале перейдите к нужному реестру контейнеров, затем выберите **Параметры > Сеть**.
1. На вкладке **Частные конечные точки** выберите созданную частную конечную точку.
1. На странице **Обзор** проверьте параметры канала и настраиваемые параметры DNS.

  ![Параметры DNS конечной точки](./media/container-registry-private-link/private-endpoint-overview.png)

Приватный канал настроен и готов к использованию.

## <a name="disable-public-access"></a>Отключение общего доступа

Во многих сценариях следует отключить доступ к реестру из общедоступных сетей. Эта конфигурация предотвращает доступ клиентов, находящихся за пределами виртуальной сети, к конечным точкам реестра. 

### <a name="disable-public-access---cli"></a>Отключение общего доступа (интерфейс командной строки)

Чтобы отключить общий доступ с помощью Azure CLI, воспользуйтесь командой [az acr update][az-acr-update] и задайте для `--public-network-enabled` значение `false`. 

> [!NOTE]
> Для использования аргумента `public-network-enabled` нужна версия Azure CLI не ниже 2.6.0. 

```azurecli
az acr update --name $REGISTRY_NAME --public-network-enabled false
```


### <a name="disable-public-access---portal"></a>Отключение общего доступа (портал)

1. На портале перейдите к нужному реестру контейнеров, затем выберите **Параметры > Сеть**.
1. На вкладке **Открытый доступ** выберите для параметра **Разрешение доступа из общедоступных сетей** значение **Отключено**. Затем нажмите кнопку **Save** (Сохранить).

## <a name="validate-private-link-connection"></a>Проверка подключения приватного канала

Необходимо убедиться, что ресурсы в одной подсети частной конечной точки подключаются к реестру через частный IP-адрес и правильно интегрируются с частной зоной DNS.

Чтобы проверить подключение приватного канала, подключитесь по протоколу SSH к виртуальной машине, которую вы настроили в виртуальной сети.

Запустите служебную программу, например `nslookup` или, `dig` чтобы найти IP-адрес реестра по частному каналу. Пример:

```bash
dig $REGISTRY_NAME.azurecr.io
```

В примере выходных данных показан IP-адрес реестра в адресном пространстве подсети.

```console
[...]
; <<>> DiG 9.11.3-1ubuntu1.13-Ubuntu <<>> myregistry.azurecr.io
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 52155
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;myregistry.azurecr.io.         IN      A

;; ANSWER SECTION:
myregistry.azurecr.io.  1783    IN      CNAME   myregistry.privatelink.azurecr.io.
myregistry.privatelink.azurecr.io. 10 IN A      10.0.0.7

[...]
```

Сравните этот результат с общедоступным IP-адресом в выходных данных `dig` для того же реестра с подключением через общедоступную конечную точку.

```console
[...]
;; ANSWER SECTION:
myregistry.azurecr.io.  2881    IN  CNAME   myregistry.privatelink.azurecr.io.
myregistry.privatelink.azurecr.io. 2881 IN CNAME xxxx.xx.azcr.io.
xxxx.xx.azcr.io.    300 IN  CNAME   xxxx-xxx-reg.trafficmanager.net.
xxxx-xxx-reg.trafficmanager.net. 300 IN CNAME   xxxx.westeurope.cloudapp.azure.com.
xxxx.westeurope.cloudapp.azure.com. 10  IN A 20.45.122.144

[...]
```

### <a name="registry-operations-over-private-link"></a>Выполнение операций с реестром по приватному каналу

Убедитесь, что вы можете выполнять операции с реестром из виртуальной машины в подсети. Создайте SSH-подключение к виртуальной машине и воспользуйтесь командой [az acr login][az-acr-login], чтобы войти в реестр. В зависимости от конфигурации виртуальной машины, возможно, к следующим командам потребуется добавить префикс `sudo`.

```bash
az acr login --name $REGISTRY_NAME
```

Выполните операции с реестром, например, воспользуйтесь командой `docker pull`, чтобы извлечь пример образа. Замените `hello-world:v1` соответствующими образом и тегом для вашего реестра, а также добавьте в качестве префикса имя сервера для входа в реестр (в нижнем регистре):

```bash
docker pull myregistry.azurecr.io/hello-world:v1
``` 

Средство Docker должно успешно передать образ на виртуальную машину.

## <a name="manage-private-endpoint-connections"></a>Управление подключениями к частным конечным точкам

Подключениями к частным конечным точкам реестра можно управлять с помощью портала Azure или с помощью команд в группе команд [az acr private-endpoint-connection][az-acr-private-endpoint-connection]. Вы можете выполнять следующие операции: утверждение, удаление, перечисление, отклонение или отображение сведений о подключениях к частным конечным точкам реестра.

Например, чтобы получить список подключений к частным конечным точкам в реестре, выполните команду [az acr private-endpoint-connection list][az-acr-private-endpoint-connection-list]. Пример:

```azurecli
az acr private-endpoint-connection list \
  --registry-name $REGISTRY_NAME 
```

При настройке подключения к частной конечной точке с помощью действий, описанных в этой статье, реестр автоматически принимает подключения от клиентов и служб, имеющих разрешения RBAC Azure для реестра. Вы можете настроить для конечной точки обязательное утверждение подключений вручную. Сведения о том, как утверждать и отклонять подключения к частным конечным точкам, см. в статье [Управление подключением к частной конечной точке](../private-link/manage-private-endpoint.md).

## <a name="add-zone-records-for-replicas"></a>Добавление записей зоны для реплик

Как показано в этой статье, при добавлении подключения к частной конечной точке в реестр создаются записи DNS в `privatelink.azurecr.io` зоне для реестра и его конечных точек данных в регионах, где выполняется [репликация](container-registry-geo-replication.md)реестра. 

Если позднее будет добавлена новая реплика, потребуется вручную добавить новую запись зоны для конечной точки данных в этом регионе. Например, если вы создаете реплику *myregistry* в расположении *northeurope*, добавьте запись зоны для `myregistry.northeurope.data.azurecr.io`. Инструкции см. в этой статье в разделе [Создание записей DNS в частной зоне](#create-dns-records-in-the-private-zone).

## <a name="dns-configuration-options"></a>Параметры конфигурации DNS

Частная конечная точка в этом примере интегрируется с частной зоной DNS, связанной с базовой виртуальной сетью. Эта программа установки использует службу DNS, предоставляемую Azure, чтобы разрешить общедоступное полное доменное имя реестра для его частного IP-адреса в виртуальной сети. 

Частная ссылка поддерживает дополнительные сценарии настройки DNS, которые используют частную зону, включая пользовательские решения DNS. Например, у вас может быть пользовательское решение DNS, развернутое в виртуальной сети или локально в сети, которое подключается к виртуальной сети с помощью VPN-шлюза или Azure ExpressRoute. 

Чтобы разрешить общедоступное полное доменное имя в реестре для частного IP-адреса в этих сценариях, необходимо настроить сервер пересылки на уровне сервера в службе Azure DNS (168.63.129.16). Точные параметры конфигурации и действия зависят от имеющихся сетей и DNS. Примеры см. в разделе [Конфигурация DNS для частной конечной точки Azure](../private-link/private-endpoint-dns.md).

> [!IMPORTANT]
> Если для обеспечения высокой доступности вы создали частные конечные точки в нескольких регионах, мы рекомендуем использовать отдельную группу ресурсов в каждом регионе и разместить в ней виртуальную сеть и связанную с ней зону DNS. Такая конфигурация также предотвращает непредсказуемое разрешение DNS, вызванное предоставлением общего доступа к одной частной зоне DNS.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы создали все ресурсы Azure в одной группе ресурсов и они вам больше не нужны, вы можете удалить все ресурсы сразу одной командой [az group delete](/cli/azure/group).

```azurecli
az group delete --name $RESOURCE_GROUP
```

Чтобы очистить ресурсы с помощью портала, перейдите к группе ресурсов. Когда загрузится группа ресурсов, щелкните действие **Удалить группу ресурсов**, чтобы удалить эту группу ресурсов вместе со всем содержимым.

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о Приватных каналах см. в документации по [Приватным каналам Azure](../private-link/private-link-overview.md).
* Если вам нужно настроить правила для доступа к реестру, расположенному за клиентским брандмауэром, воспользуйтесь руководством [Настройка правил для доступа к реестру контейнеров Azure за брандмауэром](container-registry-firewall-access-rules.md).

<!-- LINKS - external -->
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-login]: https://docs.docker.com/engine/reference/commandline/login/
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/

<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-create]: /cli/azure/acr#az-acr-create
[az-acr-show]: /cli/azure/acr#az-acr-show
[az-acr-repository-show]: /cli/azure/acr/repository#az-acr-repository-show
[az-acr-repository-list]: /cli/azure/acr/repository#az-acr-repository-list
[az-acr-login]: /cli/azure/acr#az-acr-login
[az-acr-private-endpoint-connection]: /cli/azure/acr/private-endpoint-connection
[az-acr-private-endpoint-connection-list]: /cli/azure/acr/private-endpoint-connection#az-acr-private-endpoint-connection-list
[az-acr-private-endpoint-connection-approve]: /cli/azure/acr/private-endpoint-connection#az-acr-private-endpoint-connection-approve
[az-acr-update]: /cli/azure/acr#az-acr-update
[az-group-create]: /cli/azure/group
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create
[az-vm-create]: /cli/azure/vm#az-vm-create
[az-network-vnet-subnet-show]: /cli/azure/network/vnet/subnet/#az-network-vnet-subnet-show
[az-network-vnet-subnet-update]: /cli/azure/network/vnet/subnet/#az-network-vnet-subnet-update
[az-network-vnet-list]: /cli/azure/network/vnet/#az-network-vnet-list
[az-network-private-endpoint-create]: /cli/azure/network/private-endpoint#az-network-private-endpoint-create
[az-network-private-endpoint-show]: /cli/azure/network/private-endpoint#az-network-private-endpoint-show
[az-network-private-dns-zone-create]: /cli/azure/network/private-dns/zone#az-network-private-dns-zone-create
[az-network-private-dns-link-vnet-create]: /cli/azure/network/private-dns/link/vnet#az-network-private-dns-link-vnet-create
[az-network-private-dns-record-set-a-create]: /cli/azure/network/private-dns/record-set/a#az-network-private-dns-record-set-a-create
[az-network-private-dns-record-set-a-add-record]: /cli/azure/network/private-dns/record-set/a#az-network-private-dns-record-set-a-add-record
[az-resource-show]: /cli/azure/resource#az-resource-show
[quickstart-portal]: container-registry-get-started-portal.md
[quickstart-cli]: container-registry-get-started-azure-cli.md
[azure-portal]: https://portal.azure.com
