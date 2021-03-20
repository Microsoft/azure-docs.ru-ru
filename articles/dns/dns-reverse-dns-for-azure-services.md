---
title: Обратная DNS для служб Azure — Azure DNS
description: По этой схеме обучения приступайте к настройке обратного просмотра DNS для служб, размещенных в Azure.
services: dns
documentationcenter: na
author: rohinkoul
ms.service: dns
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/29/2017
ms.author: rohink
ms.openlocfilehash: c72e17e601ebf87b0f344a4723159ae22abc81d1
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98919890"
---
# <a name="configure-reverse-dns-for-services-hosted-in-azure"></a>Настройка обратного просмотра DNS для размещенных в Azure служб

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Из этой статьи вы узнаете, как настроить обратный просмотр DNS для размещенных в Azure служб.

Службы в Azure используют IP-адреса, присвоенные им службой Azure и являющиеся собственностью Майкрософт. Эти записи в обратной зоне DNS (записи типа PTR) должны создаваться в соответствующих зонах обратного просмотра DNS, принадлежащих корпорации Майкрософт. В этой статье описано, как это сделать.

Этот сценарий не следует путать с возможностью [размещать зоны обратного просмотра DNS для присвоенных диапазонов IP-адресов в службе Azure DNS](dns-reverse-dns-hosting.md). В этом случае диапазоны IP-адресов, представленные зоной обратного просмотра, должны быть присвоены организации (как правило, поставщиком услуг Интернета).

Перед прочтением этой статьи ознакомьтесь со статьей [Основные сведения об обратной зоне DNS и ее поддержке в Azure](dns-reverse-dns-overview.md).

В Azure DNS вычислительные ресурсы (например, виртуальные машины, масштабируемые наборы виртуальных машин или кластеры Service Fabric) предоставляются с помощью ресурса PublicIpAddress. Обратный просмотр DNS настраивается с помощью свойства ReverseFqdn ресурса PublicIpAddress.


Обратная DNS в настоящее время не поддерживается для службы приложений Azure и шлюза приложений.

## <a name="validation-of-reverse-dns-records"></a>Проверка записей обратной зоны DNS

Чтобы сторонние лица не могли создавать записи обратной зоны DNS для сопоставления своих служб Azure с вашими доменами DNS, предусмотрен специальный механизм. Azure позволяет создать запись обратной зоны DNS, только если доменное имя, указанное в такой записи, совпадает с DNS-именем либо IP-адресом (или разрешается в них) ресурса PublicIpAddress или облачной службы в той же подписке Azure.

Эта проверка выполняется только при добавлении или изменении записи обратной зоны DNS. Периодическая повторная проверка не выполняется.

Предположим, ресурс PublicIpAddress имеет DNS-имя contosoapp1.northus.cloudapp.azure.com и IP-адрес 23.96.52.53. Свойство ReverseFqdn для ресурса PublicIpAddress можно указать следующим образом:
* DNS-имя для ресурса PublicIpAddress — contosoapp1.northus.cloudapp.azure.com;
* DNS-имя для другого ресурса PublicIpAddress в той же подписке — contosoapp2.westus.cloudapp.azure.com;
* личное DNS-имя — app1.contoso.com при условии, что это имя *сначала* настроено как CNAME для contosoapp1.northus.cloudapp.azure.com или другого ресурса PublicIpAddress в той же подписке;
* личное DNS-имя — app1.contoso.com при условии, что это имя *сначала* настроено как запись A для IP-адреса 23.96.52.53 или IP-адреса другого ресурса PublicIpAddress в той же подписке.

Те же ограничения применяются к обратной зоне DNS для облачных служб.


## <a name="reverse-dns-for-publicipaddress-resources"></a>Обратная зона DNS для ресурсов PublicIpAddress

Этот раздел содержит подробные инструкции по настройке обратной зоны DNS для ресурсов PublicIpAddress в модели развертывания Resource Manager с помощью Azure PowerShell, классического Azure CLI или Azure CLI. Настроить обратную зону DNS для ресурсов PublicIpAddres на портале Azure пока нельзя.

Сейчас Azure поддерживает обратную зону DNS только для IPv4-ресурсов PublicIpAddress. Для IPv6 эта возможность не поддерживается.

### <a name="add-reverse-dns-to-an-existing-publicipaddresses"></a>Добавление обратной зоны DNS в существующий ресурс PublicIpAddress

#### <a name="powershell"></a>PowerShell

Чтобы изменить обратную DNS на существующую PublicIpAddress, выполните следующие действия.

```powershell
$pip = Get-AzPublicIpAddress -Name "PublicIp" -ResourceGroupName "MyResourceGroup"
$pip.DnsSettings.ReverseFqdn = "contosoapp1.westus.cloudapp.azure.com."
Set-AzPublicIpAddress -PublicIpAddress $pip
```

Чтобы добавить обратную зону DNS в существующий ресурс PublicIpAddress, у которого еще нет DNS-имени, необходимо указать DNS-имя:

```powershell
$pip = Get-AzPublicIpAddress -Name "PublicIp" -ResourceGroupName "MyResourceGroup"
$pip.DnsSettings = New-Object -TypeName "Microsoft.Azure.Commands.Network.Models.PSPublicIpAddressDnsSettings"
$pip.DnsSettings.DomainNameLabel = "contosoapp1"
$pip.DnsSettings.ReverseFqdn = "contosoapp1.westus.cloudapp.azure.com."
Set-AzPublicIpAddress -PublicIpAddress $pip
```

#### <a name="azure-classic-cli"></a>Классический Azure CLI

Чтобы добавить обратную зону DNS в существующий ресурс PublicIpAddress, выполните следующую команду:

```azurecli
azure network public-ip set -n PublicIp -g MyResourceGroup -f contosoapp1.westus.cloudapp.azure.com.
```

Чтобы добавить обратную зону DNS в существующий ресурс PublicIpAddress, у которого еще нет DNS-имени, необходимо указать DNS-имя:

```azurecli
azure network public-ip set -n PublicIp -g MyResourceGroup -d contosoapp1 -f contosoapp1.westus.cloudapp.azure.com.
```

#### <a name="azure-cli"></a>Azure CLI

Чтобы добавить обратную зону DNS в существующий ресурс PublicIpAddress, выполните следующую команду:

```azurecli
az network public-ip update --resource-group MyResourceGroup --name PublicIp --reverse-fqdn contosoapp1.westus.cloudapp.azure.com.
```

Чтобы добавить обратную зону DNS в существующий ресурс PublicIpAddress, у которого еще нет DNS-имени, необходимо указать DNS-имя:

```azurecli
az network public-ip update --resource-group MyResourceGroup --name PublicIp --reverse-fqdn contosoapp1.westus.cloudapp.azure.com --dns-name contosoapp1
```

### <a name="create-a-public-ip-address-with-reverse-dns"></a>Создание общедоступного IP-адреса с обратной зоной DNS

Чтобы создать ресурс PublicIpAddress с уже указанным свойством обратной зоны DNS, выполните указанную ниже команду.

#### <a name="powershell"></a>PowerShell

```powershell
New-AzPublicIpAddress -Name "PublicIp" -ResourceGroupName "MyResourceGroup" -Location "WestUS" -AllocationMethod Dynamic -DomainNameLabel "contosoapp2" -ReverseFqdn "contosoapp2.westus.cloudapp.azure.com."
```

#### <a name="azure-classic-cli"></a>Классический Azure CLI

```azurecli
azure network public-ip create -n PublicIp -g MyResourceGroup -l westus -d contosoapp3 -f contosoapp3.westus.cloudapp.azure.com.
```

#### <a name="azure-cli"></a>Azure CLI

```azurecli
az network public-ip create --name PublicIp --resource-group MyResourceGroup --location westcentralus --dns-name contosoapp1 --reverse-fqdn contosoapp1.westcentralus.cloudapp.azure.com
```

### <a name="view-reverse-dns-for-an-existing-publicipaddress"></a>Просмотр обратной зоны DNS в существующем ресурсе PublicIpAddress

Чтобы просмотреть настроенное значение в существующем ресурсе PublicIpAddress, выполните следующую команду:

#### <a name="powershell"></a>PowerShell

```powershell
Get-AzPublicIpAddress -Name "PublicIp" -ResourceGroupName "MyResourceGroup"
```

#### <a name="azure-classic-cli"></a>Классический Azure CLI

```azurecli
azure network public-ip show -n PublicIp -g MyResourceGroup
```

#### <a name="azure-cli"></a>Azure CLI

```azurecli
az network public-ip show --name PublicIp --resource-group MyResourceGroup
```

### <a name="remove-reverse-dns-from-existing-public-ip-addresses"></a>Удаление обратной зоны DNS из существующих общедоступных IP-адресов

Чтобы удалить свойство обратной зоны DNS из существующего ресурса PublicIpAddress, выполните следующую команду:

#### <a name="powershell"></a>PowerShell

```powershell
$pip = Get-AzPublicIpAddress -Name "PublicIp" -ResourceGroupName "MyResourceGroup"
$pip.DnsSettings.ReverseFqdn = ""
Set-AzPublicIpAddress -PublicIpAddress $pip
```

#### <a name="azure-classic-cli"></a>Классический Azure CLI

```azurecli
azure network public-ip set -n PublicIp -g MyResourceGroup –f ""
```

#### <a name="azure-cli"></a>Azure CLI

```azurecli
az network public-ip update --resource-group MyResourceGroup --name PublicIp --reverse-fqdn ""
```


## <a name="configure-reverse-dns-for-cloud-services"></a>Настройка обратной зоны DNS для облачных служб

Этот раздел содержит подробные инструкции по настройке обратной зоны DNS для облачных служб в классической модели развертывания с помощью Azure PowerShell. Настройка обратной зоны DNS для облачных служб с помощью портала Azure, классического Azure CLI и Azure CLI не поддерживается.

### <a name="add-reverse-dns-to-existing-cloud-services"></a>Добавление обратной зоны DNS в существующие облачные службы

Чтобы добавить обратную зону DNS в существующую облачную службу, выполните следующую команду:

```powershell
Set-AzureService –ServiceName "contosoapp1" –Description "App1 with Reverse DNS" –ReverseDnsFqdn "contosoapp1.cloudapp.net."
```

### <a name="create-a-cloud-service-with-reverse-dns"></a>Создание облачной службы с обратной зоной DNS

Чтобы создать облачную службу с уже указанным свойством обратной зоны DNS, выполните следующую команду:

```powershell
New-AzureService –ServiceName "contosoapp1" –Location "West US" –Description "App1 with Reverse DNS" –ReverseDnsFqdn "contosoapp1.cloudapp.net."
```

### <a name="view-reverse-dns-for-existing-cloud-services"></a>Просмотр обратной зоны DNS в существующих облачных службах

Чтобы просмотреть свойство обратной зоны DNS для существующей облачной службы, выполните следующую команду:

```powershell
Get-AzureService "contosoapp1"
```

### <a name="remove-reverse-dns-from-existing-cloud-services"></a>Удаление обратной зоны DNS из существующих облачных служб

Чтобы удалить свойство обратной зоны DNS из существующей облачной службы, выполните следующую команду:

```powershell
Set-AzureService –ServiceName "contosoapp1" –Description "App1 with Reverse DNS" –ReverseDnsFqdn ""
```

## <a name="faq"></a>Вопросы и ответы

### <a name="how-much-do-reverse-dns-records-cost"></a>Сколько стоит обратная зона DNS?

Она бесплатна!  Никакие дополнительные затраты на записи или запросы обратной зоны DNS не требуются.

### <a name="will-my-reverse-dns-records-resolve-from-the-internet"></a>Будут ли мои записи обратной зоны DNS разрешаться из Интернета?

Да. После того, как вы укажете свойство обратной зоны DNS для своей службы Azure, облако Azure возьмет на себя управление зонами DNS и всеми операциями делегирования DNS, необходимыми для того, чтобы запись обратной зоны DNS разрешалась для всех интернет-пользователей.

### <a name="are-default-reverse-dns-records-created-for-my-azure-services"></a>Создаются ли для моих служб Azure какие-то стандартные записи обратной зоны DNS?

Нет. Обратная зона DNS — это подключаемая возможность. Если вы не захотите настраивать записи обратной зоны DNS, они не будут создаваться по умолчанию.

### <a name="what-is-the-format-for-the-fully-qualified-domain-name-fqdn"></a>Каков формат полного доменного имени (FQDN)?

Полные доменные имена указываются в прямом порядке и должны завершаться точкой (например, app1.contoso.com.).

### <a name="what-happens-if-the-validation-check-for-the-reverse-dns-ive-specified-fails"></a>Что произойдет, если при проверке указанной мной записи обратной зоны DNS произойдет ошибка?

Если при проверке обратной зоны DNS происходит ошибка, настройка записи обратной зоны DNS завершается ошибкой. В этом случае исправьте значение обратной зоны DNS и повторите попытку.

### <a name="can-i-configure-reverse-dns-for-azure-app-service"></a>Можно ли настроить обратную зону DNS для службы приложений Azure?

Нет. Служба приложений Azure не поддерживает обратную зону DNS.

### <a name="can-i-configure-multiple-reverse-dns-records-for-my-azure-service"></a>Можно ли настроить несколько записей обратной зоны DNS для моей службы Azure?

Нет. В Azure поддерживается только одна запись обратной зоны DNS для каждой облачной службы Azure или каждого ресурса PublicIpAddress.

### <a name="can-i-configure-reverse-dns-for-ipv6-publicipaddress-resources"></a>Можно ли настроить обратную зону DNS для IPv6-ресурсов PublicIpAddress?

Нет. Сейчас Azure поддерживает обратную зону DNS только для облачных служб и IPv4-ресурсов PublicIpAddress.

### <a name="can-i-send-emails-to-external-domains-from-my-azure-compute-services"></a>Можно ли отправлять электронные сообщения во внешние домены из служб вычислений Azure?

Техническая возможность непосредственной отправки по электронной почте из развертывания Azure зависит от типа подписки. Независимо от типа подписки корпорация Майкрософт рекомендует использовать доверенные службы ретрансляции для отправки исходящей почты. Дополнительные сведения см. в [обновлении за ноябрь 2017 г. для усиленной безопасности Azure для отправки электронных сообщений](../virtual-network/troubleshoot-outbound-smtp-connectivity.md).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. [в статье Википедии об обратном просмотре DNS](https://en.wikipedia.org/wiki/Reverse_DNS_lookup).
<br>
Узнайте, как [разместить зону обратного просмотра для диапазона IP-адресов, присвоенного поставщиком услуг Интернета, в службе Azure DNS](dns-reverse-dns-for-azure-services.md).
