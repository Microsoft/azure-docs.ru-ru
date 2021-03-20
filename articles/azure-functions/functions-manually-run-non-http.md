---
title: Запуск Функций Azure, не активируемых HTTP-запросом, вручную
description: Запуск Функций Azure, не активируемых HTTP-запросом, с использованием HTTP-запроса
author: craigshoemaker
ms.topic: article
ms.date: 04/23/2020
ms.author: cshoe
ms.openlocfilehash: 79aebf7ed80fea370ff7a5d5cc40911da4144414
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91537707"
---
# <a name="manually-run-a-non-http-triggered-function"></a>Запуск функции, не активируемой HTTP-запросом, вручную

В этой статье показано, как вручную запускать функцию, не активируемую HTTP-запросом, через HTTP-запрос в особом формате.

В некоторых контекстах может потребоваться запускать Функцию Azure по требованию, которая активируется косвенно.  Примерами косвенных триггеров являются [функции по расписанию](./functions-create-scheduled-function.md) или функции, которые запускаются как результат [действия другого ресурса](./functions-create-storage-blob-triggered-function.md). 

В следующем примере применяется [Postman](https://www.getpostman.com/), но вы можете использовать для отправки HTTP-запросов [cURL](https://curl.haxx.se/), [Fiddler](https://www.telerik.com/fiddler) или любой другой похожий инструмент.

## <a name="define-the-request-location"></a>Определение расположения запроса

Чтобы запустить функцию, не активируемую HTTP-запросом, необходим способ отправки запроса в Azure для запуска функции. URL-адрес, используемый для выполнения данного запроса, принимает определенный формат.

![Определение расположения запроса: имя узла + путь к папке + имя функции](./media/functions-manually-run-non-http/azure-functions-admin-url-anatomy.png)

- **Имя узла:** Общедоступное расположение приложения-функции, которое состоит из имени приложения-функции, а также *azurewebsites.NET* или личного домена.
- **Путь к папке:** Чтобы получить доступ к функциям, не активируемым HTTP, через HTTP-запрос, необходимо отправить запрос через папку *"Администраторы" или "функции"*.
- **Имя функции:** Имя функции, которую необходимо выполнить.

Используйте это расположение запроса в Postman вместе с главным ключом функции в запросе к Azure для запуска функции.

> [!NOTE]
> При локальном запуске функции главный ключ не требуется. Вы можете напрямую [вызвать функцию](#call-the-function), пропустив заголовок `x-functions-key`.

## <a name="get-the-functions-master-key"></a>Получение главного ключа функции

1. Перейдите к приложению функции в [портал Azure](https://portal.azure.com), выберите **ключи приложения**, а затем — `_master` ключ. 

    :::image type="content" source="./media/functions-manually-run-non-http/azure-portal-functions-master-key.png" alt-text="Нахождение копируемого главного ключа." border="true":::

1. В разделе **изменение ключа** скопируйте значение ключа в буфер обмена и нажмите кнопку **ОК**.

    :::image type="content" source="./media/functions-manually-run-non-http/azure-portal-functions-master-key-copy.png" alt-text="Скопируйте главный ключ в буфер обмена." border="true":::

1. После копирования ключа *_Master* выберите **код + тест**, а затем выберите **журналы**. Вы увидите сообщения из функции, которые записывались сюда, когда вы вручную запускали функцию из Postman.

    :::image type="content" source="./media/functions-manually-run-non-http/azure-portal-function-log.png" alt-text="Снимок экрана, на котором показана страница &quot;код + тест&quot; с сообщением из журналов." border="true":::

> [!CAUTION]  
> Главный ключ предоставляет высокий уровень разрешений в приложении-функции, поэтому никогда не передавайте этот ключ третьим лицам и не включайте его в состав приложения. Ключ должен быть отправлен только в конечную точку HTTPS.

## <a name="call-the-function"></a>Вызов функции

Откройте Postman и выполните следующие действия:

1. В текстовом поле URL-адреса введите **запрашиваемое расположение**.
1. Для метода HTTP задается тип **POST**.
1. Выберите вкладку **Headers** (Заголовки).
1. Введите **x-functions-Key** в качестве первого ключа и вставьте в качестве значения главный ключ (из буфера обмена).
1. Введите **Content-Type** в качестве второго ключа и введите в качестве значения **Application/JSON** .

    :::image type="content" source="./media/functions-manually-run-non-http/functions-manually-run-non-http-headers.png" alt-text="Параметры после заголовков." border="true":::

1. Откройте вкладку **Тело**.
1. Введите **{"input": "Test"}** в качестве текста для запроса.

    :::image type="content" source="./media/functions-manually-run-non-http/functions-manually-run-non-http-body.png" alt-text="Параметры тела сообщения." border="true":::

1. Нажмите кнопку **Отправить**.
        
    :::image type="content" source="./media/functions-manually-run-non-http/functions-manually-run-non-http-send.png" alt-text="Отправить запрос с помощью POST." border="true":::

    Postman затем сообщает о состоянии **202 — принято**.

1. Далее перейдите к своей функции на портале Azure. Просмотрите журналы, и вы увидите сообщения, поступающие из ручного вызова функции.

    :::image type="content" source="./media/functions-manually-run-non-http/azure-portal-functions-master-key-logs.png" alt-text="Просмотрите журналы, чтобы просмотреть результаты тестирования главного ключа." border="true":::

## <a name="next-steps"></a>Дальнейшие действия

- [Методика тестирования кода с помощью Функций Azure](./functions-test-a-function.md)
- [Локальная отладка триггера Сетки событий для функций Azure](./functions-debug-event-grid-trigger-local.md)
