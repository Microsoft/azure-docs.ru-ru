---
title: Руководство. Импорт интерфейсов API и управление ими — Управление API Azure и Visual Studio Code | Документация Майкрософт
description: Из этого руководства вы узнаете, как импортировать и тестировать интерфейсы API, а также управлять ими с помощью расширения службы "Управление API Azure" для Visual Studio Code.
ms.service: api-management
author: dlepow
ms.author: apimpm
ms.topic: tutorial
ms.date: 12/10/2020
ms.openlocfilehash: 0dea8e43d5f09b84c5795bc257cf3331ad919fcb
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101649552"
---
# <a name="tutorial-use-the-api-management-extension-for-visual-studio-code-to-import-and-manage-apis"></a>Руководство. Импорт интерфейсов API и управление ими с помощью расширения службы "Управление API" для Visual Studio Code

Из этого учебника вы узнаете, как выполнять стандартные операции по управлению API с помощью расширения службы "Управление API" для Visual Studio Code. Для импорта, обновления, тестирования API и управления ими вы будете использовать знакомую среду Visual Studio Code.

Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * импорт API в Управление API;
> * редактирование API;
> * применение политик службы "Управление API";
> * тестирование API.


:::image type="content" source="media/visual-studio-code-tutorial/tutorial-api-result.png" alt-text="Интерфейс API в расширении службы &quot;Управление API&quot;":::

Общие сведения о дополнительных возможностях службы "Управление API" см. в руководствах по этой службе на [портале Azure](import-and-publish.md).

## <a name="prerequisites"></a>Предварительные требования
- Ознакомьтесь с [терминологией для службы "Управление API Azure"](api-management-terminology.md).
- Обязательно установите [Visual Studio Code](https://code.visualstudio.com/) и последнюю версию [расширения "Управление API Azure" для Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-apimanagement&ssr=false#overview).
- [Создание экземпляра API Management](vscode-create-service-instance.md)

## <a name="import-an-api"></a>Импорт API

В следующем примере спецификация OpenAPI импортируется в формате JSON в службу "Управление API". Корпорация Майкрософт предоставляет используемый в этом примере API серверной части в Azure по адресу `https://conferenceapi.azurewebsites.net?format=json`.

1. В Visual Studio Code на панели действий щелкните значок Azure.
1. В области проводника разверните созданный экземпляр службы "Управление API".
1. Щелкните элемент **Интерфейсы API** правой кнопкой мыши и выберите **Import from OpenAPI Link** (Импорт по ссылке OpenAPI). 
1. При появлении запроса введите указанные ниже значения:
    1. **Ссылка на OpenAPI** для содержимого в формате JSON. В этом примере: *https://conferenceapi.azurewebsites.net?format=json* .
    Это URL-адрес службы, реализующей пример API. Управление API направляет запросы по этому адресу.
    1. **Имя API** (например, *demo-conference-api*), которое является уникальным в экземпляре службы "Управление API". Это имя может содержать только буквы, цифры и дефисы. Первый и последний знаки должны быть буквой или цифрой. Это имя используется в пути для вызова API.

После успешного импорта интерфейс API отобразится в области проводника, а доступные операции API — в узле **Operations** (Операции).

:::image type="content" source="media/visual-studio-code-tutorial/tutorial-api-operations.png" alt-text="Импортированный API в области проводника":::

## <a name="edit-the-api"></a>Редактирование API

API можно редактировать в Visual Studio Code. Например, измените в окне редактора JSON-описание API для Resource Manager, удалив протокол **HTTP**, используемый для доступа к API. Затем выберите **Файл** > **Сохранить**.

:::image type="content" source="media/visual-studio-code-tutorial/import-demo-api.png" alt-text="Изменение JSON-описания":::

Чтобы изменить формат OpenAPI, в области проводника щелкните имя API правой кнопкой мыши и выберите **Edit OpenAPI** (Изменить OpenAPI). Внесите нужные изменения и щелкните **Файл** > **Сохранить**.

## <a name="apply-policies-to-the-api"></a>Применение политик к API 

Служба "Управление API" предоставляет [политики](api-management-policies.md), которые можно настроить для интерфейсов API. Политика — это коллекция правил, которые выполняются последовательно над запросом или ответом API. Политики могут быть глобальными (то есть применяться ко всем API в экземпляре службы "Управление API") или могут быть ограничены определенным API или операцией API.

В этом разделе показано, как применить к API некоторые распространенные политики исходящих запросов, которые преобразуют ответ API. Политики в этом примере изменяют заголовки ответов и скрывают исходные URL-адреса серверной части, которые отображаются в тексте ответа.

1. В области проводника под импортированным API *demo-conference-api* выберите **Policy** (Политика). Файл политики откроется в окне редактора. Этот файл позволяет настроить политику для всех операций в API. 

1. Обновите файл с использованием следующего содержимого в элементе `<outbound>`:
    ```html
    [...]
    <outbound>
        <set-header name="Custom" exists-action="override">
            <value>"My custom value"</value>
        </set-header>
        <set-header name="X-Powered-By" exists-action="delete" />
        <redirect-content-urls />
        <base />
    </outbound>
    [...]
    ```

    * Первая политика `set-header` добавляет настраиваемый заголовок ответа для демонстрационных целей.
    * Вторая политика `set-header` удаляет заголовок **X-Powered-By**, если он существует. Этот заголовок может раскрывать сведения об исполняющей среде, используемой в серверной части API, и издатели часто его удаляют.
    * Политика `redirect-content-urls` перезаписывает (маскирует) ссылки в тексте ответа так, чтобы каждая из них указывала на эквивалентную ссылку через шлюз службы "Управление API".
    
1. Сохраните файл. При появлении запроса выберите **Upload** (Отправить), чтобы отправить файл в облако.

## <a name="test-the-api"></a>Тестирование API

### <a name="get-the-subscription-key"></a>Получение ключа подписки

Чтобы протестировать импортированный API и примененные политики, вам потребуется ключ подписки для экземпляра службы "Управление API".

1. В области проводника щелкните имя экземпляра службы "Управление API" правой кнопкой мыши.
1. Выберите **Copy Subscription Key** (Копировать ключ подписки).

    :::image type="content" source="media/visual-studio-code-tutorial/copy-subscription-key.png" alt-text="Копирование ключа подписки":::

### <a name="test-an-api-operation"></a>Тестирование операции API

1. В области проводника под импортированным API *demo-conference-api* разверните узел **Operations** (Операции).
1. Выберите операцию, например *GetSpeakers*.
1. В окне редактора рядом с **Ocp-Apim-Subscription-Key** замените `{{SubscriptionKey}}` ключом подписки, который вы скопировали.
1. Щелкните **Отправить запрос**. 

:::image type="content" source="media/visual-studio-code-tutorial/test-api.png" alt-text="Отправка запроса API из Visual Studio Code":::

При успешном выполнении запроса серверная часть вернет ответ **200 — ОК** и другие данные.

:::image type="content" source="media/visual-studio-code-tutorial/test-api-policies.png" alt-text="Тестирование операции API":::

В ответе обратите внимание на следующие сведения:
* В ответ добавляется заголовок **Custom**.
* Заголовок **X-Powered-By** не отображается в ответе.
* URL-адреса серверной части API перенаправляются в шлюз службы "Управление API", в нашем случае — `https://apim-hello-world.azure-api.net/demo-conference-api`.

### <a name="trace-the-api-operation"></a>Трассировка операции API

Чтобы получить подробные сведения трассировки для отладки операции API, выберите ссылку, которая отображается рядом с **Ocp-APIM-Trace-Location**. 

JSON-файл в этом расположении содержит сведения о трассировке из разделов Inbound (Входящие запросы), Backend (Запросы к серверной части) и Outbound (Исходящие запросы), позволяющие определить, где возникают проблемы после отправки запроса.

> [!TIP]
> При тестировании операций API расширение службы "Управление API" позволяет выполнять дополнительную [отладку политик](api-management-debug-policies.md) (доступна на уровне служб "Разработка").

## <a name="clean-up-resources"></a>Очистка ресурсов

Удалите экземпляр службы "Управление API", щелкнув правой кнопкой мыши и выбрав пункт **Открыть на портале**, чтобы [удалить службу Управление API](get-started-create-service-instance.md#clean-up-resources) и связанную с ней группу ресурсов.

Кроме того, можно нажать кнопку **Delete API Management** (Удалить службу "Управление API"), чтобы удалить только экземпляр службы "Управление API" (без группы ресурсов).

:::image type="content" source="media/visual-studio-code-tutorial/vscode-apim-delete.png" alt-text="Удаление экземпляра службы &quot;Управление API&quot; из VS Code":::

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали о некоторых возможностях расширения "Управление API" для Visual Studio Code, с помощью которых можно импортировать API и управлять ими. Вы ознакомились с выполнением следующих задач:

> [!div class="checklist"]
> * импорт API в Управление API;
> * редактирование API;
> * применение политик службы "Управление API";
> * тестирование API.

Расширение службы "Управление API" предоставляет дополнительные функции для работы с интерфейсами API. Например, [политики отладки](api-management-debug-policies.md) (доступные на уровне служб "Разработка") или создание [именованных значений](api-management-howto-properties.md) и управление ими.