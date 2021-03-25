---
title: Поиск и использование сведений о плане покупки Marketplace с помощью интерфейса командной строки
description: Узнайте, как использовать Azure CLI, чтобы найти образы URN и параметры плана приобретения, такие как издатель, предложение, номер SKU и версия, для образов виртуальных машин Marketplace.
author: cynthn
ms.service: virtual-machines
ms.subservice: imaging
ms.topic: how-to
ms.date: 03/22/2021
ms.author: cynthn
ms.collection: linux
ms.custom: contperf-fy21q3-portal
ms.openlocfilehash: 70cb4cc54c6f9a376d3bd38dc8bb6cd3a059a20c
ms.sourcegitcommit: a8ff4f9f69332eef9c75093fd56a9aae2fe65122
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105022856"
---
# <a name="find-azure-marketplace-image-information-using-the-azure-cli"></a>Поиск сведений об образе Azure Marketplace с помощью Azure CLI

В этой статье описывается, как с помощью Azure CLI находить образы виртуальных машин в Azure Marketplace. При создании виртуальной машины программными средствами с помощью CLI, шаблонов диспетчера ресурсов или других средств, эти сведения можно использовать для указания образа Marketplace.

Вы также можете просмотреть доступные изображения и предложения с помощью [Azure Marketplace](https://azuremarketplace.microsoft.com/) или  [Azure PowerShell](../windows/cli-ps-findimage.md). 

## <a name="terminology"></a>Терминология

Атрибуты образа Marketplace в Azure:

* **Издатель.** Организация, создавшая образ. Примеры: Canonical, MicrosoftWindowsServer.
* **Предложение.** Имя группы связанных образов, созданных издателем. Примеры: UbuntuServer, WindowsServer
* **Номер SKU.** Экземпляр предложения, например основной выпуск дистрибутива. Примеры: 18,04-LTS, 2019-Datacenter
* **Версия.** Номер версии SKU образа. 

Эти значения можно передавать по отдельности или в виде *URN* образа, объединяя значения, разделенные двоеточием (:). Например: *Издатель*:*предложение*:*SKU*:*версия*. Вы можете заменить номер версии в URN на, `latest` чтобы использовать последнюю версию образа. 

Если издатель образа предоставляет дополнительные условия лицензии и покупки, необходимо принять их, прежде чем можно будет использовать образ.  Дополнительные сведения см. [в разделе Проверка сведений о плане покупки](#check-the-purchase-plan-information).



## <a name="list-popular-images"></a>Просмотр списка популярных образов

Выполните команду [az vm image list](/cli/azure/vm/image) без параметра `--all`, чтобы просмотреть список популярных образов виртуальных машин в Azure Marketplace. Например, чтобы увидеть кэшированный список популярных образов виртуальных машин в формате таблицы, выполните следующую команду:

```azurecli
az vm image list --output table
```

Выходные данные включают URN образа. Вы также можете использовать *урналиас* , который представляет собой сокращенную версию, созданную для популярных изображений, таких как *UbuntuLTS*.

```output
Offer          Publisher               Sku                 Urn                                                             UrnAlias             Version
-------------  ----------------------  ------------------  --------------------------------------------------------------  -------------------  ---------
CentOS         OpenLogic               7.5                 OpenLogic:CentOS:7.5:latest                                     CentOS               latest
CoreOS         CoreOS                  Stable              CoreOS:CoreOS:Stable:latest                                     CoreOS               latest
debian-10      Debian                  10                  Debian:debian-10:10:latest                                      Debian               latest
openSUSE-Leap  SUSE                    42.3                SUSE:openSUSE-Leap:42.3:latest                                  openSUSE-Leap        latest
RHEL           RedHat                  7-LVM               RedHat:RHEL:7-LVM:latest                                        RHEL                 latest
SLES           SUSE                    15                  SUSE:SLES:15:latest                                             SLES                 latest
UbuntuServer   Canonical               18.04-LTS           Canonical:UbuntuServer:18.04-LTS:latest                         UbuntuLTS            latest
WindowsServer  MicrosoftWindowsServer  2019-Datacenter     MicrosoftWindowsServer:WindowsServer:2019-Datacenter:latest     Win2019Datacenter    latest
WindowsServer  MicrosoftWindowsServer  2016-Datacenter     MicrosoftWindowsServer:WindowsServer:2016-Datacenter:latest     Win2016Datacenter    latest
WindowsServer  MicrosoftWindowsServer  2012-R2-Datacenter  MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:latest  Win2012R2Datacenter  latest
WindowsServer  MicrosoftWindowsServer  2012-Datacenter     MicrosoftWindowsServer:WindowsServer:2012-Datacenter:latest     Win2012Datacenter    latest
WindowsServer  MicrosoftWindowsServer  2008-R2-SP1         MicrosoftWindowsServer:WindowsServer:2008-R2-SP1:latest         Win2008R2SP1         latest
```

## <a name="find-specific-images"></a>Поиск определенных образов

Чтобы найти конкретный образ виртуальной машины в Marketplace, выполните команду `az vm image list` с параметром `--all`. Этот процесс занимает некоторое время и может возвращать большие объемы выходных данных. Поэтому вы можете отфильтровать список по `--publisher` или другому параметру. 

Например, приведенная ниже команда отображает предложения для Debian (помните, что без параметра `--all` поиск выполняется только в локальном кэше общих образов).

```azurecli
az vm image list --offer Debian --all --output table 
```

Частичные выходные данные приведены ниже. 

```output
Offer                                    Publisher                         Sku                                      Urn                                                                                                   Version
---------------------------------------  --------------------------------  ---------------------------------------  ----------------------------------------------------------------------------------------------------  --------------
apache-solr-on-debian                    apps-4-rent                       apache-solr-on-debian                    apps-4-rent:apache-solr-on-debian:apache-solr-on-debian:1.0.0                                         1.0.0
atomized-h-debian10-v1                   atomizedinc1587939464368          hdebian10plan                            atomizedinc1587939464368:atomized-h-debian10-v1:hdebian10plan:1.0.0                                   1.0.0
atomized-h-debian9-v1                    atomizedinc1587939464368          hdebian9plan                             atomizedinc1587939464368:atomized-h-debian9-v1:hdebian9plan:1.0.0                                     1.0.0
atomized-r-debian10-v1                   atomizedinc1587939464368          rdebian10plan                            atomizedinc1587939464368:atomized-r-debian10-v1:rdebian10plan:1.0.0                                   1.0.0
atomized-r-debian9-v1                    atomizedinc1587939464368          rdebian9plan                             atomizedinc1587939464368:atomized-r-debian9-v1:rdebian9plan:1.0.0                                     1.0.0
cis-debian-linux-10-l1                   center-for-internet-security-inc  cis-debian10-l1                          center-for-internet-security-inc:cis-debian-linux-10-l1:cis-debian10-l1:1.0.7                         1.0.7
cis-debian-linux-10-l1                   center-for-internet-security-inc  cis-debian10-l1                          center-for-internet-security-inc:cis-debian-linux-10-l1:cis-debian10-l1:1.0.8                         1.0.8
cis-debian-linux-10-l1                   center-for-internet-security-inc  cis-debian10-l1                          center-for-internet-security-inc:cis-debian-linux-10-l1:cis-debian10-l1:1.0.9                         1.0.9
cis-debian-linux-9-l1                    center-for-internet-security-inc  cis-debian9-l1                           center-for-internet-security-inc:cis-debian-linux-9-l1:cis-debian9-l1:1.0.18                          1.0.18
cis-debian-linux-9-l1                    center-for-internet-security-inc  cis-debian9-l1                           center-for-internet-security-inc:cis-debian-linux-9-l1:cis-debian9-l1:1.0.19                          1.0.19
cis-debian-linux-9-l1                    center-for-internet-security-inc  cis-debian9-l1                           center-for-internet-security-inc:cis-debian-linux-9-l1:cis-debian9-l1:1.0.20                          1.0.20
apache-web-server-with-debian-10         cognosys                          apache-web-server-with-debian-10         cognosys:apache-web-server-with-debian-10:apache-web-server-with-debian-10:1.2019.1008                1.2019.1008
docker-ce-with-debian-10                 cognosys                          docker-ce-with-debian-10                 cognosys:docker-ce-with-debian-10:docker-ce-with-debian-10:1.2019.0710                                1.2019.0710
Debian                                   credativ                          8                                        credativ:Debian:8:8.0.201602010                                                                       8.0.201602010
Debian                                   credativ                          8                                        credativ:Debian:8:8.0.201603020                                                                       8.0.201603020
Debian                                   credativ                          8                                        credativ:Debian:8:8.0.201604050                                                                       8.0.201604050
...
```


## <a name="look-at-all-available-images"></a>Просмотреть все доступные образы
 
Еще один способ поиска образа в определенном расположении — это выполнить по-очереди команды [az vm image list-publishers](/cli/azure/vm/image), [az vm image list-offers](/cli/azure/vm/image) и [az vm image list-skus](/cli/azure/vm/image). С помощью этих команд определяются следующие значения:

1. Вывод списка издателей образов для расположения. В этом примере мы рассмотрим регион " *Западная часть США* ".
    
    ```azurecli
    az vm image list-publishers --location westus --output table
    ```

1. Получить список предложений нужного издателя. В этом примере мы добавляем *каноническое* значение в качестве издателя.
    
    ```azurecli
    az vm image list-offers --location westus --publisher Canonical --output table
    ```

1. Получить список номеров SKU для требуемого предложения. В этом примере мы добавим *UbuntuServer* в качестве предложения.
    ```azurecli
    az vm image list-skus --location westus --publisher Canonical --offer UbuntuServer --output table
    ```

1. Для данного издателя, предложения и SKU отображаются все версии образа. В этом примере мы добавили *18,04-LTS* в качестве номера SKU.

    ```azurecli
    az vm image list \
        --location westus \
        --publisher Canonical \  
        --offer UbuntuServer \    
        --sku 18.04-LTS \
        --all --output table
    ```

Передайте это значение столбца URN с помощью `--image` параметра при создании виртуальной машины с помощью команды [AZ VM Create](/cli/azure/vm) . Вы также можете заменить номер версии в URN на "latest", чтобы просто использовать последнюю версию образа. 

При развертывании виртуальной машины с помощью шаблона Resource Manager установите отдельные параметры образа в свойствах `imageReference`. Ознакомьтесь со статьей о [справочнике по шаблонам](/azure/templates/microsoft.compute/virtualmachines).


## <a name="check-the-purchase-plan-information"></a>Проверка сведений о плане покупки

Для некоторых образов виртуальных машин в Azure Marketplace применяются дополнительные условия лицензии и приобретения, которые нужно принять перед программным развертыванием.  

Чтобы развернуть виртуальную машину из такого образа, необходимо принять условия использования образа при первом его использовании, один раз для каждой подписки. Кроме того, необходимо указать параметры *плана покупки* для развертывания виртуальной машины из этого образа.

Чтобы просмотреть сведения о плане покупки изображения, выполните команду [AZ VM Image Показать](/cli/azure/image) , указав URN образа. Если в выходных данных значением свойства `plan` не является `null`, в образе появятся условия использования, которые необходимо принять перед программным развертыванием.

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

После запуска аналогичных команд для образа RabbitMQ, сертифицированного Bitnami, отображаются следующие свойства `plan`: `name`, `product` и `publisher`. (Некоторые образы также имеют свойство `promotion code`). 

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

Чтобы развернуть этот образ, необходимо принять условия и указать параметры плана покупки при развертывании виртуальной машины с помощью этого образа.

## <a name="accept-the-terms"></a>Принятие условий

Чтобы просмотреть и принять условия лицензии, используйте команду [az vm image accept-terms](/cli/azure/vm/image/terms). При принятии условий в вашей подписке будет включено программное развертывание. Необходимо принять условия соглашения для каждой подписки в образе. Пример:

```azurecli
az vm image terms show --urn bitnami:rabbitmq:rabbitmq:latest
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

Чтобы принять условия, введите:

```azurecli
az vm image terms accept --urn bitnami:rabbitmq:rabbitmq:latest
``` 

## <a name="deploy-a-new-vm-using-the-image-parameters"></a>Развертывание новой виртуальной машины с помощью параметров образа

Сведения о образе можно развернуть с помощью `az vm create` команды. 

Чтобы развернуть образ, не имеющий сведений о плане, как и последний образ Ubuntu Server 18,04 из канонической версии, передайте URN для `--image` :

```azurecli-interactive
az group create --name myURNVM --location westus
az vm create \
   --resource-group myURNVM \
   --name myVM \
   --admin-username azureuser \
   --generate-ssh-keys \
   --image Canonical:UbuntuServer:18.04-LTS:latest 
```


Для образа с параметрами плана покупки, например RabbitMQ, сертифицированного образом BitNami, вы передаете URN для `--image` , а также предоставляете параметры плана покупки:

```azurecli
az group create --name myPurchasePlanRG --location westus

az vm create \
   --resource-group myPurchasePlanRG \
   --name myVM \
   --admin-username azureuser \
   --generate-ssh-keys \
   --image bitnami:rabbitmq:rabbitmq:latest \
   --plan-name rabbitmq \
   --plan-product rabbitmq \
   --plan-publisher bitnami
```

Если вы получите сообщение о принятии условий образа, ознакомьтесь с разделом [«примите условия](#accept-the-terms)». Убедитесь, что выходные данные `az vm image accept-terms` возвращает значение `"accepted": true,` , показывающее, что вы приняли условия изображения.


## <a name="using-an-existing-vhd-with-purchase-plan-information"></a>Использование существующего виртуального жесткого диска с информацией о плане покупки

Если у вас есть виртуальный жесткий диск из виртуальной машины, созданной с помощью платного образа Azure Marketplace, может потребоваться предоставить сведения о плане покупки при создании виртуальной машины из этого виртуального жесткого диска. 

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


## <a name="next-steps"></a>Следующие шаги
Инструкции по быстрому созданию виртуальной машины на основе данных образа см. в статье [Создание виртуальных машин Linux и управление ими с помощью Azure CLI](tutorial-manage-vm.md).
