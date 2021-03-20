---
title: Квоты службы и доступность по регионам
description: Квоты, ограничения и доступность по регионам службы "Экземпляры контейнеров Azure".
ms.topic: article
ms.date: 07/22/2020
ms.openlocfilehash: eaaa8e0b2d72aaea546a1bc351da40932c1deb14
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87384840"
---
# <a name="quotas-and-limits-for-azure-container-instances"></a>Квоты и ограничения службы "Экземпляры контейнеров Azure"

Все службы Azure включают несколько стандартных ограничений и квот для ресурсов и функций. В этой статье подробно описываются квоты и ограничения по умолчанию для Экземпляров контейнеров Azure.

Доступность ресурсов вычислений, памяти и хранилища для экземпляров контейнеров Azure зависит от региона и операционной системы. Дополнительные сведения см. в статье [доступность ресурсов для службы "экземпляры контейнеров Azure](container-instances-region-availability.md)".

Используйте API [List Usage](/rest/api/container-instances/location/listusage) для просмотра текущего использования квоты в регионе для подписки.

## <a name="service-quotas-and-limits"></a>Квоты и ограничения службы

[!INCLUDE [container-instances-limits](../../includes/container-instances-limits.md)]

## <a name="next-steps"></a>Дальнейшие действия

Вы можете повысить лимит нескольких ограничений и квот. Чтобы запросить увеличение квоты на один или несколько ресурсов, которые поддерживают такую возможность, отправьте [запрос в службу поддержки Azure][azure-support] (указав "Квота" в качестве **типа проблемы**).

<!-- LINKS - External -->
[azure-support]: https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest
