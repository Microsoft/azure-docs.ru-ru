---
title: Подключение передней дверцы Azure к источнику внутренней подсистемы балансировки нагрузки с помощью частной ссылки
titleSuffix: Azure Private Link
description: Узнайте, как подключить переднюю дверь Azure к внутренней подсистеме балансировки нагрузки.
services: frontdoor
author: duongau
ms.service: frontdoor
ms.topic: how-to
ms.date: 03/16/2021
ms.author: tyao
ms.openlocfilehash: 6876692bcf752570ecdc5d42b65da81992ad3743
ms.sourcegitcommit: ba3a4d58a17021a922f763095ddc3cf768b11336
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104803823"
---
# <a name="connect-azure-front-door-premium-to-an-internal-load-balancer-origin-with-private-link"></a>Подключение передней дверцы Azure к источнику внутренней подсистемы балансировки нагрузки с помощью частной ссылки

В этой статье вы узнаете, как настроить номер SKU передней дверцы Azure для подключения к источнику внутренней подсистемы балансировки нагрузки с помощью службы частной связи Azure.

## <a name="prerequisites"></a>Предварительные требования

Создайте [службу частной связи](../../private-link/create-private-link-service-portal.md).

## <a name="sign-in-to-azure"></a>Вход в Azure

Войдите на [портал Azure](https://portal.azure.com).

## <a name="enable-private-link-to-an-internal-load-balancer"></a>Включение закрытой ссылки на внутреннюю подсистему балансировки нагрузки
 
В этом разделе вы будете сопоставлять службу частной связи с частной конечной точкой, созданной в частной сети передней дверцы Azure. 

1. В профиле "Премиум" на передней дверце Azure в разделе " *Параметры*" выберите **исходные группы**.

1. Выберите исходную группу, для которой необходимо включить закрытую ссылку для внутренней подсистемы балансировки нагрузки.

1. Выберите **+ Добавить источник** , чтобы добавить источник внутренней подсистемы балансировки нагрузки.

    :::image type="content" source="../media/how-to-enable-private-link-internal-load-balancer/private-endpoint-internal-load-balancer.png" alt-text="Снимок экрана: включение закрытой ссылки на внутреннюю подсистему балансировки нагрузки.":::

1. Для **выбора ресурса Azure** выберите **в моем каталоге**. Выберите или введите следующие параметры, чтобы настроить сайт, к которому будет подключаться передняя дверь Azure Premium.

    | Параметр | Значение |
    | ------- | ----- |
    | Регион | Выберите область, которая является той же или ближайшей к источнику. |
    | Тип ресурса | Выберите **Microsoft.Network/privateLinkServices**. |
    | Ресурс | Выберите свою закрытую ссылку, связанную с внутренней подсистемой балансировки нагрузки. |
    | Целевой вспомогательный ресурс | Не указывайте. |
    | Сообщение запроса | Настройте сообщение или выберите значение по умолчанию. |

1. Затем выберите **Добавить** , а затем **Обновить** , чтобы сохранить конфигурацию.

## <a name="approve-private-endpoint-connection-from-the-storage-account"></a>Утверждение подключения к частной конечной точке из учетной записи хранения

1. Перейдите в раздел "частный центр ссылок" и выберите " **службы частной связи**". Затем выберите имя вашей частной ссылки.

    :::image type="content" source="../media/how-to-enable-private-link-internal-load-balancer/list.png" alt-text="Снимок экрана со списком частных ссылок.":::

1. В разделе *Параметры* выберите **подключения к частной конечной точке** .

    :::image type="content" source="../media/how-to-enable-private-link-internal-load-balancer/overview.png" alt-text="Снимок экрана со страницей обзора частной ссылки.":::

1. Выберите *ожидающий* запрос закрытой конечной точки из Azure Front дверь Premium и нажмите кнопку **утвердить**.

    :::image type="content" source="../media/how-to-enable-private-link-internal-load-balancer/private-endpoint-pending-approval.png" alt-text="Снимок экрана с ожидающим утверждением для частной ссылки.":::

1. После утверждения он должен выглядеть, как показано на снимке экрана ниже. Для полного установления соединения может потребоваться несколько минут. Теперь вы можете получить доступ к внутренней подсистеме балансировки нагрузки из передней дверцы Azure.

    :::image type="content" source="../media/how-to-enable-private-link-storage-account/private-endpoint-approved.png" alt-text="Снимок экрана запроса утвержденной частной ссылки.":::

## <a name="next-steps"></a>Следующие шаги

Сведения о [службе Private Link](../../private-link/private-link-service-overview.md).
