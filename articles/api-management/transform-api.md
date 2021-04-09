---
title: Учебник — преобразование и защита API с помощью службы управления API Azure | Документация Майкрософт
description: В этом руководстве вы узнаете, как защитить API в службе управления API с помощью политик преобразования и регулирования (ограничение скорости).
author: vladvino
ms.service: api-management
ms.custom: mvc
ms.topic: tutorial
ms.date: 09/28/2020
ms.author: apimpm
ms.openlocfilehash: 979bdaa1e0dac4f45a321abda2a208f46983f9cd
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96010237"
---
# <a name="tutorial-transform-and-protect-your-api"></a>Руководство по Преобразование и защита API

В этом руководстве показано, как преобразовать API, чтобы он не раскрывал сведения о серверной части. Например, иногда нужно скрыть сведения о технологических решениях, которые применяются в серверной части. Или же нужно скрыть исходный URL-адрес, присутствующий в тексте HTTP-ответа от API, указав вместо него шлюз службы управления API.

Также в этом руководстве показано, насколько легко можно защитить серверный API, настроив ограничения скорости в службе управления API Azure. Например, может потребоваться ограничить частоту вызовов API, чтобы разработчик не использовал API. Дополнительные сведения см. в статье [Обработка ошибок в политиках управления API](api-management-policies.md).

В этом руководстве описано следующее:

> [!div class="checklist"]
>
> -   преобразование API для удаления заголовков ответа;
> -   замена исходных URL-адресов в тексте ответа API URL-адресом шлюза службы управления API;
> -   защита API путем добавления политик ограничения скорости (регулирования);
> -   проверка преобразований.

:::image type="content" source="media/transform-api/api-management-management-console.png" alt-text="Политики на портале":::

## <a name="prerequisites"></a>Предварительные требования

-   Ознакомьтесь с [терминологией службы управления API в Azure](api-management-terminology.md).
-   Ознакомьтесь с [концепцией использования политик в службе управления API в Azure](api-management-howto-policies.md).
-   Выполните инструкции из краткого руководства [Создание экземпляра службы управления API Azure](get-started-create-service-instance.md).
-   Также выполните инструкции из руководства [Импорт и публикация первого API](import-and-publish.md).

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="transform-an-api-to-strip-response-headers"></a>преобразование API для удаления заголовков ответа;

В этом разделе показано, как скрыть заголовки HTTP, которые вы не хотите показывать пользователям. В этом примере показано, как удалить следующие заголовки в HTTP-ответе:

-   **X-Powered-By**,
-   **X-AspNet-Version**.

### <a name="test-the-original-response"></a>Проверка исходного ответа

Чтобы увидеть исходный ответ, сделайте следующее.

1. В своем экземпляре службы управления API выберите **API**.
1. В списке API выберите **Demo Conference API**.
1. Выберите вкладку **Тест** в верхней части экрана.
1. Выберите операцию **GetSpeakers**, а затем выберите **Отправить**.

Исходный ответ должен выглядеть так:

:::image type="content" source="media/transform-api/original-response.png" alt-text="Исходный ответ API":::

Как видите, ответ включает заголовки **X-AspNet-Version** и **X-Powered-By**.

### <a name="set-the-transformation-policy"></a>Настройка политики преобразования

1. Выберите **Demo Conference API** > **Конструктор** > **Все операции**.
4. В разделе **Обработка исходящих запросов** выберите значок редактора кода ( **</>** ).

   :::image type="content" source="media/transform-api/04-ProtectYourAPI-01-SetPolicy-Outbound.png" alt-text="Переход к политике исходящих запросов" border="false":::

1. Установите курсор внутри элемента **&lt;outbound&gt;** и нажмите кнопку **Показать фрагменты слайдов** в правом верхнем углу.
1. На панели справа в разделе **Transformation policies** (Политики преобразования) дважды щелкните **Set HTTP header** (Установка HTTP-заголовка), чтобы вставить два фрагмента политики.

   :::image type="content" source="media/transform-api/transform-api.png" alt-text="Политика &quot;Установка HTTP-заголовка&quot;":::

1. Измените код элемента **\<outbound>** , чтобы он выглядел так:

   ```
   <set-header name="X-Powered-By" exists-action="delete" />
   <set-header name="X-AspNet-Version" exists-action="delete" />
   ```

   :::image type="content" source="media/transform-api/set-policy.png" alt-text="Установка заголовка HTTP":::

1. Щелкните **Сохранить**.

## <a name="replace-original-urls-in-the-body-of-the-api-response-with-apim-gateway-urls"></a>замена исходных URL-адресов в тексте ответа API URL-адресом шлюза службы управления API;

В этом разделе показано, как скрыть исходный URL-адрес, присутствующий в тексте HTTP-ответа от API, указав вместо него шлюз службы управления API.

### <a name="test-the-original-response"></a>Проверка исходного ответа

Чтобы увидеть исходный ответ, сделайте следующее.

1. Выберите **Demo Conference API** > **Тест**.
1. Выберите операцию **GetSpeakers**, а затем выберите **Отправить**.

    Как видите, ответ включает исходные URL-адреса серверной части:

    :::image type="content" source="media/transform-api/original-response2.png" alt-text="Исходные URL-адреса в ответе":::


### <a name="set-the-transformation-policy"></a>Настройка политики преобразования

1.  Выберите **Demo Conference API** > **Все операции** > **Конструктор**.
1.  В разделе **Обработка исходящих запросов** выберите значок редактора кода ( **</>** ).
1.  Установите курсор внутри элемента **&lt;outbound&gt;** и нажмите кнопку **Показать фрагменты слайдов** в правом верхнем углу.
1.  В окне справа в разделе **Политики преобразования** выберите **Mask URLs in content** (Маскировка URL-адресов в содержимом). 
1.  Щелкните **Сохранить**.

## <a name="protect-an-api-by-adding-rate-limit-policy-throttling"></a>защита API путем добавления политик ограничения скорости (регулирования);

В этом разделе показано, как добавить защиту для API внутреннего сервера, настроив для него ограничения скорости. Например, может потребоваться ограничить частоту вызовов API, чтобы разработчик не использовал API. В этом примере установлено ограничение в 3 вызова за 15 секунд для каждого идентификатора подписки. Через 15 секунд разработчик может снова повторять вызов API.

1.  Выберите **Demo Conference API** > **Все операции** > **Конструктор**.
1.  В разделе **Обработка исходящих запросов** выберите значок редактора кода ( **</>** ).
1.  Установите курсор внутри элемента **&lt;inbound&gt;** и нажмите кнопку **Show snippets** (Показать фрагменты) в правом верхнем углу.

    :::image type="content" source="media/transform-api/04-ProtectYourAPI-01-SetPolicy-Inbound.png" alt-text="Настройка политики входящего трафика" border="false":::

1.  На панели справа в разделе **Политики ограничения доступа** выберите **+ Ограничить частоту вызовов для одного ключа**.
1.  Замените код **rate-limit-by-key** в элементе **\<inbound\>** следующим фрагментом:

    ```
    <rate-limit-by-key calls="3" renewal-period="15" counter-key="@(context.Subscription.Id)" />
    ```

## <a name="test-the-transformations"></a>проверка преобразований.

Если на этом этапе взглянуть на код в редакторе, ваши политики будут выглядеть так:

   ```
   <policies>
      <inbound>
        <rate-limit-by-key calls="3" renewal-period="15" counter-key="@(context.Subscription.Id)" />
        <base />
      </inbound>
      <backend>
        <base />
      </backend>
      <outbound>
        <set-header name="X-Powered-By" exists-action="delete" />
        <set-header name="X-AspNet-Version" exists-action="delete" />
        <redirect-content-urls />
        <base />
      </outbound>
      <on-error>
        <base />
      </on-error>
   </policies>
   ```

Оставшаяся часть этого раздела посвящена проверке преобразований политики, которые вы задали ранее.

### <a name="test-the-stripped-response-headers"></a>Проверка очистки заголовков в ответе

1. Выберите **Demo Conference API** > **Тест**.
1. Выберите операцию **GetSpeakers**, а затем выберите **Отправить**.

    Здесь вы увидите, что заголовки удалены из ответа:

    :::image type="content" source="media/transform-api/final-response1.png" alt-text="Очищенный заголовок ответа":::

### <a name="test-the-replaced-url"></a>Проверка замены URL-адреса

1. Выберите **Demo Conference API** > **Тест**.
1. Выберите операцию **GetSpeakers**, а затем выберите **Отправить**.

    Здесь видно, что URL-адрес успешно заменен.

    :::image type="content" source="media/transform-api/final-response2.png" alt-text="URL-адрес заменен":::

### <a name="test-the-rate-limit-throttling"></a>Проверка предела скорости (регулирования)

1. Выберите **Demo Conference API** > **Тест**.
1. Щелкните операцию **GetSpeakers**. Нажмите **Отправить** три раза подряд.

    После третьего запроса вы получите ответ **429 — слишком много запросов**.

    :::image type="content" source="media/transform-api/test-throttling.png" alt-text="Слишком много запросов":::

1. Подождите примерно 15 секунд и снова нажмите **Отправить**. Теперь вы должны получить ответ **200 — ОК**.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как выполнять следующие задачи:

> [!div class="checklist"]
>
> -   преобразование API для удаления заголовков ответа;
> -   замена исходных URL-адресов в тексте ответа API URL-адресом шлюза службы управления API;
> -   защита API путем добавления политик ограничения скорости (регулирования);
> -   проверка преобразований.

Перейдите к следующему руководству:

> [!div class="nextstepaction"]
> [Monitor your API](api-management-howto-use-azure-monitor.md) (Мониторинг API)
