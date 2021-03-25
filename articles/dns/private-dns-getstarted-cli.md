---
title: Краткое руководство. Создание частной зоны Azure DNS с помощью Azure CLI
description: Из этого краткого руководства вы узнаете, как создать и проверить частную зону и запись DNS в Azure DNS. В нем пошагово описывается создание первой частной зоны и записи DNS, а также управление ими с помощью Azure CLI.
services: dns
author: rohinkoul
ms.service: dns
ms.topic: quickstart
ms.date: 10/20/2020
ms.author: rohink
ms.custom: devx-track-azurecli
ms.openlocfilehash: 820641af00caea4ffca450be8aa81b5357ba1261
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97652974"
---
# <a name="quickstart-create-an-azure-private-dns-zone-using-the-azure-cli"></a>Краткое руководство. Создание частной зоны Azure DNS с помощью Azure CLI

В этом кратком руководстве описывается создание первой частной зоны и записи DNS с помощью Azure CLI.

Зона DNS используется для размещения DNS-записей определенного домена. Чтобы разместить свой домен в Azure DNS, необходимо создать зону DNS для этого доменного имени. Каждая запись DNS для вашего домена создается внутри этой зоны DNS. Чтобы опубликовать частную зону DNS в виртуальной сети, укажите список виртуальных сетей, которые могут разрешать записи в зоне.  Они называются *связанными* виртуальными сетями. Когда автоматическая регистрация включена, Azure DNS также обновляет записи зоны при каждом создании виртуальной машины, изменении ее IP-адреса или удалении.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Обязательные условия

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

- Инструкции в этом кратком руководстве можно также выполнить с помощью [Azure PowerShell](private-dns-getstarted-powershell.md).

## <a name="create-the-resource-group"></a>Создание группы ресурсов

Сначала нужно создать группу ресурсов, которая будет включать зону DNS. 

```azurecli
az group create --name MyAzureResourceGroup --location "East US"
```

## <a name="create-a-private-dns-zone"></a>Создание частной зоны DNS

В следующем примере создается виртуальная сеть с именем **myAzureVNet**. Затем создается зона DNS с именем **private.contoso.com** в группе ресурсов **MyAzureResourceGroup**, которая связывает зону DNS с виртуальной сетью **MyAzureVnet** и включает автоматическую регистрацию.

```azurecli
az network vnet create \
  --name myAzureVNet \
  --resource-group MyAzureResourceGroup \
  --location eastus \
  --address-prefix 10.2.0.0/16 \
  --subnet-name backendSubnet \
  --subnet-prefixes 10.2.0.0/24

az network private-dns zone create -g MyAzureResourceGroup \
   -n private.contoso.com

az network private-dns link vnet create -g MyAzureResourceGroup -n MyDNSLink \
   -z private.contoso.com -v myAzureVNet -e true
```

Если вы хотите создать зону только для разрешения имен (без автоматической регистрации имени узла), можно использовать параметр `-e false`.

### <a name="list-dns-private-zones"></a>Список частных зон DNS

Чтобы перечислить зоны DNS, используйте `az network private-dns zone list`. Чтобы получить справку, см. `az network dns zone list --help`.

Если указать группу ресурсов, то будут перечислены только зоны в этой группе ресурсов.

```azurecli
az network private-dns zone list \
  -g MyAzureResourceGroup
```

Если не указать группу ресурсов, то будут перечислены все зоны в подписке.

```azurecli
az network private-dns zone list 
```

## <a name="create-the-test-virtual-machines"></a>Создание тестовых виртуальных машин

Теперь создайте две виртуальные машины, чтобы иметь возможность проверить частную зону DNS.

```azurecli
az vm create \
 -n myVM01 \
 --admin-username AzureAdmin \
 -g MyAzureResourceGroup \
 -l eastus \
 --subnet backendSubnet \
 --vnet-name myAzureVnet \
 --nsg NSG01 \
 --nsg-rule RDP \
 --image win2016datacenter

az vm create \
 -n myVM02 \
 --admin-username AzureAdmin \
 -g MyAzureResourceGroup \
 -l eastus \
 --subnet backendSubnet \
 --vnet-name myAzureVnet \
 --nsg NSG01 \
 --nsg-rule RDP \
 --image win2016datacenter
```

Ее выполнение может занять несколько минут.

## <a name="create-an-additional-dns-record"></a>Создание дополнительной записи DNS

Чтобы создать запись DNS, используйте команду `az network private-dns record-set [record type] add-record`. Чтобы получить справку, например по записям типа A, см. `az network private-dns record-set A add-record --help`.

 В следующем примере создается запись с относительным именем **db** в зоне DNS **private.contoso.com** в группе ресурсов **MyAzureResourceGroup**. Полное доменное имя набора записей — **db.private.contoso.com**. Тип записи — A, а ее IP-адрес — 10.2.0.4.

```azurecli
az network private-dns record-set a add-record \
  -g MyAzureResourceGroup \
  -z private.contoso.com \
  -n db \
  -a 10.2.0.4
```

### <a name="view-dns-records"></a>Просмотр записей DNS

Чтобы просмотреть список записей DNS в зоне, используйте следующую команду:

```azurecli
az network private-dns record-set list \
  -g MyAzureResourceGroup \
  -z private.contoso.com
```

## <a name="test-the-private-zone"></a>Проверка частной зоны

Теперь вы можете проверить разрешение имени для частной зоны **private.contoso.com**.

### <a name="configure-vms-to-allow-inbound-icmp"></a>Настройка виртуальных машин для получения входящего ICMP-трафика

Можно воспользоваться командой проверки связи (ping) для проверки разрешения имени. Для этого настройте брандмауэр для обеих виртуальных машин, чтобы получать входящие пакеты ICMP.

1. Подключитесь к myVM01 и откройте окно Windows PowerShell с правами администратора.
2. Выполните следующую команду:

   ```powershell
   New-NetFirewallRule –DisplayName "Allow ICMPv4-In" –Protocol ICMPv4
   ```

Повторите действия для myVM02.

### <a name="ping-the-vms-by-name"></a>Проверка связи с виртуальными машинами по имени

1. В командной строке Windows PowerShell на виртуальной машине myVM02 проверьте связь с myVM01, используя автоматически зарегистрированное имя узла.

   ```powershell
   ping myVM01.private.contoso.com
   ```

   Результат должен выглядеть примерно так:

   ```output
   PS C:\> ping myvm01.private.contoso.com

   Pinging myvm01.private.contoso.com [10.2.0.4] with 32 bytes of data:
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128
   Reply from 10.2.0.4: bytes=32 time=1ms TTL=128
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128

   Ping statistics for 10.2.0.4:
       Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
   Approximate round trip times in milli-seconds:
       Minimum = 0ms, Maximum = 1ms, Average = 0ms
   PS C:\>
   ```

2. Теперь проверьте связь с именем **db**, созданным ранее.

   ```powershell
   ping db.private.contoso.com
   ```

   Результат должен выглядеть примерно так:

   ```output
   PS C:\> ping db.private.contoso.com

   Pinging db.private.contoso.com [10.2.0.4] with 32 bytes of data:
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128

   Ping statistics for 10.2.0.4:
       Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
   Approximate round trip times in milli-seconds:
       Minimum = 0ms, Maximum = 0ms, Average = 0ms
   PS C:\>
   ```

## <a name="clean-up-resources"></a>Очистка ресурсов

Удалите группу ресурсов **MyAzureResourceGroup**, если ресурсы, созданные в этом руководстве, больше не нужны.

```azurecli
az group delete --name MyAzureResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Сценарии применения частных зон DNS Azure](private-dns-scenarios.md)

