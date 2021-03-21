---
title: Размещение зон обратного просмотра DNS в Azure DNS
description: Узнайте, как использовать Azure DNS, чтобы разместить зоны обратного просмотра DNS для диапазонов IP-адресов
author: rohinkoul
ms.service: dns
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 05/29/2017
ms.author: rohink
ms.openlocfilehash: 0c85049d6c8921432a753bf08989cab473b7c734
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99525133"
---
# <a name="host-reverse-dns-lookup-zones-in-azure-dns"></a>Размещение зон обратного просмотра DNS в Azure DNS

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

В этой статье описано размещение зон обратного просмотра DNS для назначенных диапазонов IP-адресов в Azure DNS. Диапазоны IP-адресов, представленные зонами обратного просмотра, должны быть назначены организации (как правило, поставщиком услуг Интернета).

Сведения о настройке обратного просмотра DNS для принадлежащего Azure IP-адреса, назначенного службе Azure, см. в статье [Настройка обратного просмотра DNS для размещенных в Azure служб](dns-reverse-dns-for-azure-services.md).

Перед прочтением этой статьи ознакомьтесь со статьей [Основные сведения об обратной зоне DNS и ее поддержке в Azure](dns-reverse-dns-overview.md).

Эта статья поможет вам создать свою первую зону обратного просмотра и обратную запись DNS с помощью Azure CLI, портала Azure, Azure PowerShell или классического Azure CLI.

## <a name="create-a-reverse-lookup-dns-zone"></a>Создание зоны обратного просмотра DNS

1. Войдите на [портал Azure](https://portal.azure.com).
1. В **главном** меню выберите **Создать** > **Сети**, а затем выберите пункт **Зона DNS**.

   ![Выбор "зоны DNS"](./media/dns-reverse-dns-hosting/figure1.png)

1. На панели **Создание зоны DNS** укажите имя зоны DNS. Для префиксов IPv4 и IPv6 имя зоны формируется по-разному. Чтобы указать имя зоны, выполните инструкции для [IPv4](#ipv4) или [IPv6](#ipv6). Когда закончите, нажмите кнопку **Создать**, чтобы создать зону.

### <a name="ipv4"></a>IPv4

Имя зоны обратного просмотра IPv4 формируется на основе диапазона IP-адресов, которые она представляет. Имя должно быть указано в формате `<IPv4 network prefix in reverse order>.in-addr.arpa`. Примеры см. в [обзоре обратной зоны DNS и ее поддержке в Azure](dns-reverse-dns-overview.md#ipv4).

> [!NOTE]
> Создавая бесклассовую зону обратного просмотра DNS в Azure DNS, в имени зоны необходимо использовать дефис (`-`) вместо косой черты (`/`).
>
> Например, для диапазона IP-адресов 192.0.2.128/26 в качестве имени зоны нужно использовать `128-26.2.0.192.in-addr.arpa`, а не `128/26.2.0.192.in-addr.arpa`.
>
> Хотя стандарты DNS поддерживают оба метода, Azure DNS не поддерживает имена зон DNS, которые содержат символ косой черты ( `/` ).

В примере ниже показано, как создать зону обратного просмотра DNS класса C с именем `2.0.192.in-addr.arpa` в Azure DNS при помощи портала Azure:

 ![Снимок экрана, показывающий, как создать обратную зону DNS класса C с именем 2.0.192.in-addr. arpa в Azure DNS через портал Azure.](./media/dns-reverse-dns-hosting/figure2.png)

Значение поля **Расположение группы ресурсов** определяет расположение группы ресурсов. Оно не влияет на зону DNS. Расположение зоны DNS всегда является глобальным и не отображается.

Ниже приведены примеры выполнения этой задачи с помощью Azure PowerShell и Azure CLI.

#### <a name="powershell"></a>PowerShell

```powershell
New-AzDnsZone -Name 2.0.192.in-addr.arpa -ResourceGroupName MyResourceGroup
```

#### <a name="azure-classic-cli"></a>Классический Azure CLI

```azurecli
azure network dns zone create MyResourceGroup 2.0.192.in-addr.arpa
```

#### <a name="azure-cli"></a>Azure CLI

```azurecli
az network dns zone create -g MyResourceGroup -n 2.0.192.in-addr.arpa
```

### <a name="ipv6"></a>IPv6;

Имя зоны обратного просмотра IPv6 должно быть указано в формате `<IPv6 network prefix in reverse order>.ip6.arpa`.  Примеры см. в [обзоре обратной зоны DNS и ее поддержке в Azure](dns-reverse-dns-overview.md#ipv6).


В примере ниже показано, как создать зону обратного просмотра DNS IPv6 с именем `0.0.0.0.d.c.b.a.8.b.d.0.1.0.0.2.ip6.arpa` в Azure DNS при помощи портала Azure:

 ![Панель "Создание зоны DNS" с заполненными полями](./media/dns-reverse-dns-hosting/figure3.png)

Значение поля **Расположение группы ресурсов** определяет расположение группы ресурсов. Оно не влияет на зону DNS. Расположение зоны DNS всегда является глобальным и не отображается.

Ниже приведены примеры выполнения этой задачи с помощью Azure PowerShell и Azure CLI.

#### <a name="powershell"></a>PowerShell

```powershell
New-AzDnsZone -Name 0.0.0.0.d.c.b.a.8.b.d.0.1.0.0.2.ip6.arpa -ResourceGroupName MyResourceGroup
```

#### <a name="azure-classic-cli"></a>Классический Azure CLI

```azurecli
azure network dns zone create MyResourceGroup 0.0.0.0.d.c.b.a.8.b.d.0.1.0.0.2.ip6.arpa
```

#### <a name="azure-cli"></a>Azure CLI

```azurecli
az network dns zone create -g MyResourceGroup -n 0.0.0.0.d.c.b.a.8.b.d.0.1.0.0.2.ip6.arpa
```

## <a name="delegate-a-reverse-dns-lookup-zone"></a>Делегирование зоны обратного просмотра DNS

Создав зону обратного просмотра DNS, необходимо убедиться, что она делегирована из родительской зоны. Делегирование DNS позволяет процессу разрешения имен DNS находить серверы имен, на которых размещена зона обратного просмотра DNS. Затем эти серверы имен могут отвечать на обратные запросы DNS для IP-адресов в указанном диапазоне.

Для зон прямого просмотра делегирование зоны DNS описано в разделе [Делегирование домена в Azure DNS](dns-delegate-domain-azure-dns.md). Делегирование для зон обратного просмотра выполняется аналогичным образом. Единственное различие — серверы имен настраиваются с помощью поставщика услуг Интернета, предоставившего вам диапазон IP-адресов, а не с помощью регистратора доменных имен.

## <a name="create-a-dns-ptr-record"></a>Создание записи DNS типа PTR

### <a name="ipv4"></a>IPv4

Следующий пример демонстрирует создание записи типа PTR для зоны обратного просмотра DNS в Azure DNS. Сведения о других типах записей и об изменении существующих записей см. в статье [Управление записями и наборами записей DNS с помощью портала Azure](dns-operations-recordsets-portal.md).

1. В верхней части панели **Зона DNS** щелкните **+ Набор записей**, чтобы открыть панель **Добавление набора записей**.

   ![Снимок экрана: область зоны DNS со стрелкой, указывающей на кнопку + набор записей.](./media/dns-reverse-dns-hosting/figure4.png)

1. Имя набора записей для записи типа PTR должно представлять собой оставшуюся часть адреса IPv4 в обратном порядке. 

   В этом примере первые три октета уже указаны как часть имени зоны (.2.0.192). Таким образом, в поле **Имя** содержится только последний октет. Например, для ресурса с IP-адресом 192.0.2.15 можно назвать набор записей **15**.  
1. В поле **Тип** выберите значение **PTR**.  
1. В поле **Доменное имя** введите полное доменное имя ресурса, использующего IP-адрес.
1. Чтобы создать запись DNS, нажмите кнопку **ОК** в нижней части панели.

   ![Панель "Добавление набора записей" с заполненными полями](./media/dns-reverse-dns-hosting/figure5.png)

Ниже приведены примеры выполнения этой задачи с помощью PowerShell или Azure CLI.

#### <a name="powershell"></a>PowerShell

```powershell
New-AzDnsRecordSet -Name 15 -RecordType PTR -ZoneName 2.0.192.in-addr.arpa -ResourceGroupName MyResourceGroup -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -Ptrdname "dc1.contoso.com")
```
#### <a name="azure-classic-cli"></a>Классический Azure CLI

```azurecli
azure network dns record-set add-record MyResourceGroup 2.0.192.in-addr.arpa 15 PTR --ptrdname dc1.contoso.com  
```

#### <a name="azure-cli"></a>Azure CLI

```azurecli
az network dns record-set ptr add-record -g MyResourceGroup -z 2.0.192.in-addr.arpa -n 15 --ptrdname dc1.contoso.com
```

### <a name="ipv6"></a>IPv6;

Следующий пример демонстрирует процесс создания записи типа PTR. Сведения о других типах записей и об изменении существующих записей см. в статье [Управление записями и наборами записей DNS с помощью портала Azure](dns-operations-recordsets-portal.md).

1. В верхней части панели **Зона DNS** щелкните **+ Набор записей**, чтобы открыть панель **Добавление набора записей**.

   ![Кнопка для создания набора записей](./media/dns-reverse-dns-hosting/figure6.png)

2. Имя набора записей для записи типа PTR должно представлять собой оставшуюся часть адреса IPv6 в обратном порядке. Сжатие за счет нулей не должно использоваться. 

   В этом примере первые 64 бита IPv6 уже указаны как часть имени зоны (0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2.ip6.arpa). Таким образом, в поле **Имя** содержатся только последние 64 бита. Последние 64 бита IP-адреса вводятся в обратном порядке, в качестве разделителя между каждым шестнадцатеричным числом используется точка. Например, для ресурса с IP-адресом 2001:0db8:abdc:0000:f524:10bc:1af9:405e набор записей можно назвать **e.5.0.4.9.f.a.1.c.b.0.1.4.2.5.f**.  
3. В поле **Тип** выберите значение **PTR**.  
4. В поле **Доменное имя** введите полное доменное имя ресурса, использующего IP-адрес.
5. Чтобы создать запись DNS, нажмите кнопку **ОК** в нижней части панели.

![Снимок экрана, на котором показана панель "Добавление набора записей" со стрелкой, указывающей на значение в поле "тип".](./media/dns-reverse-dns-hosting/figure7.png)

Ниже приведены примеры выполнения этой задачи с помощью PowerShell или Azure CLI.

#### <a name="powershell"></a>PowerShell

```powershell
New-AzDnsRecordSet -Name "e.5.0.4.9.f.a.1.c.b.0.1.4.2.5.f" -RecordType PTR -ZoneName 0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2.ip6.arpa -ResourceGroupName MyResourceGroup -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -Ptrdname "dc2.contoso.com")
```

#### <a name="azure-classic-cli"></a>Классический Azure CLI

```azurecli
azure network dns record-set add-record MyResourceGroup 0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2.ip6.arpa e.5.0.4.9.f.a.1.c.b.0.1.4.2.5.f PTR --ptrdname dc2.contoso.com 
```
 
#### <a name="azure-cli"></a>Azure CLI

```azurecli
az network dns record-set ptr add-record -g MyResourceGroup -z 0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2.ip6.arpa -n e.5.0.4.9.f.a.1.c.b.0.1.4.2.5.f --ptrdname dc2.contoso.com
```

## <a name="view-records"></a>Просмотр записей

Чтобы просмотреть созданные записи, перейдите к зоне DNS на портале Azure. В нижней части панели **Зона DNS** содержатся записи для зоны DNS, в частности записи типа NS и SOA по умолчанию, а также созданные вами записи. Записи типа NS и SOA создаются в каждой зоне. 

### <a name="ipv4"></a>IPv4

На панели **Зона DNS** отображаются записи IPv4 типа PTR:

![Панель "Зона DNS" вместе с записями IPv4](./media/dns-reverse-dns-hosting/figure8.png)

В примерах ниже показано, как просмотреть записи типа PTR с помощью PowerShell или Azure CLI.

#### <a name="powershell"></a>PowerShell

```powershell
Get-AzDnsRecordSet -ZoneName 2.0.192.in-addr.arpa -ResourceGroupName MyResourceGroup
```

#### <a name="azure-classic-cli"></a>Классический Azure CLI

```azurecli
azure network dns record-set list MyResourceGroup 2.0.192.in-addr.arpa
```

#### <a name="azure-cli"></a>Azure CLI

```azurecli
az network dns record-set list -g MyResourceGroup -z 2.0.192.in-addr.arpa
```

### <a name="ipv6"></a>IPv6;

На панели **Зона DNS** отображаются записи IPv6 типа PTR:

![Панель "Зона DNS" вместе с записями IPv6](./media/dns-reverse-dns-hosting/figure9.png)

В примерах ниже показано, как просмотреть записи с помощью PowerShell или Azure CLI.

#### <a name="powershell"></a>PowerShell

```powershell
Get-AzDnsRecordSet -ZoneName 0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2.ip6.arpa -ResourceGroupName MyResourceGroup
```

#### <a name="azure-classic-cli"></a>Классический Azure CLI

```azurecli
azure network dns record-set list MyResourceGroup 0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2.ip6.arpa
```

#### <a name="azure-cli"></a>Azure CLI

```azurecli
az network dns record-set list -g MyResourceGroup -z 0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2.ip6.arpa
```

## <a name="faq"></a>Вопросы и ответы

### <a name="can-i-host-reverse-dns-lookup-zones-for-my-isp-assigned-ip-blocks-on-azure-dns"></a>Можно ли разместить зоны обратного просмотра DNS для блоков IP-адресов, назначенных поставщиком услуг Интернета, в DNS Azure?

Да. Размещение зон обратного просмотра DNS для пользовательских диапазонов IP-адресов в Azure DNS полностью поддерживается.

Создайте зону обратного просмотра в Azure DNS, как описано в этой статье, а затем [делегируйте ее](dns-domain-delegation.md) с помощью поставщика услуг Интернета. Вы можете управлять записями типа PTR для каждого обратного просмотра точно так же, как и другими записями другого типа.

### <a name="how-much-does-hosting-my-reverse-dns-lookup-zone-cost"></a>Сколько стоит размещение зоны обратного просмотра DNS?

Размещение зоны обратного просмотра DNS в Azure DNS для блока IP-адресов, назначенного поставщиком услуг Интернета, оплачивается по [стандартным тарифам Azure DNS](https://azure.microsoft.com/pricing/details/dns/).

### <a name="can-i-host-reverse-dns-lookup-zones-for-both-ipv4-and-ipv6-addresses-in-azure-dns"></a>Можно ли разместить в Azure DNS зоны обратного просмотра DNS для адресов IPv4 и IPv6?

Да. В этой статье объясняется, как создать зоны обратного просмотра DNS для адресов IPv4 и IPv6 в Azure DNS.

### <a name="can-i-import-an-existing-reverse-dns-lookup-zone"></a>Можно ли импортировать существующую зону обратного просмотра DNS?

Да. Вы можете импортировать существующие зоны DNS в Azure DNS при помощи Azure CLI. Этот метод работает как с зонами прямого просмотра, так и с зонами обратного просмотра.

Дополнительные сведения см. в статье [Импорт и экспорт файла зоны DNS с помощью Azure CLI 1.0](dns-import-export.md).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. [в статье Википедии об обратном просмотре DNS](https://en.wikipedia.org/wiki/Reverse_DNS_lookup).
<br>
Узнайте, как [управлять записями обратной зоны DNS для служб Azure](dns-reverse-dns-for-azure-services.md).
