---
ms.openlocfilehash: 2e92d150851c74a84f785d1f5f0ebe2e5870a54e
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "97936770"
---


| Компонент |[План потребления](../articles/azure-functions/consumption-plan.md)|[План категории "Премиум"](../articles/azure-functions/functions-premium-plan.md)|[План ценовой категории "Выделенный"](../articles/azure-functions/dedicated-plan.md)|[ASE](../articles/app-service/environment/intro.md)| [Kubernetes](../articles/azure-functions/functions-kubernetes-keda.md) |
|----------------|-----------|----------------|---------|-----------------------| ---|
|[Ограничения на входящие IP-адреса и частный доступ к сайту](../articles/azure-functions/functions-networking-options.md#inbound-access-restrictions)|✅Да|✅Да|✅Да|✅Да|✅Да|
|[Интеграция с виртуальной сетью](../articles/azure-functions/functions-networking-options.md#virtual-network-integration)|❌Нет|✅Да (Региональный)|✅Да (Региональный и шлюз)|✅Да| ✅Да|
|[Триггеры виртуальной сети (без HTTP)](../articles/azure-functions/functions-networking-options.md#virtual-network-triggers-non-http)|❌Нет| ✅Да |✅Да|✅Да|✅Да|
|[Гибридные подключения](../articles/azure-functions/functions-networking-options.md#hybrid-connections) (только для Windows)|❌Нет|✅Да|✅Да|✅Да|✅Да|
|[Ограничения исходящих IP-адресов](../articles/azure-functions/functions-networking-options.md#outbound-ip-restrictions)|❌Нет| ✅Да|✅Да|✅Да|✅Да|