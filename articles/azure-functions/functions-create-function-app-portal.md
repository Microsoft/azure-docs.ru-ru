---
title: Создание первой функции на портале Azure
description: Узнайте, как создать первую функцию Azure, выполняемую без сервера, с помощью портала Azure.
ms.topic: how-to
ms.date: 03/26/2020
ms.custom: devx-track-csharp, mvc, devcenter, cc996988-fb4f-47
ms.openlocfilehash: 6ca187181d68d7924f97ebfac08e81dec77251e9
ms.sourcegitcommit: 4bda786435578ec7d6d94c72ca8642ce47ac628a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103492144"
---
# <a name="create-your-first-function-in-the-azure-portal"></a>Создание первой функции на портале Azure

Функции Azure позволяют выполнять код в бессерверной среде без необходимости сначала создавать виртуальную машину или публиковать веб-приложение. Из этой статьи вы узнаете, как использовать функции Azure для создания функции триггера HTTP "Hello World" в портал Azure.

[!INCLUDE [functions-in-portal-editing-note](../../includes/functions-in-portal-editing-note.md)] 

Вместо этого рекомендуется [разрабатывать функции локально](functions-develop-local.md) и публиковать их в приложении-функции в Azure.  
Используйте одну из следующих ссылок, чтобы приступить к работе с выбранной локальной средой разработки и языком:

| Visual Studio Code | Терминал / командная строка | Visual Studio |
| --- | --- | --- |
|  &bull;&nbsp;[Начало работы с C #](./create-first-function-vs-code-csharp.md)<br/>&bull;&nbsp;[Начало работы с Java](./create-first-function-vs-code-java.md)<br/>&bull;&nbsp;[Приступая к работе с JavaScript](./create-first-function-vs-code-node.md)<br/>&bull;&nbsp;[Начало работы с PowerShell](./create-first-function-vs-code-powershell.md)<br/>&bull;&nbsp;[Начало работы с Python](./create-first-function-vs-code-python.md) |&bull;&nbsp;[Начало работы с C #](./create-first-function-cli-csharp.md)<br/>&bull;&nbsp;[Начало работы с Java](./create-first-function-cli-java.md)<br/>&bull;&nbsp;[Приступая к работе с JavaScript](./create-first-function-cli-node.md)<br/>&bull;&nbsp;[Начало работы с PowerShell](./create-first-function-cli-powershell.md)<br/>&bull;&nbsp;[Начало работы с Python](./create-first-function-cli-python.md) | [Приступая к работе с C#](functions-create-your-first-function-visual-studio.md) |

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="sign-in-to-azure"></a>Вход в Azure

Войдите на [портал Azure](https://portal.azure.com) с помощью своей учетной записи Azure.

## <a name="create-a-function-app"></a>Создание приложения-функции

Для выполнения функций вам понадобится приложение-функция, позволяющее группировать функции в логические единицы и упростить развертывание, масштабирование и совместное использование ресурсов, а также управление ими.

[!INCLUDE [Create function app Azure portal](../../includes/functions-create-function-app-portal.md)]

Затем создайте функцию в новом приложении-функции.

## <a name="create-an-http-trigger-function"></a><a name="create-function"></a>Создание функции, активируемой HTTP

1. В меню слева в окне **Функции** выберите **Функции**, а затем в верхнем меню выберите **Добавить**. 
 
1. В окне **Добавление функции** выберите шаблон **триггер HTTP** .

    ![Выбор функции для триггера HTTP](./media/functions-create-first-azure-function/function-app-select-http-trigger.png)

1. В разделе **сведения о шаблоне** используйте `HttpExample` для параметра **Новая функция**, выберите **Анонимный** в раскрывающемся списке **[уровень авторизации](functions-bindings-http-webhook-trigger.md#authorization-keys)** и нажмите кнопку **Добавить**.

    Azure создает функцию, активируемую HTTP. Теперь вы можете запустить новую функцию, отправив HTTP-запрос.

## <a name="test-the-function"></a>Проверка функции

1. В новой функции, активируемой HTTP, выберите **Code + Test** (Код + Тест) в меню слева, а затем в верхнем меню выберите **Получить URL-адрес функции**.

    ![Выбор "Получить URL-адрес функции"](./media/functions-create-first-azure-function/function-app-select-get-function-url.png)

1. В диалоговом окне **Получить URL-адрес функции** в раскрывающемся списке выберите вариант **по умолчанию**, а затем выберите значок **Копировать в буфер обмена**. 

    ![Копирование URL-адреса функции с портала Azure](./media/functions-create-first-azure-function/function-app-develop-tab-testing.png)

1. Вставьте URL-адрес функции в адресную строку браузера. Добавьте значение строки запроса `?name=<your_name>` в конец этого URL-адреса и нажмите клавишу Enter, чтобы выполнить этот запрос. 

    Следующий пример демонстрирует ответ в браузере:

    ![Ответ функции в браузере.](./media/functions-create-first-azure-function/function-app-browser-testing.png)

    Если URL-адрес запроса содержит [ключ доступа](functions-bindings-http-webhook-trigger.md#authorization-keys) ( `?code=...` ), это означает, что при создании функции вы выбираете **функцию** , а не **Анонимный** уровень доступа. В этом случае вместо этого следует добавить `&name=<your_name>` .

1. При выполнении функции сведения о трассировке записываются в журналы. Чтобы просмотреть выходные данные трассировки, вернитесь на страницу **Code + Test** (Код + Тест) на портале и разверните список **Журналы** в нижней части страницы.

   ![Средство просмотра журналов Функций на портале Azure.](./media/functions-create-first-azure-function/function-view-logs.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [Clean-up resources](../../includes/functions-quickstart-cleanup.md)]

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [Next steps note](../../includes/functions-quickstart-next-steps.md)]
