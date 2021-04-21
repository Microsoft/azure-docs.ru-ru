---
title: Учебник — имитация ответов API в управлении API — портал Azure | Документация Майкрософт
description: В этом руководстве вы используете службу управления API для задания политик в отношении API, чтобы она возвращала фиктивный ответ, если серверная часть недоступна для отправки реальных ответов.
author: vladvino
ms.service: api-management
ms.custom: mvc, devx-track-azurecli
ms.topic: tutorial
ms.date: 02/09/2021
ms.author: apimpm
ms.openlocfilehash: a7617a36ed800f1765ed7723568a4b612fcb6518
ms.sourcegitcommit: 272351402a140422205ff50b59f80d3c6758f6f6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/17/2021
ms.locfileid: "107587601"
---
# <a name="tutorial-mock-api-responses"></a>Руководство по Макетирование ответов API

Серверные API можно импортировать в API службы управления API или создать их и управлять ими вручную. В этом руководстве показано, как использовать APIM для создания пустого API и управления им вручную, а затем установить политику для API, чтобы она возвращала фиктивный ответ. Этот метод позволяет разработчикам продолжить реализацию и тестирование экземпляра службы управления API, даже если серверная часть не может отправлять реальные ответы. 

Возможность имитации ответов может быть полезна в ряде сценариев:

+ Когда сначала создается интерфейсная часть API, а затем — серверная реализация. Или когда серверная часть разрабатывается параллельно.
+ Когда серверная часть временно не работает или ее невозможно масштабировать.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * создание тестового API; 
> * добавление операции в тестовый API;
> * включение макетирования ответа;
> * тестирование макетирования API.


:::image type="content" source="media/mock-api-responses/mock-api-responses01.png" alt-text="Имитированный ответ API":::

## <a name="prerequisites"></a>Предварительные требования

+ Ознакомьтесь с [терминологией службы управления API в Azure](api-management-terminology.md).
+ Ознакомьтесь с [концепцией использования политик в службе управления API в Azure](api-management-howto-policies.md).
+ Выполните инструкции из краткого руководства [Создание экземпляра службы управления API Azure](get-started-create-service-instance.md).

## <a name="create-a-test-api"></a>создание тестового API; 

В этом разделе показано, как создать пустой API без серверной части. 


1. Войдите на портал Azure и перейдите к своему экземпляру службы управления API.
1. Выберите **API** >  **+ Добавить API** > **Пустой API**.
1. В окне **Создать пустой API** выберите **Полный**.
1. Введите *Тестовый API* в качестве **отображаемого имени**.
1. Выберите **Без ограничений** в поле **Продукты**.
1. В разделе **Шлюзы** выберите **Управляемые**.
1. Нажмите кнопку **создания**.

    :::image type="content" source="media/mock-api-responses/03-mock-api-responses-01-create-test-api.png" alt-text="Создание пустого API":::

## <a name="add-an-operation-to-the-test-api"></a>добавление операции в тестовый API;

API предоставляет одну или несколько операций. В этом разделе добавьте операцию к созданному пустому API. Вызов операции после завершения действий в этом разделе вызовет ошибку. Если завершить все действия из раздела [Включение макетирования ответа](#enable-response-mocking) позже, ошибки не возникнут.

### <a name="portal"></a>[Портал](#tab/azure-portal)

1. Выберите API, созданный на предыдущем шаге.
1. Щелкните **+ Add Operation** (+ Добавить операцию).
1. В окне **Интерфейс** введите следующие значения.

     | Параметр             | Значение                             | Description                                                                                                                                                                                   |
    |---------------------|-----------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
    | **Отображаемое имя**    | *Test call* (Пробный вызов)                       | Имя, отображаемое на [портале разработчика](api-management-howto-developer-portal.md).                                                                                                                                       |
    | **URL-адрес** (HTTP-команда) | GET                               | Выберите одну из предварительно заданных HTTP-команд.                                                                                                                                         |
    | **URL-адрес**             | */test*                           | URL-путь для API.                                                                                                                                                                       |
    | **Описание**     |                                   |  Введите описание операции, с помощью которой документация предоставляется разработчикам, которые используют текущее API на портале разработчика.                                                    |
    
1. Выберите вкладку **Ответы**, которая находится под полями "URL-адрес", "Отображаемое имя" и "Описание". Введите параметры на этой вкладке, чтобы определить коды состояния ответа, типы содержимого, примеры и схемы.
1. Выберите **+ Добавить ответ** и в списке выберите **200 ОК**.
1. Справа от раздела **Представления** выберите **+ Add representation** (+ Добавить представление).
1. В поле поиска введите *application/json* и выберите тип содержимого **application/json**.
1. В текстовом поле **Образец** введите `{ "sampleField" : "test" }`.
1. Щелкните **Сохранить**.

:::image type="content" source="media/mock-api-responses/03-mock-api-responses-02-add-operation.png" alt-text="Добавление операции API" border="false":::

Хотя в данном примере это не требуется, дополнительные параметры для операции API можно настроить на других вкладках, в том числе:


|Вкладка      |Описание  |
|---------|---------|
|**Запрос**     |  Добавьте параметры запроса. Помимо имени и описания можно указать значения, привязанные к данному параметру запроса. Одно из значений может быть отмечено как значение по умолчанию (необязательно).        |
|**Запрос**     |  Задайте типы содержимого, примеры и схемы запросов.       |

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы начать работу с Azure CLI:

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

Чтобы добавить операцию в API тестирования, выполните команду [az apim api operation create](/cli/azure/apim/api/operation#az_apim_api_operation_create):

```azurecli
az apim api operation create --resource-group apim-hello-word-resource-group \
    --display-name "Test call" --api-id test-api --method GET \
    --url-template /test --service-name apim-hello-world 
```

Чтобы просмотреть все операции для API, выполните команду [az apim api operation list](/cli/azure/apim/api/operation#az_apim_api_operation_list):

```azurecli
az apim api operation list --resource-group apim-hello-word-resource-group \
    --api-id test-api --service-name apim-hello-world --output table
```

Чтобы удалить операцию, воспользуйтесь командой [az apim api operation delete](/cli/azure/apim/api/operation#az_apim_api_operation_delete). Получите идентификатор операции из предыдущей команды.

```azurecli
az apim api operation delete --resource-group apim-hello-word-resource-group \
    --api-id test-api --operation-id 00000000000000000000000000000000 \
    --service-name apim-hello-world
```

Сохраните эту операцию для последующего использования при работе с этой статьей.

---

## <a name="enable-response-mocking"></a>включение макетирования ответа;

1. Выберите API, созданный на этапе [Создание тестового API](#create-a-test-api).
1. Выберите добавленную тестовую операцию.
1. В окне справа убедитесь, что выбрана вкладка **Конструктор**.
1. Нажмите **+ Добавить политику** в окне **Обработка входящих запросов**.

    :::image type="content" source="media/mock-api-responses/03-mock-api-responses-03-enable-mocking.png" alt-text="Добавление политики обработки" border="false":::

1. Выберите **Имитировать ответы** из коллекции.

    :::image type="content" source="media/mock-api-responses/mock-responses-policy-tile.png" alt-text="Плитка политики макетирования ответов" border="false":::

1. В текстовом поле **API Management response** (Ответ службы управления API) введите **200 OK, application/json**. Этот параметр указывает, что API должен возвращать пример ответа, определенный в предыдущем разделе.

    :::image type="content" source="media/mock-api-responses/mock-api-responses-set-mocking.png" alt-text="Задание имитации ответа":::

1. Щелкните **Сохранить**.

    > [!TIP]
    > Желтая панель с текстом **Имитирование включено** указывает, что ответы, возвращаемые из управления API, созданы [политикой имитирования](api-management-advanced-policies.md#mock-response), а не реальной внутренней частью приложения.

## <a name="test-the-mocked-api"></a>тестирование макетирования API.

1. Выберите API, созданный на этапе [Создание тестового API](#create-a-test-api).
1. Откройте вкладку **Тест**.
1. Убедитесь, что выбран API **пробного вызова**. Чтобы сделать пробный вызов, выберите **Отправка**.

   :::image type="content" source="media/mock-api-responses/03-mock-api-responses-04-test-mocking.png" alt-text="тестирование макетирования API.":::

1. В качестве образца в первом разделе руководства отображается **HTTP-ответ** JSON.

    :::image type="content" source="media/mock-api-responses/mock-api-responses-test-response.png" alt-text="Имитация HTTP-ответа":::

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как выполнять следующие задачи:

> [!div class="checklist"]
> * создание тестового API;
> * добавление операции в тестовый API;
> * включение макетирования ответа;
> * тестирование макетирования API.

Перейдите к следующему руководству:

> [!div class="nextstepaction"]
> [Преобразование и защита опубликованного API](transform-api.md)
