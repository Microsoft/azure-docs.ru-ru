---
title: Выбор образов виртуальных машин Linux с помощью интерфейса командной строки Azure (Azure CLI)
description: Узнайте, как использовать Azure CLI для определения издателя, предложения, номера SKU и версии для образов виртуальных машин из Marketplace.
author: cynthn
ms.service: virtual-machines
ms.subservice: imaging
ms.topic: how-to
ms.date: 01/25/2019
ms.author: cynthn
ms.collection: linux
ms.openlocfilehash: efa0b91c9c0e43104f36017c3b1a1f3167190d63
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102562815"
---
# <a name="find-linux-vm-images-in-the-azure-marketplace-with-the-azure-cli"></a>Поиск образов виртуальных машин Linux в Azure Marketplace с помощью Azure CLI

В этой статье описывается, как с помощью Azure CLI находить образы виртуальных машин в Azure Marketplace. При создании виртуальной машины программными средствами с помощью CLI, шаблонов диспетчера ресурсов или других средств, эти сведения можно использовать для указания образа Marketplace.

Также просмотрите доступные образы и предложения в онлайн-магазине [Microsoft Azure Marketplace](https://azuremarketplace.microsoft.com/), на [портале Azure](https://portal.azure.com) или с помощью [Azure PowerShell](../windows/cli-ps-findimage.md). 

Убедитесь, что вы вошли в учетную запись Azure ( `az login` ).

[!INCLUDE [virtual-machines-common-image-terms](../../../includes/virtual-machines-common-image-terms.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

## <a name="deploy-from-a-vhd-using-purchase-plan-parameters"></a>Развертывание из виртуального жесткого диска с помощью параметров плана покупки

При наличии существующего виртуального жесткого диска, созданного с помощью платного образа Azure Marketplace, может потребоваться предоставить сведения о плане покупки при создании новой виртуальной машины из этого виртуального жесткого диска. 

Если у вас по-прежнему есть исходная виртуальная машина или другая виртуальная машина, созданная с помощью того же образа Marketplace, вы можете получить имя плана, издателя и сведения о продукте с помощью команды [AZ VM Get-instance-View](/cli/azure/vm#az_vm_get_instance_view). Этот пример получает виртуальную машину с именем *myVM* в группе ресурсов *myResourceGroup* , а затем отображает сведения о плане покупки.

```azurepowershell-interactive
az vm get-instance-view -g myResourceGroup -n myVM --query plan
```

Если вы не получили сведения о плане до удаления исходной виртуальной машины, можно [Отправить запрос в службу поддержки](https://ms.portal.azure.com/#create/Microsoft.Support). Им потребуется имя виртуальной машины, идентификатор подписки и метка времени операции удаления.

После получения сведений о плане можно создать новую виртуальную машину с помощью параметра, `--attach-os-disk` чтобы указать виртуальный жесткий диск.

```azurecli-interactive
az vm create \
   --resource-group myResourceGroup \
  --name myNewVM \
  --nics myNic \
  --size Standard_DS1_v2 --os-type Linux \
  --attach-os-disk myVHD \
  --plan-name planName \
  --plan-publisher planPublisher \
  --plan-product planProduct 
```

## <a name="deploy-a-new-vm-using-purchase-plan-parameters"></a>Развертывание новой виртуальной машины с помощью параметров плана покупки

Если у вас уже есть сведения об образе, его можно развернуть с помощью `az vm create` команды. В этом примере мы развертываем виртуальную машину с помощью образа RabbitMQ Certified by BitNami:

```azurecli
az group create --name myResourceGroupVM --location westus

az vm create --resource-group myResourceGroupVM --name myVM --image bitnami:rabbitmq:rabbitmq:latest --plan-name rabbitmq --plan-product rabbitmq --plan-publisher bitnami
```

Если появится сообщение о принятии условий образа, см. раздел [примите условия](#accept-the-terms) , приведенные далее в этой статье.

## <a name="list-popular-images"></a>Просмотр списка популярных образов

Выполните команду [az vm image list](/cli/azure/vm/image) без параметра `--all`, чтобы просмотреть список популярных образов виртуальных машин в Azure Marketplace. Например, чтобы увидеть кэшированный список популярных образов виртуальных машин в формате таблицы, выполните следующую команду:

```azurecli
az vm image list --output table
```

Выходные данные содержат URN образа (значение в столбце *URN*). При создании виртуальной машины с помощью одного из популярных образов в Marketplace в качестве альтернативы можно указать *UrnAlias*, сокращенную форму, например *UbuntuLTS*.

```output
You are viewing an offline list of images, use --all to retrieve an up-to-date list
Offer          Publisher               Sku                 Urn                                                             UrnAlias             Version
-------------  ----------------------  ------------------  --------------------------------------------------------------  -------------------  ---------
CentOS         OpenLogic               7.5                 OpenLogic:CentOS:7.5:latest                                     CentOS               latest
CoreOS         CoreOS                  Stable              CoreOS:CoreOS:Stable:latest                                     CoreOS               latest
Debian         credativ                8                   credativ:Debian:8:latest                                        Debian               latest
openSUSE-Leap  SUSE                    42.3                SUSE:openSUSE-Leap:42.3:latest                                  openSUSE-Leap        latest
RHEL           RedHat                  7-RAW               RedHat:RHEL:7-RAW:latest                                        RHEL                 latest
SLES           SUSE                    12-SP2              SUSE:SLES:12-SP2:latest                                         SLES                 latest
UbuntuServer   Canonical               16.04-LTS           Canonical:UbuntuServer:16.04-LTS:latest                         UbuntuLTS            latest
...
```

## <a name="find-specific-images"></a>Поиск определенных образов

Чтобы найти конкретный образ виртуальной машины в Marketplace, выполните команду `az vm image list` с параметром `--all`. Этот процесс занимает некоторое время и может возвращать большие объемы выходных данных. Поэтому вы можете отфильтровать список по `--publisher` или другому параметру. 

Например, приведенная ниже команда отображает предложения для Debian (помните, что без параметра `--all` поиск выполняется только в локальном кэше общих образов).

```azurecli
az vm image list --offer Debian --all --output table 

```

Частичные выходные данные приведены ниже. 

```output
Offer              Publisher    Sku                  Urn                                                    Version
-----------------  -----------  -------------------  -----------------------------------------------------  --------------
Debian             credativ     7                    credativ:Debian:7:7.0.201602010                        7.0.201602010
Debian             credativ     7                    credativ:Debian:7:7.0.201603020                        7.0.201603020
Debian             credativ     7                    credativ:Debian:7:7.0.201604050                        7.0.201604050
Debian             credativ     7                    credativ:Debian:7:7.0.201604200                        7.0.201604200
Debian             credativ     7                    credativ:Debian:7:7.0.201606280                        7.0.201606280
Debian             credativ     7                    credativ:Debian:7:7.0.201609120                        7.0.201609120
Debian             credativ     7                    credativ:Debian:7:7.0.201611020                        7.0.201611020
Debian             credativ     7                    credativ:Debian:7:7.0.201701180                        7.0.201701180
Debian             credativ     8                    credativ:Debian:8:8.0.201602010                        8.0.201602010
Debian             credativ     8                    credativ:Debian:8:8.0.201603020                        8.0.201603020
Debian             credativ     8                    credativ:Debian:8:8.0.201604050                        8.0.201604050
Debian             credativ     8                    credativ:Debian:8:8.0.201604200                        8.0.201604200
Debian             credativ     8                    credativ:Debian:8:8.0.201606280                        8.0.201606280
Debian             credativ     8                    credativ:Debian:8:8.0.201609120                        8.0.201609120
Debian             credativ     8                    credativ:Debian:8:8.0.201611020                        8.0.201611020
Debian             credativ     8                    credativ:Debian:8:8.0.201701180                        8.0.201701180
Debian             credativ     8                    credativ:Debian:8:8.0.201703150                        8.0.201703150
Debian             credativ     8                    credativ:Debian:8:8.0.201704110                        8.0.201704110
Debian             credativ     8                    credativ:Debian:8:8.0.201704180                        8.0.201704180
Debian             credativ     8                    credativ:Debian:8:8.0.201706190                        8.0.201706190
Debian             credativ     8                    credativ:Debian:8:8.0.201706210                        8.0.201706210
Debian             credativ     8                    credativ:Debian:8:8.0.201708040                        8.0.201708040
Debian             credativ     8                    credativ:Debian:8:8.0.201710090                        8.0.201710090
Debian             credativ     8                    credativ:Debian:8:8.0.201712040                        8.0.201712040
Debian             credativ     8                    credativ:Debian:8:8.0.201801170                        8.0.201801170
Debian             credativ     8                    credativ:Debian:8:8.0.201803130                        8.0.201803130
Debian             credativ     8                    credativ:Debian:8:8.0.201803260                        8.0.201803260
Debian             credativ     8                    credativ:Debian:8:8.0.201804020                        8.0.201804020
Debian             credativ     8                    credativ:Debian:8:8.0.201804150                        8.0.201804150
Debian             credativ     8                    credativ:Debian:8:8.0.201805160                        8.0.201805160
Debian             credativ     8                    credativ:Debian:8:8.0.201807160                        8.0.201807160
Debian             credativ     8                    credativ:Debian:8:8.0.201901221                        8.0.201901221
...
```

Примените аналогичные фильтры, используя параметры `--location`, `--publisher` и `--sku`. Можно искать частичные совпадения по фильтру. Так, с помощью параметра `--offer Deb` можно найти все образы Debian.

Если с помощью параметра `--location` не указать определенное расположение, то для региона возвращаются значения по умолчанию. (Задайте другое расположение по умолчанию с помощью команды `az configure --defaults location=<location>`.)

Например, следующая команда возвращает список всех номеров SKU для Debian 8 в Западной Европе:

```azurecli
az vm image list --location westeurope --offer Deb --publisher credativ --sku 8 --all --output table
```

Частичные выходные данные приведены ниже.

```output
Offer    Publisher    Sku                Urn                                              Version
-------  -----------  -----------------  -----------------------------------------------  -------------
Debian   credativ     8                  credativ:Debian:8:8.0.201602010                  8.0.201602010
Debian   credativ     8                  credativ:Debian:8:8.0.201603020                  8.0.201603020
Debian   credativ     8                  credativ:Debian:8:8.0.201604050                  8.0.201604050
Debian   credativ     8                  credativ:Debian:8:8.0.201604200                  8.0.201604200
Debian   credativ     8                  credativ:Debian:8:8.0.201606280                  8.0.201606280
Debian   credativ     8                  credativ:Debian:8:8.0.201609120                  8.0.201609120
Debian   credativ     8                  credativ:Debian:8:8.0.201611020                  8.0.201611020
Debian   credativ     8                  credativ:Debian:8:8.0.201701180                  8.0.201701180
Debian   credativ     8                  credativ:Debian:8:8.0.201703150                  8.0.201703150
Debian   credativ     8                  credativ:Debian:8:8.0.201704110                  8.0.201704110
Debian   credativ     8                  credativ:Debian:8:8.0.201704180                  8.0.201704180
Debian   credativ     8                  credativ:Debian:8:8.0.201706190                  8.0.201706190
Debian   credativ     8                  credativ:Debian:8:8.0.201706210                  8.0.201706210
Debian   credativ     8                  credativ:Debian:8:8.0.201708040                  8.0.201708040
Debian   credativ     8                  credativ:Debian:8:8.0.201710090                  8.0.201710090
Debian   credativ     8                  credativ:Debian:8:8.0.201712040                  8.0.201712040
Debian   credativ     8                  credativ:Debian:8:8.0.201801170                  8.0.201801170
Debian   credativ     8                  credativ:Debian:8:8.0.201803130                  8.0.201803130
Debian   credativ     8                  credativ:Debian:8:8.0.201803260                  8.0.201803260
Debian   credativ     8                  credativ:Debian:8:8.0.201804020                  8.0.201804020
Debian   credativ     8                  credativ:Debian:8:8.0.201804150                  8.0.201804150
Debian   credativ     8                  credativ:Debian:8:8.0.201805160                  8.0.201805160
Debian   credativ     8                  credativ:Debian:8:8.0.201807160                  8.0.201807160
Debian   credativ     8                  credativ:Debian:8:8.0.201901221                  8.0.201901221
...
```

## <a name="navigate-the-images"></a>Переход к образам
 
Еще один способ поиска образа в определенном расположении — это выполнить по-очереди команды [az vm image list-publishers](/cli/azure/vm/image), [az vm image list-offers](/cli/azure/vm/image) и [az vm image list-skus](/cli/azure/vm/image). С помощью этих команд определяются следующие значения:

1. Получить список издателей образов.
2. Получить список предложений нужного издателя.
3. Получить список номеров SKU для требуемого предложения.

Затем для выбранного номера SKU можно выбрать версию для развертывания.

Например, следующая команда позволяет получить список издателей образов в расположении "Западная часть США":

```azurecli
az vm image list-publishers --location westus --output table
```

Частичные выходные данные приведены ниже.

```output
Location    Name
----------  ----------------------------------------------------
westus      128technology
westus      1e
westus      4psa
westus      5nine-software-inc
westus      7isolutions
westus      a10networks
westus      abiquo
westus      accellion
westus      accessdata-group
westus      accops
westus      Acronis
westus      Acronis.Backup
westus      actian-corp
westus      actian_matrix
westus      actifio
westus      activeeon
westus      advantech-webaccess
westus      aerospike
westus      affinio
westus      aiscaler-cache-control-ddos-and-url-rewriting-
westus      akamai-technologies
westus      akumina
...
```

Используйте эти сведения, чтобы найти предложения от определенного издателя. Например, чтобы найти издателя *Canonical* в западной части США найдите предложения, запустив `azure vm image list-offers`. Укажите расположение и издателя, как показано в следующем примере:

```azurecli
az vm image list-offers --location westus --publisher Canonical --output table
```

Выходные данные:

```output
Location    Name
----------  -------------------------
westus      Ubuntu15.04Snappy
westus      Ubuntu15.04SnappyDocker
westus      UbunturollingSnappy
westus      UbuntuServer
westus      Ubuntu_Core
```
Вы видите, что издатель Canonical из западной части США предлагает *UbuntuServer* для Azure. Однако нам также нужны номера SKU. Чтобы получить их значения, выполните команду `azure vm image list-skus` и укажите расположение, издателя и обнаруженное предложение:

```azurecli
az vm image list-skus --location westus --publisher Canonical --offer UbuntuServer --output table
```

Выходные данные:

```output
Location    Name
----------  -----------------
westus      12.04.3-LTS
westus      12.04.4-LTS
westus      12.04.5-LTS
westus      14.04.0-LTS
westus      14.04.1-LTS
westus      14.04.2-LTS
westus      14.04.3-LTS
westus      14.04.4-LTS
westus      14.04.5-DAILY-LTS
westus      14.04.5-LTS
westus      16.04-DAILY-LTS
westus      16.04-LTS
westus      16.04.0-LTS
westus      18.04-DAILY-LTS
westus      18.04-LTS
westus      18.10
westus      18.10-DAILY
westus      19.04-DAILY
```

Наконец, выполните команду `az vm image list`, чтобы найти определенную версию номера SKU, например *18.04-LTS*:

```azurecli
az vm image list --location westus --publisher Canonical --offer UbuntuServer --sku 18.04-LTS --all --output table
```

Частичные выходные данные приведены ниже.

```output
Offer         Publisher    Sku        Urn                                               Version
------------  -----------  ---------  ------------------------------------------------  ---------------
UbuntuServer  Canonical    18.04-LTS  Canonical:UbuntuServer:18.04-LTS:18.04.201804262  18.04.201804262
UbuntuServer  Canonical    18.04-LTS  Canonical:UbuntuServer:18.04-LTS:18.04.201805170  18.04.201805170
UbuntuServer  Canonical    18.04-LTS  Canonical:UbuntuServer:18.04-LTS:18.04.201805220  18.04.201805220
UbuntuServer  Canonical    18.04-LTS  Canonical:UbuntuServer:18.04-LTS:18.04.201806130  18.04.201806130
UbuntuServer  Canonical    18.04-LTS  Canonical:UbuntuServer:18.04-LTS:18.04.201806170  18.04.201806170
UbuntuServer  Canonical    18.04-LTS  Canonical:UbuntuServer:18.04-LTS:18.04.201807240  18.04.201807240
UbuntuServer  Canonical    18.04-LTS  Canonical:UbuntuServer:18.04-LTS:18.04.201808060  18.04.201808060
UbuntuServer  Canonical    18.04-LTS  Canonical:UbuntuServer:18.04-LTS:18.04.201808080  18.04.201808080
UbuntuServer  Canonical    18.04-LTS  Canonical:UbuntuServer:18.04-LTS:18.04.201808140  18.04.201808140
UbuntuServer  Canonical    18.04-LTS  Canonical:UbuntuServer:18.04-LTS:18.04.201808310  18.04.201808310
UbuntuServer  Canonical    18.04-LTS  Canonical:UbuntuServer:18.04-LTS:18.04.201809110  18.04.201809110
UbuntuServer  Canonical    18.04-LTS  Canonical:UbuntuServer:18.04-LTS:18.04.201810030  18.04.201810030
UbuntuServer  Canonical    18.04-LTS  Canonical:UbuntuServer:18.04-LTS:18.04.201810240  18.04.201810240
UbuntuServer  Canonical    18.04-LTS  Canonical:UbuntuServer:18.04-LTS:18.04.201810290  18.04.201810290
UbuntuServer  Canonical    18.04-LTS  Canonical:UbuntuServer:18.04-LTS:18.04.201811010  18.04.201811010
UbuntuServer  Canonical    18.04-LTS  Canonical:UbuntuServer:18.04-LTS:18.04.201812031  18.04.201812031
UbuntuServer  Canonical    18.04-LTS  Canonical:UbuntuServer:18.04-LTS:18.04.201812040  18.04.201812040
UbuntuServer  Canonical    18.04-LTS  Canonical:UbuntuServer:18.04-LTS:18.04.201812060  18.04.201812060
UbuntuServer  Canonical    18.04-LTS  Canonical:UbuntuServer:18.04-LTS:18.04.201901140  18.04.201901140
UbuntuServer  Canonical    18.04-LTS  Canonical:UbuntuServer:18.04-LTS:18.04.201901220  18.04.201901220
...
```

Теперь по значению URN мы можем выбрать именно тот образ, который нам нужен. Передайте это значение с параметром `--image` при создании виртуальной машины с помощью команды [az vm create](/cli/azure/vm). Помните, что при необходимости можно заменить номер версии в URN словом latest. В этом случае всегда будет выбираться последняя версия образа. 

При развертывании виртуальной машины с помощью шаблона Resource Manager установите отдельные параметры образа в свойствах `imageReference`. Ознакомьтесь со статьей о [справочнике по шаблонам](/azure/templates/microsoft.compute/virtualmachines).

[!INCLUDE [virtual-machines-common-marketplace-plan](../../../includes/virtual-machines-common-marketplace-plan.md)]

### <a name="view-plan-properties"></a>Просмотр свойств плана

Чтобы просмотреть сведения о плане покупки образа, выполните команду [az vm image show](/cli/azure/image). Если в выходных данных значением свойства `plan` не является `null`, в образе появятся условия использования, которые необходимо принять перед программным развертыванием.

Например, образ Canonical Ubuntu Server 18.04 LTS не содержит дополнительных условий, так как в свойстве `plan` присутствует значение `null`.

```azurecli
az vm image show --location westus --urn Canonical:UbuntuServer:18.04-LTS:latest
```

Выходные данные:

```output
{
  "dataDiskImages": [],
  "id": "/Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Providers/Microsoft.Compute/Locations/westus/Publishers/Canonical/ArtifactTypes/VMImage/Offers/UbuntuServer/Skus/18.04-LTS/Versions/18.04.201901220",
  "location": "westus",
  "name": "18.04.201901220",
  "osDiskImage": {
    "operatingSystem": "Linux"
  },
  "plan": null,
  "tags": null
}
```

После запуска аналогичных команд для образа RabbitMQ, сертифицированного Bitnami, отображаются следующие свойства `plan`: `name`, `product` и `publisher`. (Некоторые образы также имеют `promotion code` свойство.) Чтобы развернуть этот образ, ознакомьтесь со следующими разделами, чтобы принять условия и включить программное развертывание.

```azurecli
az vm image show --location westus --urn bitnami:rabbitmq:rabbitmq:latest
```
Выходные данные:

```output
{
  "dataDiskImages": [],
  "id": "/Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Providers/Microsoft.Compute/Locations/westus/Publishers/bitnami/ArtifactTypes/VMImage/Offers/rabbitmq/Skus/rabbitmq/Versions/3.7.1901151016",
  "location": "westus",
  "name": "3.7.1901151016",
  "osDiskImage": {
    "operatingSystem": "Linux"
  },
  "plan": {
    "name": "rabbitmq",
    "product": "rabbitmq",
    "publisher": "bitnami"
  },
  "tags": null
}
```

## <a name="accept-the-terms"></a>Принятие условий

Чтобы просмотреть и принять условия лицензии, используйте команду [az vm image accept-terms](/cli/azure/vm/image?). При принятии условий в вашей подписке будет включено программное развертывание. Необходимо принять условия соглашения для каждой подписки в образе. Пример:

```azurecli
az vm image accept-terms --urn bitnami:rabbitmq:rabbitmq:latest
``` 

Выходные данные в лицензионных условиях содержат `licenseTextLink` и указывают, что значение параметра `accepted` равно `true`.

```output
{
  "accepted": true,
  "additionalProperties": {},
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/providers/Microsoft.MarketplaceOrdering/offertypes/bitnami/offers/rabbitmq/plans/rabbitmq",
  "licenseTextLink": "https://storelegalterms.blob.core.windows.net/legalterms/3E5ED_legalterms_BITNAMI%253a24RABBITMQ%253a24RABBITMQ%253a24IGRT7HHPIFOBV3IQYJHEN2O2FGUVXXZ3WUYIMEIVF3KCUNJ7GTVXNNM23I567GBMNDWRFOY4WXJPN5PUYXNKB2QLAKCHP4IE5GO3B2I.txt",
  "name": "rabbitmq",
  "plan": "rabbitmq",
  "privacyPolicyLink": "https://bitnami.com/privacy",
  "product": "rabbitmq",
  "publisher": "bitnami",
  "retrieveDatetime": "2019-01-25T20:37:49.937096Z",
  "signature": "XXXXXXLAZIK7ZL2YRV5JYQXONPV76NQJW3FKMKDZYCRGXZYVDGX6BVY45JO3BXVMNA2COBOEYG2NO76ONORU7ITTRHGZDYNJNXXXXXX",
  "type": "Microsoft.MarketplaceOrdering/offertypes"
}
```

## <a name="next-steps"></a>Дальнейшие действия
Инструкции по быстрому созданию виртуальной машины на основе данных образа см. в статье [Создание виртуальных машин Linux и управление ими с помощью Azure CLI](tutorial-manage-vm.md).
