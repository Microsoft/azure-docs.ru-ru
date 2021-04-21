---
title: Краткое руководство. Создание зоны и записи Azure DNS — Azure PowerShell
titleSuffix: Azure DNS
description: Узнайте, как создать зону и запись DNS в службе DNS Azure. В этом пошаговом руководстве описано, как создать зону и запись DNS, а также как управлять ими с помощью Azure PowerShell.
services: dns
author: rohinkoul
ms.author: rohink
ms.date: 10/20/2020
ms.topic: quickstart
ms.service: dns
ms.custom:
- devx-track-azurepowershell
- mode-api
ms.openlocfilehash: 9c457b435ca17ce9dc86e1a83a4d1a56d9623b17
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107537879"
---
# <a name="quickstart-create-an-azure-dns-zone-and-record-using-azure-powershell"></a>Краткое руководство. Создание зоны и записи Azure DNS с помощью Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

В этом кратком руководстве описано, как создать зону и запись Azure DNS с помощью Azure PowerShell. Эти действия также можно выполнить с помощью [портала Azure](dns-getstarted-portal.md) или [Azure CLI](dns-getstarted-cli.md). 

Зона DNS используется для размещения DNS-записей определенного домена. Чтобы разместить свой домен в Azure DNS, необходимо создать зону DNS для этого доменного имени. Каждая запись DNS для вашего домена создается внутри этой зоны DNS. Наконец, чтобы опубликовать зону DNS в Интернете, необходимо настроить серверы доменных имен для домена. Каждый из этих шагов описан ниже.

Azure DNS также поддерживает создание частных доменов. Пошаговые инструкции о том, как создать закрытую зону DNS и запись, см. в разделе [Приступая к работе с частными зонами Azure DNS с помощью PowerShell](private-dns-getstarted-powershell.md).

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- Локальная установка Azure PowerShell или Azure Cloud Shell

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-the-resource-group"></a>Создание группы ресурсов

Перед созданием зоны DNS создайте группу ресурсов, которая будет включать эту зону DNS:

```powershell
New-AzResourceGroup -name MyResourceGroup -location "eastus"
```

## <a name="create-a-dns-zone"></a>Создание зоны DNS

Зона DNS создается с помощью командлета `New-AzDnsZone`. В следующем примере будет создана зона DNS *contoso.xyz* в группе ресурсов *MyResourceGroup*. Пример можно использовать для создания зоны DNS, заменив соответствующие значения собственными.

```powershell
New-AzDnsZone -Name contoso.xyz -ResourceGroupName MyResourceGroup
```

## <a name="create-a-dns-record"></a>Создание записи DNS

Для создания наборов записей используется командлет `New-AzDnsRecordSet`. В следующем примере создается запись с относительным именем www в зоне DNS contoso.xyz, в группе ресурсов MyResourceGroup. Полное доменное имя набора записей — "www.contoso.xyz". Тип записи — A, IP-адрес — 10.10.10.10, срок жизни составляет 3600 секунд.

```powershell
New-AzDnsRecordSet -Name www -RecordType A -ZoneName contoso.xyz -ResourceGroupName MyResourceGroup -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -IPv4Address "10.10.10.10")
```

## <a name="view-records"></a>Просмотр записей

Чтобы просмотреть список записей DNS в зоне, используйте следующую команду:

```powershell
Get-AzDnsRecordSet -ZoneName contoso.xyz -ResourceGroupName MyResourceGroup
```

## <a name="test-the-name-resolution"></a>Тестирование разрешения имен

Теперь, когда у вас есть тестовая зона DNS с тестовой записью "A", можно протестировать разрешение имен с помощью инструмента *nslookup*. 

**Тестирование разрешения DNS-имени.**

1. Выполните следующий командлет, чтобы получить список серверов имен для вашей зоны.

   ```azurepowershell
   Get-AzDnsRecordSet -ZoneName contoso.xyz -ResourceGroupName MyResourceGroup -RecordType ns
   ```

1. Скопируйте имя одного из серверов доменных имен из выходных данных, полученных на предыдущем этапе.

1. Откройте командную строку и выполните приведенную ниже команду.

   ```
   nslookup www.contoso.xyz <name server name>
   ```

   Пример:

   ```
   nslookup www.contoso.xyz ns1-08.azure-dns.com.
   ```

   Результат будет подобным приведенному на следующем снимке экрана.

   ![Снимок экрана: окно командной строки с результатом выполнения команды nslookup — значениями параметров Server (Сервер), Address (Адрес), Name (Имя) и Address (Адрес)](media/dns-getstarted-portal/nslookup.PNG)

Имя узла **www\.contoso.xyz** разрешается как **10.10.10.10**, как и было настроено. Такой результат подтверждает, что разрешение имен работает правильно.

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы очистить ненужные ресурсы, созданные при работе с этим кратким руководством, удалите группу ресурсов:

```powershell
Remove-AzResourceGroup -Name MyResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы создали свою первую зону и запись DNS с помощью Azure PowerShell, можно создать записи для веб-приложения в личном домене.

> [!div class="nextstepaction"]
> [Создание записей DNS для веб-приложения в пользовательском домене](./dns-web-sites-custom-domain.md)
