---
title: Импорт спецификации OpenAPI с помощью портала Azure | Документация Майкрософт
description: Сведения о том, как импортировать спецификацию OpenAPI с помощью службы "Управление API", а также о проверке API на портале Azure и портале разработчика.
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 04/20/2020
ms.author: apimpm
ms.openlocfilehash: 05295efec3d7651c3a77dd5ad6e2c72b57a6c5a3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95994720"
---
# <a name="import-an-openapi-specification"></a>Импорт спецификации OpenAPI

В этой статье показано, как импортировать API серверной части "Спецификация OpenAPI", расположенный по адресу https://conferenceapi.azurewebsites.net?format=json. Этот API серверной части предоставляется корпорацией Майкрософт и размещен в Azure. Также здесь показано, как проверить API службы управления API.

Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Импорт API серверной части "Спецификация OpenAPI"
> * проверка API на портале Azure;
> * проверка API на портале разработчика.

## <a name="prerequisites"></a>Предварительные требования

Выполните инструкции из краткого руководства [Создание экземпляра службы управления API Azure](get-started-create-service-instance.md)

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="import-and-publish-a-back-end-api"></a><a name="create-api"> </a>Импорт и публикация API серверной части

1. На портале Azure перейдите к службе "Управление API" и в меню выберите **Интерфейсы API**.
2. Выберите **Спецификация OpenAPI** из списка **Добавление нового API**.

    ![Спецификация OpenAPI](./media/import-api-from-oas/oas-api.png)
3. Введите параметры API. Вы можете задать значения во время создания или настроить их позже, перейдя на вкладку **Параметры**. Эти параметры описаны в руководстве [Импорт и публикация первого интерфейса API](import-and-publish.md#import-and-publish-a-backend-api).
4. Нажмите кнопку **создания**.

> [!NOTE]
> См. подробнее об [ограничении импорта API](api-management-api-import-restrictions.md).

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-append-apis.md)]

[!INCLUDE [api-management-define-api-topics.md](../../includes/api-management-define-api-topics.md)]

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Преобразование и защита опубликованного API](transform-api.md)
