---
title: Изменение API с помощью портала Azure | Документация Майкрософт
description: Сведения о том, как изменить API с помощью службы "Управление API" (APIM). Добавление, удаление или переименование операций в экземпляре APIM или изменение Swagger API.
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 11/08/2017
ms.author: apimpm
ms.openlocfilehash: 1c4e64251390936e8a63ee904ec69f173cac6114
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "93146711"
---
# <a name="edit-an-api"></a>Изменение API

Из этого руководства вы узнаете, как изменить API с помощью службы управления API. 

+ Это можно сделать, используя добавление, удаление и переименование в экземпляре службы управления API. 
+ Вы можете изменить Swagger API.

## <a name="prerequisites"></a>Предварительные требования

+ [Создание экземпляра службы управления API Azure](get-started-create-service-instance.md)
+ [Импорт и публикация первого API](import-and-publish.md)

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="edit-an-api-in-apim"></a>Изменение API в службе управления API

![Снимок экрана, на котором показан процесс изменения API в APIM.](./media/edit-api/edit-api001.png)

1. Щелкните вкладку **Интерфейсы API**.
2. Выберите один из API, которые вы импортировали ранее.
3. Выберите вкладку **Конструктор**.
4. Выберите операцию, которую нужно изменить.
5. Чтобы переименовать операцию, щелкните значок **карандаша** в окне **Интерфейсный сервер**.

## <a name="update-the-swagger"></a>Обновление Swagger

Можно обновить API внутреннего сервера на портале Azure, выполнив следующие действия:

1. Выберите **Все операции**.
2. Щелкните значок карандаша в окне **Интерфейсный сервер**.

    ![Снимок экрана, на котором изображен значок карандаша в окне "Интерфейсный сервер".](./media/edit-api/edit-api002.png)

    Отобразится спецификация Swagger API.

    ![Изменение API](./media/edit-api/edit-api003.png)

3. Обновите Swagger.
4. Нажмите кнопку **Save**(Сохранить).

[!INCLUDE [api-management-define-api-topics.md](../../includes/api-management-define-api-topics.md)]

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Примеры политик службы управления API](./policy-reference.md)
> [Преобразование и защита опубликованного API](transform-api.md)