---
title: Создание функции, активируемой сообщениями очереди, в Azure
description: Используйте Функции Azure, чтобы создавать независимые от сервера функции, активируемые сообщениями, отправленными в очередь в Azure.
ms.assetid: 361da2a4-15d1-4903-bdc4-cc4b27fc3ff4
ms.topic: how-to
ms.date: 10/01/2018
ms.custom: mvc, cc996988-fb4f-47
ms.openlocfilehash: d722d420597bb459d3e7b6d2ca33fdc49bfe6f09
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90981580"
---
# <a name="create-a-function-triggered-by-azure-queue-storage"></a>Создание функции, активируемой хранилищем очередей Azure

Узнайте, как создать функцию, которая активируется при отправке сообщений в очередь Службы хранилища Azure.

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure. Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начать работу.

## <a name="create-an-azure-function-app"></a>Создание приложения-функции Azure

[!INCLUDE [Create function app Azure portal](../../includes/functions-create-function-app-portal.md)]

   :::image type="content" source="./media/functions-create-storage-queue-triggered-function/function-app-create-success.png" alt-text="Приложение-функция успешно создана." border="true":::

Затем создайте функцию в новом приложении-функции.

<a name="create-function"></a>

## <a name="create-a-queue-triggered-function"></a>Создание функции, активируемой очередью

1. Выберите **Функции**, а затем выберите **+ Добавить**, чтобы добавить новую функцию.

   :::image type="content" source="./media/functions-create-storage-queue-triggered-function/functions-app-quickstart-choose-template.png" alt-text="Выберите шаблон функции на портале Azure." border="true":::

1. Выберите шаблон **Триггер для хранилища очередей Azure**.

1. Затем используйте настройки, указанные в таблице под изображением.

    :::image type="content" source="./media/functions-create-storage-queue-triggered-function/functions-create-queue-storage-trigger-portal.png" alt-text="Присвойте имя функции, активируемой хранилищем очередей и настройте ее." border="true":::


    | Параметр | Рекомендуемое значение | Описание |
    |---|---|---|
    | **имя**; | Уникальное для вашего приложения-функции | Имя функции, активируемой очередью. |
    | **Имя очереди**   | myqueue-items    | Имя очереди для подключения к вашей учетной записи хранения. |
    | **Подключение к учетной записи хранения** | AzureWebJobsStorage | Вы можете использовать подключение к учетной записи хранения, которое уже используется вашим приложением-функцией, или создать его.  |    

1. Выберите **Создать функцию**, чтобы создать функцию.

    :::image type="content" source="./media/functions-create-storage-queue-triggered-function/functions-create-queue-storage-trigger-portal-3.png" alt-text="Создайте функцию, активируемую хранилищем очередей." border="true":::

Затем необходимо подключиться к своей учетной записи хранения Azure и создать очередь службы хранилища **myqueue-items**.

## <a name="create-the-queue"></a>Создание очереди

1. В функции на странице **Обзор** выберите группу ресурсов.

    :::image type="content" source="./media/functions-create-storage-queue-triggered-function/functions-storage-resource-group.png" alt-text="Выберите группу ресурсов портала Azure." border="true":::

1. Найдите и выберите учетную запись хранения своей группы ресурсов.

    :::image type="content" source="./media/functions-create-storage-queue-triggered-function/functions-storage-account-access.png" alt-text="Перейдите к учетной записи хранения." border="true":::

1. Выберите **Очереди**, а затем нажмите **+ Очередь**. 

    :::image type="content" source="./media/functions-create-storage-queue-triggered-function/functions-storage-add-queue.png" alt-text="Добавьте очередь в свою учетную запись хранения на портале Azure." border="true":::

1. В поле **Имя** введите `myqueue-items` и нажмите кнопку **Создать**.

    :::image type="content" source="./media/functions-create-storage-queue-triggered-function/functions-storage-name-queue.png" alt-text="Присвойте имя контейнеру хранилища очередей." border="true":::

Теперь, когда вы создали очередь хранилища, испытайте функцию, добавив сообщение в очередь.

## <a name="test-the-function"></a>Проверка функции

1. На портале Azure перейдите к вашей функции, в нижней части страницы разверните **Журналы** и убедитесь, что потоковая передача журналов не остановлена.

    :::image type="content" source="./media/functions-create-storage-queue-triggered-function/functions-queue-storage-log-expander.png" alt-text="Разверните журнал на портале Azure." border="true":::

1. В отдельном окне браузера перейдите к группе ресурсов на портале Azure и выберите учетную запись хранения.

1. Нажмите **Очереди**, а затем выберите контейнер **myqueue-items**.

    :::image type="content" source="./media/functions-create-storage-queue-triggered-function/functions-storage-queue.png" alt-text="Перейдите к очереди myqueue-items на портале Azure." border="true":::

1. Выберите **Добавить сообщение** и введите "Hello World!" в поле **Текст сообщения**. Щелкните **ОК**.

    :::image type="content" source="./media/functions-create-storage-queue-triggered-function/functions-storage-queue-test.png" alt-text="На снимке экрана показана выбранная кнопка Добавить сообщение и выделенное текстовое поле сообщения." border="true":::

1. Подождите несколько секунд, а затем вернитесь в журналы функции и убедитесь, что новое сообщение было считано из очереди.

    :::image type="content" source="./media/functions-create-storage-queue-triggered-function/function-app-in-portal-editor.png" alt-text="Просмотрите сообщение в журналах." border="true":::

1. Вернитесь к очереди хранилища, щелкните **Обновить** и убедитесь, что сообщение было обработано и больше не находится в очереди.

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [Next steps note](../../includes/functions-quickstart-cleanup.md)]

## <a name="next-steps"></a>Дальнейшие действия

Вы создали функцию, которая выполняется при добавлении сообщения в очередь хранилища. Дополнительные сведения о триггерах хранилища очередей см. в статье [Привязки очередей службы хранилища для Функций Azure](functions-bindings-storage-queue.md).

Теперь, когда вы создали первую функцию, давайте добавим к ней выходную привязку, которая позволяет записывать сообщения в другую очередь.

> [!div class="nextstepaction"]
> [Добавление сообщений в очередь службы хранилища Azure с помощью Функций](functions-integrate-storage-queue-output-binding.md)
