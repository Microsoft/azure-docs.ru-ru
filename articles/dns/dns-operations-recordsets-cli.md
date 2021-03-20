---
title: Управление записями DNS в Azure DNS с помощью интерфейса командной строки Azure
description: Управляйте наборами записей и записями DNS в службе Azure DNS при размещении вашего домена в Azure DNS.
author: rohinkoul
ms.assetid: 5356a3a5-8dec-44ac-9709-0c2b707f6cb5
ms.service: dns
ms.devlang: azurecli
ms.topic: how-to
ms.custom: H1Hack27Feb2017, devx-track-azurecli
ms.workload: infrastructure-services
ms.date: 05/15/2018
ms.author: rohink
ms.openlocfilehash: 2d3989b3c477a35d602f1ccf3e45d6f597f5d78d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96011577"
---
# <a name="manage-dns-records-and-recordsets-in-azure-dns-using-the-azure-cli"></a>Управление записями и наборами записей DNS в Azure DNS с помощью Azure CLI

> [!div class="op_single_selector"]
> * [Портал Azure](dns-operations-recordsets-portal.md)
> * [Azure CLI](dns-operations-recordsets-cli.md)
> * [PowerShell](dns-operations-recordsets.md)

В этой статье показано, как управлять записями DNS для зоны DNS с помощью кроссплатформенного Azure CLI, доступного для Windows, Mac и Linux. Записями DNS также можно управлять с помощью [Azure PowerShell](dns-operations-recordsets.md) или [портала Azure](dns-operations-recordsets-portal.md).

Для работы с этой статьей необходимо [установить интерфейс командной строки Azure, войти в учетную запись и создать зону DNS](dns-operations-dnszones-cli.md).

## <a name="introduction"></a>Введение

Чтобы создавать записи DNS в Azure DNS, нужно понимать, как Azure DNS организует записи DNS в соответствующие наборы записей.

[!INCLUDE [dns-about-records-include](../../includes/dns-about-records-include.md)]

Дополнительные сведения о записях DNS в Azure DNS см. в статье [Зоны и записи DNS](dns-zones-records.md).

## <a name="create-a-dns-record"></a>Создание записи DNS

Чтобы создать запись DNS, используйте команду `az network dns record-set <record-type> add-record` (где `<record-type>` — это тип записи, например a, SRV, txt и т. д.) Справочные сведения см `az network dns record-set --help` . в разделе.

Создавая запись, вам нужно определить для нее имя группы ресурсов, имя зоны, имя набора записей, тип записей и сведения о создаваемой записи. Имя набора записей должно быть *относительным*, т. е. оно не должно содержать имя зоны.

Если набора записей не существует, эта команда автоматически создаст его. Если набор записей существует, эта команда добавляет в него указанную запись.

При создании набора записей используется срок жизни (TTL) по умолчанию — 3600. Инструкции по использованию различных сроков жизни см. в разделе [Создание набора записей DNS](#create-a-dns-record-set).

В следующем примере будет создана запись A с именем *www* в зоне *contoso.com* в группе ресурсов *MyResourceGroup*. IP-адрес записи A — *1.2.3.4*.

```azurecli
az network dns record-set a add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name www --ipv4-address 1.2.3.4
```

Чтобы создать набор записей на вершине зоны (в нашем примере — contoso.com), используйте имя записи "\@", включая кавычки:

```azurecli
az network dns record-set a add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name "@" --ipv4-address 1.2.3.4
```

## <a name="create-a-dns-record-set"></a>Создание набора записей DNS

В приведенных выше примерах запись DNS добавлялась к существующему набору записей, или набор записей создавался *неявно*. Кроме того, можно *явно* создать набор записей, прежде чем добавлять в него записи. В Azure DNS поддерживаются пустые наборы записей, которые могут использоваться как заполнители для резервирования имен DNS перед созданием записей DNS. Пустые наборы записей отображаются на панели управления Azure DNS, но не отображаются на серверах имен Azure DNS.

Наборы записей создаются с помощью команды `az network dns record-set <record-type> create`. Чтобы получить справку, см. `az network dns record-set <record-type> create --help`.

При создании набора записей можно явно указать его свойства, например [срок жизни](dns-zones-records.md#time-to-live) и метаданные. [Метаданные набора записей](dns-zones-records.md#tags-and-metadata) используются для связывания данных приложения с каждым набором записей в виде пар "ключ — значение".

В следующем примере показано, как создать пустой набор записей типа А со сроком жизни 60 секунд. Для этого используется параметр `--ttl` (краткая форма `-l`).

```azurecli
az network dns record-set a create --resource-group myresourcegroup --zone-name contoso.com --name www --ttl 60
```

В следующем примере показано, как создать набор с двумя записями метаданных (dept=finance и environment=production) с помощью параметра `--metadata`:

```azurecli
az network dns record-set a create --resource-group myresourcegroup --zone-name contoso.com --name www --metadata "dept=finance" "environment=production"
```

После создания пустого набора записей в него можно добавить записи с помощью команды `azure network dns record-set <record-type> add-record`, как описано в разделе [Создание записи DNS](#create-a-dns-record).

## <a name="create-records-of-other-types"></a>Создание записей других типов

Вы уже узнали, как создавать записи типа А. В следующем примере показано, как создавать записи других типов, поддерживаемые в Azure DNS.

Параметры, используемые для указания данных записи, различаются в зависимости от типа записи. Например, для записи типа A необходимо указать адрес IPv4, используя параметр `--ipv4-address <IPv4 address>`. Параметры для каждого типа записи можно перечислить с помощью команды `az network dns record-set <record-type> add-record --help`.

В каждом случае мы покажем, как создать одну запись. Запись добавляется в существующий набор или набор, созданный неявно. Дополнительные сведения о создании наборов записей и явной настройке параметра набора см. в разделе [Создание набора записей DNS](#create-a-dns-record-set).

Мы не включаем пример создания набора записей типа SOA, так как такие записи создаются и удаляются только вместе с соответствующей зоной DNS. Тем не менее [записи типа SOA можно изменять, как показано в примере ниже](#to-modify-an-soa-record).

### <a name="create-an-aaaa-record"></a>Создание записи типа AAAA

```azurecli
az network dns record-set aaaa add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name test-aaaa --ipv6-address 2607:f8b0:4009:1803::1005
```

### <a name="create-an-caa-record"></a>Создание записи типа CAA

```azurecli
az network dns record-set caa add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name test-caa --flags 0 --tag "issue" --value "ca1.contoso.com"
```

### <a name="create-a-cname-record"></a>Создание записи CNAME

> [!NOTE]
> Стандарты DNS не допускают использование записей типа CNAME на вершине зоны (`--Name "@"`), а также использование наборов записей, содержащих более одной записи.
> 
> Дополнительные сведения см. в разделе [Записи типа CNAME](dns-zones-records.md#cname-records).

```azurecli
az network dns record-set cname set-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name test-cname --cname www.contoso.com
```

### <a name="create-an-mx-record"></a>Создание записи типа MX

Чтобы создать запись MX на вершине зоны (в данном случае — contoso.com), в этом примере мы используем имя набора записей "\@".

```azurecli
az network dns record-set mx add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name "@" --exchange mail.contoso.com --preference 5
```

### <a name="create-an-ns-record"></a>Создание записи типа NS

```azurecli
az network dns record-set ns add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name test-ns --nsdname ns1.contoso.com
```

### <a name="create-a-ptr-record"></a>Создание записи типа PTR

В этом случае my-arpa-zone.com представляет зону ARPA вашего диапазона IP-адресов. Каждая запись PTR в этой зоне соответствует IP-адресу в этом диапазоне.  Имя записи 10 — это последний октет IP-адреса в этом диапазоне IP-адресов, представленном данной записью.

```azurecli
az network dns record-set ptr add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name my-arpa.zone.com --ptrdname myservice.contoso.com
```

### <a name="create-an-srv-record"></a>Создание записи типа SRV

Создавая [набор записей SRV](dns-zones-records.md#srv-records), укажите в его имени *\_службу* и *\_протокол*. Если набор записей SRV создается на вершине зоны, добавлять "\@" в имя набора записей не нужно.

```azurecli
az network dns record-set srv add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name _sip._tls --priority 10 --weight 5 --port 8080 --target sip.contoso.com
```

### <a name="create-a-txt-record"></a>Создание записи типа TXT

В следующем примере показано, как создать запись типа ТХТ. Дополнительные сведения о максимальной длине строки, поддерживаемой в записях типа TXT, см. в разделе [Записи типа TXT](dns-zones-records.md#txt-records).

```azurecli
az network dns record-set txt add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name test-txt --value "This is a TXT record"
```

## <a name="get-a-record-set"></a>Получение набора записей

Чтобы извлечь существующий набор записей, используйте команду `az network dns record-set <record-type> show`. Чтобы получить справку, см. `az network dns record-set <record-type> show --help`.

Как и при создании записи или набора записей, имя набора должно быть *относительным*, т. е. оно не должно содержать имя зоны. Необходимо также указать тип записи, зону, содержащую набор записей, и группу ресурсов, содержащую зону.

В следующем примере показано получение записи *www* типа А из зоны *contoso.com* в группе ресурсов *MyResourceGroup*.

```azurecli
az network dns record-set a show --resource-group myresourcegroup --zone-name contoso.com --name www
```

## <a name="list-record-sets"></a>Перечисление наборов записей

Команда `az network dns record-set list` позволяет получить список всех записей в зоне DNS. Чтобы получить справку, см. `az network dns record-set list --help`.

В этом примере возвращаются все наборы записей в зоне *contoso.com* в группе ресурсов *MyResourceGroup* вне зависимости от имени и типа записи:

```azurecli
az network dns record-set list --resource-group myresourcegroup --zone-name contoso.com
```

В этом примере возвращаются все наборы записей, соответствующие заданному типу записи (в данном случае это записи типа A).

```azurecli
az network dns record-set a list --resource-group myresourcegroup --zone-name contoso.com 
```

## <a name="add-a-record-to-an-existing-record-set"></a>Добавление записи в существующий набор записей

Команду `az network dns record-set <record-type> add-record` можно использовать как для создания записи в новом наборе записей, так и для добавления записи в существующий набор.

Дополнительные сведения см. в разделах [Создание записи DNS](#create-a-dns-record) и [Создание записей других типов](#create-records-of-other-types) выше.

## <a name="remove-a-record-from-an-existing-record-set"></a>Удаление записи из существующего набора записей

Для удаления записи DNS из существующего набора используйте команду `az network dns record-set <record-type> remove-record`. Чтобы получить справку, см. `az network dns record-set <record-type> remove-record -h`.

Эта команда удаляет запись DNS из набора записей. Удаление последней записи в наборе не приводит к удалению самого набора. Чтобы сохранить вместо этого пустой набор записей, используйте параметр `--keep-empty-record-set`.

Необходимо указать удаляемую запись и зону, из которой ее следует удалить, используя те же параметры, что и при создании записи с помощью команды `az network dns record-set <record-type> add-record`. Эти параметры описаны в предыдущих разделах [Создание записи DNS](#create-a-dns-record) и [Создание записей других типов](#create-records-of-other-types).

В следующем примере показано удаление записи типа A со значением "1.2.3.4" из набора записей с именем *www* в зоне *contoso.com* в группе ресурсов *MyResourceGroup*.

```azurecli
az network dns record-set a remove-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name "www" --ipv4-address 1.2.3.4
```

## <a name="modify-an-existing-record-set"></a>Обновление существующего набора записей

В каждом наборе записей указан [срок жизни](dns-zones-records.md#time-to-live), [метаданные](dns-zones-records.md#tags-and-metadata) и записи DNS. В следующих разделах описано, как изменять эти свойства.

### <a name="to-modify-an-a-aaaa-caa-mx-ns-ptr-srv-or-txt-record"></a>Изменение записи типа A, AAAA, CAA, MX, NS, PTR, SRV или TXT

Чтобы изменить существующую запись типа A, AAAA, CAA, MX, NS, PTR, SRV или TXT, сначала следует добавить новую запись, а затем удалить существующую. Подробные инструкции об удалении и добавлении записей см. в предыдущих разделах этой статьи.

В следующем примере показано, как в записи типа А изменить IP-адрес 1.2.3.4 на IP-адрес 5.6.7.8.

```azurecli
az network dns record-set a add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name www --ipv4-address 5.6.7.8
az network dns record-set a remove-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name www --ipv4-address 1.2.3.4
```

В автоматически созданном наборе записей типа NS на вершине зоны (`--Name "@"`, включая кавычки) добавлять, удалять или изменять записи нельзя. В этом наборе можно изменить только срок жизни и метаданные набора записей.

### <a name="to-modify-a-cname-record"></a>Изменение записи CNAME

В отличие от большинства других типов записей, набор записей CNAME может содержать только одну запись.  Следовательно, вы не можете заменить текущее значение, добавив новую запись и удалив существующую запись, как и для других типов записей.

Вместо этого используйте `az network dns record-set cname set-record`, чтобы изменить запись типа CNAME. Чтобы получить справку, см. `az network dns record-set cname set-record --help`.

В примере ниже показано изменение набора записей CNAME с именем *www* в зоне *contoso.com* в группе ресурсов *MyResourceGroup* таким образом, чтобы он указывал на www.fabrikam.net вместо существующего значения.

```azurecli
az network dns record-set cname set-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name test-cname --cname www.fabrikam.net
``` 

### <a name="to-modify-an-soa-record"></a>Изменение записи типа SOA

В отличие от большинства других типов записей, набор записей CNAME может содержать только одну запись.  Следовательно, вы не можете заменить текущее значение, добавив новую запись и удалив существующую запись, как и для других типов записей.

Вместо этого используйте `az network dns record-set soa update`, чтобы изменить запись типа SOA. Чтобы получить справку, см. `az network dns record-set soa update --help`.

В примере ниже показано, как задать свойство email записи типа SOA для зоны *contoso.com* в группе ресурсов *MyResourceGroup*.

```azurecli
az network dns record-set soa update --resource-group myresourcegroup --zone-name contoso.com --email admin.contoso.com
```

### <a name="to-modify-ns-records-at-the-zone-apex"></a>Изменение записи NS на вершине зоны

Набор записей типа NS на вершине зоны автоматически создается вместе с каждой зоной DNS. Он содержит имена DNS-серверов Azure, назначенные зоне.

Вы можете добавить дополнительные имена серверов в этот набор записей NS, обеспечив поддержку совместного размещения доменов с использованием более чем одного поставщика DNS. Вы также можете изменить срок жизни и метаданные для этого набора записей. При этом вы не можете удалить или изменить предварительно заполненные серверы доменных имен Azure DNS.

Обратите внимание, что это относится только к набору записей NS на вершине зоны. Другие наборы записей NS в зоне (используемые для делегирования дочерних зон) можно изменять без ограничений.

В следующем примере показано, как добавить дополнительный сервер в набор записей NS на вершине зоны:

```azurecli
az network dns record-set ns add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name "@" --nsdname ns1.myotherdnsprovider.com 
```

### <a name="to-modify-the-ttl-of-an-existing-record-set"></a>Изменение срока жизни существующего набора записей

Для изменения срока жизни существующего набора записей используйте команду `azure network dns record-set <record-type> update`. Чтобы получить справку, см. `azure network dns record-set <record-type> update --help`.

В примере ниже показано, как изменить срок жизни набора записей. В этом случае устанавливается срок в 60 секунд.

```azurecli
az network dns record-set a update --resource-group myresourcegroup --zone-name contoso.com --name www --set ttl=60
```

### <a name="to-modify-the-metadata-of-an-existing-record-set"></a>Изменение метаданных существующего набора записей

[Метаданные набора записей](dns-zones-records.md#tags-and-metadata) используются для связывания данных приложения с каждым набором записей в виде пар "ключ — значение". Для изменения метаданных существующего набора записей используйте команду `az network dns record-set <record-type> update`. Чтобы получить справку, см. `az network dns record-set <record-type> update --help`.

В следующем примере показано, как изменить набор с двумя записями метаданных: dept=finance и environment=production. Обратите внимание, что все существующие метаданные *заменяются* заданными значениями.

```azurecli
az network dns record-set a update --resource-group myresourcegroup --zone-name contoso.com --name www --set metadata.dept=finance metadata.environment=production
```

## <a name="delete-a-record-set"></a>Удаление набора записей

Наборы записей можно удалить с помощью команды `az network dns record-set <record-type> delete`. Чтобы получить справку, см. `azure network dns record-set <record-type> delete --help`. При удалении набора записей также удаляются все содержащиеся в нем записи.

> [!NOTE]
> Удалить наборы записей типа SOA и NS на вершине зоны (`--name "@"`) нельзя.  Эти записи создаются и удаляются автоматически вместе с зоной.

В следующем примере показано удаление набора записей с именем *www* типа А из зоны *contoso.com* в группе ресурсов *MyResourceGroup*.

```azurecli
az network dns record-set a delete --resource-group myresourcegroup --zone-name contoso.com --name www
```

Отобразится запрос на подтверждение операции удаления. Чтобы скрыть этот запрос, используйте параметр `--yes`.

## <a name="next-steps"></a>Дальнейшие действия

См. дополнительные сведения о [зонах и записях в Azure DNS](dns-zones-records.md).
<br>
Узнайте, как [защитить зоны и записи](dns-protect-zones-recordsets.md) при использовании Azure DNS.
