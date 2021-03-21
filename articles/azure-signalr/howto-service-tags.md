---
title: Использование тегов служб
titleSuffix: Azure SignalR Service
description: Использование тегов службы для разрешения исходящего трафика в службу Azure SignalR
services: signalr
author: ArchangelSDY
ms.service: signalr
ms.topic: article
ms.date: 05/06/2020
ms.author: dayshen
ms.openlocfilehash: aaa97c2cb062f30b1260ec7f80f85a3caccf932f
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92152311"
---
# <a name="use-service-tags-for-azure-signalr-service"></a>Использование тегов службы для службы SignalR Azure

При настройке [группы безопасности сети](../virtual-network/network-security-groups-overview.md#network-security-groups)можно использовать [теги службы](../virtual-network/network-security-groups-overview.md#service-tags) для службы SignalR Azure. Он позволяет определить исходящее правило сетевой безопасности для конечных точек службы SignalR Azure, не требуя жестко определять IP-адреса.

Служба Azure SignalR управляет этими тегами службы. Нельзя создать собственный тег службы или изменить существующий. Корпорация Майкрософт управляет такими префиксами адресов, которые соответствуют тегу службы, и автоматически обновляет тег службы при изменении адресов.

## <a name="use-service-tag-on-portal"></a>Использовать тег службы на портале

Вы можете разрешить исходящий трафик в службу Azure SignalR, добавив новое правило сетевой безопасности для исходящей сети:

1. Перейдите в группу безопасности сети.

1. Щелкните меню параметров, именуемое **правила безопасности для исходящего трафика**.

1. Нажмите кнопку **+ Добавить** в верхней части страницы.

1. В разделе **назначение** выберите **тег службы** .

1. Выберите **азуресигналр** в разделе **тег целевой службы**.

1. Введите **443** в **диапазоны портов назначения**.

    ![Создание правила безопасности для исходящего трафика](media/howto-service-tags/portal-add-outbound-security-rule.png)

1. Настройте другие поля в соответствии с вашими потребностями.

1. Нажмите кнопку **Добавить**.


## <a name="next-steps"></a>Дальнейшие действия

- [Группы безопасности сети: Теги служб](../virtual-network/network-security-groups-overview.md#security-rules)