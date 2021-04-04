---
title: Управление зонами DNS в службе DNS Azure с помощью PowerShell | Документация Майкрософт
description: Зонами DNS можно управлять с помощью Azure Powershell. В этой статье описывается, как обновлять, удалять и создавать зоны DNS в службе DNS Azure.
services: dns
documentationcenter: na
author: rohinkoul
ms.service: dns
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/19/2018
ms.author: rohink
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 6abcca9d9888dc8968d7233e7aee6cd76aa215f7
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "94965754"
---
# <a name="how-to-manage-dns-zones-using-powershell"></a>Как управлять зонами DNS с помощью PowerShell

> [!div class="op_single_selector"]
> * [Портал](dns-operations-dnszones-portal.md)
> * [PowerShell](dns-operations-dnszones.md)
> * [Классический Azure CLI](./dns-operations-dnszones-cli.md)
> * [Azure CLI](dns-operations-dnszones-cli.md)

В этой статье описывается, как управлять зонами DNS с помощью Azure PowerShell. Зонами DNS также можно управлять с помощью кроссплатформенного [интерфейса командной строки Azure](dns-operations-dnszones-cli.md) или портала Azure.

В этом руководстве рассматриваются именно общедоступные зоны DNS. Сведения об использовании Azure PowerShell для управления частным зонами в Azure DNS см. в статье [Приступая к работе с частными зонами Azure DNS с помощью PowerShell](private-dns-getstarted-powershell.md).

[!INCLUDE [dns-create-zone-about](../../includes/dns-create-zone-about-include.md)]

[!INCLUDE [dns-powershell-setup](../../includes/dns-powershell-setup-include.md)]


## <a name="create-a-dns-zone"></a>Создание зоны DNS

Зона DNS создается с помощью командлета `New-AzureRmDnsZone`.

В следующем примере создается зона DNS *contoso.com* в группе ресурсов *MyResourceGroup*:

```powershell
New-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyAzureResourceGroup
```

В следующем примере демонстрируется создание зоны DNS с двумя [тегами Azure Resource Manager](dns-zones-records.md#tags) (*project = demo* и *env = test*):

```powershell
New-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyAzureResourceGroup -Tag @{ project="demo"; env="test" }
```

Azure DNS также поддерживает частные зоны DNS.  Дополнительные сведения об использовании частных зон DNS см. в статье [Using Azure DNS for private domains](private-dns-overview.md) (Использование Azure DNS для частных доменов). Инструкции по созданию частной зоны DNS см. в статье [Get started with Azure DNS private zones using PowerShell](./private-dns-getstarted-powershell.md) (Начало работы с частными зонами Azure DNS с помощью PowerShell).

## <a name="get-a-dns-zone"></a>Получение зоны DNS

Чтобы получить зону DNS, используйте командлет `Get-AzureRmDnsZone` . Эта операция возвращает объект зоны DNS, соответствующий существующей зоне в Azure DNS. Объект содержит данные о зоне (например, число наборов записей), но не сами наборы записей (см. `Get-AzureRmDnsRecordSet`).

```powershell
Get-AzureRmDnsZone -Name contoso.com –ResourceGroupName MyAzureResourceGroup

Name                  : contoso.com
ResourceGroupName     : myresourcegroup
Etag                  : 00000003-0000-0000-8ec2-f4879750d201
Tags                  : {project, env}
NameServers           : {ns1-01.azure-dns.com., ns2-01.azure-dns.net., ns3-01.azure-dns.org.,
                        ns4-01.azure-dns.info.}
NumberOfRecordSets    : 2
MaxNumberOfRecordSets : 5000
```

## <a name="list-dns-zones"></a>Перечисление зон DNS

Если опустить имя зоны в командлете `Get-AzureRmDnsZone`, можно получить список всех зон в группе ресурсов. Эта операция возвращает массив объектов зоны.

```powershell
$zoneList = Get-AzureRmDnsZone -ResourceGroupName MyAzureResourceGroup
```

Если опустить имя зоны и имя группы ресурсов в командлете `Get-AzureRmDnsZone`, то можно получить список всех зон в подписке Azure.

```powershell
$zoneList = Get-AzureRmDnsZone
```

## <a name="update-a-dns-zone"></a>Обновление зоны DNS

Изменить ресурс зоны DNS можно с помощью командлета `Set-AzureRmDnsZone`. При этом ни один набор записей DNS в пределах зоны командлетом не обновляется (см. статью [Управление записями и наборами записей DNS с помощью PowerShell](dns-operations-recordsets.md)). Эта команда используется только для обновления свойств самого ресурса зоны. В настоящее время к записываемым свойствам зоны относятся только [теги Azure Resource Manager для ресурса зоны](dns-zones-records.md#tags).

Используйте один из двух описанных ниже способов обновления зоны DNS.

### <a name="specify-the-zone-using-the-zone-name-and-resource-group"></a>Укажите зону с помощью имени и группы ресурсов.

При этом подходе существующие теги зоны заменяются указанными значениями.

```powershell
Set-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyAzureResourceGroup -Tag @{ project="demo"; env="test" }
```

### <a name="specify-the-zone-using-a-zone-object"></a>Укажите зону с помощью объекта $zone.

При этом подходе извлекается существующий объект зоны, изменяются теги, а затем фиксируются изменения. Благодаря этому существующие теги могут сохраняться.

```powershell
# Get the zone object
$zone = Get-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyAzureResourceGroup

# Remove an existing tag
$zone.Tags.Remove("project")

# Add a new tag
$zone.Tags.Add("status","approved")

# Commit changes
Set-AzureRmDnsZone -Zone $zone
```

При использовании командлета `Set-AzureRmDnsZone` с объектом $zone применяются [проверки Etag](dns-zones-records.md#etags), чтобы параллельные изменения не были перезаписаны. Чтобы отменить эти проверки, укажите необязательный параметр `-Overwrite`.

## <a name="delete-a-dns-zone"></a>Удаление зоны DNS

Зоны DNS можно удалить с помощью командлета `Remove-AzureRmDnsZone`.

> [!NOTE]
> При удалении зоны DNS также удаляются все записи DNS в этой зоне. Отменить эту операцию невозможно. Если зона DNS используется, то после ее удаления произойдет сбой служб, которые ее используют.
>
>Сведения о защите от случайного зоны удаления см. в разделе [How to protect DNS zones and records](dns-protect-zones-recordsets.md) (Как защитить зоны и записи DNS).


Используйте один из двух описанных ниже способов удаления зоны DNS.

### <a name="specify-the-zone-using-the-zone-name-and-resource-group-name"></a>Укажите зону с помощью имени зоны и имени группы ресурсов.

```powershell
Remove-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyAzureResourceGroup
```

### <a name="specify-the-zone-using-a-zone-object"></a>Укажите зону с помощью объекта $zone.

Удаляемую зону можно указать с помощью объекта `$zone`, который возвращается командлетом `Get-AzureRmDnsZone`.

```powershell
$zone = Get-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyAzureResourceGroup
Remove-AzureRmDnsZone -Zone $zone
```

Объект зоны также можно направить в конвейер, а не передавать в качестве параметра:

```powershell
Get-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyAzureResourceGroup | Remove-AzureRmDnsZone

```

Как и для командлета `Set-AzureRmDnsZone`, при указании набора записей с помощью объекта `$zone` вы включаете проверки Etag, чтобы убедиться, что параллельные изменения не удаляются. Чтобы отменить эти проверки, используйте параметр `-Overwrite`.

## <a name="confirmation-prompts"></a>Запросы на подтверждение

Командлеты `New-AzureRmDnsZone`, `Set-AzureRmDnsZone` и `Remove-AzureRmDnsZone` поддерживают запросы на подтверждение.

Оба командлета (`New-AzureRmDnsZone` и `Set-AzureRmDnsZone`) запрашивают подтверждение, если привилегированная переменная `$ConfirmPreference` в PowerShell имеет значение `Medium` или ниже. Из-за того, что удаление зоны DNS может иметь значительные последствия, командлет `Remove-AzureRmDnsZone` запрашивает подтверждение, если переменная PowerShell `$ConfirmPreference` имеет любое значение, отличное от `None`.

Поскольку переменная `$ConfirmPreference` по умолчанию имеет значение `High`, то по умолчанию подтверждение запрашивается только командлетом `Remove-AzureRmDnsZone`.

Текущее значение `$ConfirmPreference` можно переопределить с помощью параметра `-Confirm`. Если вы определите `-Confirm` или `-Confirm:$True`, командлет будет запрашивать подтверждение перед выполнением. Если вы определите `-Confirm:$False`, командлет не будет запрашивать подтверждение.

Дополнительные сведения об элементах `-Confirm` и `$ConfirmPreference` см. в статье о [привилегированных переменных](/powershell/module/microsoft.powershell.core/about/about_preference_variables).

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как [управлять наборами записей и записями](dns-operations-recordsets.md) в зоне DNS.
<br>
Узнайте, как [делегировать свой домен в Azure DNS](dns-domain-delegation.md).
<br>
Просмотрите [справочную документацию по Azure DNS PowerShell](/powershell/module/azurerm.dns).