---
title: Управление QnA Maker приложением QnA Maker
description: QnA Maker позволяет нескольким пользователям совместно работать с базой знаний. QnA Maker предлагает возможность улучшить качество базы знаний с помощью активного обучения. Один из них может просматривать, принимать или отклонять и добавлять без удаления или изменения существующих вопросов.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 11/09/2020
ms.openlocfilehash: 49bfca118e53bbe3e4287b2ce25e5baffa717175
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102217335"
---
# <a name="manage-qna-maker-app"></a>Управление приложением QnA Maker

QnA Maker позволяет совместно работать с разными авторами и редакторами содержимого, предлагая возможность ограничить доступ совместной работы на основе роли участника совместной работы.
Узнайте больше об [основных понятиях проверки подлинности QnA Maker для совместной работы](../Concepts/role-based-access-control.md).

## <a name="add-azure-role-based-access-control-azure-rbac"></a>Добавление управления доступом на основе ролей в Azure (Azure RBAC)

QnA Maker позволяет нескольким пользователям совместно работать над всеми базами знаний в одном и том же QnA Maker ресурсе. Эта функция предоставляется с помощью [управления доступом на основе ролей Azure (Azure RBAC)](../../../role-based-access-control/role-assignments-portal.md).

## <a name="access-at-the-qna-maker-resource-level"></a>Доступ на уровне ресурсов QnA Maker

Вы не можете предоставить общий доступ к определенной базе знаний в службе QnA Maker. Если требуется более детализированное управление доступом, рассмотрите возможность распределения баз знаний по различным QnA Maker ресурсам, а затем добавьте роли для каждого ресурса.

## <a name="add-a-role-to-a-resource"></a>Добавление роли в ресурс

### <a name="add-a-user-account-to-the-qna-maker-resource"></a>Добавление учетной записи пользователя в ресурс QnA Maker

В следующих шагах используется роль совместной работы, но любые роли можно добавить с помощью этих шагов.

1. Войдите на портал [Azure](https://portal.azure.com/) и перейдите к ресурсу QnA Maker.

    ![Список ресурсов QnA Maker](../media/qnamaker-how-to-collaborate-knowledge-base/qnamaker-resource-list.png)

1. Перейдите на вкладку **Управление доступом (IAM)**.

    ![IAM в QnA Maker](../media/qnamaker-how-to-collaborate-knowledge-base/qnamaker-iam.png)

1. Выберите **Добавить**.

    ![Добавление IAM в QnA Maker](../media/qnamaker-how-to-collaborate-knowledge-base/qnamaker-iam-add.png)

1. Выберите роль из следующего списка:

    |Роль|
    |--|
    |Владелец|
    |Участник|
    |Модуль чтения Cognitive Services QnA Maker|
    |Редактор Cognitive Services QnA Maker|
    |Пользователь служб Cognitive Services|

    :::image type="content" source="../media/qnamaker-how-to-collaborate-knowledge-base/qnamaker-add-role-iam.png" alt-text="QnA Maker Добавление роли IAM.":::

1. Введите адрес электронной почты пользователя и нажмите кнопку " **сохранить**".

    ![Добавление адреса электронной почты для IAM в QnA Maker](../media/qnamaker-how-to-collaborate-knowledge-base/qnamaker-iam-add-email.png)

### <a name="view-qna-maker-knowledge-bases"></a>Просмотр QnA Maker баз знаний

Когда пользователь, которому вы предоставили доступ к QnA Maker службе с журналами на [портале QnA Maker](https://qnamaker.ai), он может просматривать все базы знаний в этой службе в зависимости от их роли.

При выборе базы знаний ее текущая роль на этом QnA Makerном ресурсе отображается рядом с именем базы знаний.

:::image type="content" source="../media/qnamaker-how-to-collaborate-knowledge-base/qnamaker-knowledge-base-role-name.png" alt-text="Снимок экрана базы знаний в режиме редактирования с именем роли в круглых скобках рядом с именем базы знаний в левом верхнем углу веб-страницы.":::

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Создание базы знаний](./manage-knowledge-bases.md)
