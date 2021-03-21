---
title: Настройка сетевых конечных точек Файлов Azure | Документация Майкрософт
description: Узнайте, как настроить конечные точки файловой сети Azure.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 12/04/2020
ms.author: rogarana
ms.subservice: files
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 079d7aa9b654a318c7269a41605c3e146b08f127
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96621337"
---
# <a name="configuring-azure-files-network-endpoints"></a>Настройка сетевых конечных точек Файлов Azure

Файлы Azure предоставляют два основных типа конечных точек для доступа к общим папкам Azure: 
- общедоступные конечные точки, у которых есть общедоступный IP-адрес и к которым можно обращаться из любой точки мира;
- частные конечные точки, которые существуют только в виртуальной сети и имеют частный IP-адрес в пределах адресного пространства этой виртуальной сети.

Общедоступные и частные конечные точки размещаются в учетной записи хранения Azure. Учетная запись хранения — это конструкция управления, представляющая собой общий пул носителей, который можно использовать для развертывания нескольких общих папок и других ресурсов хранения, например контейнеров больших двоичных объектов или очередей.

В этой статье показано, как настроить конечные точки учетной записи хранения для получения доступа к общей папке Azure напрямую. Здесь также описываются особенности взаимодействия между Синхронизацией файлов Azure и общедоступными и частными конечными точками учетной записи хранения. См. сведения о требованиях к сети для развертывания Синхронизации файлов Azure в описании [параметров брандмауэра и прокси-сервера Синхронизации файлов Azure](storage-sync-files-firewall-and-proxy.md).

Мы рекомендуем ознакомиться с [рекомендациями по работе с сетью в Файлах Azure](storage-files-networking-overview.md) перед изучением этого руководства.

## <a name="prerequisites"></a>Предварительные требования

- В этой статье подразумевается, что вы уже создали подписку Azure. Если у вас еще нет подписки, вы можете [создать бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.
- В этой статье предполагается, что вы уже создали файловый ресурс Azure в учетной записи хранения, к которой вы хотите подключиться из локальной среды. Чтобы узнать, как создать общую папку Azure, ознакомьтесь с [этой статьей](storage-how-to-create-file-share.md).
- Если вы хотите использовать Azure PowerShell, [установите последнюю версию](/powershell/azure/install-az-ps).
- Если вы хотите использовать Azure CLI, [установите последнюю версию](/cli/azure/install-azure-cli).

## <a name="endpoint-configurations"></a>Конфигурации конечных точек

Вы можете настроить конечные точки, чтобы ограничить сетевой доступ к учетной записи хранения. Вы можете использовать два подхода к ограничению доступа к учетной записи хранения из виртуальных сетей.

- [Создайте одну или несколько частных конечных точек для учетной записи хранения](#create-a-private-endpoint) и запретите любой доступ к общедоступной конечной точке. Так вы будете уверены, что доступ к общим папкам Azure в этой учетной записи хранения возможен только для запросов, создаваемых внутри настроенных виртуальных сетей.
- [Ограничьте общедоступную конечную точку одной или несколькими виртуальными сетями](#restrict-public-endpoint-access). Это достигается за счет возможности виртуальной сети под названием *конечная точка службы*. Если вы ограничиваете трафик к учетной записи хранения через конечную точку службы, вы по-прежнему обращаетесь к учетной записи хранения через общедоступный IP-адрес, но доступ возможен только из расположений, указанных в конфигурации.

### <a name="create-a-private-endpoint"></a>Создание частной конечной точки

Создание частной конечной точки для учетной записи хранения приведет к развертыванию следующих ресурсов Azure:

- **Частная конечная точка.** Ресурс Azure, представляющий частную конечную точку учетной записи хранения. Его можно рассматривать как ресурс для подключения учетной записи хранения к сетевому интерфейсу.
- **Сетевой интерфейс.** Сетевой интерфейс с поддержкой частного IP-адреса в указанной виртуальной сети или подсети. Это тот же ресурс, который автоматически развертывается при развертывании виртуальной машины, но он назначается не виртуальной машине, а частной конечной точке.
- **Частная зона DNS.** Если вы никогда не развертывали частную конечную точку для этой виртуальной сети, для нее будет развернута новая частная зона DNS. В этой зоне DNS будет создана запись DNS типа A для учетной записи хранения. Если вы уже развертывали частную конечную точку в этой виртуальной сети, новая запись типа A для учетной записи хранения будет добавлена в существующую зону DNS. Развертывание зоны DNS не является обязательным, но мы настоятельно рекомендуем его выполнять. Это обязательное действие при подключении общих папок Azure с помощью субъекта-службы AD или API FileREST.

> [!Note]  
> В этой статье используется DNS-суффикс учетной записи хранения `core.windows.net`, выделенный для общедоступных регионов Azure. Этот комментарий применим и к национальным облакам Azure, таким как облако Azure для государственных организаций США и облако Azure для Китая. Вам придется лишь заменить суффиксы на другие, соответствующие вашей среде. 

# <a name="portal"></a>[Портал](#tab/azure-portal)
[!INCLUDE [storage-files-networking-endpoints-private-portal](../../../includes/storage-files-networking-endpoints-private-portal.md)]

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
[!INCLUDE [storage-files-networking-endpoints-private-powershell](../../../includes/storage-files-networking-endpoints-private-powershell.md)]

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
[!INCLUDE [storage-files-networking-endpoints-private-cli](../../../includes/storage-files-networking-endpoints-private-cli.md)]
---

## <a name="verify-connectivity"></a>Проверка подключения

# <a name="portal"></a>[Портал](#tab/azure-portal)

Если у вас есть виртуальная машина в виртуальной сети или настроена переадресация DNS, как описано в разделе [Настройка переадресации DNS для Файлов Azure](storage-files-networking-dns.md), правильность настройки частной конечной точки можно проверить, выполнив следующие команды в PowerShell, командной строке или терминале (работает для Windows, Linux или macOS). Замените `<storage-account-name>` реальным именем учетной записи хранения.

```
nslookup <storage-account-name>.file.core.windows.net
```

Если все пройдет успешно, вы увидите следующие выходные данные, где `192.168.0.5` обозначает частный IP-адрес частной конечной точки в виртуальной сети (здесь представлены выходные данные в среде Windows):

```Output
Server:  UnKnown
Address:  10.2.4.4

Non-authoritative answer:
Name:    storageaccount.privatelink.file.core.windows.net
Address:  192.168.0.5
Aliases:  storageaccount.file.core.windows.net
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Если у вас есть виртуальная машина в виртуальной сети или настроена переадресация DNS, как описано в разделе [Настройка переадресации DNS для Файлов Azure](storage-files-networking-dns.md), правильность настройки частной конечной точки можно проверить, выполнив следующие команды:

```PowerShell
$storageAccountHostName = [System.Uri]::new($storageAccount.PrimaryEndpoints.file) | `
    Select-Object -ExpandProperty Host

Resolve-DnsName -Name $storageAccountHostName
```

Если все пройдет успешно, вы увидите следующие выходные данные, где `192.168.0.5` обозначает частный IP-адрес частной конечной точки в виртуальной сети:

```Output
Name                             Type   TTL   Section    NameHost
----                             ----   ---   -------    --------
storageaccount.file.core.windows CNAME  60    Answer     storageaccount.privatelink.file.core.windows.net
.net

Name       : storageaccount.privatelink.file.core.windows.net
QueryType  : A
TTL        : 600
Section    : Answer
IP4Address : 192.168.0.5
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Если у вас есть виртуальная машина в виртуальной сети или настроена переадресация DNS, как описано в разделе [Настройка переадресации DNS для Файлов Azure](storage-files-networking-dns.md), правильность настройки частной конечной точки можно проверить, выполнив следующие команды:

```bash
httpEndpoint=$(az storage account show \
        --resource-group $storageAccountResourceGroupName \
        --name $storageAccountName \
        --query "primaryEndpoints.file" | \
    tr -d '"')

hostName=$(echo $httpEndpoint | cut -c7-$(expr length $httpEndpoint) | tr -d "/")
nslookup $hostName
```

Если все пройдет успешно, вы увидите следующие выходные данные, где `192.168.0.5` обозначает частный IP-адрес частной конечной точки в виртуальной сети: По-прежнему следует использовать storageaccount.file.core.windows.net для подключения общей папки вместо `privatelink` пути.

```Output
Server:         127.0.0.53
Address:        127.0.0.53#53

Non-authoritative answer:
storageaccount.file.core.windows.net      canonical name = storageaccount.privatelink.file.core.windows.net.
Name:   storageaccount.privatelink.file.core.windows.net
Address: 192.168.0.5
```
---

## <a name="restrict-public-endpoint-access"></a>Ограничить доступ к общедоступной конечной точке

Для ограничения доступа к общедоступной конечной точке сначала необходимо отключить общий доступ к общедоступной конечной точке. Отключение доступа к общедоступной конечной точке не влияет на частные конечные точки. После отключения общедоступной конечной точки можно выбрать определенные сети или IP-адреса, которые могут продолжать обращаться к ней. Как правило, большинство политик брандмауэра для учетной записи хранения ограничивают сетевой доступ к одной или нескольким виртуальным сетям.

#### <a name="disable-access-to-the-public-endpoint"></a>Отключение доступа к общедоступной конечной точке

Если доступ к общедоступной конечной точке отключен, к учетной записи хранения можно обращаться только через частные конечные точки. В противном случае допустимые запросы к общедоступной конечной точке учетной записи хранения будут отклонены, если они не относятся [к специально разрешенному источнику](#restrict-access-to-the-public-endpoint-to-specific-virtual-networks). 

# <a name="portal"></a>[Портал](#tab/azure-portal)
[!INCLUDE [storage-files-networking-endpoints-public-disable-portal](../../../includes/storage-files-networking-endpoints-public-disable-portal.md)]

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
[!INCLUDE [storage-files-networking-endpoints-public-disable-powershell](../../../includes/storage-files-networking-endpoints-public-disable-powershell.md)]

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
[!INCLUDE [storage-files-networking-endpoints-public-disable-cli](../../../includes/storage-files-networking-endpoints-public-disable-cli.md)]

---

#### <a name="restrict-access-to-the-public-endpoint-to-specific-virtual-networks"></a>Ограничьте доступ к общедоступной конечной точке определенными виртуальными сетями.

Определив доступ к учетной записи хранения для определенных виртуальных сетей, вы разрешите доступ к общедоступной конечной точке только из этих виртуальных сетей. Это достигается за счет возможности виртуальной сети под названием *конечная точка службы*. Ее можно использовать с частными конечными точками или без них.

# <a name="portal"></a>[Портал](#tab/azure-portal)
[!INCLUDE [storage-files-networking-endpoints-public-restrict-portal](../../../includes/storage-files-networking-endpoints-public-restrict-portal.md)]

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
[!INCLUDE [storage-files-networking-endpoints-public-restrict-powershell](../../../includes/storage-files-networking-endpoints-public-restrict-powershell.md)]

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
[!INCLUDE [storage-files-networking-endpoints-public-restrict-cli](../../../includes/storage-files-networking-endpoints-public-restrict-cli.md)]

---

## <a name="see-also"></a>См. также раздел

- [Рекомендации по работе с сетями Файлов Azure](storage-files-networking-overview.md)
- [Настройка перенаправления DNS для Файлов Azure](storage-files-networking-dns.md)
- [Настройка S2S VPN для Файлов Azure](storage-files-configure-s2s-vpn.md)