---
title: Делегирование поддомена — Azure PowerShell — Azure DNS
description: По этой схеме обучения приступайте к делегированию Azure DNS поддомена с помощью Azure PowerShell.
services: dns
author: rohinkoul
ms.service: dns
ms.topic: how-to
ms.date: 2/7/2019
ms.author: rohink
ms.openlocfilehash: 9b37d313aa5d8c2255b4e3be69831dfcb50238ea
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "84712553"
---
# <a name="delegate-an-azure-dns-subdomain-using-azure-powershell"></a>Делегирование поддомена Azure DNS с помощью Azure PowerShell

Вы можете делегировать поддомен DNS с помощью Azure PowerShell. Например, если вы являетесь владельцем домена contoso.com, вы можете делегировать поддомен *engineering* в отдельную зону, администрирование которой может осуществляться отдельно от зоны contoso.com.

При необходимости вы можете делегировать поддомен с помощью [портала Azure](delegate-subdomain.md).

> [!NOTE]
> В этой статье веб-сайт contoso.com используется в качестве примера. Подставьте собственное доменное имя вместо contoso.com.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="prerequisites"></a>Предварительные требования

Чтобы делегировать поддомен Azure DNS, сначала вам следует делегировать общедоступный домен службе Azure DNS. Дополнительные сведения о настройке делегирования в серверах имен см. в статье [Руководство. Размещение домена в Azure DNS](./dns-delegate-domain-azure-dns.md). После делегирования домена в зону Azure DNS вы можете делегировать поддомен.

## <a name="create-a-zone-for-your-subdomain"></a>Создание зоны для поддомена

Сначала создайте зону для поддомена **engineering**.

`New-AzDnsZone -ResourceGroupName <resource group name> -Name engineering.contoso.com`

## <a name="note-the-name-servers"></a>Просмотр значений серверов имен

Далее обратите внимание на четыре сервера имен для поддомена engineering.

`Get-AzDnsRecordSet -ZoneName engineering.contoso.com -ResourceGroupName <resource group name> -RecordType NS`

## <a name="create-a-test-record"></a>Создание тестовой записи

Создайте запись **А** в инженерной зоне, чтобы использовать ее для тестирования.

   `New-AzDnsRecordSet -ZoneName engineering.contoso.com -ResourceGroupName <resource group name> -Name www -RecordType A -ttl 3600 -DnsRecords (New-AzDnsRecordConfig -IPv4Address 10.10.10.10)`.

## <a name="create-an-ns-record"></a>Создание записи типа NS

Далее создайте запись сервера имен (NS) для зоны **engineering** в зоне contoso.com.

```azurepowershell
$Records = @()
$Records += New-AzDnsRecordConfig -Nsdname <name server 1 noted previously>
$Records += New-AzDnsRecordConfig -Nsdname <name server 2 noted previously>
$Records += New-AzDnsRecordConfig -Nsdname <name server 3 noted previously>
$Records += New-AzDnsRecordConfig -Nsdname <name server 4 noted previously>
$RecordSet = New-AzDnsRecordSet -Name engineering -RecordType NS -ResourceGroupName <resource group name> -TTL 3600 -ZoneName contoso.com -DnsRecords $Records
```

## <a name="test-the-delegation"></a>Проверка делегирования

Проверьте делегирование с помощью nslookup.

1. Откройте окно PowerShell.
2. В командной строке введите `nslookup www.engineering.contoso.com.`
3. Вы должны получить ответ, исходящий не из полномочного источника и содержащий адрес **10.10.10.10**.

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как [настроить обратный просмотр DNS для размещенных в Azure служб](dns-reverse-dns-for-azure-services.md).