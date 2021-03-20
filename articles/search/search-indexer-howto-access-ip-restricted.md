---
title: Разрешить доступ к диапазонам IP-адресов индексатора
titleSuffix: Azure Cognitive Search
description: Настройте правила брандмауэра IP-адресов, чтобы разрешить доступ к данным с помощью индексатора Azure Когнитивный поиск.
manager: nitinme
author: arv100kri
ms.author: arjagann
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 10/14/2020
ms.openlocfilehash: 01a88be9c54a2701130daace26c44159ee364e4c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98757249"
---
# <a name="configure-ip-firewall-rules-to-allow-indexer-connections-azure-cognitive-search"></a>Настройка правил брандмауэра IP-адресов для разрешения соединений индексатора (Когнитивный поиск Azure)

Правила брандмауэра IP-адресов для ресурсов Azure, таких как учетные записи хранения, учетные записи Cosmos DB и серверы SQL Azure, допускают только трафик, исходящий из конкретных диапазонов IP-адресов для доступа к данным.

В этой статье описывается, как настроить правила IP-адресов с помощью портал Azure для учетной записи хранения, чтобы Azure Когнитивный поиск индексаторы могли безопасно получать доступ к данным. Как и для службы хранилища Azure, подход также работает для других ресурсов Azure, использующих правила IP-брандмауэра для защиты доступа к данным.

> [!NOTE]
> Правила брандмауэра IP для учетной записи хранения вступают в силу только в том случае, если учетная запись хранения и служба поиска находятся в разных регионах. Если программа установки не позволяет это сделать, рекомендуется использовать [параметр Trusted Service Exception](search-indexer-howto-access-trusted-service-exception.md).

## <a name="get-the-ip-address-of-the-search-service"></a>Получение IP-адреса службы поиска

Получите полное доменное имя (FQDN) службы поиска. Это будет выглядеть так `<search-service-name>.search.windows.net` :. Полное доменное имя можно узнать, найдя службу поиска на портал Azure.

   ![Получение полного доменного имени службы](media\search-indexer-howto-secure-access\search-service-portal.png "Получение полного доменного имени службы")

IP-адрес службы поиска можно получить, выполнив `nslookup` (или `ping` ) полное доменное имя. В приведенном ниже примере вы добавите "150.0.0.1" в правило входящего трафика в брандмауэре службы хранилища Azure. После обновления параметров брандмауэра, чтобы индексатор службы поиска мог получить доступ к учетной записи хранения Azure, может потребоваться до 15 минут.

```azurepowershell

nslookup contoso.search.windows.net
Server:  server.example.org
Address:  10.50.10.50

Non-authoritative answer:
Name:    <name>
Address:  150.0.0.1
Aliases:  contoso.search.windows.net
```

## <a name="get-the-ip-address-ranges-for-azurecognitivesearch-service-tag"></a>Получение диапазонов IP-адресов для тега службы "Азурекогнитивесеарч"

Дополнительные IP-адреса используются для запросов, исходящих из [среды выполнения в нескольких клиентах](search-indexer-securing-resources.md#indexer-execution-environment)индексатора. Этот диапазон IP-адресов можно получить из тега службы.

Диапазоны IP-адресов для `AzureCognitiveSearch` тега службы можно получить с помощью [API обнаружения (Предварительная версия)](../virtual-network/service-tags-overview.md#use-the-service-tag-discovery-api-public-preview) или [загружаемого файла JSON](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files).

Для этого пошагового руководства при условии, что служба поиска является общедоступным облаком Azure, необходимо скачать [открытый JSON-файл Azure](https://www.microsoft.com/download/details.aspx?id=56519) .

   ![Скачать JSON файл](media\search-indexer-howto-secure-access\service-tag.png "Скачать JSON файл")

В файле JSON, предполагая, что служба поиска находится в западной центральной части США, список IP-адресов для среды выполнения индексатора с несколькими клиентами приведен ниже.

```json
    {
      "name": "AzureCognitiveSearch.WestCentralUS",
      "id": "AzureCognitiveSearch.WestCentralUS",
      "properties": {
        "changeNumber": 1,
        "region": "westcentralus",
        "platform": "Azure",
        "systemService": "AzureCognitiveSearch",
        "addressPrefixes": [
          "52.150.139.0/26",
          "52.253.133.74/32"
        ]
      }
    }
```

Для IP-адресов/32 удалите "/32" (52.253.133.74/32-> 52.253.133.74), другие можно использовать буквально.

## <a name="add-the-ip-address-ranges-to-ip-firewall-rules"></a>Добавление диапазонов IP-адресов в правила брандмауэра IP

Самый простой способ добавить диапазоны IP-адресов в правило брандмауэра учетной записи хранения — с помощью портал Azure. Найдите учетную запись хранения на портале и перейдите на вкладку **брандмауэры и виртуальные сети** .

   ![Брандмауэр и виртуальные сети](media\search-indexer-howto-secure-access\storage-firewall.png "Брандмауэр и виртуальные сети")

Добавьте три IP-адреса, полученные ранее (1 — IP-адрес службы поиска, 2 для `AzureCognitiveSearch` тега службы) в диапазоне адресов, и нажмите кнопку **сохранить**.

   ![Правила IP-адресов брандмауэра](media\search-indexer-howto-secure-access\storage-firewall-ip.png "Правила IP-адресов брандмауэра")

Для обновления правил брандмауэра требуется 5-10 минут, после чего индексаторы смогут получить доступ к данным в учетной записи хранения.

## <a name="next-steps"></a>Дальнейшие действия

- [Настройка брандмауэров службы хранилища Azure](../storage/common/storage-network-security.md)
- [Настройка брандмауэра IP для Cosmos DB](../cosmos-db/how-to-configure-firewall.md)
- [Настройка брандмауэра IP для Azure SQL Server](../azure-sql/database/firewall-configure.md)