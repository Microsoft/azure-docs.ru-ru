---
title: включить файл
description: включить файл
services: container-registry
author: dlepow
ms.service: container-registry
ms.topic: include
ms.date: 06/18/2020
ms.author: danlep
ms.custom: include file
ms.openlocfilehash: b4701260a7d8da030f9f3019060aaa83e7a3a483
ms.sourcegitcommit: 2c1b93301174fccea00798df08e08872f53f669c
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104803468"
---
| Ресурс | Basic | Standard | Premium |
|---|---|---|---|
| Включенное хранилище<sup>1</sup> (ГиБ) | 10 | 100 | 500 |
| Ограничение хранилища (ТиБ) | 20| 20 | 20 |
| Максимальный размер слоя образа (ГиБ) | 200 | 200 | 200 |
| Операции чтения за минуту<sup>2, 3</sup> | 1000 | 3000 | 10 000 |
| Операции записи за минуту<sup>2, 4</sup> | 100 | 500 | 2 000 |
| Пропускная способность скачивания<sup>2</sup> (Мбит/с) | 30 | 60 | 100 |
| Пропускная способность отправки <sup>2</sup> (Мбит/с) | 10 | 20 | 50 |
| Веб-перехватчики | 2 | 10 | 500 |
| Георепликация | Недоступно | Недоступно | [Поддерживаются][geo-replication] |
| Зоны доступности | Недоступно | Недоступно | [Предварительный просмотр][zones] |
| Доверие к содержимому | Недоступно | Недоступно | [Поддерживаются][content-trust] |
| Приватный канал с частными конечными точками | Недоступно | Недоступно | [Поддерживаются][plink] |
| &bull; Частные конечные точки | Недоступно | Недоступно | 10 |
| Правила сети общедоступных IP-адресов | Недоступно | Недоступно | 100 |
| Получение доступа к виртуальной сети конечной точки службы | Недоступно | Недоступно | [Предварительный просмотр][vnet] |
| Ключи, управляемые клиентом | Недоступно | Недоступно | [Поддерживаются][cmk] |
| Разрешения уровня репозитория | Недоступно | Недоступно | [Предварительный просмотр][token]|
| &bull; Токены | Недоступно | Недоступно | 20 000 |
| &bull; Карты области | Недоступно | Недоступно | 20 000 |
| &bull; Репозитории для карт областей | Недоступно | Недоступно | 500 |


<sup>1</sup> Хранилище, плата за которое включена в ежедневную плату для каждого уровня. Может использоваться дополнительное хранилище, за которое взимается ежедневная плата за ГиБ до достижения ограничения хранилища для реестра. Дополнительные сведения см. в статье [Цены на Реестр контейнеров Azure][pricing]. Если вам требуется хранилище, объем которого превышает ограничения хранилища реестра, обратитесь в Службу поддержки Azure.

<sup>2</sup>*Операции чтения*, *операции записи* и *пропускная способность* — это минимальные оценки. Реестр контейнеров Azure стремится обеспечить высокую производительность по мере использования.

<sup>3</sup> [docker pull](https://docs.docker.com/registry/spec/api/#pulling-an-image) преобразовывает несколько операций чтения в зависимости от количества слоев образа, а также от извлеченного манифеста.

<sup>4</sup> [docker push](https://docs.docker.com/registry/spec/api/#pushing-an-image) преобразовывает несколько операций записи, исходя из количества слоев, которые нужно отправить. Команда `docker push` включает *операции чтения* для получения манифеста для имеющегося образа.

<!-- LINKS - External -->
[pricing]: https://azure.microsoft.com/pricing/details/container-registry/

<!-- LINKS - Internal -->
[geo-replication]: ../articles/container-registry/container-registry-geo-replication.md
[content-trust]: ../articles/container-registry/container-registry-content-trust.md
[vnet]: ../articles/container-registry/container-registry-vnet.md
[plink]: ../articles/container-registry/container-registry-private-link.md
[cmk]: ../articles/container-registry/container-registry-customer-managed-keys.md
[token]: ../articles/container-registry/container-registry-repository-scoped-permissions.md
[zones]: ../articles/container-registry/zone-redundancy.md
