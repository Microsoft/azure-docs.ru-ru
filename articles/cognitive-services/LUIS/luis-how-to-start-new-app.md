---
title: Создание нового приложения — LUIS
titleSuffix: Azure Cognitive Services
description: Создание и настройка приложений на веб-странице службы "Распознавание речи" (LUIS).
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 05/18/2020
ms.openlocfilehash: 2dd06a7b4c8e6296cda747d17fd3d5be5db0af6b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95018894"
---
# <a name="create-a-new-luis-app-in-the-luis-portal"></a>Создание приложения LUIS на портале LUIS
Создать приложение LUIS можно двумя способами: на портале LUIS или с помощью [API-интерфейсов](developer-reference-resource.md) разработки LUIS.

## <a name="using-the-luis-portal"></a>Использование портала LUIS

Создать приложение на портале можно несколькими способами.

* начать с пустого приложения и создать намерения, фрагменты речи и сущности;
* начать с пустого приложения и добавить [готовую предметную область](./howto-add-prebuilt-models.md);
* Импортируйте приложение LUIS из `.lu` файла или `.json` , который уже содержит объекты, фразы продолжительностью и сущности.

## <a name="using-the-authoring-apis"></a>Использование API-интерфейсов разработки
С помощью API-интерфейсов разработки приложение можно создать несколькими способами:

* [Добавление приложения](https://westeurope.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview/operations/5890b47c39e2bb052c5b9c2f) — запуск с пустым приложением и создание целей, фразы продолжительностью и сущностей.
* [Добавьте](https://westeurope.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview/operations/59104e515aca2f0b48c76be5) готовое приложение — Начните с предварительно созданного домена, в том числе целей, фразы продолжительностью и сущностей.


<a name="export-app"></a>
<a name="import-new-app"></a>
<a name="delete-app"></a>


[!INCLUDE [Sign in to LUIS](./includes/sign-in-process.md)]

## <a name="create-new-app-in-luis"></a>Создание приложения в LUIS

1. На странице **Мои приложения** выберите **подписку** и создайте  **ресурс** , а затем — **создать**. 

> [!div class="mx-imgBorder"]
> ![Список приложений LUIS](./media/create-app-in-portal.png)

1. В диалоговом окне введите имя приложения, например `Pizza Tutorial` .

    ![Диалоговое окно создания приложения](./media/create-pizza-tutorial-app-in-portal.png)

1. Выберите язык и региональные параметры приложения, а затем нажмите кнопку **Готово**. В этом случае ресурс Description и PREDICTION является необязательным. Вы можете задать в любое время в разделе **Управление** на портале.

    > [!NOTE]
    > Язык и региональные параметры нельзя изменить после создания приложения.

    После создания приложения на портале LUIS появится список **целей** с `None` намерением, которое уже было создано ранее. Теперь у вас есть пустое приложение.

    > [!div class="mx-imgBorder"]
    > ![Список целей с намерением «нет», созданный без примеров фразы продолжительностью.](media/pizza-tutorial-new-app-empty-intent-list.png)

## <a name="other-actions-available-on-my-apps-page"></a>Другие действия, доступные на странице "Мои приложения"

Контекстная панель инструментов предоставляет другие действия:

* Переименование приложения
* Импорт из файла с помощью `.lu` или `.json`
* Экспорт приложения как `.lu` (для [лудовн](https://github.com/microsoft/botbuilder-tools/tree/master/packages/Ludown)), `.json` или `.zip` (для [контейнера Luis](luis-container-howto.md))
* Импорт журналов конечной точки контейнера для просмотра конечной точки фразы продолжительностью
* Экспорт журналов конечных точек как `.csv` для автономного анализа
* Удаление приложения

## <a name="next-steps"></a>Дальнейшие действия

Если проект приложения включает обнаружение намерения, [Создайте новые](luis-how-to-add-intents.md)цели и добавьте пример фразы продолжительностью. Если проект приложения предназначен только для извлечения данных, добавьте пример фразы продолжительностью к намерению None, затем [Создайте сущности](./luis-how-to-add-entities.md)и пометка примера фразы продолжительностью с этими сущностями.