---
author: dlepow
ms.service: container-instances
ms.topic: include
ms.date: 07/22/2020
ms.author: danlep
ms.openlocfilehash: 6878180ffedfaa53f25d2bdc6db72dcd7dd8b38b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "87384839"
---
| Ресурс | Ограничение |
| --- | :--- |
| Группы контейнеров SKU "Стандартный" для каждого региона на подписку | 100<sup>1</sup> |
| Группы контейнеров SKU "Выделенный" для каждого региона на подписку | 0<sup>1</sup> |
| Число контейнеров в группе контейнеров | 60 |
| Число томов в группе контейнеров | 20 |
| Ядра (ЦП) SKU "Стандартный" для каждого региона на подписку | 10<sup>1, 2</sup> | 
| Ядра (ЦП) SKU "Стандартный" для GPU K80 для каждого региона на подписку | 18<sup>1, 2</sup> |
| Ядра (ЦП) SKU "Стандартный" для GPU P100 или V100 для каждого региона на подписку | 0<sup>1, 2</sup> |
| Портов на каждый IP-адрес | 5 |
| Размер журнала экземпляра контейнера запущенный экземпляр | 4 МБ |
| Размер журнала экземпляра контейнера остановленный экземпляр | 16 КБ или 1000 строк |
| Созданных контейнеров в час |300<sup>1</sup> |
| Созданных контейнеров в 5 мин | 100<sup>1</sup> |
| Удаленных контейнеров в час | 300<sup>1</sup> |
| Удаленных контейнеров в 5 мин | 100<sup>1</sup> |


<sup>1</sup> Чтобы увеличить лимит, отправьте [запрос в Службу поддержки Azure][azure-support]. Для бесплатных подписок, в том числе для [бесплатной учетной записи Azure](https://azure.microsoft.com/offers/ms-azr-0044p/) и [Azure для учащихся](https://azure.microsoft.com/offers/ms-azr-0170p/), нельзя увеличить лимит или квоту. Если у вас бесплатная подписка, ее можно [обновить](../articles/cost-management-billing/manage/upgrade-azure-subscription.md) до подписки с оплатой по мере использования.<br />
<sup>2</sup> Лимит по умолчанию для подписки [с оплатой по мере использования](https://azure.microsoft.com/offers/ms-azr-0003p/). Лимит зависит от типа категории.<br/>

<!-- LINKS - External -->
[azure-support]: https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest
