---
title: Создание функции, активируемой хранилищем BLOB-объектов, в Azure
description: Создавайте независимые от сервера функции, активируемые элементами, добавляемыми в контейнер хранилища BLOB-объектов.
ms.assetid: d6bff41c-a624-40c1-bbc7-80590df29ded
ms.topic: how-to
ms.date: 10/01/2018
ms.custom: mvc, cc996988-fb4f-47
ms.openlocfilehash: bf6865d2756579f457dded90b247326d2eec137c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "83123128"
---
# <a name="create-a-function-in-azure-thats-triggered-by-blob-storage"></a>Создание функции, активируемой хранилищем BLOB-объектов, в Azure

Узнайте, как создавать функцию, активируемую при добавлении файлов или их обновлении в контейнере хранилища BLOB-объектов.

## <a name="prerequisites"></a>Предварительные требования

+ Подписка Azure. Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начать работу.

## <a name="create-an-azure-function-app"></a>Создание приложения-функции Azure

[!INCLUDE [Create function app Azure portal](../../includes/functions-create-function-app-portal.md)]

Вы успешно создали новое приложение-функцию.

:::image type="content" source="./media/functions-create-storage-blob-triggered-function/function-app-create-success.png" alt-text="Приложение-функция успешно создана." border="true":::

Затем создайте функцию в новом приложении-функции.

<a name="create-function"></a>

## <a name="create-an-azure-blob-storage-triggered-function"></a>Создание функции, активируемой хранилищем BLOB-объектов Azure

1. Выберите **Функции**, а затем выберите **+ Добавить**, чтобы добавить новую функцию.

   :::image type="content" source="./media/functions-create-storage-blob-triggered-function/function-app-quickstart-choose-template.png" alt-text="Выберите шаблон функции на портале Azure." border="true":::

1. Выберите шаблон **Триггер для хранилища BLOB-объектов Azure**.

1. Затем используйте настройки, указанные в таблице под изображением.

    :::image type="content" source="./media/functions-create-storage-blob-triggered-function/functions-create-blob-storage-trigger-portal-2.png" alt-text="Присвойте имя функции, активируемой хранилищем BLOB-объектов, и настройте ее." border="true":::

    | Параметр | Рекомендуемое значение | Описание |
    |---|---|---|
    | **Новая функция** | Уникальное для вашего приложения-функции | Имя функции, активируемой большим двоичным объектом. |
    | **Путь**   | samples-workitems/{name}    | Расположение в хранилище BLOB-объектов отслеживается. Имя файла большого двоичного объекта передается в привязке как параметр _name_.  |
    | **Подключение к учетной записи хранения** | AzureWebJobsStorage | Вы можете использовать подключение к учетной записи хранения, которое уже используется вашим приложением-функцией, или создать его.  |

1. Выберите **Создать функцию**, чтобы создать функцию.

    :::image type="content" source="./media/functions-create-storage-blob-triggered-function/functions-create-blob-storage-trigger-portal-3.png" alt-text="Создайте функцию, активируемую хранилищем BLOB-объектов." border="true":::

Затем создайте контейнер **samples-workitems**.

## <a name="create-the-container"></a>Создание контейнера

1. В функции на странице **Обзор** выберите группу ресурсов.

    :::image type="content" source="./media/functions-create-storage-blob-triggered-function/functions-storage-resource-group.png" alt-text="Выберите группу ресурсов портала Azure." border="true":::

1. Найдите и выберите учетную запись хранения своей группы ресурсов.

    :::image type="content" source="./media/functions-create-storage-blob-triggered-function/functions-storage-account-access.png" alt-text="Перейдите к учетной записи хранения." border="true":::

1. Выберите **Контейнеры**, а затем нажмите **+ Контейнер**. 

    :::image type="content" source="./media/functions-create-storage-blob-triggered-function/functions-storage-add-container.png" alt-text="Добавьте контейнер в свою учетную запись хранения на портале Azure." border="true":::

1. В поле **Имя** введите `samples-workitems` и нажмите кнопку **Создать**.

    :::image type="content" source="./media/functions-create-storage-blob-triggered-function/functions-storage-name-blob-container.png" alt-text="Присвойте имя контейнеру хранилища." border="true":::

Теперь, когда у вас есть контейнер больших двоичных объектов, вы можете проверить функцию, отправив файл в контейнер.

## <a name="test-the-function"></a>Проверка функции

1. На портале Azure перейдите к вашей функции, в нижней части страницы разверните **Журналы** и убедитесь, что потоковая передача журналов не остановлена.

    :::image type="content" source="./media/functions-create-storage-blob-triggered-function/functions-storage-log-expander.png" alt-text="Разверните журнал на портале Azure." border="true":::

1. В отдельном окне браузера перейдите к группе ресурсов на портале Azure и выберите учетную запись хранения.

1. Нажмите **Контейнеры**, а затем выберите контейнер **samples-workitems**.

    :::image type="content" source="./media/functions-create-storage-blob-triggered-function/functions-storage-container.png" alt-text="Перейдите к контейнеру samples-workitems на портале Azure." border="true":::

1. Выберите **Отправить**, а затем щелкните значок папки, чтобы выбрать файл для отправки.

    :::image type="content" source="./media/functions-create-storage-blob-triggered-function/functions-storage-manager-upload-file-blob.png" alt-text="Отправьте файл в контейнер больших двоичных объектов." border="true":::

1. Перейдите к файлу на вашем локальном компьютере, например к файлу изображения, и выберите его. Выберите **Открыть**, затем — **Передать**.

1. Вернитесь к журналам функции и убедитесь, что большой двоичный объект был считан.

    :::image type="content" source="./media/functions-create-storage-blob-triggered-function/function-app-in-portal-editor.png" alt-text="Просмотр сообщения в журналах." border="true":::

    >[!NOTE]
    > Если ваше приложение-функция выполняется в рамках плана потребления по умолчанию, между добавлением или обновлением большого двоичного объекта и активацией функции могут возникнуть задержки до нескольких минут. Выполняйте свое приложение-функцию в рамках плана службы приложений, если требуется малая задержка в функции, активируемой большим двоичным объектом.

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [Next steps note](../../includes/functions-quickstart-cleanup.md)]

## <a name="next-steps"></a>Дальнейшие действия

Вы создали функцию, которая выполняется при добавлении или обновлении большого двоичного объекта в хранилище BLOB-объектов. Дополнительные сведения о триггерах хранилища BLOB-объектов см.в статье [Привязки больших двоичных объектов службы хранилища для Функций Azure](functions-bindings-storage-blob.md).

[!INCLUDE [Next steps note](../../includes/functions-quickstart-next-steps.md)]
