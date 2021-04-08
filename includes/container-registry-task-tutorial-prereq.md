---
author: dlepow
ms.service: container-registry
ms.topic: include
ms.date: 05/02/2019
ms.author: danlep
ms.openlocfilehash: 79e2ca71d6b8178be63c3429edba89cf3bb523e2
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96030001"
---
## <a name="prerequisites"></a>Предварительные требования

### <a name="get-sample-code"></a>Получение примера кода

В этом руководстве предполагается, что вы уже выполнили шаги в [предыдущем руководстве](../articles/container-registry/container-registry-tutorial-quick-task.md), создали вилку и клонировали пример репозитория. Если вы еще этого не сделали, выполните действия, описанные в разделе [Предварительные требования](../articles/container-registry/container-registry-tutorial-quick-task.md#prerequisites) в предыдущем руководстве, прежде чем перед продолжить.

### <a name="container-registry"></a>Реестр контейнеров

Для выполнения заданий данного руководства необходимо иметь реестр контейнеров Azure в подписке Azure. Если вам нужен реестр, см. сведения в [предыдущем руководстве](../articles/container-registry/container-registry-tutorial-quick-task.md) или в статье [Краткое руководство. Создание реестра контейнеров с использованием Azure CLI](../articles/container-registry/container-registry-get-started-azure-cli.md).

### <a name="create-a-github-personal-access-token"></a>Создание личного маркера доступа GitHub

Чтобы инициировать задачу при фиксации кода в репозитории Git, решению "Задачи ACR" требуется личный маркер доступа для доступа к репозиторию. Если у вас еще нет личного маркера доступа, выполните следующие действия, чтобы создать его на GitHub.

1. Перейдите на страницу создания PAT на GitHub по адресу https://github.com/settings/tokens/new.
1. Введите краткое **описание** маркера, например "ACR Task Demo".
1. Выберите области для ACR, чтобы получить доступ к репозиторию. Для доступа к общедоступному репозиторию, как в этом учебнике, в разделе **repo** включите **repo:status** и **public_repo**

   ![Снимок экрана со страницей создания личного маркера доступа GitHub][build-task-01-new-token]

   > [!NOTE]
   > Чтобы создать личный маркер доступа для доступа к репозиторию *private*, выберите область для полного контроля **репозитория**.

1. Нажмите кнопку **Generate token** (Создать маркер) (может появиться запрос на подтверждение пароля).
1. Скопируйте и сохраните созданный маркер в **защищенном расположении** (вы используете этот маркер при определении задачи в следующем разделе).

   ![Снимок экрана созданного личного маркера доступа на GitHub][build-task-02-generated-token]

<!-- Images -->
[build-task-01-new-token]: ./media/container-registry-task-tutorial-prereq/build-task-01-new-token.png
[build-task-02-generated-token]: ./media/container-registry-task-tutorial-prereq/build-task-02-generated-token.png
