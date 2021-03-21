---
title: Перемещение подписок Azure для решений VMware и CSP
description: Узнайте, как переместить частное облако из одной подписки в другую. Перемещение может выполняться по различным причинам, таким как выставление счетов.
ms.topic: how-to
ms.date: 03/15/2021
ms.openlocfilehash: 608f46dbd84d6bb899a3e7fcd1f8a63b3a5e85fb
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103555613"
---
# <a name="move-ea-and-csp-azure-vmware-solution-subscriptions"></a>Перемещение подписок Azure для решений VMware и CSP

В этой статье вы узнаете, как переместить частное облако из одной подписки в другую. Перемещение может выполняться по различным причинам, таким как выставление счетов. 

>[!IMPORTANT]
>У вас должно быть по крайней мере права участника на исходную и целевую подписки. Шлюз виртуальной сети и виртуальной сети нельзя переместить из одной подписки в другую. Кроме того, перемещение подписок не влияет на управление и рабочие нагрузки, такие как виртуальные машины vCenter, НСКС и рабочей нагрузки.

1. Войдите в портал Azure и выберите частное облако, которое нужно переместить.

1. Выберите ссылку **Подписка (изменить)** .

   :::image type="content" source="media/private-cloud-overview-subscription-id.png" alt-text="Снимок экрана, показывающий сведения о частном облаке.":::

1. Укажите сведения о подписке для **целевого объекта** и нажмите кнопку **Далее**.

   :::image type="content" source="media/move-resources-subscription-target.png" alt-text="Снимок экрана целевого ресурса." lightbox="media/move-resources-subscription-target.png":::

1. Подтвердите проверку ресурсов, выбранных для перемещения, и нажмите кнопку **Далее**. 

   :::image type="content" source="media/confirm-move-resources-subscription-target.png" alt-text="Снимок экрана, показывающий перемещаемый ресурс." lightbox="media/confirm-move-resources-subscription-target.png":::

1. Установите флажок, указывающий, что связанные средства и сценарии не будут работать, пока вы не обновите их, чтобы они использовали новые идентификаторы ресурсов. Затем выберите **переместить**.

   :::image type="content" source="media/review-move-resources-subscription-target.png" alt-text="Снимок экрана, показывающий сводку перемещаемого ресурса. " lightbox="media/review-move-resources-subscription-target.png":::

   После завершения перемещения ресурса появится уведомление. Новая подписка появится в обзоре частного облака.

   :::image type="content" source="media/moved-subscription-target.png" alt-text="Снимок экрана, показывающий новую подписку." lightbox="media/moved-subscription-target.png":::

