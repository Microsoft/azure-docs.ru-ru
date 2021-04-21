---
title: Учебник. Использование редакций в Управлении API для безопасного внесения некритических изменений в API
titleSuffix: Azure API Management
description: Следуйте инструкциям этого руководства, чтобы научиться вносить некритические изменения с помощью редакций в службе управления API.
services: api-management
documentationcenter: ''
author: vladvino
ms.service: api-management
ms.custom: mvc, devx-track-azurecli
ms.topic: tutorial
ms.date: 02/09/2021
ms.author: apimpm
ms.openlocfilehash: 1d99d6f876e4896bb4321afb8dc4d8e7c3a404e7
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/14/2021
ms.locfileid: "107483567"
---
# <a name="tutorial-use-revisions-to-make-non-breaking-api-changes-safely"></a>Руководство по безопасному внесению некритических изменений в API с помощью редакций
Когда готовый к работе API начинают использовать разработчики, следует осторожно вносить изменения в этот API, чтобы не нарушить работу вызывающих его сторон. Полезно также сообщать разработчикам о внесенных изменениях. 

Используйте *редакции* в Управлении API Azure, чтобы вносить некритические изменения в API для безопасного моделирования и тестирования изменения. Когда все будет готово, вы сможете назначить редакцию текущей и заменить текущий API. 

Дополнительные сведения см. в статьях [Версии и редакции](https://azure.microsoft.com/blog/versions-revisions/) и [Управление версиями API с помощью службы управления API Azure](https://azure.microsoft.com/blog/api-versioning-with-azure-api-management/).

В этом руководстве вы узнаете, как:

> [!div class="checklist"]
> * Добавление новой редакции
> * Внесение некритических изменений в редакцию
> * Назначение редакции текущей и добавление записи журнала изменений
> * Обзор портала разработчика для просмотра изменений и журнала изменений

:::image type="content" source="media/api-management-getstarted-revise-api/azure-portal.png" alt-text="Редакции API на портале Azure":::

## <a name="prerequisites"></a>Предварительные требования

+ Ознакомьтесь с [терминологией службы управления API в Azure](api-management-terminology.md).
+ Выполните задачи в кратком руководстве по [созданию экземпляра службы управления API Azure](get-started-create-service-instance.md).
+ Также выполните инструкции из руководства [Импорт и публикация первого API](import-and-publish.md).

## <a name="add-a-new-revision"></a>Добавление новой редакции

1. Войдите на [портал Azure](https://portal.azure.com) и перейдите к своему экземпляру службы Управления API.
1. Выберите **Интерфейсы API**.
2. Выберите **Demo Conference API** в списке API (или другой API, в который нужно внести исправления).
3. Щелкните вкладку **Редакции**.
4. Выберите **+ Добавить редакцию**.

   :::image type="content" source="media/api-management-getstarted-revise-api/07-add-revisions-01-add-new-revision.png" alt-text="Добавление версии API":::

    > [!TIP]
    > Кроме того, команду **Добавить редакцию** можно выбрать в контекстном меню ( **…** ) API.

5. Введите описание вашей новой редакции, чтобы напомнить, для чего она будет использоваться.
6. Нажмите кнопку **Создать**.
7. Новая редакция создана.

    > [!NOTE]
    > Исходный API остается в редакции **Revision 1**. Именно эту редакцию по-прежнему будут вызывать пользователи, пока вы не выберете текущей другую редакцию.

## <a name="make-non-breaking-changes-to-your-revision"></a>Внесение некритических изменений в редакцию

1. В списке API выберите **Demo Conference API**.
1. Выберите вкладку **Конструктор** в верхней части экрана.
1. Обратите внимание, что **селектор исправлений** (непосредственно над вкладкой "Конструктор") показывает текущую версию **Revision 2**.

    > [!TIP]
    > Используйте селектор редакции для переключения между редакциям, с которыми вы хотите работать.
1. Щелкните **+ Add Operation** (+ Добавить операцию).
1. Укажите, что создается операция **POST**, а затем введите имя, отображаемое имя и URL-адрес операции — **test**.
1. **Сохраните** новую операцию.

   :::image type="content" source="media/api-management-getstarted-revise-api/07-add-revisions-02-make-changes.png" alt-text="Изменение версии":::
1. Теперь вы внесли изменение в редакцию **Revision 2**. Используйте **селектор редакции** в верхней части страницы, чтобы вернуться к редакции **Revision 1**.
1. Обратите внимание, что новая операция не отображается в редакции **Revision 1**. 

## <a name="make-your-revision-current-and-add-a-change-log-entry"></a>Назначение редакции текущей и добавление записи журнала изменений

### <a name="portal"></a>[Портал](#tab/azure-portal)

1. Выберите вкладку **Исправления** в меню в верхней части страницы.
1. Откройте контекстное меню (**…**) для редакции **Revision 2**.
1. Выберите **Сделать текущей**.
1. Установите флажок **Post to Public Change log for this API** (Записать в общедоступный журнал изменений для этого API), если нужно опубликовать примечания об этом изменении. Введите описание внесенного изменения, которое будет отображаться для разработчиков, например: **Тестирование исправлений. Добавлена новая операция test.**
1. Теперь редакция **Revision 2** является текущей.

    :::image type="content" source="media/api-management-getstarted-revise-api/revisions-menu.png" alt-text="Меню редакции в соответствующем окне":::

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы начать работу с Azure CLI, выполните следующие действия:

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

Используйте указанную ниже процедуру для создания и обновления выпуска.

1. Чтобы просмотреть идентификаторы API, выполните команду [az apim api list](/cli/azure/apim/api#az_apim_api_list):

   ```azurecli
   az apim api list --resource-group apim-hello-word-resource-group \
       --service-name apim-hello-world --output table
   ```

   Идентификатор API, используемый в следующей команде, имеет значение `Name`. Версия API находится в столбце `ApiRevision`.

1. Чтобы создать выпуск с заметкой о выпуске, выполните команду [az apim api release create](/cli/azure/apim/api/release#az_apim_api_release_create):

   ```azurecli
   az apim api release create --resource-group apim-hello-word-resource-group \
       --api-id demo-conference-api --api-revision 2 --service-name apim-hello-world \
       --notes 'Testing revisions. Added new "test" operation.'
   ```

   Выпущенная версия станет текущей версией.

1. Чтобы просмотреть выпуски, выполните команду [az apim api release list](/cli/azure/apim/api/release#az_apim_api_release_list):

   ```azurecli
   az apim api release list --resource-group apim-hello-word-resource-group \
       --api-id echo-api --service-name apim-hello-world --output table
   ```

   Заметки, которые вы укажете, отобразятся в журнале изменений. Вы увидите их в выходных данных предыдущей команды.

1. При создании выпуска параметр `--notes` является необязательным. Вы можете добавить или изменить заметки позже с помощью команды [az apim api release update](/cli/azure/apim/api/release#az_apim_api_release_update):

   ```azurecli
   az apim api release update --resource-group apim-hello-word-resource-group \
       --api-id demo-conference-api --release-id 00000000000000000000000000000000 \
       --service-name apim-hello-world --notes "Revised notes."
   ```

   Используйте значение в столбце `Name` для идентификатора выпуска.

Вы можете удалить любой выпуск с помощью команды [az apim api release delete](/cli/azure/apim/api/release#az_apim_api_release_delete):

```azurecli
az apim api release delete --resource-group apim-hello-word-resource-group \
    --api-id demo-conference-api --release-id 00000000000000000000000000000000 
    --service-name apim-hello-world
```

---

## <a name="browse-the-developer-portal-to-see-changes-and-change-log"></a>Обзор портала разработчика для просмотра изменений и журнала изменений

Если вы уже использовали [портал разработчика](api-management-howto-developer-portal-customize.md), вы можете просмотреть изменения API и журнал изменений.

1. На портале Azure выберите **API**.
1. Выберите **Портал разработчика** в верхнем меню.
1. На портале разработчика выберите **API**, а затем — **Demo Conference API**.
1. Обратите внимание на то, что теперь доступна новая операция **test**.
1. Щелкните **Журнал изменений** рядом с именем API.
1. Обратите внимание на запись журнала изменений в этом списке.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как выполнять следующие задачи:

> [!div class="checklist"]
> * Добавление новой редакции
> * Внесение некритических изменений в редакцию
> * Назначение редакции текущей и добавление записи журнала изменений
> * Обзор портала разработчика для просмотра изменений и журнала изменений

Перейдите к следующему руководству:

> [!div class="nextstepaction"]
> [Публикация нескольких версий API](api-management-get-started-publish-versions.md)
