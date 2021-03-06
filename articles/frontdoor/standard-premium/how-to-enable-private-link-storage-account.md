---
title: Подключение передней дверцы Azure к источнику учетной записи хранения с помощью частной ссылки
titleSuffix: Azure Private Link
description: Узнайте, как подключить переднюю дверцу Azure к учетной записи хранения в частном порядке.
services: frontdoor
author: duongau
ms.service: frontdoor
ms.topic: how-to
ms.date: 03/04/2021
ms.author: tyao
ms.openlocfilehash: 885b4d132208ab6f8b470d147438e26a5fd4bab7
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102201674"
---
# <a name="connect-azure-front-door-premium-to-a-storage-account-origin-with-private-link"></a>Подключение передней дверцы Azure к источнику учетной записи хранения с помощью частной ссылки

В этой статье вы узнаете, как настроить номер SKU "Премиум" для передней дверцы Azure, чтобы подключиться к источнику учетной записи хранения в частном порядке с помощью службы частной связи Azure.

## <a name="sign-in-to-azure"></a>Вход в Azure

Войдите на [портал Azure](https://portal.azure.com).

## <a name="enable-private-link-to-a-storage-account"></a>Включение закрытой ссылки на учетную запись хранения
 
В этом разделе вы будете сопоставлять службу частной связи с частной конечной точкой, созданной в частной сети передней дверцы Azure. 

1. В профиле "Премиум" на передней дверце Azure в разделе " *Параметры*" выберите **исходные группы**.

1. Выберите исходную группу, содержащую источник учетной записи хранения, для которого требуется включить закрытую ссылку.

1. Выберите **+ Добавить источник** , чтобы добавить новый источник учетной записи хранения, или выберите ранее созданный источник учетной записи хранения из списка.

    :::image type="content" source="../media/how-to-enable-private-link-storage-account/private-endpoint-storage-account.png" alt-text="Снимок экрана: включение закрытой ссылки на учетную запись хранения.":::

1. Для **выбора ресурса Azure** выберите **в моем каталоге**. Выберите или введите следующие параметры, чтобы настроить сайт, к которому будет подключаться передняя дверь Azure Premium.

    | Параметр | Значение |
    | ------- | ----- |
    | Регион | Выберите область, которая является той же или ближайшей к источнику. |
    | Тип ресурса | Выберите **Microsoft. Storage/storageAccounts**. |
    | Ресурс | Затем выберите учетную запись хранения. |
    | Целевой вспомогательный ресурс | Можно выбрать *большой двоичный объект* или *веб-узел*. |
    | Сообщение запроса | Настройте сообщение или выберите значение по умолчанию. |

1. Затем нажмите кнопку **Добавить** , чтобы сохранить конфигурацию.

## <a name="approve-private-endpoint-connection-from-the-storage-account"></a>Утверждение подключения к частной конечной точке из учетной записи хранения

1. В последнем разделе перейдите к учетной записи хранения, для которой настроена частная ссылка. В разделе **Параметры** выберите **сеть** .

1. В области **сеть** выберите **подключения к частным конечным точкам**. 

    :::image type="content" source="../media/how-to-enable-private-link-storage-account/storage-account-configure-endpoint.png" alt-text="Снимок экрана параметров сети в веб-приложении.":::

1. Выберите *ожидающий* запрос закрытой конечной точки из Azure Front дверь Premium и нажмите кнопку **утвердить**.

    :::image type="content" source="../media/how-to-enable-private-link-storage-account/private-endpoint-pending-approval.png" alt-text="Снимок экрана запроса закрытой конечной точки ожидающего хранения.":::

1. После утверждения он должен выглядеть, как показано на снимке экрана ниже. Для полного установления соединения может потребоваться несколько минут. Теперь вы можете получить доступ к учетной записи хранения из передней дверцы Azure.

    :::image type="content" source="../media/how-to-enable-private-link-storage-account/private-endpoint-approved.png" alt-text="Снимок экрана запроса утвержденной конечной точки хранилища.":::

## <a name="next-steps"></a>Дальнейшие действия

Сведения о [службе Private Link с учетной записью хранения](../../storage/common/storage-private-endpoints.md).
